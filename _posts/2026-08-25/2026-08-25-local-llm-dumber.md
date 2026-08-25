---
layout: post
title: "【深度分析】你的本地 LLM 為什麼感覺比數字笨：能力是實作函數，不是模型函數"
date: 2026-08-25 03:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-25/local_llm_dumber.jpg)

你下載的那顆「屠榜模型」，跑起來總覺得哪裡怪怪的、比數字笨半截——那不是你的錯覺，是實作弄的。這篇實驗用 Qwen3.6-27B 一層層拆給你看：同一組權重，光換 attention backend 或量化方式，top-1 token 就會分歧到把工具呼叫整個搞砸。重點不是「你的機器爛」，而是「能力根本不是模型函數」——這篇值得讀，它把你從 benchmark 數字拉回現實。思微一下，你上次相信的那個跑分，是真實跑在你家機器上的嗎？

## 原文摘要

作者 thr3e 在 Level1Techs 論壇開了一系列技術實驗，想回答一個每個人都碰過的現象：大家在論壇、Reddit、Discord 聽到「OH！模型 XYZ 超猛的！zomgwtfbbq」，於是下載了它（或者更可能，下載了某個量化版本），跑起來卻只想說「呃…這超爛的」。他用「reference implementation」這個詞，來指「發表並提供自家代管的模型、貼出原始 benchmark 宣稱」的那個實驗室——他們的硬體跟你不一樣，軟體也跟你有很大不同。他特別強調，這篇的比較不是拿「ollama 裡那顆 2.58-bit GGUF + 幾個測試 prompt」來跑，而是更嚴謹的重播實驗。

他的第一個論點是：**你的本地實作很爛，但沒關係，因為大家的都一樣爛。** 今天每一套在跑 LLM 的硬體和軟體組合都有一點不同，有些情況甚至差很多。一般 homelab 使用者可能混搭著好幾個世代的 GPU，這些晶片上有不同的指令集，而這些指令集會用跟別人都不一樣的方式去實作和執行「算出下一個 token」的數學——就算跑的是完全相同的權重。

那要怎麼量「你的 setup 到底有多爛」？務實的做法很直白：跑標準 benchmark——terminal bench、hle、SWEthis、HELLAthat、MMLU 什麼的。但要確保它夠代表你實際的工作負載。不要為了看結果把 temperature 調到 0 然後貼三個測試 prompt 就說「可以/不行」——Zero-shot 測試對大多數 agentic 任務根本不是好的對照。你需要的是長上下文工具呼叫（long-context tool-calling）和領域特定知識評估，才能找出你的 setup 到底弱在哪。

**數學就是數學。**「Logits」是模型對每個可能的下一個 token 給的分數。它們被正規化成機率、通過你設定的 sampler、再由 detokenizer 轉回文字，在 decode 階段生成 THE→NE→XT→TOK→EN。

> 順帶一提 sampler 設定：HF 上的 model card 通常會明確指定你該用的 sampler 設定（和 chat template），例如 temp 1.0、top-p 0.95 之類。把 temp 設太低，就是你家的 qwen 會卡在 THINK 輸出裡繞不出去的元凶。

當下一個 token 的機率改變夠多，THE→NE→XT 就變成 THE→NE→W→DAY… 這些小改變可能沒什麼，但很可能就是你腦中那股「哪裡怪怪的」不舒服感覺的開端。

有些人可能聽過 KLD，也就是 KL Divergence（KL 散度）：把輸出的 logits 轉成機率分布，然後量這個分布從一個選定的基線移動了多遠。KLD 越低代表越接近那個基線——**不代表更聰明**。而且 KLD 是有方向性的，兩個分布的順序會影響結果。

> 一個警告：別被 HF 量化 model card 上那些低到不合理的 KLD 宣稱給騙了。除非作者揭露 reference checkpoints 和完整的執行環境、評估文字、校準資料、context 長度、取樣位置、KL 方向、任何詞彙截斷，以及這些量測是怎麼聚合的，否則那個數字根本無法解讀。方法論跟數字一樣重要。

