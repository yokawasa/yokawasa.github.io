---
layout: post
status: publish
published: true
title: Azure Searchのアナライザーによるテキスト解析結果を出力するツール - azure-search-ta
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 120
wordpress_url: http://unofficialism.info/posts/?p=120
date: '2017-05-13 12:19:24 +0900'
date_gmt: '2017-05-13 03:19:24 +0900'
categories:
- Azure
tags:
- AzureSearch
- analyzer
---

Azure Searchのアナライザーによるテキスト解析結果を出力する（だけの）ツールを作ってみたのでここで紹介します。その名も[azure-search-ta](https://github.com/yokawasa/azure-search-ta)（ta=**T**est **A**nalyzer）。中身はAzure Searchの[Analyzer API](https://docs.microsoft.com/en-us/azure/search/search-api-2015-02-28-preview#test-analyzer)の出力結果を整形して表示させていているだけの単純なものでありますが、Azure Searchの全文検索チューニングやキーワードにヒットしない原因調査をする際には役に立つと思ってます。「どうしてこのキーワードがひっかからないの？」を突き詰めるには最終的にアナライザのテキスト解析結果と突き合わせる必要があるのと、アナライザーを選択する際にテキスト解析が視覚化されていると判断しやすいだろうと。ツールは2種類で （１）Web UIツールと（２）コマンドラインツール

## Web UI Tool

![azure-search-ta-capture](https://c1.staticflickr.com/5/4169/33813518873_c2d72f5094_c.jpg)
[https://github.com/yokawasa/azure-search-ta](https://github.com/yokawasa/azure-search-ta)

インストールは超簡単。（１）[Github](https://github.com/yokawasa/azure-search-ta)からazure-search-taをclone （２）[azure-search-ta/ui](https://github.com/yokawasa/azure-search-ta/tree/master/ui) 配下のファイルをPHPが動くWebサーバにコピー （３）analyze-api.phpをエディタで開いてお使いのAzure Searchカウント名とAzure Search API Adminキーの値を設定ください。あとはazure-search-ta-ui.htmlにアクセスいただければ上記のようなUIが出力されるはずです。なぜHTML/JSだけではなく間にPHPを挟んでいるのかについて、Azure SearchのAnalyze APIや管理系APIリクエストに位置付けられており、管理系APIはvia CORSでのリクエストを受け付けていないからである。


```shell
$ git clone https://github.com/yokawasa/azure-search-ta.git`
$ vi azure-search-ta/ui/analyze-api.php

$azureSearchAccount="";
$azureSearchApiKey = "";
```

## Command-Line Tool

### 1. インストールと設定

pipでazure-search-taパッケージをインストール。既に古いバージョンをインストール済みでアップデートする際は――upgradeをつけて実行ください。

```shell
$ pip install --user azure-search-ta
```

次に、search.confにお使いのAzure Searchカウント名とAzure Search API Adminキーの値を設定ください。

```shell
# Azure Search Service Name ( never put space before and after = )
SEARCH_SERVICE_NAME=
# Azure Search API Admin Key ( never put space before and after = )
SEARCH_API_KEY=
```

### 2. 使い方

いくつか使い方を紹介します。

```
usage: azure-search-ta [-h] [-v] [-c CONF] [-i INDEX] [-a ANALYZER]
                          [-t TEXT] [-o OUTPUT]`

This program do text analysis and generate formatted output by using Azure
Search Analyze API

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -c CONF, --conf CONF  Azure Search Configuration file. Default:search.conf
  -i INDEX, --index INDEX
                        Azure Search index name
  -a ANALYZER, --analyzer ANALYZER
                        Azure Search analyzer name
  -t TEXT, --text TEXT  A file path or HTTP(s) URL from which the command line
                        reads the text to analyze
  -o OUTPUT, --output OUTPUT
                        Output format ("simple" or "normal"). Default:normal
```

ja.microsoftアナライザーによるテキスト解析結果をnormalモードで出力する例。Analyzer APIはパラメータにインデックス名が必要なので、なんでもいいのでご自分のアカウントに設定されているインデックス名を指定ください。ここではインデックスtaでsimple.txtに対象のテキストを記入して実行しています。

```shell
$ cat sample1.txt
吾輩は猫である

$ azure-search-ta -c ./search.conf -i ta -a ja.microsoft --t sample1.txt
INPUT: 吾輩は猫である
TOKENS: [吾輩] [猫] [ある]
```

Azure Searchにビルトインされているアナライザーでけでなく皆さんが作成した[カスタムアナライザー](https://docs.microsoft.com/en-us/rest/api/searchservice/custom-analyzers-in-azure-search)によるテキスト解析結果も当然出力可能です。以下は、インデックスtacustomにNグラム分割のカスタムアナライザーmy_ngramを作成したとしてmy_ngramアナライザーによるテキスト解析結果を出力する例です。カスタムアナライザーの定義は[Githubページ](https://github.com/yokawasa/azure-search-ta#2-2-create-index-schema-to-analyze-text)のほうに詳しく書いているのでよかったらどうぞ。

```shell
$ cat sample1.txt
吾輩は猫である

$ azure-search-ta -c ./search.conf -i tacustom -a my_ngram --t sample1.txt -o simple
'吾輩' '吾輩は' '吾輩は猫で' '吾輩は猫' '輩は猫であ' '輩は' '輩は猫' '輩は猫で' 'は猫であ' 'は猫で' 'は猫' 'は猫である' '猫であ' '猫で' '猫で ある' 'である' 'であ' 'ある'
```

他には、azure-search-taはインターネット上のページのテキスト解析機能も付いているので、そいつを試してみます。ja.luceneアナライザを使ってhttp://www.yahoo.co.jpトップページの内容を解析します。

```shell
$ azure-search-ta -c ./search.conf -i ta -a ja.lucene --t http://www.yahoo.co.jp -o simple
'html' 'public' 'w' '3' 'c' 'dtd' 'html' '4' '01' 'transitional' 'en' 'http' 'www' 'w' '3' 'org' 'tr' 'html' '4' 'loose' 'dtd' 'yahoo' 'japan' 'ヘルプ' 'yahoo' 'japan' 'トップページ' '機能' '正しく' 'ご' '利用' 'いただく' '下記' '環境' '必要' 'windows' 'internet' 'explorer' '9' '0' '以上' 'chrome' '最新' '版' 'firefox' '最新' '版' 'microsoft' 'edge' 'macintosh' 'safari' '5' '0' '以上' 'internet' 'explorer' '9' '0' '以上' 'ご' '利用' '場合' 'internet' 'explorer' '互換' '表示' '参考' '互換' '表示' '無効' '化' '試し' 'くださる' '東北' '自転車' 'イベント' '参加' '方法' '事前' 'チェック' 'こだわり' 'ご' '当地' 'スイーツ' '取り寄せる' '上海' 'マージャン' '定番' 'ゲーム' '無料' '遊ぶ' 'ニュース' '11' '時' '1' '分' '更新' '韓国' '北' '太陽' '政策' '回帰' '娘' '放置' '熱中' '症' '死なす' '逮捕' '保育' '死亡' '事故' '睡眠' '中' '注意' '三菱' 'ufj' '法人' '融資' '銀行' '集約' 'フレーバ' '水' '人気' '続く' '各国' '大' '規模' 'サイバ' '攻撃' '西武' '菊池' '沢村' '超' '驚異' '被' '打率' '寺島' 'しのぶ' '長男' '超' '英才' '教育' 'もっと' '見る' '記事' '一覧' 'ゴミ' '収集' '車' '子育て' '5' '月' '13' '日' '7' '時' '55' '分' '配信' '産経新聞' 'ショッピング' 'ヤフオク' '旅行' 'ホテル' '予約' 'ニュース' '天気' 'スポーツナビ' 'ファイナンス' 'テレビ' 'gyao' 'y' 'モバゲ' '地域' '地図' '路線' '食べる' 'ログ' '求人' 'アルバイト' '不動産' '自動車' '掲示板' 'ブログ' 'ビューティ' '出会い' '電子' '書籍' '映画' 'ゲーム' '占い' 'サービス' '一覧' 'ログイン' 'id' 'もっと' '便利' '新規' '取得' 'メール' 'メールアドレス' '取得' 'カレンダ' 'カレンダ' '活用' 'ポイント' '確認' 'ログイン' '履歴' '確認' '会社' '概要' '投資' '家' '情報' '社会' '的' '責任' '企業' '行動' '憲章' '広告' '掲載' '採用' '情報' '利用' '規約' '免責' '事項' 'メディア' 'ステートメント' 'セキュリティ' '考え方' 'プライバシ' 'ポリシ' 'copyright' 'c' '2017' 'yahoo' 'japan' 'corporation' 'all' 'rights' 'reserved'
```

もう少しだけ[Github](https://github.com/yokawasa/azure-search-ta)のほうには詳しく書いてあるのでそちらも参考にしてください。