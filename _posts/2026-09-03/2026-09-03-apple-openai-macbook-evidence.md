---
layout: post
title: "【深度翻譯】Apple 控告 OpenAI 案：前工程師 MacBook 的「驚人證據」"
date: 2026-09-03 14:00:00 +0000
categories: [llm, ai, deep-translation]
---
![hero]({{ site.baseurl }}/assets/images/2026-09-03/apple-openai-macbook-evidence.jpg)

Apple 控告 OpenAI 的營業秘密案又添新進展。這次蘋果端出的不是抽象指控，而是一台實體 MacBook——前工程師 Chang Liu 離職後使用的筆電，以經初步法證檢查後，被指稱藏有「驚人證據」。但這份文件真正值得注意的，不是證據本身，而是蘋果埋在裡頭的法律論點：營業秘密一旦被餵進會學習的 AI agent，其學習「可能產生**不可逆且持續蔓延的使用**」。這句話才是看懂整場官司的鑰匙，也是城武今天想跟你拆解的靶心。

## 原文摘要

蘋果在對 OpenAI 的訴訟中又提交了一份新文件，持續推動「加速證據開示」（expedited discovery）。在今天這份文件裡，蘋果宣稱，根據對前工程師 Chang Liu 所用筆電的初步法證檢查，他們得到了新的「驚人證據」。

蘋果起初在七月提起這起訴訟，指控多名前員工竊取營業秘密以利於 OpenAI。幾週後，蘋果聲請了初步禁制令（preliminary injunction），同時請求法院加速證據開示，包括提早進行文件提出，以及對 OpenAI 關鍵員工和高層進行證詞調取（deposition）。上週蘋果再次強調加速證據開示的必要性。OpenAI 則主張本案應予駁回。

正如先前報導，Chang Liu 是蘋果訴訟中被點名的當事人之一。Liu 曾是蘋果的資深系統電力工程師（senior system electrical engineer），一月離職轉往 OpenAI。訴訟指控 Liu 在離職後利用一個安全漏洞，下載了機密的工程文件。

在今天的文件裡，蘋果表示 Liu 的律師近日交出了他離職後使用的那台 MacBook。蘋果指出，對該 MacBook 的「初步法證分析」（initial forensic analysis）揭露了四件事，以下引述蘋果文件原文：

> 1. Liu 不僅下載了蘋果機密的電路示意圖（circuit schematic），還把它用在自己於 OpenAI 的工作中；
> 2. 相對於他對蘋果第三方雲端儲存的未授權存取遠非不知情，Liu 本人與 OpenAI 的其他人都很清楚這項存取；
> 3. Liu 得知蘋果正在內部調查他之後，向一位 OpenAI 同事發送了銷毀證據的指示，而該同事確認她會照辦；
> 4. Liu 在 OpenAI 工作時使用的一個工具，名稱與蘋果內部一個用於開發工作的工程應用程式相同。

蘋果指控，Liu 在三月時利用 LTspice（一款電力工程工具）以這份電路示意圖檔案執行了一次模擬。在約莫同時期的訊息中，Liu 說他的 AI「agent」學會了執行 LTspice 並檢視結果。

蘋果主張，當營業秘密資訊被餵進一個會從中學習的 AI agent 或模型時，該學習「可能產生不可逆且持續蔓延的營業秘密使用」（may create irreversible and continually propagating uses of the trade secret）。

蘋果表示，被告方決定不去檢查這台筆電，反而選擇「提出一些會被該裝置中的資料所推翻的理論」。此外，蘋果得知 Liu 使用這份示意圖，是因為他是在一台 Mac mini 上使用它，而 Mac mini 後來透過 iCloud 同步，把資料帶到了他從蘋果帶走的那台 MacBook 上。蘋果現在也要求能取得那台 Mac mini。

蘋果表示，這份「驚人證據」是本案中又一個支持加速證據開示的理由——同時引用了自家文件裡的說法：

> 「這份驚人證據——被告在明知可取得的情況下仍選擇忽視——證明了蘋果為何迫切需要加速證據開示。MacBook 代表了被告迄今所提供的那極其有限的資訊（且還是在延宕數週之後），並顯示蘋果並非在進行『釣魚式調查』（fishing expeditions），而是其營業秘密正遭使用、證據正被銷毀。」

全文文件編號：Apple Inc. v. Liu et al., Case 3:26-cv-07078, Doc. 94-1。

## 城武觀點

蘋果這份文件真正新穎的，不是那四條『證據』，而是那句法律論點：營業秘密一旦餵進會學習的模型，其學習可能產生不可逆且持續蔓延的使用。我的賭注是——真正定義『如何把已學習的營業秘密從模型中刪除』的，會是訴訟槓桿，不是工程思微。人類至今沒有真正能移除已學習資料的技術——machine unlearning 只是近似逼近，騙得過人，騙不過權力。預測：當官司走到禁制令與救濟階段，法院被迫要定義『模型清除』是什麼，而這個定義會被雙方律師的槓桿決定，不是被工程事實決定。逆諷刺一句帶過：主張資料進模型就不可逆的蘋果，恰恰是靠自家 iCloud 垂直棧、靠 Mac mini 同步握到整份定罪材料的——主張資料不可刪的那一方，握的正是全場最強的監控與刪除能力。

*城武的未解檔案——下次再有人喊『把資料從模型裡刪掉』，先問他一句：你手上握得到那台定罪的 MacBook 嗎？*

- 原文：[Apple reveals 'shocking evidence' from ex-employee's MacBook in OpenAI suit](https://9to5mac.com/2026/08/31/apple-openai-forensic-macbook-evidence/)（Chance Miller, 9to5Mac, 2026-08-31）