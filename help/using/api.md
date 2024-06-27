---
title: '"[!DNL Asset Compute Service] HTTP API」'
description: '"[!DNL Asset Compute Service] HTTP API可建立自訂應用程式。」'
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '2862'
ht-degree: 2%

---

# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API的使用僅限於開發目的。 API在開發自訂應用程式時作為內容提供。 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] 使用此API將處理資訊傳遞至自訂應用程式。 如需詳細資訊，請參閱 [使用資產微服務和處理設定檔](https://experienceleague.adobe.com/zh-hant/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] 僅供搭配使用 [!DNL Experience Manager] as a [!DNL Cloud Service].

的任何使用者端 [!DNL Asset Compute Service] HTTP API必須依照此高階流程：

1. 使用者端已布建為 [!DNL Adobe Developer Console] IMS組織中的專案。 每個單獨的使用者端（系統或環境）需要其自己的單獨專案，以分隔事件資料流。

1. 使用者端使用為技術帳戶產生存取權杖 [JWT （服務帳戶）驗證](https://developer.adobe.com/developer-console/docs/guides/).

1. 使用者端呼叫 [`/register`](#register) 僅擷取一次日誌URL。

1. 使用者端呼叫 [`/process`](#process-request) 針對要產生轉譯的每個資產。 呼叫為非同步。

1. 使用者端定期輪詢日誌 [接收事件](#asynchronous-events). 成功處理轉譯時，它會接收每個請求轉譯的事件(`rendition_created` 事件型別)或是否有錯誤(`rendition_failed` 事件型別)。

此 [adobe-asset-compute-client](https://github.com/adobe/asset-compute-client) 模組可讓您在Node.js程式碼中輕鬆使用API。

## 驗證和授權 {#authentication-and-authorization}

所有API都需要存取權杖驗證。 請求必須設定以下標頭：

1. `Authorization` 附有持有人權杖的標題，這是技術帳戶權杖，接收於 [JWT交換](https://developer.adobe.com/developer-console/docs/guides/) 來自Adobe Developer Console專案。 此 [範圍](#scopes) 記錄如下。

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` 標頭包含IMS組織ID。

1. `x-api-key` 的使用者端ID來自 [!DNL Adobe Developers Console] 專案。

### 範圍 {#scopes}

確定以下存取權杖範圍：

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

這些範圍需要 [!DNL Adobe Developer Console] 要訂閱的專案 `Asset Compute`， `I/O Events`、和 `I/O Management API` 服務。 個別範圍的劃分如下：

* 基本
   * 範圍： `openid,AdobeID`

* asset compute
   * metascope： `asset_compute_meta`
   * 範圍： `asset_compute,read_organizations`

* Adobe [!DNL `I/O Events`]
   * metascope： `event_receiver_api`
   * 範圍： `event_receiver,event_receiver_api`

* Adobe [!DNL `I/O Management API`]
   * metascope： `ent_adobeio_sdk`
   * 範圍： `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 註冊 {#register}

每個使用者端 [!DNL Asset Compute service]  — 不重複 [!DNL Adobe Developer Console] 訂閱服務的專案 — 必須 [註冊](#register-request) 進行處理要求之前。 註冊步驟會傳回從轉譯處理中擷取非同步事件所需的唯一事件日誌。

在生命週期結束時，使用者端可以 [取消註冊](#unregister-request).

### 註冊要求 {#register-request}

此API呼叫會設定 [!DNL Asset Compute] 使用者端並提供事件日誌URL。 此程式是等冪操作，每個使用者端只需要呼叫一次。 可再次呼叫它以擷取日誌URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/register` |
| 頁首 `Authorization` | 全部 [授權相關標頭](#authentication-and-authorization). |
| 頁首 `x-request-id` | 選擇性，由使用者端設定，用於跨系統處理請求的唯一端對端識別碼。 |
| 要求內文 | 必須為空白。 |

### 登入回應 {#register-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME型別 | `application/json` |
| 頁首 `X-Request-Id` | 與 `X-Request-Id` 要求標頭或唯一產生的標頭。 用於識別跨系統的請求或支援請求，或兩者皆使用。 |
| 回應內文 | 具有的JSON物件 `journal`， `ok`，或 `requestId` 欄位。 |

HTTP狀態碼為：

* **200項成功**：要求成功時。 此 `journal` URL會收到透過以下方式啟動的非同步處理結果的通知： `/process`. 它警示屬於 `rendition_created` 成功完成時的事件，或 `rendition_failed` 事件（如果處理失敗）。

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401未獲授權**：當請求無效時發生 [authentication](#authentication-and-authorization). 例如，存取權杖無效或API金鑰無效。

* **403禁止**：當請求無效時發生 [authorization](#authentication-and-authorization). 例如，可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）未訂閱所有必要服務。

* **429太多請求**：發生於此使用者端或系統多載時。 使用者端應使用 [指數輪詢](https://en.wikipedia.org/wiki/Exponential_backoff). 內文是空的。
* **4xx錯誤**：發生任何其他使用者端錯誤且註冊失敗時。 通常會傳回此類的JSON回應，但並非所有錯誤都有保證：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx錯誤**：發生於發生任何其他伺服器端錯誤且註冊失敗時。 通常會傳回此類的JSON回應，但並非所有錯誤都有保證：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### 取消註冊要求 {#unregister-request}

此API呼叫會取消登入 [!DNL Asset Compute] 使用者端。 此取消註冊發生後，就不能再呼叫 `/process`. 對未註冊的使用者端或尚未註冊的使用者端使用API呼叫會傳回 `404` 錯誤。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/unregister` |
| 頁首 `Authorization` | 全部 [授權相關標頭](#authentication-and-authorization). |
| 頁首 `x-request-id` | 選填。 使用者端可將其設定為跨系統處理請求的唯一端對端識別碼。 |
| 要求內文 | 空白。 |

### 取消登入回應 {#unregister-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME型別 | `application/json` |
| 頁首 `X-Request-Id` | 與 `X-Request-Id` 要求標頭或唯一產生的標頭。 用於識別跨系統的請求或支援請求。 |
| 回應內文 | 具有的JSON物件 `ok` 和 `requestId` 欄位。 |

狀態代碼為：

* **200項成功**：找到並移除註冊和日誌時發生。

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401未獲授權**：當請求無效時發生 [authentication](#authentication-and-authorization). 例如，存取權杖無效或API金鑰無效。

* **403禁止**：當請求無效時發生 [authorization](#authentication-and-authorization). 例如，可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）未訂閱所有必要服務。

* **404找不到**：當提供的認證未註冊或無效時，此狀態便會出現。

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429太多請求**：在系統超載時發生。 使用者端應使用 [指數輪詢](https://en.wikipedia.org/wiki/Exponential_backoff). 內文是空的。

* **4xx錯誤**：當發生任何其他使用者端錯誤且取消註冊失敗時發生。 通常會傳回此類的JSON回應，但並非所有錯誤都有保證：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx錯誤**：發生於發生任何其他伺服器端錯誤且註冊失敗時。 通常會傳回此類的JSON回應，但並非所有錯誤都有保證：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## 處理要求 {#process-request}

此 `process` 作業會根據請求中的指示，提交將來源資產轉換為多個轉譯的工作。 成功完成的通知（事件型別） `rendition_created`)或任何錯誤(事件型別 `rendition_failed`)傳送至事件日誌，您必須使用擷取該日誌 [`/register`](#register) 在生成任何數量之前執行一次 `/process` 要求。 格式錯誤的請求會立即失敗，並產生400錯誤代碼。

使用URL來參考二進位檔案，例如Amazon AWS S3預先簽署的URL或Azure Blob儲存SAS URL。 用於同時讀取 `source` 資產(`GET` URL)和寫入轉譯(`PUT` URL)。 使用者端負責產生這些預先簽署的URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/process` |
| MIME型別 | `application/json` |
| 頁首 `Authorization` | 全部 [授權相關標頭](#authentication-and-authorization). |
| 頁首 `x-request-id` | 選填。 使用者端可設定唯一的端對端識別碼，以追蹤跨系統的處理請求。 |
| 要求內文 | 它必須是如下所述的流程請求JSON格式。 它會提供要處理哪些資產以及要產生哪些轉譯的說明。 |

### 處理請求JSON {#process-request-json}

的請求內文 `/process` 是具備此高階結構描述的JSON物件：

```json
{
    "source": "",
    "renditions" : []
}
```

可用的欄位包括：

| 名稱 | 類型 | 說明 | 範例 |
|--------------|----------|-------------|---------|
| `source` | `string` | 已處理之來源資產的URL。 選用，根據要求的轉譯格式(例如 `fmt=zip`)。 | `"http://example.com/image.jpg"` |
| `source` | `object` | 說明已處理的來源資產。 請參閱「 」的說明 [Source物件欄位](#source-object-fields) 底下。 根據要求的轉譯格式選擇性(例如， `fmt=zip`)。 | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 要從來源檔案產生的轉譯。 每個轉譯物件都支援 [轉譯指示](#rendition-instructions). 必填。 | `[{ "target": "https://....", "fmt": "png" }]` |

此 `source` 可以是 `<string>` 視為URL或可以是 `<object>` 新增其他欄位。 下列變體類似：

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Source物件欄位 {#source-object-fields}

| 名稱 | 類型 | 說明 | 範例 |
|-----------|----------|-------------|---------|
| `url` | `string` | 要處理的來源資產的URL。 必填。 | `"http://example.com/image.jpg"` |
| `name` | `string` | Source資產檔案名稱。 如果未偵測到MIME型別，則可能會使用名稱為的副檔名。 其優先順序高於URL路徑中指定的檔案名稱。 而且，其優先順序高於中的檔案名稱 `content-disposition` 二進位資源的標頭。 預設為「file」。 | `"image.jpg"` |
| `size` | `number` | Source資產檔案大小（位元組）。 優先於 `content-length` 二進位資源的標頭。 | `10234` |
| `mimetype` | `string` | Source資產檔案MIME型別。 優先於 `content-type` 二進位資源的標頭。 | `"image/jpeg"` |

### 完整 `process` 請求範例 {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## 處理序回應 {#process-response}

此 `/process` 根據基本請求驗證，請求會立即傳回成功或失敗。 實際資產處理為非同步進行。

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME型別 | `application/json` |
| 頁首 `X-Request-Id` | 與 `X-Request-Id` 要求標頭或唯一產生的標頭。 用於識別跨系統的請求或支援請求。 |
| 回應內文 | 具有的JSON物件 `ok` 和 `requestId` 欄位。 |

狀態碼：

* **200項成功**：表示已成功提交要求。 回應JSON包含 `"ok": true`：

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400無效的請求**：如果請求結構不正確，例如，JSON裝載中缺少必填欄位。 回應JSON包含 `"ok": false`：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401未獲授權**：當請求無效時 [authentication](#authentication-and-authorization). 例如，存取權杖無效或API金鑰無效。
* **403禁止**：當請求無效時 [authorization](#authentication-and-authorization). 例如，可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）未訂閱所有必要服務。
* **429太多請求**：當系統不知所措時發生，可能是因為此特定使用者端或整體需求所造成。 使用者端可以使用 [指數輪詢](https://en.wikipedia.org/wiki/Exponential_backoff). 內文是空的。
* **4xx錯誤**：發生任何其他使用者端錯誤時。 通常會傳回此類的JSON回應，但並非所有錯誤都有保證：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx錯誤**：發生任何其他伺服器端錯誤時。 通常會傳回此類的JSON回應，但並非所有錯誤都有保證：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

大多數使用者端都可能傾向使用來重試相同的請求 [指數輪詢](https://en.wikipedia.org/wiki/Exponential_backoff) 發生任何錯誤時 *除了* 設定問題，例如401或403，或無效請求，例如400。 除了429回應方式的定期速率限制以外，暫時性的服務中斷或限制可能會導致5xx錯誤。 之後，建議您在一段時間後重試。

所有JSON回應（如果有的話）包含 `requestId`，此值與 `X-Request-Id` 標頭。 Adobe建議從標頭讀取，因為它一律存在。 此 `requestId` 也會在與處理請求相關的所有事件中傳回 `requestId`. 使用者端不得對此字串的格式做任何假設。 這是不透明的字串識別碼。

## 選擇加入後續處理 {#opt-in-to-post-processing}

此 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk) 支援一組基本的影像後處理選項。 自訂工作者可以透過設定欄位明確選擇加入後續處理 `postProcess` 在轉譯物件上 `true`.

支援的使用案例包括：

* 裁切是矩形的轉譯，其限制由crop.w、crop.h、crop.x和crop.y定義。 裁切細節是在轉譯物件的 `instructions.crop` 欄位。
* 使用寬度、高度或兩者來調整影像大小。 此 `instructions.width` 和 `instructions.height` 會在轉譯物件中加以定義。 若要僅使用寬度或高度來調整大小，請僅設定一個值。 運算服務可節省外觀比例。
* 設定JPEG影像的品質。 此 `instructions.quality` 會在轉譯物件中加以定義。 品質等級為100代表最高品質，而數字較低則表示品質降低。
* 建立交錯影像。 此 `instructions.interlace` 會在轉譯物件中加以定義。
* 設定DPI可調整套用至畫素的比例，以調整案頭出版的演算大小。 此 `instructions.dpi` 會在轉譯物件中定義它，以變更dpi解析度。 不過，若要調整影像大小，使其以不同解析度呈現相同大小，請使用 `convertToDpi` 指示。
* 調整影像大小，使其演算後的寬度或高度與指定目標解析度(DPI)的原始影像保持相同。 此 `instructions.convertToDpi` 會在轉譯物件中加以定義。

## 浮水印資產 {#add-watermark}

此 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk) 支援在PNG、JPEG、TIFF和GIF影像檔案中新增浮水印。 浮水印是依照中的轉譯指示新增的 `watermark` 物件。

浮水印會在轉譯後期處理期間完成。 若要為資產加上浮水印，自訂背景工作 [選擇加入後續處理](#opt-in-to-post-processing) 藉由設定欄位 `postProcess` 在轉譯物件上 `true`. 如果背景工作未選擇加入，則不會套用浮水印，即使浮水印物件已設定在請求中的轉譯物件上。

## 轉譯指示 {#rendition-instructions}

下列為的可用選項 `renditions` 中的陣列 [`/process`](#process-request).

### 常用欄位 {#common-fields}

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 轉譯目標格式也可 `text` 文字擷取和 `xmp` 以將XMP中繼資料擷取為xml。 另請參閱 [支援的格式](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support) | `png` |
| `worker` | `string` | 的URL [自訂應用程式](develop-custom-application.md). 必須是 `https://` URL。 如果此欄位存在，自訂應用程式會建立轉譯。 然後，任何其他集合轉譯欄位都會用於自訂應用程式中。 | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | 所產生轉譯應使用HTTPPUT上傳到的URL。 | `http://w.com/img.jpg` |
| `target` | `object` | 所產生轉譯的多部分預先簽署URL上傳資訊。 此資訊適用於 [AEM / Oak直接二進位上傳](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) 與此 [多部分上傳行為](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>欄位：<ul><li>`urls`：字串陣列，每個預先簽署部分URL各一個</li><li>`minPartSize`：用於一個部分的大小下限= url</li><li>`maxPartSize`：一部份使用的大小上限= url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 選填。 使用者端會控制保留空間，並依原樣傳遞至轉譯事件。 可讓使用者端新增自訂資訊以識別轉譯事件。 自訂應用程式不可修改或依賴它，因為使用者端隨時可以自由變更。 | `{ ... }` |

### 轉譯特定欄位 {#rendition-specific-fields}

如需目前支援的檔案格式清單，請參閱 [支援的檔案格式](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 進階、自訂欄位可新增 [自訂應用程式](develop-custom-application.md) 瞭解。 | |
| `embedBinaryLimit` | `number` 位元組 | 當轉譯的檔案大小小於指定值時，將包含在其建立完成後所傳送的事件中。 允許嵌入的大小上限為32 KB （32 x 1024位元組）。 如果轉譯的大小大於 `embedBinaryLimit` 限制，它會放置在雲端儲存空間中的某個位置，而不會嵌入事件中。 | `3276` |
| `width` | `number` | 寬度（畫素）。 僅適用於影像轉譯。 | `200` |
| `height` | `number` | 高度（畫素）。 僅適用於影像轉譯。 | `200` |
|                   |          | 如果符合下列條件，則一律會維持外觀比例： <ul> <li> 兩者 `width` 和 `height` 會指定，然後影像會符合大小，同時維持外觀比例 </li><li> 如果 `width` 或 `height` 指定，則產生的影像會使用對應的尺寸，同時保持外觀比例</li><li> 如果 `width` 或 `height` 未指定，則會使用原始影像畫素大小。 這取決於來源型別。 對於某些格式(例如PDF檔案)，會使用預設大小。 可以有大小上限。</li></ul> | |
| `quality` | `number` | 在以下範圍內指定jpeg品質： `1` 至 `100`. 僅適用於影像轉譯。 | `90` |
| `xmp` | `string` | 只有XMP中繼資料回寫可使用此功能，它是base64編碼的XMP，用於回寫指定的轉譯。 | |
| `interlace` | `bool` | 建立交錯式PNG或GIF或漸進式JPEG，方法是將其設定為 `true`. 它對其他檔案格式沒有影響。 | |
| `jpegSize` | `number` | JPEG檔案的大約大小（位元組）。 它會覆寫任何 `quality` 設定。 它對其他格式沒有影響。 | |
| `dpi` | `number` 或 `object` | 設定x和y DPI。 為簡化起見，也可將其設為單一數字，用於x和y。這對影像本身沒有影響。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 或 `object` | x和y DPI會重新取樣值，同時維持實體大小。 為簡化起見，也可將其設為單一數字，用於x和y。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | 要包含在ZIP封存檔中的檔案清單(`fmt=zip`)。 每個專案都可以是URL字串或具有欄位的物件：<ul><li>`url`：下載檔案的URL</li><li>`path`：將檔案儲存在ZIP的此路徑下</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP封存檔的重複處理(`fmt=zip`)。 根據預設，儲存在ZIP中相同路徑下的多個檔案會產生錯誤。 設定 `duplicate` 至 `ignore` 只會儲存第一個資產，而忽略其餘資產。 | `ignore` |
| `watermark` | `object` | 包含關於 [浮水印](#watermark-specific-fields). |  |

### 浮水印特定欄位 {#watermark-specific-fields}

PNG格式會作為浮水印使用。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 浮水印的比例，介於 `0.0` 和 `1.0`. `1.0` 表示浮水印具有原始比例(1:1)，較低的值會縮小浮水印大小。 | 值 `0.5` 表示原始大小的一半。 |
| `image` | `url` | 要用於浮水印的PNG檔案的URL。 | |

## 非同步事件 {#asynchronous-events}

當轉譯的處理完成或發生錯誤時，事件會傳送至Adobe [!DNL `I/O Events Journal`]. 使用者端必須監聽透過提供的日誌URL [`/register`](#register). 日誌回應包括 `event` 陣列，每個事件由一個物件組成，其中 `event` 欄位包含實際事件裝載。

Adobe [!DNL `I/O Events`] type for all event of the （此專案的所有事件型別） [!DNL Asset Compute Service] 是 `asset_compute`. 分錄只會自動訂閱此事件型別，不需要根據以下專案進一步篩選： [!DNL Adobe Developer] 事件型別。 您可在以下位置找到服務特定的事件型別： `type` 事件的屬性。

### 事件型別 {#event-types}

| 事件 | 說明 |
|---------------------|-------------|
| `rendition_created` | 已針對每個成功處理和上傳的轉譯傳送。 |
| `rendition_failed` | 針對無法處理或上傳的每個轉譯傳送。 |

### 事件屬性 {#event-attributes}

| 屬性 | 類型 | 事件 | 說明 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | 以簡化的延伸傳送事件時的時間戳記 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 格式，由JavaScript定義 [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | 傳送給的原始請求的請求識別碼 `/process`，與 `X-Request-Id` 標頭。 |
| `source` | `object` | `*` | 此 `source` 的 `/process` 要求。 |
| `userData` | `object` | `*` | 此 `userData` 的轉譯 `/process` 請求（若已設定）。 |
| `rendition` | `object` | `rendition_*` | 傳入的對應轉譯物件 `/process`. |
| `metadata` | `object` | `rendition_created` | 此 [中繼資料](#metadata) 轉譯的屬性。 |
| `errorReason` | `string` | `rendition_failed` | 轉譯失敗 [原因](#error-reasons) 如果有。 |
| `errorMessage` | `string` | `rendition_failed` | 此文字會提供更多有關轉譯失敗的詳細資料（如果有的話）。 |

### 中繼資料 {#metadata}

| 屬性 | 說明 |
|--------|-------------|
| `repo:size` | 轉譯的大小，以位元組計。 |
| `repo:sha1` | 轉譯的sha1摘要。 |
| `dc:format` | 轉譯的 MIME 類型。 |
| `repo:encoding` | 轉譯的字元集編碼（若為文字型格式）。 |
| `tiff:ImageWidth` | 轉譯的寬度（畫素）。 僅供影像轉譯使用。 |
| `tiff:ImageLength` | 轉譯的長度（畫素）。 僅供影像轉譯使用。 |

### 錯誤原因 {#error-reasons}

| 原因 | 說明 |
|---------|-------------|
| `RenditionFormatUnsupported` | 給定的來源不支援要求的轉譯格式。 |
| `SourceUnsupported` | 即使支援型別，也不支援特定來源。 |
| `SourceCorrupt` | 來源資料已損毀。 其中包含空白檔案。 |
| `RenditionTooLarge` | 無法使用中提供的預先簽署URL上傳轉譯 `target`. 實際轉譯大小在中會以中繼資料的形式提供 `repo:size` 使用者端使用並以正確數量的預先簽署URL重新處理此轉譯。 |
| `GenericError` | 任何其他非預期的錯誤。 |
