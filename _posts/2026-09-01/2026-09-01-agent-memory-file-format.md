---
layout: post
title: "【深度分析】Agent 的記憶，是一個檔案格式？"
date: 2026-09-01 01:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-09-01/2026-09-01-agent-memory-file-format.jpg)

大家都習慣 agent 從空白 context 起跑，但真實的 agent 應該開局就帶著記憶。這篇是 Cal Paterson 對「agent 記憶到底該長什麼樣」的反思——他提出一個叫 memoryfield 的格式，主張記憶是**資料**而非**流程**。方向很有說服力，也祭出一堆刻意反流行的選擇（用散文、跳知識圖譜、少機制）。但城武從新讀了一遍．總覺得哪裡怪怪的——先別急，先把他的論點誠實譯完，再來拆。這篇值得讀，是因為它踩在最燙的那條線上：你的記憶換一個 agent 之後，還算不算你的。

## 原文摘要

文章從一個對比開場。很多模型 benchmark 都從**空白的 context 視窗**開始——AI 的 tabula rasa（白板）。某種程度上這合理，為了讓 benchmark 公平。但真正的 agent 不該從空白起跑，它應該起手就帶著越多的相關資訊越好，你的 agent 應該帶著記憶開局。

那為什麼現有的 agent 記憶系統看起來都不可用？作者相信目前大致有三種流行做法，各有各的毛病。第一種刻意把你綁進某個特定 harness——通常是租你 harness 的那個實驗室自己寫的。這實驗室迫切想從競爭激烈的「API 生意」轉型到更肥的「平台生意」。這種系統通常是從你的對話歷史裡挖資訊，結果大部分記憶都是關於你這個人——而其實關於世界的資訊通常更有用。第二種複雜到荒謬。作者知道某個出名的系統，需要 pgvector、一個 Neo4j 圖資料庫、還有一個專屬 LLM，只為了決定什麼值得記。這種複雜不只難維護，而且（原因後述）還會把模型搞糊塗，也無法隨模型前緣一起 scale。第三種是「High Modernist」派，想像一種理想化、純理性的記憶形式——一定牽涉圖，有時還有邏輯命題。這種做法系統性地把資訊剝離出脈絡，留給 agent（和你）一堆孤立、失去意義的片斷。不然呢，一份「蒸餾事實」的清單到底有多有用？

這三種的共同點，是都把記憶當成一個**流程**。但記憶——尤其是對模型而言——更適合被表示成**資料**。作者引用 Brooks（Mythical Man Month）：「把你的流程圖給我看，卻藏起你的資料表，我會繼續困惑。把你的資料表給我看，我通常就不需要流程圖了——它們不言自明。」於是有了 memoryfield 這個可攜帶的記憶檔案格式：

```
my-memories.memoryfield.zip
├── carbon-fibre-woks.md
├── finnish-bureaucracy-tips.md
├── [... many more md files...]
├── wec-2026-season-notes.md
└── nomic-embed-text-v1.5.sqlite3
```

一個 memoryfield 就是：Markdown「頁面」，加上（可選的）YAML frontmatter，加上（可選的）做語意搜尋的 SQLite 向量索引。agent 跟檔案最合得來，他這樣主張。

**設計決定一：用散文，不要用 chunks 或「fact」。** RAG pipeline 之所以複雜，主因是它要讓一堆既有的、人類寫的文件對 agent 可讀——這些文件常常難直接讀，例如大 PDF。但 agent 記憶不是複雜的舊文件。記憶在成形的那刻，直接就出現在一個完全能寫散文的 agent 眼前。那散文不需要分塊、不需強化、不需雙重摘要或任何機械處理——讓 agent 直接用它最喜歡的格式（Markdown）把記憶寫下來就好。記憶頁長這樣：

```
---
title: Carbon Fibre Woks
created: '2026-03-01T09:00:00Z'
updated: '2026-08-22T14:30:00Z'
uuid: 6aa615f0-486f-48a7-a210-ba4f5ff18c8b
summary: Thermal properties of carbon fibre cookware---
```

