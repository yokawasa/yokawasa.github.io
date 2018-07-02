---
layout: post
status: publish
published: true
title: Azure仮想マシンの一括起動・停止 （Classic版）
author:
  display_name: Yoichi Kawasaki
  login: yoichi
  email: yokawasa@gmail.com
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 50
wordpress_url: http://unofficialism.info/posts/?p=50
date: '2015-11-29 10:48:15 +0900'
date_gmt: '2015-11-29 01:48:15 +0900'
categories:
- Azure
tags:
- AzureVM
- PowerShell
- AzureCLI
- ASM
---

Azure仮想マシンV1の一括起動・停止する方法についてのナレッジをここに整理しておく。基本的にはPowerShellや[Azure CLI](https://azure.microsoft.com/ja-jp/documentation/articles/virtual-machines-command-line-tools/)のVM起動・停止コマンドを並列で実行する方法でいけます。ただし同一クラウドサービス内に複数のVMがある場合は単純なコマンドの並列処理実行ではうまくいかない。その場合はAzureで用意されている複数ロールの一括起動・停止用REST APIを使いましょうというお話し。尚、ここでの一括処理はyoichikavm[001-100]の100VMを対象とする。

## VM起動・停止コマンドの並列実行

yoichikavm[001-100]がそれぞれ別のクラウドサービスを持っている場合は、VM起動・停止コマンドを単純に並列実行するという手法で問題なく一括起動・停止ができる。例えばWindowsマシンの場合はPowershellの(Start|Stop)-AzureVMコマンドをStart-Jobを使用してバックグラウンドジョブとして実行させてやる、MacやLinuxであればvm (start|stop)コマンドをforkさせて複数故プロセスを走らせてやるなど。ここではStart-Jobでの並列実行例を紹介する。

複数VMの一括起動: parallel-start.ps1
`

foreach($node in $( Get-AzureVM | Where-Object{$_.Name -match "yoichikavm*"}) ) {

    Write-Host $node.Name

    Start-Job -ScriptBlock{

        param($node)

        Start-AzureVM -ServiceName $node.ServiceName -Name $node.Name

    } -Arg $node

}
`

複数VMの一括停止: parallel-stop.ps1
`

foreach($node in $( Get-AzureVM | Where-Object{$_.Name -match "yoichikavm*"}) ) {

    Write-Host $node.Name

    Start-Job -ScriptBlock{

        param($node)

        Stop-AzureVM -ServiceName $node.ServiceName -Name $node.Name -Force

    } -Arg $node

}
`

## Start/Stop Roles APIの利用(同一クラウドサービス内に複数VMがある場合)

これは実際に私が躓いた際に[@ksasakims](https://twitter.com/ksasakims)さんに教えてもらった方法。冒頭で簡単に説明したとおり同一クラウドサービス内に複数のVMがある場合は上記のような単純なコマンドの並列処理実行ではうまくいかない。この場合、[Start Roles](https://msdn.microsoft.com/ja-jp/library/azure/dn469419.aspx)、[Shutdown Roles](https://msdn.microsoft.com/ja-jp/library/azure/dn469421.aspx)のREST APIを使うことで同一クラウドサービス内の複数VMの同時起動。停止を実現することができる。

以下、APIについて簡単にご説明する。両APIともPOST先は下記同一のエントリーポイントとなっており、またBodyにそれぞれ下記のように起動VM,停止VMを記述する。

POST用エントリーポイント
`

https://management.core.windows.net/サブスクリプションID/services/hostedservices/クラウドサービス/deployments/デプロイ名/roles/Operations
`

BODY例: 一括起動 yoichikavm[001-100]
`

StartRolesOperation

yoichikavm001
yoichikavm002

    ... skip ...
yoichikavm100

`

BODY例) 一括停止 yoichikavm[001-100]
`

ShutdownRolesOperation

yoichikavm001
yoichikavm002

    ... skip ...
yoichikavm100

StoppedDeallocated

`

そして、共通ヘッダとして下記の２つが必要になる。
`

Content-Type: application/xml

x-ms-version: 2015-04-01    << サービス管理バージョン、これは最新の2015-04-01でOK
`

認証については他のサービス管理APIと同様にAzure ADの使用もしくは証明書を利用した認証を行う必要がある。詳細についてはを「[サービス管理要求の認証](https://msdn.microsoft.com/ja-jp/library/azure/ee460782.aspx)」参照いただければと思う。

## LINKS

- [Azure サービス管理での Mac、Linux、および Windows 用 Azure CLI の使用](https://azure.microsoft.com/ja-jp/documentation/articles/virtual-machines-command-line-tools/)
- [Start-Jobでバックグラウンドジョブを実行](https://technet.microsoft.com/ja-jp/library/hh849698.aspx)
- [仮想マシンの操作 - Start Roles](https://msdn.microsoft.com/ja-jp/library/azure/dn469419.aspx)
- [仮想マシンの操作 - Shutdown Roles](https://msdn.microsoft.com/ja-jp/library/azure/dn469421.aspx)
- [サービス管理要求の認証](https://msdn.microsoft.com/ja-jp/library/azure/ee460782.aspx)
