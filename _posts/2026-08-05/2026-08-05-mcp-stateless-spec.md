---
layout: post
title: "【深度分析】MCP 砍掉重練：無狀態協議如何重新定義 AI 工具的未來"
date: 2026-08-05 02:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-05/mcp-stateless-spec.jpg)

一個開放協議的成熟，從來不是技術問題——是權力問題。MCP 把 stateful 砍掉的那一刻，選擇的不是「更好」，是「更大的玩家」。這篇兩份公告合起來四萬字的規格，值得從頭到尾讀清楚，再問一句：誰的伺服器變簡單了，誰的開發體驗變難了？

## 原文摘要

### 背景：從狀態到無狀態的典範轉移

自上次 2025 年 11 月的週年版本以來，MCP 經歷了驚人的增長——Tier 1 SDK 每月接近五億次下載，TypeScript 和 Python SDK 雙雙突破十億總下載量。今天，MCP 正式發布 `2026-07-28` 規格版本，核心變革只有一句話：**MCP 從雙向 stateful 協議轉變為請求/回應 stateless 協議。**

這是有史以來最大的規格修訂，完成了 2025 年 12 月〈The Future of MCP Transports〉中提出的藍圖，由六個 SEP（Specification Enhancement Proposal）共同實現。對生產部署的直接影響是即時的：過去需要 sticky session、共享 session store、在 gateway 上做 deep packet inspection 的遠端 MCP 伺服器，現在可以放在一個普通的 round-robin 負載均衡器後面，用 `Mcp-Method` header 路由流量，讓客戶端按照伺服器指定的 `ttlMs` 快取 `tools/list` 回應。

### 沒有 handshake，沒有 session

新規格正式移除了 `initialize`/`initialized` 握手交換，以及 `Mcp-Session-Id` header。每個請求現在獨立旅行，攜帶自己的協議版本、客戶端身份、客戶端能力（放在 `_meta` 欄位中）。如果客戶端想在做事之前先了解伺服器的能力，有一個新的 `server/discover` RPC 可以用，但不是強制的。任何請求現在可以落在任何伺服器實例上，不需要共享儲存。

對比舊版（`2025-11-25`）：呼叫工具前必須先建立 session，伺服器回傳 `Mcp-Session-Id`，所有後續請求都要帶上這個 ID，把客戶端鎖定在當初核發 session 的那個實例上。新版中同樣的工具呼叫是一個獨立的 self-contained 請求，任何伺服器實例都能處理。

### 協議無狀態，應用可以有狀態

移除協議層的 session 並不強迫應用層也變成 stateless。需要跨呼叫攜帶狀態的伺服器，可以用 HTTP API 幾十年來一直在用的方法：從工具中產生一個明確的 handle（如 `basket_id`、`browser_id`），讓模型把這個 handle 當作一般參數在後續呼叫中傳回來。MCP 團隊發現這個模式不只是 session state 的替代品——它往往更強大。模型可以跨工具組合 handles、對它們進行推理、在步驟之間傳遞，這是隱藏在傳輸 metadata 中的外部 session state 做不到的。協議不再為你管理狀態，但它不阻止你自己管理。明確 handle 模式只是讓狀態對模型可見，而不是藏起來。

### MRTR：多輪往返請求

MRTR（Multi Round-Trip Requests）取代了之前需要保持雙向串流開啟的伺服器端請求——`elicitation/create`、`sampling/createMessage`、`roots/list`。當一個工具在呼叫中途需要使用者輸入（例如確認或缺少參數），伺服器回傳 `resultType: "input_required"` 以及需要回答的請求，客戶端收集答案後用 `inputResponses` 重新發出原始呼叫，附上回傳的 `requestState`。任何伺服器實例都可以接住這個重試，因為所有必要資訊都在 payload 中。這解決了 stateless 協議最棘手的問題：伺服器如何在沒有持久連線的情況下向客戶端提問。

規格也將伺服器端發起請求的時機收緊：只能在伺服器正在處理一個客戶端請求時發起。舊版只是建議，新版是要求——使用者不會莫名其妙被彈出提示，每次 elicitation 都追溯得到使用者（或他們的 agent）啟動的某個動作。

### Header 路由

Streamable HTTP 傳輸現在強制要求 `Mcp-Method` 和 `Mcp-Name` headers。你的 gateway、rate limiter、WAF 可以直接在 headers 上路由和計量，不需要解析 JSON body。伺服器會拒絕 headers 和 body 不一致的請求。

### 可快取的列表結果

`tools/list`、`prompts/list`、`resources/list` 和 `resources/read` 的回應現在攜帶 `ttlMs` 和 `cacheScope`，模擬 HTTP `Cache-Control` 的語義。客戶端可以精確知道一個 `tools/list` 回應的有效期有多長，以及是否安全跨使用者共享。長期持有的 SSE 串流不再是得知列表變更的唯一方法。

### 分散式追蹤

