---
title: ' [!DNL Asset Compute Service]  カスタムアプリケーションのテストとデバッグ'
description: ' [!DNL Asset Compute Service]  カスタムアプリケーションのテストとデバッグ。'
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: f199cecfe4409e2370b30783f984062196dd807d
workflow-type: tm+mt
source-wordcount: '775'
ht-degree: 98%

---

# カスタムアプリケーションのテストとデバッグ {#test-debug-custom-worker}

## カスタムアプリケーションのユニットテストの実行 {#test-custom-worker}

[Docker Desktop](https://www.docker.com/get-started) をコンピューターにインストールします。 カスタムワーカーをテストするには、アプリケーションのルートで次のコマンドを実行します。

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

このコマンドにより、以下で説明するように、プロジェクト内の Asset Compute アプリケーションアクションに対してカスタムユニットテストフレームワークが実行されます。 これは `package.json` ファイル内の設定を通じて接続されます。 また、Jest などの JavaScript ユニットテストをおこなうこともできます。 `aio app test` は、両方を実行します。

[aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) プラグインは、カスタムアプリケーションアプリに開発依存関係として埋め込まれているので、ビルド／テストシステムにインストールする必要はありません。

### アプリケーションユニットテストフレームワーク {#unit-test-framework}

Asset Compute アプリケーションユニットテストフレームワークを使用すると、コードを記述することなく、アプリケーションをテストできます。 アプリケーションのソースからレンディションへのファイル原則を利用します。 テストソースファイル、オプションパラメーター、予想されるレンディション、カスタム検証スクリプトを含むテストケースを定義するには、特定のファイルおよびフォルダー構造を設定する必要があります。 デフォルトでは、レンディションのバイト等価性が比較されます。 さらに、単純な JSON ファイルを使用して外部 HTTP サービスのモックを容易に作成できます。

### テストの追加 {#add-tests}

テストは、プロジェクトのルートレベルにある `test` フォルダー内で行われることが想定されます。 各アプリケーションのテストケースは、パス `test/asset-compute/<worker-name>` 内にテストケースごとに 1 つのフォルダーとして配置する必要があります。

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

いくつかの例については、[カスタムアプリケーションの例](https://github.com/adobe/asset-compute-example-workers/)を参照してください。 以下に詳細を説明します。

### テスト出力 {#test-output}

Adobe Developer App Builder アプリのルートにある `build` ディレクトリには、カスタムアプリケーションの詳細なテスト結果とログが格納されます。 また、これらの詳細は、`aio app test` コマンドの出力にも表示されます。

### モック用外部サービス {#mock-external-services}

テストシナリオ用の `mock-<HOST_NAME>.json` ファイルを作成し、HOST_NAME を模倣する特定のホストにすれば、アクション内で外部サービス呼び出しをシミュレートできます。 ユースケースの例として、S3 への個別の呼び出しを行うアプリケーションが挙げられます。 新しいテスト構造は次のようになります。

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

モックファイルは、JSON 形式の http 応答です。 詳しくは、[こちらのドキュメント](https://www.mock-server.com/mock_server/creating_expectations.html)を参照してください。 モック作成対象のホスト名が複数ある場合は、複数の `mock-<mocked-host>.json` ファイルを定義します。 以下は、`mock-google.com.json` という名前の、`google.com` のサンプルモックファイルです。

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

サンプル `worker-animal-pictures` には、やり取りするウィキメディアサービスの[モックファイル](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json)が含まれます。

#### テストケース間でのファイルの共有 {#share-files-across-test-cases}

`file.*`、`params.json` または `validate` スクリプトを複数のテスト間で共有する場合は、相対シンボリックリンクを使用することをお勧めします。 これらは Git でサポートされます。 共有ファイルには異なる名前をつけることもあるので、必ず一意の名前を付けてください。 以下の例では、いくつかの共有ファイルとテスト自体のファイルを組み合わせています。

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

### 予期されるエラーのテスト {#test-unexpected-errors}

エラーテストケースには、予期される `rendition.*` ファイルを含めず、予期される `errorReason` を `params.json` ファイル内で定義してください。

>[!NOTE]
>
>テストケースに期待された `rendition.*` ファイルが含まれておらず、`params.json` ファイル内で期待される `errorReason` が定義されていない場合、`errorReason` のエラーケースと見なされます。

エラーテストケースの構造：

```json
<error_test_case>/
    file.jpg
    params.json
```

エラー理由を含んだパラメーターファイル：

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

[Asset Compute のエラー理由](https://github.com/adobe/asset-compute-commons#error-reasons)の完全なリストと説明を参照してください。

## カスタムアプリケーションのデバッグ {#debug-custom-worker}

次の手順は、Visual Studio Code を使用したカスタムアプリケーションのデバッグ方法を示しています。 ライブログの確認、ブレークポイントのヒット、コードのステップスルー、ローカルコード変更のライブ再読み込みをアクティベーションのたびにおこなうことができます。

`aio` では、これらの手順の多くが標準で自動化されます。 [Adobe Developer App Builder ドキュメント](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#)のアプリケーションのデバッグの節を参照してください。 現時点では、以下の手順には回避策が含まれています。

1. GitHub の最新の [wskdebug](https://github.com/apache/openwhisk-wskdebug) とオプションの [ngrok](https://www.npmjs.com/package/ngrok) をインストールします。

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. JSON ファイルのユーザー設定に追加を行います。 これにより、古い Visual Studio Code デバッガーが引き続き使用されます。 新しいデバッガーには、wskdebug：`"debug.javascript.usePreview": false` に関する[いくつかの問題](https://github.com/apache/openwhisk-wskdebug/issues/74)があります。
1. `aio app run` を通じて開いているアプリのインスタンスをすべて閉じます。
1. `aio app deploy` を使用して最新のコードをデプロイします。
1. `aio asset-compute devtool` を使用して Asset Compute 開発者ツールのみを実行します。 ツールを開いたままにします。
1. Visual Studio Code エディターで、`launch.json` に次のデバッグ設定を追加します。

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

   `aio app deploy` の出力から `ACTION NAME` を取得します。

1. 「run/debugged configuration」から「`wskdebug worker`」を選択し、「再生」アイコンを押します。 **[!UICONTROL デバッグコンソール]**&#x200B;ウィンドウに&#x200B;**[!UICONTROL アクティベーションの準備完了]**&#x200B;と表示されるまで、起動を待ちます。

1. 開発者ツールで「**[!UICONTROL run]**」をクリックします。 実行中のアクションが Visual Studio Code エディターに表示され、ログの表示が開始されます。

1. コードにブレークポイントを設定します。 その後、もう一度実行すると、ヒットするはずです。

コードの変更内容はすべてリアルタイムで読み込まれ、次回アクティベーションがおこなわれるとすぐに有効になります。

>[!NOTE]
>
>カスタムアプリケーションでは、リクエストごとに 2 つのアクティベーションが存在します。 最初のリクエストは、SDK コード内で自分自身を非同期的に呼び出す Web アクションです。 2 つ目のアクティベーションは、コードにヒットするものです。
