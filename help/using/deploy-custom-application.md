---
title: 部署 [!DNL Asset Compute Service] 自訂應用程式
description: 部署 [!DNL Asset Compute Service] 自訂應用程式。
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '170'
ht-degree: 0%

---

# 部署自訂應用程式 {#deploy-custom-application}

若要部署您的應用程式，請使用[aio應用程式部署](https://github.com/adobe/aio-cli#aio-appdeploy)命令。 在終端機中，命令會顯示存取自訂應用程式的URL。 URL的格式為`https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`。

若要在不重新部署應用程式的情況下取得相同的URL，請使用[`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action)命令。

在[處理設定檔（位於 [!DNL Experience Manager] 作為 [!DNL Cloud Service]](https://experienceleague.adobe.com/zh-hant/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)）中使用URL將您的應用程式與[!DNL Experience Manager]作為[!DNL Cloud Service]整合。

確保您的App Builder專案和工作區對應至[!DNL Experience Manager]，作為您想要使用動作的[!DNL Cloud Service]環境。 它有不同的開發、測試和生產環境。 您可以檢查在Adobe Developer App Builder應用程式根目錄之ENV檔案中定義的`AIO_runtime_*`認證，以驗證環境。 例如，若要部署至`Stage`工作區，`AIO_runtime_namespace`的格式為`xxxxxx_xxxxxxxxx_stage`。 若要以[!DNL Cloud Service]生產環境形式與[!DNL Experience Manager]整合，請使用您Adobe Developer App Builder `Production`工作區中的應用程式URL。

>[!CAUTION]
>
>請勿在關鍵[!DNL Experience Manager]環境中使用個人工作區。

>[!MORELIKETHIS]
>
>* [瞭解並管理 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments)中的環境。
