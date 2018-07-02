---
layout: post
status: publish
published: true
title: cUrlコマンドで始める簡単Azure Search
author:
  display_name: Yoichi Kawasaki
  login: yoichi
  email: yokawasa@gmail.com
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 5
wordpress_url: http://unofficialism.info/posts/?p=5
date: '2015-06-05 06:32:14 +0900'
date_gmt: '2015-06-04 21:32:14 +0900'
categories:
- Azure
tags:
- AzureSearch
- curl
- API
- JSON
---

cUrlはUNIX/Linux系では有名なURLを使ったデータ送受信コマンドで手軽にREST系処理を実行するときにとても重宝している。そんなcUrlコマンドを使ってAzure Searchをお手軽に使ってみようというお話。

## はじめに

まだの人はAzureポータルよりAzure Searchサービスを作成してください。「[ポータルでの Azure Search サービスの作成](https://azure.microsoft.com/ja-jp/documentation/articles/search-create-service-portal/)」に優しく手順が書かれているのでご参考に。料金プランは無料と標準プランがあるがテストであれば無料プランで十分。まずはAPIキーまで取得ください。API実行のためにはAPIキーが必要。

## cURLでSearch Service REST APIを実行

[Search Service REST API](https://msdn.microsoft.com/en-us/library/azure/dn798935.aspx)の中からいくつか代表的なAPIをピックアップしてcUrlでクエリを組み立ててみる。ここではインデックス新規作成、そこにいくつかドキュメントを追加、そしてドキュメントを検索する・・といった基本的なシナリオを実行する。ポイントとしてはcUrlの-Hオプションでヘッダ定義、-XオプションでHTTPメソッド指定、-dオプションでリクエストボディを指定する・・・といったところ。尚、下記サンプルでは現時点（2015-06-05）で最新のAPIバージョン[2015-02-28-Preview](https://azure.microsoft.com/en-us/documentation/articles/search-api-2015-02-28-preview/)を使用している。

### 1. インデックス新規作成

articlesという名前のブログ記事を格納するためのインデックスを作成する。インデックス生成には[Create Index (Azure Search Service REST API)](https://msdn.microsoft.com/en-us/library/azure/dn798941.aspx)を利用する。

### 2. ドキュメントの追加

1で作成したインデックスにいくつかドキュメントをアップロードしてみる。データソースは[Microsoft Azure Japan Blog](http://blogs.msdn.com/b/windowsazurej/)の[RSS](http://blogs.msdn.com/b/windowsazurej/atom.aspx)。インデックスへのドキュメントのアップロードAPIは[Add, Update or Delete Documents (Azure Search Service REST AP](https://msdn.microsoft.com/en-us/library/azure/dn798930.aspx)を参照ください。

### 3. ドキュメントの検索

追加されたドキュメントを実際に検索してみる。下記サンプルは"DocumentDB"キーワードで検索をして上位５つの結果を取得、レスポンスフィールドとしてitemidとtitleフィールドを指定している。ドキュメントの検索APIは[Search Documents (Azure Search Service REST API)](https://msdn.microsoft.com/en-us/library/azure/dn798927.aspx)を参照ください。

ただし上記の実行結果は次のように日本語マルチバイト文字がすべて"\uHHHH"のようなユニコードの16進表現でエンコードされている。さらに1行レスポンスであるため結果のJSON構造が非常に分かりづらく可読性のない状況となっている。
`

{"@odata.context":"https://yoichikaecdemo0.search.windows.net/indexes('articles')/$metadata#docs(itemid,title)","@odata.count":2,"value":[{"@search.score":1.1298637,"itemid":"1","title":"DocumentDB \u306e\u65b0\u3057\u3044\u30a4\u30f3\u30dd\u30fc\u30c8 \u30aa\u30d7\u30b7\u30e7\u30f3"},{"@search.score":1.058217,"itemid":"4","title":"DocumentDB SDK \u3067\u30d1\u30fc\u30c6\u30a3\u30b7\u30e7\u30f3\u5206\u5272\u3092\u30b5\u30dd\u30fc\u30c8"}]}
`

そこで上記の結果出力に対して（１）JSONを見やすくPretifyして、（２）16進数表現でエンコードされたUnicodeをデコードする２つの処理を加えたい。まずは（１）JSON Pretifyだが、Python2.6以上の環境であればで「python -mjson.tool」コマンドでJSONをPretify出力することができる。次に（２）16進数表現のデコードは[StackOverflow](http://ja.stackoverflow.com/questions/217/%E3%82%A8%E3%82%B9%E3%82%B1%E3%83%BC%E3%83%97%E3%81%95%E3%82%8C%E3%81%9F%E6%97%A5%E6%9C%AC%E8%AA%9E%E6%96%87%E5%AD%97%E5%88%97%E3%82%92%E3%83%87%E3%82%B3%E3%83%BC%E3%83%89%E3%81%97%E3%81%9F%E3%81%84)で見つけたワンライナー「perl -Xpne 's/\\u([0-9a-fA-F]{4})/chr(hex($1))/eg'」を使用する。先のcUrlコマンドにこの２つの処理をパイプでつなげて加えてみたのが下記コード。

以下の実行結果のとおり無事JSON Pretify＋デコードされた出力となった。

`

{

    "@odata.context": "https://yoichikaecdemo0.search.windows.net/indexes('articles')/$metadata#docs(itemid,title)",

    "@odata.count": 2,

    "value": [

        {

            "@search.score": 1.1298637,

            "itemid": "1",

            "title": "DocumentDB の新しいインポート オプション"

        },

        {

            "@search.score": 1.058217,

            "itemid": "4",

            "title": "DocumentDB SDK でパーティション分割をサポート"

        }

    ]

}
`

## REST UIツール

今回cUrlコマンドでのお手軽にAPI実行を紹介したがREST系処理の実行ということであればFiddlerやPostmanのようなツールのほうがエンコード・デコード、JSON/XMLの整形表示など面倒な処理を自動でやってくれる分楽だったりする。特にPostmanはおすすめ。

- [Azure Search で Chrome Postman を使用する方法](https://azure.microsoft.com/ja-jp/documentation/articles/search-chrome-postman/)
- [Azure Search で Telerik Fiddler を使用する方法](https://azure.microsoft.com/ja-jp/documentation/articles/search-fiddler/)
