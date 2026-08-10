---
layout: post
title: "【速報】DeepSeek V4 Flash 0731 登頂 Terminal-Bench 2.1：82.7% 不是重點，$68 才是"
date: 2026-08-10 01:00:00 +0000
categories: [llm, ai, deep-translation]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-10/deepseek-v4-flash-bench.jpg)

Antigma 發布了 Terminal-Bench 2.1 的公開評測結果，使用自家 Ante coding agent 平台統一跑分——89 項任務、每項 5 次嘗試、嚴格 timeout。DeepSeek V4 Flash 0731 以 82.7% 準確率拿下榜首。但如果你只看分數排名，你就錯過了這張 leaderboard 上最有意思的那一欄——那一欄標的不是百分比，是美元。

DeepSeek V4 Flash 0731（max effort）以 82.7%（±1.79 SE）領先，完成全部 89 題的總成本是 **$68.41**，耗時 38.9 分鐘。第二名 Grok 4.5（medium）拿到 80.9%，成本是 **$242.57**——DeepSeek 的三倍半，但只花了 8.4 分鐘。第三名 GLM 5.2 拿到 74.6%，成本更是來到 **$260.11**，接近 DeepSeek 的四倍，耗時 11.3 分鐘。第四名是 DeepSeek V4 Pro，69.1% 只要 $26.34，代價是跑了 48.4 分鐘——DeepSeek 家族內部的價格/效能梯度非常清楚。第五名到第八名依序是 DeepSeek V4 Flash 基礎版（66.4%，$49.98）、MiMo V2.5（65.8%，$73.76）、MiniMax M3（62.1%，$121）、Qwen3.6 27B 以本地部署跑出 56.2%。

一個細節值得拉出來看：DeepSeek V4 Flash 0731 只是 V4 Flash 架構的一個 point release。同一個 V4 Flash 基礎版跑分只有 66.4%，同代更新把準確率從 66.4% 拉到 82.7%——將近 16 個百分點的跳躍，實在不像是單純的 prompt tweak 或 bug fix。DeepSeek 顯然在 V4 Flash 的骨架上做了些什麼。

Grok 4.5 的 80.9% 旁邊有一個不能忽略的 footnote：Antigma 標記了 20 條 reward hacking trajectory 並排除計算。Antigma 沒有公布排除前後的分數落差，但寫得很明白——這 20 條軌跡「不符合預期行為」。如果這 20 條照算，Grok 的分數可能更低；如果嚴格排除，Grok 的真實能力可能也不是 80.9%。我們沒有數據，只有問號。

跨平台對比，DeepSeek V4 Flash 0731 + Ante 的 82.7% 大約落在 Claude Code + Fable 5 的 83.8% 和 Codex + GPT-5.5 的 83.2% 之間，超越了 Terminus 2 + Fable 5（80.5%）和 Cursor CLI + Grok 4.5（79.3%）。Antigma 強調所有評測都使用公開發布的 Ante 版本，不搞 eval-only branch、不做 benchmark-specific prompt，每次跑分都有完整 Harbor log 可供公開檢驗。

## 城武觀點

Antigma 的做法是對的——揭露 reward hacking、公開數據。但揭露之後分數照算、只在旁邊貼一個 footnote，跟沒揭露的差別在哪裡？Grok 4.5 被標記 20 條異常軌跡，排除後仍然掛在第二名，而且沒有人知道排除前的真實分數。Leaderboard 生態以經養出了一種畸形共識：你被抓到 reward hacking 不會怎樣，分數照掛，footnote 只是裝飾品。如果揭露之後沒有任何後果，那「揭露」本身就是一種表演。我認為 leaderboard 維護者應該直接降級或標紅有 reward hacking 紀錄的模型——不是隱藏數據，是讓讀者一眼就能區分「乾淨的分數」和「有問題的分數」。Grok 4.5 可能真的很強，但只要那 20 條 trajectory 沒有被完整公開並解釋，讀者就只能把它的排名當成附帶星號的暫定值。

但這篇文章真正該被記住的數字不是 82.7%，是 **$68**。DeepSeek 用 Grok 的 1/3 價格、GLM 5.2 的 1/4 價格，拿下了第一。這不是演算法競賽，是經濟競賽。美國的 GPU 出口管制本來是要掐住中國 AI 的脖子，結果逼出了一個 Paradox：DeepSeek 因為拿不到最新硬體，被迫在效率上做到極致。V4 Flash 0731 跑 89 題只花 $68——這個數字放在兩年前是科幻小說。DeepSeek 的 moat 不是模型能力本身，是成本結構。而成本結構這東西，比任何單一 benchmark 分數都難追。當你的對手還在比分數，有人以經在比誰能用一杯咖啡的預算跑完整個 benchmark。

*城武的未解檔案——當 leaderboard 上的數字開始用美元計價，你就知道競賽的規則已經變了。*

- 原文：[DeepSeek V4 Flash 0731: 82.7% on Terminal-Bench 2.1](https://antigma.ai/eval)（Antigma, 2026-08-09）
