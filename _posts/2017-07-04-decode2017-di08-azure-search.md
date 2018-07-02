---
layout: post
status: publish
published: true
title: de:code 2017 DI08 - Azure Searchセッションフォローアップ
author:
  display_name: Yoichi Kawasaki
  login: yoichi
  email: yokawasa@gmail.com
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 124
wordpress_url: http://unofficialism.info/posts/?p=124
date: '2017-07-04 14:58:05 +0900'
date_gmt: '2017-07-04 05:58:05 +0900'
categories:
- Announcement
- Azure
tags:
- AzureSearch
- Bot
---

de:code 2017で私が担当したAzure Searchセッションのスライドと動画が公開されたので、セッション中でもでお見せしたマテリアルと合わせてここでフォローアップさせていただきます。

セッションスライドはこちら。セッション持ち時間が50分と短い中での説明なのでいろいろとカットしたのですが、Appendixという形でカットしたスライドや補足情報を載せてあります。P47〜65です。

**[[DI08] その情報うまく取り出せていますか? ～ 意外と簡単、Azure Search で短時間で検索精度と利便性を向上させるための方法](//www.slideshare.net/decode2017/di08-azure-search)** from **[de:code 2017](https://www.slideshare.net/decode2017)**

 

その他、セッション中のデモで使用したツールやBotのソースコードは[Github](https://github.com/yokawasa/decode2017)にアップしております。

- [Azure Search Analyzer Test Web UI](https://github.com/yokawasa/azure-search-ta) - 基礎編アナライザーデモで使用したAnalyzer APIの結果表示Viewer
- [Azure Search Explorer](https://github.com/yokawasa/decode2017/tree/master/azure-search-explorer) - 基礎編デモ使った検索クエリ―結果を表示Viewer。あいまい(Fuzzy:N=2)、同義語辞書、スコアリングプロファイルの動作確認時用パラメータ入力可 - azure-search-explorer
- [Azure Search QnA Bot](https://github.com/yokawasa/decode2017/tree/master/azure-search-qna-bot) - 実践編のSlackアプリデモで使用したQ&A Botのソースコード。Microsoft Bot Framework SDKを使用しバックエンドにAzure Searchを使用

最後に、今回のセッションは検索サイド（クエリーを投げてインデックスにヒットして、ランキング処理されて結果が返却されるまで）に絞ってますが、他にもインデックスデータを生成するまでの話、モニタリングやオペレーション的な話、もう少しデープに言語処理的な話など取り上げたいトピックがまだまだあるので時間を見つけて私なりにブログやスライドに整理していきたいと思うております。
