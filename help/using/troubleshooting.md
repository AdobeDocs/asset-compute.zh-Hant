---
title: 疑難排解 [!DNL Asset Compute Service]
description: 疑難排解和偵錯自訂應用程式，使用 [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '273'
ht-degree: 0%

---

# 疑難排解 {#troubleshoot}

可協助您針對Asset Compute服務進行疑難排解的一些一般疑難排解提示如下：

* 請確定JavaScript應用程式在啟動時不會當機。 這類當機通常與缺少資料庫或相依性有關。
* 請確定應用程式的所有要安裝的相依性都已參照 `package.json` 檔案。
* 確保任何因失敗而清除的錯誤不會產生隱藏原始問題的錯誤。

* 第一次使用新工具啟動開發人員工具時 [!DNL Asset Compute Service] 整合，如果Asset compute事件日誌未完全設定，則第一個處理請求可能會失敗。 請等候一段時間以設定日誌，然後再傳送另一個請求。
* 確保所有必要的APIAsset compute、Adobe [!DNL I/O Events]、事件管理和執行階段 — 包含在您的Adobe中 [!DNL `I/O Project`] 和Workspace避免 `/register` 或 `/process` 請求錯誤。

## 透過Adobe方式登入問題 [!DNL aio-cli] {#login-via-aio-cli}

如果您無法登入 [!DNL Adobe Developer Console] [流經Adobe [!DNL aio-cli]](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)，然後手動新增開發、測試和部署自訂應用程式所需的認證：

1. 導覽至您的Adobe Developer App Builder專案和工作區，位於 [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) 然後按下 **[!UICONTROL 下載]** 從右上角。 開啟此檔案，並將此JSON儲存至您電腦上的安全位置。

1. 導覽至您Adobe Developer App Builder應用程式中的環境檔案。

1. 新增Adobe [!DNL I/O Runtime] 認證。 取得Adobe [!DNL I/O Runtime] 憑證來自下載的JSON。 認證位於 `project.workspace.services.runtime`. 新增 [!DNL Adobe I/O] 中的執行階段認證 `AIO_runtime_XXX` 變數：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步驟1新增已下載JSON的絕對路徑：

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 設定其餘的 [必要的認證](develop-custom-application.md) 開發人員工具所需。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
