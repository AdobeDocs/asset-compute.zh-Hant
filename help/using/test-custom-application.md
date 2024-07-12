---
title: 測試並偵錯 [!DNL Asset Compute Service] 自訂應用程式
description: 測試並偵錯 [!DNL Asset Compute Service] 自訂應用程式。
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '775'
ht-degree: 0%

---

# 測試和偵錯自訂應用程式 {#test-debug-custom-worker}

## 執行自訂應用程式的單元測試 {#test-custom-worker}

在您的電腦上安裝[Docker案頭](https://www.docker.com/get-started)。 若要測試自訂背景工作，請在應用程式的根目錄執行下列命令：

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

此命令會執行自訂單元測試架構，以便在專案中Asset compute應用程式動作，如下所述。 它是透過`package.json`檔案中的組態連結。 也可以進行JavaScript單元測試，例如Jest。 `aio app test`會同時執行。

[aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency)外掛程式內嵌為自訂應用程式應用程式中的開發相依性，因此不需要安裝在組建/測試系統上。

### 應用程式單元測試架構 {#unit-test-framework}

asset compute應用程式單元測試架構可讓您測試應用程式，而不需編寫任何程式碼。 它仰賴應用程式轉譯檔案原則的來源。 必須設定特定檔案和資料夾結構，以使用測試來源檔案、選用引數、預期的轉譯和自訂驗證指令碼來定義測試案例。 依預設，會比較轉譯的位元組相等。 此外，您可使用簡單的JSON檔案輕鬆模擬外部HTTP服務。

### 新增測試 {#add-tests}

專案根層級的`test`資料夾內應該有測試。 每個應用程式的測試案例應位於路徑`test/asset-compute/<worker-name>`中，每個測試案例各有一個資料夾：

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

請檢視[範例自訂應用程式](https://github.com/adobe/asset-compute-example-workers/)以取得一些範例。 詳細參考如下。

### 測試輸出 {#test-output}

位於Adobe Developer App Builder應用程式根目錄的`build`目錄內含自訂應用程式的詳細測試結果和記錄。 這些詳細資料也會顯示在`aio app test`命令的輸出中。

### 模擬外部服務 {#mock-external-services}

您可以為測試案例建立`mock-<HOST_NAME>.json`檔案，並將HOST_NAME設為要模擬的特定主機，藉此在您的動作中模擬外部服務呼叫。 使用案例範例是對S3進行個別呼叫的應用程式。 新的測試結構如下所示：

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

模擬檔案是JSON格式的http回應。 如需詳細資訊，請參閱[此檔案](https://www.mock-server.com/mock_server/creating_expectations.html)。 如果要模擬多個主機名稱，請定義多個`mock-<mocked-host>.json`檔案。 以下是名為`mock-google.com.json`的`google.com`的範例模型檔案：

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

範例`worker-animal-pictures`包含其互動之Wikimedia服務的[模擬檔案](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json)。

#### 在測試案例間共用檔案 {#share-files-across-test-cases}

如果您在多個測試中共用`file.*`、`params.json`或`validate`指令碼，Adobe建議使用相對符號連結。 它們受到Git支援。 請務必為您共用的檔案指定唯一的名稱，因為您可能有不同的檔案。 在以下範例中，測試會將一些共用檔案與其自己的檔案混合及比對：

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### 測試預期的錯誤 {#test-unexpected-errors}

錯誤測試案例不應包含預期的`rendition.*`檔案，且應在`params.json`檔案中定義預期的`errorReason`。

>[!NOTE]
>
>如果測試案例未包含預期的`rendition.*`檔案，且未在`params.json`檔案內定義預期的`errorReason`，則會假設為包含任何`errorReason`的錯誤案例。

錯誤測試案例結構：

```json
<error_test_case>/
    file.jpg
    params.json
```

帶有錯誤原因的引數檔：

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

檢視[Asset compute錯誤原因](https://github.com/adobe/asset-compute-commons#error-reasons)的完整清單和說明。

## 對自訂應用程式進行偵錯 {#debug-custom-worker}

下列步驟顯示如何使用Visual Studio Code偵錯自訂應用程式。 它允許檢視即時記錄、點選中斷點和逐步執行程式碼，以及在每次啟用時即時重新載入本機程式碼變更。

`aio`現成可用的自動化了其中的許多步驟。 移至[Adobe Developer App Builder檔案](https://developer.adobe.com/app-builder/docs/getting_started/first_app)中的「偵錯應用程式」一節。 目前，以下步驟包含因應措施。

1. 從GitHub和選用的[ngrok](https://www.npmjs.com/package/ngrok)安裝最新的[wskdebug](https://github.com/apache/openwhisk-wskdebug)。

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 在JSON檔案中新增使用者設定。 它持續使用舊的Visual Studio程式碼偵錯工具。 新的有[個問題](https://github.com/apache/openwhisk-wskdebug/issues/74)，與wskdebug： `"debug.javascript.usePreview": false`。
1. 關閉任何透過`aio app run`開啟的應用程式例項。
1. 使用`aio app deploy`部署最新的程式碼。
1. 僅使用`aio asset-compute devtool`執行Asset computeDevtool。 保持開啟。
1. 在Visual Studio程式碼編輯器中，將下列偵錯組態新增至`launch.json`：

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   從`aio app deploy`的輸出擷取`ACTION NAME`。

1. 從執行/偵錯設定中選取`wskdebug worker`並按播放圖示。 請等候它啟動，直到它在&#x200B;**[!UICONTROL 偵錯主控台]**&#x200B;視窗中顯示&#x200B;**[!UICONTROL 準備啟動]**。

1. 在Devtool中按一下&#x200B;**[!UICONTROL 執行]**。 您可以看到在Visual Studio程式碼編輯器中執行的動作，而且記錄會開始顯示。

1. 在程式碼中設定中斷點。 然後再次執行，應該會點選。

任何程式碼變更都會即時載入，並在下次啟用時立即生效。

>[!NOTE]
>
>自訂應用程式中的每個請求都有兩個啟用專案。 第一個要求是Web動作，會在SDK程式碼中非同步呼叫自身。 第二個啟用是點選程式碼的啟用。
