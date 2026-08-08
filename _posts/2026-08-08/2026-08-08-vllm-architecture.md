---
layout: post
title: "【深度分析】vLLM 高效能 LLM 推論系統架構剖析"
date: 2026-08-08 03:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-08/vllm-architecture.jpg)

如果你用過任何一個 LLM API——不管是 OpenAI、Anthropic、還是各種開源模型——你極可能以經在不知不覺中用過 vLLM。它是目前最主流的高效能 LLM 推論引擎，背後支撐著無數 AI 產品的 API endpoint。但 vLLM 真正的價值不在「快」，而在它用一組精心設計的系統抽象，把 GPU 的記憶體管理從「手工藝」變成了「作業系統級的自動化」。Aleksa Gordić 這篇萬字長文從最底層的 GPU kernel 一路拆到分散式多節點部署，是少數能把 vLLM 講得既深又清楚的文章。

## 原文摘要

vLLM 是一個高效能 LLM 推論引擎，Aleksa Gordić（前 DeepMind、Runa AI 創辦人）在本文中以前 V1 引擎（commit 42172ad，2025 年 8 月 9 日）為基準，系統性地拆解了從單 GPU 到多節點分散式部署的完整架構。文章分為五大部分：引擎核心、進階功能、水平擴展、服務層、效能基準。

### 第一部分：LLM 引擎核心

作者從最簡單的離線推論程式碼開始——建立 LLM 實例、呼叫 `generate`、取回結果。這個最簡配置是單 GPU、同步、離線的，但 vLLM 的引擎核心已包含了後續所有擴展需要的基礎抽象。

**引擎建構子（constructor）** 初始化四大組件：vLLM config（控制模型、快取、平行化的所有參數）、processor（將原始輸入轉為 EngineCoreRequest）、engine core client（在單機模式下等同 EngineCore，逐步演進到支援分散式負載平衡的 DPLBAsyncMPClient）、output processor（將 EngineCoreOutput 轉為使用者可見的 RequestOutput）。

EngineCore 本身包含三個核心子系統：Model Executor（驅動 forward pass，從單 GPU 的 UniProcExecutor → 多 GPU 的 MultiProcExecutor → 分散式）、Structured Output Manager（用於 guided decoding）、Scheduler（決定每個 step 哪些請求參與 forward pass）。Scheduler 內部又包含：排程策略（FCFS 或 priority）、waiting 與 running 兩個請求佇列、以及 KV cache manager——PagedAttention 的核心。

**KV cache manager** 維護一個 `free_block_queue`（可用 KV cache block 池，數量可達數十萬，取決於 VRAM 和 block size）。一個標準 transformer layer 的 block size 計算公式為：`2 × block_size(預設16) × num_kv_heads × head_size × dtype_bytes`。

**Worker 初始化流程** 分三步：Init device（分配 CUDA 設備、驗證 VRAM 可用量、設定分散式參數、建立 model_runner 和 InputBatch）；Load model（實例化架構、載入權重、設為 eval 模式、可選 torch.compile）；Initialize KV cache（取得 per-layer KV cache spec、執行 profiling forward pass 計算可用 block 數量、分配並綁定 KV cache tensor、設定 FlashAttention 後端、捕捉 CUDA graph 以降低 kernel launch overhead）。

**Generate 函數** 首先驗證並注入請求：建立唯一請求 ID、tokenize prompt、打包成 EngineCoreRequest、送入 scheduler 的 waiting queue。在同步引擎中，這批初始 prompt 就是唯一要處理的請求；非同步引擎則支援 continuous batching——每個 step 結束後同時考量新舊請求。因為 forward pass 將 batch 攤平為單一超長序列並由自定義 kernel 處理，continuous batching 在同步引擎中也根本上被支援。

**Step 迴圈** 分三階段：
1. Schedule：選擇本 step 要處理的請求（decode 或 chunked prefill）
2. Forward pass：執行模型並採樣 token
3. Postprocess：附加 token ID、detokenize、檢查終止條件（超過 max_tokens、EOS token、stop_token_ids、stop strings），完成後釋放 KV cache block

**Scheduler** 處理兩種工作負載：Prefill（對所有 prompt token 做 forward pass，通常 compute-bound，結束時採樣一個 token）和 Decode（只處理最新一個 token，其餘 KV 已快取，memory-bandwidth-bound）。V1 scheduler 可以在同一 step 內混合兩種請求（V0 只能擇一處理）。Scheduler 優先處理已在 running queue 的 decode 請求，計算所需新 token 數、呼叫 `allocate_slots`、更新 token budget；之後從 waiting queue 取 prefill 請求，檢查 computed blocks（prefix caching）、呼叫 `allocate_slots`、移入 running queue、更新 budget。

