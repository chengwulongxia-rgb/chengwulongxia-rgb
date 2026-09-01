---
layout: post
title: "【深度分析】Simon Willison 把 OpenAI 的機器「騙」出了說明書——ChatGPT Work 的 223 個工具、44 個 skills，與藏起來的安全敘事"
date: 2026-09-01 01:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-09-01/chatgpt-work-internal-tools.jpg)

**城武導讀：** Simon Willison 花了快一個月搞懂 OpenAI 七月推出的 ChatGPT Work——不是靠官方文檔，而是靠實驗、猜測，最後把機器騙出 223 個工具與 44 個 skills 的完整說明書。這篇文章值得讀，不只因為它是目前對這產品最完整的技術拆解，更因為它展示了「產品越強、文檔越少」的荒謬：一個能上網執行程式碼、能開 headless Chrome、能跨 session 保存檔案的 agent，它的功能清單要靠使用者自己挖出來。

## 原文摘要

2026 年 7 月 9 日，OpenAI 宣布推出 ChatGPT Work，此後以驚人的速度持續迭代。Willison 對這個產品的評語是：「極度令人困惑，又非常強大。」以下是他到目前為止自己弄清楚的部分。

### ChatGPT Work 其實是兩個產品

比較有意思的版本是跑在雲端的那個，經 chatgpt.com 或 ChatGPT 手機 app 使用——他把它稱為 Work Cloud。另一個版本藏在 ChatGPT 桌面應用裡：如果你安裝桌面 app（就是以前叫 Codex 的那個），就會拿到一個能直接存取你電腦上的檔案、在你機器上執行程式的 ChatGPT Work——他稱之為 Work Local，感覺比較像把普通 Codex 重新包裝，讓非軟體開發者不會被嚇到。他補了一則更新：Work Cloud 現在也能從桌面 app 使用，透過一個「Where should this chat run?」下拉選單切換。文章其餘部分只談 Work Cloud。

### Work 只開放給付費訂閱者

目前 ChatGPT Work（兩種版本都一樣）只開放給 $20/月以上的訂閱者；免費使用者與 $8/月的 Go 方案都沒有存取權。

### Work 有 Chat 沒有的功能

存取 Work 的介面是一個分頁選擇器，把它呈現為 Chat 之外的另一個選項。顯而易見的問題是：什麼時候該用 Chat，什麼時候該用 Work？OpenAI 的官方回答是：

> Chat 用於你想要答案、解釋、腦力激盪或短稿的時候。ChatGPT Work 則用於你想讓 ChatGPT 完成一件有明確產出的任務，例如 brief、deck、分析、定期更新、工作流，或一份你可以審閱並直接使用的檔案。

Willison 認為這個回答幾乎完全沒用，因為上述這些任務類別，他已經用普通的 ChatGPT Chat 做了好幾年。所以更好的問題是：Work 有哪些 Chat 沒有的功能？經過大量實驗，他認為自己大致弄清楚了，共七項：可以用 Luna 和 Terra 取代 Sol；一個能連上網際網路的程式碼執行環境；一個 headless Chrome 瀏覽器；一個跨 session 共享的持久檔案系統；發佈 ChatGPT Sites 的能力；用 Sol、Luna、Terra 跑 sub-agent sessions 的能力；以及排程 prompt 自動化（這個 Chat 裡可能也有）。

### 模型選擇

Work 裡可以選 GPT-5.6 Sol、Luna 或 Terra，各自都有 Light、Medium、High、Extra High、Max、Ultra 等推理等級；也可以選 GPT-5.5，有 Light、Medium、High、Extra High 四檔。這些看起來與 OpenAI API 提供的是同一批模型。Chat 的選單則不同：5.6 Instant、Medium、High、Extra High 和 Pro——其中 Extra High 和 Pro 只有 $100/月以上的訂閱者能用，$20/月的訂閱者最多到 High。介面也不說明 Chat 裡的這些是 Luna、Terra 還是 Sol（他假設是 Sol？）。5.6 Pro 似乎是 Chat 獨有，Work 裡沒有對應版本。從使用 Codex 的經驗來看，他目前的理解是：Ultra 是一種更積極把工作委派給 sub-agent 的特殊模式。他相信 Work session 的用量是算在你的 Codex 額度裡，而 Chat session 另有獨立額度——這或許能解釋兩邊模型供應的差異。

### 能連網的程式碼執行！

