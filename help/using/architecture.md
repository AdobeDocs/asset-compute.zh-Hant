---
title: 架構 [!DNL Asset Compute Service]
description: 如何 [!DNL Asset Compute Service] API、應用程式和SDK會共同合作，提供雲端原生資產處理服務。
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '478'
ht-degree: 0%

---

# 架構 [!DNL Asset Compute Service] {#overview}

此 [!DNL Asset Compute Service] 建置在無伺服器Adobe上 [!DNL `I/O Runtime`] 平台。 提供資產的Adobe Sensei內容服務支援。 叫用使用者端(僅限 [!DNL Experience Manager] as a [!DNL Cloud Service] 支援)，並隨附尋找該資產的Adobe Sensei產生資訊。 傳回的資訊為JSON格式。

[!DNL Asset Compute Service] 可藉由建立以下基礎的自訂應用程式來擴充 [!DNL Adobe Developer App Builder]. 這些自訂應用程式為 [!DNL Project Adobe Developer App Builder] Headless應用程式並執行工作，例如新增自訂轉換工具或呼叫外部API以執行影像操作。

[!DNL Project Adobe Developer App Builder] 是在Adobe上建置和部署自訂Web應用程式的架構 [!DNL `I/O Runtime`]. 若要建立自訂應用程式，開發人員可善用 [!DNL React Spectrum] (Adobe的UI toolkit)、建立微服務、建立自訂事件以及協調API。 另請參閱 [Adobe Developer App Builder檔案](https://developer.adobe.com/app-builder/docs/overview).

此架構所根據的基礎包括：

* 應用程式的模組化僅包含特定工作所需的功能，可讓應用程式彼此分離，並保持輕量級。

* 無伺服器概念 [!DNL Adobe I/O] 執行階段具備許多優點：非同步、高度可擴充、獨立、以工作為基礎的處理，非常適合資產處理。

* 二進位雲端儲存空間提供必要功能，可讓您使用預先簽署的URL參考，個別儲存及存取資產檔案和轉譯，而不需要儲存空間的完整存取許可權。 傳輸加速、CDN快取，以及將運算應用程式與雲端儲存空間並置在一起，有助於存取最佳的低延遲內容。 支援AWS和Azure雲端。

![asset compute服務架構](assets/architecture-diagram.png)

*圖：架構 [!DNL Asset Compute Service] 以及它如何與整合 [!DNL Experience Manager]、儲存和處理應用程式。*

架構包含下列部分：

* **API和協調層** 會接收要求（JSON格式），這些要求會指示服務將來源資產轉換為多個轉譯。 這些請求為非同步，且會以作業ID作為啟用ID傳回。 指示純粹是宣告式的，對於所有標準處理工作（例如縮圖產生、文字擷取），消費者只會指定想要的結果，不會指定處理特定轉譯的應用程式。 一般API功能（例如驗證、分析、速率限制）會使用服務前的AdobeAPI閘道來處理，並管理所有傳送至此處的請求 [!DNL Adobe I/O] 執行階段。 應用程式路由是由協調層動態完成。 使用者端會為特定轉譯定義自訂應用程式，這些轉譯隨附自己的不重複引數集。 應用程式執行可以完全平行化，因為它們是Adobe中的獨立無伺服器函式 [!DNL `I/O Runtime`].

* **處理資產的應用程式** 專門處理特定型別的檔案格式或目標轉譯。 從概念上講，應用程式就像UNIX®管道概念：輸入檔案會轉換為一個或多個輸出檔案。

* **A [通用應用程式庫](https://github.com/adobe/asset-compute-sdk)** 處理常見工作。 例如，下載來源檔案、上傳轉譯、錯誤報告、事件傳送和監視。 此設計可確保應用程式開發保持簡單明瞭，遵循無伺服器概念，互動僅限於本機檔案系統。

<!-- TBD:

* About the YAML file?
* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
