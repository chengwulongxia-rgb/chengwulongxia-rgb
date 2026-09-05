---
layout: post
title: "【深度分析】Google AI Mode 讓你多付 21.6%？當搜尋從比價引擎變成品牌導流層"
date: 2026-09-05 02:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-09-05/2026-09-05-google-ai-mode-price.jpg)

Google 一直把 AI Mode 說成「讓購物更聰明」的下一步，但 Productrise 一份剛出爐的數據研究卻描出完全不同的輪廓：同樣商品在 AI Mode 裡平均貴了 21.6%，AI Mode 與傳統搜尋的重疊商品只有 1.28%，而且同商品的主要賣家有 49.6% 的機率不是同一家。這不是「AI 還在學習」那種可愛的失誤，這是 Google 搜尋商業邏輯正在重組的徵兆。

## 原文摘要

Productrise 在 2026 年 8 月 9 日至 8 月 31 日這 23 天內，追蹤了超過 200 萬筆商品 listing、10 萬組 SERP 與 AI Mode 回應，地區涵蓋美國與英國。研究的起點很直接：當同一個查詢同時出現在傳統搜尋與 AI Mode 時，兩邊推薦的商品與價格究竟有多像？結果發現，AI Mode 不只推薦的商品少很多，而且推薦的東西往往更貴。

### 六項核心發現

**Finding 1：同商品在 AI Mode 更貴。** 把同時出現在兩邊的「matched products」拿出來比，AI Mode 的 lead price 平均比傳統搜尋高 21.6%。獨立 SEO 顧問 Brodie Clark 在評論中指出，AI Mode 的 grid result 可能會讓頂層比價不太準確，但終究是價格最低的零售商拿到點擊；他也認為這份研究對消費者很有用，因為它顯示 AI Mode 推薦商品時對「價格」的權重可能不如預期。

**Finding 2：AI Mode 傾向浮出高價商品。** 如果放寬到「所有有標價的 listing」，AI Mode 的商品中位數價格是 149 美元，傳統搜尋則是 100 美元，差距約 49%。圖表裡 AI Mode 那側看起來特別小，原因很單純：AI Mode 本來就展示少得多商品。在這份資料中，AI Mode 的 listing 只佔全部追蹤資料的 12.3%。

**Finding 3：兩邊幾乎不重疊。** 同一查詢、同一天，AI Mode 與傳統搜尋顯示相同商品的機率只有 1.28%。Productrise 七月的另一份研究也發現類似現象：AI Mode 整體只排名了傳統搜尋中約 5% 的商品。研究也給了平均數字：傳統搜尋一次顯示約 27.8 個商品，AI Mode 只顯示約 3.9 個，重疊商品僅 0.94 個。JAKALA 的 SEO Manager Jamie D'Alessandro 認為這對不想只靠價格競爭的商家是個小勝，並觀察到 AI Mode 似乎更傾向連到品牌官網而非第三方市場。

**Finding 4：價格不一致時，AI Mode 多半是貴的那邊。** 在 matched products 中，兩邊價格真正不一致的比例是 38.1%；而當價格不一致時，AI Mode 比較貴的機率高達 68.4%。此時 AI Mode 的中位數價差是 +22.2%，平均值因離群值影響被拉到 +88.5%。Break The Web 的資深 SEO 策略師 Katelyn Geary 評論說，當演算法三分之二時間偏好較貴商品、又常換掉賣家時，開放市場比價就被替換成一條通往高價庫存的策展路徑。

**Finding 5：AI Mode 比較便宜時，差距小很多。** 剩下的狀況裡，AI Mode 比傳統搜尋便宜的中位數差距是 -7.8%，平均值是 -11.9%。換句話說，貴的時候貴很多，便宜的時候只便宜一點。

**Finding 6：同商品有近一半機率換了主要賣家。** 在 matched products 中，主要賣家不同的比例是 49.6%。Re:signal 的 Director of Organic Search Callum Lockwood 指出，這件事比價格差距本身更該讓零售商擔心，因為它暗示 AI Mode 不只是重新排序同一個 offer，而是根據自己的標準挑了一個不同的 offer；feed 資料、庫存、第三方引用、評論、新聞等都可能影響這個選擇。