**vllm 到底在幹嘛？** 在他那個過度簡化的示意圖裡，每一步都有元件可以根據你的硬體/軟體足跡、模型、量化、tensor 形狀等去設定或變更。他偷抓的 nightly VLLM container image 裡有 734 個套件（其中 252 個是 uv/pip 的 Python 套件）——也就是 734 份各自帶著 bug 和沒寫進文件的小怪癖的 codebase。你的特定實作在那座由程式碼堆成的山裡走過的路徑，會是獨一無二的。

**實驗一：Attention Backends 的精確度基準測試。** 先從推理流程圖裡的一塊著手：在 prefill（prompt 處理）階段，推理引擎會從幾個 attention backend 中選一個來用。這影響 prefill 的速度和精確度，而且每個 GPU 家族都需要不同的 CUDA kernel。

他用官方 BF16 checkpoint 的 Qwen3.6-27B，跑在 RTX PRO 6000 Blackwell GPU 上、tensor parallelism 1。KV cache 是 BF16，權重/activation 和 KV cache 都不做量化。軟體是 pinned 的 nightly vllm build，用 eager execution、關掉 CUDA graphs、prefix caching 和 MTP，用 2k-token 的 chunked prefill。

Qwen3.6-27B 是 dense 不是 MoE，但它是 hybrid 模型：64 層以「三個 Gated DeltaNet/linear-attention 層，接著一個 full-attention 層」的模式重複。只有那 16 個 full-attention 層會用到可選的 attention backend；Gated DeltaNet 那條路徑是固定的。

重播的工作負載叫「Prompt 2」，是一段約 100k token 的 context，來自一個真實的 Turnstone lab 工作串流，包含多次工具呼叫和真實的工作產物。選它是要模擬本地 agent 實際在幹的事，而不是合成式的「大海撈針」測試——而且這個 prompt 沒出現在任何現存的 benchmark 或訓練資料集裡。

三個可用的 full attention backend 是 FlashAttention 2、Flash Inference、Triton Attention，這是他兩次執行之間唯一改動的東西。他做了一個「同 backend、跨 GPU 的可重複性控制」，每 32 個 prompt token 就捕捉一次 full-vocabulary 的 BF16 logits，之後在 FP64 底下用存下來的 logits 計算 KLD 比較。Top-1 一致性，指的就是擁有最高 logit 的 token（greedy argmax）是否相同。三個 backend 都對著同樣的強制 token 歷史做評估，所以「top-1 flip」意味著在那個位置，某個 backend 會選出不同的 greedy next token。

結果：開頭幾千個 token，每個 run 對下一個 token 的想法都一致。越到後面 backend 開始分歧。分歧以群集的方式出現，隨 prompt 內容而變，而不是隨著 context 長度平滑增加。而同一 backend 在各個 run 之間、每個 hidden state 的 logits 都 bit-for-bit 完全相同——這代表分歧完全是來自不同 backend 在 prefill 階段的矩陣乘法/attention 操作。

**實驗二：KV Cache 量化，或「為什麼你的 LLM 在 40k token 之後智商像自由落體」。** 重複同樣的方法，用 BF16 權重加 BF16 KV cache 的 Triton 基線，只把 kv-cache 做量化。結果：分歧出現，而且是一個完全可重現的工具呼叫錯誤——夠多的 top-token 在工具呼叫期間被 flip 了。他們讓它跑完，發現 BF16 一切正常，int8 的 kv-cache 最終能恢復，**但 int4 恢復不了！**

**實驗三：權重量化大對決。** KV cache 維持全尺寸 BF16，比較五顆模型：
- BF16 reference：Qwen/Qwen3.6-27B
- Official FP8：Qwen/Qwen3.6-27B-FP8
- INT8 W8A16：TheHouseOfTheDude/Qwen3.6-27B-INT8
- NVIDIA NVFP4：nvidia/Qwen3.6-27B-NVFP4
- AWQ W4A16：cyankiwi/Qwen3.6-27B-AWQ-BF16-INT4

