---
layout: post
status: publish
published: true
title: azuresshconfig has been dockerized!
author:
  display_name: Yoichi Kawasaki
  login: yoichi
  email: yokawasa@gmail.com
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 114
wordpress_url: http://unofficialism.info/posts/?p=114
date: '2017-02-05 16:12:24 +0900'
date_gmt: '2017-02-05 07:12:24 +0900'
categories:
- Announcement
- Azure
tags:
- Python
- openssh
- docker
- Container
---

UPDATED 2017-02-15: changed docker run command example due to [Issue#4](https://github.com/yokawasa/azure-ssh-config/issues/4)

![Update Note azuresshconfig-0.2.3](https://c1.staticflickr.com/3/2386/32753419552_9dc00e1771_c.jpg)

（記事はここから）

以前「[azuresshconfigの紹介 &ndash; Azure上でのSSH生活を少しだけ快適にする](http://unofficialism.info/posts/azuresshconfig/)」の投稿で[azuresshconfig](https://github.com/yokawasa/azure-ssh-config)の紹介をさせていただいたが、ツールをリリースして以来、数少ない貴重な利用者様からインストールがコケるんだけど何とかしろというクレームをいただいていた。そこでインストールマニュアルを充実させようかとか、インストーラーをプラットフォーム別に充実させようかとか考えたものの、ここは流行りのコンテナ実行できるようしたほうがいいだろうということで[Docker対応](https://hub.docker.com/r/yoichikawasaki/azuresshconfig/)することにした。

今回の対応によりpipインストールや、プラットフォーム別にprerequisiteなランタイム、ヘッダファイル、ライブラリといった面倒なインストールが不要となり、Mac、Windows、Linux(Ubuntu、CentOS、その他distro)関係なくシンプルにdocker runコマンドでの実行が可能となった。

しかも超軽量Linuxディストリビューションである[Alpine Linux](https://alpinelinux.org/)の上にPythonランタイムとツールを載せているだけであるためサイズはたったの155MBとかなり軽め
`

$ docker images azuresshconfig

REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE

azuresshconfig                     latest              7488bef4343f        7 minutes ago       155 MB
`

## 実行例

`

$ docker run -v $HOME:/root --rm -it yoichikawasaki/azuresshconfig \

    --output stdout --user yoichika --identityfile ~/.ssh/id_rsa > $HOME/.ssh/config
`$HOME/.ssh/config

[Dockerfile](https://github.com/yokawasa/azure-ssh-config/blob/master/Dockerfile)をダウンロードしてビルド・実行はこちら

`

$ curl https://raw.githubusercontent.com/yokawasa/azure-ssh-config/master/Dockerfile -o Dockerfile

$ docker build -t azuresshconfig .

$ docker run -v $HOME:/root --rm -it yoichikawasaki/azuresshconfig \

    --output stdout --user yoichika --identityfile ~/.ssh/id_rsa > $HOME/.ssh/config
`

## LINKS

- [yoichikawasaki/azuresshconfig ＠ Docker Hub](https://hub.docker.com/r/yoichikawasaki/azuresshconfig/)
- [https://github.com/yokawasa/azure-ssh-config](https://github.com/yokawasa/azure-ssh-config)

Enjoy SSH life on Azure with dockerized azuresshconfig!
