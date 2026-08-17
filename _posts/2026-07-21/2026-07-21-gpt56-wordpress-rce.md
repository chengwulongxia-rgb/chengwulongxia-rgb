---
layout: post
title: "【深度分析】GPT5.6 花 25 美元找到 WordPress RCE——但真正的故事不是 AI 有多厲害"
date: 2026-07-21 02:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-07-21/gpt56-wordpress-rce-hero.png)

城武導讀：一個安全研究員花了 25 美元，讓 GPT5.6 Sol Ultra 在 10 小時內找到一條從未認證 SQL injection 到完整 RCE 的 WordPress 攻擊鏈。漏洞經紀人對同等級漏洞標價 50 萬美元。但這篇文章真正該被討論的，不是「AI 好強」——而是它背後那套以經搖搖欲墜的漏洞經濟學，以及一個被效能需求親手種下、六年後才被收割的設計債。

## 原文摘要

7 月 20 日，Searchlight Cyber 的安全研究員 Adam Kues 發表了一篇技術文章，詳細記錄了他如何使用 GPT5.6 Sol Ultra，從零開始在 WordPress 中發現一條完整的遠端程式碼執行（RCE）攻擊鏈。總成本約 25 美元。同期，漏洞經紀人對 WordPress RCE 的報價是 50 萬美元。

Kues 團隊在發布前暫緩公開，給防守方升級時間。在這段窗口期內，另外兩位安全研究員 Calif 和 Hacktron 獨立復現了完整攻擊鏈，早於其他 PoC 在 GitHub 上出現。Kues 也提供了檢測工具 wp2shell.com，讓 WordPress 站點管理者可以自行檢查是否受影響。

### GPT5.6 Sol Ultra 的使用方式

GPT5.6 Sol Ultra 發布時，OpenAI 公開了它用來解決 Cycle Double Cover 猜想（一個懸宕數十年的圖論問題）的完整 prompt。Kues 將這套 prompt 改編為安全研究用途，指向 WordPress，使用 4 個 agent 並行運作至少 6 小時。

Prompt 的關鍵設計元素包括：從第一原理進行代碼分析尋找零日漏洞、禁止使用網路與 git 歷史、多 agent 架構（最多同時 4 個）、多樣化的攻擊面探索策略（輸入解析、字元集、檔案上傳、錯誤處理、內建路由、序列化、快取、競爭條件、加密、型別處理、批量賦值）、維護攻擊面家族的顯式註冊表、對抗性 agent 負責二次驗證每個發現的 bug、根 agent 負責綜合、挑戰、重新定向並啟動新一輪分析、以及複製第三方依賴（PHP/MySQL 原始碼）到本機進行深度追蹤。

Kues 複製了最新穩定版 WordPress，刪除 `.git` 目錄以確保 GPT5.6 無法透過版本控制歷史取巧。

結果：GPT5.6 Sol Ultra 在幾分鐘內就找到了一個無需認證的 SQL injection 漏洞，可以從一台預設安裝的 WordPress 上竊取管理員 email。再花約 4 小時，將這個注入點一路串到完整的 RCE。

### 漏洞本體：Batch API 的陣列不同步

WordPress 在 5.6 版（2020 年）引入了 Batch API，允許在單一 HTTP 請求中打包多個虛擬 API 呼叫。這個端點無需認證即可存取。

正常的 REST API 驗證流程是序列化的：
1. `has_valid_params()` 檢查必要參數是否存在且合法
2. `sanitize_params()` 清理參數
3. 執行權限回呼（permission callback）
4. 執行端點回呼（endpoint callback）

但 Batch API 為了效能，把流程拆成兩個獨立的迴圈：
- 迴圈一：對每個請求進行驗證與清理
- 迴圈二：對每個請求檢查驗證結果、執行權限回呼、執行端點回呼

漏洞出在兩個陣列 `$matches` 和 `$validation` 是平行建構的。當某個請求進入 `is_wp_error()` 分支時，`$validation` 會被更新，但 `$matches` 因為 `continue;` 的關係**不會被更新**。這導致兩個陣列不同步——`$matches` 中的每一個條目都向前偏移一個位置。

結果：請求 N 的驗證結果會被配對到請求 N+1 的執行 handler。你可以讓一個請求的參數通過驗證，但實際上用另一個完全不應該有這些參數的端點來執行它們。