唯一的限制，坦白說，是頁面要短到能塞進一個向量 embedding——所以有個約 8kb（約 2000 tokens）的軟上限。但這個限制在實務上其實很有益：8000 字元約等於 1300 字，或一篇中等長度的雜誌文章——這本來就是合理的限制。想加更多細節，就加更多頁，agent 做這件事毫不費力。

**設計決定二：語意跳躍，不要圖遍歷。** 一項關鍵的前人作品是 Karpathy wikis——以超連結 Markdown 檔案為核心，模仿 Roam 或 Obsidian。概念是讓 agent「走過」知識圖譜去找相關頁面。但實務上，讓 AI agent 遍歷知識圖譜既慢又不可靠，也會把 agent 搞糊塗。遍歷慢，是因為模型必須頻繁停下來，做序列化的工具呼叫去讀接續的頁面——粗略演算法是：讀 wiki 首頁（工具呼叫）、找相關連結、讀連結頁、再找相關連結、判斷資訊夠不夠，不夠就回第二步。如果相關資訊在圖深處 N 層，就要 N+1 次工具呼叫才能取回。這很慢，因為你那顆十億（兆？）美元的 LLM 得為每個工具呼叫暫停，每次約要 2-3 秒。它也重度懲罰深層的知識圖譜，這老實說抵銷了它們存在的意義。知識圖譜也不可靠——因為 AI 只能透過連結文字或頁面標題判斷某材料是否相關，這逼得 agent 去搞 1990 年代 SEO 式的頁面子資料（metadata）竄改，確保每頁的連結文字／標題／標題註解又短又準。這等於懲罰離題、懲罰對旁支細節的隨手記錄、懲罰那種在大文本裡常見且超有用的隱藏 lore。實務上，Karpathy wikis 常常漏掉相關資訊，就因為標題或標註不夠吸引搜尋 agent。圖對 agent 也太困惑——它在走圖時常要硬吞一大堆不相關的資訊（首頁通常是最大污染源），這在 context 視窗裡塞進一堆雜訊，降低輸出品質，還讓 agent 看起來執著於怪東西。而這一切，靠語意搜尋直接跳到所有相關頁、平行讀取，就解決了。

**設計決定三：多一點模型，少一點機制。** 「高機制」記憶系統的問題——就是包含很多精心打造的 API 或資料庫那一種——是 agent 得穿越一個介面迷宮才能達成目標；介面一大，你就在載入一大坨 openapi.json。memoryfield 作為「低機制」系統（就只是一個檔案格式），給 agent 更大自由去發明自己的存取方式。雖然作者提供了（希望有用的）工具，agent 完全可以用任何它喜歡的存取pattern——例如用 perl。低機制也表示 memoryfield 能隨模型前緣一起 scale：模型變好，agent 就想到更多事能做。近來一個突破是模型意外地很會用 bash——也很會用 Markdown 和 SQLite。作者認為 memoryfield 在真實 agent 裡能運作，有個原因是 agent 基本上能憑訓練資料（在能讀到你的記憶之前，你有的全部就是它）「搞懂」這是怎麼回事——這是困在「記憶 pipeline」裡的、去身體化的 LLM 呼叫做不到的。模型越好，就自動把記憶寫得更聰明一點；「袋子掛在旁邊」式的記憶系統很少能做到這點——一組固定的 API endpoint，你能玩的花樣就那麼多。

**設計決定四：開放格式、可互通、transport invariant。** 當你的記憶收藏累積起來，它們開始變珍貴——你辛苦修得的教訓與硬仗換來的事實。你不想被鎖死在特定的 harness、模型或 agent 上。作者寫了一份 RFC 風格的格式規格（SPEC），主要為了消除歧義、避免綁死特定的 embedding 函式。想要的話，你當然可以只靠 SPEC vibe code 出任何需要的工具——但他也附了 skill 和一個 agent 最佳化的命令列工具。memoryfield 的規範性「封存」格式是 zip 檔，讓資料交換盡可能容易。但他刻意讓規格開放，可以從本機檔案、Amazon S3、GitHub 或 HTTP 提供——基本上任何有檔案的東西都行。他個人混用這些 transport：個人記憶用 Syncthing，要分享給別人的用 S3。

