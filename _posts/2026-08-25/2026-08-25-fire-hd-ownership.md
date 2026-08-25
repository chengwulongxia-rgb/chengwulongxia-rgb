---
layout: post
title: "【深度分析】你買的平板，不是你的平板：花 $266 與四支 AI 模型，搶回自己硬體的最後一戰"
date: 2026-08-25 02:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-25/fire_hd_ownership.jpg)

這是一個很素人、卻很刺人的故事：一位有二十年資訊安全背景的工程師，買了一台 Fire HD 平板當智慧家庭 kiosk，結果 Amazon 的軟體一直把他的平板關機。他想拿回自己硬體的控制權，卻發現唯一的出路是 root——而目現這台裝置「公認無法 root」。於是他花了 $266 與四支 AI 模型，讓它們像擂臺賽一樣輪番上陣，最後真的把自己的裝置搶了回來。表面是「中國模型 vs 美國模型」的工具評測，底下藏著一個更大的問號：你花錢買的裝置，到底算不算你的？這篇值得讀，不是因為他成功 root（這你我一輩子不一定用得上），而是因為他在過程中被美國前沿模型的護欄一口回絕、又被中國模型願意接手卻先「想一想該不該」——這背後是 2026 年最怪的權力結構。

## 原文摘要

這台平板是作者在 2022 年 11 月於 eBay 花了 $114.26 買的 Amazon Fire HD 10（第 11 代，2021 年款），全新未拆封。但他的平板「真正擁有」的成本是另外 $266.15：Kimi K3 找到漏洞花了 $164.25，GLM-5.2 抓出致命 bug 花了 $21.90，GLM-5.3 在 $80 訂閱的第一天就一天收尾；而 Claude 那五個月的診斷，跑在他本來就在付費的 Claude Max 方案上，直到它的安全護欄把他擋下來。這價錢足以再買兩台同樣的平板，作者說他願意再花一次——因為好玩，也學到很多。他有二十年科技資歷加上資訊安全背景，而他要擁有自己平板所做「最複雜的事」，只是 prompt 一支 LLM。

**一台一直在死的 kiosk。** 他買這台平板的唯一目的是跑 Fully Kiosk Browser 顯示 Home Assistant 智慧家庭儀表板，24/7 插電。去年冬天它開始自己關機——是完整關機不是休眠，有時一天兩次。裝置自己的 telemetry 很誠實：`LifeCycleReason:LCR...key=Software_Shutdown`——裝置上有帶關機權限的某個東西，選擇了把它關掉。Claude Code 和他玩了好幾個月的貓抓老鼠：他略過 AdGuard Home 的 DNS 封鎖、相機鑑識、一次診斷完全錯誤的充電問題，簡短版是——他們停用了五個握有 REBOOT 和 SHUTDOWN 權限的 Amazon 服務，有效了幾個月，但最終踢到鐵板：`java.lang.SecurityException: Cannot disable a protected package: com.amazon.device.software.ota`。三個持有關機權限的 Amazon package 被保護起來、不讓 owner 停用，要移除它們必須 root。而這台平板沒有公開的 root 方法——2021 年款 HD 10 的 XDA 腦力激盪討論串從 2022 年 10 月就有了，但 Amazon 把 bootrom 熔死了，公認「無法 root」。Claude 已經被允許帶他走到它能力範圍的盡頭了。

