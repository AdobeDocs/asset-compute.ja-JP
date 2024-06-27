---
title: ' [!DNL Asset Compute Service] の拡張について'
description: カスタムアセット処理を実行するために [!DNL Asset Compute Service] の機能を拡張するタイミングと方法。
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 76%

---

# 拡張機能の概要 {#introduction-to-extensibilty}

形式の変換や画像のサイズ変更など、多くのレンディション要件は、[Adobe  [!DNL Experience Manager]  as a  [!DNL Cloud Service] の処理プロファイル](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview)で対処します。より複雑なビジネス要件の場合は、組織のニーズに合ったカスタムメイドのソリューションが必要になる場合があります。[!DNL Asset Compute Service] は、Adobe [!DNL Experience Manager] の処理プロファイルから呼び出されるカスタムアプリケーションを作成することで拡張することができます。これらのカスタムアプリケーションは、[サポート対象ユースケース ](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use)に対応しています。

>[!NOTE]
>
>[!DNL Asset Compute Service] は、Adobe [!DNL Experience Manager] as a [!DNL Cloud Service] でのみ使用できます。

カスタムアプリケーションは、ヘッドレスな [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) アプリです。[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) および Adobe Developer App Builder 開発者ツールを使用すると、[!DNL Asset Compute Service] をカスタムアプリケーションで簡単に拡張できるようになります。これらのツールを使用すると、開発者はビジネスロジックに集中できます。 カスタムアプリケーションの作成は、単純なサーバーレスAdobeの作成と同じくらい簡単です [!DNL I/O Runtime] アクション。 カスタムアプリケーションは 1 つの Node.js JavaScript 関数です。詳しくは、[基本的なカスタムアプリケーションの例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)を参照してください。

## 前提条件とプロビジョニング要件 {#prerequisites-and-provisioning}

次の前提条件を満たしている必要があります。

* Adobe Developer App Builder ツールがコンピューターにインストールされている。
* [!DNL Experience Cloud] 組織である。詳細については、次を参照してください。 [App Builderのジャーニーを開始](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* Experience 組織では、Adobe [!DNL Experience Manager] as a [!DNL Cloud Service] が有効になっている。
* [!DNL Adobe Experience Cloud] 組織はの一部です [!DNL Adobe Developer App Builder] 開発者 sneak peek プログラム。 に移動 [アクセスを申請する方法](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* 開発者の組織に開発者ロールまたは管理者権限がある。
* Adobeの確保 [[!DNL aio-cli]](https://github.com/adobe/aio-cli) はローカルにインストールされます。

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