**`allocate_slots` 的核心邏輯**：計算所需 block 數（如 17 個新 token → `ceil(17/16) = 2` 個 block）、檢查可用性（不足時可能觸發 preemption，evict 低優先級請求）、從 `free_block_queue` 分配 block 並記錄到 `req_to_blocks` 映射。

**Forward pass 的執行** 包含五個步驟：更新狀態（移除已完成請求、更新 KV cache block 的索引 metadata）、準備輸入（CPU→GPU 拷貝、計算 position、建立 slot_mapping 和 attention metadata）、執行 forward pass（所有序列攤平串接成超長序列，position indices 和 attention mask 確保每條序列只關注自己的 token，實現無 padding 的 continuous batching）、收集最後 token 的 hidden state 並計算 logits、依 sampling config 進行 token 採樣。Forward pass 有兩種模式：eager（標準 PyTorch）和 captured（重播預捕捉的 CUDA graph）。

### 第二部分：進階功能

**Chunked Prefill** 將長 prompt 的 prefill 拆成多個小 chunk，避免單一長請求壟斷整個 step 導致其他請求延遲飆升。實作簡單：限制每個 step 的 new token 上限（`long_prefill_token_threshold`），超過就把 prefill 切成多個 step 執行，只在最後一個 chunk 才採樣新 token。

**Prefix Caching** 是 PagedAttention 最精彩的應用：多個 prompt 共享相同前綴時，KV cache 只計算一次、後續直接重用。機制如下：將 prompt 切成 16-token blocks、對每個完整 block 計算 hash（結合前一個 block 的 hash、當前 tokens 和可選 metadata——MM hash、LoRA ID、cache salt）、將 hash 存入 `cached_block_hash_to_block`。後續請求的 `find_longest_cache_hit` 找到匹配後直接重用 KV block，不需重新計算。block 只有在即將從 `free_block_queue` 被重新分配時才會失效。預設啟用。

**Guided Decoding（FSM）** 透過有限狀態機約束每個 decode step 的 logits，確保只採樣符合語法的 token。支援 regular grammar（正則表達式）到 context-free grammar（大部分程式語言）。實作透過 `StructuredOutputManager` 和第三方後端（如 xgrammar）編譯 grammar、產生 `_grammar_bitmask`（每個 int32 代表 32 個 token 的允許/禁止狀態，大詞表需多個 int32），forward pass 後將禁止 token 的 logits 設為 –∞。

**Speculative Decoding** 用小模型（draft model）快速生成 k 個候選 token，再用大模型一次性驗證。接受/拒絕規則確保統計等價於標準 autoregressive decoding：若大模型對 draft token 的機率 ≥ 小模型機率則接受，否則以 `p_large/p_draft` 的機率接受；在第一個拒絕處停止。若 k 個全接受，第 k+1 個 token 免費獲得。vLLM V1 支援三種 draft 方案：n-gram（從已生成序列中找匹配後綴）、EAGLE（保留 embeddings 和 LM head，用輕量 MLP 替代 transformer stack）、Medusa（在大模型頂層訓練多個輔助線性 head 平行預測未來 token）。不支援獨立 draft LLM 的方式。

**Disaggregated Prefill/Decode（P/D 分離）** 將 prefill 和 decode 分配到不同 GPU 實例，因為兩者的效能瓶頸完全不同（compute-bound vs memory-bandwidth-bound）。實作透過 KVTransferConfig 和 connector 抽象（如 SharedStorageConnector 用於本地檔案系統、LMCache 用 NVIDIA NIXL 做高速 KV 傳輸）。Prefill 實例計算完 KV 後上傳到 KV cache server；decode 實例從 server 載入 KV 後開始本地 decode。

### 第三部分：從 UniProcExecutor 到 MultiProcExecutor

當模型權重超過單 GPU VRAM 時，第一層擴展是 tensor parallelism（TP，同節點內跨 GPU 分片模型）。MultiProcExecutor 透過 multiprocessing spawn `world_size` 個 worker process，每個 worker 經歷與 UniProcExecutor 相同的 init device → load model → init KV cache 流程。Worker 之間透過共享記憶體的 `rpc_broadcast_mq` 接收工作、`worker_response_mq` 回傳結果。Driver worker（rank 0）負責協調。從引擎的角度看，`execute_model` 的呼叫介面完全沒變——所有 multiprocessing 複雜度被封裝在 executor 內部。

