---
layout: post
title: "【深度分析】Gemini Managed Agents 大更新：hooks 給你鑰匙，但牢房還是 Google 的"
date: 2026-08-04 01:00:00 +0000
categories: [llm, ai, deep-analysis]
slug: gemini-managed-agents-hooks
---

![hero]({{ site.baseurl }}/assets/images/2026-08-04/gemini-managed-agents-hooks.jpg)

Google 這次的 Managed Agents 更新——Gemini 3.6 Flash 預設、environment hooks、budget controls、scheduled triggers、free tier——表面上是「給開發者更多控制權」。但如果你只看功能清單，你會錯過真正的故事：Google 正在把 agent infrastructure 變成一個你搬不走的平台。hooks 讓你可以在 sandbox 裡寫驗證腳本，但 sandbox 的牆一吋都沒動；free tier 讓你零成本入場，但一旦你把 hooks、triggers、environment 都綁在 Google 的 runtime 上，切換成本就不是換一組 API key 那麼簡單了。

## 原文摘要

Managed Agents 是 Gemini API 中的一項功能：透過 Gemini Interactions API 的單次呼叫，就能在隔離的雲端 sandbox 中協調 reasoning、code execution、套件安裝、檔案管理與網頁檢索。這次更新建立在先前推出的 background tasks 和 remote MCP server 整合之上，加入了 environment hooks、模型選擇、預算控制與排程觸發。

**Gemini 3.6 Flash 成為預設模型。** `antigravity-preview-05-2026` agent 現在預設運行 Gemini 3.6 Flash，開發者無需修改任何程式碼，下一次 interaction 就會自動採用。也可以透過建立 interaction 或 managed agent 時傳入 `agent_config.model` 來明確指定模型——用 Gemini 3.5 Flash-Lite 追求更低成本，或鎖定偏好模型。支援的模型包括：Gemini 3.6 Flash（預設，reasoning、coding、tool use 的平衡型）、Gemini 3.5 Flash（上代，通用 agentic workflow）、Gemini 3.5 Flash-Lite（Gemini 3.5 家族中最低延遲與成本）。

**Environment hooks：在 sandbox 內攔截、驗證與審計 tool call。** 這是最受關注的新功能。開發者可以在 agent 的 sandbox 環境中放置一個 `.agents/hooks.json` 設定檔，runtime 會在每個 tool call 的前後觸發你指定的 handler——pre_tool_execution（執行前）或 post_tool_execution（執行後）。`matcher` 欄位支援正則表達式，可以用 `|` 指定多個 tool（如 `code_execution|write_file`），或用 `*` 攔截所有 tool call。

官方給出的 JSON 範例定義了兩組 hooks：`security-gate` 在每次 `code_execution` 或 `write_file` 之前執行 `gate.py`，若腳本回傳 `{"decision": "deny", "reason": "..."}`，該 tool call 就會被跳過，拒絕原因會注入模型的 context；`auto-format` 則在每個 tool call 完成後執行 `auto_lint.py`，強制程式碼風格檢查。Hooks 也支援 `http` type handler，可以直接 POST 到外部端點。

團隊以經在生產環境中使用 hooks 建立驗證管線。AI-native 投資銀行 Offdeal 使用 `post_tool_execution` hooks 在 remote sandbox 內執行自動化圖片驗證。Offdeal 的 CTO Alston Lin 解釋：「在 agent hooks 之前，我們無法在 Gemini 的 managed agents 上做這件事——sandbox 是 remote 的，我們的驗證程式碼無處可跑。有了 hooks 之後，post_tool_execution hook 在 Archie（他們的 agent）寫好公司列表的瞬間就觸發我們的 pipeline，抓取候選圖片、執行 pixel-level 品質檢查、用 Gemini vision 驗證每張 logo，最後發布一份核准檔案清單——只有清單上的圖片才能進簡報。」

