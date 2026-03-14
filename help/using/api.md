---
title: '[!DNL Asset Compute Service] HTTP API'
description: '[!DNL Asset Compute Service] HTTP API可建立自訂應用程式。'
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: aed361a577fc53caec4118e417b1c0c814617b51
workflow-type: tm+mt
source-wordcount: '2995'
ht-degree: 3%

---

# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API的使用僅限於開發目的。 API在開發自訂應用程式時作為內容提供。[!DNL Adobe Experience Manager] as a [!DNL Cloud Service]使用此API將處理資訊傳遞至自訂應用程式。 如需詳細資訊，請參閱[使用資產微服務和處理設定檔](https://experienceleague.adobe.com/zh-hant/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)。

>[!NOTE]
>
>[!DNL Asset Compute Service]僅可與[!DNL Experience Manager]搭配[!DNL Cloud Service]使用。

[!DNL Asset Compute Service] HTTP API的任何使用者端都必須遵循此高階流程：

1. 使用者端已布建為IMS組織中的[!DNL Adobe Developer Console]專案。 每個單獨的使用者端（系統或環境）需要其自己的單獨專案，以分隔事件資料流。

1. 使用者端使用[JWT （服務帳戶）驗證](https://developer.adobe.com/developer-console/docs/guides/)為技術帳戶產生存取權杖。

1. 使用者端只會呼叫[`/register`](#register)一次，以擷取日誌URL。

1. 使用者端會針對要產生轉譯的每個資產呼叫[`/process`](#process-request)。 呼叫為非同步。

1. 使用者端定期輪詢日誌以接收[個事件](#asynchronous-events)。 成功處理轉譯時（`rendition_created`事件型別）或發生錯誤時（`rendition_failed`事件型別），它會接收每個要求轉譯的事件。

[adobe-asset-compute-client](https://github.com/adobe/asset-compute-client)模組可讓您在Node.js程式碼中輕鬆使用API。

## 驗證和授權 {#authentication-and-authorization}

所有API都需要存取權杖驗證。 請求必須設定以下標頭：

1. 從Adobe Developer Console專案透過[JWT交換](https://developer.adobe.com/developer-console/docs/guides/)收到的具有持有人權杖的`Authorization`標頭，這是技術帳戶權杖。 [領域](#scopes)記錄如下。

<!-- 
TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. 具有IMS組織ID的`x-gw-ims-org-id`標題。

1. `x-api-key`具有來自[!DNL Adobe Developers Console]專案的使用者端ID。

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

這些範圍需要訂閱`Asset Compute`、`I/O Events`和`I/O Management API`服務的[!DNL Adobe Developer Console]專案。 個別範圍的劃分如下：

* 基本
   * 範圍： `openid,AdobeID`

* Asset Compute
   * metascope： `asset_compute_meta`
   * 範圍： `asset_compute,read_organizations`

* Adobe [!DNL `I/O Events`]
   * metascope： `event_receiver_api`
   * 範圍： `event_receiver,event_receiver_api`

* Adobe [!DNL `I/O Management API`]
   * metascope： `ent_adobeio_sdk`
   * 範圍： `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 註冊 {#register}

[!DNL Asset Compute service]的每個使用者端（訂閱此服務的唯一[!DNL Adobe Developer Console]專案）都必須[註冊](#register-request)，才能提出處理要求。 註冊步驟會傳回從轉譯處理中擷取非同步事件所需的唯一事件日誌。

在其生命週期結束時，使用者端可以[取消註冊](#unregister-request)。

### 註冊要求 {#register-request}

此API呼叫會設定[!DNL Asset Compute]使用者端並提供事件日誌URL。 此程式是等冪操作，每個使用者端只需要呼叫一次。 可再次呼叫它以擷取日誌URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/register` |
| 標頭`Authorization` | 所有[授權相關的標頭](#authentication-and-authorization)。 |
| 標頭`x-request-id` | 選擇性，由使用者端設定，用於跨系統處理請求的唯一端對端識別碼。 |
| 要求內文 | 必須為空白。 |

### 登入回應 {#register-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME型別 | `application/json` |
| 標頭`X-Request-Id` | 與`X-Request-Id`要求標頭相同或唯一產生的標頭。 用於識別跨系統的請求或支援請求，或兩者皆使用。 |
| 回應內文 | 具有`journal`、`ok`或`requestId`欄位的JSON物件。 |

HTTP狀態碼為：

* **200成功**：要求成功時。 `journal` URL會收到有關透過`/process`啟動的非同步處理結果的通知。 它會在成功完成時發出`rendition_created`個事件的警示，如果處理失敗，則會發出`rendition_failed`個事件的警示。

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401未獲授權**：當要求沒有有效的[驗證](#authentication-and-authorization)時發生。 例如，存取權杖無效或API金鑰無效。

* **403禁止存取**：當要求沒有有效的[授權](#authentication-and-authorization)時發生。 例如，可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）未訂閱所有必要服務。

* **429太多要求**：發生在此使用者端或是系統多載時。 使用者端應以[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試。 內文是空的。
* **4xx錯誤**：發生任何其他使用者端錯誤時註冊失敗。 通常會傳回此類的JSON回應，但並非所有錯誤都有保證：

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

此API呼叫會取消註冊[!DNL Asset Compute]使用者端。 此取消註冊發生後，就不能再呼叫`/process`。 對未註冊的使用者端或尚未註冊的使用者端使用API呼叫會傳回`404`錯誤。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/unregister` |
| 標頭`Authorization` | 所有[授權相關的標頭](#authentication-and-authorization)。 |
| 標頭`x-request-id` | 選擇性。 使用者端可將其設定為跨系統處理請求的唯一端對端識別碼。 |
| 要求內文 | 空白。 |

### 取消登入回應 {#unregister-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME型別 | `application/json` |
| 標頭`X-Request-Id` | 與`X-Request-Id`要求標頭相同或唯一產生的標頭。 用於識別跨系統的請求或支援請求。 |
| 回應內文 | 具有`ok`和`requestId`欄位的JSON物件。 |

狀態代碼為：

* **200成功**：找到並移除登入和日誌時發生。

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401未獲授權**：當要求沒有有效的[驗證](#authentication-and-authorization)時發生。 例如，存取權杖無效或API金鑰無效。

* **403禁止存取**：當要求沒有有效的[授權](#authentication-and-authorization)時發生。 例如，可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）未訂閱所有必要服務。

* **404找不到**：當提供的認證未登入或無效時，就會顯示此狀態。

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429太多要求**：系統超載時發生。 使用者端應以[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試。 內文是空的。

* **4xx錯誤**：發生於發生任何其他使用者端錯誤且解除登入失敗時。 通常會傳回此類的JSON回應，但並非所有錯誤都有保證：

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

`process`作業會根據請求中的指示，提交將來源資產轉換為多個轉譯的工作。 有關成功完成（事件型別`rendition_created`）或任何錯誤（事件型別`rendition_failed`）的通知會傳送至事件日誌，必須先使用[`/register`](#register)一次擷取該日誌，然後才能發出任何數量的`/process`個請求。 格式錯誤的請求會立即失敗，並產生400錯誤代碼。

使用URL來參考二進位檔案，例如Amazon AWS S3預先簽署的URL或Azure Blob儲存SAS URL。 用於讀取`source`資產(`GET` URL)和寫入轉譯(`PUT` URL)。 使用者端負責產生這些預先簽署的URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/process` |
| MIME型別 | `application/json` |
| 標頭`Authorization` | 所有[授權相關的標頭](#authentication-and-authorization)。 |
| 標頭`x-request-id` | 選擇性。 使用者端可設定唯一的端對端識別碼，以追蹤跨系統的處理請求。 |
| 要求內文 | 它必須是如下所述的流程請求JSON格式。 它會提供要處理哪些資產以及要產生哪些轉譯的說明。 |

### 處理請求JSON {#process-request-json}

`/process`的請求本文是具有此高階結構描述的JSON物件：

```json
{
    "source": "",
    "renditions" : []
}
```

可用的欄位包括：

| 名稱 | 類型 | 說明 | 範例 |
|--------------|----------|-------------|---------|
| `source` | `string` | 已處理之來源資產的URL。 選擇性，根據要求的轉譯格式（例如，`fmt=zip`）。 | `"http://example.com/image.jpg"` |
| `source` | `object` | 說明已處理的來源資產。 請參閱下列[Source物件欄位](#source-object-fields)的說明。 根據要求的轉譯格式（例如，`fmt=zip`）選擇性。 | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 要從來源檔案產生的轉譯。 每個轉譯物件都支援[轉譯指示](#rendition-instructions)。 必要。 | `[{ "target": "https://....", "fmt": "png" }]` |

`source`可以是視為URL的`<string>`，或可以是具有額外欄位的`<object>`。 下列變體類似：

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
| `url` | `string` | 要處理的來源資產的URL。 必要。 | `"http://example.com/image.jpg"` |
| `name` | `string` | Source資產檔案名稱。 如果未偵測到MIME型別，則可能會使用名稱為的副檔名。 其優先順序高於URL路徑中指定的檔案名稱。 而且，其優先順序高於二進位資源的`content-disposition`標頭中的檔案名稱。 預設為「file」。 | `"image.jpg"` |
| `size` | `number` | Source資產檔案大小（位元組）。 優先於二進位資源的`content-length`標頭。 | `10234` |
| `mimetype` | `string` | Source資產檔案MIME型別。 優先於二進位資源的`content-type`標頭。 | `"image/jpeg"` |

### 完整的`process`要求範例 {#complete-process-request-example}

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

根據基本要求驗證，`/process`要求會立即傳回成功或失敗。 實際資產處理為非同步進行。

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME型別 | `application/json` |
| 標頭`X-Request-Id` | 與`X-Request-Id`要求標頭相同或唯一產生的標頭。 用於識別跨系統的請求或支援請求。 |
| 回應內文 | 具有`ok`和`requestId`欄位的JSON物件。 |

狀態碼：

* **200成功**：若已成功提交要求。 回應JSON包含`"ok": true`：

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400無效的請求**：如果請求結構不正確（例如，JSON承載中缺少必要欄位）。 回應JSON包含`"ok": false`：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401未獲授權**：當要求沒有有效的[驗證](#authentication-and-authorization)時。 例如，存取權杖無效或API金鑰無效。
* **403禁止存取**：要求沒有有效的[授權](#authentication-and-authorization)時。 例如，可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）未訂閱所有必要服務。
* **429太多要求**：系統因這個特定使用者端或整體需求而受壓時發生。 使用者端可以使用[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試。 內文是空的。
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

大部分使用者端可能會在任何錯誤&#x200B;*上以[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試相同要求，但*&#x200B;設定問題（例如401或403）或無效要求（例如400）除外。 除了429回應方式的定期速率限制以外，暫時性的服務中斷或限制可能會導致5xx錯誤。 之後，建議您在一段時間後重試。

所有JSON回應（如果存在）都包含`requestId`，其值與`X-Request-Id`標頭相同。 Adobe建議從標頭讀取，因為它一律存在。 `requestId`也會在與處理要求相關的所有事件中傳回為`requestId`。 使用者端不得對此字串的格式做任何假設。 這是不透明的字串識別碼。

## 選擇加入後續處理 {#opt-in-to-post-processing}

[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk)支援一組基本的影像後處理選項。 自訂背景工作可透過將轉譯物件上的欄位`postProcess`設定為`true`，明確選擇加入後續處理。

支援的使用案例包括：

* 裁切是矩形的轉譯，其限制由crop.w、crop.h、crop.x和crop.y定義。 裁切詳細資料是在轉譯物件的`instructions.crop`欄位中指定的。
* 使用寬度、高度或兩者來調整影像大小。 `instructions.width`和`instructions.height`在轉譯物件中定義它。 若要僅使用寬度或高度來調整大小，請僅設定一個值。 運算服務可節省外觀比例。
* 設定JPEG影像的品質。 `instructions.quality`在轉譯物件中定義它。 品質等級為100代表最高品質，而數字較低則表示品質降低。
* 建立交錯影像。 `instructions.interlace`在格式副本對象中定義它。
* 設定DPI以通過調整應用於像素的比例來調整呈現大小以用於案頭發佈。 `instructions.dpi`在格式副本對象中定義它以更改dpi解析度。 但是，要調整影像大小，使其在不同的解析度下大小相同，請使用`convertToDpi`說明。
* 調整影像的大小，使其渲染的寬度或高度在指定目標解析度(DPI)下保持與原始影像相同。 `instructions.convertToDpi`在格式副本對象中定義它。

## 浮水印資產 {#add-watermark}

[Asset computeSDK](https://github.com/adobe/asset-compute-sdk)支援向PNG、JPEG、TIFF和GIF影像檔案添加水印。 將按照格式副本上的`watermark`對象中的格式副本說明添加水印。

在格式副本後處理期間進行水印處理。 要對資產進行水印，自定義工作程式[通過將格式副本對象上的欄位`postProcess`設定為`true`，選擇後處理](#opt-in-to-post-processing)。 如果工作程式不選擇加入，則不應用水印，即使在請求中的格式副本對象上設定了水印對象。

## 格式副本說明 {#rendition-instructions}

以下是[`/process`](#process-request)中`renditions`陣列的可用選項。

### 常用欄位 {#common-fields}

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 格式副本目標格式也可以是文本提取的`text`格式副本，也可以是將元資料提取為xmlXMP格式副本的`xmp`格式副本。 請參閱[支援的格式](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support) | `png` |
| `worker` | `string` | [自定義應用程式](develop-custom-application.md)的URL。 必須是`https://` URL。 如果存在此欄位，則自定義應用程式將建立格式副本。 然後，在自定義應用程式中使用任何其他集格式副本欄位。 | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | 應使用HTTPPUT將生成的格式副本上載到的URL。 | `http://w.com/img.jpg` |
| `target` | `object` | 生成的格式副本的多部分預簽名URL上載資訊。 此資訊用於[AEM / Oak Direct Binary Upload](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html)，具有此[多部分上載行為](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html)。<br>欄位：<ul><li>`urls`：字串陣列，每個預簽名的部件URL為一個</li><li>`minPartSize`：用於一個部件的最小大小= url</li><li>`maxPartSize`：單一部分使用的大小上限= url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 選擇性。 使用者端會控制保留空間，並依原樣傳遞至轉譯事件。 可讓使用者端新增自訂資訊以識別轉譯事件。 自訂應用程式不可修改或依賴它，因為使用者端隨時可以自由變更。 | `{ ... }` |

### 轉譯特定欄位 {#rendition-specific-fields}

如需目前支援的檔案格式清單，請參閱[支援的檔案格式](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support)。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 可新增[自訂應用程式](develop-custom-application.md)瞭解的進階自訂欄位。 | |
| `embedBinaryLimit` | `number`位元組 | 當轉譯的檔案大小小於指定值時，將包含在其建立完成後所傳送的事件中。 允許嵌入的大小上限為32 KB （32 x 1024位元組）。 如果轉譯的大小超過`embedBinaryLimit`限制，則會將其放置在雲端儲存空間中的位置，而不會嵌入事件中。 | `3276` |
| `width` | `number` | 寬度（畫素）。 僅適用於影像轉譯。 | `200` |
| `height` | `number` | 高度（畫素）。 僅適用於影像轉譯。 | `200` |
|                   |          | 如果符合下列條件，則一律會維持外觀比例： <ul> <li> 同時指定了`width`和`height`，然後影像會符合大小，同時維持外觀比例 </li><li> 如果只指定`width`或`height`，產生的影像會使用對應的維度，同時保持外觀比例</li><li> 如果未指定`width`或`height`，則會使用原始影像畫素大小。 這取決於來源型別。 對於某些格式，例如PDF檔案，會使用預設大小。 可以有大小上限。</li></ul> | |
| `quality` | `number` | 指定`1`到`100`範圍內的jpeg品質。 僅適用於影像轉譯。 | `90` |
| `xmp` | `string` | 此範本僅供XMP中繼資料回寫使用，是base64編碼的XMP，可回寫至指定的轉譯。 | |
| `interlace` | `bool` | 將交錯式PNG、GIF或漸進式JPEG設定為`true`，以建立它。 它對其他檔案格式沒有影響。 | |
| `jpegSize` | `number` | JPEG檔案的大致大小（位元組）。 它會覆寫任何`quality`設定。 它對其他格式沒有影響。 | |
| `dpi` | `number` 或 `object` | 設定x和y DPI。 為簡化起見，也可將其設為單一數字，用於x和y。 這對影像本身沒有影響。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 或 `object` | x和y DPI會重新取樣值，同時維持實體大小。 為簡化起見，也可將其設為單一數字，用於x和y。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | 要包含在ZIP封存檔中的檔案清單(`fmt=zip`)。 每個專案都可以是URL字串或具有欄位的物件：<ul><li>`url`：下載檔案的URL</li><li>`path`：將檔案儲存在ZIP的這個路徑下</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP封存檔的重複處理(`fmt=zip`)。 根據預設，儲存在ZIP中相同路徑下的多個檔案會產生錯誤。 將`duplicate`設定為`ignore`只會儲存第一個資產，而忽略其餘資產。 | `ignore` |
| `watermark` | `object` | 包含有關[浮水印](#watermark-specific-fields)的指示。 |  |

### 浮水印特定欄位 {#watermark-specific-fields}

PNG格式會作為浮水印使用。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 浮水印的比例，介於`0.0`到`1.0`之間。`1.0` 表示浮水印具有其原始比例(1:1)，而較低的值會縮小浮水印大小。 | 值`0.5`表示原始大小的一半。 |
| `image` | `url` | 要用於浮水印的PNG檔案的URL。 | |

## 非同步事件 {#asynchronous-events}

處理完格式副本或發生錯誤時，事件將發送到Adobe[!DNL `I/O Events Journal`]。 客戶端必須偵聽通過[`/register`](#register)提供的日誌URL。 日誌響應包括`event`陣列，該陣列由每個事件的一個對象組成，其中`event`欄位包含實際事件負載。

[!DNL Asset Compute Service]的所有事件的Adobe[!DNL `I/O Events`]類型為`asset_compute`。 日記帳僅自動訂閱此事件類型，並且不再需要基於[!DNL Adobe Developer]事件類型進行篩選。 特定於服務的事件類型在事件的`type`屬性中可用。

### 事件類型 {#event-types}

| 事件 | 說明 |
|---------------------|-------------|
| `rendition_created` | 為每個成功處理和上載的格式副本發送。 |
| `rendition_failed` | 為無法處理或上載的每個格式副本發送。 |

### 事件屬性 {#event-attributes}

| 屬性 | 類型 | 事件 | 說明 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | 按照JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString)的定義，以簡化的擴展[ISO-8601](https://en.wikipedia.org/wiki/ISO_8601)格式發送事件的時間戳。 |
| `requestId` | `string` | `*` | 原始請求到`/process`的請求ID，與`X-Request-Id`標頭相同。 |
| `source` | `object` | `*` | `/process`請求的`source`。 |
| `userData` | `object` | `*` | 如果已設定，則`/process`請求中的格式副本的`userData`。 |
| `rendition` | `object` | `rendition_*` | 在`/process`中傳遞的相應格式副本對象。 |
| `metadata` | `object` | `rendition_created` | 格式副本的[元資料](#metadata)屬性。 |
| `errorReason` | `string` | `rendition_failed` | 格式副本失敗[原因](#error-reasons)（如果有）。 |
| `errorMessage` | `string` | `rendition_failed` | 提供有關格式副本失敗（如果有）的更詳細資訊的文本。 |

### 後設資料 {#metadata}

| 屬性 | 說明 |
|--------|-------------|
| `repo:size` | 轉譯的大小，以位元組計。 |
| `repo:sha1` | 格式副本的sha1摘要。 |
| `dc:format` | 轉譯的 MIME 類型。 |
| `repo:encoding` | 格式副本的字元集編碼（如果它是基於文本的格式）。 |
| `tiff:ImageWidth` | 格式副本的寬度（以像素為單位）。 僅用於影像格式副本。 |
| `tiff:ImageLength` | 格式副本的長度（以像素為單位）。 僅用於影像格式副本。 |

### 錯誤原因 {#error-reasons}

| 原因 | 說明 |
|---------|-------------|
| `RenditionFormatUnsupported` | 給定的來源不支援要求的轉譯格式。 |
| `SourceUnsupported` | 即使支援型別，也不支援特定來源。 |
| `SourceCorrupt` | 來源資料已損毀。 其中包含空白檔案。 |
| `RenditionTooLarge` | 無法使用`target`中提供的預先簽署URL上傳轉譯。 實際轉譯大小可在`repo:size`中當做中繼資料使用，使用者端會使用正確數目的預先簽署URL來重新處理此轉譯。 |
| `GenericError` | 任何其他非預期的錯誤。 |