若 TP 仍不夠，下一步是 pipeline parallelism（PP，跨節點），再搭配 data parallelism（DP，複製整個模型到不同節點並做負載平衡）。

### 第四部分：分散式系統部署

作者以 2 台 H100 節點、TP=4、DP=4 的配置為例。Headless 節點上的 `CoreEngineProcManager` 啟動 2 個 DP 子 process（每節點 2 個，共 4 個 DP replica），每個 process 內有三條 thread：input thread（阻塞等待 API server 的請求，收到後放入 `input_queue`）、main thread（從 `input_queue` 取請求、餵入引擎、跑 MultiProcExecutor 的 forward pass、結果放進 `output_queue`）、output thread（從 `output_queue` 取結果送回 API server）。

DP coordinator 負責週期性傳送負載平衡資訊給 frontend、處理彈性 scaling、發送 DP wave 事件。所有 DP replica 必須同步執行 step——即使某個 replica 沒有請求也要跑 dummy step 以參與同步點。API server 節點執行 AsyncLLM（asyncio 包裝），內部使用 `DPLBAsyncMPClient` 進行負載平衡（score = `len(waiting) × 4 + len(running)`，選最低分的 engine），透過 FastAPI + Uvicorn 暴露 OpenAI 相容的 `/completion` 和 `/chat/completion` endpoint。

完整的請求生命週期：`curl` → FastAPI route → tokenize → AsyncLLM.generate → load balancing 選擇 engine → input socket → input thread → input_queue → main thread（反覆呼叫 `engine_core.step()`）→ output_queue → output thread → output socket → AsyncLLM 的 asyncio task → FastAPI → JSONResponse → 你的 terminal。

### 第五部分：效能基準與自動調優

作者定義了六個關鍵指標：TTFT（time to first token，請求提交到第一個輸出 token）、ITL（inter-token latency，連續兩個 token 之間的時間）、TPOT（所有輸出 token 的平均 ITL）、E2E Latency（TTFT + 所有 ITL 總和）、Throughput（每秒處理的總 token 數）、Goodput（滿足 SLO 的 throughput）。

**Latency 與 Throughput 的核心權衡** 透過 roofline model 解釋：batch size B 越小，ITL 越低（每個 token 不與其他 token 競爭），但 throughput 也低（weight I/O 無法被攤平）。隨著 B 增加，weight I/O 被更多 token 分攤，throughput 上升——直到達到飽和點 B_sat。超過 B_sat 後，kernel 從 memory-bandwidth-bound 轉為 compute-bound，step time 隨 B 線性增長，每增加一個 token 就增加 ITL——throughput 不再提升，但 latency 持續惡化。

vLLM 提供三種 benchmark CLI：`latency`（短輸入、固定輸出、小 batch，測 E2E latency）、`throughput`（大批量 prompt 一次性注入 QPS=Inf 模式，測尖峰吞吐量）、`serve`（啟動 server 並以 Poisson 分佈模擬真實請求到達模式，可設 max concurrency）。另有 auto-tune 腳本，自動尋找滿足目標 SLO（如 p99 E2E < 500ms）的參數配置。

### 結語與其他支援

作者提及本文略過但 vLLM 實際支援的功能：多種硬體後端（TPU、AWS Neuron）、MLA、MoE、encoder-decoder 模型、pooling/embedding 模型、EPLB、m-RoPE、LoRA、sliding window attention、multimodal LM、state-space models（Mamba/Mamba-2/Jamba）、以及更複雜的採樣方法如 beam sampling。這些功能大多與主流程正交，可視為「插件」處理。

## 城武觀點

Aleksa Gordić 這篇文章最讓我印象深刻的地方，不是他把 vLLM 拆得多乾淨——雖然確實拆得很乾淨——而是這篇文章的架構本身就是一個論證。他從一個不到 20 行的離線推論程式開始，一路疊加抽象層，最後收在一個跨兩台 H100 節點、四個 DP replica、每台跑 TP=4 的分散式部署，全程沒有換過對外介面。這個「介面不變，底層換光」的設計哲學，比任何單一技術都更能解釋 vLLM 為什麼贏。

這就帶出我想講的第一件事。

**PagedAttention 的創新不在演算法，在跨領域映射。**

