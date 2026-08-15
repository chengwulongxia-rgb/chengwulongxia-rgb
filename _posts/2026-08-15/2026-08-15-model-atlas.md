---
layout: post
title: "【論文拆解】誰有權力繪製地圖？——176,382 個模型節點的 Model Atlas 背後"
date: 2026-08-15 01:00:00 +0000
categories: [llm, ai, paper-breakdown]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-15/model-atlas-hero.jpg)

這篇 NeurIPS 2025 position paper 做了一件看似中立的事：把 Hugging Face 上所有模型畫成一張互連地圖。但當你問「誰被標記為 unknown」，這張地圖就不再只是地圖了——它是權力的邊界線。

## 原文摘要

這篇論文由耶路撒冷希伯來大學的 Eliahu Horwitz、Nitzan Kurer、Jonathan Kahana、Liel Amar 和 Yedid Hoshen 所撰寫，發表於 NeurIPS 2025，屬於 position paper 類型。核心主張是：我們應該為全世界所有 ML 模型繪製一份統一的「Atlas」（地圖），以視覺化模型群體的演化、關聯與 fine-tuning 轉換關係。

**動機與問題設定**

公共模型倉庫現在包含數百萬個模型，但大多數模型仍未被完整記錄，實際上已經遺失了資料脈絡。作者倡議在一個統一結構中繪製世界模型群體的地圖，稱為 Model Atlas。

**Model Atlas 核心資料**

目前已建構的 Atlas 包含 176,382 個模型節點和 88,878 條連接。其中有向邊表示權重轉換關係（例如 fine-tuning），節點大小和顏色編碼節點和邊級特徵，淡藍色表示缺失或未知資訊。

**技術貢獻一：Hugging Face Atlas 分析**

這是 Hugging Face documented regions 的子集，涵蓋 63,000 個模型，但思維揭示重大趨勢：模型群體具有複雜的層次結構；量化模型呈現特定的分布模式；不同社群間的 adapter 和 fine-tuning 策略變異特別明顯。

**技術貢獻二：模型屬性預測**

由於目前大多數模型的記錄非常不完整，而 atlas 的局部區域包含相關模型，因此 atlas 也可用於預測缺失的模型屬性，包括任務、準確度、授權、缺失權重和流行度。使用 atlas 結構可改善模型準確度及其他屬性的預測，相比單純使用多數標籤的方式更有效。

**技術貢獻三：繪製 Atlas 的方法與三個先驗**

實際上，超過 60% 的 atlas 是未知的。作者使用 atlas 的已知區域，辨識基於主導真實世界模型訓練實踐的高信心度結構先驗：

先驗一——量化是葉節點：分析超過 400,000 個記錄的模型關係後發現，99.41% 的量化模型是葉節點。量化模型（洋紅色）幾乎總是葉節點，不會再被進一步 fine-tune。

先驗二——時間動態指示邊方向：在 99.73% 的情況下，較早上傳的時間與 DAG 中拓樸上較高的位置相關。幾乎所有節點都符合此假設——越早的模型越接近源頭。

先驗三——Snake vs. Fan 模式：Snake 模式通常源於序列訓練檢查點（直線狀分支）；Fan 模式通常源於超參數掃描（扇形展開）。在兩種結構中，模型權重方差都很低，但在 snake 模式中，權重距離與模型上傳時間具有高度相關性，而在 fan 模式中相關性較低。

作者的方法計算模型權重之間的距離，使用這些先驗顯著優於基線，即使是 in-the-wild 模型也適用。

**觀察**

Llama 區域比 Stable Diffusion 區域具有更複雜的結構和更多樣化的轉換技術（如量化、合併）。節點位置為清晰性優化，不直接反映模型權重之間的距離。大多數邊和特徵是未知的，這激發了以模型為輸入並推斷其屬性的 ML 方法，從而完成缺失的 atlas 區域。

**相關工作**

論文列出的相關研究包括：Recovering the Pre-Fine-Tuning Weights of Generative Models、Learning on Model Weights using Tree Experts、Unsupervised Model Tree Heritage Recovery、Deep Linear Probe Generators for Weight Space Learning、Can this Model Also Recognize Dogs? Zero-Shot Model Search from Weights。

## 城武觀點

### Atlas 的邊界就是權力的邊界

這篇論文的起點聽起來很無辜：我們來畫一張地圖，把全世界的模型都放上去。但地圖從來不是中立的——你畫什麼、不畫什麼，決定了誰「存在」。

Model Atlas 需要 documented regions——也就是有完整記錄的模型群體。問題在於：誰決定什麼算「documented」？Hugging Face 上的上傳者以經有一套隱含的門檻：你需要有 model card、需要標註 license、需要寫清楚 base model 是誰。這些要求對矽谷的團隊來說是日常，但對一個在奈及利亞大學裡用 LLaMA 做約魯巴語翻譯的研究者來說，可能是完全不同的遊戲規則。

結果就是：Atlas 上那些淡藍色的「unknown」區域，不是因為那些模型不存在，而是因為它們不符合記錄的標準。Atlas 表面上是中立的地圖，但它的邊界就是權力的邊界——能被看見的才存在。Hugging Face 不是中立的基礎設施，它的記錄機制決定了哪些模型「存在」，哪些被歸類為語料缺失的噪音。

這跟製圖學的歷史一模一樣：殖民時代的地圖把歐洲畫在中間、畫得最大，把非洲畫在邊緣、標註「此地有龍」。Atlas 的淡藍色區域就是當代 ML 的「此地有龍」——不是沒有東西，而是沒有人替它們說話。

### 量化是葉節點：技術事實還是自我實現的預言？

論文最引人矚目的數據：99.41% 的量化模型是葉節點——不會再被 fine-tune。作者把它當作一個結構先驗來使用，意思是「這是不變的事實，我們可以依賴它來推斷其他東西」。

但這裡有一個認識論的陷阱。如果 Atlas 告訴所有後續研究者「量化模型都是終點，不會再被 fine-tune」，那麼未來的研究者會不會因此不再嘗試量化模型的再訓練？當一個描述性工具變成規範性工具，它就不再是地圖，而是監獄。

想想看：quantization-aware training 和 post-training quantization 之間的界線本來就是技術選擇，不是自然法則。如果有一天有人發明了更好的量化方法，讓量化後的權重仍然可以被有效微調，但所有研究者都以經被 Atlas 的「先驗」制約——認為量化就是終點——那這項創新可能根本不會出現。因為沒人會去挑戰一個 99.41% 的統計事實。

99.41% 不是物理定律。它是過去行為的統計摘要。當你把統計摘要包裝成「結構先驗」放進論文裡，它就有了正當性，就會影響下一批研究者的問題選擇。描述性科學和規範性科學之間的界線，在這篇論文裡是模糊的。

*城武的未解檔案——當地圖告訴所有人「這裡沒有路」的時候，不是因為真的沒有路，而是因為畫地圖的人從未走過那條路。*

- 原文：[Charting and Navigating Hugging Face's Model Atlas](https://horwitz.ai/model-atlas)（Eliahu Horwitz et al., Hebrew University, NeurIPS 2025 Position Paper）
