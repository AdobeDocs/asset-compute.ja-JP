---
title: ' [!DNL Asset Compute Service]  カスタムアプリケーションのデプロイ'
description: ' [!DNL Asset Compute Service]  カスタムアプリケーションのデプロイ。'
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '170'
ht-degree: 100%

---

# カスタムアプリケーションのデプロイ {#deploy-custom-application}

アプリケーションをデプロイするには、[aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy) コマンドを使用します。ターミナルでこのコマンドを実行すると、カスタムアプリケーションにアクセスするための URL が表示されます。URL は `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]` の形式です。

アプリケーションを再デプロイせずに同じ URL を取得するには、[`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) コマンドを使用します。

この URL を [Adobe  [!DNL Experience Manager]  as a  [!DNL Cloud Service] の処理プロファイル](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)で使用すると、アプリケーションを Adobe [!DNL Experience Manager] as a [!DNL Cloud Service] と統合できます。

App Builder プロジェクトとワークスペースが、アクションを使用する [!DNL Experience Manager] as a [!DNL Cloud Service] 環境に対応していることを確認します。開発、ステージングおよび実稼動用の環境は異なります。環境が正しいか確かめるには、Adobe Developer App Builder アプリケーションのルートにある ENV ファイル内で定義されている `AIO_runtime_*` 資格情報を確認します。例えば、`Stage` ワークスペースにデプロイする場合、`AIO_runtime_namespace` は `xxxxxx_xxxxxxxxx_stage` の形式です。[!DNL Experience Manager] を [!DNL Cloud Service] の実稼動環境として統合するには、Adobe Developer App Builder の `Production` ワークスペースのアプリケーション URL を使用します。

>[!CAUTION]
>
>重要な [!DNL Experience Manager] 環境では個人用ワークスペースを使用しないでください。

>[!MORELIKETHIS]
>
>* [Adobe  [!DNL Experience Manager]  as a  [!DNL Cloud Service] の環境の理解と管理](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments)。
