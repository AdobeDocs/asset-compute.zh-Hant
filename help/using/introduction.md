---
title: ' [!DNL Asset Compute Service]簡介'
description: '[!DNL Asset Compute Service]是雲端原生資產處理服務，可降低複雜度並改善擴充能力。'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
TQID: https://experienceleague.adobe.com/7B0ghzyLWIZFe5sthLaorFf8w4Az7FKW4ces27eo8I0
product_v2:
  - id: d09181b5-a36a-43de-ba01-36641440bc43
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
feature_v2:
  - id: a01bfd36-4ab8-4bf8-9dc0-5b45b890552e
  - id: da0dfbce-df02-4f8b-b32d-a4e3b1d05085
role_v2:
  - id: ff6a42d2-313e-452e-93a6-792e4fad9ff8
topic_v2:
  - id: a004cc84-67b9-4a33-a3a7-8ec7273ef4dc
source-git-commit: 2510f77fed8d0f0708e09f32d0b13a437d2ede4f
workflow-type: tm+mt
source-wordcount: 355
ht-degree: 12%

---

# [!DNL Asset Compute Service]的概觀 {#overview}

[!DNL Asset Compute Service]是可擴充的[!DNL Adobe Experience Cloud]服務，用於處理數位資產。 它可以將影像、視訊、檔案和其他檔案格式轉換為不同的轉譯，包括縮圖、擷取的文字和中繼資料以及封存。

開發人員可外掛自訂資產應用程式（也稱為自訂工作角色），以處理自訂使用案例。 此服務適用於Adobe [!DNL I/O Runtime]。 它可透過以Node.js撰寫的[!DNL Adobe Developer App Builder]個Headless應用程式延伸。 他們可以執行自訂作業，例如呼叫外部API以執行影像作業或運用[!DNL Adobe Sensei]支援。

[!DNL Adobe Developer App Builder]是一個架構，可在Adobe [!DNL I/O Runtime]上建置和部署自訂Web應用程式以擴充Adobe Experience Cloud解決方案。 若要建立自訂應用程式，開發人員可以利用[!DNL React Spectrum] （Adobe的UI工具組）、建立微服務、建立自訂事件以及協調API。 請參閱Adobe Developer App Builder[&#128279;](https://developer.adobe.com/app-builder/docs/intro_and_overview/#)的檔案。

>[!NOTE]
>
>目前，[!DNL Asset Compute Service]只能透過[!DNL Experience Manager]用作[!DNL Cloud Service]。 管理員建立處理設定檔，可呼叫[!DNL Asset Compute Service]以傳遞資產以供處理。 請參閱[使用資產微服務及處理設定檔](https://experienceleague.adobe.com/zh-hant/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)。

## [!DNL Asset Compute Service]的支援使用案例 {#possible-use-cases-benefits}

[!DNL Asset Compute Service]支援一些常見的業務使用案例，例如基本影像處理、Adobe應用程式特定轉換，以及協調複雜業務需求的自訂應用程式建立。

您可以使用[!DNL Asset Compute] Web服務來產生不同檔案型別的縮圖，以及[支援的檔案格式](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support)的高品質影像轉譯。 檢視透過自訂組態支援的[使用案例](https://experienceleague.adobe.com/zh-hant/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)。

>[!NOTE]
>
>此服務不提供資產儲存。 使用者會提供它，並參考雲端儲存空間中的來源和轉譯檔案位置。

<!-- 
TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [在 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/zh-hant/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview)中使用資產微服務的資產處理概觀。
>* [Adobe Developer App Builder檔案](https://developer.adobe.com/app-builder/docs/intro_and_overview/#)。
>* [支援的檔案格式以進行處理](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support)。

<!-- 
**TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
