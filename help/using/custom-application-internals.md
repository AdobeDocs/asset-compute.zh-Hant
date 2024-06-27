---
title: 瞭解自訂應用程式的運作方式
description: 內部工作 [!DNL Asset Compute Service] 自訂應用程式，協助瞭解其運作方式。
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '691'
ht-degree: 0%

---

# 自訂應用程式的內部結構 {#how-custom-application-works}

使用下圖來瞭解當使用者端使用自訂應用程式處理數位資產時的端對端工作流程。

![自訂應用程式工作流程](assets/customworker.svg)

*圖：使用Adobe處理資產時涉及的步驟 [!DNL Asset Compute Service].*

## 註冊 {#registration}

使用者端必須呼叫 [`/register`](api.md#register) 在第一次要求前完成一次 [`/process`](api.md#process-request) 以便設定並擷取分錄URL以接收Adobe [!DNL I/O Events] 用於AdobeAsset compute的事件。

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

此 [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript程式庫可用於NodeJS應用程式，以處理從註冊、處理到非同步事件處理的所有必要步驟。 如需必要標題的詳細資訊，請參閱 [驗證和授權](api.md).

## 處理中 {#processing}

使用者端傳送 [處理](api.md#process-request) 要求。

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

使用者端負責使用預先簽署的URL正確格式化轉譯。 此 [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript程式庫可用於NodeJS應用程式，以預先簽署URL。 目前資料庫僅支援Azure Blob儲存和AWS S3容器。

處理請求傳回 `requestId` 可用於輪詢的 [!DNL Adobe I/O] 事件。

以下為範例自訂應用程式處理請求。

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

此 [!DNL Asset Compute Service] 傳送自訂應用程式轉譯請求給自訂應用程式。 它會使用HTTPPOST連線提供的應用程式URL，這是來自App Builder的安全網路動作URL。 所有要求都會使用HTTPS通訊協定，以提升資料安全性。

此 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 由自訂應用程式使用，用於處理HTTPPOST請求。 它也會處理來源下載、上傳轉譯、傳送Adobe [!DNL I/O Events] 和錯誤處理。

<!-- TBD: Add the application diagram. -->

### 應用程式程式碼 {#application-code}

自訂程式碼只需要提供取用本機可用來源檔案(`source.path`)。 此 `rendition.path` 是放置資產處理請求之最終結果的位置。 自訂應用程式會使用回呼，使用傳入的名稱將本機可用的來源檔案轉換為轉譯檔案(`rendition.path`)。 自訂應用程式必須寫入 `rendition.path` 若要建立轉譯：

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### 下載來源檔案 {#download-source}

自訂應用程式只會處理本機檔案。 此 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 處理來源檔案的下載。

### 建立轉譯 {#rendition-creation}

SDK會以非同步方式呼叫 [轉譯回呼函式](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 每個轉譯。

回呼函式可以存取 [來源](https://github.com/adobe/asset-compute-sdk#source) 和 [轉譯](https://github.com/adobe/asset-compute-sdk#rendition) 物件。 此 `source.path` 已經存在，並且是來源檔案的本機復本路徑。 此 `rendition.path` 是必須儲存已處理轉譯的路徑。 除非 [disableSourceDownload旗標](https://github.com/adobe/asset-compute-sdk#worker-options-optional) 設定，應用程式必須完全使用 `rendition.path`. 否則，SDK將無法找到或識別轉譯檔案，並會失敗。

範例的過度簡化是為了說明和專注於自訂應用程式的剖析。 應用程式只會將來源檔案複製到轉譯目的地。

如需有關轉譯回呼引數的詳細資訊，請參閱 [ASSET COMPUTE SDK API](https://github.com/adobe/asset-compute-sdk#api-details).

### 上傳轉譯 {#upload-rendition}

建立每個轉譯後，並將其儲存在具有所提供路徑的檔案中 `rendition.path`，則 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 將每個轉譯上傳到雲端儲存空間(AWS或Azure)。 若且唯若傳入請求具有多個指向相同應用程式URL的轉譯時，自訂應用程式才會同時取得多個轉譯。 上傳至雲端儲存空間會在每個轉譯之後以及下一個轉譯的執行回呼之前完成。

此 `batchWorker()` 具有不同的行為。 它會處理所有轉譯，並在所有轉譯均已處理完畢後上傳它們。

## [!DNL Adobe I/O] 活動 {#aio-events}

SDK會傳送Adobe [!DNL I/O Events] 每個轉譯。 這些事件為任一型別 `rendition_created` 或 `rendition_failed` 視結果而定。 如需詳細資訊，請參閱 [asset compute非同步事件](api.md#asynchronous-events).

## 接收 [!DNL Adobe I/O] 活動 {#receive-aio-events}

使用者端輪詢Adobe [!DNL I/O Events] 依據其沖銷邏輯分類的分錄。 初始日誌URL是中提供的URL `/register` API回應。 可使用以下專案識別事件 `requestId` 會顯示在事件中，與在中傳回的一樣 `/process`. 每個轉譯都有個別事件，會在轉譯上傳（或失敗）後立即傳送。 當使用者端收到相符事件時，可以顯示或以其他方式處理產生的轉譯。

JavaScript資料庫 [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) 使用簡化日誌輪詢 `waitActivation()` 方法以取得所有事件。

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

如需如何取得日誌事件的詳細資訊，請參閱Adobe [[!DNL I/O Events] API](https://developer.adobe.com/events/docs/guides/api/journaling_api/).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
