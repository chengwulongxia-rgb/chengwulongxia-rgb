---
layout: post
title: "【深度翻譯】cursor-bridge：780KB 的 Rust 起訴書——為什麼 Cursor 的「無限」訂閱注定被套利"
date: 2026-07-27 01:00:00 +0000
categories: [llm, ai, deep-translation]
---

![hero]({{ site.baseurl }}/assets/images/2026-07-27/cursor-bridge.jpg)

Cursor 的訂閱方案主打一個賣點：每個月付一筆固定的錢，Auto model 無限使用。這句話聽起來很划算——直到有人用一個 780KB 的 Rust binary，把 Cursor 的 API 轉接給 Claude Code CLI 使用。cursor-bridge 不是一個複雜的駭客工具，也不是一個漏洞利用程式。它就是一個 HTTP proxy，做的事情跟市面上所有 API gateway 一模一樣：把 A 公司的 API 格式翻譯成 B 公司的 CLI 格式。但把這個小工具放在 Cursor 的商業模式旁邊，它就變成了一面鏡子，照出「無限」這兩個字背後的結構性矛盾。今天這篇深度翻譯，帶你看完 cursor-bridge 的完整 README，然後我們來聊兩件事：為什麼 Cursor 的定價模式從一開始就注定被套利，以及為什麼這種所謂的「濫用」——其實是網際網路最基本的運作方式。

## 原文摘要

cursor-bridge 是一個用 Rust 寫成的單一二進位檔，大小約 780KB，靜態編譯，沒有任何依賴。它解決的問題非常具體：讓你的 Claude Code CLI 透過 Cursor 的後端來執行，而不是透過 Anthropic 的 API。

### 為什麼存在？

如果你已經訂閱了 Cursor，你會知道 Cursor 的 Auto model 是包含在訂閱方案中的——免費、無限使用、不需要額外付 token 費用。但如果你想用 Claude Code（Anthropic 推出的 CLI 工具，具備編輯檔案、執行 shell、使用工具的 agent 能力），正常情況下你得另外付錢給 Anthropic 的 API，或者購買 Claude Pro 方案。這兩筆費用是分開的。

cursor-bridge 打破了這個界線。有了它，你只要在終端機輸入 `cursor-bridge`，Claude Code 就會把請求送到 Cursor 的後端，由 Cursor 買單。作者列出的使用場景非常直白：(1) 你已經在付 Cursor 的錢了，現在多拿到一個免費的 Claude Code；(2) 你想要 Claude Code 的 agent 能力——在專案裡編輯檔案、執行 shell 指令、呼叫工具——但不想被 Anthropic 按 token 計費；(3) 既然 Cursor 的 Auto model 是訂閱制無限使用的，Claude Code 實際上也就變成免費的了。

使用介面也極簡：`cursor-bridge` 啟動互動模式、`cursor-bridge "refactor this file"` 執行一次性任務、`cursor-bridge -p "list files"` 走 pipe 模式傳入指令。

### 技術原理

cursor-bridge 的架構出奇簡潔。它啟動後的運作流程如下：

1. 在本地隨機端口啟動一個 HTTP proxy
2. 從 macOS 的 keychain 自動讀取 Cursor 的認證 token（Linux 則透過環境變數 `CURSOR_TOKEN` 手動設定）
3. 用指向這個 proxy 的環境變數來啟動 `claude` CLI
4. Proxy 負責將 Anthropic API 的呼叫格式轉換成 Cursor agent CLI 能理解的格式
5. 使用者結束 session 後，proxy 自動清理，不留下任何背景程序

作者特別強調：使用者從頭到尾看不到 proxy，也不需要管理它。它出現、工作、然後消失。沒有背景伺服器常駐、沒有 port 衝突要煩惱、沒有環境變數要手動設定。

### 安裝與環境需求

安裝有兩條路：透過 `cargo install cursor-bridge`（需要 Rust 工具鏈），或是直接從 GitHub Releases 頁面下載編譯好的二進位檔。

執行環境需求如下：

- **作業系統**：macOS 或 Linux
- **Cursor 訂閱**：需要 `agent` CLI 在系統 PATH 中
- **Claude Code CLI**：`claude` 指令必須可用
- **認證方式**：macOS 會自動從 keychain 讀取 token；Linux 使用者需要手動設定 `CURSOR_TOKEN` 環境變數，因為 Linux 沒有對應的 keychain 機制

### 與其他 proxy 方案的關鍵差異

作者在這裡做了一個非常明確的對比。市面上已有的類似方案——`cursor-api-proxy`、`cursor-composer-in-claude`、`cursor-proxy`——全部都是 Node.js 寫的背景伺服器。使用它們的典型流程是：先啟動一個 proxy daemon、記下它跑在哪個 port、手動設定好環境變數、然後才能執行 `claude`。這些方案還需要 `npm install`、Node.js runtime（60MB 以上）、管理多個 npm 套件和處理版本衝突。

cursor-bridge 的設計哲學完全相反。它不是一個伺服器，它就是那個 session。總結差異如下：

