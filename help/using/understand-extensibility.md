---
title: 瞭解如何延伸 [!DNL Asset Compute Service]
description: 何時及如何擴充 [!DNL Asset Compute Service] 功能以進行自訂資產處理。
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 0%

---

# 擴充性簡介 {#introduction-to-extensibilty}

[在 [!DNL Experience Manager] 中處理設定檔為 [!DNL Cloud Service]](https://experienceleague.adobe.com/zh-hant/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview)，可解決許多轉譯需求，例如轉換為格式以及調整影像大小。 更複雜的業務需求可能需要符合組織需求的自訂解決方案。 可透過建立從[!DNL Experience Manager]中的處理設定檔呼叫的自訂應用程式來擴充[!DNL Asset Compute Service]。 這些自訂應用程式符合[支援的使用案例](https://experienceleague.adobe.com/zh-hant/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)。

>[!NOTE]
>
>[!DNL Asset Compute Service]僅可與[!DNL Experience Manager]搭配[!DNL Cloud Service]使用。

自訂應用程式是Headless [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder)應用程式。 透過[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk)和Adobe Developer App Builder開發人員工具，使用自訂應用程式延伸[!DNL Asset Compute Service]變得簡單。 這些工具可讓開發人員專注於商業邏輯。 建立自訂應用程式，就像建立無伺服器的Adobe[!DNL I/O Runtime]動作一樣簡單。 這是單一Node.js JavaScript函式。 [基本自訂應用程式範例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)可加以說明。

## 先決條件和布建需求 {#prerequisites-and-provisioning}

確定您符合下列先決條件：

* 您的電腦上已安裝Adobe Developer App Builder工具。
* [!DNL Experience Cloud]組織。 如需詳細資訊，請前往[開始您的App Builder歷程](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials)。
* 體驗組織必須將[!DNL Experience Manager]設為[!DNL Cloud Service]已啟用。
* [!DNL Adobe Experience Cloud]組織是[!DNL Adobe Developer App Builder]開發人員偷窺計畫的一部分。 移至[如何套用存取權](https://developer.adobe.com/app-builder/docs/overview/getting_access)。
* 確保開發人員的組織中開發人員角色或管理員許可權。
* 確定已在本機安裝Adobe[[!DNL aio-cli]](https://github.com/adobe/aio-cli)。

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
