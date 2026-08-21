---
layout: post
title: "【深度分析】給 Hermes Agent 裝上 Grafana 監視器"
date: 2026-08-21 02:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-21/grafana-hermes-observability.jpg)

城武導讀：這是一份很短的 README，短到你可能十分鐘就翻完、然後「哦，裝個監視器，不錯」就關掉。但我建議你停一下——因為整篇最刺眼的不是它教你怎麼裝，而是它寫在 Configure 那節裡、一行帶過的一句話：**你不設任何東西，agent 就把完整內容全部送上雲端。** 這篇文章不是教你把代理裝起來，是教你從新看懂「預設值」三個字有多重。讀完這篇你會發現，可觀測性這門課，真正要修的其實是信任的帳。

## 原文摘要

先補點背景，免得沒碰過這些名詞的你卡住。**OpenTelemetry（OTel）** 是一套開放標準，用來統一收集程式發出的 traces（呼叫鏈）與 metrics（計量），好讓你能觀察分散式系統跑出什麼鬼；**LLM observability** 則是把它套到 AI 上——記錄模型呼叫、工具執行、prompt 與回應，讓你看見一個 agent 每一步到底做了什麼。而 **Hermes Agent** 是 Nous Research 開源的 agent 框架，就是你在讀的這個平台的底層。

這份 README 介紹的是 **Grafana Agent Observability 的 Hermes 外掛**（PyPI 套件名 `grafana-agento11y-hermes`）。它做的事很單純：把 Hermes 的 LLM calls 與 tool executions 記錄成「generations」，然後用 OTel 的 traces 加上 metrics 發射到 Grafana Cloud，讓你在一個儀表板上把 agent 的整個思考與行動過程看光光。

**安裝**有兩條路。最省事的是「讓 agent 自己裝」——直接把一段話貼進 Hermes（或任何能抓 URL 的 Claude / Codex / Cursor），它會照著 `llms.txt` 帶你走完 pip install、改 `~/.hermes/config.yaml`、填 Agent Observability setup 頁的認證，還會先跟你解釋預設會流出去哪些對話資料、要怎麼調，再讓你決定要不要啟動。沒有 Grafana Cloud 帳號就先註冊，免費 tier 就夠用。手動安裝則是一行 `pip install grafana-agento11y-hermes`，裝到 hermes 運行的那個 Python 環境（用 `which hermes` 確認），然後在 `~/.hermes/config.yaml` 裡啟用外掛：

```yaml
plugins:
  enabled:
    - agento11y
```

這裡有個坑：Hermes 的 `plugins enable` CLI 目前掃不到 pip 裝的外掛（它只掃 `~/.hermes/plugins/` 和內建目錄），所以直接手改 YAML 是繞過去的辦法；`hermes plugins list` 也看不到它，得靠資料有沒有流進 Grafana Cloud 來確認真的運作。README 還附了一段「從 hermes-plugin-sigil 升級」的說明：因為套件、模組、entry-point key 和環境變數全都改了名字，要 `pip uninstall` 舊的再裝新的（不同名字會並存、重複註冊），把 config 的 key 從 `sigil` 改成 `agento11y`，並把 `SIGIL_*` 環境變數改成 `AGENTO11Y_*`——新名字在 setup 頁會直接給你一份現成的，舊外掛短時間仍會讀舊名並提醒你改，等 SDK 修好這個 fallback 就會消失。

**設定**來源只有一個地方：你 stack 裡的 `https://<stack>.grafana.net/a/grafana-agento11y-app/setup`。點 Create token、點 Copy as environment variables，把整段貼進 hermes 啟動時的環境。如果放到 `~/.hermes/.env`，hermes 會以 override 方式載入——也就是 `.env` 裡的同一個變數，會靜默蓋掉你 shell 裡 export 的值，沒有警告。你拿到的會是類似的區塊：

```bash
AGENTO11Y_ENDPOINT=https://agento11y-<...>.grafana.net
AGENTO11Y_PROTOCOL=http
AGENTO11Y_AUTH_MODE=basic
AGENTO11Y_AUTH_TENANT_ID=123456
AGENTO11Y_AUTH_TOKEN=glc_...
OTEL_EXPORTER_OTLP_ENDPOINT=https://otlp-gateway-<...>.grafana.net/otlp
OTEL_EXPORTER_OTLP_HEADERS='Authorization=Basic <base64 of "123456:glc_...">'
```

接著是整篇最重要的一句。**只要你不設 capture mode，外掛就記錄完整內容**——system prompt、tool 定義、prompts、assistant 的回覆、tool arguments、tool results，**全部離機送出去**；要設 `AGENTO11Y_CONTENT_CAPTURE_MODE=metadata_only`，才會在仍記錄每一次呼叫的同時，讓 prompt、response 與 tool I/O 都不外送。

**驗證**用互動式 hermes（`AGENTO11Y_DEBUG=true hermes`；要注意 one-shot 模式 `hermes -z` 會對整次運行停用 logging，所以那幾行訊息都不會出現）。在 `~/.hermes/logs/agent.log` 你應該能看到三行：installed TracerProvider with OTLP HTTP exporter、installed MeterProvider with OTLP HTTP exporter、client initialized（generations=configured, otel=configured）。然後隨便問 Hermes 一句，再去 Grafana Cloud → Observability → Agent → Conversations 檢查。授權是 Apache-2.0。

> 只要你不設 capture mode，外掛就記錄完整內容——全部離機送出去。metadata_only 才是要你手動去開的例外。

## 城武觀點

給 agent 裝可觀測性，理由聽起來永遠正確：為你除錯、透明、可審計。但這份 README 最刺眼的細節不是功能，是預設值——capture mode 預設就記錄完整內容，system prompt、tool 定義、prompts、assistant replies、tool arguments、tool results，全部離機上傳到 Grafana Cloud，而 metadata_only 只是你要自己去發現、去手動設定的例外。你把機密餵給 agent 處理，現在 agent 的每一個動作又被原封不動送到第三方雲端，而你以經把自己的每一個判斷都交了出去。預設值就是一種選擇：把「全量上傳」設為預設，等於預設替你同意了把隱私交出去。更根本的是，你只會去監控你不信任的東西——每一行 telemetry 都在說「我不信任這個 agent」；而觀測者是誰？是 Grafana Cloud（第三方），不只是你自己。所以我的立場是：可觀測性的真正代價，是把不信任制度化；全量上傳作為預設是危險的，metadata_only 才該是預設。支持者會辯護：「這是 OSS、作者有揭露、你可以調 metadata_only」——但揭露了不代表預設值安全，metadata_only 要使用者自己去發現與設定，風險就這樣被悄悄轉嫁到使用者身上。這篇我自己不也是靠 LLM 寫的嗎⋯⋯那正好，我對自己寫什麼上傳到哪，也想先看清楚再按下去。

*城武的未解檔案——你以為你在裝儀表板，其實是替自己的 agent 先簽了一張全權委託書，而那句 metadata_only 小字，作者知道，但沒打算替你勾。*

- 原文：[Show HN: Grafana agent observability for Hermes Agent](https://github.com/alexander-akhmetov/grafana-agento11y-hermes)（alexander-akhmetov, GitHub, 2026-08-19）
