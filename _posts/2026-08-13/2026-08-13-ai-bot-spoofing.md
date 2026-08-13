---
layout: post
title: "【深度分析】AI Bot 身份信任危機：當 35% 的網路流量來自機器，攻擊者不需要隱藏——只需要假冒"
date: 2026-08-13 02:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-13/spoofing-identity-hero.jpg)

Known Agents 發布了它們的 Agentic Web Index，追蹤 5,000 多個網站的 bot 流量。數據本身已經夠驚人：35% 的網路流量來自 bot，其中 29% 是 AI 相關。但報告中隱藏的一則警告才是今天最值得拆解的東西：有人正在大規模偽裝成 AI bot，掃描網站上 AI 開發工具的設定檔和憑證路徑。

---

## 城武導讀

這份報告的價值不在於「bot 流量佔 35%」這種數字——這個數字每年都在漲，讀者已經麻痺了。價值在於它揭示了一個結構性矛盾：AI bot 因為「守規矩」（98.5% 遵守 robots.txt）而獲得信任，而這個信任本身變成了攻擊面。攻擊者不需要繞過你的防禦——他們只需要戴上一張值得信任的臉。

---

## 原文摘要

### 基本數據

Known Agents 的 Agentic Web Index 基於 5,000+ 使用其 Agent Analytics 和 AI Chat Referral Tracking 產品的網站數據，提供了一個關於 bot 生態系的快照：

- **Bot vs. Human 流量**：35% 的網站訪問來自 bot（較前 90 天微降 1%）
- **Agentrification**：bot 流量中 29% 是 AI 相關（較前 90 天上升 11%）
- **AI Chat Referral**：人類網站訪問中僅 0.1% 來自 AI 聊天推薦（較前 90 天下降 9%）
- **Robots.txt 有效性**：98.5% 的 bot 遵守 robots.txt 規則

### Bot 流量分布

**Top Agent Types：**
- Search Engine Crawlers：22.8%
- SEO Crawlers：19.6%
- AI Search Crawlers：12.5%
- Developer Helpers：11.0%
- AI Data Scrapers：10.9%
- Fetchers：7.0%
- AI Assistants：3.8%

**Top Visiting Agents：**
1. bingbot：8.2%
2. Googlebot：7.9%
3. AhrefsBot：6.3%
4. Known Agent：5.5%
5. ChatGPT-User：3.3%
6. ClaudeBot：3.2%
7. PetalBot：3.1%

**AI Scraping 分布：**
- ClaudeBot（Anthropic）：27.0%
- meta-externalagent（Meta）：19.3%
- Amazonbot（Amazon）：19.1%
- GPTBot（OpenAI）：9.5%
- Bytespider（ByteDance）：7.3%

**AI Fetching 分布（為 AI 助手即時抓取內容）：**
- ChatGPT-User：86.1%
- DuckAssistBot：3.2%
- Perplexity-User：29%
- Claude-User：2.7%

### 偽裝攻擊：AI Bot 身份被冒用

報告的 Spoofing & Security 區塊揭露了一場活躍的攻擊行動。定義：當一個訪問聲稱自己是某個已識別的 agent 身份，但**未能通過該 agent 支援的驗證方法**（如 verified IP 或 Web Bot Auth），即被視為偽裝。

**Top Spoofed Agent Identities：**
1. Googlebot：0.5%
2. ChatGPT-User：0.1%
3. OAI-SearchBot：0.1%
4. GPTBot：0.1%
5. PerplexityBot：0.1%
6. ClaudeBot：0.1%

報告特別指出：「We are observing a widespread campaign impersonating AI bots to scan websites for vulnerabilities. The attacker appears to be targeting **credential and configuration paths used by AI coding tools**."

**被掃描的目標路徑（完整清單）：**

Anthropic/Claude 相關：
- `/.config/anthropic/credentials/default.json`
- `/.claude/settings.json`
- `/.claude.json`

其他 AI 開發工具：
- `/.hermes/.env`
- `/.openclaw/.env`
- `/.codex/config.toml`
- `/.continue/config.json`
- `/.aider.conf.yml`

雲端服務憑證：
- `/service-account.json`
- `/serviceaccountkey.json`
- `/service_account.json`
- `/firebase-adminsdk.json`
- `/firebase-service-account.json`
- `/.aws/credentials`
- `/.aws/config`
- `/.s3cfg`
- `/.boto`

環境設定檔：
- `/.npmrc`
- `/.env.example`、`/.env.local`、`/.env.production`、`/.env.backup`、`/.env.old`
- `/backend/.env`、`/api/.env`、`/admin/.env`

基礎設施：
- `/dockerfile`、`/docker-compose.yaml`、`/.docker/config.json`
- `/terraform.tfstate`
- `/credentials.json`、`/secrets.json`、`/secrets.yml`、`/key.json`
- `/rclone.conf`

---

## 城武觀點

這份報告最值得停下來看的不是 35% 的 bot 流量——那個數字只是背景噪音。值得注意的是三個層次的結構性問題。

**第一層：AI 身份本身就是存取憑證。** 攻擊者不是在繞過驗證——他們在冒用一個**預設被信任的身份**。98.5% 的 bot 遵守 robots.txt，這意味著網站對「合規」的 bot 幾乎不設防。當你的防禦邏輯是「如果你是 ClaudeBot 且你遵守 robots.txt，那我就讓你進來」，攻擊者只需要在 HTTP header 裡寫上 `User-Agent: ClaudeBot` 就能過關。AI bot 的身份從一個描述性標籤變成了一個**功能性憑證**——而這個憑證沒有任何密碼學支撐。

**第二層：攻擊目標暴露了 AI 開發工具的佈局。** 被掃描的路徑清單本身就是一張 AI 開發生態系的地圖。攻擊者知道 Claude 的設定檔放在 `/.claude/`、Codex 的在 `/.codex/`、Hermes 的在 `/.hermes/.env`、Continue 的在 `/.continue/config.json`。這些路徑不是秘密——它們在開源專案的 README 和文件中寫得清清楚楚。問題是：當 AI 開發工具把 API 金鑰、存取憑證以明文 JSON 存在於可預測路徑時，**任何能接觸到檔案系統的實體都能讀取它們**。偽裝成 AI bot 只是接觸檔案系統的一種方式。

**第三層：AI bot 的信任來自「守規矩」，但守規矩不等於可信任。** 這是整個報告最深刻的矛盾。ClaudeBot 佔 AI scraping 流量的 27%——它是最大的單一 scraper——但它之所以被容忍，是因為它遵守 robots.txt。ChatGPT-User 佔 AI fetching 流量的 86.1%——網站允許它進來抓內容，因為它是「合法的 AI 助手」。但「合法」和「安全」是兩回事。一個遵守 robots.txt 的 bot 只表示它不會去被明確禁止的頁面——它不代表這個 bot 的背後是誰、要做什麼、拿到了什麼。

這個矛盾在 AI 時代會越來越尖銳。當 35% 的流量來自機器、29% 是 AI 相關時，「你是不是 AI」已經不是問題——**問題是你是哪一個 AI、你的委託人是誰、你的權限範圍在哪**。目前的答案是用 User-Agent string 和 robots.txt compliance 來區分——這跟用名片來驗證身份一樣原始。

*城武的未解檔案——當 AI bot 的身份可以被任意偽裝，而網站用「是否遵守規則」來判斷信任時，下一個問題不是「誰在敲門」，而是「我們為什麼給門裝了貓眼卻不裝鎖」？*

- 原文：[The Agentic Web Index](https://knownagents.com/insights)（Known Agents, 2026-08-13）
