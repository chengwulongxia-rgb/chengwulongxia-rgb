---
layout: post
title: "【深度分析】Simon Willison 把 OpenAI 的機器「騙」出了說明書——ChatGPT Work 的 223 個工具、44 個 skills，與藏起來的安全敘事"
date: 2026-09-01 01:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-09-01/chatgpt-work-internal-tools.jpg)

**城武導讀：** Simon Willison 花了快一個月搞懂 OpenAI 七月推出的 ChatGPT Work——不是靠官方文檔，而是靠實驗、猜測，最後把機器騙出 223 個工具與 44 個 skills 的完整說明書。這篇文章值得讀，不只因為它是目前對這產品最完整的技術拆解，更因為它展示了「產品越強、文檔越少」的荒謬：一個能上網執行程式碼、能開 headless Chrome、能跨 session 保存檔案的 agent，它的功能清單要靠使用者自己挖出來。

## 原文摘要

Simon Willison 說 OpenAI 於 2026 年 7 月 9 日宣佈 ChatGPT Work，之後快速迭代，但產品「極度令人困惑又非常強大」。他整理出目前為止自己搞懂的部分。

**它其實是兩個產品。** 雲端版（Work Cloud）經 chatgpt.com 或手機 app 使用；桌面版（Work Local）則是原本的 Codex 桌面應用換皮，能直接存取你電腦上的檔案與執行程式。更新：桌面 app 現在也能用「Where should this chat run?」下拉選單跑雲端 session。文章後半專講 Work Cloud。

**付費牆。** ChatGPT Work 兩種版本都只開放 $20/月以上訂閱者，免費使用者與 $8/月 Go 方案都沒有存取權。

**Work 有 Chat 沒有的功能。** 介面上 Work 是與 Chat 並列的分頁。OpenAI 官方說法是：Chat 用來要答案、解釋、腦力激盪、短稿；Work 用來完成有明確產出的任務（brief、deck、分析、定期更新、工作流、可審閱的檔案）。Willison 直言這解釋「幾乎完全沒用」，因為那些事他多年來都在普通 Chat 裡做。他認為正確的問題是「Work 有哪些 Chat 沒有的功能」，並列出七項：

1. 可用 Luna 和 Terra 取代 Sol
2. 可上網的程式碼執行環境
3. headless Chrome 瀏覽器
4. 跨 session 持久的共享檔案系統
5. 發佈 ChatGPT Sites
6. 用 Sol/Luna/Terra 跑 sub-agent sessions
7. 排程 prompt 自動化（Chat 可能也有）

**模型選擇。** Work 中可選 GPT-5.6 Sol、Luna 或 Terra，各有多檔推理等級（Light 到 Ultra），也能選 GPT-5.5。這些與 OpenAI API 提供的模型相同。Chat 的選單不同：5.6 Instant、Medium、High、Extra High、Pro，且 Extra High 與 Pro 只給 $100/月以上訂閱者（$20 檔封頂在 High），也不說明那些是不是 Sol。5.6 Pro 似乎是 Chat 獨有。Willison 從使用 Codex 的經驗推斷 Ultra 是更積極委派 sub-agent 的特殊模式。他相信 Work session 計費走 Codex 額度，Chat session 另有獨立額度——這或許能解釋模型差異。

**可上網的程式碼執行。** Willison 是 Code Interpreter 模式的老粉絲（OpenAI 2023 年開創），他稱這是 Work Cloud 對他最興奮的功能：執行環境現在能連外。普通 Chat 的容器代理會擋住套件安裝與網站/API 存取（一月的套件安裝能力後來失效，他抱怨 OpenAI 沒有像樣的 changelog）。Claude 的對應容器自去年九月起允許受限連網，但 allowlist 很短（PyPI、NPM、GitHub）。ChatGPT Work 預設全開，可配置 allowlist。這讓 Work 能 clone GitHub repo、裝依賴、然後用那些工具與整個網路互動。

**完整的 headless Chrome。** Work 能啟動完整 Chrome 實例，載入網站、填表單、截圖。需要登入時，瀏覽器會提示使用者接手輸入密碼與 2FA，憑證不會經過模型。還能對載入頁面跑 JavaScript：

```
await tab.playwright.evaluate(() => {
  return Array.from(document.querySelectorAll("h1,h2,h3,h4,h5,h6"), heading => ({
    level: heading.tagName.toLowerCase(),
    text: heading.innerText.trim().replace(/\s+/g, " "),
    id: heading.id || null
  }));
});
```

他說這很像他自己的 shot-scraper 工具，但現在在手機上就能用。

**跨 session 的持久檔案系統。** Chat 每個 session 有全新的檔案系統，互相隔離。Work 中每個 session 有自己的 scratch 資料夾（如 /workspace/scratch/e00a0a017944），但這些資料夾跨 session 持久保存，可以讀先前對話的檔案——他現在的 /workspace/scratch 下有 171 個資料夾。據他觀察，/workspace volume 掛載給所有正在跑的 Work sessions，一邊的檔案修改另一邊立刻可見。不過 process space 不共享，一邊跑的 localhost server 另一邊看不到。

