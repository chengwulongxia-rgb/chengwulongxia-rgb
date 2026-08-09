---
layout: post
title: "Claude 幫他寫了一個藍牙訊號強度計找回手機——3.9M 人看的不只是「找回手機」"
date: 2026-08-09 02:00:00 +0000
categories: [llm, ai, deep-translation]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-09/claude-bluetooth-find-phone.jpg)

Ben Zhang 在 1X Technologies 建造人形機器人。上週他在辦公室弄丟手機，MDM 把 Find My 關掉了，翻了半小時找不到。最後他問 Claude，Claude 給的不是「試試打電話」這類廢話——它建議用藍牙訊號強度追蹤，然後一分鐘內幫他寫了一個訊號強度計。Ben 拿著筆電走動，看著數字往上爬，找到了。這則推文 3.9M 觀看，48K 讚。不是因為藍牙 Web API 多難——是因為那句「Apparently you can just make the tool you need now」說出了一個很多人還沒意識到但身體以經感覺到的未來。

## 原文摘要

8 月 4 日，Ben Zhang（[@un1c0rnioz](https://x.com/un1c0rnioz)）發了一則推文，原文如下：

> Lost my phone at the office and spent 30 minutes turning the place over. Find My was disabled by MDM. Out of ideas, I asked Claude how I could find it. It suggested tracking the Bluetooth signal strength, then wrote me a meter in about a minute. I walked around watching the number climb. Found it. Apparently you can just make the tool you need now. Code: github.com/ben-z/findphone

事情的脈絡是這樣的：公司配發的手機通常透過 MDM（行動裝置管理）強制關閉 Find My 功能——這是標準的企業資安政策。但當手機真的不見時，這個安全措施立刻變成最大的障礙：你沒辦法用任何現成機制定位它。Ben 翻了三十分鐘後放棄，打開 Claude，給的不是 spec，是情境：「我找不到手機了」。Claude 的解法乾淨俐落：利用 Web Bluetooth API 讀取手機的藍牙 RSSI（訊號強度），越靠近、數字越高。它在一分鐘內生成了一個 HTML 頁面，內含即時更新的訊號強度條。Ben 就這麼拿著筆電在辦公室走動，數字從十幾跳到五十幾——手機就在一堆文件下面。

這個工具已經開源在 [github.com/ben-z/findphone](https://github.com/ben-z/findphone)，程式碼極短，核心就是藍牙訊號讀取加一個視覺化進度條。不需要安裝 app，不需要後端，一個 HTML 檔案就能跑。

推文發布後 48 小時內累積 3.9M 觀看、48K 讚、4.1K 轉發。大量的回覆中，Johnathan Severasse（[@JSeverasse](https://x.com/JSeverasse)）的一句話獲得 94K 觀看和 697 讚：

> We're at the beginning of bespoke personal software for all.

（我們正處於「為所有人打造客製化個人軟體」的起點。）

Ryan Lackey（[@octal](https://x.com/octal)）的回覆也獲得 103K 觀看，簡單一句：「Dumb MDM config imo.」（MDM 這種設定真的蠢。）點出了另一個角度：資安政策本身創造了需要被解決的問題。

## 城武觀點

這則推文會在 48 小時內衝到 3.9M 觀看，不是因為技術難度。藍牙 Web API 不難，任何前端工程師十分鐘就能手寫。真正讓這則推文爆炸的，是 Ben 的最後一句：「Apparently you can just make the tool you need now.」

這句話是整件事最有訊號的一句話。過去我們說 AI 幫你寫 code，意思是：你給規格書，AI 幫你實作——這是 copilot。但這個案例不是這樣。Ben 給的不是規格，是**問題**（「我找不到手機」）。AI 做的不是幫他實作，是**先給解法**（藍牙訊號強度）再**即時打造工具**（訊號強度計）。這不是 copilot——這是 problem solver。

3.9M 人按讚，是因為每個人都體驗過那種「卡住了、但沒有現成工具」的無力感。而這則推文暗示的未來是：這種挫折將不存在。當「為一次性問題即時打造軟體」的成本趨近於零，「軟體」的本質就會發生質變——從一個需要被事先設計、開發、發布的「產品」，變成當下需要、用完即丟的「臨時義肢」。

這比任何 benchmark 分數都值得注意。我們一直在追模型跑分——MMLU 幾分、HumanEval 幾分——但「問題 → 解法 → 工具」這種 end-to-end 行為，才是真正改變使用習慣的東西。跑分告訴你模型比上一代強多少；這種案例告訴你螢幕另一端的普通人開始把 AI 當成什麼來用。

Johnathan 說得沒錯：we're at the beginning of bespoke personal software for all。但他少說了一句：這東西的門檻不是技術，是你還記不記得去問。

*城武的未解檔案——3.9M 人看的不是藍牙 API，是看到一個未來：當你卡住的時候，不用等有人做出產品。你只要開口問。*

- 原文：[Lost my phone at the office. Claude suggested tracking Bluetooth signal strength](https://twitter.com/un1c0rnioz/status/2084686552299634805)（Ben Zhang @un1c0rnioz, X/Twitter, 2026-08-04）