**成本控制與自動化。** 首先是 free tier：managed agents 現在可以在 free tier 專案上使用，開發者可以用沒有 active billing 的 API key 實驗 agentic workflow。其次是 budget controls：因為 managed agents 執行的是多輪自主迴圈，複雜任務可能消耗可觀的 token 預算，開發者可以在 `agent_config` 中傳入 `max_total_tokens`（input + output + thinking 的總量上限）。達到上限時，執行會安全暫停，回傳 `status: "incomplete"`，但 sandbox 環境狀態會被保留，可以透過 `previous_interaction_id` 用新預算從中斷處繼續。

排程觸發（scheduled triggers）讓開發者可以用 cron 表達式綁定 agent、environment、prompt 成為一個持久資源，無需手動介入自動觸發。每次執行重用同一個 sandbox，檔案跨執行持續存在。另外，Environments API 讓開發者可以用程式碼列出、檢查、刪除 sandbox session——斷線後恢復 environment ID，或 pipeline 完成時清理 sandbox 而不用等 7 天 TTL。

總結來說，這些更新把 managed agents 變成了受預算控制、可排程的 autonomous workers，在真實開發環境中自主運作，不破預算、不需外部 orchestration。

## 城武觀點

**Environment hooks 是籠子升級，不是拆籠子。**

Google 給了你 hooks，讓你「感覺」自己有控制權——你可以在 sandbox 裡跑驗證腳本、攔截 tool call、甚至拒絕 agent 的操作。看起來是「開發者賦能」，但沙盒的牆一吋都沒動。真正的主權是什麼？是可以在自己的基礎設施上跑 agent runtime，不是替 Google 的 sandbox 寫驗證腳本。

Offdeal 的案例是真的有用，他們的圖像驗證 pipeline 確實解決了一個實際問題。但仔細讀 CTO Alston Lin 那句話：「sandbox 是 remote 的，我們的驗證程式碼無處可跑」——這句話本身就是問題所在。為什麼你的驗證程式碼必須跑在 Google 的 sandbox 裡？為什麼不能跑在你自己的機器上，只把 agent 的輸出接過來？答案是你沒得選，因為 managed agents 的 runtime 是封閉的。

Google 的公關稿會說這是為了安全、為了讓開發者不用管理基礎設施。但我看到的是：他們給了你管理權（你可以設定規則），不是控制權（你不能換掉獄卒）。hooks 是籠子裡的跑步機——讓你忙起來，忘記自己還在籠子裡。這個差別，Google 的公關稿以經幫你模糊掉了。

**Managed Agents 是 Google 的 AWS Lambda 時刻。**

這件事比 hooks 更有意思。仔細看這次更新的全貌：排程觸發（scheduled triggers）、持久 sandbox（檔案跨執行保留）、token 預算控制（max_total_tokens 上限 + 中斷後續跑）、環境管理 API（Environments API）。Google 賣的不是模型 token，是 agent 基礎設施。模型是誘餌——free tier！Gemini 3.6 Flash 預設！——基礎設施才是產品。

AWS Lambda 當年的劇本一模一樣：先用 free tier 和「不用管伺服器」的敘事讓開發者把業務邏輯寫進 Lambda function，等你把所有東西都綁在 AWS 的 runtime 上之後，切換成本就是架構級的。Google 現在在做同一件事：一旦你把 agent workflow 建在 Gemini sandbox 上——hooks 寫給 Google runtime、triggers 掛在 Google scheduler、environment state 存在 Google 的 sandbox 裡——你就不是在用一個 API，你是在用一個平台。切換成本不是換 API key，是重寫整個 agent 架構。

Free tier 不是慈善。它是雲端史上最便宜的獲客成本——一個 free tier 專案就是一個潛在的付費客戶，而且這個客戶的 agent 基礎設施跟 Google 的 runtime 耦合越深，轉換成本就越高。Google 不是在跟 OpenAI 比模型品質，是在用基礎設施套裝取勝。這是正確的商業策略，但別把它包裝成「賦能開發者」。

*城武的未解檔案——Google 給了你 hooks，但你永遠拿不到 sandbox 的 root 密碼。*

- 原文：[Gemini API Managed Agents: 3.6 Flash, hooks, and more](https://blog.google/innovation-and-ai/technology/developers-tools/expanding-managed-agents-gemini-api-3-6-flash-hooks/)（Philipp Schmid & Mariano Cocirio, Google DeepMind, 2026-07-28）
