---
layout: post
status: publish
published: true
title: SOCKSプロキシを経由したAzure VNETプライベートリソースへのアクセス
author:
  display_name: Yoichi Kawasaki
  login: yoichi
  email: yokawasa@gmail.com
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 40
wordpress_url: http://unofficialism.info/posts/?p=40
date: '2015-08-18 11:16:03 +0900'
date_gmt: '2015-08-18 02:16:03 +0900'
categories:
- Azure
tags:
- openssh
- chrome
- firefox
- SOCKS
- AzureVNET
- VPN
---

UPDATED 2017-03-22: Added SOCKS Proxy Configuration for Internet Explorer

外部からの接続（SSH、HTTPなど）を受け付けていないAzure 仮想ネットワーク(以下VNET)内のリソースにSOCKSプロキシを経由して外部からアクセスしましょうというお話。本記事ではAzure VNET内の外部からのアクセス許可していないVMへのSSHログインとHTTPサーバコンテンツへのブラウジングの2つの方法を紹介する。

SOCKS([RFC1928](http://www.ietf.org/rfc/rfc1928.txt)) とはさまざまなアプリケーションが間にファイアーウォールを挟んでいても安全に快適にやり取りができるようにすることを目的として作られたプロトコルのことで、SOCKSプロキシはSOCKSプロトコルを受け取りファイアウォール内外との接続を可能にするものである。エンドポイントやNetwork Security Group (NSG)によりネットーワーク分離設定されたAzure VNET内のリソースに対して一時的に本来直接アクセス許可しないネットワークからアクセスが必要な状況はあるかと思う。そのような時に毎回設定変更で必要なプロトコル、アクセス先に対して穴をあけるのは非常に面倒であり、またサイト間VPN、ポイント対サイトVPNとなるとさらに手間がかかる。お手軽に、もしくは定常的ではないが一時的に内部リソースにアクセスしたい場合にSOCKSプロキシ経由でのアクセスを検討してみてはいかがだろうか。以下はSOCKSプロキシ経由によるAzure VNET内のプライベートリソースへのアクセスイメージである。

![Accessing-AzureVNET-via-SOCKSProxy](https://farm6.staticflickr.com/5759/20658569122_72324d9ed9_c.jpg)

## SOCKSプロキシの作成

まずはOpenSSHのダイナミックポートフォワード機能を使ってSOCKSプロキシを作成する。ダイナミックポートフォワードはSSHをSOCKSプロキシとして振舞うことを可能にする。SSHでアクセス先ホストと DynamicFoward(-D)でポートを指定することでlocalhostにSOCKSプロキシが立ち上がり指定のTCPポート(SOCKSプロキシサーバは基本的は1080だが、割り当て可能なポートであればどのポートでもOK)をlocalhost側からログイン先ホストのSSHサーバに転送することができるようになる。もちろん経路は暗号化される。現状のサポートプロトコルはSOCKS4とSOCKS5。

例えば上図でいうとJump Server(踏み台)にDynamicFoward(-D)1080でログインすると、Jump Serverにポート1080を転送するSOCKSプロキシが localhostに立ち上がり、そのlocalhost:1080に対してSOCKS4またはSOCKS5プロトコルで接続することでJump Serverを経由して通信を行うことができるようになる。

localhost ポート1080のJump Serverへのダイナミックフォワードは次のように-Dオプションで行う。
`

$ ssh -2 -D 1080 -l [Account] [Jump Server]
`

毎回-Dオプション指定が面倒な場合は、次のようにconfg(ssh_config)にDynamicForwardの記述することも可能。

 ~/.ssh/config
`

Host JumpServer

   User         [Account名]

   HostName/IP  [Jump Serverホスト/IPアドレス]

   Protocol 2

   ForwardAgent  yes

   DynamicForward 1080
`

上記OpenSSHの設定は、Linux/Macの場合は標準Terminalを使えばよいが、Windowsの場合はCygwin、XmingなどX端末エミュレータソフトをインストールしていただく必要がある。またX端末エミュレーターをインストールしなくともWindowsでは有名なSSHクライアントソフトPuttyがダイナミックポートフォワードに対応しているためPuttyを使ってSOCKSプロキシ作成することも可能。詳しくは「[Dynamic Port Forwarding with SOCKS over SSH](http://dimitar.me/dynamic-port-forwarding-with-socks-over-ssh/)」が参考になるかと。

## SOCKSプロキシを使ったSSH接続

次に上記で作成したSOCKSプロキシを経由してVNET内のサーバにSSH接続をする。 netcatでSOCKSプロキシを経由してlocalhostから目的のVNET内サーバ（ServerX）間にnetcatトンネルを作成してServerXにはそのnetcatトンネルを通じて接続する。
`

local$ ssh -2 -l [Account] -o 'ProxyCommand nc -x localhost:1080 %h %p' [ServerX]
`

netcat のプロキシ指定は-xオプションで行う。 ここでは事前に作成したSOCKSプロキシ(localhost:1080)を指定。 netcatトンネルの作成コマンドはProxyCommandに記述する。こちらも毎回長いオプション入力を避けるために config（ssh_config）設定すると便利である。

 ~/.ssh/config
`

Host ServerX

   User        [Account]

   Protocol 2

   ForwardAgent  yes

   ProxyCommand nc -x localhost:1080 -w1 %h %p
`

注意点として、netcatにはGNU本家版とそれ以外にいくつか派生があるが-x オプションの利用可能なnetcatである必要がある。オリジナルのnetcatやGNU netcatには-xオプションはない。ここで使用しているnetcatはIPv6に対応しているOpenBSD netcat。

もちろんVNET内のVMへのSSH接続の別解としてProxyCommandでJump ServerからServerXへnetcatトンネルを作成してlocalからServerXにログインすることも可能。
`

local$ ssh -2 -o 'ProxyCommand ssh [Jump Server] nc -w1 %h %p' [ServerX]
`

実はこの方法のほうがProxyCommand でプロキシ設定を行うため事前に別コンソールで何かを用意する必要がなく間違いなくSOCKSプロキシ経由での接続よりも楽。ではSOCKSプロキシ経由で接続する理由は何か？ 理由は単純で、通常Jump Serverはセキュリティ設定上の理由でログインしてもほとんど何もできないようにするために機能を無効化していることが多く、よってnetcatが使えない(ncはおろかsshコマンド以外ほとんど何も使えない)環境だったりする。この場合、上記で説明したようにSOCKSプロキシ経由でのSSHログインが有効な手段となる。

## SOCKSプロキシ経由でのブラウジング

ブラウザーにSOCKSプロキシ対応のプラグインを	入れることでブラウザーによるプライベートリソース（内部ネットワーク内のアプリ・ミドルウェア等のウェブUIを持った管理ツールなど）の閲覧も可能となる。有名なものに[SwitchySharp](http://www.akiyan.com/blog/archives/2012/10/ultimate-chrome-proxy-change-is-switchy-sharp-for-windows-and-osx.html)(Google Chrome)や[Foxyproxy](https://getfoxyproxy.org/sshproxy.html)(Firefox)などがある。以下簡単な設定スクリーンショットを乗せておく。共に「http://yoichika-*.cloudapp.net*」なURIパターンの時にのみSOCKSプロキシlocalhost:12345経由でアクセスする設定となっている。

Google Chrome - SwitchySharp

![GoogleChrome-SwitcySharp](https://farm1.staticflickr.com/718/20049113383_587178705c_c.jpg)

Firefox - Foxyproxy

![Firefox-FoxyProxyStandard](https://farm6.staticflickr.com/5781/20482135210_b770135d92_c.jpg)

Internet Explorer - Out-of-box feature

As answered in [Stack Overflow](http://stackoverflow.com/questions/18375234/enable-socks-4a-5-in-internet-explorer), Internet Explorer does support SOCKS proxies.

Tools > Internet Options > Connections > LAN Settings > Proxy Server > Advanced

![IE-SOCKS-PROXY](https://c1.staticflickr.com/3/2824/32766799853_46c7abab12_n.jpg)

Enjoy Accessing private resources with SOCKS Proxy!
