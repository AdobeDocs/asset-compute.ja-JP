---
title: 「[!DNL Asset Compute Service] HTTP API」
description: 「カスタムアプリケーションを作成するための [!DNL Asset Compute Service] HTTP API」。
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: ht
source-wordcount: '2862'
ht-degree: 100%

---

# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

この API の使用は開発目的に限られています。この API は、カスタムアプリケーションの開発用として提供されます。[!DNL Adobe Experience Manager] as a [!DNL Cloud Service] では、この API を使用して処理情報をカスタムアプリケーションに渡します。詳しくは、[アセットマイクロサービスと処理プロファイルの使用](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)を参照してください。

>[!NOTE]
>
>[!DNL Asset Compute Service] は、Adobe [!DNL Experience Manager] as a [!DNL Cloud Service] でのみ使用できます。

[!DNL Asset Compute Service] HTTP API のクライアントはすべて、次の大まかなフローに従う必要があります。

1. クライアントが IMS 組織で [!DNL Adobe Developer Console] プロジェクトとしてプロビジョニングされます。イベントデータフローを分離するために、クライアント（システムまたは環境）ごとに個別のプロジェクトが必要です。

1. クライアントが、[JWT（サービスアカウント）認証](https://developer.adobe.com/developer-console/docs/guides/)を使用して、テクニカルアカウントのアクセストークンを生成します。

1. クライアントが [`/register`](#register) を 1 回だけ呼び出して、ジャーナル URL を取得します。

1. クライアントが、レンディションの生成対象となるアセットごとに [`/process`](#process-request) を呼び出します。この呼び出しは非同期的に実行されます。

1. クライアントがジャーナルを定期的にポーリングして、[イベントを受け取り](#asynchronous-events)ます。レンディションが正常に処理された場合（`rendition_created` イベントタイプ）またはエラーが発生した場合（`rendition_failed` イベントタイプ）に、要求されたレンディションごとにイベントを受け取ります。

[adobe-asset-compute-client](https://github.com/adobe/asset-compute-client) モジュールを利用すると、Node.js コードで API を簡単に使用できます。

## 認証と承認 {#authentication-and-authorization}

すべての API にはアクセストークン認証が必要です。リクエストには、次のヘッダーを設定する必要があります。

1. bearer トークン（Adobe Developer Console プロジェクトから [JWT 交換](https://developer.adobe.com/developer-console/docs/guides/)を使用して受け取ったテクニカルアカウントトークン）を含んだ `Authorization` ヘッダー。[スコープ](#scopes)については以下で説明します。

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. IMS 組織 ID を含んだ `x-gw-ims-org-id` ヘッダー。

1. [!DNL Adobe Developers Console] プロジェクトからのクライアント ID を含んだ `x-api-key`。

### スコープ {#scopes}

アクセストークンのスコープを確認します。

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

これらのスコープでは、[!DNL Adobe Developer Console] プロジェクトで `Asset Compute`、`I/O Events`、`I/O Management API` の各サービスを購読する必要があります。個々のスコープの内訳は次のとおりです。

* 基本
   * スコープ：`openid,AdobeID`

* Asset Compute
   * メタスコープ：`asset_compute_meta`
   * スコープ：`asset_compute,read_organizations`

* アドビ [!DNL `I/O Events`]
   * メタスコープ：`event_receiver_api`
   * スコープ：`event_receiver,event_receiver_api`

* Adobe [!DNL `I/O Management API`]
   * メタスコープ：`ent_adobeio_sdk`
   * スコープ：`adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 登録 {#register}

[!DNL Asset Compute service] の各クライアント（このサービスをサブスクライブしている一意の [!DNL Adobe Developer Console] プロジェクト）は、処理リクエストをおこなう前に[登録](#register-request)する必要があります。登録ステップは、レンディション処理から非同期イベントを取得するために必要な固有のイベントジャーナルを返します。

ライフサイクルの終わりに、クライアントは[登録解除](#unregister-request)できます。

### 登録リクエスト {#register-request}

この API 呼び出しは、[!DNL Asset Compute] クライアントを設定し、イベントジャーナル URL を提供します。このプロセスはべき等性がある操作であり、クライアントごとに 1 回だけ呼び出す必要があります。このメソッドを再度呼び出せば、ジャーナル URL を取得できます。

| パラメーター | 値 |
|--------------------------|------------------------------------------------------|
| メソッド | `POST` |
| パス | `/register` |
| ヘッダー `Authorization` | すべての[認証関連ヘッダー](#authentication-and-authorization)。 |
| ヘッダー `x-request-id` | オプション。複数のシステムにわたる処理リクエストの一意のエンドツーエンド識別子としてクライアントで設定されます。 |
| リクエスト本文 | 空にする必要があります。 |

### 登録応答 {#register-response}

| パラメーター | 値 |
|-----------------------|------------------------------------------------------|
| MIME タイプ | `application/json` |
| ヘッダー `X-Request-Id` | `X-Request-Id` リクエストヘッダーと同じか、一意に生成されたヘッダー。複数のシステムやサポートリクエスト、またはその両方を識別するために使用します。 |
| 応答本文 | `journal`、`ok`、`requestId` のいずれかのフィールドを含んだ JSON オブジェクト。 |

HTTP ステータスコードは次のとおりです。

* **200 Success**：リクエストが成功した場合。`journal` URL は、`/process` 経由で開始された非同期処理の結果に関する通知を受け取ります。正常に完了すると `rendition_created` イベント、プロセスが失敗した場合は `rendition_failed` イベントのアラートが表示されます。

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 Unauthorized**：有効な[認証](#authentication-and-authorization)がリクエストにない場合に発生します。例えば、無効なアクセストークンや無効な API キーなどがあります。

* **403 Forbidden**：有効な[承認](#authentication-and-authorization)がリクエストにない場合に発生します。例えば、アクセストークンが有効でも、Adobe Developer Console プロジェクト（テクニカルアカウント）が、必要なすべてのサービスをサブスクライブしていない場合などです。

* **429 Too many requests**：このクライアントまたはそれ以外の場合でシステムが過負荷状態になると発生します。クライアントは[指数関数的バックオフ](https://en.wikipedia.org/wiki/Exponential_backoff)を使用して再試行する必要があります。本文は空です。
* **4xx エラー**：その他の任意のクライアントエラーが発生し、登録に失敗した場合。通常、次のような JSON 応答が返されますが、これはすべてのエラーに対して保証されているわけではありません。

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx エラー**：その他の任意のサーバー側エラーが発生し、登録に失敗した場合に発生します。通常、次のような JSON 応答が返されますが、これはすべてのエラーに対して保証されているわけではありません。

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### 登録解除リクエスト {#unregister-request}

この API 呼び出しは、[!DNL Asset Compute] クライアントの登録を解除します。この登録解除が行われると、`/process` を呼び出すことができなくなります。未登録のクライアントに対してこの API 呼び出しを使用すると、`404` エラーが返されます。

| パラメーター | 値 |
|--------------------------|------------------------------------------------------|
| メソッド | `POST` |
| パス | `/unregister` |
| ヘッダー `Authorization` | すべての[認証関連ヘッダー](#authentication-and-authorization)。 |
| ヘッダー `x-request-id` | オプション。クライアントは、複数のシステムにわたる処理リクエストの一意のエンドツーエンド識別子として設定できます。 |
| リクエスト本文 | 空白。 |

### 登録解除応答 {#unregister-response}

| パラメーター | 値 |
|-----------------------|------------------------------------------------------|
| MIME タイプ | `application/json` |
| ヘッダー `X-Request-Id` | `X-Request-Id` リクエストヘッダーと同じか、一意に生成されたヘッダー。複数のシステムやサポートリクエストにわたってリクエストを識別するために使用されます。 |
| 応答本文 | `ok` フィールドと `requestId` フィールドを含んだ JSON オブジェクト。 |

ステータスコードは次のとおりです。

* **200 Success**：登録とジャーナルが見つかり、削除されたときに発生します。

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 Unauthorized**：有効な[認証](#authentication-and-authorization)がリクエストにない場合に発生します。例えば、無効なアクセストークンや無効な API キーなどがあります。

* **403 Forbidden**：有効な[承認](#authentication-and-authorization)がリクエストにない場合に発生します。例えば、アクセストークンが有効でも、Adobe Developer Console プロジェクト（テクニカルアカウント）が、必要なすべてのサービスをサブスクライブしていない場合などです。

* **404 Not found**：このステータスは、指定した資格情報が登録解除されたか無効な場合に表示されます。

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 Too many requests**：システムが過負荷になっている場合に発生します。クライアントは[指数関数的バックオフ](https://en.wikipedia.org/wiki/Exponential_backoff)を使用して再試行する必要があります。本文は空です。

* **4xx エラー**：その他の任意のクライアントエラーが発生し、登録解除に失敗した場合に発生します。通常、次のような JSON 応答が返されますが、これはすべてのエラーに対して保証されているわけではありません。

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx エラー**：その他の任意のサーバー側エラーが発生し、登録に失敗した場合に発生します。通常、次のような JSON 応答が返されますが、これはすべてのエラーに対して保証されているわけではありません。

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## 処理リクエスト {#process-request}

`process` 操作では、リクエスト内の指示に基づいてソースアセットを複数のレンディションに変換するジョブを送信します。正常完了（イベントタイプ `rendition_created`）または任意のエラー（イベントタイプ `rendition_failed`）に関する通知がイベントジャーナルに送信されます。このジャーナルは、`/process` リクエストを行う前に [`/register`](#register) を 1 回使用して取得する必要があります。形式が正しくないリクエストは即座に失敗し、400 エラーコードが返されます。

バイナリは、Amazon AWS S3 の事前署名済み URL や Azure Blob Storage SAS URL などの URL を使用して参照されます。`source` アセットの読み取り（`GET` URL）とレンディションの書き込み（`PUT` URL）の両方に使用されます。これらの署名済み URL の生成はクライアントが担当します。

| パラメーター | 値 |
|--------------------------|------------------------------------------------------|
| メソッド | `POST` |
| パス | `/process` |
| MIME タイプ | `application/json` |
| ヘッダー `Authorization` | すべての[認証関連ヘッダー](#authentication-and-authorization)。 |
| ヘッダー `x-request-id` | オプション。クライアントは、複数のシステムにわたる処理リクエストを追跡するために、一意のエンドツーエンド識別子を設定できます。 |
| リクエスト本文 | 以下に説明する処理リクエスト JSON 形式にする必要があります。処理するアセットと生成するレンディションに関する指示を提供します。 |

### 処理リクエスト JSON {#process-request-json}

`/process` のリクエスト本文は、次の大まかなスキーマに従う JSON オブジェクトです。

```json
{
    "source": "",
    "renditions" : []
}
```

使用可能なフィールドは次のとおりです。

| 名前 | 種類 | 説明 | 例 |
|--------------|----------|-------------|---------|
| `source` | `string` | 処理するソースアセットの URL。オプション。リクエストしたレンディション形式に基づきます（例：`fmt=zip`）。 | `"http://example.com/image.jpg"` |
| `source` | `object` | 処理するソースアセットの記述。以下の [source オブジェクトのフィールド](#source-object-fields)の説明を参照してください。オプション。リクエストしたレンディション形式に基づきます（例：`fmt=zip`）。 | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | ソースファイルから生成するレンディション。各レンディションオブジェクトでは、[レンディション命令](#rendition-instructions)をサポートします。必須。 | `[{ "target": "https://....", "fmt": "png" }]` |

`source` は、URL と見なされる `<string>` か、追加フィールドを含んだ `<object>` のいずれかにすることができます。次のような似たバリエーションがあります。

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### source オブジェクトのフィールド {#source-object-fields}

| 名前 | 種類 | 説明 | 例 |
|-----------|----------|-------------|---------|
| `url` | `string` | 処理するソースアセットの URL。必須。 | `"http://example.com/image.jpg"` |
| `name` | `string` | ソースアセットファイル名。MIME タイプが検出されない場合は、名前にファイル拡張子が使用される可能性があります。URL パスで指定したファイル名よりも優先されます。また、バイナリリソースの `content-disposition` ヘッダー内のファイル名よりも優先されます。デフォルトは「file」です。 | `"image.jpg"` |
| `size` | `number` | ソースアセットファイルのサイズ（バイト単位）。バイナリリソースの `content-length` ヘッダーよりも優先されます。 | `10234` |
| `mimetype` | `string` | ソースアセットファイルの MIME タイプ。バイナリリソースの `content-type` ヘッダーよりも優先されます。 | `"image/jpeg"` |

### 完全な `process` リクエストの例 {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## 処理応答 {#process-response}

`/process` リクエストは、基本的なリクエスト検証に基づいて、成功または失敗を即座に返します。実際のアセット処理は非同期でおこなわれます。

| パラメーター | 値 |
|-----------------------|------------------------------------------------------|
| MIME タイプ | `application/json` |
| ヘッダー `X-Request-Id` | `X-Request-Id` リクエストヘッダーと同じか、一意に生成されたヘッダー。複数のシステムやサポートリクエストにわたってリクエストを識別するために使用されます。 |
| 応答本文 | `ok` フィールドと `requestId` フィールドを含んだ JSON オブジェクト。 |

ステータスコード:

* **200 Success**：リクエストが正常に送信された場合。応答 JSON には `"ok": true` が含まれています。

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Invalid request**：リクエストが不適切に構造化されている場合（例えば、JSON ペイロードに必須フィールドがない場合）。応答 JSON には `"ok": false` が含まれています。

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 Unauthorized**：有効な[認証](#authentication-and-authorization)がリクエストにない場合です。例えば、無効なアクセストークンや無効な API キーなどがあります。
* **403 Forbidden**：有効な[承認](#authentication-and-authorization)がリクエストにない場合です。例えば、アクセストークンが有効でも、Adobe Developer Console プロジェクト（テクニカルアカウント）が、必要なすべてのサービスをサブスクライブしていない場合などです。
* **429 Too many requests**：この特定のクライアントまたは全体的な需要により、システムが過負荷になった際に発生します。クライアントは[指数関数的バックオフ](https://en.wikipedia.org/wiki/Exponential_backoff)を使用して再試行できます。本文は空です。
* **4xx エラー**：その他の任意のクライアントエラーが発生した場合です。通常、次のような JSON 応答が返されますが、これはすべてのエラーに対して保証されているわけではありません。

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx エラー**：その他の任意のサーバー側エラーが発生した場合です。通常、次のような JSON 応答が返されますが、これはすべてのエラーに対して保証されているわけではありません。

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

ほとんどのクライアントは、401 や 403 などの設定の問題や、400 などの無効なリクエストを&#x200B;*除き*、エラーが発生すると[指数バックオフ](https://en.wikipedia.org/wiki/Exponential_backoff)を使用して同じリクエストを再試行する傾向があります。429 応答による通常のレート制限とは別に、一時的なサービス停止または制限により、5xx エラーが発生する可能性があります。その場合は、しばらくしてから再試行することをお勧めします。

すべての JSON 応答（存在する場合）には、`X-Request-Id` ヘッダーと同じ値の `requestId` が含まれます。アドビでは、ヘッダーが常に存在するたので、ヘッダーから読み取ることをお勧めします。また、`requestId` は、処理リクエストに関連するすべてのイベントでも `requestId` として返されます。クライアントは、この文字列の形式についていかなる仮定も行ってはなりません。これは、不透明な文字列識別子です。

## 後処理へのオプトイン {#opt-in-to-post-processing}

[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) では、一連の基本的な画像後処理オプションをサポートしています。カスタムワーカーは、レンディションオブジェクトの `postProcess` フィールドを `true` に設定して、後処理を明示的にオプトインすることができます。

次のユースケースがサポートされています。

* 切り抜きは、crop.w、crop.h、crop.x、crop.y による制限が定義された長方形へのレンディションです。切り抜きの詳細は、レンディションオブジェクトの `instructions.crop` フィールドで指定します。
* 幅、高さまたはその両方を使用して画像のサイズを変更する。`instructions.width` と `instructions.height` では、レンディションオブジェクト内で定義します。幅または高さのみを使用してサイズを変更する場合は、一方の値だけを設定します。Compute Service では縦横比を保持します。
* JPEG 画像の画質を設定する。`instructions.quality` では、レンディションオブジェクト内で定義します。画質レベル 100 は最高画質を表し、数値が小さいほど画質が低下します。
* インターレース画像を作成する。`instructions.interlace` では、レンディションオブジェクト内で定義します。
* ピクセルに適用するスケールを調整することで、DPI を設定してデスクトップパブリッシング用にレンダリングサイズを調整する。`instructions.dpi` では、レンディションオブジェクト内で定義し、DPI 解像度を変更します。ただし、異なる解像度で同じサイズになるように画像のサイズを変更する場合は、`convertToDpi` 指示を使用します。
* レンダリング後の幅または高さが指定のターゲット解像度（DPI）で元の画像と同じになるように画像のサイズを変更する。`instructions.convertToDpi` では、レンディションオブジェクト内で定義します。

## アセットへの透かしの適用 {#add-watermark}

[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) では、PNG、JPEG、TIFF、GIF の各画像ファイルへの透かしの追加をサポートしています。透かしは、レンディションの `watermark` オブジェクトのレンディション指示に従って追加されます。

透かしの追加はレンディションの後処理中におこなわれます。アセットに透かしを追加するには、レンディションオブジェクトの `postProcess` フィールドを `true` に設定して、カスタムワーカーが[後処理をオプトイン](#opt-in-to-post-processing)します。ワーカーがオプトインしない場合、リクエストのレンディションオブジェクトに透かしオブジェクトが設定されている場合でも、透かしは適用されません。

## レンディション指示 {#rendition-instructions}

以下は、[`/process`](#process-request) の `renditions` 配列で使用可能なオプションです。

### 共通のフィールド {#common-fields}

| 名前 | 種類 | 説明 | 例 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | レンディションのターゲット形式は、テキスト抽出の場合は `text`、XMP メタデータを xml として抽出する場合は `xmp` にすることもできます。[サポートされる形式](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/file-format-support)を参照してください | `png` |
| `worker` | `string` | [カスタムアプリケーション](develop-custom-application.md) の URL。`https://` URL にする必要があります。このフィールドが存在する場合、カスタムアプリケーションによってレンディションが作成されます。設定されたその他のレンディションフィールドはすべて、カスタムアプリケーションで使用されます。 | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | 生成されたレンディションを HTTP PUT を使用してアップロードする URL。 | `http://w.com/img.jpg` |
| `target` | `object` | 生成されたレンディションの署名済み URL へのマルチパートアップロードの情報。この情報は、この[マルチパートアップロード動作](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html)に対応する [AEM／Oak ダイレクトバイナリアップロード](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html)に使用されます。<br>フィールドは次のとおりです。<ul><li>`urls`：文字列配列。署名済みのパート URL ごとに 1 つの文字列が割り当てられます。</li><li>`minPartSize`：1 つのパート（URL）に使用する最小サイズ</li><li>`maxPartSize`：1 つのパート（URL）に使用する最大サイズ</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | オプション。クライアントは予約スペースを制御し、レンディションイベントにそのまま渡します。クライアントがカスタム情報を追加してレンディションイベントを識別できるようにします。クライアントはいつでも自由に変更できるので、カスタムアプリケーションで変更したり依存したりしないでください。 | `{ ... }` |

### レンディション固有のフィールド {#rendition-specific-fields}

現在サポートされているファイル形式の一覧については、[サポートされているファイル形式](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/file-format-support)を参照してください。

| 名前 | 種類 | 説明 | 例 |
|-------------------|----------|-------------|---------|
| `*` | `*` | [カスタムアプリケーション](develop-custom-application.md)で認識できる高度なカスタムフィールドを追加できます。 | |
| `embedBinaryLimit` | `number`（バイト単位） | レンディションのファイルサイズが指定された値より小さい場合、そのレンディションは作成完了後に送信されるイベントに含まれます。埋め込み可能な最大サイズは 32 KB（32 x 1024 バイト）です。レンディションのサイズが上限 `embedBinaryLimit` を超える場合、レンディションはクラウドストレージ内の場所に配置され、イベントには埋め込まれません。 | `3276` |
| `width` | `number` | 幅（ピクセル単位）。画像レンディションの場合のみ。 | `200` |
| `height` | `number` | 高さ（ピクセル単位）。画像レンディションの場合のみ。 | `200` |
|                   |          | 次の場合、縦横比は常に維持されます。 <ul> <li> `width` と `height` の両方を指定した場合、画像は縦横比を維持しながら指定のサイズに合わせて調整されます </li><li> `width` か `height` のどちらかを指定した場合、結果として生成される画像では縦横比を維持しながら、対応する寸法が使用されます</li><li> `width` も `height` も指定しなかった場合は、元の画像のピクセルサイズが使用されます。ソースのタイプによって異なります。PDF ファイルなど、一部の形式では、デフォルトサイズが使用されます。最大サイズの制限が設定されていることがあります。</li></ul> | |
| `quality` | `number` | JPEG 画質を `1`～`100` の範囲で指定します。画像レンディションの場合にのみ適用できます。 | `90` |
| `xmp` | `string` | XMP メタデータの書き戻しでのみ使用されます。指定したレンディションに書き戻す Base64 エンコード済み XMP です。 | |
| `interlace` | `bool` | これを `true` に設定すると、インターレースの PNG、GIF、プログレッシブ JPEG のいずれかが作成されます。他のファイル形式には影響しません。 | |
| `jpegSize` | `number` | JPEG ファイルのおおよそのサイズ（バイト単位）。これは `quality` 設定よりも優先します。他の形式には影響しません。 | |
| `dpi` | `number` か `object` のどちらかにする必要があります。 | x および y の DPI を設定します。簡単にするために、x と y の両方に適用される、1 つの数値に設定することもできます。画像自体には影響しません。 | `96` か `{ xdpi: 96, ydpi: 96 }` のどちらかにする必要があります。 |
| `convertToDpi` | `number` か `object` のどちらかにする必要があります。 | 物理サイズを維持しながら x および y DPI の値が再サンプリングされます。簡単にするために、x と y の両方に適用される、1 つの数値に設定することもできます。 | `96` か `{ xdpi: 96, ydpi: 96 }` のどちらかにする必要があります。 |
| `files` | `array` | ZIP アーカイブ（`fmt=zip`）に含めるファイルのリスト。各エントリは、URL 文字列か、次のフィールドを持つオブジェクトのどちらかにすることができます。<ul><li>`url`：ファイルをダウンロードするための URL</li><li>`path`：ZIP 内のこのパスにファイルを格納します</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP アーカイブ（`fmt=zip`）の重複処理。デフォルトでは、ZIP 内の同じパスに複数のファイルを格納しようとすると、エラーが発生します。`duplicate` を `ignore` に設定すると、最初のアセットのみが保存され、残りは無視されます。 | `ignore` |
| `watermark` | `object` | [透かし](#watermark-specific-fields)に関する指示が含まれています。 |  |

### 透かし固有のフィールド {#watermark-specific-fields}

PNG 形式が透かしとして使用されます。

| 名前 | 種類 | 説明 | 例 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 透かしの倍率（`0.0`～`1.0`）。`1.0` は、透かしが元画像のサイズと同じ（1:1）であることを意味します。この値を小さくすると透かしのサイズが小さくなります。 | 値が `0.5` の場合は、元のサイズの半分であることを意味します。 |
| `image` | `url` | 透かしに使用する PNG ファイルの URL。 | |

## 非同期イベント {#asynchronous-events}

レンディションの処理が完了した場合、またはエラーが発生した場合は、Adobe [!DNL `I/O Events Journal`] にイベントが送信されます。クライアントは、[`/register`](#register) を通じて提供されたジャーナル URL をリッスンする必要があります。ジャーナル応答は `event` 配列を含んでいます。その要素は各イベントに対応する 1 つのオブジェクトで、そのオブジェクトの `event` フィールドに実際のイベントペイロードが含まれています。

[!DNL Asset Compute Service] のすべてのイベントに対する Adobe [!DNL `I/O Events`] タイプは `asset_compute` です。ジャーナルは、このイベントタイプのみを自動的にサブスクライブし、[!DNL Adobe Developer] イベントタイプに基づいてさらにフィルタリングする必要はありません。このサービス固有のイベントタイプは、イベントの `type` プロパティで得られます。

### イベントタイプ {#event-types}

| イベント | 説明 |
|---------------------|-------------|
| `rendition_created` | 正常に処理およびアップロードされたレンディションごとに送信されます。 |
| `rendition_failed` | 処理またはアップロードに失敗したレンディションごとに送信されます。 |

### イベント属性 {#event-attributes}

| 属性 | 種類 | イベント | 説明 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | イベント送信時のタイムスタンプで、簡略化された拡張 [ISO-8601](https://ja.wikipedia.org/wiki/ISO_8601) 形式（JavaScript の [Date.toISOString()](https://developer.mozilla.org/ja-JP/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString) 関数で定義）で表されます。 |
| `requestId` | `string` | `*` | `/process` に対する元のリクエストのリクエスト ID（`X-Request-Id` ヘッダーと同じ）。 |
| `source` | `object` | `*` | `/process` リクエストの `source`。 |
| `userData` | `object` | `*` | 設定された場合は、`/process` リクエストから生成されるレンディションの `userData`。 |
| `rendition` | `object` | `rendition_*` | `/process` に渡された対応するレンディションオブジェクト。 |
| `metadata` | `object` | `rendition_created` | レンディションの[メタデータ](#metadata)プロパティ。 |
| `errorReason` | `string` | `rendition_failed` | レンディションの失敗[理由](#error-reasons)（存在する場合）。 |
| `errorMessage` | `string` | `rendition_failed` | レンディションの失敗に関する詳細を示すテキスト（存在する場合）。 |

### メタデータ {#metadata}

| プロパティ | 説明 |
|--------|-------------|
| `repo:size` | レンディションのサイズ（バイト単位）。 |
| `repo:sha1` | レンディションの sha1 ダイジェスト。 |
| `dc:format` | レンディションの MIME タイプ。 |
| `repo:encoding` | レンディションがテキストベース形式の場合、レンディションの文字セットエンコーディング。 |
| `tiff:ImageWidth` | レンディションの幅（ピクセル単位）。画像レンディションにのみ存在します。 |
| `tiff:ImageLength` | レンディションの長さ（ピクセル単位）。画像レンディションにのみ存在します。 |

### エラー理由 {#error-reasons}

| 理由 | 説明 |
|---------|-------------|
| `RenditionFormatUnsupported` | 要求されたレンディション形式は、指定されたソースではサポートされていません。 |
| `SourceUnsupported` | タイプはサポートされていても、この特定のソースはサポートされていません。 |
| `SourceCorrupt` | ソースデータが破損しています。空のファイルが含まれています。 |
| `RenditionTooLarge` | `target` で指定された署名済み URL を使用してレンディションをアップロードできませんでした。実際のレンディションサイズは、`repo:size` でメタデータとして提供されています。クライアントはこれを使用して、適切な数の署名済み URL でこのレンディションを再処理します。 |
| `GenericError` | その他の予期しないエラー。 |
