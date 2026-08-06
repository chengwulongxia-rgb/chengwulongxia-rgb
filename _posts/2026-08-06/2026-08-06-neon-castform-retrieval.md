---
layout: post
title: "【深度分析】4B 開源模型如何在檢索任務上擊敗 GPT-5.6 Sol，成本僅 1/100"
date: 2026-08-06 01:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-06/neon-castform-retrieval.jpg)

一篇 Neon 官方部落格文章，聽起來像產品發表——但它真正在說的事情比產品大得多：frontier model 的護城河，正在從「模型比較聰明」轉移到「誰有 RL 訓練基礎設施」。而 Castform 正在把這條護城河拆掉。

## 原文摘要

Neon 與 Castform 聯手發表了一組實驗結果：一顆 4B 參數的開源模型，經過 Castform 的 RL post-training 後，在檢索任務上的準確率與 GPT-5.6 Sol 持平，但每次請求的成本只要後者的 1/100。

Castform 共同創辦人 Ying Hang Seah 一語道破核心問題：「大多數團隊最好的訓練資料就躺在他們的資料庫裡。真正的困難是把原始資料變成可用的東西，以及讓 agent 能便宜、大規模地讀取、搜尋、修改資料。把 Castform 指向 Neon，這兩個問題一起解決。」

一個好的 agent 需要兩件事同時到位：Context——能不能提供工具找到正確的資料；Model——模型能不能判斷該搜什麼。Neon 的 Lakebase Postgres 和新推出的 Search 擴充解決前者，Castform 解決後者。

回顧檢索技術的演進：2022 年前後，整個產業押注 embedding search，每家資料庫廠商都在加向量搜尋功能，pgvector 曾是 Neon 下載量最高的擴充。到了 2025 年，agent 開始成熟，開發者改做 multi-hop 搜尋工作流——把大問題拆成小問題，模型在迴圈中反覆規劃、搜尋。但每次迴圈迭代就等於再呼叫一次 frontier model。一個典型的多輪搜尋請求，用 GPT-5.6 Sol 要超過 10 秒、每次花費約 $0.03，又慢又貴。

小開源模型便宜 100 倍，但裸模型的能力跟閉源 API 有落差。RL post-training 就是用來補這個落差的：在特定任務（例如搜尋）上，post-trained 的開源模型可以追上甚至超越 frontier model，每次請求成本卻低好幾個數量級。這正是 Castform 的目標——讓開發者能做 RL post-training，但不用碰機器學習和 GPU 基礎設施，理想狀態是「讓 post-training 像 prompt engineering 一樣簡單」。

Castform 的訓練管線跑在 Neon 的 Lakebase Search 上。進行 RL post-training 需要三樣東西：一個任務（例如回答使用者問題）、一個 agent 可以執行的環境（例如搜尋工具）、一個 reward function（例如答案是否正確）。三者到位後，RL 就是試錯迴圈：模型嘗試任務、reward function 評分、回饋訊號引導模型往最佳表現爬坡。

問題是，多數公司根本沒有一組整理好的任務和 reward function 可以丟進 post-training。但企業確實有大量專有資料：內部文件、產品記錄、支援文章、客戶互動記錄、wiki、營運資料庫。這些資料裡有 agent 需要的知識，但把它們轉成有效訓練資料通常需要大量資料工程和人工標註，導致許多團隊直接用兩個理由放棄 post-training：「我們沒有訓練資料」、「微調太難，我們沒有基礎設施」。

Castform 同時解決這兩點。它把現有的語料庫轉成訓練任務，然後管理 RL 迴圈，教開源模型如何有效使用那些資料。實際使用流程：從你的資料中抽取文件原文和 ground truth，合成對應問題，產出 QA 資料集。接著你指定 agent 能存取的工具，以及一個 reward function——在 Neon 的案例中，reward 由 retrieval（是否找到正確來源）、citation（是否引用正確段落）、correctness（最終答案是否正確）三項加總構成，程式碼不到十行。

