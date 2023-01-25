---
author: Yoichi Kawasaki
categories:
- Azure
date: "2017-10-22T13:35:42Z"
date_gmt: 2017-10-22 04:35:42 +0900
published: true
status: publish
images: ["/assets/20171022-build-search-app.png"]
tags:
- AzureSearch
- DocumentDB
- CosmosDB
- Javascript
- SPA
title: Developing Full Managed Search Application in Azure
---

これは9/29 Azure Web Seminar 「Azure サービスを活用して作るフルマネージドな全文検索アプリケーション」のフォローアップ記事です。なかなか暇ができず少々時間が経過してしまいました。

[![](https://image.slidesharecdn.com/webinar-azuresearchappdev-20170929v3-171010164213/95/azure-1-1024.jpg?cb=1507883150)](//www.slideshare.net/yokawasa/azure-80659652)
**[Azure サービスを活用して作るフルマネージドな全文検索アプリケーション](//www.slideshare.net/yokawasa/azure-80659652)** from **[Yoichi Kawasaki](https://www.slideshare.net/yokawasa)**

## Sample Application & Source Code

セミナーで紹介したサンプルアプリはAzure公式サイトに載せてある代表的なサービスのFAQデータを元にしたHTML/CSS/JavascriptによるQ＆Aナレジッジベース検索のシングルページアプリケーションです。検索エンジンにAzure Searchを使い、データソースにCosmos DBを使いAzure SearchのCosmosDB Indexerでクローリングする構成にしてます。ソースコードと設定手順は以下Githubプロジェクトにアップしてあります。もしバグや設定手順等でご質問があればGithubでIssue登録いただければ時間を見つけて対応させていただきます。

![](/assets/20171022-build-search-app.png)

Source Code: [https://github.com/yokawasa/azure-search-qna-demo/](https://github.com/yokawasa/azure-search-qna-demo/)

## Demo: AI Digital Media Search

セミナー中に紹介した非構造化データの全文検索デモとして紹介したAI Digital Media Searchアプリケーション。メディア x 音声認識 x 機械翻訳 x 全文検索全てを絡めた面白いアプリケーションなのでこちらでデモ動画とソースコードを共有します。またこのアプリはAzure PaaSサービスを組み合わせてプレゼンテーションレイヤー(Web App for Container)のみならずデータ生成部分（AMS, Functions, Logic App）も全てサーバレスで実現しているのでこのエリアのサンプルアプリとしてもとても良いものになっていると思います。

- Demo Video: [AI Digital Media Search Demo](https://www.youtube.com/watch?v=BvjKuFE2o8s)
- Source Code: [https://github.com/shigeyf/ai-digitalmedia](https://github.com/shigeyf/ai-digitalmedia)

## AzureSearch.js - Azure Search UIライブラリ

AzureSearch.jsはAzure SearchのUIライブラリで、Azure Searchプロダクトチーム主要開発者により開始されたOSSライブラリです。TypeScriptで書かれているのでとても読みやすく、また、ライブラリが提供するオブジェクト操作により非常に短いコードでサーチボックス、結果出力、ページネーション、ファセット、サジェスションなどで構成されるサーチ用UIを簡単に組み立てることが可能です。なかなかいけているライブラリにもかかわらず、あまり世の中に知られていないのはもったいないと思いセミナーの最後で紹介させていただきました。これ使わない手はないです。手っ取り早くは、下記のAzureSearch.jsアプリテンプレートジェネレータページで皆さんのAzure SearchアカウントのQueryKeyとインデックススキーマ（JSONフォーマット）を入力するとAzureSearch.jsアプリの雛形が生成されますので、そこから始めるのがよいかと思います。

- [AzureSearch.jsプロジェクトトップ＠Github](https://github.com/Yahnoosh/AzSearch.js)
- [デモアプリサイト](https://azsearchstore.azurewebsites.net/realestate.html)
- [AzureSearch.jsアプリのテンプレートジェネレータ](http://azsearchstore.azurewebsites.net/azsearchgenerator/index.html)

END
