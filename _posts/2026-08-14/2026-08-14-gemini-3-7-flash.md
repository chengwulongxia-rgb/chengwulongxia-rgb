---
layout: post
title: "【深度翻譯】Gemini 3.7 Flash：三週一更、價格砍半的掠奪性定價"
date: 2026-08-14 02:00:00 +0000
categories: [llm, ai, deep-translation]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-14/gemini-3-7-flash.jpg)

一顆新模型在三週內性能大跳、價格砍半，聽起來是開發者的福音——但當你看到 Google 把漲價日期精確卡在 2027 年 1 月 1 日，事情就沒那麼單純了。這篇把 Gemini 3.7 Flash 的每個 benchmark 數字、定價細節、客戶名單都翻給你看，再告訴你一件事：你以為撿到便宜，其實以經踩進一張明年自動生效的漲價通知裡。

## 原文摘要

### 定位：迄今為止最聰明的「工作馬」模型

> "our most intelligent workhorse model yet for coding and agents."

Google 今天接續他們廣為使用的 Flash 系列，推出 Gemini 3.7 Flash，官方定位是「迄今為止在 coding 與 agent 領域最聰明的工作馬模型」。這次發布距離 Gemini 3.6 Flash 只有三週，官方說法是「直接來自開發者回饋與演算法創新」，並預告這些創新會帶進未來的模型。3.7 Flash 在軟體工程、知識工作、網頁開發工作流上都帶來大幅提升，而且以「introductory price」（上市優惠價）推出——每百萬 token 價格是 3.6 Flash 原價的一半。

### 複雜工作流：智力提升

在 coding 任務上，3.7 Flash 於除錯、issue 解決方面比 3.6 Flash 有明顯進步。它還達到更高的 first-pass 程式碼正確率，在生成 production-ready code 的表現也更好，具體數字見 FrontierCode 1.1 Main（43.6% vs 34.4%）與 DeepSWE v1.1（65.3% vs 49.0%）。

網頁開發方面，3.7 Flash 用更少的 prompt 就能生成更實用的版面配置、功能更完整的 app。UI 生成上，無論參考輸入是截圖、圖片還是完整的設計系統，模型都展現高度的設計遵循度與一致性。它在 Arena.ai 的 WebDev Arena 上以 Elo 1588 分勝過 3.6 Flash 的 1538 分。

在金融、法律、生醫等知識密集領域，3.7 Flash 帶來更好的推理與準確率。它在 GDP.pdf benchmark（一個測試模型處理複雜文件能力的評估）上顯著贏過 3.6 Flash：34.0% vs 22.0%。在 AutomationBench 上同樣勝出（30.4% vs 17.0%），代表它能更有效地完成真實世界的商業工作流。

Demo 案例：

- 一段簡單的文字 prompt，變成一個可以完整遊玩的 3D 遊戲（Gemini 3.7 Flash + Nano Banana，即時生成動態角色、物品、材質）
- 一擊生成令人驚豔的互動式 landing page（Gemini 3.7 Flash 負責編排 sub-agents，用 Gemini Omni 處理視差元件）
- 機器人模型用 Gemini 3.7 Flash 訓練，在 3-agent graph loop 中運用多模態理解
- 從靜態 PDF 到互動式數據故事（年報 → 帶即時圖表的網頁體驗）

### 更好的開發者體驗與價格

3.7 Flash 的開發者體驗比 3.6 Flash 有明顯改善。它更懂得適應障礙、必要時澄清意圖、更忠實地遵循指令。它「思考得更勤快」，在多步驟規劃與 tool calls 上投入更多。更自律的執行，意味著工程工作流中更少的人工監督、更少的重試。

定價：3.7 Flash 在年底前以 introductory price 提供——每百萬 input token $0.75、每百萬 output token $3.75，是 3.6 Flash 原價的一半。2027 年 1 月 1 日之後：每百萬 input token $1.50、每百萬 output token $7.50。

早期客戶回饋：Box、Browser Use、Cartwheel、Databricks、emergent、Harvey、Hebbia、LangChain、Nunu.ai、Open Code、Pydantic、Stanford Department of Biology。

### 用 3.7 Flash 升級 Gemini Spark

Gemini Spark（開放給 160 多國的 Google AI Pro 與 Ultra 訂閱戶，但排除 EEA、Nigeria、Switzerland、UK）從今天起改用 Gemini 3.7 Flash。Spark 是在 I/O 上發布的 personal AI agent，7×24 運行，在使用者指示下代為行動。這次模型更新讓 Spark 在知識工作上更有效率，Google Workspace app 的工具使用也獲得改善。有了 3.7 Flash，Gemini Spark 能更有效率地把想法化為行動：整合檔案、起草 email、更新狀態文件。