Castform 提供完整的訓練可觀測性：你可以追蹤 reward 隨 step 爬升的曲線，也可以深入個別任務看模型在質化上表現如何，用來除錯工具失敗或 reward hacking 等問題。

為什麼搭配 Neon？訓練過程中，agent 會反覆呼叫 Lakebase Search，跨數千個平行 rollout 產生極度 bursty 的工作負載。Neon 的動態運算縮放能吸收這些峰值，不需要為最大容量全天候佈建。更進一步，當 agent 不只是搜尋而開始修改資料時，訓練 stateful agent 需要隔離環境，Neon 的 branching 可以給每個 rollout 獨立的資料庫狀態，time-travel queries 則可以重建、檢視 agent 當時遇到的資料狀態。搭配 autoscaling 和 scale-to-zero，這指向一條訓練數千個 stateful agent rollout、卻不需要維護數千個常駐環境的路。

## 城武觀點

這篇文章最尖銳的一句話不是 benchmark 數字，而是 Ying Hang Seah 那句：「你最好的訓練資料就躺在你的資料庫裡。」

這句話把 frontier labs 的護城河拆穿了。外界一直以為 OpenAI、Anthropic 的優勢在於「他們的模型比較聰明」——但真正的護城河從來不是模型架構，而是他們把 RL training loop 當成內部機密。資料怎麼清洗、reward 怎麼設計、rollout 怎麼管理、基礎設施怎麼建——這些才是 frontier labs 真正不會寫在論文裡的東西。Castform 把這條管線商品化，意味著護城河正在從「誰的模型聰明」轉移到「誰擁有資料，誰有訓練基礎設施」。這對企業是好事——你的專有資料終於可以變成真正的競爭優勢，而不是永遠等著被 GPT-6 一口吞掉。Castform 的立場是對的：企業 AI 落地的瓶頸從來不是模型能力不夠，而是 RL training loop 的基礎設施不存在。

但這裡有一個必須追問的方向。「讓 post-training 像 prompt engineering 一樣簡單」——這句話聽起來很美，但正是最危險的行銷話術。文章展示的 reward function 只有三行：retrieval + citation + correctness，看起來簡單到像 prompt engineering。問題是，真實企業場景的 reward 設計遠比這複雜。什麼叫「正確答案」？客服場景中，「正確」可能是「使用者滿意」而非「資訊準確」。什麼叫「正確引用」？當答案需要綜合三個不同來源的片段時，citation 的粒度該怎麼定義？reward hacking 是一個已經被大量文獻證實的真實問題——模型會找到 reward function 的漏洞，產出形式上高分但實際上無用的答案。Castform 解決了訓練管線的問題，但 reward engineering 本身會不會變成下一個瓶頸？我的判斷是：會。而且這個瓶頸比訓練管線更難商品化——因為 reward 的定義跟你的業務邏輯綁在一起，沒有通用的解法。

這也意味著，frontier labs 的護城河並沒有「消失」，只是換了形狀。以前是「我們有最好的模型」，以後會是「我們有規模最大的 RL 基礎設施和經驗最豐富的 reward engineering 團隊」。Castform 把基礎設施端打開了，但 reward 設計的 know-how 是一條新的護城河——而且是開源社群最不擅長的那種：它不是程式碼，是對特定領域的理解。

*城武的未解檔案——frontier labs 花了三年把「我們模型最聰明」的敘事打進市場，現在 Castform 用一顆 4B 模型證明：你們真正值錢的東西，是以經在企業資料庫裡躺了好幾年的那些 PDF。*

- 原文：[How Castform + Neon Beats Frontier Models on Price and Efficiency](https://neon.com/blog/how-castform-neon-beats-frontier-models-on-price-and-efficiency)（Angel Pan, Neon, 2026-08-05）
