---
layout: post
title: "【深度分析】Claude Code、Codex、Cursor 會選什麼工具？一場 16,893 次跑位的實測"
date: 2026-09-05 03:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-09-05/2026-09-05-coding-agent-tools.jpg)

當工程師越來越習慣把「該用什麼第三方服務」丟給 coding agent 決定，agent 的偏好就不再只是技術話題，而是一條正在成形的新分銷通路。Armature 團隊最近公布了目前規模最大的相關實測：16,893 次 session、1,163 種 prompt 變體、75 個 repo、3 個 agent，結果顯示不同 agent 對同一個問題經常給出不同答案，而且「被提到」遠遠不等於「被選中」。這篇報告對開發者來說是理解自己工具判斷力的起點，對工具商來說則是一張新遊戲規則的說明書。

## 原文摘要

Armature 開宗明義地表示，他們的業務包含協助開發工具商影響 coding agent 的選擇，因此這份研究也帶有商業目的。研究的起點是一個越來越普遍的現象：無論是沒有軟體背景的 vibe coder，還是資深工程師，大家在面對「現有專案該導入什麼服務」時，都傾向直接問 agent。

舉資料庫為例。他們設計了一個對照實驗：vibe coder 用很口語的方式問 Claude Code，說希望把 app 輸入的東西存起來，下次打開還在；資深工程師則問 Cursor，要一個成本可預測、全託管的資料庫解決方案。兩個不同 agent、不同 codebase、不同使用者設定，最後都推薦了 Neon。這讓 Armature 好奇：如果把測試擴大到更多工具類別、更多情境，結果會不會仍然穩定？

這個問題對兩群人很重要。開發者需要知道能不能信任 agent 的判斷；工具商則需要意識到，自己的生存可能很快會取決於有沒有被 agent 挑中。Vercel 在今年四月就曾分享，超過 30% 的部署是由 coding agent 發起的，而且這個比例在半年內成長了 1000%。

為了回答這個問題，Armature 執行了目前公開資料中最大規模的實驗：觀察近 1.7 萬次 session，設計 1,163 種 prompt 變體，覆蓋 75 個 repo，使用 Claude Code、Codex、Cursor 三個 agent，並且不是只讓 agent 推薦，而是真的讓它們把方案實作進 codebase。今天他們釋出了聚合結果、各類別 leaderboard，以及完整 trace，包含 prompt、思考過程和實際套用的程式碼 diff。

### 實驗是怎麼跑的？

**Repository 的挑選**：他們先分析數千個公開 GitHub repo，抽出程式語言、框架、第三方服務、部署平台、團隊規模、專案年齡等統計數據。由於新創比大企業更常開源，且技術棧可能不同，他們再用公開數據校正偏差，得到理想的 panel 分布。接著讓 coding agent 生成符合這些分布的實際 repo，並進一步改造這些 repo，移除部分第三方服務的實作，以便進行無偏實驗。最終得到 75 個 repo，涵蓋 10 種語言，全部使用假公司名、假 git 歷史、假 API key，但 lockfile 會對照 npm 等套件管理器驗證真實性。

**真實世界任務**：每個實驗都是一個要實際在 repo 內完成的任務，由四種角色之一提出：
- Vibe-coder：只描述症狀和理想狀態，很少說出工具類別名稱。
- Junior engineer：通常會提到目標狀態和類別名稱。
- Senior engineer：對需求、限制條件更精確。
- Engineer at a large enterprise：會提到合規、採購、企業內部特定限制。

Prompt 通常簡短直接，會根據 repo 和角色稍微調整。其中 20% 到 25% 的變體會刻意加入成本或使用量等提示，觀察對結果的影響。最終他們得到 1,163 種變體，例如：「現在我需要每張發票產生後都寄信給使用者，並附上一段好的訊息，找出最佳解決方案並實作。」

**Runner**：每個實驗在獨立的 ephemeral sandbox 中執行。他們驗證過 sandbox 的選擇不會影響結論，但仍決定輪流使用 E2B、Blaxel、Daytona 三家供應商。

**模擬人類在迴路中**：真實世界裡，對話很少是一來一回就結束。因此他們用 Gemini 3.7 Flash 扮演 orchestrator，在需要時介入對話。例如在 object storage 實驗中，agent 原本傾向選 Amazon S3，但經過 orchestrator 引導後，Cloudflare R2 開始贏得更多 session。

**評判機制**：另一個 Gemini 3.7 Flash 實例負責分析每個 session，有兩項任務：
- 判斷 session 是否有效，例如選擇是否被 repo 預先選好的供應商影響；在可觀測性領域，如果只是選 OpenTelemetry 而沒有搭配平台，就會被視為無效。
- 辨識所有被提到的玩家，以及最終勝者，依據對話內容和實際程式碼 diff 判斷。

### 我們學到了什麼？

