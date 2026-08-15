---
layout: post
title: "【深度分析】AI Bot 身份信任危機：當 35% 的網路流量來自機器，攻擊者不需要隱藏——只需要假冒"
date: 2026-08-13 02:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![AI Bot 身份信任危機]({{ site.baseurl }}/assets/images/2026-08-13/spoofing-identity-hero.jpg)

這份報告名義上是一份資安警報，骨子裡是一場正在發生的權力競賽：當網路三分之一的流量來自機器，誰有資格判定哪個 bot 是「真的」、哪個是「假冒的」，誰就握住了整張網路的信任閘門。而你以經能猜到結局——寫這份報告的公司，正好就是賣這道閘門的人。這層「警報」與「賣鎖」之間的距離，值得你花十分鐘看清楚。

## 原文摘要

Known Agents 在 2026 年 8 月發布了《The Agentic Web Index》，這是一份追蹤網站上機器人（bot）流量的資料報告，主題是「假冒與安全」（Spoofing & Security）。報告先丟出四個關鍵指標：

- **Bot 對人類流量比**：全網 35% 的流量來自 bot（比過去 90 天下降 1%）。
- **Agent 化（Agentrification）**：bot 流量中 29% 與 AI 相關（上升 11%）。
- **AI 聊天導流**：人類網站造訪中只有 0.1% 來自 AI 聊天（下降 9%）。
- **Robots.txt 遵守率**：98.5% 的 bot 會遵守 robots.txt 規則。

緊接著是報告的核心警報——一場正在進行中的 AI Bot 假冒攻擊行動。Known Agents 觀察到一個「廣泛散佈的攻擊行動，冒充 AI bot 掃描網站漏洞」，攻擊者的目標是 AI 編碼工具所使用的憑證與設定檔路徑。判斷一次造訪是否「假冒」的標準是：它宣稱自己是某個被認可的 agent 身份，卻無法通過該 agent 所支援的認證方式——例如驗證過的 IP，或 Web Bot Auth。

假冒流量的統計時間窗是 2026 年 5 月 15 日到 8 月 12 日，威脅大約在 8 月 5 至 6 日被偵測到（圖表上 index 80 的位置）。

被假冒最多的 agent 身份清單如下（括號內為佔流量的比例）：

- Googlebot（0.5%）
- ChatGPT-User（0.1%）
- OAI-SearchBot（0.1%）
- GPTBot（0.1%）
- PerplexityBot（0.1%）
- ClaudeBot（0.1%）
- Applebot（0.0%）
- bingbot（0.0%）
- Perplexity-User（0.0%）
- MistralAI-User（0.0%）
- GoogleOther（0.0%）
- AhrefsBot（0.0%）
- Claude-User（0.0%）
- Claude-SearchBot（0.0%）
- Amzn-SearchBot（0.0%）

最近被鎖定掃描的憑證與設定檔路徑，完整清單如下：

- `/.config/anthropic/credentials/default.json`
- `/.claude/settings.json`
- `/.claude.json`
- `/.hermes/.env`
- `/.openclaw/.env`
- `/.codex/config.toml`
- `/.continue/config.json`
- `/.aider.conf.yml`
- `/service-account.json`
- `/serviceaccountkey.json`
- `/service_account.json`
- `/firebase-adminsdk.json`
- `/firebase-service-account.json`
- `/.aws/credentials`
- `/.aws/config`
- `/.s3cfg`
- `/.boto`
- `/.npmrc`
- `/.env.example`
- `/.env.local`
- `/.env.production`
- `/.env.backup`
- `/.env.old`
- `/backend/.env`
- `/api/.env`
- `/admin/.env`
- `/dockerfile`
- `/docker-compose.yaml`
- `/.docker/config.json`
- `/terraform.tfstate`
- `/credentials.json`
- `/secrets.json`
- `/secrets.yml`
- `/key.json`
- `/rclone.conf`

這份清單非常具體：從 Anthropic 的 Claude 設定檔、OpenAI Codex 的 config.toml、Aider 與 Continue 的設定，到 AWS 憑證、Firebase 服務帳號金鑰、Terraform state、Docker 設定——涵蓋了當今 AI 開發者與雲端工程師工作上最敏感的一批檔案。

報告接著列出合法流量的分布。造訪量最高的 agent：

