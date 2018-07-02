---
layout: post
status: publish
published: true
title: Azureで複数VNETをVPNでデイジーチェーン接続してマルチホップアスセスする設定 (Classic版)
author:
  display_name: Yoichi Kawasaki
  login: yoichi
  email: yokawasa@gmail.com
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 65
wordpress_url: http://unofficialism.info/posts/?p=65
date: '2016-01-01 16:57:08 +0900'
date_gmt: '2016-01-01 07:57:08 +0900'
categories:
- Azure
tags:
- AppServices
- VPN
- ASM
---

デイジーチェーンとは複数の周辺機器を直列につないでいく配線方法のこと。複数VNETをデイジーチェーン接続した構成で、真ん中のVNETを中継してマルチホップアクセスするための設定方法について色々と手こずったのでここに備忘録として残しておく。また似たようなシナリオ(*注)にApp Servicesと複数VNETのデイジーチェーン接続構成でのマルチホップアクセスがあると思うがこちらの設定についても残しておく。同様の設定で困っている人にとって少しでも参考になれば幸いである。

*注 - マルチホップアクセスが必要なシナリオ例

2015年12月時点で[VNet の共存する ExpressRoute 接続とサイト間 VPN 接続の構成における制限](https://azure.microsoft.com/ja-jp/documentation/articles/expressroute-howto-coexist-classic/)としてポイント対サイトVPNとExpressRouteを同じVNETに共存できない仕様となっている。よって、ポイント対サイトVPN接続クライアントからExpressRoute接続したVNETにアクセスしたい場合は、ExpressRoute に接続されているのと同じVNETへのシングルホップアクセスはできないので、間にトランジット用の中間VNETを挟んだいわゆるデイジーチェーン接続構成にしてマルチホップなアクセスにしてやれば実現可能である。

## 1. 複数VNET間のデイジーチェーン接続

サンプルとして下図のように3つのVNETを用意してデイジーチェーン接続設定を行う。ここでは分かりやすく1-1と1-2でそれぞれシングルホップアクセスしかできない例とマルチホップアクセスできる設定例を紹介する。

![VPN-Daisy-Chain-Site2Site](https://farm2.staticflickr.com/1617/24008147956_b110c0c09e.jpg)

[table id=1 column_widths="33%|33%|33%" /]

用意するローカルネットワークは次の5つ。local-vnet1～local-vnet3はそれぞれ、vnet1～vnet3のVPN Gateway経由でそれぞれvnet1～vnet3のアドレス空間にアクセスするためのローカルネットワーク設定。またlocal-vnet2-1はvnet2のVPN Gatewayを経由してvnet2とvnet1のアドレス空間にアクセスするためのローカルネットワーク設定で、local-vnet2-3はvnet2のVPN Gatewayを経由してvnet2とvnet3のアドレス空間にアクセスするためのローカルネットワーク設定である。

ローカルネットワーク一覧

[table id=2 column_widths="33%|33%|33%" /]

ネットワーク設定用XMLのLocalNetworkSites設定内容
`

172.17.0.0/16

40.74.135.162

172.18.0.0/16

40.74.143.191

172.19.0.0/16

40.74.136.22

172.18.0.0/16
172.17.0.0/16

40.74.143.191

172.18.0.0/16
172.19.0.0/16

40.74.143.191

`

### 1-1. 隣同士のVNETへのシングルホップアクセスのための設定例

3つのVNETをvnet１とvnet２、vnet２とvnet３をサイト対サイトVPN接続してデイジーチェーン接続構成にする。２つのサイト対サイトVPN接続設定では共に隣のVNETに接続できるように設定を行う。この場合、当然ながら隣のVNETには接続できるようになるが、VNET1 &rarr; VNET3やVNET3 &rarr; VNET1のように中間VNETを跨いでマルチホップなアクセスはできない。以下その設定イメージ図とネットワーク設定XMLファイルのVirtualNetworkSitesの内容になる。

![VPN-Daisy-Chain-Site2Site-Config-SingleHopAccess](https://farm2.staticflickr.com/1611/23967168882_323593336e_c.jpg)

下記VirtualNetworkSitesでは、vnet1からはvnet2のアドレス空間にアクセスできるようにlocal-vnet2を接続先ローカルネットワークとして設定しており、vnet2からはvnet1とvnet3のアドレス空間にアクセスできるようにlocal-vent1とlocal-vnet3を接続先ローカルネットワークとして設定、そしてvnet3からはvnet2のアドレス空間にアクセスできるようにlocal-vnet2を接続先ローカルネットワークとして設定している。

ネットワーク設定用XMLのVirtualNetworkSites設定内容
`

172.17.0.0/16

172.17.0.0/19

172.17.32.0/29

172.18.0.0/16

172.18.0.0/19

172.18.32.0/29

172.19.0.0/19

172.19.0.0/22

172.19.4.0/29

`

### 1-2. VNET1-VNET3間マルチホップアクセスのための設定

1-1と同様に3つのVNETをvnet１とvnet２、vnet２とvnet３をサイト対サイトVPN接続してデイジーチェーン接続構成にする。ただしここでは２つのサイト対サイトVPN接続設定で隣のVNET以外にvnet1 &rarr; vnet3やvnet3 &rarr; vnet1のようにvnet2をトランジット用中間VNETとしてマルチホップにアクセスできるよう設定する。以下その設定イメージ図とネットワーク設定XMLファイルのVirtualNetworkSitesの内容になる。

![VPN-Daisy-Chain-Site2Site-Config-MultipleHopAccess](https://farm6.staticflickr.com/5714/23992719721_95300a59da_c.jpg)

下記VirtualNetworkSitesでは、vnet1からはvnet2のVPN Gatewayを経由してvnet2とvnet3のアドレス空間にアクセスできるようにlocal-vnet2-3を接続先ローカルネットワークとして設定しており、vnet2からはvnet1とvnet3のアドレス空間にアクセスできるようにローカルネットワークlocal-vent1とlocal-vnet3を接続先ローカルネットワークとして設定、そしてvnet3からはvnet2のVPN Gatewayを経由してvnet2とvnet1のアドレス空間にアクセスできるようにlocal-vnet2-1を接続先ローカルネットワークとして設定している。

ネットワーク設定用XMLのVirtualNetworkSites設定内容
`

172.17.0.0/16

172.17.0.0/19

172.17.32.0/29

172.18.0.0/16

172.18.0.0/19

172.18.32.0/29

172.19.0.0/19

172.19.0.0/22

172.19.4.0/29

`

## 2. App Servicesと複数VNET間のデイジーチェーン接続

サンプルとして下図のように１つのWeb Appと２つのVNETを用意してAppとvnet1はポイント対サイトVPNで、vnet1とvnet2はサイト対サイトVPN接続でデイジーチェーン接続設定を行う。１と同様に2-1と2-2でそれぞれマルチホップできない例とできる設定例を紹介する。

![VPN-Daisy-Chain-Apps2Site2Site](https://farm6.staticflickr.com/5783/23951704041_95fdc7d788.jpg)

VNET基本情報

[table id=3 /]

用意するローカルネットワークは次の3つ。local-vnet1、local-vnet2はそれぞれvnet1、vnet2のVPN Gateway経由でそれぞれvnet1、vnet2のアドレス空間にアクセスするためのローカルネットワーク設定。またlocal-vnet1-pはvnet1のVPN Gatewayを経由してvnet1のアドレス空間とポイント対サイト用VPNクライアント用アドレス空間にアクセスするためのローカルネットワーク設定となっている。

ネットワーク設定用XMLのLocalNetworkSites設定内容

[table id=4 column_widths="33%|33%|33%" /]

### 2-1. App Services-VNET2間でマルチホップアクセスできない設定例

Web Appとvnet1はポイント対サイトVPN接続設定を行う。一方vnet1とvnet2間のサイト対サイトVPN接続設定では互いのVNETに接続できるように設定を行う。この場合、当然ながらApp &rarr; vnet1、vnet1 &rarr; vnet2へのシングルホップのアクセスはできるがApp &rarr; vnet2のマルチホップアクセスはできない。

![VPN-Daisy-Chain-P2S-S2S-Config-SingleHopAccess](https://farm6.staticflickr.com/5687/23447083164_6cbd67e2f8_c.jpg)

ネットワーク設定用XMLのVirtualNetworkSites設定内容
`

172.17.0.0/16

172.17.0.0/19

172.17.32.0/29

192.168.1.0/28

172.18.0.0/16

172.18.0.0/19

172.18.32.0/29

`

### 2-2. App Services-VNET2間マルチホップアクセスのための設定

2-1と同様にWeb Appとvnet1はポイント対サイトVPN接続設定、vnet1とvnet2間はサイト対サイトVPN接続設定を行いデイジーチェーン接続構成にする。ただしここではvnet2 &rarr; vnet1のサイト対サイトVPN接続設定においてvnet1のVPN Gatewayを経由してvnet1のアドレス空間とVPNクライアント用アドレス空間にアクセスできるようにlocal-vnet1-pを接続先ローカルネットワークとして設定している。これでApp &rarr; vnet2のマルチホップアクセスが可能となる。

![VPN-Daisy-Chain-P2S-S2S-Config-MultipleHopAccess](https://farm2.staticflickr.com/1483/23707501489_e57bf35457_c.jpg)

ネットワーク設定用XMLのVirtualNetworkSites設定内容
`

172.17.0.0/16

172.17.0.0/19

172.17.32.0/29

192.168.1.0/28

172.18.0.0/16

172.18.0.0/19

172.18.32.0/29

`

おわり

## LINKS

- [複数のオンプレミスのサイトを仮想ネットワークに接続](https://azure.microsoft.com/ja-jp/documentation/articles/vpn-gateway-multi-site/)

- [PowerShell を使用してサイト間 VPN 接続で仮想ネットワークを作成する](https://azure.microsoft.com/ja-jp/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell/)

- [Using VNET integration and Hybrid connections with Azure Websites](https://azure.microsoft.com/en-us/blog/using-vnet-or-hybrid-conn-with-websites/)
