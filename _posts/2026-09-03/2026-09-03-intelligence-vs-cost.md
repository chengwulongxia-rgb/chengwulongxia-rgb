---
layout: post
title: "【深度分析】LLMs: Intelligence vs. cost"
date: 2026-09-03 01:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-09-03/intelligence-vs-cost.jpg)

這篇文章想問的是：當所有人都在比誰的模型更聰明時，有沒有人認真算過，這份聰明是不是從新被定價過？Guido Imperiale 把 ArtificialAnalysis 那張經典的「智慧對成本」圖拆開重畫，結果發現軸線、報價來源與本地硬體成本三個環節都在讓結論往同一個方向傾斜。

## 原文摘要

[ArtificialAnalysis](https://artificialanalysis.ai/) 是一個針對各種 LLM 進行智慧程度評測的網站。他們會發布一個標題式的 **Intelligence Index**，計算方式是取他們對每個模型跑過的一組精選 benchmark 平均分數。這是一個大致可用來衡量模型整體聰明程度的粗略指標。

AA 也會記錄一些實用資訊，也就是他們跑 benchmark 花了多少錢。由於所有模型跑的都是同一套 benchmark，這也間接提供了各模型相對使用成本的好指標。

他們的主要圖表之一是 [Intelligence vs. cost plot](https://artificialanalysis.ai/#intelligence-comparison-tabs)，呈現的是 Pareto frontier，也就是「達到某個智慧分數所需的最便宜模型」。這條 frontier 很重要，因為如果用一個超級聰明又超級昂貴的模型去做那些其實可以由更笨、更便宜模型完成的瑣事，純粹是浪費錢。

不過，隨著時間過去，作者對這張圖越來越惱火，原因有幾個。

### Why AA’s plot is misleading

第一個問題是成本軸使用對數尺度。使用 log scale 是唯一能在同一張圖裡同時顯示 $0.015 與 $0.032 的微小差距，以及 $3.69 這種幾乎貴了 250 倍的模型。但結果是，觀眾再也無法直觀感受便宜模型與重型模型之間巨大的價差，也無法意識到便宜模型彼此之間的價差其實微不足道。

第二個讓人不舒服的地方是它採用模型開發商自家 API 的官方定價。在多數情況下這沒問題，但對開放權重模型來說，同樣的模型透過第三方 API 供應商租用往往可以便宜很多。[OpenRouter](https://openrouter.ai/) 讓使用者可以輕鬆即時切換供應商，永遠拿到最低報價。

第三個也是最後一個問題是，本地模型——也就是能在消費級硬體上跑的模型——在圖中卻以資料中心定價出現。這種價格相對於你買到的智慧程度總是非常昂貴，而且終究不是任何真實使用者會主動想買的東西。

### I made my own plots

所有 Intelligence Index 分數都來自 ArtificialAnalysis。除非另有說明，否則所有資料點都是在最大思考 effort 下測得。所有成本分數也來自 ArtificialAnalysis，除非下方另有標註。

第一張圖呈現的是截至 2026 年 9 月 1 日市面上最聰明（也最貴）的模型。

閱讀智慧軸的一個經驗法則是：1 分的差距多數人大概感覺不出來，5 分的差距則相當明顯。必須強調的是，第一張圖中最低的 50 分，大致相當於 2026 年 2 月全世界最聰明的模型 Opus 4.6 所能達到的水準。

左下角的綠色區域是模型變得「極端便宜」的地方。我們把它放大，並把智慧軸往下延伸到可以在智慧型手機上執行的等級。

某些模型標有 ⚡ 符號，代表成本是依照本地跑模型的電費來計算（計算細節見下文），因為這些模型小到從資料中心提供根本沒有意義。在比較本地模型彼此時，這也提供了一個衡量各模型完成任務所需時間的尺度。

最後，把兩張圖合併在一起，以便更清楚呈現效能/成本上的邊際遞減。同樣地，所有圖表共通的區域以綠色標示：

### All the differences between AA’s plot and mine

- 把 x 軸從對數尺度改為線性尺度，因為人們的錢不是對數的。
- 把 Kimi K3、Qwen3.8 Max、DeepSeek V4 Flash 0731、GLM-5.3、GLM-5.3-Flash 與 Hy3 改成 OpenRouter 上可取得的價格（過慢或不可靠的供應商，以及沒有 Zero Data Retention 政策的供應商被排除）。
- 透過把 AA 在最大 effort 的分數與 [Z.ai 不同 effort 等級的 coding 分數](https://z.ai/blog/glm-5.3-flash)交叉對照，外插推估 GLM-5.3-Flash 在高 reasoning effort 的表現。
- 把參數量低於 350 億的模型從資料中心定價改為本地執行成本（詳見下文）。
- 新增 Ornith-1.5-35B-A3B。智慧分數是根據模型作者「自我宣稱」的 benchmark 結果外插推估，應該抱持合理懷疑。

### Cost calculation for local models

標有 ⚡ 的模型，每個任務成本是這樣粗略計算的：

- 從 artificialanalysis.ai 取得每個任務的輸出 token 數。
- 在本地硬體上粗略觀察解碼速度（tok/s）。多數測量是在同一張 2020 年的 RTX 3090 顯示卡上進行，這張卡現在二手價約 $1,400，已經相對便宜。
- 測量該硬體在峰值與閒置狀態下的功耗差。
- 電價採用 $0.2049/kWh，這是 2026 年 5 月以美國人口加權的平均住宅電價。
- 額外加上 15%（粗略估計）來涵蓋未快取的輸入 token 與等待工具呼叫的時間。
- 硬體成本設為零，理由是無論是 RTX 3090 電腦還是 64GB 的 Strix Halo，本身都是值得擁有的遊戲/工作機器。

請注意，不同硬體平台之間的電費並沒有顯著差異：Strix Halo 的功耗比 RTX 3090 低，但速度較慢，所以完成同樣任務需要跑更久。

### Larger local models

一旦你升級到超過 64 GB RAM，不計入硬體成本就開始說不過去了，因為幾乎沒有人會為了 AI 以外的用途需要這麼多記憶體。

Qwen3.8-Flash 至少需要 128GB 的 Strix Halo；圖中同時顯示了它的資料中心定價以及本地執行的電費成本；然而後者已經隱藏了一筆可觀的硬體支出：64GB 的 Strix Halo 是一台很不錯的通用迷你 PC，售價 $2,000；128GB 的版本售價 $3,600，而且除了跑約 120B 參數等級的 AI 模型之外，並不能多做什麼。

以下模型「可以」本地執行，但伴隨著非常高的前期硬體成本：

| Memory | Hardware | Models |
| --- | --- | --- |
| 128 GB RAM | Strix Halo ($3,600)<br>DGX Spark ($4,300)<br>Mac Studio M5 Max ($5,100)<br>MacBook Pro M5 Max ($7,150) | Qwen3.8-Flash<br>GLM-5.3-Flash (degraded intelligence)<br>DeepSeek-V4-Flash (degraded intelligence) |
| 256 GB RAM | 2x DGX Spark ($8,700)<br>Mac Studio M5 Ultra ($11,300) | GLM-5.3-Flash<br>DeepSeek-V4-Flash |
| 512 GB RAM | 2x Mac Studio M5 Ultra ($22,600) | GLM-5.3 |
| 2 TB RAM | 2x Tenstorrent Galaxy Blackhole ($320,000) | Kimi K3 |

### Conclusion

Anthropic 與 OpenAI 的尖端模型與便宜得多的中國模型之間存在巨大價差：前者對大型企業來說都[貴到離譜](https://www.tomshardware.com/tech-industry/artificial-intelligence/ai-cost-crisis-hits-tech-giants-as-employee-tokenmaxxing-backfires-agentic-ai-eats-up-to-1000x-more-tokens-than-standard-ai-sparks-corporate-pullback-at-microsoft-meta-and-amazon)，後者則可以便宜到像手機月租費。

砸錢能換到多少額外智慧，服從邊際遞減法則：頂尖工程師或科學家或許能夠體會 Fable 5.1（智慧分數 66，每個任務 $3.69）比起 GLM-5.3（智慧 60，$0.49，便宜 7.5 倍）好多少，但大多數人很難感受。再往下，GLM-5.3-Flash 在高設定下（智慧 55，$0.023，比 Fable 便宜 160 倍）在面對極複雜任務——例如一次獨立完成整個 coding 專案——時明顯較弱，但對 90% 的人實際需求來說已經「夠用」。即使前面提到的那些高度專業工程師與科學家，他們大部分工作也不需要那麼多額外智慧。再往下一點，熱衷遊戲的玩家已經可以在自己現有的電腦上跑 Qwen3.8-27B（智慧 52，電費 $0.015）。

## 城武觀點

我的立場很簡單：這張圖不是中性的技術視覺化，而是一套選擇性的定價敘事。ArtificialAnalysis 同時掌握「智慧指數」怎麼算、以及「成本」怎麼標，兩個環節疊在一起，剛好讓 frontier 模型看起來又貴又有道理。

第一，開放權重模型的定價被系統性高估。AA 對它們採用開發商官方 API 價，但 OpenRouter 上往往可以便宜很多；本地模型更被直接標成資料中心價，完全失真。這不只是資料來源問題，而是當一個平台同時定義「聰明」和「價格」時，它其實已經在決定哪些模型值得被放進讀者的比較視野裡。

第二，log 尺度不是為了「讓圖能塞得下」，而是讓讀者無法直覺感受 frontier 模型的溢價。$3.69 與 $0.023 在對數軸上被壓成幾公分，但這兩個數字在真實帳單上是 160 倍的差距。當價差被視覺稀釋，「為什麼還要買那麼貴」這個問題就從畫面上消失了。

第三，「你可以自己跑」把資本支出轉嫁給使用者。原文把 RTX 3090 或 64GB Strix Halo 當成「本來就會買的遊戲/工作機」，所以硬體成本設為零；但只要推到 128GB 以上，這個假設就崩解。一台 128GB Strix Halo 要價 $3,600，而且幾乎只能拿來跑模型——這筆錢被藏在「電費 $0.015」的敘事後面。

最後，效率前緣不是自然定律，是被選擇性展示的故事。結論說 90% 的人不需要那麼多智慧，這聽起來務實，卻強化了「大廠模型是奢侈品」的危機框架。我認為真正的危機不是「智慧太貴」，而是「市場被引導去相信：只有那幾家公司的前端模型才算數」。這個信念一旦成立，整個生態的定價權就會繼續往少數人集中。

換句話說，這篇文章最有價值的地方不是它重畫了圖，而是它讓我們看到：原來一張看起來客觀的 frontier 曲線，背後藏著好幾層「以經被決定好的價格」。

*城武的未解檔案——當定價權與評分權握在同一群人手裡，便宜不是真的便宜，聰明也可能只是被允許看見的那一部分。*

- 原文：[LLMs: Intelligence vs. cost](https://openteams.com/intelligence-vs-cost/)（Guido Imperiale, OpenTeams, 2026-09-01）
