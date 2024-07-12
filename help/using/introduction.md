---
title: ' [!DNL Asset Compute Service] について'
description: 「[!DNL Asset Compute Service] は、複雑さを軽減しスケーラビリティを向上させるクラウドネイティブなアセット処理サービスです」。
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '278'
ht-degree: 100%

---

# [!DNL Asset Compute Service] の概要 {#overview}

[!DNL Asset Compute Service] は、デジタルアセットを処理するためのスケーラブルかつ拡張可能な [!DNL Adobe Experience Cloud] サービスです。 画像、ビデオ、ドキュメントなどのファイル形式を、サムネール、抽出テキストとメタデータ、アーカイブなど、様々なレンディションに変換できます。

開発者は、カスタムアセットアプリケーション（カスタムワーカーとも呼ばれます）をプラグインとして使用し、カスタムユースケースに対応できます。 このサービスは Adobe [!DNL I/O Runtime] で動作します。 Node.js で作成された [!DNL Adobe Developer App Builder] ヘッドレスアプリを通じて拡張できます。 これらのアプリでは、外部 API を呼び出して画像操作を実行したり、[!DNL Adobe Sensei] サポートを活用したりするなどのカスタム操作を実行できます。

[!DNL Adobe Developer App Builder] は、Adobe [!DNL I/O Runtime] 上にカスタム web アプリケーションを作成およびデプロイして、Adobe Experience Cloud ソリューションを拡張するためのフレームワークです。 カスタムアプリケーションを作成するために、開発者は [!DNL React Spectrum]（アドビの UI ツールキット）を活用し、マイクロサービスを作成し、カスタムイベントを作成し、API を調整できます。 詳しくは、[Adobe Developer App Builder のドキュメント](https://developer.adobe.com/app-builder/docs/overview/)を参照してください。

>[!NOTE]
>
>現在、[!DNL Asset Compute Service] は Adobe [!DNL Experience Manager] as a [!DNL Cloud Service] 経由でのみ使用できます。 管理者は、[!DNL Asset Compute Service] を呼び出してアセットを処理に渡すことができる処理プロファイルを作成します。 [アセットマイクロサービスと処理プロファイルの使用](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)を参照してください。

## [!DNL Asset Compute Service] のサポート対象ユースケース {#possible-use-cases-benefits}

[!DNL Asset Compute Service] は、基本的な画像処理、Adobe アプリケーション固有の変換、複雑なビジネス要件をオーケストレーションするカスタムアプリケーションの作成など、いくつかの一般的なビジネスユースケースをサポートしています。

[!DNL Asset Compute] web サービスを使用すると、様々なファイルタイプのサムネールや、[サポートされているファイル形式](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/file-format-support)の高画質な画像レンダリングを生成できます。 詳しくは、[カスタム設定でサポートされるユースケース](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)を参照してください。

>[!NOTE]
>
>このサービスはアセットストレージを提供しません。 ユーザーがアセットストレージを提供し、クラウドストレージ内のソースファイルとレンディションファイルの場所を参照して提供します。

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [ [!DNL Adobe Experience Manager]  as a  [!DNL Cloud Service] のアセットマイクロサービスを使用したアセット処理の概要](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview)。
>* [Adobe Developer App Builder のドキュメント](https://developer.adobe.com/app-builder/docs/overview)。
>* [処理でサポートされるファイル形式](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/file-format-support)。

<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
