---
layout: post
status: publish
published: true
title: Azure Insights REST APIを使ってAzure各リソースのメトリックを抽出する
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 37
wordpress_url: http://unofficialism.info/posts/?p=37
date: '2015-08-15 01:57:57 +0900'
date_gmt: '2015-08-14 16:57:57 +0900'
categories:
- Azure
tags:
- API
- ResourceGroup
- ARM
- metrics
- monitoring
- AzureInsights
- AppServices
---

## ARMとAzure Insights API

Azure上のさまざまなサービスのメトリック情報をAPI経由で取得したい。そういうことであれば[Azure Service Management API](https://msdn.microsoft.com/en-us/library/azure/ee460799.aspx)を使えばいいじゃないかという声が聞こえてきそうなところだが実は既にこのやり方は時代遅れとなっていることをご存じだろうか？
[2014年5月ごろ](http://www.buildinsider.net/web/azureportal/201405)？に登場したAzureの新しい考え方にResource、ResourceGroup、Azure Resource Managerというものがある。簡単な説明すると、Azure上のPaaSインスタンス、仮想マシンなどすべての管理可能な資源をリソース(Resource)とよばれる単位に細分化し、それらをグループ化したものがResourceGroup、そして全てのリソースはAzure Resource Manager（以下ARM）というもので管理可能になっている。そしてこのARMで管理可能な世界のリソース群に紐づくメトリックデータは[Azure Insights API](https://msdn.microsoft.com/en-us/library/azure/dn931943.aspx)で取得可能となっている。本記事ではさまざまなリソースの中でもWeb Appsに絞って、[Azure Insights REST API (Metric)](https://msdn.microsoft.com/en-us/library/azure/dn931930.aspx)を使ってそのメトリックを取得する方法について紹介する。

## ARM Explorerでどのメトリックが取得可能なのか確認する

ARM Explorer ([https://resources.azure.com/](https://resources.azure.com/)) をご存じだろうか？　これはその名の通りAzure上の全てのリソース（ご利用のサブスクリプションに紐づく全てのリソース）のエクスプローラーであり、これを使うことでこのARM管理下の世界のすべてのリソースをエクスプローラービューで閲覧することができる。このARM Explorerで閲覧可能な各リソースの情報の中にmetricdefinitionsというものがあって、これにはそのリソースに対して指定可能なメトリックの種類やその定義情報などが格納されている。リソースのメトリック取得をする際は、まずはARM Explorerで目的のリソースのmetricdefinitionsから指定可能なメトリックの種類を把握してからAPIリクエストを組み立てていただければと思う。ARM Explorerを使って本記事で取得対象としているWeb Apps（ここではサイト名yoichikademoを対象）のmetricdefinitionsを閲覧しているのが以下のスクリーンショットになる。

![ARMExplorer](https://farm6.staticflickr.com/5670/20566807955_dbc0bd1ab7_c.jpg)

## Azure Insights REST APIメトリック取得インターフェース

[Azure Insights API](https://msdn.microsoft.com/en-us/library/azure/dn931943.aspx)には次のような(1)メトリック定義一覧の取得と(2)対象リソースのメトリック情報取得の2つのインターフェースがある。当然ながらメトリックの取得には(2)のインターフェースを使用する。

(1)メトリック定義一覧取得

```
GET https://management.azure.com
/subscriptions/{-id}/resourceGroups/{resource-group-name}/providers/{resource-provider-namespace}/{resource-type}/{resource-name}/metricDefinitions

[Parameters]
api-version={api-version}
$filter={filter}
```

 (2)メトリック情報取得

```
GET https://management.azure.com
/subscriptions/{-id}/resourceGroups/{resource-group-name}/providers/{resource-provider-namespace}/sites/{sitename}/metrics

[Parameters]
api-version={api-version}
$filter={filter}
```

APIの共通部分は下記の通り。Azure Insights APIへの全ての要求はAzure Active Directoryを使用して認証する必要があり、この認証により得られたトークンを各APIリクエストのAuthorizationヘッダに指定する必要がある。トークン取得の方法にはPowerShellを使用した方法とAzure管理ポータルを使用して認証する2つの方法がある。詳しくは「[Azure インサイト要求を認証する](https://msdn.microsoft.com/ja-jp/library/azure/dn931949)」を参照ください。

- `{api-version}`："2014-04-01" 
- `{subscription-id}` ： サブスクリプションID 
- `{resource-group-name}`： リソースグループを指定。詳細は「[リソースグループを使用した Azure リソースの管理](https://azure.microsoft.com/ja-jp/documentation/articles/resource-group-portal/)」を参照ください
- `Acceptヘッダー`：`"application/json"`を指定。これを指定しない場合、結果はXMLで返却される
- `Authorizationヘッダー`にAzure Active Directory から取得する `JSON Web Token（JWT）` に設定する。詳細は「[Azure インサイト要求を認証する](https://msdn.microsoft.com/ja-jp/library/azure/dn931949)」を参照ください

実際のメトリクス取得APIでは$filterパラメータの付与が必要となる。`$filter`には主にメトリックの種類(name.value)、時間レンジ(startTime - endTime)、インターバール(timeGrain)の3種類の条件を指定する。

CpuTimeとMemoryWorkingSetメトリック指定例 (実際は1行にまとめる)

```
$filter=
  (name.value eq ‘CpuTime’ or name.value eq ‘MemoryWorkingSet’) 
  and startTime eq ‘2015-08-01T15:00:00Z’ and endTime eq ‘2015-08-02T15:00:00Z’ 
  and timeGrain eq duration’PT1M’
```

`$filter`パラメータには実際はURL Encodeかけた文字列を渡す。メトリックのインターバル（timeGrain）は1分単位なら`PT1M`、5分単位なら`PT5M`、1時間単位ならば`PT1H`、1日単位ならば`PT1D`などを指定する。

## サンプルコード

Azure Insightsのメトリック取得APIを使ってWeb Appのメトリックを取得するサンプルコードをGithubにアップした。同様のことを考えている人にとって少しでも参考になれば幸いである。

[https://github.com/yokawasa/azure-samples/tree/master/GetAppServiceMetrics](https://github.com/yokawasa/azure-samples/tree/master/GetAppServiceMetrics)

このサンプルではCpuTime、MemoryWorkingSet、AverageResponseTimeの3つのメトリック情報を1時間ごとのインターバルで2015-08-01～2015-08-30の期間分取得するようになっている。この部分はハードコーディングしているので必要に応じてソースコード（Program.cs）を適宜変更いただければと思う。また利用者ごとの情報は設定ファイルApp.configで指定する。以下簡単にApp.configの設定内容を解説する。

**App.config**

```xml
<appSettings>
    <add key="subscriptionId" value="[your subscription Id]" />
    <add key="resourceGroup" value="[your resource group name]" />
    <add key="siteName" value="[site Name]" /> 
    <add key="tenantId" value="[you tenant ID]" /> 
    <add key="applicationId" value="[your application id]" />
    <add key="redirectUri" value='[redirect url]' />    
</appSettings>
```

- siteNameはWeb AppでいうところのApp名、xxxx.azurewebsites.netのxxx部分
- tenant IDとapplication IDの取得方法については「[Azure インサイト要求を認証する](https://msdn.microsoft.com/ja-jp/library/azure/dn931949)」を参照ください
- redirect UriはAAD認証トークン取得時のリダイレクトURI。とりあえずhttp://foobarなど何か指定しておくでOK

## LINKS

- [Azure Service Management REST API Reference](https://msdn.microsoft.com/en-us/library/azure/ee460799.aspx) (old)
- [Azure Service Management monitoring (classic) API](https://msdn.microsoft.com/en-us/library/azure/dn510414.aspx) (old)
- [Azure Resource Management APIs](https://msdn.microsoft.com/en-us/library/azure/dn948464.aspx)
- [Azure Insights REST API Reference](https://msdn.microsoft.com/en-us/library/azure/dn931943.aspx)  ([Metric取得API](https://msdn.microsoft.com/en-us/library/azure/dn931930.aspx))
- [Azure インサイト要求を認証する](https://msdn.microsoft.com/ja-jp/library/azure/dn931949)
- [リソースグループを使用した Azure リソースの管理](https://azure.microsoft.com/ja-jp/documentation/articles/resource-group-portal/)
