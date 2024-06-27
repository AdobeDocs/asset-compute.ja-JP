---
title: 「[!DNL Adobe Asset Compute Service] ユーザーガイド」
description: このドキュメントの内容は次のとおりです [!DNL Asset Compute Service] カスタムコードの概要、開発方法、管理方法、デプロイ方法、トラブルシューティング方法などのタスクを実行できます。
exl-id: 5acf87d1-a391-4802-bfce-e367fc8564df
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '206'
ht-degree: 64%

---

# [!DNL Asset Compute Service] について

[!DNL Asset Compute Service] は、デジタルアセットを処理するためのスケーラブルかつ拡張可能な Adobe Experience Cloud サービスです。画像、ビデオ、ドキュメントなどのファイル形式を、サムネール、抽出したテキストとメタデータ、アーカイブなど、様々なレンディションに変換できます。開発者は、を使用して構築された、カスタムユースケースに対処するためのカスタムアプリケーション（カスタムワーカーとも呼ばれます）をプラグインできます [Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview) サーバーレスAdobeで実行中 [[!DNL I/O Runtime]](https://developer.adobe.com/runtime/).

このドキュメントでは、カスタムコードの開発、管理、デプロイ、トラブルシューティングの方法など、[!DNL Asset Compute Service] に関連するトピックについて説明しています。知って知る [!DNL Asset Compute Service] が、このに移動 [はじめに](introduction.md). 関連トピック [サービスの機能](introduction.md#possible-use-cases-benefits).

[!DNL Asset Compute Service] は、多くのファイル形式の変換をサポートしており、多くの Adobe サービスと統合されています。のリストを参照してください [サポートされているファイル形式と統合サービス](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

[ [!DNL Adobe Experience Manager]  as a  [!DNL Cloud Service] で利用可能なアセットマイクロサービス機能](https://experienceleague.adobe.com/ja/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview)および [!DNL Experience Manager] のマイクロサービスの使用方法については、概要を参照してください。

[!DNL Asset Compute Service] の拡張機能は、拡張機能開発者からの貢献を歓迎するオープン開発モデルの下、[github.com/adobe](https://github.com/adobe) で開発されています。カスタムアプリケーションの開発、作成、テスト、デプロイに関連するコンポーネントはすべてオープンソースです。Compute Service への貢献の方法と場所については、[こちら](contribute-to-compute-service.md)を参照してください。

<!--
Possible to record the below info here in this landing page to centralize the miscellaneous info about Asset Compute Service?
 List of dependencies and requirements SDK, CLI, Devtools, etc.? Or may be a link to the prerequisites.
 Introduction video when Tech Marketing team shares one.
-->
