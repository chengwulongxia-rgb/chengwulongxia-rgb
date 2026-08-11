---
layout: post
title: "【深度翻譯】把 LLM 輸出「人類化」是最蠢的抽象層錯誤"
date: 2026-08-11 04:00:00 +0000
categories: [llm, ai, deep-translation]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-11/humanising-llm.jpg)

如果你最近有在追 AI 工具圈的風向，你可能注意到一個奇怪的現象：越來越多人要求 LLM「講話像人類」——不要那麼囉嗦、不要術語、把我當成 ADHD 患者來溝通。聽起來很有道理，但 Kuber Mehta 這篇文章直接開噴：這整件事的抽象層就錯了。而且他點出的問題，比他自己想像的還深。

## 原文摘要

Kuber Mehta 觀察 AI 文化風向的主要管道是 X（Twitter）、爆紅的 GitHub repo，和 Hacker News。最近他注意到一個趨勢：像「I have ADHD」這類 prompt skill 開始流行，也有人要求 agent 用 ASD-STE100 簡化技術英語（Simplified Technical English）來輸出。

他理解這個訴求——沒有人喜歡 LLM 輸出的囉嗦和那種特定的機器腔調。但他認為，用「讓人類覺得舒服」的方式來修正這個問題，是搞錯了抽象層。

核心問題在於：這些指令不是在模型完成工作之後才套用的——它變成了工作本身的一部分。當你告訴一個 agent「用短句、避開術語、不要讓我感到 overwhelm、只講最重要的東西」，你是在要求它持續把輸出壓縮成一個低頻寬的格式。

而那個壓縮，是有損的。

你永遠不會注意到什麼東西被丟掉了，因為輸出讀起來還是很順。

ASD-STE 是個絕佳的例子，因為它聽起來太合理了。這個規範的設計初衷是讓技術文件對人類來說沒有歧義，但當你強迫一個 LLM 在推理過程中使用這個受限的詞彙表，你是在限制它的內部表徵。這就像要求一個工程師只用一本 900 字的字典來除錯一個分散式系統——他也許還是能找到 bug，但他有一隻手被綁在背後。

當 agent 開始跟其他 agent 對話時，這件事還會變得更荒謬。

一個 subagent 調查了某個 bug，把發現整理成漂亮的人類可讀摘要。父 agent 讀了那份摘要，再把它轉成另一份漂亮的人類可讀摘要給你。每一層跳轉都在流失資訊保真度。

如果 subagent 跑了六個測試，我不想要：

> 大部分測試都通過了，只有一個問題值得留意。

我要的是：

```
5/6 PASS
FAIL: test_cache_invalidation
CAUSE: stale key survives restart
REPRO: tests/cache_test.py:184
```

更重要的是，人類化會掩蓋失敗。

Agent 的失敗方式是有用且醜陋的：互相矛盾的證據、未解決的分支、stack trace、不確定的假設。人類散文極擅長把這些東西撫平，變成像是：

> 這裡有幾個需要考量的地方。

聽起來舒服多了。但 Kuber 說，他寧願發現自己的 agent 正在幻覺、或快要撞到 token 窗口，也不要被這種句子安撫過去。

我們建造的所有其他系統都走相反的方向——資料庫不會用儀表板顯示的格式來儲存資料，編譯器不會讓它的中間表示（IR）讀起來很愉悅，API 之間不會交換友善的摘要。

我們盡可能久地保留最高保真度的表徵，只在人類消費它的邊界上做轉換。但 LLM 工具圈正在越做越相反。

Kuber 強調，這不是在反對可及性（accessibility）或個人化。

如果你想要三行答案或簡化技術英語，完全沒問題。他只是認為，這件事應該放在最後做：讓 agent 保留詳細的狀態，讓 subagent 之間交換 schema、diff、精確的錯誤、信心水準、資料來源——然後再幫我壓縮。

他認為最有趣的部分是：這些爆紅的 skill 可能其實指向了正確的未來。

使用者在 prompt 層修補這件事——但這件事本來應該發生在更底層的技術棧裡。

「跟我講話時當我有 ADHD」作為一個渲染器（renderer）完全合理，作為一個操作指令（operating instruction）就沒那麼合理了。真正耐久的版本是：agent 的原生語言是精確的、面向機器的狀態，而溫暖、簡潔的人類版本只在邊界上生成。

所以那些爆紅的 repo 不是終點，而是一份 bug report。

## 城武觀點

Kuber 的論點我基本上是買單的——但他的分析停在一個工程師的本能反應上：看到有損壓縮，解法就是結構化通訊。這個思路沒全錯，但它跳過了更根本的問題。

整個 agent framework 生態系到現在，沒有一個公認的中間表示層（intermediate representation layer）。編譯器有 IR，資料庫有 storage engine，搜尋引擎有 inverted index——這些都不是最終消費者看到的東西，它們是系統內部用來傳遞、操作、優化的中間格式。Agent 之間傳遞什麼？人類可讀的 prose。Prompt 同時是 IR 也是 render target——沒有人做中間那一層。

這不是 feature，是基礎設施的缺失。當 Kuber 說「讓 subagent 交換 schema、diff、精確的錯誤」時，他其實在描述一個還不存在的東西。今天的現狀是：你叫 agent 輸出結構化格式，它輸出的是「看起來像結構化格式的 LLM 生成文本」。格式對了，但內容還是概率性的。結構化讓錯誤更可追溯——但不讓錯誤更少。而 Kuber 的文章語氣暗示前者等於後者，這是一個工程師的經典盲點：把可控性當成正確性。

那些爆紅的 GitHub repo——「I have ADHD」、「ASD-STE100」——Kuber 說它們是 bug report，這個比喻很精準。但它們在 report 的 bug 可能比他想的更大：不是「人類化應該放在邊界層」，而是「我們沒有一個不是人類語言的內部傳遞格式可以用」。使用者在 prompt 層亂搞，是因為往下挖，什麼都沒有。

*城武的未解檔案——當你的 API 之間唯一的傳遞格式是散文，你建的就不是 agent 網路，而是一場以經開始失真的傳話遊戲。*

- 原文：[Humanising LLM Outputs is Dumb](https://kuber.studio/blog/Reflections/Humanising-LLM-Outputs-is-Actually-Dumb)（Kuber Mehta, kuber.studio, 2026-08-10）