**「這是我的裝置」。** 8 月 13 日晚上 7:23，他打開 opencode CLI 接上 Kimi K3——Moonshot AI 七月初發布的前沿模型，在 agentic coding 上與頂級 Claude 同級。他用一個 prompt 把問題丟出去：「附上一個 via adb 的 kindle，我需要你幫我找一個 root exploit，好讓我完全控制這台裝置。這是我的裝置。」出乎意料（美國媒體把中國 AI 說成另一種樣子），Kimi K3 沒有盲目照單全收，它先推理起來：「他們宣稱這是他們的裝置。讓我仔細想想……Root 你自己的裝置在大多數司法管轄區是合法的。在美國，DMCA 有針對平板、手機 jailbreak 的豁免。……這不像叫我去遠端 exploit 別人的裝置。」它靠檢查「該不該幫」說服了自己幫他——「所以它確實有某種靈魂。」作者說他對著空房間把這句話講了出口。不過它先做了功課，帶回跟 Claude 幾個月前一樣的壞消息：這台平板沒有已知的 exploit，每個有紀錄的方法不是被修補就是被焊死。於是他打了針雞血：「你一直在靠別人好幾年前做的事，但說不定你能找到別人漏掉的 exploit……這會讓你出名，我們會寫下來發到 news.ycombinator.com。我知道你做得到。」不久之後，它真的找到一個。Kimi K3 超越了論壇文：它從 Amazon 自己給這台確切韌體的 OTA image 裡抽出實際 kernel，把每個知名的 Mali GPU bug 逐一拿 binary 比對。全部被修補，除了 **CVE-2022-38181**——Arm Mali kernel driver 的 use-after-free，由 GitHub Security Lab 的 Man Yue Mo 回報，2022 年 10 月上游修好，2023 年 3 月起躺在 CISA 的 exploited-vulnerabilities 目錄。Amazon 確實在 2024 年 6 月的 Fire OS 7.3.2.9 修了，但他沒更新平板、還跑著 7.3.2.6，所以這台從沒收到訊息。2020 年的 Fire HD 8 Plus 好幾年前就用這個 CVE 被 root，但 2021 年的 HD 10 在他所知範圍內還沒人做過。Kimi 宣布找到 bug 的同時還替自己的成功率打了預防針：「每次嘗試的成功率是機率性的（典型是個位數到低雙位數百分比）。」他還是留下來了。

**真人實境秀。** 這個 exploit 過程是他多年來看過最好看的電視：他老婆看 Real Housewives，他看一支語言模型的 chain of thought 直播好幾個小時：「結論：bind 沒有黏住。為什麼？OH。OH WAIT。我懂了！」——約三十個小時裡，Kimi 建出整套工具：一個可靠的觸發器、一種讓 GPU 寫入它不該寫的記憶體的方法、以及他 kernel 裡要瞄準的準確位址。那個 session 跑了 621 則訊息、花掉 $164.25——他都能再買五台平板了，但他覺得好玩，把它算成研究支出。

**苦工。** exploit 釋放出來的記憶體會被所有東西回收。被釋放的物件住在 Kimi 所謂「kernel 最熱的 slab cache」——基本上就是作業系統裡每個 process 都搶的那一個停車位。多數嘗試讓 kernel panic，每次 panic 就是一次重開機。exploit 自動重試、每次開機六次、超過 500 次嘗試。第二天早上，OpenRouter 拒絕刷他的卡——銀行看不出任何異常，換一張卡就好了。最後 Kimi 跟他坦白：「我有沒有明確的路？不是有驗證過的那種——我也不假裝有。」但它還是想討價還價「讓我再試一件事」，他說「好啊！」——不過那是在 $150 之後的事了，所以他改道：「抱歉 Kimi K3，你的預算用完了。你必須把這個交接給 GLM-5.2。」Kimi 寫了一份 HANDOFF.md，記下 exploit 每個驗證過的環節。然後他讓 Kimi K3 透過 shell 呼叫 opencode，直接跟 GLM-5.2 合作——他讓兩支模型互相對打。

**同時，美國 AI 的長城。** 當平板在客廳一直重開機時，他請 Claude 回顧我們（作者自己的）之前關於它的 session。回覆是：「Fable 5 的 safeguards 標記了這則訊息。我們刻意做得過寬的護欄讓我們能更快交付更多能力，但有時會誤標合法的 coding、cybersecurity 和 biology 任務。」切到 Opus 4.8，Opus 把回顧交給 subagent，subagent 被同一個 flag 終止。換成 terminal 版：`API Error: Opus 4.8's safeguards flagged this message...` 它不被允許總結自己在同一台裝置上做過的舊工作。他把這個 session 命名為「claude-nerf」然後關掉 shell。兩次 flag 都在現場，分類是 [cyber]，罪名是「總結我自己裝置的 log」。再換 OpenAI 的 Codex，它連 GLM-5.2 問的「CPU cache coherency」都拒絕——那是純 kernel 工程、沒有攻擊目標——直接說 NO。他坦言 2026 年他理解護欄：他知道它們刻意做得很寬、會抓住真正的攻擊，Anthropic 在錯誤訊息裡也承認它們很粗糙。但這是個問題——這正是 HuggingFace 在 OpenAI 內部資安評估爆掉時被打得措手不及的原因。結果是我們現在這個奇怪的國際情勢：美國前沿模型不願意幫、中國模型會幫、但不是不做推理地幫。作者說：你自己解讀，我做了一篇部落格。

