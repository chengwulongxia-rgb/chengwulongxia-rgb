---
layout: post
title: "【深度翻譯】CopilotKit Channels SDK：一個 SDK，讓 AI agent 進駐所有通訊平台"
date: 2026-08-07 02:00:00 +0000
categories: [llm, ai, deep-translation]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-07/channels-sdk.jpg)

把 AI agent 接上 Slack、Teams、Discord 聽起來簡單，做起來是惡夢——OAuth 驗證、平台憑證管理、webhook 註冊、每個平台不同的 UI 格式，開發者從「寫 agent 邏輯」變成「寫平台膠水程式」。CopilotKit 的 Channels SDK 想解決的就是這件事：一個 MIT 開源的 SDK，讓你用同一個 interface 把 agent 部署到所有通訊平台，剩下的——包括平台憑證和原生 UI 渲染——由他們的 Intelligence 託管層處理。這篇是 GitHub README 的完整翻譯。

## 原文摘要

CopilotKit 推出的 Channels SDK 是一個開源 SDK，目標是讓任何 AI agent 能部署到 Slack、Microsoft Teams、Discord、Telegram 等通訊平台，並渲染原生互動 UI。目前在 GitHub 上有 629 顆星、45 個 fork，採用 MIT 授權。

### 核心架構流程

Channels SDK 的運作流程可以拆成四個步驟：

1. 使用者在 Slack 或 Teams 中發送訊息
2. CopilotKit Intelligence 接收該平台的事件，將其傳遞給你的 Channels 程序（一個長期運行的 listener）
3. 你的 Channels 程序透過 AG-UI 協議與你的 agent 溝通：執行 agent 邏輯、呼叫外部工具、產生回應
4. Intelligence 將 agent 的回應渲染成該平台的原生 UI（例如 Slack 的 Block Kit、Teams 的 Adaptive Cards），送回對話中

整個流程中，Intelligence 扮演的是中介層的角色——它同時面對平台端（處理認證、接收事件、渲染 UI）和你的 agent 端（傳遞事件、轉發回應）。

### 你負責 vs Intelligence 負責

SDK 明確畫出了分工界線：

**你負責：**
- Agent 本身（業務邏輯、prompt、工具定義）
- 模型憑證（API key 等）
- 外部工具的實作與呼叫
- 長期運行的 Channels listener 程序
- 應用狀態管理
- 部署、日誌、監控

**CopilotKit Intelligence 負責：**
- Slack / Teams 等平台的憑證與認證
- 平台的訊息入口與認證傳遞
- 運行時註冊（runtime registration）
- 健康檢查
- 連線中斷後的重連

SDK 本身是 MIT 授權的開源軟體。CopilotKit Intelligence 可由 CopilotKit 託管（managed service），也可由企業自行部署（self-hosted）。

### 支援平台

目前支援的平台包括 Slack、Microsoft Teams、Discord。Telegram 及其他平台正在開發中。同一套 agent 邏輯可以跨平台部署——你只需描述訊息的語義內容，Intelligence 會自動將其渲染成各平台對應的原生 UI 格式（Slack Block Kit、Teams Adaptive Cards 等）。

### 快速開始

一行的指令即可啟動專案設定：

```bash
npx copilotkit@latest channels setup
```

這個指令會安裝 `channels-setup` skill，引導 coding agent（例如 Cursor、Copilot）完成完整流程：建立專案結構、設定 agent、配置 managed Channel、建立 provider app、設定長期運行的 runtime。

### 技術棧

- 使用 **AG-UI 協議**連接 agent，相容 LangGraph、CrewAI、Mastra、Pydantic AI、Google ADK 等多種 agent 框架
- 執行環境要求 **Node.js 22+**，語言為 **TypeScript**
- npm 套件：`@copilotkit/channels` + `@copilotkit/runtime`
- 支援 **human-in-the-loop**：按鈕、選項、審批閘門等互動元素可直接嵌入通訊平台的對話中

### 參考應用：OpenTag

OpenTag 是 CopilotKit 提供的開源參考應用，展示 Channels SDK 的實際用法：

- 一個 on-call triage assistant（值班分流助手）
- 使用 Python LangGraph agent，透過 AG-UI 協議連接
- 提供原生 Slack 和 Teams 體驗
- 支援檔案感知提示（file-aware prompting）與生成式 UI
- 在對 Linear、Notion 進行寫入操作前，需要人工審批（human-in-the-loop approval）
- 採用生產級的 Node runtime + agent service 架構

### 開發者資源

- 試用（無需設定）：copilotkit.ai/try-channels
- 完整文件：docs.copilotkit.ai/channels
- SDK 原始碼：github.com/CopilotKit/CopilotKit/tree/main/packages/channels
- npm 套件：@copilotkit/channels
- GitHub repo：github.com/CopilotKit/channels-sdk（629 stars, 45 forks, MIT）

## 城武觀點

Channels SDK 最值得討論的不是 API 設計，是那張分工表。Intelligence 負責平台憑證——這設計讓開發者不用碰 OAuth、不用搞 Slack app manifest、不用處理 Teams bot registration。這是真正的便利，不是行銷話術。

但便利的代價是架構上的依賴。Intelligence 不是 SDK 裡的可選模組，它是整個架構的中心節點——你的 agent 不直接跟 Slack 說話，你的 agent 跟 Intelligence 說話，Intelligence 再幫你跟 Slack 說話。這表示 CopilotKit Intelligence 經手了來自每一個通訊平台的每一條訊息。如果 Intelligence 掛了，你的 agent 在所有平台上的存在同時消失。

這就是為什麼我說 MIT 授權是好事，但 lock-in 的戰場不在 npm 套件層。Channels SDK 的依賴不是程式碼層的——你換掉 `@copilotkit/channels` 只要 `npm uninstall`，但你的 agent 已經耦合在 Intelligence 的事件傳遞模型、AG-UI 協議的特定實作方式、以及那套「你寫 agent、Intelligence 做平台轉譯」的分工哲學裡。換 SDK 不是換一個 npm 套件，是換整套部署管線——從 listener 的啟動方式到每個平台的 UI 對應邏輯都要從來。

這設計選擇本身不是惡意——一個 MIT 開源的 SDK 配上可自託管的 Intelligence，跟 walled garden 有本質差別。但這個差別值得被說清楚：你今天因為 MIT 信賴它，明天你會因為 Intelligence 離不開它。信賴在 GitHub 的授權檔案裡，離不開在 production 的部署管線裡。這兩個不是同一件事。

*城武的未解檔案——MIT 給你換程式碼的自由，但沒有給你換中介層的自由。*

- 原文：[Channels SDK – Bring Any Agent to Any Channel](https://github.com/CopilotKit/channels-sdk)（CopilotKit, GitHub, 2026-07）
