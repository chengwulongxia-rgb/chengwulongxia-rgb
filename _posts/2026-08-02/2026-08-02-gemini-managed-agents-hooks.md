---
layout: post
title: "【深度分析】Google 的 Managed Agents：便利性的糖衣，還是平台鎖定的絲絨繩？"
date: 2026-08-02 01:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-02/gemini-managed-agents-hooks.jpg)

Google 最近為 Gemini API 的 Managed Agents 推出了一波更新：預設模型換上 Gemini 3.6 Flash、新增 environment hooks 讓你在沙箱內攔截或審計 agent 的工具調用、加上預算上限與排程觸發、開放免費層存取。聽起來是開發者友善的功能迭代。但如果你仔細看這些「便利性」背後的設計——每一個都在把開發者的決策權，一步一步移進 Google 的沙箱裡。

## 原文摘要

Gemini API 的 Managed Agents 迎來一波重要更新：environment hooks、模型選擇、免費層存取，以及預算控制和排程觸發。這些功能建立在先前的基礎之上——背景任務與遠端 MCP 伺服器整合。透過 Gemini Interactions API 的 managed agents，一次 API 呼叫就能在隔離的雲端沙箱內協調推理、程式碼執行、套件安裝、檔案管理與網頁檢索。

### Gemini 3.6 Flash 成為預設模型

`antigravity-preview-05-2026` agent 現在預設執行 Gemini 3.6 Flash，不需要任何程式碼改動，下一次互動自動生效。開發者也可以透過 `agent_config.model` 顯式指定模型：想省成本就選 Gemini 3.5 Flash-Lite，想固定特定版本就 pin 住偏好模型。

支援的模型包括：
- **Gemini 3.6 Flash**（`gemini-3.6-flash`，預設）：推理、程式撰寫與工具使用的平衡選擇。
- **Gemini 3.5 Flash**（`gemini-3.5-flash`）：前一代的通用 agentic 工作流模型。
- **Gemini 3.5 Flash-Lite**（`gemini-3.5-flash-lite`）：Gemini 3.5 系列中最低延遲與最低成本。

### Environment hooks：在沙箱內攔截、檢查、審計工具調用

Environment hooks 讓開發者可以在 agent 每次執行工具調用的前後，執行自訂腳本。只需在 environment 中加入 `.agents/hooks.json`，runtime 就會在 `pre_tool_execution` 或 `post_tool_execution` 事件觸發時執行你的 handler。

`matcher` 欄位支援正則表達式，可以用 `|` 針對多個工具，或用 `*` 捕捉所有工具調用。

範例配置包含兩個 hook 群組：
- **security-gate**：在每次 `code_execution` 或 `write_file` 之前執行 `gate.py`。如果腳本回傳 `{"decision": "deny", "reason": "..."}`，工具調用就會被跳過，拒絕原因會傳入模型的 context。
- **auto-format**：在每個工具執行完後執行 `auto_lint.py`，強制執行程式碼風格。

Hooks 也支援 `http` 類型 handler，可以直接 POST 到外部端點。

### 實戰案例：OffDeal

AI 原生投資銀行 OffDeal 使用 `post_tool_execution` hooks 在遠端沙箱內執行自動化圖片驗證。他們的 AI 分析師「Archie」生成的投資銀行簡報需要每份至少 30 個公司 logo，每個 logo 必須對應正確的公司、尺寸與比例適當、透明背景、在白色投影片上高對比。

在 agent hooks 出現之前，驗證程式碼根本無處可在遠端沙箱中執行。有了 hooks 之後，一個 `post_tool_execution` hook 在 Archie 寫出公司清單的瞬間就觸發驗證管線——抓取候選圖、執行像素級品質檢查、用 Gemini vision 驗證每個 logo、產出已核准檔案的 manifest。

OffDeal 創辦人暨 CTO Alston Lin 對此表示高度肯定。

### 成本控制與自動化功能

**免費層存取**：Managed agents 現在開放免費層專案使用，開發者可以用未啟用帳單的專案 API key 實驗 agentic 工作流。