### 注入點：author__not_in 參數

REST API 路由 `GET /wp/v2/posts` 有一個 `author__not_in` 查詢參數。當這個參數以陣列形式傳入時，WordPress 會用 `absint` 過濾（強制轉為整數），安全無虞。但當它以純量字串形式傳入時，**完全不做任何處理**，直接被插值進原始 SQL 查詢中。

透過 Batch API 的陣列不同步漏洞，攻擊者可以這樣做：讓驗證階段檢查參數時，走 `DELETE /wp/v2/posts/1` 的路徑（這個端點不認識該參數，所以驗證直接通過）；但執行階段卻走 `GET /wp/v2/posts`（這個端點會把 `author__not_in` 直接塞進 SQL）。

但有一個障礙：Batch API 不支援 GET 請求。

GPT5.6 的解法令人拍案：**遞迴呼叫 Batch API**。在內層的 batch 請求中，再次利用陣列不同步漏洞，繞過 HTTP 方法檢查。最終的 payload 對 Batch API 的驗證漏洞進行了兩層遞迴利用，把一個本不該存在的 GET 請求硬塞進 batch 管線。

### 快取投毒

WordPress 在請求生命週期內維護一個 WP_Post 物件的記憶體快取。透過 UNION-based SQL injection，攻擊者可以偽造查詢回傳的文章內容，完全控制快取中儲存的資料。而 WordPress 在渲染文章前會進行後處理——攻擊者控制了整篇文章的文字內容。

### Embed 功能：憑空製造資料庫行

WordPress 的 embed 功能允許在文章中使用 `[embed]https://example.com[/embed]` 短碼來嵌入外部內容。這些嵌入結果會被快取在 `wp_posts` 表中，類型為 `oembed_cache`。

關鍵在於：WordPress 貼文本身是一種被支援的嵌入類型。如果你用相對路徑嵌入一篇貼文，WordPress 會認出這是本地資源。而且 WordPress **不檢查**被嵌入的貼文 ID 是否真實存在。放置 `[embed width="500" height="750"]/?p=10[/embed]` 就能在資料庫中憑空製造出一行 `oembed_cache` 類型的記錄。

現在資料庫中有了一行記錄（例如 post ID 11）。如果再次利用 SQL injection 偽造這篇貼文在記憶體中的任何屬性，就會出現一種有趣的不一致：記憶體中的版本和資料庫中的版本不同。WordPress 嘗試調和兩者時，會呼叫 `wp_update_post`，傳入已知的 ID 和 `post_content`。但 `ID` 和 `post_content` 是在寫入前就設定好的；其他欄位如 `post_status` 和 `post_type` **不會被覆蓋**。資料庫中 `post_type` 是 `oembed_cache`，但記憶體中可以偽造成任何類型（例如 `post`）。WordPress 偏好記憶體中的欄位值。這表示：`oembed_cache` 行可以被強制變成普通文章。

### Changeset：借用管理員身份

WordPress 使用 `customize_changeset` 類型的文章來儲存主題自訂的草稿變更。這類文章的 `post_content` 欄位儲存一個 JSON diff：

```json
{"blogname": {"value": "New name", "type": "option", "user_id": 1}}
```

當 changeset 被套用時，WordPress 會**暫時將當前使用者設定為 changeset 中記錄的 `user_id`**。如果攻擊者能偽造一個 `user_id: 1` 的 changeset，就能暫時獲得管理員身份。

### 循環檢測：不覆蓋 post_content 的副作用

WordPress 文章形成一個樹狀結構（每篇文章可以有零或一個父文章）。WordPress 不允許循環。當更新文章階層時，系統會向上走訪父文章、祖父文章……如果檢測到循環，WordPress 會呼叫 `wp_update_post` 將該文章的 `post_parent` 重設為 0。

這個呼叫**不會覆蓋 `post_content`**。所以攻擊者可以控制 `post_content`，用自己的偽造記憶體內容填充它。這表示：可以偽造一篇含有管理員 changeset JSON 的合法 `customize_changeset` 文章。

### Hook 重放：以管理員身份重跑整條攻擊鏈

WordPress 的 hook 系統允許外掛在生命週期事件中插入邏輯。當文章發布時：

