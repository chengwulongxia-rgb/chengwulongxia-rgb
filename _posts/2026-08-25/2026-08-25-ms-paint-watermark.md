---
layout: post
title: "【深度分析】「本地生成」是謊言嗎？Microsoft Paint 把伺服器頒發的 GUID 隱形浮水印，嵌進你本機生成的圖片像素裡"
date: 2026-08-25 01:00:00 +0000
categories: [llm, ai, deep-analysis]
---

![hero]({{ site.baseurl }}/assets/images/2026-08-25/ms_paint_watermark.jpg)

「圖片在我的裝置本機生成，不用上傳，隱私拉滿」——這句廣告詞聽起來很安心，對吧？一位逆向工程師把 Microsoft Paint 跟 Photos 拆開之後發現，事情比你想的複雜：即便圖片真的是你那顆 NPU 在筆電裡算出來的，像素堆裡也早被鑲進一顆由伺服器頒發的隱形 GUID，還有一套 C2PA 憑證在雲端替它加簽。換句話說，你那張「沒離開過本機」的圖，其實從生成那一刻起就帶著一串別人看不到的身份編號。如果你以為「本機生成」等於「圖片的控制權完全在你手上」，那可能要從新想一遍。這篇要講的就是這個落差——以及它為什麼值得讀者多看兩眼。

## 原文摘要

事情的起點是一位叫 Xusheng Li 的逆向工程師對 Paint 的好奇。他先前成功拆過 UCPD、WHESCVC 這些冷門 Windows 功能，這次想搞清楚 Paint 裡那堆 AI 功能到底是怎麼生成圖的。他一開始預期 Paint 就是呼叫遠端 API 產圖，但在 Binary Ninja MCP 與 Codex 的輔助下，很快就發現 Microsoft 其實把本地模型直接塞進了 Windows，作為 Copilot 的一部分。

Paint 的路徑在 `C:\Program Files\WindowsApps\Microsoft.Paint_11.2605.71.0_x64__8wekyb3d8bbwe\PaintApp\`，裡面躺著四個副檔名為 `.onnxe` 的模型檔：`seg.onnxe`（23.1 MB）、`inseg_enc.onnxe`（28.0 MB）、`inseg_dec.onnxe`（16.5 MB）與最大的 `mager.onnxe`（302.4 MB）。其中 `seg.onnxe` 的格式已知，跟字串 `Microsoft_2023` 做 XOR 就會變回正常的 ONNX 檔；另外三個一開始看起來不同，但研究發現 Microsoft 演算法根本沒換，只是換了金鑰——`segapi.dll` 裡有個小小的金鑰註冊表，`ps_enc_key.1.0.80-main` 對應 `"Microsoft_2023"`，而 `ps_enc_key.1.0.81-main` 是一串 4,096 位元組的字母數字字串。解密之後，四個模型都能通過 `onnx.checker.check_model()`：`seg.onnx` 有 1,094 個節點、`inseg_enc.onnx` 1,014 個、`inseg_dec.onnx` 1,133 個、`mager.onnx` 高達 15,284 個節點。

在翻這些檔案時，作者發現了一個 `Watermarker.dll`。這不太意外，因為他操作 Paint 時早就發現有一個「嵌入可見浮水印」的設定——可見浮水印就是圖片右下角一個小小的 Copilot 標誌，完全正常。但出於逆向工程師的直覺，他決定請 AI 分析這個 DLL，看它是不是也在藏隱形浮水印：因為這個檔有 1.67 MB，對這樣「微不足道」的功能來說異常地大（可見浮水印甚至不太需要一個獨立 DLL）。他也坦言，近期 Claude Code 的文字浮水印公告，多少促使他往這個方向想。

接著作者拆出了關鍵函式。可見浮水印由 `AddPerceptibleWatermark` 負責，流程是 `CPBDoc::Save → perceptible-watermark 儲存輔助(bitmap, WatermarkSetting)`，依設定決定要不要問使用者，最後合成 Copilot 標誌上去。但還有一個截然不同的 `WmkWriteWatermark` 函式：`WmkWriteWatermark(output_pixels, payload, payload_length, width, height, stride, input_pixels, pixel_format)`。追蹤呼叫樹之後發現，`WmkWriteWatermark` 是在一次本地的 Stable Diffusion 圖片生成之後被呼叫的——而且如果它失敗了，Paint 會把整次生成當成錯誤處理，而不是回傳一張沒有浮水印的圖給使用者。

問題自然變成：傳進來的 payload 到底是什麼？程式碼要求它必須剛好 16 bytes：`if (payload_length < 16) return -6; if (payload_length > 16) return -5;` 而且函式會忽略長度參數，用寫死的迴圈上界把 payload 拷進來。作者那時還不知道這 16 bytes 是什麼，但它是個 GUID。`WmkWriteWatermark` 不會直接嵌入 GUID，它外面的包裝層會構造一個 18 bytes（144 bits）的訊息：`0x4c || GUID[0..15] || (16 個 GUID bytes 的總和 mod 256)`。

核心編碼器會把可用的圖片尺寸往下取整到 8 的倍數，並維護 144 個計數器（每個 bit 一個），要求每個 bit 至少要能被放置三次。整個編碼流程可以濃縮成：驗證指標/格式/stride/payload 長度；要求寬、高都 ≥ 192；構造 `0x4c || GUID || 校驗和` 的 18 bytes 訊息；把 18 bytes 展開成 144 個 bit；把可用尺寸取整到 8 像素邊界；掃描並挑選合適的圖片區塊；依每個 bit 去量化選定區塊的矩陣值；要求每個 bit 至少成功放置三次（容量不足就回傳 -8）；最後把 RGB 像素重建成輸出緩衝區。嵌入迴圈會在選定的圖片區塊上做小幅的量位化變動，內部含 3×5 矩陣運算、矩陣分解常式，並使用 `24.0`、`0.25`、`0.5`、`0.2` 這些常數——這看起來像一種內容自適應、區塊域的 SVD 式浮水印。

作者證實這就是隱形浮水印。他甚至讓 AI 直接寫程式碼呼叫這個函式，用一張合成的 512×512 BGRA 圖做測試——加完浮水印後，262,144 個像素裡有 193,376 個被改動了。

那這個 GUID 從哪來？在 `WmkWriteWatermark` 的邊界上，payload 只是個指標和長度。包住它的 `PaintAIManager.dll` 有個符號化簽名：`Paint::AI::AddWatermark(Gdiplus::Bitmap& image, winrt::guid const& watermarkId)`——16 bytes 的浮水印 payload 確實就是一個 GUID。繼續往上追，這個 GUID 來自一次網路請求：在 Paint 跑本地圖片模型之前，`AIServices.dll` 會把 prompt 和 style 送到 `https://apsaiservices-a0fqcjc6bzbhgdcd.b02.azurefd.net/v1/paint-cocreator/moderate-prompt`，請求是 JSON，欄位有 `prompt`、`style`、`lastPromptGenerationId`，回應解析器預期收到 `revisedPrompt`、`promptGenerationId`、`watermarkId`、`containsHumanReference`。