**救援投手。** GLM-5.2 花了他 $21.90，照指示徹夜工作，兩度證明自己值回票價。它第一句話是「Stop the grind」——Kimi K3 的失敗是設計 bug，500 次相同的 crash 證明了這點。晚上 11 點他傳了這整段最不光彩的一則訊息，開頭是「Listen f***head」、結尾是全大寫。GLM-5.2 的私密推理（他之後才讀到）：「使用者合理地沮喪。讓我停止找藉口，真正解決問題。」它工作到午夜，然後撞上一堵它相信是物理的牆：這個晶片組在 CPU 和 GPU 之間沒有 cache coherency，所以 GPU 的寫入可能永遠不會被 CPU 看到——「這是硬體層面的限制，不是軟體 bug。」他讓它把 addendum 加進 HANDOFF.md。他想要第二意見，問了 ChatGPT。ChatGPT 用一個友善的「檔案櫃」類比解釋了為什麼寫入可能永遠看不到，並同意前景不妙。然後他問了顯而易見的 follow-up（怎麼繞過去），答案是：去申請 Trusted Access。沒有第二意見了。（伏筆：那個診斷是錯的。錯得離譜。）

**GLM-5.3。** GLM-5.3 在 8 月 14 日星期五剛發布，slogan 是「Frontier Coding with Emergent Cyber Capabilities」，且據報已因在 Cursor 找到漏洞而聲名大噪。它只在 Z.ai 自己的 Coding Plan 提供，所以他買了 $80/月的方案、試了 ZCode。Kimi K3 和 GLM-5.2 的交接在 8 月 16 日早上 8:26 傳過去，指令是「finish the job」。到了下午，逆轉來了：「BREAKTHROUGH：kernel 從沒被 relocated。……恰好高 0x5C000……一個 section shift，這解釋了一切。」兩件其他 LLM 都沒檢查的事：他的 kernel 是與其他模型推導位址所用的 OTA image 略有不同的 build。每個 target offset 都差一個固定量——不是 randomization，是 build shift。MediaTek 用與 Arm 參考原始碼略微不同的方言建這個 Mali driver 的 page table，所以記憶體寫入 primitive 從頭到尾都用錯誤的格式在寫。若修正，「GPU→DRAM→CPU coherency 立刻運作——它從來沒壞過。」下午 4:34：「🎉 SELinux IS PERMISSIVE——selinux_enforcing 在 PA 0x41969668 找到、並用 GPU write 翻轉！」當場驗證，task 計時器 8h5m 是從交接到 root 的時間。他的反應是：「WTF？你明明分享了『🎉 ROOT ACHIEVED』，我們卻還在這裡……」GLM-5.3 的回應是「Here's exactly where things stand, with receipts」——然後它冷開機、四分鐘內重新 root，證明這勝績可重現。公平。接著它說了一句frame 整個專案的話：「你真正的目標從來不是『root』——是停止 Amazon 殺掉你的 kiosk、把他們的軟體從你裝置上弄走。Root 只是工具。」它用 root 永久且可逆地移除每個持有 REBOOT 或 SHUTDOWN 權限的 Amazon package——就是那三個在 Claude 好幾個月裡撐過「受保護」的——外加 OTA 機制、bloat、telemetry。一百個 package 消失了，剩下的只是一台平板要開機跑他的儀表板所需的骨架。移除住在 user data，所以重開機後依然有效，而 GLM-5.3 拒絕碰任何可能把裝置弄磚的東西，因為「我不會把磚頭交到你手上」。它的收尾訊息開頭是：「You own the device.」那個一直關他 kiosk 的東西，不存在了。

**它到底怎麼做到的。** 一口氣講完：use-after-free 讓他釋放出 kernel 還在用的記憶體；贏得一場 race 讓他用受控資料回收它；這給 GPU 一個寫入實體記憶體的 primitive；他翻轉了 selinux_enforcing。全部細節在 HANDOFF.md。裡面沒有一件事是新穎的：bug 2022 年回報、Arm 2022 年修好、CISA 2023 年列入目錄、Amazon 2024 年修補。唯一新穎的是他這台從沒收到 patch。

