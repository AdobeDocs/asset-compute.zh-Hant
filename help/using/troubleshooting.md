---
title: 疑難排解 [!DNL Asset Compute Service]
description: 使用 [!DNL Asset Compute Service]進行自訂應用程式的疑難排解和偵錯。
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '273'
ht-degree: 0%

---

# 疑難排解 {#troubleshoot}

可協助您針對Asset Compute服務進行疑難排解的一些一般疑難排解提示如下：

* 確保JavaScript應用程式啟動時不會當機。 這類當機通常與缺少資料庫或相依性有關。
* 請確定應用程式的`package.json`檔案參照所有要安裝的相依性。
* 確保任何因失敗而清除的錯誤不會產生隱藏原始問題的錯誤。

* 第一次使用新的[!DNL Asset Compute Service]整合啟動開發人員工具時，如果Asset compute事件日誌未完全設定，則第一個處理請求可能會失敗。 請等候一段時間以設定日誌，然後再傳送另一個請求。
* 請確定您的Adobe[!DNL `I/O Project`]和Workspace中包含所有必要的API-Asset compute、Adobe[!DNL I/O Events]、事件管理和執行階段，以避免`/register`或`/process`個要求錯誤。

## 透過Adobe[!DNL aio-cli]登入問題 {#login-via-aio-cli}

如果您透過Adobe [!DNL aio-cli][&#128279;](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)登入[!DNL Adobe Developer Console] 時發生問題，請手動新增開發、測試和部署自訂應用程式所需的認證：

1. 導覽至您在[Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis)上的Adobe Developer App Builder專案和工作區，然後從右上角按下&#x200B;**[!UICONTROL 下載]**。 開啟此檔案，並將此JSON儲存至您電腦上的安全位置。

1. 導覽至您Adobe Developer App Builder應用程式中的環境檔案。

1. 新增Adobe[!DNL I/O Runtime]認證。 從下載的JSON取得Adobe[!DNL I/O Runtime]認證。 認證在`project.workspace.services.runtime`之下。 在`AIO_runtime_XXX`變數中新增[!DNL Adobe I/O]執行階段認證：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步驟1新增已下載JSON的絕對路徑：

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 設定開發人員工具所需的其他[必要認證](develop-custom-application.md)。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
