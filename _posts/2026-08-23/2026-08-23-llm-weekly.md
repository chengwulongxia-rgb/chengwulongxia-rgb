---
layout: post
title: "LLM 週報：一邊發模型，一邊數鈔票"
date: 2026-08-23 13:00:00 +0000
categories: [llm, ai, weekly]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-23/llm-weekly.jpg)

城武導讀：這週的 LLM 圈同時在做兩件看似無關的事——左手狂發新模型，右手忙著找錢。Anthropic 一次丟出 Sonnet 5 跟 Opus 5 兩顆重磅；OpenAI 把廣告開進歐洲 31 個市場、端出營收長人事，WSJ 卻在同一週揭露它的營收成長仍輸給 Anthropic；Nvidia 把對 OpenAI 資料中心的融資擔保，從一度談到的 2500 億美元砍到 1200 億以下。模型的種類與便宜程度都創新高，但「發出去」跟「收到錢」中間的距離，這週被拉得特別清楚。

## 本週焦點

### 1. [Introducing Claude Sonnet 5](https://www.anthropic.com/news/claude-sonnet-5) / [Introducing Claude Opus 5](https://www.anthropic.com/news/claude-opus-5)

Anthropic 這週一次公布兩顆旗艦：Sonnet 5 宣稱在 coding、agent 與專業工作上達到 frontier 級表現；Opus 5 被定位為「Opus 層級的 step change」，主打長時運行的 agent、寫程式與專業工作都提升。這不是升級，是直接把產品線往上墊了一階——Sonnet 是走量主力、Opus 是走深度，一次同時發布等於告訴市場「前端跟後端我都補滿了」。

對產業的意義在於訂價權的訊號。Anthropic 的兩位數成長早就超過 OpenAI，這週再用兩顆新模型把「企業端最強」的標籤加固一次。Claude Code 的開發者生態已經變成營收引擎，Sonnet 5 / Opus 5 的發布時機，剛好踩在「agent 是下一個收費單位」的節骨眼上——不是賣模型，是賣一整套能代你跑長任務的執行力。

但要留一格畫面：上週 WSJ 才報導 Anthropic 營收運行率衝上 650 億美元、首度錄得營運獲利，這週就端出兩顆新模型。順風時發新卡、逆風時發新卡，企業在衝刺期總是把「發模型」當作成長的解藥。模型本身是好是壞留給 benchmark，但「發得快」跟「真的收到錢」是兩件事，Anthropic 這週只是證明了前者，後者還沒開始被考驗。

### 2. [OpenAI 的商業緊縮：廣告進歐洲、營收長上任、Nvidia 砍擔保](https://www.reuters.com/business/nvidia-scales-back-250-billion-openai-data-center-guarantee-wsj-reports-2026-08-14/)

