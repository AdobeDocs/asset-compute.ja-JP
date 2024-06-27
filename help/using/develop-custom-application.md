---
title: ' [!DNL Asset Compute Service] に対応した開発'
description: ' [!DNL Asset Compute Service] を使用してカスタムアプリケーションを作成します。'
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '1507'
ht-degree: 56%

---

# カスタムアプリケーションの開発 {#develop}

カスタムアプリケーションの開発を始める前に、以下をおこないます。

* すべての[前提条件](/help/using/understand-extensibility.md#prerequisites-and-provisioning)が満たされていることを確認します。
* [必要なソフトウェアツール](/help/using/setup-environment.md#create-dev-environment)をインストールします。
* [環境の設定](setup-environment.md)を参照して、カスタムアプリケーションを作成する準備ができていることを確認します。

## カスタムアプリケーションの作成 {#create-custom-application}

必ずしてください [Adobe aio-cli](https://github.com/adobe/aio-cli) ローカルにインストールされます。

1. カスタムアプリケーションを作成するには、[App Builder プロジェクトを作成](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli)します。それには、次を実行します `aio app init <app-name>` ターミナルの中で。

   まだログインしていない場合は、このコマンドを実行すると、Adobe ID で [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) にログインするように促すメッセージがブラウザーに表示されます。CLI からのログインについて詳しくは、[こちら](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)を参照してください。

   Adobeでは、最初にログインすることをお勧めします。 問題が発生している場合は、手順に従ってください [ログインせずにアプリを作成するには](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. ログイン後、CLI のプロンプトに従って、アプリケーションに使用する `Organization`、`Project`、`Workspace` を選択します。作成したプロジェクトとワークスペースを選択します [環境の設定](setup-environment.md). 「`Which extension point(s) do you wish to implement ?`」というプロンプトが表示されたら、必ず `DX Asset Compute Worker` を選択します。

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. 「`Which Adobe I/O App features do you want to enable for this project?`」というプロンプトが表示されたら、「`Actions`」を選択します。Web アセットは別の認証および承認チェックを使用するので、必ず「`Web Assets`」オプションの選択を解除してください。

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 「`Which type of sample actions do you want to create?`」というプロンプトが表示されたら、必ず `Adobe Asset Compute Worker` を選択します。

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 残りのプロンプトに従い、Visual Studio Code（または、お好きなコードエディター）で新しいアプリケーションを開きます。カスタムアプリケーションの基礎モードとサンプルコードが含まれています。

   [App Builder アプリの主なコンポーネント](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application)については、こちらを参照してください。

   テンプレートアプリケーションは、Adobeのを利用します [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) アプリケーションレンディションのアップロード、ダウンロードおよびオーケストレーションでは、開発者はカスタムアプリケーションロジックの実装だけで済みます。 `actions/<worker-name>` フォルダー内の `index.js` ファイルがカスタムアプリケーションコードの追加先です。

カスタムアプリの例とアイデアについては、[カスタムアプリケーションの例](#try-sample)を参照してください。

### 資格情報の追加 {#add-credentials}

アプリケーションの作成時にログインすると、App Builder 資格情報のほとんどが ENV ファイルに収集されます。ただし、開発者ツールを使用するには、追加の資格情報が必要です。

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 開発者ツールストレージの資格情報 {#developer-tool-credentials}

を使用してカスタムアプリを評価する開発者向けのツール [!DNL Asset Compute service] クラウドストレージコンテナの使用が必要です。 このコンテナは、テストファイルの保存や、アプリで生成されたレンディションの受信と表示に不可欠です。

>[!NOTE]
>
>このコンテナは、のクラウドストレージとは別のものです [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]. これは、Asset Compute の開発者ツールを使用した開発およびテストにのみ当てはまります。

[サポート対象のクラウドストレージコンテナ](https://github.com/adobe/asset-compute-devtool#prerequisites)にアクセスできることを確認してください。このコンテナは、様々な開発者が必要に応じて、様々なプロジェクト用にまとめて使用します。

#### ENV ファイルへの資格情報の追加 {#add-credentials-env-file}

開発ツールの後続の資格情報をに挿入します。 `.env` ファイル。 ファイルはApp Builder プロジェクトのルートにあります。

1. App Builder プロジェクトにサービスを追加する際に作成された、秘密鍵ファイルへの絶対パスを追加します。

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Adobe Developer Console からファイルをダウンロードします。プロジェクトのルートに移動し、右上隅の「すべてをダウンロード」をクリックします。ファイルは「`<namespace>-<workspace>.json`」というファイル名でダウンロードされます。次のいずれかの操作をおこないます。

   * ファイル名を「`console.json`」に変更し、プロジェクトのルートに移動します。
   * オプションとして、Adobe Developer Console 統合 JSON ファイルへの絶対パスを追加できます。このファイルは同じです [`console.json`](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) プロジェクトワークスペースにダウンロードされたファイル。

     ```conf
     ASSET_COMPUTE_INTEGRATION_FILE_PATH=
     ```

1. S3 ストレージか Azure ストレージのいずれかの資格情報を追加します。1 つのクラウドストレージソリューションへのアクセスのみ必要です。

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

>[!TIP]
>
>`config.json` ファイルには資格情報が含まれています。共有を防ぐため、プロジェクト内で、`.gitignore` ファイルに JSON ファイルを追加します。同じことが当てはまります `.env` および `.aio` ファイル。

## アプリケーションの実行 {#run-custom-application}

asset computeデベロッパーツールを使用してアプリケーションを実行する前に、 [資格情報](#developer-tool-credentials).

開発者ツールでアプリケーションを実行するには、`aio app run` コマンドを使用します。アクションをAdobeにデプロイします [!DNL I/O Runtime]ローカルコンピューター上で開発ツールを起動します。 このツールは、開発中にアプリケーションリクエストのテストに使用されます。レンディションリクエストの例を次に示します。

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
>`run` コマンドで `--local` フラグを使用しないでください。では動作しません [!DNL Asset Compute] カスタムアプリケーションとAsset compute開発ツール。 カスタムアプリケーションは、 [!DNL Asset Compute] 開発者のローカル コンピューターで実行されているアクションにアクセスできないサービス。

アプリケーションのテストとデバッグの方法については、[こちら](test-custom-application.md)を参照してください。カスタムアプリケーションの開発が完了したら、[カスタムアプリケーションをデプロイ](deploy-custom-application.md)します。

## アドビ提供のサンプルアプリケーションの試行 {#try-sample}

次のサンプルカスタムアプリケーションが用意されています。

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [worker-animal-pictures](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### テンプレートカスタムアプリケーション {#template-custom-application}

[worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) はテンプレートアプリケーションです。ソースファイルをコピーするだけで、レンディションが生成されます。このアプリのコンテンツは、aio アプリケーションの作成で `Adobe Asset Compute` を選択した場合に受け取るテンプレートになります。

アプリケーションファイル [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) では、[`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) を使用してソースファイルをダウンロードし、各レンディション処理をオーケストレーションして、結果のレンディションをクラウドストレージにアップロードします。

この [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) アプリケーションコード内で定義され、すべてのアプリケーション処理ロジックを実行する場所です。 `worker-basic` のレンディションコールバックは、ソースファイルの内容をレンディションファイルにコピーするだけです。

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 外部 API の呼び出し {#call-external-api}

アプリケーションコードで外部 API を呼び出して、アプリケーションの処理に役立てることができます。外部 API を呼び出すアプリケーションファイルの例を以下に示します。

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

例えば、[`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) では、[`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) ライブラリを使用してウィキメディアの静的 URL に対して取得リクエストを実行しています。

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### カスタムパラメーターの受け渡し {#pass-custom-parameters}

カスタム定義のパラメーターをレンディションオブジェクトに渡すことができます。これらは、アプリケーション内の [`rendition` 手順](https://github.com/adobe/asset-compute-sdk#rendition)で参照できます。レンディションオブジェクトの例を次に示します。

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

カスタムパラメーターにアクセスするアプリケーションファイルの例を次に示します。

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

`example-worker-animal-pictures` では、カスタムパラメーター [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) を渡して、どのファイルをウィキメディアから取得するかを指定しています。

## 認証と承認のサポート {#authentication-authorization-support}

デフォルトでは、Asset computeカスタムアプリケーションには、App Builder プロジェクトの認証チェックと認証チェックが付属しています。 有効にするには、 `require-adobe-auth` 注釈： `true` が含まれる `manifest.yml`.

### 他の Adobe API へのアクセス {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

設定時に作成した [!DNL Asset Compute] コンソールワークスペースに API サービスを追加します。これらのサービスは、[!DNL Asset Compute Service] で生成された JWT アクセストークンの一部です。トークンとその他の資格情報には、アプリケーションアクションの `params` オブジェクト内でアクセスできます。

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### サードパーティ製システムの資格情報の受け渡し {#pass-credentials-for-tp}

他の外部サービスの資格情報を処理するには、それらをアクションのデフォルトのパラメーターとして渡します。 送信時に自動的に暗号化されます。 詳しくは、を参照してください [Adobe I/O Runtime開発者ガイドでのアクションの作成](https://developer.adobe.com/runtime/docs/guides/using/creating_actions/). 次に、デプロイ時に環境変数を使用してそれらを設定します。これらのパラメーターには、アクション内の `params` オブジェクトでアクセスできます。

`manifest.yml` の `inputs` 内にデフォルトパラメーターを設定します。

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

`$VAR` 式は、`VAR` という名前の環境変数から値を読み取っています。

開発時に、ローカルで値を割り当てることができます `.env` ファイル。 その理由は `aio` は環境変数を次の場所から自動的に読み込みます `.env` ファイルと、起動シェルによって設定された変数。 この例では、 `.env` ファイルは次のようになります。

```CONF
#...
SECRET_KEY=secret-value
```

実稼働デプロイメントの場合は、GitHub アクションでのシークレットの使用など、CI システムで環境変数を設定する場合もあります。最後に、アプリケーション内で次のようにデフォルトパラメーターにアクセスします。

```javascript
const key = params.secretKey;
```

## アプリケーションのサイズ調整 {#sizing-workers}

Adobeのコンテナ内でアプリケーションが実行されます [!DNL I/O Runtime] （を使用） [制限](https://developer.adobe.com/runtime/docs/guides/using/system_settings/) は、 `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

asset computeアプリケーションによって広範な処理が行われるので、最適なパフォーマンス（バイナリアセットを処理できる大きさ）と効率性（未使用のコンテナメモリによるリソースの無駄がない）を得るために、これらの制限を調整する必要があります。

Runtime ではアクションのデフォルトのタイムアウトは 1 分ですが、`timeout` の制限値（ミリ秒）を設定して増やすことができます。サイズの大きいファイルを処理することが想定される場合は、この時間を長くしてください。ソースのダウンロード、ファイルの処理およびレンディションのアップロードに要する合計時間を考慮します。 アクションがタイムアウトした場合（指定されたタイムアウト制限の前にアクティベーションが返されない場合）、Runtime はコンテナを破棄し、再利用しません。

本来、Asset computeアプリケーションは、ネットワークとディスクの入力または出力にバインドされる傾向があります。 まずソースファイルをダウンロードする必要があります。 処理では多くの場合リソースが消費され、生成されるレンディションは再度アップロードされます。

アクションコンテナに割り当てるメモリは、 `memorySize` パラメーター。 現在、このパラメーターは、コンテナが CPU アクセスをどの程度受け取るかを定義します。最も重要なのは、Runtime を使用するコストの重要な要素です（コンテナが大きいほどコストがかかります）。 処理により多くのメモリまたは CPU が必要な場合は、ここで大きな値を使用します。コンテナが大きいほど、全体のスループットが低くなるので、リソースを無駄にしないように注意してください。

さらに、この `concurrency` 設定を使用して、コンテナ内でのアクションの同時実行性を制御することができます。この設定は、（同じアクションの）単一のコンテナが同時に受け取るアクティベーションの数です。 このモデルでは、アクションコンテナは、複数の同時リクエストをその上限まで受け取る Node.js サーバーのようなものです。デフォルト `memorySize` ランタイムは 200 MB に設定されており、App Builderの小さなアクションに最適です。 asset computeアプリケーションの場合、ローカルの処理とディスクの使用量が多いため、このデフォルトは過度になる可能性があります。 実装によっては、一部のアプリケーションは、同時実行アクティビティではうまく動作しない場合もあります。asset compute SDK は、ファイルを様々な一意のフォルダーに書き込むことで、アクティベーションが分離されるようにします。

アプリケーションをテストして、`concurrency` と `memorySize` の最適な値を見つけてください。コンテナが大きい、つまりメモリ上限が大きいと、同時実行性が高くなる可能性がありますが、同時に、トラフィック量が少ない場合にリソースが無駄になる可能性があります。
