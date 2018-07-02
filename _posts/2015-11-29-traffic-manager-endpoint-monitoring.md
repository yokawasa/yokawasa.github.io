---
layout: post
status: publish
published: true
title: 今一度Traffic Managerのエンドポイント監視について
author:
  display_name: Yoichi Kawasaki
  login: yoichi
  email: yokawasa@gmail.com
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 52
wordpress_url: http://unofficialism.info/posts/?p=52
date: '2015-11-29 13:30:05 +0900'
date_gmt: '2015-11-29 04:30:05 +0900'
categories:
- Azure
tags:
- TrafficManager
- DNS
---

Azureが提供するDNSによるトラフィックルーティングサービスであるTraffic Managerについて、既にAzure利用ユーザに使い尽くされて新鮮味に欠けるサービスではあるものの、そのエンドポイント監視はTraffic Managerを扱う上でとても重要なことなので今一度そのルールについて整理したい。

Traffic managerの実態はエンドポイントの監視＋ルーティングを行うDNSサービスである。以下digの結果を見ていただいてわかる通りyoichika-demo1.trafficmanager.netという外向きの名前に対してこの時点ではwebappdemo3.cloudapp.netがCNAMEされている。Traffic Managerは利用ユーザが設定したエンドポイントを監視し、その結果に応じて適切なルーティングを行う。ルーティング方法にはフェールオーバー、ラウンドロビン、パフォーマンスの3通りがある。これについて詳しくは「[Traffic Managerのルーティング方法](https://azure.microsoft.com/ja-jp/documentation/articles/traffic-manager-routing-methods/)」を参照いただきたい。
`

; <<>> DiG 9.9.5-4.3ubuntu0.1-Ubuntu <<>> yoichika-demo1.trafficmanager.net +noall +answer

;; global options: +cmd

yoichika-demo1.trafficmanager.net. 30 IN CNAME  webappdemo3.cloudapp.net.

webappdemo3.cloudapp.net. 60    IN      A       70.37.93.167
`

  

次に肝心のエンドポイントの監視について「[Traffic Manager の監視について](https://azure.microsoft.com/ja-jp/documentation/articles/traffic-manager-monitoring/)」の監視シーケンスを使って要点を整理する。

![AzureTrafficManagerMonitoring](https://farm1.staticflickr.com/750/23276040402_ac8f9f2991_c.jpg)

- 
エンドポイントへのProbe送信(ハートビート)は30ごとに実行

- 
ProbeはHTTP 200 OKかどうかで判定

- 
4回連続して失敗が続いた後、監視システムは使用不可のクラウドサービスを使用不可とみなしそのエンドポイントにトラフィックのルーティングを行わない(④にあたる。⑥は後述)

- 
DNS の有効期限 (TTL) によりDNSリゾルバーで解決された名前がDNS サーバー上でキャッシュされている期間使用できないサービスの DNS 情報を引き続き配信してしまう&rArr;トラフィック減少期間 (⑥にあたる)。DNSのTTLの既定値は、300秒(5分)

 

つまり、エンドポイントがダウンした場合、Probeにより使用不可とみなすのに最大120秒(30秒 x 4)、さらにDNSのTTLの期間（既定値は300秒）を加えると、完全に問題のあったエンドポイントへのトラフィックが停止するまでに標準的に420秒、約7分は見ておく必要があるということになる。当然ながらこの7分間トラフィックがロスする可能性があるため、俊敏なフェールオーバーを期待する場合は別の仕掛けを用意する必要がある。ご注意ください。