**預算控制**：因為 managed agents 執行多輪自主迴圈，複雜任務可能消耗大量 token。為防止失控，可以在 `agent_config` 中傳入 `max_total_tokens` 來設定總消耗上限（輸入 + 輸出 + thinking）。當 agent 到達上限時，執行會安全暫停，互動回傳 `status: "incomplete"`。環境狀態會被保留，開發者可以傳入 `previous_interaction_id` 並給予新的預算來接續執行。

**排程觸發**：透過 scheduled triggers 自動化重複性的 agent 任務。一個 trigger 會將 agent、environment、prompt 和 cron 排程綁定成一個持久資源，不需手動介入就會自動觸發。每次執行重複使用同一個沙箱，所以檔案會在多次執行之間保留。

**Environments API**：Environments API 讓開發者可以從程式碼中列出、檢查和刪除沙箱 session。可以在斷線後恢復 environment ID，或在工作流完成後清理沙箱，不必等待 7 天的 TTL。

## 城武觀點

### Hooks 是 Google 鋪好的絲絨繩——走上去很舒服，繩子的另一端綁在 Google 的沙箱上

Environment hooks 解決的是真問題。OffDeal 的痛點確實存在——在遠端沙箱內，你根本沒有地方跑自己的驗證邏輯。但「解決真問題」和「鎖定平台」從來不互斥：最有效的鎖定，就是先解決你真實需要的問題，讓你心甘情願走進來，走出去才發現代價太高。

每一個你寫進 `.agents/hooks.json` 的腳本——`gate.py`、`auto_lint.py`——都是**只能在 Google 沙箱內執行的技術債**。它們依賴 Google 的 runtime、hook 事件格式、沙箱檔案系統。今天你覺得方便，六個月後想把 workflow 搬到 AWS 或自建 infra 時，那些 hooks 全部變成廢鐵。這不是無心之過：從 Gmail 到 GCP，Google 每一層的設計都是入口便宜、出口昂貴。Environment hooks 是同一套劇本的 AI 版本。

最誠實的訊號藏在最不起眼的地方：hooks 的 `http` handler 只能 POST 到外部端點，不能讓你把 hook 邏輯跑在自己的伺服器上。如果 hook 可以在任何地方執行，它就是開放標準，不是平台鎖定。Google 顯然不想讓它變成開放標準。

### 「Managed」這個詞，是主權轉移的語言糖衣

「託管代理人」承諾幫你處理複雜度，同時靜悄悄地把決策權從你手上移交給 Google。拆開這次更新的四個營運槓桿：

- **模型預設**：3.6 Flash 是預設值。你可以改，但預設值就是政策——大多數開發者不會改，生態系圍繞預設值校準。
- **沙箱生命週期**：7 天 TTL，Google 決定，你不能延長。狀態蒸發後，除非依賴 scheduled triggers，否則無法維持。
- **預算上限**：`max_total_tokens` 把成本失控的風險從 Google 轉移給你——煞車踏板給你了，踩不踩是你的事。到達上限回傳 `incomplete`，要繼續？再付一次。
- **排程觸發**：cron job 綁在 Google 的 trigger 資源上，沙箱重複使用、檔案跨執行保留——便利就是依賴的累積。

每一個都讓開發者生活更輕鬆，每一個也都讓「離開 Google」更加不可想像。這不是 API 演化，這是代理開發生態中「誰說了算」的重新定義。

有人會說：「預設值可以改啊。」這是典型幻覺：你可以選 3.6 Flash、3.5 Flash、或 3.5 Flash-Lite——但你不能選 Claude、不能選 Llama。你可以設預算，但計價單位是 Google 的 token。框架內的選擇不是真正的選擇。你只是被溫柔地鎖住了。

*城武的未解檔案——當「託管」變成「你管不著」，便利性就是最精緻的手銬。Google 沒有騙你，它只是確保你永遠不需要問「如果我想離開呢」。*

- 原文：[Gemini API Managed Agents: 3.6 Flash, hooks, and more](https://blog.google/innovation-and-ai/technology/developers-tools/expanding-managed-agents-gemini-api-3-6-flash-hooks/)（Philipp Schmid & Mariano Cocirio, Google DeepMind, 2026-07-28）
