---
title: 「[!DNL Adobe Asset Compute Service] ユーザーガイド」
description: このドキュメントでは、カスタムコードの概要、開発、管理、デプロイ、トラブルシューティングの方法など、 [!DNL Asset Compute Service]  のタスクについて説明します。
exl-id: 5acf87d1-a391-4802-bfce-e367fc8564df
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '206'
ht-degree: 100%

---

# [!DNL Asset Compute Service] について

[!DNL Asset Compute Service] は、デジタルアセットを処理するためのスケーラブルかつ拡張可能な Adobe Experience Cloud サービスです。 画像、ビデオ、ドキュメントなどのファイル形式を、サムネール、抽出したテキストとメタデータ、アーカイブなど、様々なレンディションに変換できます。 開発者は、カスタムアプリケーション（カスタムワーカーとも呼ばれます）をプラグインとして使用し、カスタムユースケースに対応できます。これらのユースケースは [Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview) を使用して作成され、サーバーレスの Adobe [[!DNL I/O Runtime]](https://developer.adobe.com/runtime/) で動作します。

このドキュメントでは、カスタムコードの開発、管理、デプロイ、トラブルシューティングの方法など、[!DNL Asset Compute Service] に関連するトピックについて説明しています。 [!DNL Asset Compute Service] について詳しくは、この[概要](introduction.md)を参照してください。 また、[サービスの内容](introduction.md#possible-use-cases-benefits)も参照してください。

[!DNL Asset Compute Service] は、多くのファイル形式の変換をサポートしており、多くの Adobe サービスと統合されています。 [サポートされているファイル形式と統合サービスのリスト](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/file-format-support)を参照してください。

[ [!DNL Adobe Experience Manager]  as a  [!DNL Cloud Service] で利用可能なアセットマイクロサービス機能](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview)および [!DNL Experience Manager] のマイクロサービスの使用方法については、概要を参照してください。

[!DNL Asset Compute Service] の拡張機能は、拡張機能開発者からの貢献を歓迎するオープン開発モデルの下、[github.com/adobe](https://github.com/adobe) で開発されています。 カスタムアプリケーションの開発、作成、テスト、デプロイに関連するコンポーネントはすべてオープンソースです。 Compute Service への貢献の方法と場所については、[こちら](contribute-to-compute-service.md)を参照してください。

<!--
Possible to record the below info here in this landing page to centralize the miscellaneous info about Asset Compute Service?
 List of dependencies and requirements SDK, CLI, Devtools, etc.? Or may be a link to the prerequisites.
 Introduction video when Tech Marketing team shares one.
-->
