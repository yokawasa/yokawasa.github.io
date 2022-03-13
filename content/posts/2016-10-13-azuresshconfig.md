---
author: Yoichi Kawasaki
categories:
- Azure
date: "2016-10-13T01:22:24Z"
date_gmt: 2016-10-12 16:22:24 +0900
published: true
status: publish
tags:
- Python
- openssh
title: Making SSH lives in Azure easier with azuresshconfig
---

UPDATED 2016-10-31: paramsオプション + Bash Completion追加

みんな大好きSSHとAzureのお話し。物理サーバ、EC2/仮想マシン、コンテナなどなんでもよいがその上にLinuxサーバをたてたらまずやることの1つにSSHログインのためにそのIPアドレス調べて`~/.ssh/config`にそのエントリーを追加してやることがあるんじゃないかと思います。この作業、エントリー数が少なければ大したことはないものの、追加対象のホストが大量にある場合はかなり面倒な作業になってきます。さらにDHCPなどでアドレスを動的に取得するような設定であればサーバの上げ下げのたびにIPアドレスが変わってくるので`~/.ssh/config`の更新が必要になってきて、どうしようもなく面倒になってきます。こういった単純でどうしようもなくつまらない作業は自動化したいですよね？ ここではそんな皆さんのために[azuresshconfig](https://github.com/yokawasa/azure-ssh-config)というツールを紹介させていただきます。

これは皆さんのAzureサブスクリプション下に作られた仮想マシン一覧（ARMに限る）の情報を取得して各仮想マシンごとのエントリー情報（マシン名とIPアドレス）を~/.ssh/configに追加・更新してくれるツール。新規に仮想マシンを追加した際や、仮想マシンのIPアドレスが追加した際には`azuresshconfig`を実行してあげることで`~/.ssh/config`が最新のエントリー情報でアップデートされ、各マシンにマシン名でSSHログインできるようになります。

ちなみに、`~/.ssh/config`とは何ですか？という人はQiitaの記事「[~/.ssh/configについて](http://qiita.com/passol78/items/2ad123e39efeb1a5286b)」がとても分かりやすく書かれているので参考になるかと。

## インストール

Pythonパッケージ管理ツール`pip`を使って`azuresshconfig`をインストールしてください。インストール時に何かエラーが発生した場合は、[こちら](https://github.com/yokawasa/azure-ssh-config/blob/master/Issues.md)のページを参照いただき特に該当する事象がないか確認ください。

```sh
$ pip install azuresshconfig
```

## 設定ファイルの編集（サービスプリンシパル）

```sh
$ vi $HOME/.azure/azuresshconfig.json

{
    "subscription_id": "",
    "client_id": "",
    "client_scret": "",
    "tenant_id": ""
}
```

`サービスプリンシパル`を作る必要があります。`サービスプリンシパル`の作り方が分からない人、とってもよいドキュメントがあります。こちらを参照ください：「[Use Azure CLI to create a service principal to access resources](https://azure.microsoft.com/en-us/documentation/articles/resource-group-authenticate-service-principal-cli/)」

## 使い方
```
azuresshconfig --help

usage: azuresshconfig.py [-h] [--version] [--init] [--profile PROFILE]
                         [--user USER] [--identityfile IDENTITYFILE]
                         [--private] [--resourcegroups RESOURCEGROUPS]
                         [--params PARAMS]

This program generates SSH config from Azure ARM VM inventry in subscription

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  --init                Create template client profile at
                        $HOME/.azure/azuresshconfig.json only if there is no
                        existing one
  --profile PROFILE     Specify azure client profile file to use
                        ($HOME/.azure/azuresshconfig.json by default)
  --user USER           SSH username to use for all hosts
  --identityfile IDENTITYFILE
                        SSH identity file to use for all hosts
  --private             Use private IP addresses (Public IP is used by
                        default)
  --resourcegroups RESOURCEGROUPS
                        A comma-separated list of resource group to be
                        considered for ssh-config generation (all resource
                        groups by default)
  --params PARAMS       Any ssh-config params you want to add with query-
                        string format: key1=value1&key2=value2&...
```

## 実行する

### 1. パラメータ指定なしで実行

```sh
$ azuresshconfig
```

`~/.ssh/config`には下記のように`### AZURE-SSH-CONFIG BEGIN ### ～ ### AZURE-SSH-CONFIG END ###`のブロック内にマシン名とそのIPアドレス（デフォルト：パブリック）のエントリー一覧が追加・更新されます。

```sh
$ cat ~/.ssh/config

### AZURE-SSH-CONFIG BEGIN ###

Host myvm1
    HostName 40.74.124.30

Host myvm2
    HostName 40.74.116.134
....

### AZURE-SSH-CONFIG END ###
```

### 2. SSHユーザと鍵指定

```sh
$ azuresshconfig --user yoichika --identityfile ~/.ssh/id_rsa
```

`~/.ssh/config`には各エントリーにIPアドレスに加えてユーザ名と鍵のパス情報が追加されます。


```sh
$ cat ~/.ssh/config

### AZURE-SSH-CONFIG BEGIN ###

Host myvm1
    HostName 40.74.124.30
    IdentityFile /home/yoichika/.ssh/id_rsa
    User yoichika

Host myvm2
    HostName 40.74.116.134
    IdentityFile /home/yoichika/.ssh/id_rsa
    User yoichika
....

### AZURE-SSH-CONFIG END ###
```

### 3. プライベートIPを指定

```sh
$ azuresshconfig --user yoichika --identityfile ~/.ssh/id_rsa --private
```

`private`オプションを付けて実行することで`~/.ssh/config`の各エントリーにはデフォルトのパブリックIPアドレスではなくてプライベートIPアドレスが追加されます。

### 4. リソースグループで絞る

```sh 
$ azuresshconfig --user yoichika --identityfile ~/.ssh/id_rsa --resourcegroups mygroup1,mygroup2
```

`resourcegroups`オプションを指定することで指定されたリソースグループに所属する仮想マシンのエントリーのみが`~/.ssh/config`に追加されます。

### 5. 追加ssh-configパラメータの指定

```sh
$ azuresshconfig.py --user yoichika \
        --identityfile ~/.ssh/id_rsa \
        --params "Port=2222&Protocol=2&UserKnownHostsFile=~/.ssh/known_hosts&ForwardAgent=yes"
```

`params`オプションを指定することでその他指定可能な`ssh-config`パラメータを追加することができます。上記のように`params`に`ssh-config`の`Port、Protocol、UserKnownHostsFile、ForwardAgent`キーと値をセットすることで次のように出力される`ssh-config`に指定したキーと値がセットされます。


```sh
$ cat ~/.ssh/config

### AZURE-SSH-CONFIG BEGIN ###

Host myvm1
    HostName 40.74.124.30
    IdentityFile ~/.ssh/id_rsa
    User yoichika
    Port 2222
    Protocol 2
    UserKnownHostsFile ~/.ssh/known_hosts
    ForwardAgent yes

Host myvm2
    HostName 40.74.116.134
    IdentityFile /home/yoichika/.ssh/id_rsa
    User yoichika
    Port 2222
    Protocol 2
    UserKnownHostsFile ~/.ssh/known_hosts
    ForwardAgent yes
....

### AZURE-SSH-CONFIG END ###
```

## Shell Completion

### Bashでの補完

次のようにbash/[azuresshconfig_completion.bash](https://github.com/yokawasa/azure-ssh-config/blob/master/bash/azuresshconfig_completion.bash)をbash起動時に読み込ませてあげることで`azuresshconfig`のパラメータ補完ができるようになります.

```sh
# copy this under either of following directories
$ cp azuresshconfig_completion.bash (/etc/bash_completion.d | /usr/local/etc/bash_completion.d | ~/bash_completion.d)`

# or append 'source /path/to/azuresshconfig_completion.bash' to .bashrc like this
$ echo 'source /path/to/azuresshconfig_completion.bash' >> .bashrc
```

次のようにtabでazuresshconfigのパラメータ補完を行います。

```sh
$ azuresshconfig -[tab]
-h                --identityfile    --params          --profile         --user
--help            --init            --private         --resourcegroups

$ azuresshconfig --i[tab]
--identityfile  --init

$ azuresshconfig --p[tab]
--params   --private  --profile

$ azuresshconfig --user [tab]
$ azuresshconfig --user <ssh_user>
$ azuresshconfig --user <ssh_user> --identityfile [tab]
$ azuresshconfig --user <ssh_user> --identityfile <ssh_identity_file>
```

## その他

インストール時のエラーや実行時のエラーについてはこちらに見つけ次第事象とその対応方法を追加しています。
[https://github.com/yokawasa/azure-ssh-config/blob/master/Issues.md](https://github.com/yokawasa/azure-ssh-config/blob/master/Issues.md)

もしバグを見つけたり、追加機能のリクエストがある場合にこちらにIssue追加ください。頑張って時間をみつけて対応します。
[https://github.com/yokawasa/azure-ssh-config/issues](https://github.com/yokawasa/azure-ssh-config/issues)

## LINKS

- [https://github.com/yokawasa/azure-ssh-config](https://github.com/yokawasa/azure-ssh-config)
- [https://pypi.python.org/pypi/azuresshconfig/](https://pypi.python.org/pypi/azuresshconfig/)
- [Use Azure CLI to create a service principal to access resources](https://azure.microsoft.com/en-us/documentation/articles/resource-group-authenticate-service-principal-cli/)

azuresshconfig makes your SSH life on Azure easy!