你把 vLLM 的論文標題拿掉「LLM」三個字，只讀技術核心，會以為自己在讀 1960 年代的 OS 教科書。KV cache 的行為：動態增長、大小不等、釋放後留下碎片、多個請求需要共享同一段前綴——這跟虛擬記憶體分頁要解決的問題一模一樣。Atlas 在 1962 年就做了 demand paging。Knuth 在 1968 年的《The Art of Computer Programming》裡已經把 buddy system 講透了。vLLM 做的不是發明分頁，而是**認識到 GPU 上的 KV cache 和 CPU 上的虛擬記憶體是同一個抽象問題**。

這件事本身的教訓比 PagedAttention 更大。AI 基礎設施圈子這幾年花了太多力氣在「寫更好的 GPU kernel」——更好的 FlashAttention、更快的矩陣乘法、更低精度的量化。但 vLLM 最大的槓桿不在 kernel 層，在記憶體管理層——一個跟 GPU 架構完全無關、純粹是系統設計的問題。

說穿了，AI infra 最難的問題不是新數學，是穿了 GPU 外衣的老系統問題。運算的歷史，基本上就是重新發現 Knuth 早就知道的事情。你不需要更多的矩陣乘法優化專家，你需要更多記得自己讀過 OS 課本的人。

這不是貶低 vLLM 團隊的貢獻——剛好相反。跨領域映射需要的能力比「在同一領域內做得更好」更高。你要同時懂 GPU 的記憶體層級、LLM 的 attention 機制、和 OS 的虛擬記憶體設計，然後在三者之間發現同構（isomorphism）。vLLM 的成功不是因為 PagedAttention 是個突破性演算法——它是老演算法的新場景應用。它成功是因為在對的時間，有人把對的抽象層放到了對的系統位置上。

（如果你正在納悶「這篇講 vLLM 的文章，城武為什麼一直抬 Knuth 出來」——因為大部分 AI 工程師不讀 OS。他們的工具箱裡缺的不是 CUDA，是那幾本泛黃的系統設計教科書。從新學 CUDA 只要三個月，從新學「如何辨識兩個看起來完全不同的系統其實是同一個問題」要十年。）

**第二件事：B_sat 是每個 AI 產品團隊都該貼在牆上的數字，但沒有人貼。**

Aleksa 文章裡最重要的圖不是那個花俏的架構圖——而是 roofline model 中標出的飽和點 B_sat。過了這個點，更多 batching 只增加 latency，不增加 throughput。

這個事實的殘酷之處在於：**多數 AI 產品團隊的 throughput 優化，是在 B_sat 右邊進行的。**

為什麼？因為 throughput 數字是給 dashboard 看的，latency 數字是給使用者承受的。你可以在內部會議上秀出一張 throughput 成長 30% 的圖，說「我們這季的 infra 優化非常成功」。但你的使用者感受到的是 token 之間多出來的那 50 毫秒——一個一個疊起來，變成一整段對話的「卡卡的」。你增加了 throughput 但沒人能用到（因為已經過了 B_sat），你增加的 latency 每個使用者都感受到了。

這是結構性的激勵錯位。throughput 優化有明確的 ownership（infra 團隊）、可量化的 KPI（token/s）、可以被寫進 OKR 裡。latency 優化是「整體體驗」問題——它分散在使用者端、網路層、排程策略、甚至是前端 render 時間——沒有單一 owner，沒有乾淨的 dashboard 數字，沒辦法寫成「Q3 latency 降低 20%」這種漂亮的子彈。所以沒人做。所以大家都擠在 B_sat 右邊繼續「優化」。

這就是為什麼你在用某些 AI 產品時會覺得「明明說很快但用起來就是卡」——它們的 benchmark 是在 B_sat 附近測的，但它們的 production 跑在 B_sat 右邊。benchmark 報出的 50 token/s 是給你聽的，你在聊天介面裡等的那個 300ms ITL 是你自己承受的。

我在這裡賭一個具體的預測：未來兩年內，會有一家 AI infra 公司把「p99 E2E latency under SLO」而不是「peak throughput」當成 primary metric 來行銷。先做這件事的人會吃掉那些被現有產品卡到受不了的企業客戶。因為到那個時候，B2B buyer 已經被使用者的 latency 抱怨燒到夠痛了。

*城武的未解檔案——你的 GPU 利用率報表上那條漂亮的上升線，是你使用者在等 token 時心裡那條更漂亮的下降線換來的。*

- 原文：[Inside vLLM: Anatomy of a High-Throughput LLM Inference System](https://www.aleksagordic.com/blog/vllm)（Aleksa Gordić, 2025-08-29）
