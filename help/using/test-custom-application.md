---
title: ' [!DNL Asset Compute Service]  カスタムアプリケーションのテストとデバッグ'
description: ' [!DNL Asset Compute Service]  カスタムアプリケーションのテストとデバッグ。'
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '775'
ht-degree: 53%

---

# カスタムアプリケーションのテストとデバッグ {#test-debug-custom-worker}

## カスタムアプリケーションの単体テストの実行 {#test-custom-worker}

[Docker Desktop](https://www.docker.com/get-started) をコンピューターにインストールします。カスタムワーカーをテストするには、アプリケーションのルートで次のコマンドを実行します。

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

このコマンドは、以下に示すように、プロジェクト内のAsset computeアプリケーション アクションに対してカスタム単体テスト フレームワークを実行します。 これは `package.json` ファイル内の設定を通じて接続されます。また、Jest などの JavaScript ユニットテストをおこなうこともできます。この `aio app test` 両方を実行します。

この [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) プラグインは、開発依存関係としてカスタムアプリケーションアプリに埋め込まれているため、ビルド/テストシステムにインストールする必要はありません。

### アプリケーションユニットテストフレームワーク {#unit-test-framework}

asset computeアプリケーション単体テストフレームワークを使用すると、コードを記述せずにアプリケーションをテストできます。 アプリケーションのソースからレンディションへのファイル原則を利用します。テストソースファイル、オプションパラメーター、想定されるレンディション、カスタム検証スクリプトを含むテストケースを定義するには、特定のファイルとフォルダー構造を設定する必要があります。 デフォルトでは、レンディションのバイト等価性が比較されます。さらに、単純な JSON ファイルを使用して外部 HTTP サービスのモックを容易に作成できます。

### テストの追加 {#add-tests}

内でのテストが想定されています `test` プロジェクトのルートレベルのフォルダー。 各アプリケーションのテストケースは、パス `test/asset-compute/<worker-name>` 内にテストケースごとに 1 つのフォルダーとして配置する必要があります。

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

いくつかの例については、[カスタムアプリケーションの例](https://github.com/adobe/asset-compute-example-workers/)を参照してください。以下に詳細を説明します。

### テスト出力 {#test-output}

この `build` Adobe Developer App Builder アプリケーションのルートにあるディレクトリには、カスタムアプリケーションの詳細なテスト結果とログが格納されます。 これらの詳細は、の出力にも表示されます `aio app test` コマンド。

### モック用外部サービス {#mock-external-services}

以下を作成することで、アクション内での外部サービス呼び出しをシミュレートできます。 `mock-<HOST_NAME>.json` テストシナリオ用のファイル。HOST_NAME は、模倣する特定のホストです。 ユースケースの例としては、S3 に対して個別の呼び出しを行うアプリケーションがあります。 新しいテスト構造は次のようになります。

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

モックファイルは、JSON 形式の http 応答です。詳しくは、[こちらのドキュメント](https://www.mock-server.com/mock_server/creating_expectations.html)を参照してください。モック作成対象のホスト名が複数ある場合は、複数の `mock-<mocked-host>.json` ファイルを定義します。以下は、`mock-google.com.json` という名前の、`google.com` のサンプルモックファイルです。

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

例 `worker-animal-pictures` 次を含む [モックファイル](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) （連携するウィキメディアサービス用）

#### テストケース間でのファイルの共有 {#share-files-across-test-cases}

共有する場合、Adobeでは相対シンボリックリンクの使用をお勧めします `file.*`, `params.json` または `validate` 複数のテストのスクリプト。 Git でサポートされています。 共有ファイルには異なる名前をつけることもあるので、必ず一意の名前を付けてください。以下の例では、いくつかの共有ファイルとテスト自体のファイルを組み合わせています。

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

の完全なリストと説明を参照してください [Asset computeエラーの理由](https://github.com/adobe/asset-compute-commons#error-reasons).

## カスタムアプリケーションのデバッグ {#debug-custom-worker}

次の手順は、Visual Studio Code を使用したカスタムアプリケーションのデバッグ方法を示しています。ライブログの確認、ブレークポイントのヒット、コードのステップスルー、ローカルコード変更のライブ再読み込みをアクティベーションのたびにおこなうことができます。

この `aio` これらの手順の多くは、すぐに使用できる自動化機能です。 のアプリケーションのデバッグの節に移動します。 [Adobe Developer App Builder ドキュメント](https://developer.adobe.com/app-builder/docs/getting_started/first_app). 現時点では、以下の手順には回避策が含まれています。

1. GitHub の最新の [wskdebug](https://github.com/apache/openwhisk-wskdebug) とオプションの [ngrok](https://www.npmjs.com/package/ngrok) をインストールします。

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. JSON ファイルのユーザー設定に追加を加えます。 古い Visual Studio Code デバッガーが引き続き使用されます。 新しい方は [いくつかの問題](https://github.com/apache/openwhisk-wskdebug/issues/74) wskdebug の場合： `"debug.javascript.usePreview": false`.
1. 経由で開いているアプリのインスタンスをすべて閉じる `aio app run`.
1. `aio app deploy` を使用して最新のコードをデプロイします。
1. を使用して、Asset compute開発ツールのみを実行します。 `aio asset-compute devtool`. ツールを開いたままにします。
1. Visual Studio Code Editor で、次のデバッグ設定をに追加します `launch.json`:

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

1. 実行／デバッグ設定から `wskdebug worker` を選択し、再生アイコンを押します。表示されるまで開始するのを待ちます **[!UICONTROL アクティベーションの準備完了]** が含まれる **[!UICONTROL デバッグコンソール]** ウィンドウ。

1. 開発者ツールで「**[!UICONTROL run]**」をクリックします。Visual Studio のコードエディターで実行されているアクションが表示され、ログが表示され始めます。

1. コードにブレークポイントを設定します。 その後、もう一度実行すると、ヒットします。

コードの変更内容はすべてリアルタイムで読み込まれ、次回アクティベーションがおこなわれるとすぐに有効になります。

>[!NOTE]
>
>カスタムアプリケーションでは、リクエストごとに 2 つのアクティベーションが存在します。最初のリクエストは、SDK コード内で自分自身を非同期的に呼び出す Web アクションです。2 つ目のアクティベーションは、コードにヒットするものです。
