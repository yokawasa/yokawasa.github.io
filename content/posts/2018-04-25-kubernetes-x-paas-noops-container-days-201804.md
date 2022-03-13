---
author: Yoichi Kawasaki
categories:
- Azure
- Kubernetes
date: "2018-04-25T13:40:40Z"
date_gmt: 2018-04-25 04:40:40 +0900
published: true
status: publish
tags:
- Container
- Kubernetes
- Azure Container Instances
- AKS
- Istio
- Open Service Broker
- Helm
title: Kubernetes x PaaS コンテナアプリケーションのNoOpsへの挑戦 (Japan Container Days v18.04)
---

先日、4月19日に開催された[Japan Container Days v18.04](https://containerdays.jp/)にて「Kubernetes x PaaS &ndash; コンテナアプリケーションのNoOpsへの挑戦」というタイトルでセッションを担当させていただいた。その名の通りメインがKubernetesで、KubernetesアプリケーションにおいてNoOps（運用レス）を目指すためのにどういった工夫ができるのか、どういったものを活用していけばよいのか、という内容です。このブログではJapan Container Daysでの発表に使用したスライドの共有とセッションに中のサンプルやデモについて補足させていただく。

# Session Slides
[![](https://image.slidesharecdn.com/jkd201804-kubernetespassnoopsmicrosoft-kawasaki-180419045509/95/kubernetes-x-paas-noops-1-1024.jpg?cb=1527122000)](//www.slideshare.net/yokawasa/kubernetes-x-paas-noops)
**[Kubernetes x PaaS &ndash; コンテナアプリケーションの NoOpsへの挑戦](//www.slideshare.net/yokawasa/kubernetes-x-paas-noops)** from **[Yoichi Kawasaki](https://www.slideshare.net/yokawasa)**

# 補足情報

## 1. Open Service Broker for AzureでAzure Database for MySQLの利用

スライドでお見せした実際のファイルを使ってAzure Database for MySQLのサービスインスタンス作成、バインディング、そして実際のアプリケーションからの利用までの流れを紹介させていただく。

Open Service Broker for AzureプロジェクトのGithubにあるサンプルファイル[mysql-instance.yaml](https://github.com/Azure/open-service-broker-azure/blob/master/contrib/k8s/examples/mysql/mysql-instance.yaml)と[mysql-binding.yaml](https://github.com/Azure/open-service-broker-azure/blob/master/contrib/k8s/examples/mysql/mysql-binding.yaml)を使ってそれぞれServiceInstanceとServiceBindingを作成する
`
```shell
# Provisioning the database, basic50 plan ...
$ kubectl create -f mysql-instance.yaml

# Wait until ServiceInstance named example-mysql-instance get ready 'Status => Ready',
# then execute the following to create a binding for this new database,
$ kubectl create -f mysql-binding.yaml
```

これでexample-mysql-secretという名前のシークレットオブジェクトが作成され、そこにアプリケーションがAzure Database for MySQLインスタンスとの接続必要な情報が格納される。ここではMySQLに接続して単純な処理をするだけのPythonアプリでテストする。アプリをデプロイメントのために下記YAML (`mysql-deploy.yaml`)を用意したが注目すべきはYAMLの中でMySQLの接続必要な情報をシークレットexample-mysql-secretから取得している点。

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: sample-osba-mysql
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: sample-osba-mysql
    spec:
      containers:
      - name: osba-mysql-demo
        image: yoichikawasaki/sample-osba-mysql:0.0.1
        ports:
        - containerPort: 80
        env:
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              key: username
              name: example-mysql-secret
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: example-mysql-secret
        - name: MYSQL_HOST
          valueFrom:
            secretKeyRef:
              key: host
              name: example-mysql-secret
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              key: database
              name: example-mysql-secret
```
また、アプリケーションの中ではYAMLで指定された環境変数からMySQL接続に必要な情報を取得している。以下、サンプルアプリのコード(`sample-osba-mysql.py`)。
```python
import MySQLdb as db
import os

# MySQL configurations
MYSQL_USER = os.environ['MYSQL_USER']
MYSQL_PASSWORD = os.environ['MYSQL_PASSWORD']
MYSQL_HOST = os.environ['MYSQL_HOST']
MYSQL_DATABASE = os.environ['MYSQL_DATABASE']

print(MYSQL_USER)
print(MYSQL_PASSWORD)
print(MYSQL_HOST)
print(MYSQL_DATABASE)

def main():
    conn = db.connect(
            user=MYSQL_USER,
            passwd=MYSQL_PASSWORD,
            host=MYSQL_HOST,
            db=MYSQL_DATABASE
        )
    c = conn.cursor()

    sql = 'drop table if exists test'
    c.execute(sql)

    sql = 'create table test (id int, content varchar(32))'
    c.execute(sql)

    sql = 'show tables'
    c.execute(sql)
    print('===== table list =====')
    print(c.fetchone())

    # insert records
    sql = 'insert into test values (%s, %s)'
    c.execute(sql, (1, 'hoge'))

    datas = [
        (2, 'foo'),
        (3, 'bar')
    ]
    c.executemany(sql, datas)

    # select records
    sql = 'select * from test'
    c.execute(sql)
    print('===== Records =====')
    for row in c.fetchall():
        print('Id:', row[0], 'Content:', row[1])

    conn.commit()
    c.close()
    conn.close()


if __name__ == '__main__':
    main()
```


それでは次のようにアプリケーションをデプロイする
```shell
$ kubectl create -f mysql-deploy.yaml
```

全ての過程が問題なければ、デプロイ後のPodのログをみると、下記のような出力が確認できるはずだ。これはアプリケーションがMySQLとの接続して処理が行われたことを表す。ここでは[stern](https://github.com/wercker/stern)を使ってログを参照している。
```shell
$ stern <アプリケーションのPod名>

+ sample-osba-mysql-684ccd679f-nl252 › osba-mysql-demo
sample-osba-mysql-684ccd679f-nl252 osba-mysql-demo o36e6ivtni@8ae3fca5-9b5c-471b-99d8-0eeb567c6acb
sample-osba-mysql-684ccd679f-nl252 osba-mysql-demo sE8UFm5cN4tgIeNF
sample-osba-mysql-684ccd679f-nl252 osba-mysql-demo 8ae3fca5-9b5c-471b-99d8-0eeb567c6acb.mysql.database.azure.com
sample-osba-mysql-684ccd679f-nl252 osba-mysql-demo v8i0xxlerz
sample-osba-mysql-684ccd679f-nl252 osba-mysql-demo ===== table list =====
sample-osba-mysql-684ccd679f-nl252 osba-mysql-demo ('test',)
sample-osba-mysql-684ccd679f-nl252 osba-mysql-demo ===== Records =====
sample-osba-mysql-684ccd679f-nl252 osba-mysql-demo Id: 1 Content: hoge
sample-osba-mysql-684ccd679f-nl252 osba-mysql-demo Id: 2 Content: foo
sample-osba-mysql-684ccd679f-nl252 osba-mysql-demo Id: 3 Content: bar
```

## 2. デモ - Virtual Kubelet + Azure Container Instances

- Demo Video: [Image Processing with Virtual Kubelet + ACI Demo](https://youtu.be/2GdXGvIOo40)
- Demo Code: [https://github.com/rbitia/aci-demos](https://github.com/rbitia/aci-demos)

## 3. デモ - Traffic Routing with Istio

- Demo Video: [Traffic Routing with Istio Demo](https://youtu.be/P_FFLvaMIX8)

セッション中に時間が足りなくてチラ見せ程度しかお見せできなかったサービスメッシュにIstio（Istio-0.5.0）を使ったトラフィックルーティングデモ。せっかくなのでここで全ての手順を紹介させていただく。アプリな単純なコンテナイメージのタグ名を表示するだけの[Flaskアプリ](https://github.com/yokawasa/docker-image-samples/tree/master/python-version-app)を利用している

必要ファイル一覧

- [myversion-v1.yaml](https://gist.github.com/yokawasa/bd675fec7f6e3246cdef278605203575)
- [myversion-v2.yaml](https://gist.github.com/yokawasa/704d7263995c55b43252083aa2479a5c)
- [route-default-v1.yaml](https://gist.github.com/yokawasa/8b1dfd352b6971fd3f194c7800886d47)
- [route-canary.yaml](https://gist.github.com/yokawasa/5fe5f93b88a87b7113a4ca81aa9b8e1d)
- [route-conditional-v2.yaml](https://gist.github.com/yokawasa/f7263456b20c8576ceff7c53759d0e8d)

まずはV1とV2アプリケーションのデプロイメント。ここではアプリケーションのManifestに対してistioctlのkube-injectでIstioの設定が組み込まれたManifestを元にデプロイメントしている。

```shell
## ver1.0
$ kubectl apply -f <(istioctl kube-inject --debug -f myversion-v1.yaml)
## ver2.0
$ kubectl apply -f <(istioctl kube-inject --debug -f myversion-v2.yaml)
```

上記のデプロイメントで作られたIngressコントローラーのIP、PORTを取得を次のように取得する。サンプルで使用しているFlaskアプリで指定しているエンドポイントが/versionであるからアプリケーションのエンドポイントはGATEWAY_IP:GATEWAY_PORT/versionがとなる。


```shell
GATEWAY_IP=$(kubectl get po -n istio-system -l \
        istio=ingress -n istio-system \
        -o 'jsonpath={.items[0].status.hostIP}')

GATEWAY_PORT=$(kubectl get svc istio-ingress \
        -n istio-system -n istio-system \
        -o 'jsonpath={.spec.ports[0].nodePort}')

GATEWAY_URL=$GATEWAY_IP:$GATEWAY_PORT
echo $GATEWAY_URL
```

まずは、Istioに全てのトラフィックをv1アプリにルーティングするように設定する

```shell
$ istioctl create -f route-default-v1.yaml
```

次のように連続でアクセスして全てのトラフィックがV1にルーティングされていることを確認

```shell
$ while true; do curl http://${GATEWAY_URL}/version; sleep 1; done

I am v1.111111111
I am v1.111111111
I am v1.111111111
I am v1.111111111
```

続いて、Istioにさきほどの100% V1の設定を削除して、新しく90%のトラフィックをv1アプリに 10%をV2アプリにルーティングするように設定する

```shell
$ istioctl delete routerule myversion-default-v1 -n default
$ istioctl create -f route-canary.yaml
```

前と同様に連続でアクセスして90%のトラフィックはV1に、10％はV2にルーティングされていることを確認

```shell
$ while true; do curl http://${GATEWAY_URL}/version; sleep 1; done

I am v1.111111111
I am v1.111111111
I am v1.111111111
I am v2.222222222  ... たまにV2
I am v1.111111111
```

最後に、Istioに特定の条件でのみ100%のトラフィックをV2アプリ向けるように設定する。下記設定ファイル`route-conditional-v2.yaml`に記述されているように、ここではHTTPヘッダのuserの値が"yoichi"であることを特定の条件とする。

```yaml
---
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: myversion-conditional-v2
spec:
  destination:
    name: myversion
  precedence: 2
  match:
    request:
      headers:
        user:
          regex: "yoichi"
  route:
  - labels:
      version: v2.0
```

Istioに特定条件用のルーティング設定を追加

```shell
$ istioctl create -f route-conditional-v2.yaml
```

次のようにヘッダーに"user: yoichi"の条件を追加して連続でアクセスして全てのトラフィックがV2にルーティングされていることを確認

```shell
$ while true; do curl -H "user: yoichi" http://${GATEWAY_URL}/version; sleep 1; done

I am v2.222222222
I am v2.222222222
I am v2.222222222
I am v2.222222222
```

以上、補足情報でした。

### Links

- [Japan Container Days v18.04](https://containerdays.jp/)
- [Open Service Broker API Server for Azure](https://github.com/Azure/open-service-broker-azure)
- [Azure Container Instances](https://azure.microsoft.com/en-us/services/container-instances/)
- [Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet)
- [Istio-0.5.0 Document](https://archive.istio.io/v0.5/docs/setup/)