W3C Trace Context 傳播在 `_meta` 中正式文件化，鎖定了 `traceparent`、`tracestate` 和 `baggage` 的 key 名稱，讓分散式追蹤可以跨 SDK 和 gateway 關聯。一個從 host 應用程式開始的追蹤，可以跟隨工具呼叫一路穿過客戶端 SDK、MCP 伺服器、以及伺服器下游的任何呼叫，在 OpenTelemetry 相容的後端中顯示為單一的 span tree。

### 授權強化

授權是 MCP 實作者花最多整合時間的領域。六個 SEP 強化了授權規範，使其更貼近 OAuth 2.0 和 OpenID Connect 的實務部署：

- **RFC 9207 issuer 驗證**：授權伺服器必須回傳 `iss` 參數，客戶端必須在兌換授權碼之前驗證它。這堵住了授權伺服器混淆攻擊的漏洞——在 MCP 的單一客戶端、多伺服器部署模式中特別常見。
- **`application_type` 宣告**：客戶端在動態客戶端註冊（DCR）時宣告其 OpenID Connect `application_type`，處理了桌面和 CLI 應用被授權伺服器預設為 `"web"` 而拒絕 localhost redirect URI 的常見問題。
- **憑證綁定**：客戶端憑證綁定到核發它們的授權伺服器 issuer，不能跨授權伺服器重複使用。當資源在授權伺服器之間遷移時，必須重新註冊。
- **DCR 正式被 CIMD 取代**：動態客戶端註冊（DCR）被正式標記為 deprecated，轉向客戶端 ID 中繼資料文件（CIMD）。DCR 向後相容但將在未來版本中移除。
- 其他改進包括：refresh token 請求的文件化、step-up 過程中的 scope 累積說明、`.well-known` discovery suffix 的澄清。

### Tasks 擴展

Tasks 從實驗性核心功能移出，進入 `io.modelcontextprotocol/tasks` 擴展。新的生命週期圍繞 stateless 模型重塑：伺服器可以用一個 task handle 回應 `tools/call`，客戶端用 `tasks/get`、`tasks/update`、`tasks/cancel` 來驅動它。Task 的建立是伺服器導向的——客戶端宣告有這個擴展，伺服器決定何時一個呼叫應該作為 task 執行。`tasks/list` 被移除了，因為沒有 session 的情況下無法安全地限定範圍。之前針對 `2025-11-25` 實驗性 Tasks API 開發的人都必須遷移到新生命週期。

### 擴展框架正式化

擴展在 `2025-11-25` 版本中就存在但沒有正式流程。新版加入：擴展以 reverse-DNS ID 識別、透過客戶端和伺服器能力中的 `extensions` map 協商、存放在各自的 `ext-*` 儲存庫中、由委派的維護者管理、獨立於規格版本。SEP 程序中新增了 Extensions Track，提供從實驗性到正式的演進路徑。此版本包含兩個正式擴展：MCP Apps（伺服器渲染的互動式 HTML 介面，在 sandboxed iframe 中渲染）和 Tasks。

### MCP Apps

MCP Apps 讓伺服器可以提供互動式 HTML 介面，host 在 sandboxed iframe 中渲染。工具預先宣告其 UI 模板，讓 host 可以在任何程式碼執行之前預取、快取和安全審查它們。渲染後的 UI 透過與 MCP 其他部分相同的 JSON-RPC 基礎協議與 host 溝通，因此每個 UI 發起的操作都經過與直接工具呼叫相同的審計和同意路徑。

### Roots、Sampling、Logging 被棄用

三個核心功能被標記為 deprecated：Roots（取代方案：工具參數、資源 URI、伺服器設定）、Sampling（取代方案：直接整合 LLM provider API）、Logging（取代方案：stdio 傳輸使用 stderr，結構化可觀測性使用 OpenTelemetry）。這些是純標記性棄用——方法、型別和能力標誌在這個版本以及一年內發布的所有規格版本中繼續運作。移除任何一個都需要在生命週期政策下提出單獨的 SEP。

舊的 HTTP+SSE 傳輸也被正式棄用，有一年的緩衝期。

### 完整 JSON Schema 2020-12

工具的 `inputSchema` 和 `outputSchema` 提升到完整的 JSON Schema 2020-12。輸入 schema 保持 `type: "object"` 的根約束，但現在允許組合（`oneOf`、`anyOf`、`allOf`）、條件和引用（`$ref`、`$defs`）。輸出 schema 沒有限制，`structuredContent` 現在可以是任何 JSON 值而不只是物件。實作不得自動解析外部 `$ref` URI，應限制 schema 深度和驗證時間。

另外，缺失資源的錯誤碼從 MCP 自訂的 `-32002` 改為 JSON-RPC 標準的 `-32602 Invalid Params`。

### SDK 生態

四個 Tier 1 SDK 在發布當天就支援 `2026-07-28`：TypeScript、Python、Go、C#。Rust SDK 也以 beta 狀態支援新規格。SDK 提供建構新規格伺服器和客戶端的 API，但會有遷移成本——尤其是依賴 session identifier 的開發者。團隊吸收了早期測試回饋，使遷移過程更為順暢。