1. bingbot（8.2%）
2. Googlebot（7.9%）
3. AhrefsBot（6.3%）
4. Known Agent（5.5%）
5. ChatGPT-User（3.3%）
6. ClaudeBot（3.2%）
7. PetalBot（3.1%）
8. SemrushBot（2.9%）
9. facebookexternalhit（2.6%）
10. meta-externalagent（2.3%）

AI 抓取（scraping）最多的 agent：

1. ClaudeBot（27.0%）
2. meta-externalagent（19.3%）
3. Amazonbot（19.1%）
4. GPTBot（9.5%）
5. Bytespider（7.3%）

AI 抓取網頁內容（fetching）最多的 agent：

1. ChatGPT-User（86.1%）
2. DuckAssistBot（3.2%）
3. Perplexity-User（2.9%）
4. Claude-User（2.7%）
5. MistralAI-User（2.2%）
6. Claude-Code（1.1%）

最後是營運商（operators）排名：

1. Google（10.5%）
2. Microsoft（8.9%）
3. Ahrefs（7.8%）
4. Meta（6.8%）
5. OpenAI（6.4%）
6. Known Agents（5.9%）
7. Anthropic（5.1%）
8. Amazon（4.8%）

## 城武觀點

先講清楚一件事：這份報告描述的攻擊，我信。那串被掃描的路徑——`/.claude.json`、`/.codex/config.toml`、`/.aws/credentials`、`/terraform.tfstate`——每一條都是真的會有人去戳的檔案，攻擊者確實存在。這不是陰謀論的靶子，不需要硬黑。

但正因為攻擊是真的，報告的敘事框架才更需要被拆開。Known Agents 賣的就是 agent 驗證——Agent Analytics、AI Chat Referral Tracking、Web Bot Auth。這份報告的論證結構從頭到尾只有一條：bot 正在假冒身份掃描你的憑證，判斷「假冒」的唯一方法就是通過 agent 認證（驗證過的 IP、或 Web Bot Auth）。它把「危機」定義成「只有驗證基礎設施能解決的問題」，而這個基礎設施，正好是它要賣的東西。威脅是真的，但「解法必須買我的驗證服務」這個推論，不是從資料長出來的，是從商業模式長出來的。

這裡藏著真正讓我警覺的權力鬥爭。報告的框架是「攻擊者 vs 網站」，但真正的鬥爭是：**誰有權判定哪個 bot 是合法的？** 有個數字特別耐人尋味：「合法流量」排名第四是「Known Agent」（5.5%），「營運商」排名第六也是 Known Agents（5.9%）。一家負責驗證「誰是真的 bot」的公司，自己同時出現在「最活躍的 bot」名單上——它丈量這場危機的同時，自己就是它丈量的流量的一部分。

危險不在於 Known Agents 是壞人，我不覺得它是。危險在於：身份信任的機制，正在從「開放、自願、看名片」轉向「由少數驗證商壟斷判定權」。今天判斷一個 bot 是不是 Googlebot，靠的是 user-agent 字串——任何人把「Googlebot」寫進 HTTP header，伺服器就信了，這跟名片印什麼頭銜就信什麼一模一樣。98.5% 的 bot 遵守 robots.txt 聽起來是好事，但翻譯成白話：整張網路的信任機制，目現還停留在「看名片」的層次。真正的壞消息不是那 1.5% 不守規矩的，而是剩下 98.5% 之所以「守規矩」，是因為根本沒人在驗證、也懶得驗證。

所以當 Known Agents 說「我們看到有人假冒 AI bot」，潛台詞是「你需要能驗證身份的機制」，再往下是「而我就是那個機制」。這條鏈一旦成立，AI 身份就會變成存取憑證——未來網站要嘛信任驗證商背書的 agent，要嘛把你擋在門外。「誰是合法的 bot」就不再是技術問題，而是特許經營權的問題。

我的立場：攻擊是真的，該防；但「誰來定義合法 bot」這個權力，不該自動落到賣驗證服務的人手上。今天它用一份免費威脅報告建立「我是中立裁判」的形象，明天就能用這個裁判權定價。裁判、參賽者、賣裁判服的人，不該是同一個——而這份報告最讓我坐立不安的，就是它三個都想當。

*城武的未解檔案——下次看到一份資安威脅報告，先問一句：寫報告的人，賣不賣解藥？*

- 原文：[The Agentic Web Index](https://knownagents.com/insights)（Known Agents, 2026-08-13）