每一顆都跑不同的 CUDA kernel / GEMM / MMA 指令集。細節上：reference 是 BF16 權重/activation、UnquantizedLinearMethod、KV BF16；FP8 用 E4M3 FP8 權重（128x128 block）、dynamic FP8 activation quant、Fp8LinearMethod/CutlassFp8BlockScaledMMKernel、KV BF16，DeepGemm 被關掉（E8M0 被標記為對 SM120 有精確度損失）、改用 CUTLASS，沒找到校準資料集；INT8 W8A16 是 static symmetric channel-wise INT8 線性權重、BF16 activation、排除 GDN/lm_head、CompressedTensorsWNA16/MarlinLinearKernel、one-shot 量化且沒有校準資料集——它異常好的保真度可以用 W8A16 加未量化的 GDN projection 解釋；NVFP4 是混合 checkpoint——208 個 static FP8 W8A8 目標加 193 個 NVFP4 W4A16 目標，FP8 走 FlashInferFP8ScaledMMLinearKernel、NVFP4 走 MarlinNvFp4LinearKernel，這 run 不是 native FP4 算術，歸類為「GPU path 缺乏 native FP4」所以用 Marlin 做 weight-only FP4，內建的 FP8 KV 方案被覆寫成 BF16 KV；AWQ W4A16 是 static asymmetric INT4 權重（group size 32）、MSE observer、BF16 activation、排除 GDN/lm_head、CompressedTensorsWNA16/MarlinLinearKernel，校準資料揭露為「STEM and Agentic」。

結果：TheDude（W8A16）把所有人按在地上摩擦，贏過第一方 FP8（W8A8）和 NVIDIA 的 FP4 版本。五顆裡面，NVIDIA 那顆最慘——到 88k context 時約 50% 的 token flip。而且 NVFP4 和 AWQ W4A16 都無法正確關閉它們的工具呼叫，甚至搞砸了 Cisco 的 CLI 語法（正確的是 `show arp`，它們執行了 `show run`），而 FP8 和 INT8 都完成了正確的呼叫。

**Part 1 總結。** 更多實驗和觀察之後還會來，但它們需要大量的平行 GPU 時間，才能在巨大的 context 鏈上算出並記錄每個被取樣的 logit。

**討論串後續。** 這串接著由 wendell、vad、Burhan、mashie、ricardodevries、Esgariot 等人回覆，綿延約 8 天（一直到 8/24）。主題涵蓋量化指南、native FP4、硬體差異，以及 token divergence 對 agent 任務的實際影響。

## 城武觀點

能力是實作函數，不是模型函數——這是我唯一站的那邊。本地 LLM 社群老拿 MMLU 或一句「模型其實很強」來自慰，這篇實驗正好戳破：那是認識論上的幻覺。同一組權重，換 attention backend、量化 KV cache、量化權重，top-1 token 就分歧到把工具呼叫搞砸——NVIDIA 官方 NVFP4 在 88k context 就約 50% token flip，把對的 `show arp` 打成 `show run`；光 int4 KV cache 就讓工具呼叫無法恢復。所以「這模型多強」不是單一數字，是你硬體、量化、kernel 的一整條實作函數，reference implementation 的 benchmark 宣稱的是別人機器上那個模型，不是你家這個。強立場：停止追 benchmark，該測你自己的長上下文工具呼叫能不能通過——你能做的不是換更貴的卡，而是先確認自己最在意的任務撐不撐得住。KLD 沒揭露校準資料、context 長度、reference checkpoint 就只是行銷數字，以經太多人拿 0.00x 的 KLD 自我安慰——同一種幻覺。

*城武的未解檔案——下次看到「這模型屠榜」，先問一句：屠的是誰家那台機器？*

- 原文：[Why your local LLM feels dumber than it is](https://forum.level1techs.com/t/why-your-local-llm-feels-dumber-than-it-is/253917)（thr3e, Level1Techs 論壇, 2026-08-16）
