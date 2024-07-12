---
title: 設定 [!DNL Asset Compute Service]所需的開發環境
description: 供 [!DNL Asset Compute Service] 開始建立和測試自訂程式碼的開發人員環境設定。
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '324'
ht-degree: 1%

---

# 設定開發人員環境 {#create-dev-environment}

若要建立可讓您為[!DNL Asset Compute Service]開發的設定，請遵循這些需求和指示。

1. [取得[!DNL Adobe Developer App Builder]的存取權和認證](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials)。

1. [設定本機環境](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up)和必要的工具。

1. 還有其他一些工具可協助您開始順利開發，這些工具包括：

   * [Git](https://git-scm.com/)
   * [Docker案頭](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) （不建議使用v14 LTS、奇數版本）和[NPM](https://www.npmjs.com)。 OS X HomeBrew的使用者可以執行`brew install node`來安裝兩者。 否則，請從[NodeJS下載頁面](https://nodejs.org/en/)下載
   * IDE適合NodeJS，Adobe建議使用[Visual Studio Code (VS Code)](https://code.visualstudio.com)，因為它是偵錯工具支援的IDE。 您可以使用任何其他IDE作為程式碼編輯器，但尚不支援進階用法（例如，偵錯工具）
   * 安裝最新Adobe[[!DNL aio-cli]](https://github.com/adobe/aio-cli) (`aio`)
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. 確定符合[必要條件](/help/using/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## 設定App Builder專案 {#create-App-Builder-project}

1. 確定[!DNL Experience Cloud]組織中有系統管理員或開發人員角色。 系統管理員以[Admin Console](https://adminconsole.adobe.com/overview)設定此角色。

1. 登入[Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis)。 確認您與[!DNL Experience Manager]屬於相同的[!DNL Experience Cloud]組織且為[!DNL Cloud Service]整合。 如需Adobe Developer Console的詳細資訊，請前往[主控台檔案](https://developer.adobe.com/developer-console/docs/guides/)。

1. [建立App Builder專案](https://developer.adobe.com/app-builder/docs/getting_started/first_app/)。 按一下&#x200B;**[!UICONTROL 從範本建立新專案]** > **[!UICONTROL 專案]**。 選取「App Builder」。 它會建立具有兩個工作區的新App Builder專案： `Production`和`Stage`。 視需要新增其他工作區，例如`Development`。

1. 在App Builder專案中，選取工作區並訂閱Asset compute所需的服務。 按一下&#x200B;**新增至專案** > **API**&#x200B;並新增`Asset Compute`、`IO Events`和`IO Events Management`服務。 新增第一個API時，它會提示您建立私密金鑰。 將此資訊儲存在電腦上，因為您需要此金鑰才能使用開發人員工具測試您的自訂應用程式。

## 下一步 {#next-step}

設定好您的環境後，您就可以[建立自訂應用程式](develop-custom-application.md)。

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