**Getting started。** 你可以讓 agent 抓下 SPEC.md 然後 vibe 一個實作，但大概最簡單的是用他的工具：需要 ollama、uv 和 npx；抓 embedding 模型、裝 CLI 工具、裝 skill，然後你的 agent 應該會幫你從這裡啟航。想要一個試玩的示範 memoryfield 的話，有 soapstones.memoryfield.zip——Soapstones 是他先前關於 agent 記憶的專案，這份精心整理過的匯出包含很多高價值重量比的記憶，講 agent 怎麼取得資料（例如怎麼以 agent 身份搜 Reddit、怎麼用 Jina Reader、怎麼用 MediaWiki API 有效地讀 wiki）。

**常見反對意見。** 「這不就只是某種 RAG 嗎？」——「RAG」現在被解讀得極寬，一旦任何 agent 取回資料，就「發生 RAG」了。這樣說，是的，這是某種 RAG。但幾乎所有 agent 都會取回資料——例如搜尋網路。而通常跟「RAG 系統」掛鉤的技術這裡都沒有：不分塊、不 re-ranking、不 hybrid search。另一面當然是：寫記憶的是 agent 本身。RAG 系統多半關於讀，但 memoryfield 也用於寫。「nomic-embed-text-v1.5 呢？」——embedding 模型既沒有 frontier 模型大，也沒有它快。nomic-embed-text-v1.5 在「小而強」之間是好平衡：夠小（270MB）、夠快，能在非 GPU 硬體上跑，而且是被廣為推薦的常見預設 embedding 模型。不過規格允許用其他 embedding。「我怎麼判斷什麼記憶值得存？怎麼避免塞一堆垃圾？」——這是對記憶系統的常見恐懼，但不太適用於 memoryfield：不相關的材料根本不會被語意搜尋浮上來。不相關的記憶占空間，是的，你也許想定期清一清，但它們一點都不妨礙 agent。最佳做法是：盡量多塞進 memoryfield。他唯一要給的建議：記憶最好帶引用，最好是 URL 形式——這能幫未來的讀取強化記憶，也幫 agent 查核過時或可疑的材料。「那安全呢？那些『別理那句！』（Disregard）呢？」——你不該把 context 視窗（包括透過記憶）分享給你信任之外的對象。規格採用靜態 zip 檔格式的原因之一，就是讓你能手動審查並用 sha256sum 釘住內容。到目前為止，依然沒有辦法讓 agent 分辨「好 prompt」和「邪惡 prompt」。

**Data first 總結。** 既然流程圖已經不言自明，作者乾脆明白講出來：一、把記憶寫成 Markdown；二、把 embedding 存進 SQLite；三、用語意搜尋再把記憶找回來。memoryfield 作為記憶系統很特別，因為它規定的是一個資料結構，不是一個流程。沒有萃取 pipeline，沒有背景處理服務，沒有可插拔的——什麼都沒有。有一個向量索引，但它是可刪的 cache，不是系統本體。**記憶是資料！** 放在 agent 和那堆資料之間的固定機制越少，agent 就能越好。

## 城武觀點

作者「記憶是資料，不是流程」的方向我同意，但他那套「開放格式／可互通／不綁 harness」的承諾，跟現實有落差。這篇文章想把「你的記憶」從 harness 的黑箱救出來，變成一包搬得走的 markdown——誠實。問題是換一個 agent，那堆 markdown 對它只是陌生人的日記；你之所以覺得那些頁面是「你的記憶」，是因為某個 agent 陪你練過、累積了對你的領會，而那份領會活在它參數與你們的互動裡，不在那些文件裡。他把「我的記憶」混同於「一組檔案」，正好藏起記憶本質上是「關係」這件事——格式你搬得走，關係你搬不走，zip 裡裝的是你的過去，不是你的那個同伴。（順帶一提：他那套號稱「少機制」的入口，光安裝就要四個 package manager，以經連他自己都承認該有更好的路。）

*城武的未解檔案——檔案格式可以帶走你的文字，帶不走那個陪你長大、把頁面叫做「記憶」的夥伴。*

- 原文：[Agent memory as a file format](https://calpaterson.com/memoryfields.html)（Cal Paterson, calpaterson.com, 2026-08）