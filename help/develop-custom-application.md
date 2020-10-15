---
title: 開發對象 [!DNL Asset Compute Service]。
description: 使用建立自訂應用程式 [!DNL Asset Compute Service]。
translation-type: tm+mt
source-git-commit: 127895cf1bab59546f9ba0be2b3b7a935627effb
workflow-type: tm+mt
source-wordcount: '1496'
ht-degree: 0%

---


# 開發自訂應用程式 {#develop}

開始開發自訂應用程式之前：

* 請確保符合所有 [必要](/help/understand-extensibility.md#prerequisites-and-provisioning) 。
* 安裝所需 [的軟體工具](/help/setup-environment.md#create-dev-environment)。
* 請參 [閱設定您的環境](setup-environment.md) ，以確定您已準備好建立自訂應用程式。

## 建立自訂應用程式 {#create-custom-application}

請務必在本 [機安裝Adobe I/O CLI](https://github.com/adobe/aio-cli) 。

1. 若要建立自訂應用程式，請 [建立Firefly應用程式](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli)。 若要這麼做，請在您的 `aio app init <app-name>` 終端中執行。

   如果您尚未登入，此命令會提示瀏覽器要求您使用Adobe ID登入 [Adobe Developer Console](https://console.adobe.io/) 。 請參 [閱此處](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli) ，以取得從cli登入的詳細資訊。

   Adobe建議您登入。 如果您有問題，請依照指示建立應 [用程式，而不需登入](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user)。

1. 登錄後，請按照CLI中的提示操作，並選 `Organization`擇 `Project`、 `Workspace` 和以用於應用程式。 選擇您在設定環境時建立的 [專案和工作區](setup-environment.md)。

   ```sh
   $ aio app init <app-name>
   Retrieving information from Adobe I/O Console..
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. 出現提示時 `Which Adobe I/O App features do you want to enable for this project?`，請至少選擇 `Actions`:

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 出現提示 `Which type of sample actions do you want to create?`時，請務必選取 `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 依照其他提示，在Visual Studio代碼（或您最愛的代碼編輯器）中開啟新應用程式。 它包含自訂應用程式的架構和范常式式碼。

   請在這裡閱讀有關 [Firefly應用程式的主要元件](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application)。

   範本應用程式運用我們的 [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) ，來上傳、下載和協調應用程式轉譯，因此開發人員只需建置自訂應用程式邏輯。 在資料 `actions/<worker-name>` 夾內，檔 `index.js` 案是新增自訂應用程式碼的位置。

如需自 [訂應用程式的範例](#try-sample) ，請參閱自訂應用程式的範例和構想。

### 新增認證 {#add-credentials}

當您在建立應用程式時登錄時，大多數Firefly憑據都會收集到ENV檔案中。 不過，使用開發人員工具需要額外的認證。

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 開發人員工具儲存認證 {#developer-tool-credentials}

用來測試自訂應用程式與實際應用程式的開發人員工具 [!DNL Asset Compute service] ，需要有雲端儲存容器來代管測試檔案，以及接收和顯示應用程式產生的轉譯。

>[!NOTE]
>
>這與雲端服務的雲端儲 [!DNL Adobe Experience Manager] 存區不同。 它僅適用於使用資產計算開發人員工具進行開發和測試。

請確定可存取支援的雲 [端儲存容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。 此容器可視需要由多位開發人員跨不同專案共用。

#### 將憑據添加到ENV檔案 {#add-credentials-env-file}

將開發人員工具的下列認證新增至Firefly專案根目錄的ENV檔案：

1. 將服務新增至Firefly專案時建立的私密金鑰檔案的絕對路徑：

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. 新增S3或Azure儲存憑證。 您只需要存取一個雲端儲存解決方案。

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

## 執行應用程式 {#run-custom-application}

在使用資產計算開發人員工具執行應用程式之前，請正確設定 [憑證](#developer-tool-credentials)。

若要在開發人員工具中執行應用程式，請使用 `aio app run` 命令。 它會將動作部署至Adobe I/O Runtime，並在您的本機電腦上啟動開發工具。 此工具用於在開發期間測試應用程式要求。 以下是範例轉譯請求：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>請勿將旗標 `--local` 與命令一起 `run` 使用。 它無法用於自訂應 [!DNL Asset Compute] 用程式和資產計算開發人員工具。 自訂應用程式由呼叫， [!DNL Asset Compute Service] 無法存取在開發人員本機電腦上執行的動作。

請參 [閱這裡](test-custom-application.md) ，如何測試和除錯您的應用程式。 當您完成自訂應用程式的開發後，請 [部署自訂應用程式](deploy-custom-application.md)。

## 試用Adobe提供的範例應用程式 {#try-sample}

以下是自訂應用程式的範例：

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [工人——動物——圖片](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 範本自訂應用程式 {#template-custom-application}

The [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) is a template application. 它只要複製來源檔案，就會產生轉譯。 此應用程式的內容是在建立aio應用程式時 `Adobe Asset Compute` 收到的範本。

應用程式檔案 [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 會使用來 [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) 源檔案下載、協調每個轉譯處理，並將產生的轉譯上傳回雲端儲存空間。

在應 [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 用程式碼中定義的，是執行所有應用程式處理邏輯的位置。 中的轉譯回呼 `worker-basic` 只會將來源檔案內容複製到轉譯檔案。

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 呼叫外部API {#call-external-api}

在應用程式碼中，您可以進行外部API呼叫，以協助應用程式處理。 下面是調用外部API的示例應用程式檔案。

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

例如，使用 [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) 程式庫，從維基媒體對靜態URL進行擷取 [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) 要求。

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 傳遞自訂參數 {#pass-custom-parameters}

您可以透過轉譯物件傳遞自訂定義的參數。 您可在應用程式內依說明參 [`rendition` 考](https://github.com/adobe/asset-compute-sdk#rendition)。 轉譯物件的範例是：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

存取自訂參數的應用程式檔案範例為：

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

會傳 `example-worker-animal-pictures` 遞自訂參數， [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) 以決定要從維基媒體擷取的檔案。

## 驗證與授權支援 {#authentication-authorization-support}

根據預設，Asset Compute自訂應用程式隨附Firefly應用程式的授權與驗證檢查。 通過將注釋設定 `require-adobe-auth` 為 `true` 中來啟用 `manifest.yml`。

### 存取其他Adobe API {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

將API服務新增至在設定中 [!DNL Asset Compute] 建立的主控台工作區。 這些服務是由產生的JWT存取Token的一部分 [!DNL Asset Compute Service]。 可在應用程式動作物件記憶體取Token和其他認 `params` 證。

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 傳遞協力廠商系統的認證 {#pass-credentials-for-tp}

要處理其他外部服務的憑據，請將這些作為操作的預設參數傳遞。 在傳輸中會自動加密這些檔案。 如需詳細資訊，請參閱「執 [行階段開發人員指南」中的建立動作](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)。 然後在部署期間使用環境變數來設定這些變數。 這些參數可在動作內 `params` 的物件中存取。

在中，設定預設 `inputs` 參數 `manifest.yml`:

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

表達式 `$VAR` 從名為的環境變數中讀取該值 `VAR`。

在開發過程中，除了從調用的殼中設定的變數外，還可以在本地ENV檔案中 `aio` 設定該值，以自動從ENV檔案中讀取環境變數。 在此示例中，ENV檔案的外觀如下：

```CONF
#...
SECRET_KEY=secret-value
```

對於生產部署，您可以在CI系統中設定環境變數，例如使用GitHub動作中的機密。 最後，存取應用程式內的預設參數，如下：

```javascript
const key = params.secretKey;
```

## 調整應用程式大小 {#sizing-workers}

應用程式會在Adobe I/O Runtime的容器中執行， [並限制](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) （可透過下列項目設定） `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

由於資產計算應用程式通常會進行更廣泛的處理，因此您更可能必須調整這些限制，以獲得最佳效能（大到足以處理二進位資產）和效率（而不會因為未使用的容器記憶體而浪費資源）。

執行階段中動作的預設逾時為一分鐘，但可透過設定限制(以毫秒為單 `timeout` 位)來增加。 如果您希望處理較大的檔案，請增加此次。 請考慮下載來源、處理檔案及上傳轉譯所需的總時間。 如果動作逾時，例如在指定逾時限制之前未傳回啟動，則執行階段會放棄容器，而不會重複使用它。

資產計算應用程式在性質上往往與網路和磁碟IO綁定。 必須先下載來源檔案，處理常常會耗費IO，然後產生的轉譯會再次上傳。

動作容器可用的記憶體是以MB為 `memorySize` 單位。 目前這也定義了容器可存取的CPU數量，最重要的是，它是使用Runtime成本的關鍵元素（較大容器的成本較高）。 當您的處理需要更多記憶體或CPU時，請在此處使用較大的值，但請小心不要浪費資源，因為容器越大，總體吞吐量越低。

此外，使用該設定可以控制容器內的動作併發 `concurrency` 性。 這是單個容器（相同動作）所獲取的並行啟動次數。 在此模型中，動作容器就像Node.js伺服器，可接收多個並行請求，但最高限制為此。 如果未設定，則執行階段中的預設值為200，這對較小的Firefly動作非常有用，但對於資產計算應用程式而言，由於其本機處理和磁碟活動較密集，因此通常會過大。 某些應用程式（視其實作而定）也可能無法與並行活動搭配運作。 Asset Compute SDK可將檔案寫入不同的唯一資料夾，以確保個別啟動。

測試應用程式，以找出和的最 `concurrency` 佳數 `memorySize`目。 較大的容器=記憶體限制較高可能允許更多併發，但流量較低也可能會浪費。