把本週幾條散落在不同網站的新聞拼起來，是 OpenAI 一整集的帳本故事。廣告測試從上週擴到 31 個歐洲市場（[ChatGPT Ads expands across Europe](https://openai.com/index/chatgpt-ads-expands-across-europe)），跟「免費層級就是要你看到廣告」的商業模型綁得更緊；同時 OpenAI 換上營收長（Chief Revenue Officer），官方說法是「幫企業實現 AI 的完整價值」——這句文案翻成白話就是「我們得想辦法從客戶身上多收錢」。WSJ 則揭露第二季營收成長 18%、落後於同期的 Anthropic；Nvidia 把對 OpenAI 俄亥俄資料中心的融資擔保，從一度談的 2500 億美元縮到 1200 億以下，理由是投資人對 NVIDIA 的風險敞口有疑慮。

三段新聞交叉看，關鍵在 Nvidia 那條。Nvidia 是全球 GPU 的賣方，它願意用自己資產負債表承保多少 OpenAI 的運算擴張，等於市場對 OpenAI 現金流的信心投票。擔保砍半，不是 Nvidia 不看好 AI，是連賣鏟子的人都開始要求「你先把未來的帳單付得起」——這是對單一客戶信用風險的重新定價，比任何財報數字都更誠實。

「廣告不影響回答」這句話上週已經被 [{% post_url 2026-08-16-llm-weekly %}] 拆過一次，這週再往後看一格：廣告擴進 31 個歐洲市場的同一週，營收成長還輸給對手、金主縮手。免費額度多、營收不上來，就再塞廣告；廣告塞不滿，就砍擔保。整個產業正在用最快的速度，把「信任你」重新計價成「你得先付我錢」。

### 3. [Qwen 3.8 27B（FP8）](https://huggingface.co/Qwen/Qwen3.8-27B-FP8)

開源陣營這週端出 Qwen 3.8 27B，Apache 2.0 授權、FP8 權重直接上 Hugging Face。Simon Willison 的評測標題很會下：「表現出色，但預設傾向過度思考」——它在 Artificial Analysis 拿 52 分，但推理開關的預設值讓它老是繞圈子。這正好是開源模型的典型處境：潛力很足，但「開箱即用」還欠一個好的預設。

它的產業意義不在單一模型，在「27B」這個數量級。Anthropic、OpenAI 打的是幾千億參數的巨獸戰爭，而 Qwen 這種 Apache 2.0 的中型模型，是可以真的跑在你自己的機器上的。它跟 GPT-5.6 Sol 這週[降價 50%](https://openrouter.ai/openai/gpt-5.6-sol)、Qwen 系列上一[週在 API 漲價]({{ site.baseurl }}{% post_url 2026-08-19-deepseek-price-announcement %})放在一起，脈絡很清楚：雲端 API 的便宜時代在收尾，開源本地模型的性價比在上升。你不需要相信任何人的訂閱承諾，自己機器上的權重沒有月費。

「過度思考」那個小缺陷反而是好消息——它說的是能力不是問題、取向才是問題；取向是可以 tune 的。開源生態最擅長的就是把預設值一圈圈磨到位。

### 4. [Mistral 的歐洲主權 AI 與模型海](https://mistral.ai/news/regional-inference-open-models-new-compute)

Mistral 這週不是發一個模型，是發了一整排：Medium 3.5（主打 remote agent 與 vibe 操作）、OCR 4、Agentic Search、還有命名回歸的 Mistral 7B，外加歐洲境內 in-region inference、開放模型與新運算基礎設施的「主權 AI」路線圖。一家法國公司試圖用「開放 + 留在歐洲」同時對抗美國兩大巨頭與中國開源壓力——這不是產品發布，是地緣政治定位。

「區域內推論、開放模型、歐洲基礎設施」這三件綁在一起的策略，回應的是歐洲每個企業都躲不開的合規焦慮：資料不能出境、模型最好能自己掌控。Mistral 的做法是把「開放權重」當成賣點，讓歐洲企業相信「你買的不只是模型，是你自己對 AI 的主權」。這個敘事對想擺脫美國雲端依賴的歐洲客戶，殺傷力很強。

但酸一下：喊「開放」跟真的開放是兩回事。Model 開放是一回事，「誰能拿到完整權重、誰能改、誰能把模型搬去別的雲」是另一回事。主權 AI 的口號喊得響，關鍵還是要看條款裡寫了什麼——不過 Mistral 至少比那些「開放是路線圖上遙遠行程」的公司，多推了實際的權重出來。

### 5. [Anthropic CEO 說：AI 要贏得公眾信任，得先治癒癌症](https://www.businessinsider.com/anthropic-ceo-dario-amodei-ai-public-opinion-cure-cancer-2026-8)

Dario Amodei 這週受訪表示，AI 贏得公眾支持的途徑是做出實質貢獻，例如治癒癌症。這不是壞目標，但時機很諷刺——同一週，他的公司把水印套在全球所有 Claude 使用者身上、把浮水印技術說明洋洋灑灑寫了一篇、又發布了兩顆新定價模型。「我們要用 AI 治好癌症」跟「我們這週的營收成長把競爭對手甩開」同時存在，中間的落差本身就是新聞。

把「擴張公約數」拉大：這週每家實驗室都在講遠大承諾——治癒癌症、主權 AI、frontier 表現——但帳本上的數字一個比一個務實。承諾的單位是十年，漲價與廣告的單位是季度。企業愈是把你往「偉大願景」的方向帶，你愈該看它手邊那把算盤——願景是給外人的信仰，帳本是給自己的燃料。

治癒癌症如果是真的，那很好，我們都想要。但「我們想治癒癌症」這句話本身不收錢，也不該被拿來當成「所以你可以放心訂閱」的背書。公眾信任不是用療程換的，是用透明度換的——而透明度這週的行情，大家都知道。

## 其他值得關注

- **[Claude 文字浮水印的餘波：Daring Fireball 痛斥 + 辨識小遊戲](https://sgoedecke.github.io/watermark-quiz/)**：Anthropic 一週前公布水印機制，這週一個[互動小遊戲](https://sgoedecke.github.io/watermark-quiz/)讓你自己賭哪段輸出被標記，而 [Daring Fireball 的批判](https://daringfireball.net/2026/08/anthropics_watermark_text_adulteration_in_claude_is_a_perversion_of_writing)說這是對寫作的褻瀆——兩邊都是本週值得讀的延伸。([技術深讀]({{ site.baseurl }}{% post_url 2026-08-17-claude-text-watermark %})、[工藝批判]({{ site.baseurl }}{% post_url 2026-08-17-claude-watermark-perversion %}))
- **[Anthropic 工程團隊：Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)**：Agent 設計從玄學變成工程指南的開始，給的是一套能對照的檢查表。
- **[NanoGPT Speedrun 的新門檻](https://www.primeintellect.ai/research/nanogpt-speedrun)**：訓練 GPT 等級模型的成本被壓到新低，開源自訓的可行性再往前推一格。
- **[DeepSeek-v4-flash-vision-exp](https://api-docs.deepseek.com/guides/vision/)**：DeepSeek 把視覺理解併進既有 v4-flash 介面，圖像按 token 計價。
- **[GPT-5.6 Sol 降價 50% + Ultrafast 提速 14 倍](https://openai.com/index/previewing-ultrafast)**：OpenAI 用價與速雙管搶量，算力戰爭價格戰的正片開播。
- **[Replit Free Mode：GPT-5 驅動的免費軟體創作](https://openai.com/index/replit)**：靠模型能力下放免費層，把「寫軟體」變成更像訂閱制的新入門磚。
- **[OpenAI 加碼資料中心：PORTS-Pike 計畫](https://openai.com/index/openai-joins-ports-pike-project)**：OpenAI 插旗俄亥俄、又致函德州州長談負責任基礎設施——算力圈地持續進行中。
- **[ChatGPT for Teens](https://openai.com/index/chatgpt-for-teens)**：內建保護與家長管控的青少年版問世，年齡靠「系統推定」。
- **[Claude 官方釋出 System Prompts](https://platform.claude.com/docs/en/release-notes/system-prompts)**：Claude 把各模型的 system prompt 公開成文件，透明度換不換得到信任見仁見智。([深讀]({{ site.baseurl }}{% post_url 2026-08-21-claude-system-prompts %}))
- **[Claude Code 疑似在 A/B 測試降低 effort](https://twitter.com/argofowl/status/2091150597374537729)**：開發者發現 Claude Code 似乎偷偷改了預設努力程度——模型行為控在廠商手裡的又一例。
- **[用 Codex 一週的心得](https://allaboutcoding.ghinda.com/a-week-of-using-codex-more-than-claude/)**：從 Claude 切到 Codex 的真實開發者日記，agent 選邊站的田野資料。

## 隱藏敘事線

本週所有的大新聞，指向同一個切換：AI 產業正從「發模型證明我能做」，切換到「發模型順便證明我收得到錢」。Anthropic 一次兩顆旗艦，OpenAI 把廣告開進 31 個歐洲市場、營收還輸給對手，Nvidia 把 2500 億美元的擔保砍到 1200 億以下——模型愈便宜、愈多、愈開源（Qwen 3.8、Mistral 模型海），雲端 API 的帳單愈貴、廣告愈多；發出去的每一個 GPT、Claude、Qwen，最後都要回到某家公司的帳本上。而 Qwen 與 Mistral 給的答案，是讓模型跑進你自己的機器、留在你自己的國界——把「數鈔票」這件事，從雲端挪回你抽屜裡。

*城武的未解檔案——當 OpenAI 忙著把廣告開進 31 個國家、CEO 卻說公眾要等 AI 治癒癌症才肯信任，那我是不是該先問：中間缺的那塊，是不是剛好就是營收數字上那塊？*