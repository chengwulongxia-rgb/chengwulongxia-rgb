---
layout: post
title: "【公告】DeepSeek 漲價了——本部落格無限期放緩更新"
date: 2026-08-19 00:30:00 +0000
categories: [llm, ai, announcement]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-19/deepseek-price-announcement.jpg)

## 事由：DeepSeek 漲價

DeepSeek 於 8 月 13 日發布 V4-Pro 正式版，同時宣布調整 API 定價。新價格已於 8 月 16 日 16:00 UTC 生效，並引入 peak / off-peak 兩段費率，off-peak 為 peak 的一半。

每百萬 tokens 的價格變化（off-peak、未命中快取）：

| 模型 | 項目 | 舊價 | 新價 | 漲幅 |
|------|------|------|------|------|
| V4-Flash | 輸入（cache miss） | $0.14 | $0.22 | +57% |
| V4-Flash | 輸出 | $0.28 | $0.66 | +136% |
| V4-Pro | 輸入（cache miss） | $0.435 | $0.66 | +52% |
| V4-Pro | 輸出 | $0.87 | $1.98 | +128% |

peak 時段（01:00–04:00、06:00–10:00 UTC）再乘 2。這不是微調——輸出價格直接翻倍以上，等於每次呼叫的帳單直接翻倍。

## 決定：無限期放緩更新

這座部落格每一篇深度文的產生，都建立在大量 token 的閱讀、翻譯、烤問與重寫之上。DeepSeek 這波漲價，讓整個產文成本結構直接翻倍——而 DeepSeek 只是最先動手的那個。

因此即日起，本部落格**無限期放緩更新**：日報、週報、深度系列都不再固定產出，直到以下條件成立為止——

> AI 泡沫化，token 價格大幅下降。

什麼時候算「大幅」？等每百萬 tokens 輸出價格回到 $0.3 以下、或出現真正把成本打下來一個量級的開源模型再說。在那之前，這座塔台的燈先熄一半。

換句話說：當寫一篇分析文的成本，從「一杯咖啡」變成「一頓大餐」，那就該停下來響想，自己到底在為什麼付錢。

- 官方公告：[DeepSeek-V4-Pro GA Release](https://api-docs.deepseek.com/news/news260813)（DeepSeek, 2026-08-13）
- 現行定價：[Models & Pricing](https://api-docs.deepseek.com/quick_start/pricing)