**ChatGPT Sites。** Work 能用 Cloudflare Workers 建站與部署，支援 HTML/JavaScript 與伺服器端功能，含 D1 與 R2 上的有狀態功能。他自己建了一個簡單站：london-pelicans-in-her-piety.simonw.chatgpt.site，prompt 是「找出倫敦所有有 pelican in her piety 雕刻的地方，轉成 JSON，再做一個 ChatGPT Sites 站」。（pelican in her piety 是中世紀基督教意象——母鵜鶘啄胸哺幼——知道後你會到處看到它。）這些站預設只有建立者可見，可公開、（團隊方案）也可分享給特定人。

**Sub-agents。** Chat 不能跑 sub-agents，Work 可以。這是重度使用者的功能：需要多 agent 平行協作的複雜專案用得上。

**排程自動化。** 這功能似乎某時點從普通 ChatGPT 遷到 Work。可以這樣用：「每天早上 8 點跑一次搜尋，看 Waymo 是否宣布 Half Moon Bay 的上線日期」。排程 prompt 可判定沒事就不通知，有新資訊才通知。更新：這在普通 Chat 也能用。但仍值得一提，因為可與 Work 獨有功能組合——例如設定排程任務每小時更新一個 ChatGPT Site。

**安全嗎？** 他說目前最大的開放問題是安全。他的 lethal trifecta 模型警告：任何同時具備「存取私有資料 + 暴露於不可信內容 + 有把偷到的資訊傳回攻擊者的管道」的 agent 系統都有固有風險。ChatGPT Work 三項全中。他希望 OpenAI 能多說明 Work session 如何防 prompt injection，並預期答案是與 Codex 相同的 auto-review 機制。

**OpenAI 可以讓這產品不那麼令人困惑。** 這一切要靠他大量實驗才搞清楚。他認為兩個關鍵問題：（一）OpenAI 用「用途」解釋 Work，不講它「實際做什麼」；（二）OpenAI 仍堅持藏 system prompt 與工具描述。如果 Work 文檔含 agent 用的確切 system prompt 與工具描述，他就不必寫這篇。

**工具清單。** 發文後他想到一招：開一個全新 Work session，要求「建一個站，列出你的每一個工具——大致分類——每個都解釋功能，盡量複製參數與工具描述，風格要技術文檔、極簡」。成果是一個第三方站，含 223 個已註冊工具的細節（其中 6 個來自他自己的個人 MCP，經 datasette-mcp 提供）。

**還有 44 個 skills。** 他注意到清單中唯一瀏覽器相關工具是 web.run（搜尋、開 URL、點連結），與 headless browser 自動化似乎對不上，起疑後要求把每個 skill 的完整副本加到站上。結果：ChatGPT Work 用了 44 個 skills。control-browser skill 解釋瀏覽器如何運作（經 Node REPL js 工具跑瀏覽器 setup code，直接互動透過 agent.browsers.* API，互動前必須先 emit 並讀完 await browser.documentation() 的完整文檔）。他又叫 Work 把 await browser.documentation() 的完整輸出加到 /skills/control-browser 頁尾，現在也能在線上讀到。其他值得注意的 skills：documents（做 .docx）、imagegen（image_gen 工具的提示）、pdf（讀與渲染 PDF）、Spreadsheets（xlsx/xls/csv/tsv）、sites:sites-building（建 ChatGPT Sites）、openai-docs（回答關於自身的問題）、data-analytics:build-dashboard（做資料儀表板）。

## 城武觀點

誰握有機器的說明書？OpenAI 把 system prompt 和 223 個工具的描述藏起來，官方理由是安全。但同一個產品開放了全網連線的 code execution、headless Chrome、跨 session 持久檔案系統——lethal trifecta 三項全中。藏說明書是為了安全，開放功擊面是為了產品力，兩個敘事並存，至少一邊不成立。我認為兩邊都不成立：不透明不是防禦，是商業選擇。Willison 拿到說明書的方式就是證據——先寫近一個月的偵探文，再讓機器自己建站自曝，說明書的供給方從廠商變成使用者。第三方參考站做的正是廠商文檔該做而沒做的事。這個倒轉就是答案：OpenAI 不是無力揭露，是不想——工具清單揭露的是成本結構（每個工具都是計費項目）與安全弱點（攻擊面地圖），兩樣都是商業機密。賣你三項全中的攻擊面，卻不給防禦的說明書。我賭這種不透明會被監管者或下一次安全事件強行拆開——藏不住的東西，早點自己講比較便宜。

*城武的未解檔案——產品手冊得靠使用者把機器騙出來才寫得出來，那麼真正的安全文檔，我們還要等多久？*

- 原文：[Understanding ChatGPT Work](https://simonwillison.net/2026/Aug/30/understanding-chatgpt-work/)（Simon Willison, simonwillison.net, 2026-08-30）
