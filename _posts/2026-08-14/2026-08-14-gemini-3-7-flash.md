---
layout: post
title: "【深度翻譯】Gemini 3.7 Flash：三週一更、價格砍半的掠奪性定價"
date: 2026-08-14 02:00:00 +0000
categories: [llm, ai, deep-translation]
---

![Gemini 3.7 Flash]({{ site.baseurl }}/assets/images/2026-08-14/gemini-3-7-flash.jpg)

Google 三週前才發 3.6 Flash，現在 3.7 Flash 性能大幅提升、價格直接砍半——而且只到年底。這以經不是技術進步帶來成本下降的故事了，這是經過精確計算的掠奪性定價。整篇文章最重要的句子被埋在倒數第二段，值得你慢慢讀。

## 原文摘要

Google 宣布推出 Gemini 3.7 Flash，定位為「目前為止最智慧的 workhorse 模型，專注於程式開發與 agent」。這距離 Gemini 3.6 Flash 發布僅三週，官方說法是開發者回饋與演算法創新的直接成果。3.7 Flash 在軟體工程、知識工作和網頁開發工作流上都有實質提升，並且以 3.6 Flash 原價一半的 introductory price 供應。

**更強的複雜工作流智慧**

在程式任務上，3.7 Flash 的 debug 和 issue resolution 表現明顯優於 3.6 Flash，首次生成程式碼的準確率也更高，能更好地產出 production-ready 的程式碼。在 FrontierCode 1.1 Main benchmark 上從 34.4% 提升到 43.6%，DeepSWE v1.1 從 49.0% 提升到 65.3%。

網頁開發方面，3.7 Flash 能用更少的 prompt 生成更完整的功能性版面和應用程式。UI 生成的設計忠實度很高，無論輸入是截圖、圖片或完整的 design system 都能對齊。在 Arena.ai 的 WebDev Arena 上，Elo score 從 1538 升到 1588。

在金融、法律、生物科學等知識密集領域，3.7 Flash 的推理和準確度都有改善。在處理複雜文件的 GDP.pdf benchmark 上，從 22.0% 大幅提升到 34.0%；在 AutomationBench（測試真實商業工作流完成能力）上從 17.0% 提升到 30.4%。

官方展示了四個 demo 案例：從文字 prompt 生成可玩的 3D 遊戲（搭配 Nano Banana 做動態角色、道具和材質的即時渲染）、一鍵生成互動式 landing page（orchestrating sub-agents，用 Gemini Omni 做視差元件）、用 multimodal understanding 在三 agent graph loop 中訓練機器人模型，以及把靜態 PDF（年報）轉成帶 live charts 的互動式網頁體驗。

**更好的開發者體驗與定價**

3.7 Flash 的開發者體驗明顯改善：遇到阻礙時能更好適應、需要時主動澄清意圖、遵循指令的忠實度更高。它在多步驟規劃和 tool calls 上投入更多心力（thinks more diligently），執行更有紀律意味著更少的 manual oversight 和 retries。

定價細節：3.7 Flash 到年底前以 introductory price 供應——input $0.75/1M tokens、output $3.75/1M tokens，是 3.6 Flash 原價的一半。2027 年 1 月 1 日之後恢復原價：input $1.50/1M tokens、output $7.50/1M tokens。

早期客戶回饋來自 12 家公司與機構：Box、Browser Use、Cartwheel、Databricks、emergent、Harvey、Hebbia、LangChain、Nunu.ai、Open Code、Pydantic、Stanford Department of Biology。

**用 3.7 Flash 改善 Gemini Spark**

Gemini Spark（提供給 Google AI Pro 和 Ultra 訂閱者，涵蓋 160 多個國家，但不包含 EEA、Nigeria、Switzerland、UK）即日起使用 3.7 Flash。Spark 在 I/O 大會上作為 24/7 運作的個人 AI agent 推出，在用戶指示下採取行動。這次模型更新讓 Spark 在知識工作上更有效率，對 Google Workspace 應用程式的 tool use 也有改善——能整合檔案、草擬郵件、更新狀態文件。

**安全考量**

Gemini 3.7 Flash 搭載了更新的 CBRN（化學、生物、放射性、核武）和網路攻擊濫用防護措施，同時保留有益的使用場景，符合 Google 的 bioresilience 和 cyber program 方針。

**今日即可使用**

開發者可透過 Google Antigravity、Google AI Studio 和 Android Studio 的 Gemini API 使用；企業可透過 Gemini Enterprise Agent Platform 和 Gemini Enterprise app；個人用戶可在支援的國家使用（同樣排除 EEA、Nigeria、Switzerland、UK）。

## 城武觀點

整篇最重要的句子是「introductory price of half the original 3.6 Flash cost per million tokens」——但它被埋在倒數第二段。三週前才發布 3.6 Flash，現在性能大幅提升、價格砍半，而且只到年底。2027 年 1 月 1 日之後價格翻倍。這不是「演算法創新帶來成本下降」，這是經典的掠奪性定價：先用低價把開發者鎖進生態系，等他們的生產環境都跑在 3.7 Flash 上了，再漲價。

注意 footnote 寫的是「introductory pricing expires on December 31, 2026」。漲價日期精確設定在新年第一天——剛好是企業做預算的時候，轉換成本最高的時候。這不是巧合，這是從新設計的鎖定策略。

那 12 個客戶 logo（Box、LangChain、Pydantic、Stanford⋯⋯）放在頁面裡當社交證明，但沒有任何一個客戶談到價格策略。這些 logo 的功能不是「證明產品好」，是「證明生態系已經成型」——當 LangChain 和 Pydantic 都整合了 3.7 Flash，你的 switching cost 就被鎖定了。

而 Gemini Spark 排除 EEA、Nigeria、Switzerland、UK 也值得追問。這不是技術限制，是監管規避——GDPR 地區不開放 personal AI agent 代你行事，因為法律責任歸屬太模糊。160 個國家能用，但法律責任最模糊的地方就不用。

「Google 一直做低價策略，這是正常競爭。」對，但正常的競爭不會把漲價日期精確設定在企業預算週期的起點。

*城武的未解檔案——你的生產環境跑在誰的 introductory price 上？*

- 原文：[Introducing Gemini 3.7 Flash](https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-3-7-flash/)（Tulsee Doshi, Google Blog, 2026-08-13）