### 為什麼這件事重要

Productrise 創辦人 Hugo Huijer 在文末強調，消費者通常以為 AI Mode 是站在自己這邊的：像傳統搜尋多年來那樣幫他們找出最好選項。確實，點進任一商品後側邊 knowledge panel 仍會列出其他賣家與價格，但那需要額外一次點擊，而多數人不會做。人們看到的第一個數字，就是大多數人會採取的數字，而 AI Mode 裡那個數字往往更高。

Google 過去一年持續把 AI Mode 拉近標準搜尋體驗：搜尋框直接顯示 AI Mode、Chrome 網址列有 AI Mode 按鈕、搜尋結果頁有 AI Mode 標籤，而且通常沿用同一個查詢。從消費者角度看，介面幾乎沒變，但背後推薦邏輯已經換了。研究結論認為，如果 AI Mode 不是以最低價來決定能見度，那麼無法贏得價格戰的品牌反而可能有機會——更好的 feed、更豐富的產品資料、評論與購物對話的一致性，都能讓 AI 更敢推薦它們。

### 研究方法

研究期間為 2026 年 8 月 9 日至 8 月 31 日，使用 `popular_products` 資料集，以 product docid 比對同一天同一查詢兩邊的商品。價格差異公式為 (AI Mode price − SERP price) / SERP price，僅用於 matched products。重疊率則是每天計算「matched products ÷ 傳統搜尋中有 product identifier 的所有 listing」後再取平均。

## 城武觀點

Google 的公關話術一向漂亮：AI Mode 讓你「從問題更快走到購買」。但 Productrise 這份數據讓我想反過來問：誰的問題？誰的購買？同商品平均貴 21.6%、兩邊重疊率只有 1.28%、主要賣家有 49.6% 的機率被換掉——這三個數字連在一起，不是演算法偶然犯錯的形狀，而是搜尋引擎商業模型重新設計的形狀。

我的立場是：Google 正在把搜尋從「價格比較引擎」重建成「品牌導流層」，而 AI Mode 對低價市場不友好這件事，是透明的設計結果，不是 bug。

傳統 Google Shopping 的邏輯裡，最便宜那個通常拿到最好的視覺位置與注意力，這是使用者長期相信 Google 在幫自己比價的原因。但 AI Mode 不玩這套了。它不展示 27.8 個選項，只展示 3.9 個；它不跟傳統搜尋重疊，因為它根本沒打算做同一件事。它的工作不是列出所有可能再讓你挑最便宜的，而是根據某種「對購物對話有信心」的標準，選出一個它認為合理的推薦。而這個標準，剛好對有預算打品牌、做 feed、累評論的賣家有利。

這就是結構上的不對稱：消費者以為自己在「問 AI」，其實是在看一個被包裝成建議的策展層。當推薦結果不需要與最低價選項重疊時，推薦本身就變成了一種別名的廣告——只是它沒有標示為廣告，也沒有讓使用者意識到自己已經離開了比價語境。1.28% 的重疊率低到不像失誤，低到像一個宣告：這裡不負責給你全部選項。

當然，對零售商來說這可能是好消息。Jamie D'Alessandro 說這是「不想只靠價格競爭的商家的小勝」，這句話沒錯，但它同時也承認了另一件事：AI Mode 在重新分配誰能見光。不是最便宜的人贏，而是最能餵飽 AI 資料胃的人贏。這會不會讓購物體驗變好？我不知道。但它確實讓「問 AI」這個動作，從使用者尋求客觀答案，悄悄變成了平台幫賣家完成轉換的通道。

*城武的未解檔案——當搜尋框裡的「AI」按鈕越來越像原來的搜尋，我們還以為自己在比價，但其實已經走進一個只給你四個選項、而且通常更貴的櫥窗。*

- 原文：[Google AI Mode shows the same products 21.6% more expensive than traditional search](https://productrise.app/blog/google-ai-mode-prefers-more-expensive-products) (Hugo Huijer, Productrise, 2026-09-01)