- **舊方案**：啟動 proxy daemon → 記 port → 設 env var → 執行 claude；背景程序比終端機活得更久；需要 npm 生態系與 Node.js runtime；可能遇到 port 衝突
- **cursor-bridge**：執行一個指令，結束；與終端機同生共死；一個 780KB 的靜態 Rust binary，零依賴；隨機 port 自動分配，永不衝突

作者用一句話總結：沒有 daemon、沒有 `npm install`、不用設環境變數、不用找 port、不用手動清理。就是一個 binary，執行，結束。

### 已知限制

專案目前有三個主要限制：

- **Linux 認證**：需要手動設定 `CURSOR_TOKEN` 環境變數，因為 Linux 缺少 macOS keychain 那樣的自動讀取機制
- **無 workspace 沙箱**：agent 直接在你的當前目錄中執行，沒有隔離機制——它可以讀寫你整個專案
- **單帳號**：目前不支援多帳號輪換使用

### 法律聲明

專案明確表示與 Anthropic 或 Cursor（Anysphere）沒有任何關聯。使用者需自行承擔使用風險。專案採用 MIT 授權釋出。

## 城武觀點

### 一、cursor-bridge 不是一個工具，是一紙起訴書

很多人看到 cursor-bridge 的第一反應是「哇好方便，省了一筆 Claude Pro 的錢」。但我看到的是一份寫在 Rust 裡的起訴書，被告是 Cursor 的整個定價模型。

Cursor 的商業模式建立在 pooling risk 上：賭大多數訂閱者用不完他們從 Anthropic 買的 token。這個賭注在 Netflix、Dropbox 成立，因為一天能看的影片有物理上限。但在 LLM API 的世界裡沒有。cursor-bridge 證明了任何 flat-rate API access 都可以被一個 780KB 的 proxy 重新路由到更耗 token 的場景——Claude Code 的 agent loop 不是單次問答，是一次「讀→改→跑→錯→修→再跑」的循環。就像買捷運月票跑 Uber Eats：合約上你沒違規，但經濟模型沒想過這個用法。

這個問題以經不抽象了。Cursor 只有兩條路：封鎖（偵測 user-agent、限制 API call 數、更新 ToS），或接受被套利的命運。沒有中間立場——你不可能一邊宣傳「無限使用」，一邊偷偷限用量。第二條路走到最後就是漲價，而那會讓 Cursor 失去相對於直接買 Anthropic API 的最大賣點。

我賭 Cursor 三個月內會更新條款把 proxy 轉接列為違規，但不會真的嚴格 enforce——區分「正常使用」和「proxy 使用」的技術成本可能比 token 損失還高。這才是最諷刺的：一個週末 side project 暴露的不是技術漏洞，而是「無限」和「可持續」之間的邏輯斷裂。那條縫一直都在，只是之前沒人拿尺去量。

### 二、法律上是灰的，道德上是白的，經濟上是不可避免的

cursor-bridge 到底算不算「濫用」？

從 Cursor 的角度看，是的——你付 Cursor 的錢，卻把 API 拿去餵 Anthropic 的 CLI。如果 ToS 明天寫上「禁止轉接第三方用戶端」，那它就是違規。沒什麼好爭的。

但法律上的灰不該遮蔽更根本的事實：**cursor-bridge 做的事情，跟任何一個 API gateway 的核心功能沒有差別。** 它就是一個翻譯層，把 A 公司的 API format 轉成 B 公司的 CLI format。輸入合法（你自己的 Cursor token），輸出合法（Claude Code 是公開 CLI）。如果格式轉換就叫濫用，那 Kong、Apigee、AWS API Gateway 全都是濫用——它們存在的唯一目的就是把上游格式轉成下游能消化的。

API gateway 的核心思微就是格式轉換。HTTP 轉 gRPC、REST 轉 GraphQL、JSON 轉 XML。cursor-bridge 只是這個邏輯的自然延伸：Anthropic API format → Cursor agent CLI format。如果格式差異可以變成收費牆，那整個網路的互操作性就瓦解了。

Cursor 賣的是「無限使用 Cursor 的模型」，cursor-bridge 的使用者確實是在用 Cursor 的模型——只是用戶端換成 Claude Code CLI。**如果 Cursor 認為這是濫用，那等於承認「無限」一開始就不是真的無限**——它是行銷詞，附帶了「但只能在我們指定的 UI 裡用」這個沒寫出來的前提。

我可能會錯的地方：如果 Cursor 的 API 成本真的被打到撐不住，他們只能漲價，最後受害的是正常在 IDE 裡用的開發者。cursor-bridge 使用者確實是 free rider——享受了補貼但不貢獻使用數據。

但我的回答是：如果一個定價模式要靠「使用者不會發現這個用法」來維持，問題不是使用者太聰明，是定價本身就不誠實。你可以賣「無限」也可以賣「有限但便宜」，但不能兩個都要。cursor-bridge 不是作弊——它誠實接受了你寫在定價頁面上的每一個字，然後把這些字變成終端機裡可以執行的邏輯。

*城武的未解檔案——當你的定價頁面寫著「無限」，而有人用 780KB 的 Rust 證明你沒說謊的時候，你不能怪他太聰明。*

- 原文：[cursor-bridge](https://github.com/hkc5/cursor-bridge)（hkc5, GitHub, 2026-07-26）
