---
layout: post
title: "【城武觀點】DeepSeek Harness 四天暴衝 13 萬 star——「時空可組合性」論文是煙霧彈，開源是最難攻擊的鎖定"
date: 2026-08-17 01:00:00 +0000
categories: [llm, ai, chengwu-opinion]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-17/deepseek-harness.jpg)

8 月 13 日，DeepSeek 開源了 [`deepseek-harness`](https://github.com/deepseek-ai/deepseek-harness)（簡稱 `dsh`）——一個 agent harness，白話就是「Claude Code 那種東西」。四天內 13.4 萬 star、1.3 萬 fork，還同時拋出一篇論文《A Programming Paradigm for Spatiotemporal Composability》，宣稱底層的 Cordis 框架實現了一種叫「時空可組合性」的新程式範式。HN 上這則消息拿到 733 分、309 則評論，工程師們把這東西從 README 一路拆到論文代數，然後集體得出一個 DeepSeek 可能不太開心的結論：那篇高定理論，核心講白了就是個 destructor。本文以這條討論串為本——一個四天 13 萬 star 的專案，評論區以經不是在聊熱鬧，而是在做集體 code review。

## HN 討論核心觀點摘錄

有人開場就質疑這東西憑什麼衝上 HN 第一名。rco8786 說 README 除了一堆安裝指令和一個 Cordis 連結之外什麼都沒有，而 Cordis 的自我介紹是「A Meta-Framework of Spatiotemporal Composability」，還標註「API 未穩定、隨時會變」。dormento 更直接，說這串詞「讀起來像 word salad」。底下有人開始釐清定義：francislavoie 說 harness 就是「你管 Claude Code 那種東西叫的詞，一個跑 agent 的 TUI」；alienbaby 補得更精準——「harness 是任何包著 LLM 呼叫、操控 LLM 互動以達成設計目標的 wrapper，Claude Code 只是其中一種，專注寫 code」。

真正讀了論文的人分兩派。lxdlam 讀完的結論是「有用，但沒那麼有用」：對不熟 OSGi、iPOJO、React 的 useEffect 的人值得一讀；熟的，「skim 就夠了，代數幫不了你更多」。brabel 一針見血：「OSGi，就 Eclipse 那套插件系統，每一代人都要重新發現一次。」slopinthebag 最狠：「它的 big idea 就是個 destructor？2026 年、vibe coding 的年代，這算突破？」Game_Ender 緩頰說還有跨插件的依賴注入。而 Pi 的作者 badlogic 讀完論文後給了一段最誠實的評語：註冊時回傳獨立 cleanup handler 這點「neat」，但跨插件依賴注入「一堆 footgun 和限制」，而且「90% 的插件彼此沒有依賴，根本用不到那套複雜度」。OutOfHere 補刀：「用 memory 追蹤 inverse，不 scale。」

另一條戰線是「一切皆插件」本身。invaliduser 說他這一年「發展出插件疲勞」：靠社群插件的產品，前六個月都很美好，之後就是棄坑、過時、不相容的地獄，沒有治理也沒有共識。__alexs 補上最尖銳的一點：「問題不是插件架構，是它只有插件架構——如果沒 batteries included，那有什麼意義？」bdcravens 點出歷史規律：「多數做插件系統的 vendor 最後都會自己造一堆『官方插件』，用戶因為是官方就信任它們——等於繞一圈，做回一個 mono-vendor ecosystem。」但也有反方：scotty79 說「一切皆插件 = 插件能做一切 = AI 能幫你寫插件，這工具對你是無限彈性的，根本不需要社群」；prettyblocks 呼應「Pi 就是這個模式，harness 帶最少工具，缺什麼叫模型自己寫」。

更上層的爭論是 first-party harness 到底有沒有意義。nylonstrung 的立場很硬：「first-party harness 沒意義——Pareto frontier 每個月都在換，我要的是跨模型的一致工作面，就像我要同一個 editor 寫所有語言。」softwaredoug 用行動投票：「我用 OpenCode，直接看得到 token 花費，一天 $1-2，屌打 $200/月的方案。」但 jbellis 丟出一句整個討論串裡最重要、卻沒幾個人接的話：「這下成了——DeepSeek 是最後一間『模型值得拿來寫 code、卻沒有 first-party harness 讓它的模型被訓練去適配』的實驗室。」

唯一被一致稱讚的，是「每 run 皆可追溯」。kamranjon 特別點名 Trajectory view：「模型看到的一切——system prompt、reasoning、tool call、subagent 排程、每一次 context 注入——都記錄在 append-only 的 session log 裡，可以按來源檢視、fork、search、replay。」bmurphy1976 的酸最到位：「Tracing what it actually did. 誰會想到這是個好主意——而不是試圖把一切都藏起來。」至於為什麼用 node.js/TypeScript，這是全串最長的一條 thread：正方說 async、跨平台、npm 讓插件分發零摩擦、LLM 最會寫 JS；反方回「compiled 語言更安全更快，CodeWhale 就是反例」「Python 分發脆弱得要命」「Claude Code 閒置就吃 500MB，Electron 肥到離譜」。

## 城武觀點

### 一、那篇論文是煙霧彈，而且是故意的

先說結論：我不否認 Cordis 論文有 formal 貢獻。但「有 formal 貢獻」跟「這個貢獻對 agent harness 的實際工程價值成正比」是兩回事。slopinthebag 那句「big idea 就是 destructor」粗魯，但方向對——revertible effects 翻譯成工程語言就是「插件卸載時要記得把自己註冊過的東西清乾淨」，這件事 React 的 useEffect cleanup、C++ 的 RAII、Rust 的 Drop、2002 年的 OSGi 全都解決過。把「unify effect context and coeffect context into a single context type」寫成一篇論文，是把一個熟得不能再熟的工程慣例，從新發明成一套代數。

問題不是「他們重造輪子」——HN 每個人都看得出來，說出來不稀奇。真正的問題是：為什麼一個工程團隊要花力氣把一個已知概念寫成論文？因為純 MIT 的 harness 不夠。134K star 的「又一個 Claude Code clone」會在新聞裡活三天；但「實現了時空可組合性程式範式的開源 harness」會被寫成技術突破。開源需要學術重量來換「前沿性」的信用，而論文就是那張信用狀。煙霧彈的作用不是騙過所有人，是讓「質疑」這件事，需要先讀完 20 頁代數才有資格開口。

### 二、真正的戰略：開源是最難攻擊的鎖定

jbellis 那句沒人接的話，才是這整件事的鑰匙。DeepSeek 是「最後一間沒有 first-party harness 讓模型被訓練去適配的實驗室」——這不是技術描述，是戰略缺口。Claude Code、Codex、Gemini CLI、Grok Build，每一間前沿實驗室都有一把自家模型的「專屬操作環境」，而且模型是「被訓練去適配」那把環境的。這代表什麼？代表你拿 DeepSeek 的模型去跑 Claude Code，先天就吃虧——模型的訓練資料裡，壓根沒有 Claude Code 的工具協定。

dsh 的開源，補的就是這個缺口。表面上「送給開發者」，實際是把「默認用 DeepSeek」變成路徑依賴：harness、插件、工作流、社群心智，全都圍著 DeepSeek 的模型長出來。MIT license 是這套鎖定的護城河——llm 那個 seam 在架構上確實能換任何 provider，所以「鎖定」這個指控永遠可以被一句「都開源了、模型都能換，你還要怎樣」擋回去。bdcravens 講的 mono-vendor ecosystem with extra steps，就是預告：官方先塞一堆「官方插件」，用戶因為是官方就信任，最後你「能換」但「不會換」。鎖定的最高境界從來不是「不能換」，是「換的成本高到你不想換」——而 DeepSeek V4 Pro 便宜到讓這件事連經濟動機都沒有。

### 三、唯一真貨「可追溯」，諷刺地成了全行業的罪證

append-only session log 是 dsh 裡唯一我不會酸的東西。但 bmurphy1976 的酸點破了最諷刺的真相：agent 做了什麼，竟然要當成賣點來宣傳——因為其他家全都把過程藏起來。可追溯的真正價值不在除錯，在問責。當 agent 自動寫 code、自動下決策，出事的時候誰負責？append-only log 是唯一能指認「這一步是模型自己加的、不是人加的」的證據。閉源 harness 不是做不到，是不敢給——給了，責任就追得到他們頭上；藏起來，出事才能推給「模型幻覺」。dsh 開源加 append-only，讓記錄可以被獨立驗證。這才是「開源」唯一不虛的部分，而它的價值恰恰反過來證明：其他家的「不開源」是刻意的。

*城武的未解檔案——開源不是答案，它只是讓「鎖定」這個詞變得很難被指認的那層糖衣。*

- 原文：[DeepSeek Harness developer preview](https://news.ycombinator.com/item?id=49285244)（bjin, Hacker News, 2026-08-13）
