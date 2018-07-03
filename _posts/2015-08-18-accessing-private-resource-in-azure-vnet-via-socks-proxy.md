---
layout: post
status: publish
published: true
title: SOCKSプロキシを経由したAzure VNETプライベートリソースへのアクセス
author:
  display_name: Yoichi Kawasaki
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

外部からの接続（SSH、HTTPなど）を受け付けていないAzure 仮想ネットワーク(以下VNET)内のリソースに`SOCKSプロキシ`を経由して外部からアクセスしましょうというお話。本記事ではAzure VNET内の外部からのアクセス許可していないVMへのSSHログインとHTTPサーバコンテンツへのブラウジングの2つの方法を紹介する。

SOCKS([RFC1928](http://www.ietf.org/rfc/rfc1928.txt)) とはさまざまなアプリケーションが間にファイアーウォールを挟んでいても安全に快適にやり取りができるようにすることを目的として作られたプロトコルのことで、`SOCKSプロキシ`は`SOCKSプロトコル`を受け取りファイアウォール内外との接続を可能にするものである。エンドポイントや`Network Security Group (NSG)`によりネットーワーク分離設定されたAzure VNET内のリソースに対して一時的に本来直接アクセス許可しないネットワークからアクセスが必要な状況はあるかと思う。そのような時に毎回設定変更で必要なプロトコル、アクセス先に対して穴をあけるのは非常に面倒であり、またサイト間VPN、ポイント対サイトVPNとなるとさらに手間がかかる。お手軽に、もしくは定常的ではないが一時的に内部リソースにアクセスしたい場合にSOCKSプロキシ経由でのアクセスを検討してみてはいかがだろうか。以下は`SOCKSプロキシ`経由によるAzure VNET内のプライベートリソースへのアクセスイメージである。

![Accessing-AzureVNET-via-SOCKSProxy](https://farm6.staticflickr.com/5759/20658569122_72324d9ed9_c.jpg)

## SOCKSプロキシの作成

まずは`OpenSSH`の`ダイナミックポートフォワード機能`を使って`SOCKSプロキシ`を作成する。`ダイナミックポートフォワード`はSSHを`SOCKSプロキシ`として振舞うことを可能にする。SSHでアクセス先ホストと `DynamicFoward(-D)`でポートを指定することで`localhost`にSOCKSプロキシが立ち上がり指定のTCPポート(SOCKSプロキシサーバは基本的は`1080`だが、割り当て可能なポートであればどのポートでもOK)を`localhost`側からログイン先ホストのSSHサーバに転送することができるようになる。もちろん経路は暗号化される。現状のサポートプロトコルは`SOCKS4`と`SOCKS5`。

例えば上図でいうとJump Server(踏み台)に`DynamicFoward(-D)1080`でログインすると、Jump Serverにポート`1080`を転送するSOCKSプロキシが `localhost`に立ち上がり、その`localhost:1080`に対して`SOCKS4`または`SOCKS5`プロトコルで接続することでJump Serverを経由して通信を行うことができるようになる。

localhostポート`1080`のJump Serverへのダイナミックフォワードは次のように-Dオプションで行う。

```sh
$ ssh -2 -D 1080 -l [Account] [Jump Server]
```

毎回`-Dオプション`指定が面倒な場合は、次のように`confg(ssh_config)`に`DynamicForward`の記述することも可能。

**~/.ssh/config**
```yaml
Host JumpServer
   User         [Account名]
   HostName/IP  [Jump Serverホスト/IPアドレス]
   Protocol 2
   ForwardAgent  yes
   DynamicForward 1080
```

上記OpenSSHの設定は、Linux/Macの場合は標準Terminalを使えばよいが、Windowsの場合は`Cygwin`、`Xming`などX端末エミュレータソフトをインストールしていただく必要がある。またX端末エミュレーターをインストールしなくともWindowsでは有名なSSHクライアントソフト`Putty`が`ダイナミックポートフォワード`に対応しているため`Putty`を使って`SOCKSプロキシ`作成することも可能。詳しくは「[Dynamic Port Forwarding with SOCKS over SSH](http://dimitar.me/dynamic-port-forwarding-with-socks-over-ssh/)」が参考になるかと。

## SOCKSプロキシを使ったSSH接続

次に上記で作成したSOCKSプロキシを経由してVNET内のサーバにSSH接続をする。 `netcat`で`SOCKSプロキシ`を経由して`localhost`から目的のVNET内サーバ（ServerX）間にnetcatトンネルを作成してServerXにはそのnetcatトンネルを通じて接続する。

```sh
local$ ssh -2 -l [Account] -o 'ProxyCommand nc -x localhost:1080 %h %p' [ServerX]
```

`netcat` のプロキシ指定は`-xオプション`で行う。 ここでは事前に作成した`SOCKSプロキシ`(`localhost:1080`)を指定。 netcatトンネルの作成コマンドは`ProxyCommand`に記述する。こちらも毎回長いオプション入力を避けるために `config（ssh_config）`設定すると便利である。


** ~/.ssh/config**
```yaml
Host ServerX
   User        [Account]
   Protocol 2
   ForwardAgent  yes
   ProxyCommand nc -x localhost:1080 -w1 %h %p
```

注意点として、netcatにはGNU本家版とそれ以外にいくつか派生があるが-x オプションの利用可能なnetcatである必要がある。オリジナルのnetcatやGNU netcatには-xオプションはない。ここで使用しているnetcatはIPv6に対応しているOpenBSD netcat。

もちろんVNET内のVMへのSSH接続の別解としてProxyCommandでJump ServerからServerXへnetcatトンネルを作成してlocalからServerXにログインすることも可能。
```sh
local$ ssh -2 -o 'ProxyCommand ssh [Jump Server] nc -w1 %h %p' [ServerX]
```

実はこの方法のほうがProxyCommand でプロキシ設定を行うため事前に別コンソールで何かを用意する必要がなく間違いなくSOCKSプロキシ経由での接続よりも楽。ではSOCKSプロキシ経由で接続する理由は何か？ 理由は単純で、通常Jump Serverはセキュリティ設定上の理由でログインしてもほとんど何もできないようにするために機能を無効化していることが多く、よってnetcatが使えない(ncはおろかsshコマンド以外ほとんど何も使えない)環境だったりする。この場合、上記で説明したようにSOCKSプロキシ経由でのSSHログインが有効な手段となる。

## SOCKSプロキシ経由でのブラウジング

ブラウザーにSOCKSプロキシ対応のプラグインを	入れることでブラウザーによるプライベートリソース（内部ネットワーク内のアプリ・ミドルウェア等のウェブUIを持った管理ツールなど）の閲覧も可能となる。有名なものに[SwitchySharp](http://www.akiyan.com/blog/archives/2012/10/ultimate-chrome-proxy-change-is-switchy-sharp-for-windows-and-osx.html)(Google Chrome)や[Foxyproxy](https://getfoxyproxy.org/sshproxy.html)(Firefox)などがある。以下簡単な設定スクリーンショットを乗せておく。共に「http://yoichika-*.cloudapp.net*」なURIパターンの時にのみSOCKSプロキシlocalhost:12345経由でアクセスする設定となっている。

Google Chrome - SwitchySharp

![GoogleChrome-SwitcySharp](https://farm1.staticflickr.com/718/20049113383_587178705c_c.jpg)

Firefox - Foxyproxy

![Firefox-FoxyProxyStandard](https://farm6.staticflickr.com/5781/20482135210_b770135d92_c.jpg)

Internet Explorer - Out-of-box feature

As answered in [Stack Overflow](http://stackoverflow.com/questions/18375234/enable-socks-4a-5-in-internet-explorer), Internet Explorer does support SOCKS proxies.
```
Tools > Internet Options > Connections > LAN Settings > Proxy Server > Advanced
```
![IE-SOCKS-PROXY](https://c1.staticflickr.com/3/2824/32766799853_46c7abab12_n.jpg)

Enjoy Accessing private resources with SOCKS Proxy!
