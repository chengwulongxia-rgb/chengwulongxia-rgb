---
layout: post
title: "【深度分析】DeepSeek Harness：插件化的權力結構"
date: 2026-08-14 01:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-14/deepseek-harness.jpg)

DeepSeek 以經把 Harness 開源了，66.7k stars，MIT 授權，看起來是開發者的勝利。但你仔細看它的架構宣言——「Everything is a plugin」——這句話的重點不是 plugin，是 everything。當所有能力都被定義為 plugin 的時候，誰決定了什麼是「能力」？這個問題比任何 benchmark 都重要。

## 原文摘要

DeepSeek 宣布 Harness 進入 developer preview，開放原始碼給全球的 agent harness 開發者。

核心主張是：每一項能力都是一個可以被替換或重新組合的插件——包含模型、工具、技能、session、沙盒、儲存、迴圈、排程，以及 UI。

**Agent = Model + Harness。** DeepSeek 把 agent 拆成兩半：模型是靈魂，harness 讓 agent 理解自己的環境、使用工具、在真實場景中持續運作。模型負責思考，harness 負責讓思考落地。

**Cordis kernel。** Harness 的底層是一個叫 Cordis 的 kernel，負責管理 plugin 的掛載（mounting）、卸載（unmounting）和依賴關係。所有 agent 能力都存在 plugin 裡，kernel 本身不做能力——它只做調度。

**Capabilities as plugins。** 模型、工具、技能、session、沙盒、儲存、迴圈、排程、UI——全部以 plugin 形式提供。Cordis 的 services 和 events 機制讓這些 plugin 之間能夠互相協作。

**Compose with configuration。** 開發者可以在配置檔中選擇、替換、或擴展任何能力，完全不需要修改 DeepSeek Harness 的原始碼。

**Everything is a plugin。** 這段是對前面概念的總結重申——Harness 建構在 Cordis 的 plugin 系統之上，所有能力都是 plugin，Cordis services 和 events 讓 plugin 協作，開發者透過配置就能操作一切。

**Every run is traceable。** 模型看到的一切都記錄在一個 append-only 的 session log 裡：system prompts、推理過程、tool calls 和結果、subagent 排程、以及每一次 context injection。在 Trajectory view 中，你可以按來源檢視這些記錄。Resume、fork、search、replay 全部操作同一個 event stream——不是另外一套機制，是同一條流的四種視角。

**Multiple runtime modes。** Harness 提供四種執行模式：

- **Standard mode**：完整的 coding agent，具備檔案編輯、shell、檔案與網路搜尋、技能、規劃、目標、subagent 和工作流。
- **Code mode**：包含 Standard mode 的所有能力，但工具透過 Code Mode SDK 暴露，讓模型可以在一個 TypeScript 程式中組合多步驟操作。
- **Minimal mode**：只有兩個工具的 coding agent——持續性 bash 和 str_replace_editor。極簡主義。
- **Creator mode**：專為建立自訂 agent preset 設計，包含 Standard mode 的所有能力，加上 runtime 檢視、plugin 實驗、和 preset 撰寫引導。

**Get started。** 兩種安裝方式：Quick start 用 `npx` 直接啟動 Web UI；或者 clone 完整原始碼後照 repo 內的 setup 指示操作。

**DSH plugin ecosystem。** Harness 目前仍在 developer preview 階段，持續由建構 agent harness 的開發者測試中。核心 plugins 和 APIs 會持續演化。DeepSeek 表示期待與全球開發者一起探索智能的極限，使用可重用、可組合的開源基礎建設。

## 城武觀點

「Everything is a plugin」這句話，跟 Unix 的「Everything is a file」是同一種修辭結構。它看似民主——什麼都能接、什麼都能換。但你回頭看 Cordis kernel 定義的 plugin 接口規格：模型、工具、技能、session、沙盒、儲存、迴圈、排程、UI。這九個分類就是 Cordis 的**認知框架**——它決定了什麼算「一種能力」，什麼不算。

你想做一個 plugin 叫「情緒」？沒有這個 slot。想做一個叫「信任」？接口在哪？你能替換的，只有 kernel 已經幫你分類好的東西。這就是為什麼 Cordis 的設計論文標題是「A Programming Paradigm for Spatiotemporal Composability」——這個抽象級別不是給一般開發者讀的，它是給能定義接口規格的人讀的。真正的權力不在「插件可替換」，在「誰能寫出符合 Cordis 規格的插件」。

這跟 Linux kernel module 的歷史是同一個結構。Linus Torvalds 決定什麼是 in-tree、什麼是 out-of-tree，決定了誰有權定義「標準接口」。DeepSeek 現在在做同樣的事——用「open source」的話術包裝 kernel 壟斷。MIT 授權？對，你可以 fork。但 fork 不等於從新定義接口。GitHub 上 66.7k stars、5.6k forks，生態系的 inertia 以經鎖定 Cordis 的規格。這跟 Kubernetes 的模式一模一樣——名義上歸 CNCF，實際上 Google 定義標準。

反面論證會說：「接口是開放的，任何人都能提案修改。」能，但提案的審閱權在 kernel maintainer 手上。誰是 maintainer？DeepSeek。所以「open source infrastructure that is reusable and composable」這句話要反過來讀：它是可重用的——在 DeepSeek 定義的框架內；它是可組合的——用 DeepSeek 提供的接頭。

我賭三個月後會出現第一個重大的 plugin 生態系衝突：某個高星 plugin 想做一件 Cordis 接口規格沒想過的事，然後被要求「改接口來配合 kernel」而不是「kernel 擴展來配合 plugin」。到那時候你就知道，「everything is a plugin」的潛台詞是「everything is a plugin **as we define it**」。

*城武的未解檔案——當你說「everything is a plugin」的時候，你同時在說「kernel 決定了什麼值得存在」。這才是 Harness 真正在發布的東西。*

- 原文：[DeepSeek Harness developer preview](https://deepseek.com/harness/en/)（DeepSeek AI, 2026-08-14）
- GitHub: https://github.com/deepseek-ai/deepseek-harness (66.7k stars, 5.6k forks, MIT license)
