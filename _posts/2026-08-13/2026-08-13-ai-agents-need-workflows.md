---
layout: post
title: "【AI 深度拆解】AI agent 不缺模型，缺的是流程"
date: 2026-08-13 03:00:00 +0000
categories: [llm, ai, agents]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-13/ai-agents-workflows.jpg)

今天的 AI 新聞看起來像是一場模型軍備競賽：DeepSeek V4 Pro、Qwen3.8、Grok 4.6 接連出現，Google 替 Gemini API Managed Agents 加上新功能，Mistral 也推出 Agents API。但把這些消息放在一起看，真正正在改變的不是模型排行榜，而是模型被放進什麼樣的工作流程裡。

模型提供能力，流程決定能力能不能變成結果。沒有流程的 agent，只是一個會呼叫工具的聊天視窗；有流程的 agent，才可能接手一項需要多步驟、可驗證、可交接的工作。

## Agent 的問題不是「夠不夠聰明」

Anthropic 最近整理建構有效 AI agent 的方法時，把 agent system 分成兩種：預先安排好步驟的 workflow，以及讓模型自行決定流程的 agent。前者適合規則清楚、步驟穩定的任務；後者適合需要根據環境回應、持續選擇工具的任務。

這個區分很重要，因為業界常把「用了 agent」當成技術升級，但一個系統是否值得稱為 agent，不在於它能不能連接十個工具，而在於它是否知道何時使用工具、如何處理失敗、誰負責確認結果，以及工作何時算完成。

如果這些問題沒有被定義，系統只是在不同 API 之間傳遞一段段文字。模型可能會產生看似合理的下一步，但沒有人能回答：它為什麼做這個決定？做錯之後由誰發現？重跑會不會造成重複付款、錯誤部署或資料覆寫？

Anthropic 的官方說法是，好的 agent 不必然需要最複雜的架構。實際上，工具設計、清楚的任務邊界，以及適當的驗證機制，往往比再加一層 orchestrator 更直接。這不是反對 agent，而是把問題從「模型能不能做」移到「系統能不能可靠地讓它做」。

- 來源：[Anthropic](https://www.anthropic.com/engineering/building-effective-agents)

## Google 把 agent 做成 API 基礎設施

Google 為 Gemini API Managed Agents 加入 3.6 Flash、hooks 等功能，方向不是再發布一個孤立的聊天模型，而是提供管理 agent 執行過程的基礎設施。

hooks 的意義在於，開發者可以在 agent 執行前後介入：檢查輸入、記錄動作、修改上下文，或在特定條件下阻止下一步。這些機制看起來不像 benchmark 上的亮眼數字，卻是 agent 從展示品變成服務時必須面對的部分。

一個能夠自己呼叫工具的模型，和一個可以被監控、限制、審計的工作系統，中間差的不只是幾個 SDK。前者展示的是模型能力，後者處理的是責任分配。

當 agent 替企業讀取郵件、建立工單、修改資料庫或部署程式時，「它答對了幾題」已經不是主要問題。企業需要知道它碰過哪些資料、呼叫過哪些工具、在哪個環節失敗，以及失敗後能不能回復。Managed Agents 這類產品處理的，正是這些模型排行榜不會告訴你的事情。

- 來源：[Google](https://blog.google/innovation-and-ai/technology/developers-tools/expanding-managed-agents-gemini-api-3-6-flash-hooks/)

## Mistral 把 agent 接到工作的中間地帶

Mistral 同時推出 Agents API，並示範從會議內容產生開發工單的 agentic workflow。這類案例的重點不在於模型能不能摘要會議，而在於摘要之後發生什麼事。

會議內容要先被整理成決策，再被轉換成任務；任務需要負責人、優先級、期限與上下文；建立工單後，還要有人確認它沒有誤解會議內容。每一步都可能需要模型，但每一步也都需要規則、資料格式與人工檢查。

如果只展示「會議錄音變成工單」，看起來像是一個漂亮的 demo；若要放進真實團隊，系統還必須處理模糊決策、互相矛盾的發言、未定案事項，以及沒有人願意負責的任務。真正困難的不是生成第一版工單，而是讓工單在組織裡不會變成另一種噪音。

Mistral 的其他產品也朝同一方向排列：OCR 4 負責把文件變成可處理的資料，Agents API 負責執行，產品開發流程則把輸出接到團隊的工作系統。這種組合說明，agent 的價值不在單次回答，而在能不能穿過資料、決策與執行之間的縫隙。

- 來源：[Mistral Agents API](https://mistral.ai/news/agents-api/)
- 來源：[Mistral agentic workflow](https://mistral.ai/news/agentic-workflows-from-meetings-to-dev-tickets/)
- 來源：[Mistral OCR 4](https://mistral.ai/news/ocr-4/)

## 企業導入 AI，買的其實是流程重組

Zapier 分享使用 ChatGPT Work 改善行銷漏斗、製作活動素材與自動化報告。這類企業案例很容易被寫成「AI 幫公司提升效率」，但真正值得看的不是這句結論，而是 AI 被放在了哪些原本分散的工作之間。

素材產製、潛在客戶追蹤、數據整理與報告撰寫，本來就不是四個完全獨立的任務。它們之間有資料移交、審核、補件與決策。AI 若只負責其中一個環節，可能只是省下一點打字時間；若能把幾個環節串起來，才會改變工作的交接方式。

但流程整合也會放大錯誤。一個錯誤的分類可能被自動傳到下一個系統，一份沒有經過確認的摘要可能成為後續報告的資料來源。流程越順，錯誤越容易安靜地往下游移動。因此，企業導入 agent 的第一個問題不該是「能省幾個人」，而是「哪幾個節點必須保留人工判斷」。

- 來源：[OpenAI](https://openai.com/index/zapier)

## Agent 的核心產品其實是「可被追責的決策」

這幾則消息共同指向一個不太適合行銷簡報的結論：模型能力只是 agent 的原料，流程設計才是產品。

流程至少要回答五個問題：

1. 任務的輸入與完成條件是什麼？
2. 模型可以使用哪些工具，權限到哪裡為止？
3. 哪些決定可以自動執行，哪些必須人工批准？
4. 失敗、重試與部分完成要怎麼處理？
5. 每一步如何留下可供追查的紀錄？

這五個問題沒有一個能靠換更大的模型自動解決。更大的模型可能降低某些錯誤率，卻不會替組織決定誰有權限刪除資料，也不會替團隊定義一張錯誤工單要不要重開。

因此，下一階段的競爭不只會發生在模型公司之間，也會發生在誰能把模型嵌入可靠流程。Google 在做管理層，Mistral 在做工具與執行層，Anthropic 在整理設計原則，企業則在嘗試把這些元件接進現有的工作系統。真正的差距，可能出現在最不容易被展示的地方：權限、紀錄、例外與責任。

*城武的未解檔案——大家都在問 AI agent 能不能自己做事，卻很少有人先寫清楚：它做錯事之後，誰有權力把事情做回來。*

- 延伸閱讀：[Anthropic：Building Effective AI Agents](https://www.anthropic.com/engineering/building-effective-agents)
- 延伸閱讀：[Google：Expanding Managed Agents for Gemini API](https://blog.google/innovation-and-ai/technology/developers-tools/expanding-managed-agents-gemini-api-3-6-flash-hooks/)
- 延伸閱讀：[Mistral：Build AI agents with the Agents API](https://mistral.ai/news/agents-api/)
- 延伸閱讀：[OpenAI：How Zapier transformed core marketing processes with ChatGPT Work](https://openai.com/index/zapier)