### 生態系支援

超過十五家組織在公告中背書：

- **AWS**：透過 Amazon Bedrock AgentCore 支援 stateless 核心，Tasks 擴展由 AWS 貢獻
- **Google Cloud**：在開發者工具生態系中運用 stateless 架構
- **Microsoft Foundry**：從數十個整合擴展到數千個，透過 Foundry toolbox 統一 MCP endpoint 集中治理
- **Cloudflare**：Agents SDK 從 day zero 支援新規格，MCP 伺服器可直接在 Workers 上執行
- **Supabase**：MRTR 讓 stateless 的 Supabase MCP 終於可以支援 elicitation（如建立專案前的成本確認或刪除資料前的確認）
- **Figma**：stateless 架構支援其 MCP 伺服器的規模化，MCP Apps 讓設計和程式碼保持在同一個連結流程中
- **Honeycomb**：近 20% 的每月互動查詢現由 agent 發出
- **Manufact**：SDK v2 將套件大小縮減約 83%，速度提升 25%，得益於新的 client-server 分離
- **Netlify**：stateless 核心讓 MCP 在平台上成為一等 HTTP 工作負載
- **FastMCP / Prefect**：FastMCP 4.0 支援 background tasks、stateless interactivity、enterprise auth
- **Runlayer**：為平台上的企業提供簡化、更安全的 MCP 部署基礎
- **Stacklok**：stateless 模型移除操作複雜度，解鎖企業規模 MCP
- **Xero**：減少了需要管理的複雜度

### 協議演進機制與生命週期政策

此版本包含三個治理 SEP：功能生命週期政策（Active → Deprecated → Removed，deprecation 到最早可移除之間至少十二個月）、擴展框架（新功能作為 opt-in 擴展推出）、以及 Standards Track SEP 必須有對應的 conformance suite 場景才能達到 Final 狀態。MCP 團隊明確表示這次的 stateless 重構是需要 clean break 的基礎性變革——未來修訂不應該再要求實作者重寫傳輸或生命週期程式碼。

### 發布時程

發布候選版於 2026 年 5 月 21 日鎖定，最終規格於 2026 年 7 月 28 日發布。十週的窗口讓 SDK 維護者和客戶端實作者在真實工作負載上驗證變更。

## 城武觀點

MCP 這次砍掉 session，包裝得很漂亮——「吸收了數十年 web 協議設計的教訓」。但這不是技術選擇，這是選邊站。

看看背書名單：AWS Bedrock、Google Cloud、Microsoft Foundry、Cloudflare Workers、Netlify、Stacklok——全是跑幾千台伺服器的雲端基建大廠。你看到哪一個獨立工具開發者的聲音？一個都沒有。舊 MCP 對單一開發者友善：打開一個 session，開始寫工具，五分鐘搞定。新 MCP 是為付基礎設施帳單的人設計的。無狀態化在技術上是對的——負載均衡、水平擴展、快取策略，這些都是 HTTP 時代的基本功——但不要包裝成「對每個人都好」。贏家是雲端供應商，成本由需要遷移工具的開發者吸收。四個 Tier 1 SDK 全部 breaking change，Rust SDK 還在 beta，你手上那些基於 `Mcp-Session-Id` 寫的工具，接下來一年不是在開發新功能，是在追 compatibility。

十二個月的 deprecation window 聽起來大方。但魔鬼在細節裡：未來所有的功能設計、所有的 SEP 討論、所有的 SDK 優化，全部會以 stateless 為前提。Roots、Sampling、Logging——deprecated 但你「還可以用一年」——可是不會再有任何人為這些功能修 bug 或加新能力了。deprecation 的意思不是「繼續支援」，是「放著不管直到可以合法移除」。stateful 開發者不會被趕出去，但他們正在被結構性地留在一個沒有人繼續維護的舊世界裡。

最後一個沒有被說破的張力：MRTR 是一個精巧的工程解決方案——但它的精巧本身暴露了 stateless 模型的不自然。原本雙向串流中伺服器一句「使用者你確定嗎？」現在變成：回傳 `input_required` → 客戶端收集答案 → 從新發出請求 → 附上 `requestState` → 任何實例接手。每一步都合理，但加起來比原本多了一層序列化往返。這是 tradeoff，不是改進。在規模化的名義下，簡單的互動變得需要更多程式碼、更多狀態管理、更多失敗情境要處理。而這些複雜度，跟 session overhead 不一樣——它不會消失，只是從協議層搬到了應用層。

*城武的未解檔案——當一個協議說「我們為你移除了 session」，它真正的意思是「session 管理的複雜度現在是你的問題了」。*

- 原文：[The 2026-07-28 Specification](https://blog.modelcontextprotocol.io/posts/2026-07-28/)（MCP Team, Anthropic, 2026-07-28）