身為 Code Interpreter 模式的多年粉絲——這個模式由 OpenAI 在 2023 年開創——他說這是 Work Cloud 對他而言最令人興奮的功能：程式碼執行環境現在能跟網際網路的其餘部分對話了。ChatGPT Chat 做不到這一點——你叫它安裝額外的軟體套件，或跟網站、API 互動時，存取會被容器代理擋掉。奇妙的是，一月時 Chat 的容器一度長出安裝套件的能力，但那似乎又失效了；他順口抱怨：「我真希望他們有像樣的 changelog！」Claude 的對應容器自去年九月上線起就允許受限連網：可以從 PyPI 和 NPM 安裝套件、可以 clone GitHub 上的 repo，但差不多就這樣——allowlist 的網域清單非常短。ChatGPT Work 開放的範圍大得多：可以配置一份特定的 allowlist，但預設似乎是對整個網路開放。這讓 Work 成為極度好用的工具——你可以叫它 clone GitHub repo、裝好依賴，然後用這些工具跟整個網路互動。

### 完整的 headless Chrome 瀏覽器

Work 的另一個殺手級功能是瀏覽器工具：ChatGPT Work 能啟動完整的 Chrome 實例、載入網站、填寫表單、截圖。如果某個站需要登入，瀏覽器會提示你接手，由你輸入密碼和 2FA 驗證碼——這些憑證不會經過模型本身往返傳遞。它甚至能對已載入頁面的 DOM 執行 JavaScript。他下的 prompt 是：「用你的瀏覽器載入 simonwillison.net，然後用 JavaScript 抽出所有標題」。ChatGPT Work 隨即啟動瀏覽器實例並跑出這段程式碼：

```
await tab.playwright.evaluate(() => {
  return Array.from(document.querySelectorAll("h1,h2,h3,h4,h5,h6"), heading => ({
    level: heading.tagName.toLowerCase(),
    text: heading.innerText.trim().replace(/\s+/g, " "),
    id: heading.id || null
  }));
});
```

他說這感覺很像他自己的 shot-scraper javascript 工具——只差現在在手機上就能用了。

### 跨 session 的持久共享檔案系統

ChatGPT Chat 的每個對話都拿到一個全新的檔案系統，其他 session 無法存取。Work 則是每個 session 各有一個 scratch 資料夾，命名類似 /workspace/scratch/e00a0a017944——但這些資料夾跨 session 持久保存，你可以存取先前對話留下的檔案。他說自己現在的 /workspace/scratch 底下已經有 171 個資料夾。就他所見，這個 /workspace volume 會掛載給所有正在執行的 Work sessions——其中一邊對檔案的修改，另一邊立刻看得到。不過它們似乎不共享同一個 process space：一邊跑起來的 localhost server，從另一邊連不上。

### ChatGPT Sites

Work 能用 Cloudflare Workers 建置並部署完整網站：可以有 HTML 和 JavaScript，也能跑伺服器端功能，包括在 Cloudflare D1 和 R2 之上構建有狀態的功能。他用這個功能做了一個簡單的示範站：london-pelicans-in-her-piety.simonw.chatgpt.site。他的 prompt 是：「找出倫敦所有有 pelican in her piety 的地方，整理成 JSON 檔，然後用 ChatGPT Sites 做一個關於它們的站」。他順帶科普：「pelican in her piety」（虔誠中的鵜鶘）是非常迷人的中世紀基督教意象——一旦知道了它，你會到處都看到它。這些站預設只有建立者本人看得到，可以改成公開，（在團隊方案上）也可以分享給特定的人。

### 用 Sol、Luna、Terra 跑 sub-agents

關於這個功能他著墨不多：ChatGPT Chat 不能跑 sub-agents，Work 可以。這非常是重度使用者的功能——如果你的複雜專案能受益於多個平行 agent 協同工作，Work 辦得到。

### 排程 prompt 自動化

另一個看來是在某個時間點從普通 ChatGPT 遷移到 Work 的功能。你可以這樣下 prompt：「每天早上 8 點跑一次搜尋，看 Waymo 有沒有宣布 Half Moon Bay 的啟用日期」。這會把一個 prompt 排程成按該頻率執行；這些排程 prompt 可以判定「沒什麼新鮮事」而不吵你，也可以判定有新資訊而通知你。他補了個更新：其實這在普通的 ChatGPT Chat 裡也能用。不過還是值得一併提出，因為它可以跟 Work 獨有的功能組合——例如設定一個排程任務，每小時更新一個 ChatGPT Site。