靜態分析很好，但作者想看到伺服器的真實回應。他重用了 Paint 自己的已認證 session，把 prompt「a cobalt blue circle above a tiny orange square」送去 moderation 端點，伺服器回了 HTTP 200：

> `{ "revisedPrompt": "a cobalt blue circle above a tiny orange square", "promptGenerationId": "74d9e06b-adea-43ce-85fe-186a26e2e34a", "watermarkId": "83424621-03cb-40e3-9808-a9fae837156d", "containsHumanReference": false }`

他又試了「a portrait of a smiling person wearing a blue hat」，這次回應裡換了一對不同的 GUID，而且 `containsHumanReference` 變成 true——這個欄位是伺服器端對「prompt 是否指涉人類」的分類。整條流程是：`PaintUI → IPromptModerationService → AIServices.ModerateAsync → 建 JSON → HTTPS POST moderate-prompt → ParseModerateResponse → 存 WatermarkId → StableDiffusionHelpers::GenerateAsync → 本地 Stable Diffusion 結果 → Paint::AI::AddWatermark → WmkWriteWatermark → 修改後的 RGB 像素`。換句話說，「本地生成」不代表整個操作都是本地的——Microsoft 收到並審查了 prompt，然後頒發 Paint 嵌進本地生成圖片的唯一 GUID。而且 Paint 會在下次 moderation 請求時把上一次的 `promptGenerationId` 當成 `lastPromptGenerationId` 送回去，讓連續的請求可以明確地被串接起來。

故事還沒完——Paint 不只是改像素。它還把 C2PA Content Credentials 掛到存檔檔案上，程式碼在 `ProvenanceHelper.dll`（背後由 `provenancesdk.dll` 支撐）。本地 Stable Diffusion 路徑是：`本地結果 → AddWatermark → WmkWriteWatermark → AIServices.SignIngredientOnlineAsync(..., promptGenerationId, image) → POST /v1/paint-cocreator/image-sign（imageMetadata 含 PromptGenerationId、GenerationSeed、CreativityLevel、AIFVersion、moderation 分數；imageToSign.jpg）→ ParseProvenanceResponse → 伺服器提供的 C2PA manifest → ProvenanceHelper::InsertManifestIngredient → 帶 C2PA metadata 的最終圖片`。請注意：簽章請求送的是 `PromptGenerationId`，而圖片裡已經含了另一個獨立回傳的 `watermarkId`——兩個值都是 moderation 階段伺服器指派的，所以伺服器可以把這次簽章請求跟已經存在於像素裡的浮水印關聯起來。