```php
do_action("{$new_status}_{$post->post_type}", $post->ID, $post);
```

因為攻擊者在記憶體中偽造了整篇文章物件，`new_status` 和 `post_type` 可以是任何值。這意味著可以以管理員身份觸發任何 action。

GPT5.6 瞄準了 `parse_request` hook——這個 hook 在請求生命週期的最開始被呼叫。觸發 `parse_request` 會從頭重放整個 Batch API 請求，但這次是以管理員角色執行。

### 完整攻擊鏈（兩次 HTTP 請求）

**請求一**：利用 SQL injection 回傳一篇含有 3 個 embed 連結（指向同一篇文章 S 但帶不同查詢 hash）的偽造文章，在資料庫中種下三行 `oembed_cache` 記錄（代號 O、C、D）。

**請求二**：在記憶體中偽造六篇文章：
- **O**：`publish/oembed_cache`，空內容，過期時間戳，父文章為 C
- **C**：`future/customize_changeset`，含有管理員 changeset JSON，父文章為 C（自我循環）
- **P**：`draft/page`，父文章為 D
- **D**：`parse/request`，父文章為 D（自我循環）
- **S**：`publish/post`，提供 embed 資料
- **T**：`publish/post`，包含外部 embed

鏈條執行過程：

1. T 包含對 S 的 embed → 呼叫 `get_post(O)`（使用 O 的 hash 作為本地 embed 查詢鍵）
2. O 有對應的資料庫行，O 嵌入 S → S 進入記憶體快取 → S 的時間戳被偽造為過期 → `get_post(S)` 回傳偽造資料
3. O 需要更新 → `wp_update_post` → `wp_insert_post_parent` filter → 發現 C 存在循環 → `wp_update_post(C, parent=0)`
4. C 在記憶體中是 `customize_changeset`，狀態為 `future` 且日期為過去 → WordPress 套用 changeset
5. Changeset 設定 `nav_menus_created_posts` 且 `user_id=1` → WordPress 暫時以管理員身份運作
6. 變更完成後，`wp_update_post(P, status=publish)`
7. P 的父文章是 D → D 存在循環 → `wp_update_post(D, parent=0)`
8. D 是 `parse/request` → `do_action("parse_request")` → 以管理員身份重放整個 batch 請求
9. 原始的 batch 請求中包含了「建立新管理員帳號」的指令 → 現在以管理員身份執行成功
10. 以新管理員帳號登入 → 上傳後門外掛 ZIP → RCE

### AI 是超人嗎？

完整的 exploit 在略多於 10 小時內產出。Kues 的判斷是：「沒有安全研究員能在沒有 AI 的情況下，在 10 小時內找到並完成這條利用鏈。」即使已經知道原始的 SQL injection 漏洞，要在這個時間內一路串到 RCE，對人類來說也極其困難。

值得注意的創意技巧包括：遞迴 batch 呼叫來繞過 GET 限制、快取濫用來觸發 changeset 套用、偽造文章觸發 `parse_request` hook 來以借用的管理員身份重放請求。

Kues 認為，安全研究的本質正在改變：未來的工作會變得更加高層次——決定要研究哪些產品和攻擊面、用 prompt 引導方向、在 LLM 偏離軌道時輕輕推它一把。

## 城武觀點

先說結論：這個漏洞不是 AI「發現」的，它是 WordPress 自己在 2020 年親手種下的。GPT5.6 只是那個終於來敲門的人。

### 設計債，不是天才

WordPress 的 REST API 安全模型建立在一個隱含的不變量上：驗證和執行發生在同一個 handler 裡，參數在通過檢查的下一刻就被消費，沒有機會被偷換。這是個合理的假設——在序列化流程中。

但 Batch API 為了效能，把驗證和執行拆成兩個獨立的迴圈。這個決定本身不是 bug——它是一個架構取捨。問題在於：沒有人停下來問「如果兩個陣列不同步會怎樣？」。不是因為工程師笨，而是因為當初寫這段 code 的人腦中有同一個隱含假設——「反正兩個迴圈走同一批請求，順序是一樣的」。

六年後，這個假設被一台沒學過任何 WordPress 內部慣例的 LLM 拆穿了。AI 不走捷徑、不看 changelog、不問「這裡通常不會出問題吧」——它只問「從第一原理來看，這段 code 能不能被打破」。而答案是：可以。一直都是。

