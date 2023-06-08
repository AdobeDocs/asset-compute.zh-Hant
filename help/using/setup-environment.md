---
title: 設定所需的開發環境 [!DNL Asset Compute Service]
description: 適用的開發人員環境設定 [!DNL Asset Compute Service] 以開始建立和測試自訂程式碼。
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '357'
ht-degree: 3%

---

# 設定開發人員環境 {#create-dev-environment}

若要建立可讓您為以下專案開發的設定： [!DNL Asset Compute Service]，請遵循這些需求和指示。

1. [取得存取權和認證](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) 的 [!DNL Adobe Developer App Builder].

1. [設定本機環境](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) 以及所需的工具。

1. 還有其他一些工具可協助您開始順利開發，其中包括：

   * [Git](https://git-scm.com/)
   * [Docker案頭](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) （v14 LTS，不建議使用奇數版本）及 [NPM](https://www.npmjs.com). OSX HomeBrew的使用者可以 `brew install node` 以同時安裝兩者。 否則，請從以下網址下載： [NodeJS下載頁面](https://nodejs.org/en/)
   * 我們建議使用適合NodeJS的IDE [Visual Studio Code (VS Code)](https://code.visualstudio.com) 因為它是Debugger支援的IDE。 您可以使用任何其他IDE作為程式碼編輯器，但目前不支援進階用法（例如偵錯工具）
   * 安裝最新的[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. 請務必符合 [必備條件](/help/using/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## 設定App Builder專案 {#create-App-Builder-project}

1. 確定中的系統管理員或開發人員角色 [!DNL Experience Cloud] 組織。 這是由系統管理員在 [Admin Console](https://adminconsole.adobe.com/overview).

1. 登入 [Adobe Developer主控台](https://console.adobe.io/). 確保您屬於相同群組 [!DNL Experience Cloud] 組織作為 [!DNL Experience Manager] as a [!DNL Cloud Service] 整合。 如需Adobe Developer Console的詳細資訊，請參閱 [主控台檔案](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [建立App Builder專案](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). 按一下 **[!UICONTROL 建立新專案]** > **[!UICONTROL 從範本專案]**. 選取App Builder。 它會建立新的App Builder專案，其中包含兩個工作區： `Production` 和 `Stage`. 新增其他工作區，例如 `Development`，視需要而定。

1. 在App Builder專案中，選取工作區並訂閱Asset compute所需的服務。 按一下 **新增至專案** > **API** 並新增 `Asset Compute`， `IO Events`、和 `IO Events Management` 服務。 新增第一個API時，它會提示您建立私密金鑰。 將此資訊儲存在您的電腦上，因為您需要此金鑰才能使用開發人員工具測試您的自訂應用程式。

## 下一步 {#next-step}

現在您的環境已設定完畢，您可以 [建立自訂應用程式](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