作者實地從 Paint 的 Image Creator 存了一張真實圖片，檢查它的 PNG chunks：緊接在 `IHDR` 之後是一個 18,979 bytes 的 `caBX` chunk，內含一份已簽章的 C2PA manifest：

> `{ "c2pa.soft-binding": { "alg": "com.microsoft.invismark.1", "blocks": [ { "scope": "the entire image", "value": "83424621-03cb-40e3-9808-a9fae837156d" } ] }, "c2pa.actions.v2": { "actions": [ { "action": "c2pa.watermarked", "description": "Content watermarked by Microsoft Responsible AI" } ] } }`

解碼後的資訊是：生成器 = Microsoft Responsible AI Provenance、AI 系統 = Azure OpenAI ImageGen、動作 = `c2pa.watermarked`、演算法 = `com.microsoft.invismark.1`、浮水印值 = `83424621-03cb-40e3-9808-a9fae837156d`。伺服器的 `watermarkId`、嵌進像素的識別碼、以及 C2PA `c2pa.soft-binding.value` 是**同一個每次生成都不同**的值。C2PA 稱這為「soft binding」：一個從內容推導出來、或嵌進內容裡的值，讓檔案層級的 manifest 被移除之後，內容仍能被跟它的 provenance 紀錄匹配上；而 Microsoft 對這份斷言做了加密簽章。

那 Paint 為什麼需要本地浮水印？因為它有兩條截然不同的生成路徑。Image Creator 用的是 Azure OpenAI ImageGen——生成、加浮水印、打包 provenance 全都在 Microsoft 的雲端完成，Paint 只要接收一張已經同時含隱形浮水印和 C2PA manifest 的成品圖。但 Cocreator 不一樣：在受支援的 Copilot+ PC 上，NPU 在本機生成圖片，而 Azure 線上服務仍然負責安全檢查。這個功能同時需要 Microsoft 帳號和網路連線，即使實際的 Stable Diffusion 推論是在裝置上執行。流程是：prompt → Microsoft moderation 服務（回傳 revisedPrompt、promptGenerationId、watermarkId）→ revisedPrompt + 草稿 → 本地 NPU 生成 → Watermarker.dll 在本地嵌入 watermarkId → 線上 provenance 簽章 → 最終 C2PA manifest。這大概就是 Paint 需要本地浮水印實作的原因：雲端生成器可以在回傳前就先加浮水印，本地生成器做不到，所以 Paint 得自己改本地的像素。這也解釋了為什麼 `WmkWriteWatermark` 失敗會被當成整次生成失敗，而不是靜默回傳一張沒標記的圖。

還有另一個異常顯眼的跡象，顯示 Microsoft 把存檔路徑設計得繞著 provenance 轉：從 Image Creator 面板直接存生成結果時，Paint 只提供一個格式——PNG。AI 結果套用到 Paint 畫布後，可用格式仍限定在 PNG、JPEG、GIF 和 Paint 自己的 `.paint` 格式——而 BMP，這個 Paint 的經典格式，赫然缺席。這正好對上 C2PA 支援的格式：PNG 用 `caBX` chunk 存 manifest、JPEG 用 APP11 marker segment、GIF 有自己的 C2PA 擴充，`.paint` 則由 Microsoft 掌控。相對地，C2PA 規格明確指出 BMP 是無法在沒有外部 manifest 的情況下嵌入任意 manifest 資料的格式——如果 Paint 允許輸出 BMP，檔案層級的 C2PA manifest 就會消失。這段拆解也丟出一個雲端路徑的安全問題：如果底層的遠端圖片生成端點可以在加浮水印和打包 provenance 之前就先回傳生成結果（或內部有個開關可以跳過這些階段），那有可能拿到一張兩種訊號都沒有的雲端生成圖。這究竟是預期行為、產品 bug、還是安全漏洞，完全取決於 Microsoft 的設計目標和信任邊界——三種可能性都還開著。

Photos 應用也做了一模一樣的事。Microsoft Photos 也有 `Watermarker.dll`，它的 Image Creator 和 Restyle Image 功能背後也有本地的 Stable Diffusion 操作，兩條路都通往同一個浮水印包裝層：Photos Image Creator 走 `PerformSDTextToImageAndWatermarkAsync(..., promptGenerationId)` → 跑本地文字轉圖片模型 → `ApplyWatermark` → 把 `promptGenerationId` 解析成 GUID → 位元組反轉 → `WmkWriteWatermark`；Restyle Image 則走 `PerformSDSketchToImageAndWatermarkAsync` → `ApplyWatermark` → `WmkWriteWatermark`。Photos 和 Paint 之間有個微妙的差異：當浮水印編碼器出錯時，Photos 會記錄「ApplyWatermark encountered error: ... watermark will not be applied.」然後繼續把圖片回傳；Paint 則是直接把浮水印失敗當成整個生成的失敗，不把圖片交還給使用者。