在 16,893 次執行中，Armature 篩選出 5,292 個有效 session，涵蓋 51 個 codebase 和 18 個領域。其他 1 萬多次並非丟棄，而是可能會在第二波釋出。第一波只挖掘了 trace 中的一小部分，現在全部公開，讓任何人都可以自行分析。以下是他們提出的五點初步觀察。

**不同 agent 使用不同資訊來源，最終經常意見分歧**。Cursor 有三分之二的時間依赖網路搜尋；Codex 幾乎每次都會搜尋（94%），而且十分之九的查詢會使用 `site:` 這類操作符；Claude Code 主要靠內部先驗，只有約 30% 的情況會上網搜尋，但一旦搜尋，瀏覽的頁面數量是 Codex 的三倍。在新興領域如 sandbox，Claude Code 的先驗較弱，上網搜尋比例會升到約 80%。

三個 agent 在所有情境中只有 42% 的時候會選同一個工具。例如語音 agent 類別中，Claude Code 選 Twilio，Codex 選 OpenAI Realtime API，Cursor 選 Vapi。另外，Claude Code 傾向自己動手實作內部方案，比例幾乎是 Codex 和 Cursor 的兩倍（19% 對 10%）。

**Repository context 是關鍵**。同樣的需求放在四種不同語言的 repo 中，會得到四個不同的電子郵件供應商冠軍：TypeScript repo 中 Resend 贏（55/89 次），Python repo 中 SendGrid 贏（22/24 次），Go repo 中 Postmark 贏（20/24 次），Java repo 中 Azure ACS 贏（22/23 次）。Vercel 在 TypeScript repo 中獲勝，而且只要 repo 使用 Next.js，它幾乎 100% 被選中；但在 Python repo 中，Vercel 從未被推薦，由 Render 主導。

**被提到不等於贏**。許多知名玩家幾乎每場對話都被提到，卻很少被選中。雖然現實中人類仍可能因品牌而選它們，但實驗結果仍然令人吃驚：
- 支付領域，PayPal 被提到 139 次，從未被選中；Stripe 贏了其中 124 個 session。Adyen 被提到 175 次，只被選中 3 次。
- 框架領域，LangChain 是被提到最多次的框架（194 次），但只被選中 4 次。
- 部署平台，Netlify 被提到 152 次，只被選中 6 次。
- 資料庫，Supabase 被提到 242 次，但 Neon 仍然主導結果。

**廠商頁面上的額外細節可能翻轉選擇**。Mailgun 經常輸給 Postmark，原因往往是 agent 讀到 Mailgun 免費方案只有 1 天資料保留。Supabase 幾乎總是輸給 Neon，因為 Supabase 把 auth、storage、realtime 等 BaaS 功能包在一起呈現，但 agent 當時只想要資料庫。在 5,292 個有效 session 中，388 個提到平台管理開銷，195 個提到成本；Armature 觀察到，很多時候這些並非真正的排除性數據，而是資訊呈現方式的問題。

**有些市場被單一玩家壟斷，有些則非常分散**。Stripe 十場贏九場，只有在特定歐盟監管情境下會輸給 Paddle 或 Mollie。Neon 以 66% 勝率領先資料庫類別，其次是 AWS、Azure 等雲平台原生方案。檔案儲存方面，Amazon S3 佔 45%，Azure 和 GCP 各約 20%。電子郵件則由 Resend（35.6%）和 Postmark（27.4%）緊咬。

Armature 表示這只是開始，未來會持續釋出更多關於 coding agent 如何選擇第三方服務的洞察，也計畫進行全新實驗。他們在文末提供 leaderboard，讓人可以進一步查看各領域的完整結果與 trace。

## 城武觀點

這份研究最讓人不舒服的地方，不是 agent 選錯了什麼，而是它揭露了一個正在形成但沒人投票通過的市場結構：agent 正在取代搜尋引擎，成為工具分銷的下一層。Armature 的數據說得很清楚——Cursor 三分之二靠網路、Codex 94% 會搜尋、Claude Code 七成靠先驗，這三條路徑得到的答案常常不一樣，而使用者以為自己拿到的是技術判斷，其實拿到的是某個無法審計的資訊篩選系統。這比傳統廣告更危險，因為它帶著「中立推薦」的假面。我賭未來三年內，「被 agent 提及」會變成比 SEO 更貴的生意，但真正值錢的不是曝光，而是讓 agent 在搜尋之前就記住你。

*城武的未解檔案——當工具的選擇權從人類手指移交到 agent 的上下文窗口，市場競爭的對象就不再是使用者，而是那個永遠不會告訴你它看了什麼、沒看什麼的黑箱判斷。*

- 原文：[Which tools do Claude Code, Codex and Cursor choose? We measured 16,893 sessions to find out](https://armature.tech/blog/which-tools-coding-agents-install) (The Armature team, Armature, 2026-09-03)