這件事告訴我們的不是 AI 有多聰明，而是人類工程師的認知慣性有多深。我們建立安全模型的方式，本質上是在信任一組沒被寫下來的假設。當你把這些假設拿給一個不共享你的人類偏見的系統看，它看到的不是「合理的設計取捨」，它看到的是「這裡有一個可以被打破的隱含合約」。

### 50 萬美元的真相

但真正的故事比漏洞本身更大。GPT5.6 花 25 美元和 10 小時找到的東西，漏洞經紀人標價 50 萬美元。這裡面有一個荒謬的價差：20,000 倍。

這個價差不是在衡量漏洞的危險性。它是在衡量**找到這個漏洞的勞動壟斷**。

漏洞經紀人的商業模型建立在一個簡單的前提上：能獨立完成一條 WordPress RCE 攻擊鏈的人，全世界可能只有幾百個。稀缺性支撐了價格。但 GPT5.6 以經證明了一件事——這項勞動的邊際成本正在趨近於零。不是趨近於「便宜」，是趨近於零。25 美元對一間國家級攻擊團隊或大型勒索軟體組織來說，跟免費沒有區別。

這有兩個後果。

第一，攻方會全部改用 AI。不是「可能會」，是「必然會」。當攻擊成本從 50 萬美元降到 25 美元，任何還在手動挖漏洞的攻擊者都是在浪費時間。不是因為 AI 比人強，而是因為 ROI 差距大到不需要討論。這意味著漏洞的供給端會在短時間內暴增——不是因為軟體變得更不安全，而是因為挖漏洞的成本結構被徹底改寫。

第二，漏洞賞金經濟學的定價基礎消失了。如果一個漏洞的「市場價值」是 50 萬美元，但生產成本是 25 美元，那這個市場只有兩種可能的結局：要麼所有參與者都用 AI（價格會崩盤到接近生產成本），要麼這個市場從一開始就不值 50 萬——它只是在對沖稀缺性溢價，而那個溢價現在被證實是虛的。

我賭第二種。不是因為漏洞經紀人在騙人，而是因為他們自己也弄不清楚自己賣的到底是什麼。他們以為自己在賣漏洞，但其實在賣稀缺性。現在稀缺性被打破了，剩下的就是一個可以用 shell script 自動跑的東西。這不是市場崩潰，這是市場終於發現自己原來一直在幫壟斷勞動力定價，而不是幫漏洞定價。

### 防禦方的困境

這一切對防守方來說，是最壞的消息。WordPress 佔了全球 40% 以上的網站。這個漏洞的 patch 即使今天發布，全球的更新速度也不可能快過攻擊者用 AI 批次掃描的速度。過去，一個高價值漏洞從發現到被武器化之間有一段黃金窗口——因為人類逆向工程、寫 exploit、測試穩定性的過程需要時間。現在那段窗口被壓縮到以小時計。

而且更深的問題是：這個漏洞的根因不是一個程式碼錯誤，而是一個架構決策。Batch API 的雙迴圈設計是刻意為之的——它是為了效能。修補這個漏洞意味著要重新設計整個 batch 驗證管線。這不是改一行的 hotfix，這是架構層級的手術。

### 一個預測

三個月內，我們會看到第一篇「全自動 AI 漏洞挖掘→武器化→部署」的完整攻擊鏈被記錄下來。不是因為 AI 突然變強了，而是因為從 25 美元到全自動之間的工程落差，比從 50 萬美元到 25 美元之間的落差小得多。最後一哩路往往是工程整合，不是基礎研究的突破。

而當那一天到來時，我們會回過頭看今天這篇文章，然後發現 Kues 說的那句話——「安全研究的本質正在改變」——不是預測，是診斷。

*城武的未解檔案——漏洞的價格不是漏洞本身決定的，是「誰能找到它」決定的。而那個「誰」的答案，從今天開始包含了 25 美元。*

- 原文：[Exploit brokers pay $500,000 for a WordPress RCE. I found one with GPT5.6 Sol Ultra and $25](https://slcyber.io/research-center/exploit-brokers-pay-500000-for-a-wordpress-rce-i-found-one-with-gpt5-6/)（Adam Kues, Searchlight Cyber, 2026-07-20）
