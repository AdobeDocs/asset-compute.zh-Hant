---
title: 架構 [!DNL Asset Compute Service]。
description: HowAPI、應 [!DNL Asset Compute Service] 用程式和SDK可搭配運作，以提供雲端原生資產處理服務。
translation-type: tm+mt
source-git-commit: 0fb256f7d9f83fbae564d9fd52ee6b2f34c5d7e9
workflow-type: tm+mt
source-wordcount: '496'
ht-degree: 0%

---


# 建築 [!DNL Asset Compute Service] {#overview}

它 [!DNL Asset Compute Service] 建立在無伺服器Adobe I/O Runtime平台之上。 它為資產提供Adobe Sensei內容服務支援。 叫用的用戶端( [!DNL Experience Manager] 僅支援雲端服務)會隨附Adobe Sensei為資產所搜尋的資訊。 傳回的資訊為JSON格式。

[!DNL Asset Compute Service] 可擴充，方法是根據建立自訂應用程式 [!DNL Project Firefly]。 這些自訂應用程 [!DNL Project Firefly] 式是無頭應用程式，執行例如新增自訂轉換工具或呼叫外部API以執行影像作業等工作。

[!DNL Project Firefly] 是在執行時期上建立和部署自訂Web應用程式的架 [!DNL Adobe I/O] 構。 若要建立自訂應用程式，開發人 [!DNL React Spectrum] 員可運用（Adobe的UI工具套件）、建立微型服務、建立自訂事件和協調API。 請參 [閱Project Firefly的檔案](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)。

該體系結構的基礎包括：

* 應用程式的模組化——僅包含特定工作所需的功能——可讓應用程式彼此分離，並維持其輕量級。

* Adobe I/O Runtime的無伺服器概念可帶來許多優點：非同步、高可擴充、隔離、以工作為基礎的處理，最適合資產處理。

* 二進位雲端儲存空間提供個別儲存和存取資產檔案和轉譯的必要功能，而不需使用預先簽署的URL參考來存取儲存空間的完整權限。 傳輸加速、CDN快取，以及搭配雲端儲存空間來共同定位計算應用程式，可提供最佳的低延遲內容存取。 支援AWS和Azure雲。

![資產計算服務體系結構](assets/architecture-diagram.png)

*圖：它的架 [!DNL Asset Compute Service] 構及其與、儲存和處 [!DNL Experience Manager]理應用程式的整合方式。*

該體系結構由以下部分組成：

* **API與協調層** （以JSON格式）會接收要求，指示服務將來源資產轉換為多個轉譯。 這些請求是非同步的，並會以啟動ID（亦即「工作ID」）傳回。 說明完全是宣告性的，對於所有標準處理工作（例如縮圖產生、文字擷取），使用者只會指定所需的結果，而不會指定處理特定轉譯的應用程式。 一般API功能（例如驗證、分析、速率限制）是使用服務前面的Adobe API閘道處理，並管理所有前往I/O Runtime的請求。 應用程式路由由協調層動態完成。 客戶端可以為特定轉譯指定自訂應用程式，並包含自訂參數。 應用程式執行可以完全並行，因為它們是I/O Runtime中獨立的無伺服器函式。

* **應用程式** ，以處理專門處理特定檔案格式或目標轉譯的資產。 從概念上講，應用程式與Unix管道概念類似：輸入檔案被轉換成一個或多個輸出檔案。

* **通用 [應用程式庫](https://github.com/adobe/asset-compute-sdk)** ，可處理常用工作，例如下載來源檔案、上傳轉譯、錯誤報告、事件傳送和監控。 這樣，開發應用程式就會盡可能簡單，遵循無伺服器的理念，而且可限制在本機檔案系統互動。

<!-- TBD:

* About the YAML file?
* See [https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->