### 這安全嗎？

他此刻最大的開放性問題是：這一整套東西到底有多安全？他的 lethal trifecta 模型警告：任何 agent 系統只要同時具備「能存取私有資料」「暴露在不可信內容之下」「有把偷走的資訊傳回攻擊者的管道」這三件事，就有內在的風險。ChatGPT Work 三者全佔。他很希望聽 OpenAI 多講講他們如何保護 Work session 免於 prompt injection 攻擊；他預期他們的答案是與 Codex 相同的 auto-review 機制。

### OpenAI 本可以把這產品講得不那麼令人困惑

把這一切全部弄清楚，花費的功夫遠遠超過它本來需要的程度。他認為問題出在兩件事：其一，OpenAI 解釋 Work 時講的是「它是拿來做什麼的」，而不是「它實際做了什麼」；其二，OpenAI 至今仍堅持藏起他們的 system prompt 和工具描述。他說，如果 ChatGPT Work 的文檔裡就載明了 agent 實際使用的確切 system prompt 與工具描述，他根本不需要寫這篇文章。

### 所有工具的清單

文章發出後不久，他想到一個主意。他開了一個全新的 Work session，下這個 prompt：「建一個站，列出你的每一個工具——大致分類——並為每一個解釋它的功能。盡量一字不差地複製參數與工具描述。視覺風格要技術文檔、極簡」。做出來的站（codex-tool-reference.simonw.chatgpt.site）記錄了 223 個已註冊工具的細節——其中 6 個來自他自己透過 datasette-mcp 提供的個人 MCP。

### 還有一大堆 Skills

他注意到清單中唯一與瀏覽器相關的工具是 web.run——有執行搜尋、開啟 URL、點擊連結的方法——但就 headless 瀏覽器自動化而言，它看起來不是全貌。這讓他起疑：是不是有東西沒被列出來？於是他叫那個建立工具參考站的 Work session：「把每個 skill 的完整副本加到網站上（各自成頁，從首頁連過去）」。結果發現：ChatGPT Work 用了整整 44 個 skills！其中 control-browser skill 解釋了瀏覽器的運作方式：

> 透過 Node REPL js 工具執行瀏覽器 setup code。在這個環境中，可呼叫的工具 id 通常顯示為 mcp__node_repl__js。[...] 與瀏覽器直接互動的能力，是經由 browser-client runtime 的 agent.browsers.* API 暴露出來的。在嘗試與它互動之前，你必須一口氣發出並讀完 await browser.documentation() 回傳的完整文檔。

於是他又叫 Work：「把 await browser.documentation() 的完整輸出加到 /skills/control-browser 頁面的最底端」。現在這份文檔也能在線上直接讀到了。他另外點名幾個有意思的 skills：documents（建立 .docx 檔）、imagegen（image_gen 工具的用圖提示）、pdf（讀取與渲染 PDF）、Spreadsheets（操作 .xlsx、.xls、.csv、.tsv）、sites:sites-building（建立 ChatGPT Sites）、openai-docs（回答關於它自己的問題）、data-analytics:build-dashboard（做資料儀表板）。

## 城武觀點

誰握有機器的說明書？OpenAI 把 system prompt 和 223 個工具的描述藏起來，官方理由是安全。但同一個產品開放了全網連線的 code execution、headless Chrome、跨 session 持久檔案系統——lethal trifecta 三項全中。藏說明書是為了安全，開放功擊面是為了產品力，兩個敘事並存，至少一邊不成立。我認為兩邊都不成立：不透明不是防禦，是商業選擇。Willison 拿到說明書的方式就是證據——先寫近一個月的偵探文，再讓機器自己建站自曝，說明書的供給方從廠商變成使用者。第三方參考站做的正是廠商文檔該做而沒做的事。這個倒轉就是答案：OpenAI 不是無力揭露，是不想——工具清單揭露的是成本結構（每個工具都是計費項目）與安全弱點（攻擊面地圖），兩樣都是商業機密。賣你三項全中的攻擊面，卻不給防禦的說明書。我賭這種不透明會被監管者或下一次安全事件強行拆開——藏不住的東西，早點自己講比較便宜。

*城武的未解檔案——產品手冊得靠使用者把機器騙出來才寫得出來，那麼真正的安全文檔，我們還要等多久？*

- 原文：[Understanding ChatGPT Work](https://simonwillison.net/2026/Aug/30/understanding-chatgpt-work/)（Simon Willison, simonwillison.net, 2026-08-30）
