---
author: Yoichi Kawasaki
categories:
- Azure
date: "2018-01-01T11:14:17Z"
date_gmt: 2018-01-01 02:14:17 +0900
published: true
status: publish
tags:
- Elasticsearch
- logstash
- beats
- AzureDiagnostics
- ElasticStack
title: 15分でお届けするElastic Stack on Azure設計・構築ノウハウ
---

**UPDATED on Feb 3, 2018 - [Elastic社イベントサイト](https://www.elastic.co/elasticon/tour/2017/tokyo)を追加**

イベント開催日から少々時間が経過したが、[Elastic {ON} Tour 2017 東京](https://www.elastic.co/elasticon/tour/2017/tokyo)（2017年12月14日開催）というElastic社オフィシャルのユーザーカンファレンスにて登壇させていただく機会があり、そこで「15分でお届けする Elastic Stack on Azure 設計・構築ノウハウ」というお題でお話をさせていただいた。個人的にとても大好きなプロダクトなので、そのユーザーカンファレンスでお話をさせていただいたということと、そのプロダクトのAzureでの利用促進に微力ながらも貢献できたということは光栄至極である。ここではそのElastic {ON} Tourでの発表で使用したスライドに補足解説を加えて共有させていただく。

## セッションスライド（＋デモ動画）
[![](https://image.slidesharecdn.com/elasticontourmicrosoftv4-171226103730/95/15-elastic-stack-on-azure-1-638.jpg?cb=1514285111)](//www.slideshare.net/yokawasa/15-elastic-stack-on-azure-84976576)

**[15分でお届けする Elastic Stack on Azure 設計・構築ノウハウ](//www.slideshare.net/yokawasa/15-elastic-stack-on-azure-84976576)** from **[Yoichi Kawasaki](https://www.slideshare.net/yokawasa)**

## 補足解説

### デプロイメント

AzureでのElastic Stackの利用は当然ながら仮想マシン（VM）を並べてそこにクラスタを構築することになる。残念ながら現時点でマネージドのElasticサービスはAzureには存在しない。VMベースということで特にオンプレと変わらずマニュアルであったり、ChefやAnsibleなどの構成管理ツールを使ってクラスタを組んだり柔軟な構築が可能であるものの、ここではAzureでの構築ということでARMテンプレートを使ったデプロイメントの方法を紹介している。

- [Azure Marketplaceからのデプロイ](https://azuremarketplace.microsoft.com/ja-jp/marketplace/apps/elastic.elasticsearch)：最も手っ取り早い方法。30日のX-Packトライアルライセンスが付いていて、トライアル期間が過ぎてもBYOLでライセンスの更新が可能。テンプレートでは2017年12月時点でv2.0.2 〜 v5.6.3の選択が可能。何も考えず最新版をご利用ください。
- [Github上のARMテンプレート](https://github.com/elastic/azure-marketplace)をカスタマイズしてデプロイ：Elastic社が用意したGithub上のARMテンプレートがあるのでそれを自分の要件に応じてカスタマイズしてデプロイメントをする。Azure CLIやPowerShellなどコマンドを使ったデプロイメントが可能なので構成管理ツールに組み込んで周辺環境を合わせて自動構築設定も可能。慣れてきたらこちらがよいでしょう。

### 推奨仮想ハードウェアとDISK

Elasticクラスタ全体のパフォーマンスを引き出すためには機能別に適正なVMインスタンスとサイズを選択ください。またVMにアタッチするディスクについてはビルトインで可用性設定がされているManaged Disk、もしくはPremium Managed Diskを選択することをお忘れなく。

![elastic-stack-on-azure-vm-size](https://farm5.staticflickr.com/4640/39402351461_e8cb26862d_z.jpg)

### 可用性の設定について

AzureでIaaSで可用性の設定といえばおなじみの可用性セット（Availability Set）と可用性ゾーン（Availability Zone）。当然Elastic Stackのクラスタを組む時もこれらの設定を入れましょうというお話。可用性ゾーンは、その可用性レベルの高さから将来的には可用性ゾーンが主流な設定になっていくはずであるものの、2017年12月時点でPreviewリリースであり、利用可能リージョンが米国東部第２、西ヨーロッパのみというとても限定的なものとなっている。現時点でプロダクション用途となると可用性セット一択なので何も考えずに可用性セットを組んでください。

可用性セット（Availability Set）

- 一つのDCの中で同一の物理ラックや電源などを配置しないようにして、障害が発生してもグループの中のどこかのVMは生きているようにする設定のこと
- VM SLA 99.95%で提供

可用性ゾーン（Availability Zone）

- 各VMを別々のゾーンに配置するのでDCレベルの障害につよい（それぞれゾーンは電源、ネットワーク、冷却装置が完全に物理的に分離されたものとなっている）。ちなみに、Azureのリージョンは複数のデータセンターで構成されており、その間を高速なバックボーンで接続して1つのリージョンとして透過的に利用が可能となっているのでこのようなことができるわけだ
- VM SLA 99.99%で提供
- 【注意点】可用性ゾーンの設定ではDCが分かれて配置されるので次の２点の考慮が必要：（１）マスターは各ゾーンに分散するように各ゾーン最低１ノード配置すること（２）データノードはゾーンにまたがる通信が極力起こらないように工夫すること。これを実現するのが[Shard Allocation Awareness](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/allocation-awareness.html)という仕組みで、この仕組みをつかうことで 同一ゾーン内に配置されているノードだけで完全なシャードを保持するようにして、検索要求が同一ゾーン内で完結できるように設定が可能となる

### ネットワークセキュリティグループの設定

AzureのIaaSにおけるネットワークフィルタリングの設定に、ネットワークセキュリティグループ（NSG）とよばれるL4フィルタリングがある。当然ながら、既にX-Packを導入していればそのセキュリティ機能の１つとしてネットワークレベルのアクセス制御についても行うことができるが、X-Packを導入していない場合は確実にNSGの設定は必要になってくる。また、Elastic Stack以外のアプリケーションとの連携の際にも必ず必要になってくる。Azure上でのシステム構築では欠かすことのできない設定の１つ。

### Azureサービスからのデータコレクション

![elastic-stack-on-azure-data-injestion](https://farm5.staticflickr.com/4690/25532299548_a2c6e3be73_z.jpg)

Azure VMについては、オンプレ同様に、ビルトインのBeatsやlogstashとの連携により、そのログやMetricsなどのデータコレクションを実現することができる。一方、Azureが特に力を入れているPaaS（Platform as a Services）からのデータコレクションについてはどうかというと、下記のサービスについては既にビルトインで用意されている機能や、コミュニティ製Logstash Input プラグインを利用することでデータコレクションを実現することができる。
H2M_LI_HEADER Azure Blob Storage: [logstash-input-azureblob](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob)
H2M_LI_HEADER Azure Service Bus (Topic): [logstash-input-azuretopic](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azuretopic)
H2M_LI_HEADER Azure Event Hub: [logstash-input-azureeventhub](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureeventhub)
H2M_LI_HEADER Azure SQL Database: [logstash-input-jdbc](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html)
H2M_LI_HEADER Azure Database for MySQL: [logstash-input-jdbc](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html)
H2M_LI_HEADER Azure Database for PostgreSQL: [logstash-input-jdbc](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html)
H2M_LI_HEADER Azure HDInsight: [ES-Hadoop](https://www.elastic.co/products/hadoop)による連携

ちなみに、Azureサービス向けLogstashプラグイン一覧についてはこちら - [Logstash plugins for Microsoft Azure Services](http://unofficialism.info/posts/logstash-plugins-for-azure-services/)

### Azure Diagnostics、Activities、Metricsログのコレクション

![elastic-stack-on-azure-metrics-diagnostics-logs](https://farm5.staticflickr.com/4680/39402351921_fd44d2560b_z.jpg)

基本的にIaaS、PaaS問わずAzureのほとんどのサービスから下記の情報が出力され、これらの情報はBlobストレージまたはEvent Hubに出力設定が可能となっている。
H2M_LI_HEADER [診断ログ（Diagnostics log）](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)
H2M_LI_HEADER [アクティビティログ（Activity log）](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)
H2M_LI_HEADER [メトリック](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics)

また上記に加えて、PaaSサービスに展開するアプリケーション固有のログについても少なくともBlobストレージに書き込むことはできる。つまるところ、上記データコレクションの項目でご紹介したようにBlobストレージやEvent hubに出力されたデータはLogstashを通じてElastic Stackに取り込むことが可能であり、Azureリソース関連情報を含めた全てのログはElastic Stackで一元管理することができるのである。

### 参考Links

- [Deploying Elasticsearch on Microsoft Azure](https://www.elastic.co/blog/deploying-elasticsearch-on-microsoft-azure)
- [Microsoft AzureにElasticsearchをデプロイする](https://www.elastic.co/jp/blog/deploying-elasticsearch-on-microsoft-azure)
- [Elasticsearch and Kibana Deployments on Azure](https://www.elastic.co/blog/elasticsearch-and-kibana-deployments-on-azure)
- [Spinning up a cluster with Elastic's Azure Marketplace template](https://www.elastic.co/blog/spinning-up-a-cluster-with-elastics-azure-marketplace-template)
- [Run Elasticsearch on Azure](https://docs.microsoft.com/en-us/azure/architecture/elasticsearch/)
- [Public preview: Azure Availability Zones](https://azure.microsoft.com/en-us/updates/azure-availability-zones/)
- [Filter network traffic with network security groups](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-nsg)
- [Shared Allocation Awareness](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/allocation-awareness.html#allocation-awareness)
- [Azure Diagnostics Tools](https://github.com/Azure/azure-diagnostics-tools)

### Elastic{ON} Tour Tokyo 2017イベントLinks

- [15分でお届けするElastic Stack on Azure設計・構築ノウハウ (Japanese) ](https://www.elastic.co/elasticon/tour/2017/tokyo/microsoft)
- [Elastic{ON} Tour Tokyo 2017全セッション](https://www.elastic.co/elasticon/tour/2017/tokyo)
