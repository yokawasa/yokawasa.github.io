---
layout: post
status: publish
published: true
title: Simple Lift and Shift using Web App for Containers and Azure Database for MySQL
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 131
wordpress_url: http://unofficialism.info/posts/?p=131
date: '2017-11-23 18:39:51 +0900'
date_gmt: '2017-11-23 09:39:51 +0900'
categories:
- Azure
tags:
- AppServices
- docker
- web app for containers
- mysql
- Container
---

去る10月、11月と[Web App for Containers](https://azure.microsoft.com/ja-jp/services/app-service/containers/)と[Azure Database for MySQL](https://azure.microsoft.com/ja-jp/services/mysql/)をつかったLife and Shiftシナリオのコンテンツを２つばかり作ったのでブログにのせておきます。それぞれRailsとPHPの簡単なLAMP構成アプリを題材に[Web App for Containers](https://azure.microsoft.com/ja-jp/services/app-service/containers/)と[Azure Database for MySQL](https://azure.microsoft.com/ja-jp/services/mysql/)を使ってAzure上で完全マネージドな構成にしましょうというお話です。データストアにMySQLやPostgreSQLをお使いで、シングルコンテナにおさまるアプリであれば、細かい違いはあれどだいたい同じような手順でここで紹介している完全マネージド構成に移行することができると思います。他には、よくある構成としてデータストアにMongoDBのようなNoSQLをお使いのものもありますが、そいつも[Cosmos DB](https://docs.microsoft.com/ja-jp/azure/cosmos-db/) ([Mongo API版](https://docs.microsoft.com/ja-jp/azure/cosmos-db/mongodb-introduction)) + [Web App for Containers](https://azure.microsoft.com/ja-jp/services/app-service/containers/)構成に載せ替えることも可能かと思います。

これらサービスは、パッチあてなどUpdateとランタイム更新のような面倒な作業は当然ながら、標準で冗長化対策、高負荷対策、セキュリティ対策などをやってくれていて、さらに効率的な監視・アラート、CI/CDを可能にする機能を提供しております。使える状況下ならば絶対に使わないと損なわけです。是非ともこれらサービスを活用してオフロードできるところはオフロードして、その分のリソースを皆さんにしかできない新規開発やビジネスベーションにフォーカスいただくことできっと今までよりも幸せになれるのでははなかいと思います。

## Rails + MySQLアプリのLife and Shiftシナリオ
[![](https://image.slidesharecdn.com/webinar-app-and-db-rails-171012211316/95/web-app-for-containers-mysqlrails-1-1024.jpg?cb=1510960998)](//www.slideshare.net/yokawasa/web-app-for-containers-mysqlrails-80754390)
**[Web App for Containers + MySQLでコンテナ対応したRailsアプリを作ろう！](//www.slideshare.net/yokawasa/web-app-for-containers-mysqlrails-80754390)** from **[Yoichi Kawasaki](https://www.slideshare.net/yokawasa)**

## PHP + MySQLアプリのLife and Shiftシナリオ

[![](https://image.slidesharecdn.com/webinar-app-and-db-php-171117134141/95/web-app-for-containers-mysqlphp-1-1024.jpg?cb=1510960691)](//www.slideshare.net/yokawasa/web-app-for-containers-mysqlphp)
**[Web App for Containers + MySQLでコンテナ対応したPHPアプリを作ろう！ ](//www.slideshare.net/yokawasa/web-app-for-containers-mysqlphp)** from **[Yoichi Kawasaki](https://www.slideshare.net/yokawasa)**

## Source Code

上記２シナリオに加えてPython/Django版も加えたアプリソースとCI設定ファイルのレポジトリはこちら

- Ruby on railsアプリとCI: [https://github.com/yokawasa/ci-demo-rails-app](https://github.com/yokawasa/ci-demo-rails-app)
- PHP/WordPressアプリとCI: [https://github.com/yokawasa/WordPress](https://github.com/yokawasa/WordPress)
- Python/DjangoアプリとCI:  [https://github.com/yokawasa/ci-demo-django-app](https://github.com/yokawasa/ci-demo-django-app)
