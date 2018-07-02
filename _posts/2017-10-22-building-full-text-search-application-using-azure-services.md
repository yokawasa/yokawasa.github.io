---
layout: post
status: publish
published: true
title: Azure サービスを活用して作るフルマネージドな全文検索アプリケーション
author:
  display_name: Yoichi Kawasaki
  login: yoichi
  email: yokawasa@gmail.com
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 129
wordpress_url: http://unofficialism.info/posts/?p=129
date: '2017-10-22 13:35:42 +0900'
date_gmt: '2017-10-22 04:35:42 +0900'
categories:
- Azure
tags:
- AzureSearch
- DocumentDB
- CosmosDB
- Javascript
- SPA
---

これは9/29 Azure Web Seminar 「Azure サービスを活用して作るフルマネージドな全文検索アプリケーション」のフォローアップ記事です。なかなか暇ができず少々時間が経過してしまいました。

**[Azure サービスを活用して作るフルマネージドな全文検索アプリケーション](//www.slideshare.net/yokawasa/azure-80659652)** from **[Yoichi Kawasaki](https://www.slideshare.net/yokawasa)**

## Sample Application & Source Code

セミナーで紹介したサンプルアプリはAzure公式サイトに載せてある代表的なサービスのFAQデータを元にしたHTML/CSS/JavascriptによるQ＆Aナレジッジベース検索のシングルページアプリケーションです。検索エンジンにAzure Searchを使い、データソースにCosmos DBを使いAzure SearchのCosmosDB Indexerでクローリングする構成にしてます。ソースコードと設定手順は以下Githubプロジェクトにアップしてあります。もしバグや設定手順等でご質問があればGithubでIssue登録いただければ時間を見つけて対応させていただきます。

![](https://raw.githubusercontent.com/yokawasa/azure-search-qna-demo/master/img/screenshots.png)

Source Code: [https://github.com/yokawasa/azure-search-qna-demo/](https://github.com/yokawasa/azure-search-qna-demo/)

## Demo: AI Digital Media Search

セミナー中に紹介した非構造化データの全文検索デモとして紹介したAI Digital Media Searchアプリケーション。メディア x 音声認識 x 機械翻訳 x 全文検索全てを絡めた面白いアプリケーションなのでこちらでデモ動画とソースコードを共有します。またこのアプリはAzure PaaSサービスを組み合わせてプレゼンテーションレイヤー(Web App for Container)のみならずデータ生成部分（AMS, Functions, Logic App）も全てサーバレスで実現しているのでこのエリアのサンプルアプリとしてもとても良いものになっていると思います。

Source Code: [https://github.com/shigeyf/ai-digitalmedia](https://github.com/shigeyf/ai-digitalmedia)

## AzureSearch.js - Azure Search UIライブラリ

AzureSearch.jsはAzure SearchのUIライブラリで、Azure Searchプロダクトチーム主要開発者により開始されたOSSライブラリです。TypeScriptで書かれているのでとても読みやすく、また、ライブラリが提供するオブジェクト操作により非常に短いコードでサーチボックス、結果出力、ページネーション、ファセット、サジェスションなどで構成されるサーチ用UIを簡単に組み立てることが可能です。なかなかいけているライブラリにもかかわらず、あまり世の中に知られていないのはもったいないと思いセミナーの最後で紹介させていただきました。これ使わない手はないです。手っ取り早くは、下記のAzureSearch.jsアプリテンプレートジェネレータページで皆さんのAzure SearchアカウントのQueryKeyとインデックススキーマ（JSONフォーマット）を入力するとAzureSearch.jsアプリの雛形が生成されますので、そこから始めるのがよいかと思います。

- [AzureSearch.jsプロジェクトトップ＠Github](https://github.com/Yahnoosh/AzSearch.js)
- [デモアプリサイト](https://azsearchstore.azurewebsites.net/realestate.html<br />
)
- [AzureSearch.jsアプリのテンプレートジェネレータ](http://azsearchstore.azurewebsites.net/azsearchgenerator/index.html)

END
