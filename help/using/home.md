---
title: '"[!DNL Adobe Asset Compute Service] 使用手冊」'
description: 本檔案涵蓋 [!DNL Asset Compute Service] 簡介、如何開發、管理、部署和疑難排解您的自訂程式碼等工作。
exl-id: 5acf87d1-a391-4802-bfce-e367fc8564df
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '206'
ht-degree: 0%

---

# 關於 [!DNL Asset Compute Service]

[!DNL Asset Compute Service] 是可擴充的Adobe Experience Cloud服務，用於處理數位資產。 它可以將影像、視訊、檔案和其他檔案格式轉換為不同的轉譯，包括縮圖、擷取的文字和中繼資料、封存等等。 開發人員可外掛自訂應用程式（也稱為自訂工作角色），以處理自訂使用案例，使用建置 [Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview) 並以無伺服器Adobe執行 [[!DNL I/O Runtime]](https://developer.adobe.com/runtime/).

本檔案涵蓋 [!DNL Asset Compute Service] 如何開發、管理、部署和疑難排解自訂程式碼等主題。 瞭解什麼 [!DNL Asset Compute Service] 是，移至此 [簡介](introduction.md). 另請參閱 [此服務可為您做什麼](introduction.md#possible-use-cases-benefits).

[!DNL Asset Compute Service] 支援轉換多種檔案格式，並整合許多Adobe服務。 檢視清單 [支援的檔案格式和整合服務](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

檢視關於以下內容的概述： [中可用的資產微服務功能 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/zh-hant/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview) 以及如何在中使用微服務 [!DNL Experience Manager].

[!DNL Asset Compute Service] 擴充性是在的開放式開發模型下開發 [github.com/adobe](https://github.com/adobe) 歡迎擴充功能開發人員協助撰寫。 所有與開發、建立、測試和部署自訂應用程式相關的元件都是開放原始碼。 另請參閱 [如何以及在何處貢獻運算服務](contribute-to-compute-service.md).

<!--
Possible to record the below info here in this landing page to centralize the miscellaneous info about Asset Compute Service?
 List of dependencies and requirements SDK, CLI, Devtools, etc.? Or may be a link to the prerequisites.
 Introduction video when Tech Marketing team shares one.
-->