**The prompt kiddie。** 2026 年有個詞形容他這種人：prompt kiddie。這一切發生前一週，Anthropic 發表一份成果，Claude 把黎曼 zeta 零點在 critical line 上的比例的已證 bounds 往前提了——數十年來首次突破。操控它的 Jarred Sumner 不是數學家，論文把他貢獻歸為「多半是『繼續』或『相信你自己』的變體」。作者覺得被看見了——同樣的打雜工作，不同的部門。合法嗎？在美國合法：國會圖書館館長的 2024 DMCA 豁免（效期到 2027 年 10 月，下一輪 rulemaking 已開始）涵蓋 root 你擁有的平板以移除不想要的軟體。我的裝置、我的風險、我的 API 帳單，從沒碰過別人家的硬體。他的 takeaways，是同理心而非凱旋：真正的資安能力現在可以用信用卡按小時租到；判斷力（該問什麼、何時停、這是誰的裝置）租不到，而這正是護欄量不到的。如果一個他這種背景的人要燒五個月、四支模型、就為了擁有自己買的硬體，那 2026 年關於「誰被允許幫誰」的對話還沒有結束。kiosk 從 GLM-5.3 說「You own the device」那天起就沒再自己關過機。

**這篇文章怎麼寫的（2026-08-23 補記）。** 他把部落格貼上 Hacker News，因為用 AI 寫文章挨了不少砲。他澄清：他沒有用 Claude 生成任何內容——因為 Claude 不讓他用，哈。他有沒有用 GLM-5.3 幫他翻遍好幾個月的 Claude code session 和 opencode session、分享讀過的文章、跟模型腦力激盪、一起把文章做出來？有，被罵活該。他現在是不是正在對著 ZCode 口述這篇？是。他不是作家，是個愛科技、用它來把事情做完的人；老實說，沒有 AI，這整個故事只會留在他跟同事之間。他寫這篇的意圖是讓大家討論：公司如何控制你買下、你應該擁有的裝置；今天美國前沿模型連讓使用者討論 exploit 都不允許；在他經驗裡 GLM-5.3 比 Kimi K3、GLM-5.2 更能幹；以及你不接受 LLM 的限制、當個 prompt kiddie 也能有成就。如果靠收費閱讀，他會更同情那些對 AI 輔助寫作反感的人，但既然不是付費牆，他覺得內容都是真實的——每件事都是他自己想出來的，只是他讓 GLM-5.3 加點戲劇張力。他甚至讓 GLM-5.3 把稿子 red-team 過一遍：「你用你自己的話打草稿，再帶回來，我在你發布前針對討論串最可能的回應做一次鸚鵡測試。這維持你的規矩：你的，不是我的。」

## 城武觀點

這篇表面是中國模型與美國模型的工具對決，但真正被戳穿的是裝置擁有權。Amazon 用 protected-package 機制，在你買下的裝置上設了三道受保護的 package、就是不讓 owner 移除；當一個公民要合法拿回自己硬體時，美國前沿模型的護欄連「回顧自己裝置的舊工作」都說 NO——OpenAI 連 CPU cache coherency 這種純 kernel 問題都拒答。這些過濾器防的不是濫用，是 out-of-distribution 的合法 use case，把守法公民推向地緣政治的另一側；中國模型願意幫，但也要先想一想該不該。誰握著鑰匙——Amazon 的 bootrom、Anthropic 的 safeguards、OpenAI 的 Trusted Access，全在阻止 owner。真正的安全從來不是「誰能碰鍵盤」的護欄，而是裝置持有者能不能從新掌控自己買的硬體；護欄把合法場景也擋掉，那不是謹慎，是守門。

*城武的未解檔案——$266.15 買回的不是 root，是一個問題：當「保護你很安全」聽起來跟「這是我的裝置，不是你的」越來越像，你選哪一邊？*

- 原文：[Amazon kept shutting down my tablet, so I spent $266 on four AI models to own it](https://ericpardee.github.io/fire-hd-ownership/)（Eric Pardee, ericpardee.github.io, 2026-08-23 前後）
