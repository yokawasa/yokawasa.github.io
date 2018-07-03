---
layout: post
status: publish
published: true
title: Wikipediaデータベースを元にAzure Searchインデックスを生成する
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 29
wordpress_url: http://unofficialism.info/posts/?p=29
date: '2015-06-09 08:33:23 +0900'
date_gmt: '2015-06-08 23:33:23 +0900'
categories:
- Azure
tags:
- AzureSearch
- wikipedia
- perl
---

Wikipediaのコンテンツは Creative Commons Licenseおよび GNU Free Documentation Licenseの下にライセンスされておりWikipedia財団は再配布や再利用のために惜しげもなくこの貴重なデータベースのダンプファイル（XMLファイル）を一般提供している。全文検索の検証で大量のデータが必要なときこのWikipediaのような生きたデータを使えるのは非常に有りがたい。これはこのWikipediaデータベースダンプ（日本語）を元にAzure Searchインデックスを生成してみましょうというお話。

- [Wikipedia:データベースダウンロード](http://ja.wikipedia.org/wiki/Wikipedia:%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E3%83%80%E3%82%A6%E3%83%B3%E3%83%AD%E3%83%BC%E3%83%89)
- [ウィキメディア財団による全プロジェクトのデータベース・ダンプ](http://dumps.wikimedia.org/)
- [Wikipedia日本語版のダンプデータレポジトリ](http://dumps.wikimedia.org/jawiki/)([日本語最新版](http://dumps.wikimedia.org/jawiki/latest/))

## 利用するWikipedia XMLファイルとその定義

[最新版日本語レポジトリ](http://download.wikimedia.org/jawiki/latest/)には複数のXMLファイルが用意されているがここでは全ページのタイトル、ディスクリプションといった要約データを集約しているファイル[jawiki-latest-abstract.xml](http://dumps.wikimedia.org/jawiki/latest/jawiki-latest-abstract.xml)を利用する。XMLのフォーマットは次のとおり。この中からtitle, url, abstractを抽出してAzure Searchに投入する。

```xml
<doc>
<title>Wikipedia: 自然言語</title>
<url>http://ja.wikipedia.org/wiki/%E8%87%AA%E7%84%B6%E8%A8%80%E8%AA%9E</url>
<abstract>自然言語（しぜんげんご、）とは、人間によって日常の意思疎通のために用いられる、文化的背景を持って自然に発展してきた記号体系である。大別すると音声に>よる話し言葉と文字や記号として書かれる書き言葉がある。</abstract>
<links>
<sublink linktype="nav"><anchor>概要</anchor><link>http://ja.wikipedia.org/wiki/%E8%87%AA%E7%84%B6%E8%A8%80%E8%AA%9E#.E6.A6.82.E8.A6.81</link></sublink>
<sublink linktype="nav"><anchor>関連項目</anchor><link>http://ja.wikipedia.org/wiki/%E8%87%AA%E7%84%B6%E8%A8%80%E8%AA%9E#.E9.96.A2.E9.80.A3.E9.A0.85.E7.9B.AE</link></sublink>
</links>
</doc>
```

尚、実際にダウンロードしてみるとわかると思うがこのファイルはサイズが比較的大きく集約されているドキュメント数も実に多い。カウントしてみたところ現時点（2015/06/09）で969541　件あった。[Azure Searchの料金プラン](http://azure.microsoft.com/en-us/pricing/details/search/)のうちFreeプランは最大ドキュメント数が10,000であることからここで利用する料金プランはFreeではなく標準プランを選択する必要がある。

## Index Schema

インデックス名はwikipedia、フィールドはキーフィールドのためのitemidフィールドと上記wikipedia XMLファイルのtitle, url, abstractを格納するための3フィールドを定義。

```json
{
    "name": "wikipedia",
    "fields": [
        { "name":"itemid", "type":"Edm.String", "key": true, "searchable": false },
        { "name":"title", "type":"Edm.String", "filterable":false, "sortable":false, "facetable":false},
        { "name":"abstract", "type":"Edm.String", "filterable":false, "sortable":false, "facetable":false, "analyzer":"ja.lucene" },
        { "name":"url", "type":"Edm.String", "sortable":false, "facetable":false }
    ]
}
```

## Azure Search投入用JSONデータの生成

Wikipedia XMLファイルからAzure Search投入用のJSONデータを生成するスクリプト([xml2json.pl](https://gist.github.com/yokawasa/f1cb68cd168f50dbf873))を作ってみた。巨大ファイルのDOM構造を一度にオンメモリ展開することは物理的に不可能であるため、[XML::Twigモジュール](http://search.cpan.org/~mirod/XML-Twig-3.49/Twig.pm)を使ってドキュメント毎にコールバック関数からフィールドデータを抽出するように工夫を凝らしている。

```perl
#!/usr/bin/perl -w
use strict;
use XML::Twig;
use Getopt::Std;
use Encode;
use JSON;

my $UPLOAD_THRESHOLD=100;
my %opts=();
getopts('c:o:', \%opts) or die "Wrong Options!\n";

my $inputfile=$opts{'c'};
my $outputpath=$opts{'o'};
usage() if ( $inputfile eq '' || $outputpath eq '' );

my($tmpcounter,$itemcounter, $filecounter) = (0,0,0);
my $itemsarr = [];

my $twig = new XML::Twig(
            twig_handlers => { doc => \&doc }
        );
$twig->parsefile($inputfile);
if ($tmpcounter) {
    flush2json($itemsarr, "items-$filecounter.json");
}

sub doc {
    my($twig, $doc)= @_;
    my $item = {
            itemid => "$itemcounter",
            title=> utf8string($doc->first_child('title')->text),
            abstract=> utf8string($doc->first_child('abstract')->text),
            url =>$doc->first_child('url')->text
        };
    push($itemsarr,$item);
    $tmpcounter++; $itemcounter++;
    if ($tmpcounter % $UPLOAD_THRESHOLD == 0 ){
        flush2json($itemsarr, "items-$filecounter.json");
        $tmpcounter=0;
        $filecounter++;
        $itemsarr=[];
    }
    $twig->purge;
}

sub flush2json {
    my ($iarr, $f)=@_;
    my $outarr->{'value'}=$iarr;
    my $json = JSON->new();
    my $js = $json->encode($outarr);
    open(P, "> $outputpath/$f") or die "can't open: $outputpath/$f \n";
    print P "$js";
    close P;
}

sub usage {
    print STDERR "Usage: $0 -c <inputxml> -o <outputpath>\n";
    exit(1);
}

sub utf8string {
    my $s= shift;
    if (utf8::is_utf8($s)) {
        return encode('utf-8', $s);
    }
    return $s;
}

__END__
```

次のように-cパラメータでインプットファイルを、-oパラメータでJSON出力のためのパスを指定する。実行すると-oパラメータで指定したパスにitems-N.json (N:0以上の整数)なファイルが大量に生成される。現時点でのデータから生成されたJSONファイルを数えてみたところitems-0.json ...items-9692.jsonで合計9693個だった。

```sh
$ xml2json.pl -c ./jawiki-latest-abstract.xml -o <path to output>
```

## Azure SearchにJSONデータを投入

上記ステップで生成されたJSONファイルからAzure SearchにPOSTするための簡易スクリプト（[AzureSearch-AddItemsFromFile.sh](https://gist.github.com/yokawasa/24847d378b370187c03e#file-azuresearch-additemsfromfile-sh)）を作る。

```sh
#!/bin/sh

if [ $# -ne 1 ]
then
    echo "Usage: $0 <json.file>"
    exit 1;
fi

SERVICE_NAME='<Azure Search Service Name>'
API_VER='2015-02-28-Preview'
ADMIN_KEY='<API KEY>'
CONTENT_TYPE='application/json'
INDEX_NAME='wikipedia'
URL="https://$SERVICE_NAME.search.windows.net/indexes/$INDEX_NAME/docs/index?api-version=$API_VER"
JSON=`cat $1`

curl \
 -H "Content-Type: $CONTENT_TYPE"\
 -H "api-key: $ADMIN_KEY"\
 -XPOST $URL -d "$JSON"
```

最後に大量に生成されたJSONファイル群をファイル名を１つずつこのスクリプトの第一引数に指定して実行していく。1万近いファイルなので全て投入が完了するまでに少々時間はかかります。

```sh
for i in `ls -1tr *.json`;do echo $i; ./AzureSearch-AddItemsFromFile.sh $i;done
```