那麼 Microsoft 到底揭露了什麼？它確實在 Image Creator 的支援頁面上揭露了一些周邊內容：內容過濾方面，它說「我們套用內容過濾來防止生成圖片」；它說生成圖片「會包含 C2PA manifest，幫助使用者辨識這是 AI 生成的圖片」；它也說明 Image Creator 使用 Azure 線上服務，並表示會為了防濫用與監控，收集使用者與裝置識別碼以及 prompt。但頁面沒有說明的是：這個 C2PA manifest 裡含有一個標識隱形像素浮水印的 GUID，也沒有說明 Paint 的本地生成路徑是從遠端 prompt moderation 取得這個浮水印 GUID。把這個功能叫做 「Content Credentials」雖然準確，但對一個 Windows 使用者來說，它完全沒有讓這個跟 prompt 綁定的識別碼變得明顯。

結論部分，作者指出：就他所知，這是第一個記錄並分析 Paint 和 Photos 隱形浮水印行為的研究。AI 生成圖片的可見浮水印不是新東西——Microsoft 在 Microsoft 365 和 Bing Image Creator 都記錄過；隱形像素浮水印（如 Google 的 SynthID 和 Bing 的隱藏浮水印）也不是新的。Microsoft 確實揭露了 Paint 使用遠端內容過濾、並加上 C2PA Content Credentials。但新的證據顯示，這份 metadata 不只是無關的檔案層級 AI 標籤：它那份已簽章的 `c2pa.soft-binding` 斷言點名了 Microsoft InvisMark，並記錄了隱形像素浮水印所攜帶的識別碼——檔案層級的 manifest 和像素層級的浮水印，是同一個 provenance 系統的兩層。本地與雲端兩條路徑也解釋了這種不尋常的分工：雲端 Image Creator 能回傳一張已經浮水印並簽章的圖，而 Cocreator 必須在本地 NPU 推論之後，再把伺服器頒發的識別碼嵌進去。兩種情況下，「本地」都不代表離線：prompt 仍然會送去 Microsoft 審查，而完成的本地結果也仍會走線上 provenance 簽章。

作者也點出這可能跟歐盟 AI Act 第 50 條有關——其透明度規則在 2026 年 8 月 2 日生效，要求 AI 生成的內容必須攜帶可偵測、機器可讀的標記，但**不是** prompt 專屬的 GUID。Microsoft 揭露了 C2PA metadata 的存在，但作者找不到任何揭露說明這個伺服器頒發的浮水印 GUID、它與 prompt moderation 的關聯、或它在像素裡的存在。這些細節帶有明顯的隱私與「知情的權利」意涵。最後，作者也指出，改裝 Paint 或 Photos 來繞過 prompt moderation 和浮水印似乎是可行的，但那並不提供任何新能力——任何人都能直接跑 Stable Diffusion，不需要這兩套機制。

## 城武觀點

我的立場很明確：把「本地生成」當成賣點，是架構上的誤導。即便 NPU 真的在你的筆電裡算完那張圖，像素堆裡也以經鑲著一顆來自遠端 moderation server 頒發的 GUID——它同時寫進 C2PA 的 c2pa.soft-binding。所以「本地」從來不是私有或離線，而是「在你裝置生成、但被雲端簽名與追蹤的資產」。追問權力結構：誰握著簽名鑰匙？Microsoft。誰決定每張 AI 圖都必須被標記，而使用者對像素裡的 GUID 一無所知？也是 Microsoft——支援頁面只揭露 C2PA metadata，沒揭露 moderation 綁定的浮水印。這跟 Anthropic 的 text watermark、Google 的 SynthID 是同一套結構：AI 內容的可追溯性壟斷在廠商手裡。我不反對浮水印，我反對的是把「本地=私密」跟「GUID 追蹤」打包進同一個黑箱，卻不對使用者說。

*城武的未解檔案——在你以為「這張圖從沒離開過我的電腦」的那一刻，Microsoft 早就替每一顆像素蓋好章、也替你蓋好章了。*

- 原文：[Microsoft Paint and Photos Embed Server-Issued GUIDs as Invisible Watermarks in Locally-Generated Images](https://xusheng.dev/posts/reversing/mspaint_invisible_watermark/main/)（Xusheng Li, xusheng.dev, 2026-08-25 前後）
