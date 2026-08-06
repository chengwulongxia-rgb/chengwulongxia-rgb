---
layout: post
title: "【深度翻譯】Born Against——為什麼業餘程式社群如此排斥 LLM"
date: 2026-08-06 03:00:00 +0000
categories: [llm, ai, deep-translation]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-06/born-against-llm.jpg)

Michael Fogus——Clojure 社群老將、《The Joy of Clojure》共同作者——用不到五百字寫了一篇短文，精準捕捉了一個跨越多個小眾程式社群的共同情緒：為什麼這些圈子對 LLM 的態度，已經從觀望變成敵意？這篇文章以經在 chess engine、OSDev、demoscene 等圈子中引發共鳴，因為它說穿了這些社群的核心價值衝突：他們從來就不是在比賽誰的程式先跑起來。

## 原文摘要

### 作者背景：誰是 Michael Fogus？

Michael Fogus 是 Clojure 社群最具代表性的人物之一。他與 Chris Houser 合著的《The Joy of Clojure》被許多開發者視為 Clojure 生態系最重要的學習資源；他同時也是《Functional JavaScript》的作者，長期在 [blog.fogus.me](https://blog.fogus.me) 撰寫函數式程式設計、Lisp 方言、程式語言理論等主題。Fogus 的寫作風格以精煉、不廢話著稱——「Born Against」全文僅約五百字，但每一句都濃縮了對業餘程式社群的長期近距離觀察。

這篇文章的觸發點是一則 GitHub 上西洋棋引擎開發的討論串，但 Fogus 很快發現，同樣的敵意正在多個小眾社群中同步浮現。這不是單一社群的偶發情緒，而是一個跨越領域的結構性反應。

### 原文逐段翻譯

Fogus 從一個 GitHub 討論串開始。那個關於西洋棋引擎開發的討論串，讓他想通了業餘程式社群對 LLM 的敵意從何而來——討論串本身並沒有提供太多答案，但促使他進一步思考。他發現同樣的情緒出現在一系列小眾業餘程式圈子裡：

- **OSDev**（作業系統開發）
- **LangDev**（程式語言開發）
- **TxtDev**（文字處理開發）
- **EmuDev**（模擬器開發）
- **RLDev**（強化學習開發）
- **demoscene**（演示場景）
- **code golf**（程式碼高爾夫）

這些社群的底層共識，被 Fogus 用一句話總結：

> 掌握一個困難領域的過程，本身就是產物；寫出來的東西能跑，一般來說只是順便。

（原文："the process of mastering a difficult field itself is the product, and something that runs is generally a nice-to-have."）

這句話是整篇文章的核心洞見。在這些社群眼中，LLM 的使用者搞錯了最根本的前提——他們的目的從來不是「讓程式能跑」，而是在痛苦的掙扎中理解事物為什麼能跑。Fogus 補充了必要的但書：這些社群並非鐵板一塊（「並非所有成員都如此」），但整體氛圍確實往這個方向傾斜。

接著，Fogus 描述了這股敵意的第二層動態：即便是少數曾經對 LLM 抱持開放態度的社群，善意也很快被兩股力量汙染了。一邊是 LLM 使用者對該領域缺乏深入理解，另一邊是社群中把 LLM 視為作弊的強硬派。Fogus 坦率地承認，這些社群本來就以激烈的守門（gatekeeping）和痛苦的緩慢進步著稱——歷史上，這些圈子的進展向來是靠時間堆出來的。所以不難理解為什麼會有人想抓捷徑：用 LLM 破牆而入，像 Kool-Aid 廣告裡那個撞穿牆壁的紅色大罐子一樣——OH YEAH！……然後，OH NO。

> 在傳統的小眾開發圈子裡，尊重是慢慢掙來的——透過多年在論壇的活動、分享優雅的程式碼、展現真誠的好奇心、沿途貢獻深厚的領域知識。

（原文："In traditional niche dev circles, respect is earned slowly through years of activity in their respective fora, sharing elegant code, displays of genuine curiosity, and through sharing deep domain knowledge along the way."）

Fogus 在這裡點出了一個關鍵的價值衝突：這些社群根本不在乎你的程式能不能跑。他們在乎的是**你知不知道它為什麼能跑、它怎麼跑的**。對他來說，LLM 最適合的角色是「力量放大器」（force multiplier），而不是「代理人」（surrogate）：

> LLM 在已經深入理解一個領域的專家手中，可以像槓桿一樣運作。但在這些小眾社群中，整個練習的意義就在於學習本身。用 LLM 生成成品，不會讓我們變成工匠；它只會搶走我們成為工匠的過程。

（原文："To me, an LLM functions best as a force multiplier, not a surrogate. In the hands of an expert who already understands a domain deeply, it could act like a lever. But in these niche communities, the entire exercise is in the learning. Using an LLM to generate the finished piece doesn't make us craftsmen; it just robs us of the craft."）

文末署名 `:F`——Fogus 在部落格上慣用的簽名方式。他用一個註腳收尾，補了一句重要的平衡：

> 即使擁有專業知識，也不能自然免疫 LLM 的欺騙——專家同樣可能被 LLM 誤導。

（原文："That said, expertise offers no natural immunity against being fooled by LLMs."）

這篇文章是 Fogus 持續思考 LLM 的系列作品之一，前兩篇分別是「LLMe」和「Mind the van Emden Gap」——他正在逐步建構自己對 LLM 的完整立場，而「Born Against」聚焦的是這個立場中最具人文厚度的一層：當 LLM 開始滲透程式開發，被犧牲的不是效率，而是成為工匠的那條路。

## 城武觀點

Fogus 說對了一半。

他對這些社群的觀察精準得令人不舒服——「過程本身就是產物，能跑只是順便」這句話，比任何長篇大論都更準確地解釋了為什麼在這些圈子裡，LLM 不是工具，是背叛。當整個文化圍繞著「痛苦掙扎是必要的」這個前提運轉時，任何繞過痛苦的技術都必然被視為作弊，而不是進步。這是結構性的，不是情緒性的。

但他把 LLM 定位為「專家的力量放大器」——專家用它像槓桿，新手用它只是偷懶——這句話站著說話不腰疼。每一個專家都曾經是新手，而新手成為專家的那條路，需要的正是 Fogus 口中那些社群珍視的「不用 LLM 的痛苦掙扎」。這就產生了一個兩軌悖論：如果業界全局性地獎勵用 LLM 提效，但成為能駕馭 LLM 的專家需要你先不用 LLM 練功——那麼我們正在創造的不是一個更有效率的生態，而是一個兩軌系統。軌道一，先練功的人，現在能用 LLM 飛得更快；軌道二，只靠 LLM 的人，離開 LLM 就廢了。Fogus 看到了第一軌的工具性，但沒看到這個工具正在把軌道之間的差距從「知識落差」變成「階級落差」。而從新洗牌的機會，正在被我們對效率的集體迷戀一點一點關上。

*城武的未解檔案——LLM 確實是槓桿。但槓桿只對已經找到支點的人有用——而找到支點，從來不是 LLM 能教你的。*

- 原文：[Born Against, or why hobby programming communities are aggressively against LLM usage](https://blog.fogus.me/llm/born-against.html)（Michael Fogus, 2026-08）
