---
title: 瞭解自訂應用程式的運作方式
description: ' [!DNL Asset Compute Service] 自訂應用程式的內部運作，以協助瞭解其運作方式。'
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '691'
ht-degree: 0%

---

# 自訂應用程式的內部結構 {#how-custom-application-works}

使用下圖來瞭解當使用者端使用自訂應用程式處理數位資產時的端對端工作流程。

![自訂應用程式工作流程](assets/customworker.svg)

*圖：使用Adobe[!DNL Asset Compute Service]處理資產時涉及的步驟。*

## 註冊 {#registration}

使用者端必須先呼叫[`/register`](api.md#register)一次，才能在第一次對[`/process`](api.md#process-request)發出要求之前，設定並擷取分錄URL，以接收AdobeAsset compute的Adobe[!DNL I/O Events]事件。

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

[`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript程式庫可用於NodeJS應用程式，以處理從註冊、處理到非同步事件處理的所有必要步驟。 如需必要標頭的詳細資訊，請參閱[驗證和授權](api.md)。

## 處理中 {#processing}

使用者端傳送[處理](api.md#process-request)要求。

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

使用者端負責使用預先簽署的URL正確格式化轉譯。 [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript程式庫可用於NodeJS應用程式，以預先簽署URL。 目前資料庫僅支援Azure Blob儲存和AWS S3容器。

處理要求傳回可用於輪詢[!DNL Adobe I/O]事件的`requestId`。

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

[!DNL Asset Compute Service]傳送自訂應用程式轉譯要求給自訂應用程式。 它會使用HTTPPOST連線提供的應用程式URL，這是來自App Builder的安全網路動作URL。 所有要求都會使用HTTPS通訊協定，以提升資料安全性。

自訂應用程式使用的[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)會處理HTTPPOST要求。 它也會處理來源下載、上傳轉譯、傳送Adobe[!DNL I/O Events]和錯誤處理。

<!-- TBD: Add the application diagram. -->

### 應用程式程式碼 {#application-code}

自訂程式碼只需要提供取用本機可用原始程式檔(`source.path`)的回呼。 `rendition.path`是放置資產處理請求之最終結果的位置。 自訂應用程式會使用回呼，使用傳入的名稱(`rendition.path`)，將本機可用的來源檔案轉換為轉譯檔案。 自訂應用程式必須寫入`rendition.path`才能建立轉譯：

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

自訂應用程式只會處理本機檔案。 [Asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)會處理來源檔案的下載。

### 建立轉譯 {#rendition-creation}

SDK會呼叫每個轉譯的非同步[轉譯回呼函式](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required)。

回呼函式可以存取[來源](https://github.com/adobe/asset-compute-sdk#source)和[轉譯](https://github.com/adobe/asset-compute-sdk#rendition)物件。 `source.path`已經存在，而且是來源檔案的本機復本路徑。 `rendition.path`是必須儲存已處理轉譯的路徑。 除非已設定[disableSourceDownload旗標](https://github.com/adobe/asset-compute-sdk#worker-options-optional)，否則應用程式必須完全使用`rendition.path`。 否則，SDK將無法找到或識別轉譯檔案，並會失敗。

範例的過度簡化是為了說明和專注於自訂應用程式的剖析。 應用程式只會將來源檔案複製到轉譯目的地。

如需有關轉譯回呼引數的詳細資訊，請參閱[Asset computeSDK API](https://github.com/adobe/asset-compute-sdk#api-details)。

### 上傳轉譯 {#upload-rendition}

在建立每個轉譯並儲存在`rendition.path`提供路徑的檔案中後，[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)會將每個轉譯上傳至雲端儲存空間(AWS或Azure)。 若且唯若傳入請求具有多個指向相同應用程式URL的轉譯時，自訂應用程式才會同時取得多個轉譯。 上傳至雲端儲存空間會在每個轉譯之後以及下一個轉譯的執行回呼之前完成。

`batchWorker()`有不同的行為。 它會處理所有轉譯，並在所有轉譯均已處理完畢後上傳它們。

## [!DNL Adobe I/O]個事件 {#aio-events}

SDK會針對每個轉譯傳送Adobe[!DNL I/O Events]。 視結果而定，這些事件是型別`rendition_created`或`rendition_failed`。 如需詳細資訊，請參閱[Asset compute非同步事件](api.md#asynchronous-events)。

## 接收[!DNL Adobe I/O]個事件 {#receive-aio-events}

使用者端根據其使用邏輯輪詢Adobe[!DNL I/O Events]日誌。 初始日誌URL是`/register` API回應中提供的日誌URL。 可以使用存在於事件中的`requestId`來識別事件，並且與`/process`中傳回的相同。 每個轉譯都有個別事件，會在轉譯上傳（或失敗）後立即傳送。 當使用者端收到相符事件時，可以顯示或以其他方式處理產生的轉譯。

JavaScript程式庫[`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage)使用`waitActivation()`方法取得所有事件，讓日誌輪詢變得簡單。

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

如需有關如何取得日誌事件的詳細資訊，請參閱Adobe[[!DNL I/O Events] API](https://developer.adobe.com/events/docs/guides/api/journaling_api/)。

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