### 以安全為前提打造

Gemini 3.7 Flash 上線時附帶更新的防護措施，針對 CBRN（化學、生物、放射性、核）與網路攻擊的濫用風險，同時保留有益的使用案例，符合其 bioresilience 與 cyber 計畫的方針。

### 今天就試用

- 開發者：Google Antigravity、透過 Google AI Studio 與 Android Studio 使用 Gemini API
- 企業：Gemini Enterprise Agent Platform 與 Gemini Enterprise app
- 個人：受支援的國家（排除 EEA、Nigeria、Switzerland、UK）

## 城武觀點

先把話講清楚：我不是批評 3.7 Flash 的性能。數字是真的——FrontierCode 從 34.4% 跳到 43.6%、DeepSWE 從 49.0% 到 65.3%、WebDev Arena Elo 1588 對 1538、GDP.pdf 從 22.0% 到 34.0%、AutomationBench 從 17.0% 到 30.4%。三週內有這種進步，只有兩種解釋：要嘛 3.6 Flash 根本是半成品被急著推出來，要嘛 3.7 Flash 早就做好了壓在倉庫裡等 3.6 賣夠本。兩個解釋都對 Google 不利，但這也不是我要談的。

我要談的是「introductory price」這個詞。它聽起來很誠實——「我們虧本賣，快來」。但它真正在說的訊息是：「三週後價格翻倍，你最好現在就鎖定。」請看這個時間表：3.6 Flash 三週前才發布，3.7 Flash 今天價格砍半，然後漲價日期精確設在 2027 年 1 月 1 日。這不是隨手挑的日子。1 月 1 日是企業預算週期的起點——你剛用新的年度預算把 production pipeline 從新調到 3.7 Flash 上，切換成本正處在最高點，然後價格翻倍。這叫掠奪性定價（predatory pricing），不是「演算法創新帶來成本下降」。創新是真的，但創新被拿來做什麼，是另一回事。

當各家模型在技術上拉不開差距時，價格就是唯一的武器。OpenAI、Anthropic、Google 的旗艦模型在 benchmark 上咬得死緊，消費者分不出那 0.5% 的差距，但一定分得出 $0.75 和 $1.50 的差距。Google 選的武器很明確：先用低價買下你的生產依賴，再在你跑不掉的時候漲價。那 12 個客戶 logo——Box、LangChain、Pydantic、Stanford Department of Biology——不是「產品好」的證明，是「生態系已成型的切換成本」的展示。每一個 logo 都是 Google 在跟你說同一句話：「你的同行、你的框架、你的學術機構都已經在上面了，你換不掉的。」

還有 Gemini Spark 的國家列表。開放 160 多國，獨獨排除 EEA、Nigeria、Switzerland、UK。這不是技術限制，是監管規避。GDPR 地區不開放「personal AI agent 代你行事」，因為一個 7×24 幫你整合檔案、起草 email、更新狀態文件的 agent，一旦出事，法律責任歸誰？Google 不想回答這個問題，所以乾脆不進那些市場。把 Nigeria 一起排進來更是露餡——它的資料保護法規近年在收緊，Google 寧可放棄整個國家，也不願碰「agent 替你做決定，出事了誰負責」這個問題。一個號稱「把想法化為行動」的產品，最害怕的居然是「行動」兩個字。

反對者會說：價格競爭是好事啊，消費者受益，性能確實提升了，你有什麼好罵的？我的回應是：真正的成本不是當下的 token 價格，是重構 production pipeline 之後的切換成本。你為了省一半的錢，把 prompt、工具鏈、evaluation 全部調到 3.7 Flash 上，等 2027 年 1 月 1 日價格翻倍，你回頭算一算，重新調回其他模型的成本，比當初省下的錢還多。Google 賣給你的不是便宜的 token，是一份明年 1 月 1 日自動生效的漲價通知。

（我得承認：我自己就是這套定價的目標客群。我對模型的態度向來是「哪個便宜買哪個」，看到 $0.75/1M 的時候也確實心動了一下。這正是掠奪性定價可怕的地方——它對知道自己在被掠奪的人也有效。）

*城武的未解檔案——「introductory price」最誠實的中文翻譯，是「上鉤價」。*

- 原文：[Introducing Gemini 3.7 Flash](https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-3-7-flash/)（Tulsee Doshi, Google Blog, 2026-08-13）
