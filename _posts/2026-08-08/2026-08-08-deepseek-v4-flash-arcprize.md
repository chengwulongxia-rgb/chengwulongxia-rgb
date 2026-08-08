---
layout: post
title: "【深度分析】DeepSeek V4 Flash 登上 ARC-AGI 排行榜：89% 不是重點，15.4 個百分點的懸崖才是"
date: 2026-08-08 01:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-08/deepseek-v4-flash-arcprize.jpg)

DeepSeek 的「Flash」系列向來以性價比著稱，但這次 V4 Flash 0731 在 ARC Prize 的成績單上，藏著一個比表面數字更值得追問的問題：當一個 benchmark 的設計初衷是對計算量免疫，而數據卻說它比舊版 benchmark 對計算量敏感十倍的時候——到底是模型在進步，還是 benchmark 在退步？

## 原文摘要

2026 年 7 月 31 日，DeepSeek 釋出 V4 Flash 模型，並在 ARC Prize 的 ARC-AGI 基準測試上取得正式成績。ARC Prize 將此模型列為「Verified」（驗證通過），附有技術論文（arxiv.org/abs/2606.19348）及 HuggingFace 模型權重。

### 三個推理變體的核心數據

DeepSeek V4 Flash 0731 提供三個推理強度等級，對應不同的計算量配置：

| 變體 | ARC-AGI-1 | ARC-AGI-2 | 每題成本 |
|------|-----------|-----------|----------|
| Max  | 89.0%     | 61.4%     | $0.02–$0.04 |
| High | 87.0%     | 56.0%     | — |
| Low  | 84.0%     | 46.0%     | — |

ARC-AGI-1 使用 Semi-Private 評估集（400 題 Public Eval），ARC-AGI-2 同樣使用 Semi-Private 評估集（120 題 Public Eval），兩者皆非 Public Eval subset——ARC Prize 的 Semi-Private 機制旨在防止針對公開題目的過度擬合。模型尚未在 ARC-AGI-3 上進行評估。

### 從 Max 到 Low：一個不對稱的崩塌

最關鍵的數據不在絕對分數，而在三個變體之間的**落差結構**。

ARC-AGI-1 從 Max（89.0%）降到 Low（84.0%）只掉了 5 個百分點——三種計算量下的表現幾乎是平的。這符合 François Chollet 最初設計 ARC 的核心理念：這些題目測試的是抽象推理能力，理論上不該對計算資源高度敏感。

但 ARC-AGI-2 完全是另一回事。從 Max（61.4%）降到 Low（46.0%），一口氣掉了 15.4 個百分點。High（56.0%）夾在中間，呈現一個近乎線性的崩塌——每降一級計算量，就掉 5 個百分點以上。

ARC Prize 在頁面上公開了 ARC-AGI-2 Public Eval 全部 120 題的逐題通過/失敗矩陣。Max 設定下通過的題目（如 `0934a4d8`、`13e47133`、`16b78196`），在 Low 設定下大量轉為失敗。表格中可以觀察到一個清晰的模式：Max 下為 ✓ 的題目，High 還有一定比例保留，但到 Low 就大量潰堤。典型案例如 `195c6913`（Max ✓, High ✗, Low ✗）、`28a6681f`（Max ✓, High ✗, Low ✗）、`3e6067c3`（Max ✓, High ✗, Low ✗）。有少數題目甚至呈現不連續的翻轉——如 `4c3d4a41`（Max ✗, High ✓, Low ✓），顯示計算量與正確率之間並非單調關係。

### 與其他模型的對比

在 ARC-AGI-2 排行榜上，DeepSeek V4 Flash 0731 的 61.4%（Max）與 Claude Opus 5 的 ~60% 處於同一級別。但成本面懸殊：Claude Opus 5 屬於 Anthropic 的最頂級模型，推論成本遠高於 DeepSeek V4 Flash 的 $0.02–$0.04/題。排行榜上的其他近期模型——Gemini 3.5 Flash-Lite、GPT-5.6 Luna、Claude Fable 5——提供了更廣泛的性價比光譜對照。

### 技術背景

DeepSeek V4 Flash 是 DeepSeek V4 系列的輕量推理變體。「Flash」在 DeepSeek 的命名體系中代表低延遲、低成本推論，通常透過模型蒸餾與推論優化實現。技術細節見於伴隨發布的 arxiv 論文（2606.19348），模型權重已在 HuggingFace 開放下載。

## 城武觀點

### 一、ARC-AGI-2 的 15.4 百分點懸崖，暴露了它自己

Chollet 設計 ARC 的核心前提是：真正的推理能力應該對 scale 免疫。一個靠更多計算量硬解的模型，不算真正「理解」了題目。這是 ARC 跟 ImageNet、MMLU 最根本的區別——後者從不假裝自己測的是比「更多參數、更多數據」更深的東西。

ARC-AGI-1 大致守住了這條線。三個計算等級只差 5 個百分點——你可以說它在 ARC-AGI-1 上的推理是相對「乾淨」的。

但 ARC-AGI-2 的 15.4 百分點落差完全是反向證據。這不是邊際效益遞減，這是計算量當主力引擎。每降一級計算預算就穩定掉 5 個百分點以上——這種線性關係在「推理 benchmark」上出現時，你測的已經不是推理，是「你在推論階段花了多少錢」。

ARC Prize 團隊可能會說：這正好證明 ARC-AGI-2 更難，所以更敏感。但「更難」和「對計算量更敏感」是兩個命題。一個真正測推理的 benchmark，變難應該是題目需要更深層的抽象操作，而不是更多次的 token generation。ARC-AGI-2 的數據暗示後者比重遠大於前者。

弔詭的是：ARC Prize 讓 ARC-AGI-2 更難、更不容易被破解的努力——可能恰好讓它變成了更純粹的計算量測試。當難度推到極限，剩下的差異化因子就只是誰的推論預算更大了。數據不會說謊。

### 二、當「Flash」追上了「Frontier」，標籤就只是帳單

DeepSeek 把這模型叫「Flash」——便宜、快速、輕量。但它在 ARC-AGI-2 上跟 Claude Opus 5 差距不到 2 個百分點，成本可能是 1/50。如果這條趨勢持續——那「前沿」（frontier）就會從能力標籤，變成純粹的支出報表。

前沿模型的真實定義正在從「誰最聰明」滑向「誰在單次推論上燒最多錢」。當 GPT-5.6 Sol Max 跟 DeepSeek V4 Flash 的能力差距縮到個位數百分點以內、成本差了一個數量級——「我們是前沿」那張價格標籤，本質上只是早期採用者的奢侈稅。

DeepSeek 在 ARC-AGI-2 上的成績不是「追趕者終於達標」的故事，是「價格訊號正在瓦解能力敘事」的故事。只看分數不看成本，你不會注意到 DeepSeek 跟 OpenAI / Anthropic 的本質差異。加上成本維度，那個差異是結構性的。

我賭六個月內會看到第一篇嚴肅分析，標題長這樣：「Frontier AI is dead — long live cost-efficient AI」。到那時候回頭看這張排行榜，15.4 個百分點的懸崖和 $0.02/題的數字，會比 89% 更值得被記住。

*城武的未解檔案——如果「前沿」的定義是「誰花最多錢」，那最前沿的模型是你鄰居用八張 H100 跑了一整夜的 GPT-5.6 Sol Max。而 DeepSeek 用一杯咖啡的預算，站在離它不到 2 個百分點的地方。*

- 原文：[DeepSeek V4 Flash 0731 - ARC-AGI Results](https://arcprize.org/results/deepseek-v4-flash-0731)（ARC Prize, 2026-07-31）
