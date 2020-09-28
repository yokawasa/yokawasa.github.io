---
layout: post
status: publish
published: true
title: GitHub Actions - Kubernetes tools installer
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
date: '2020-07-05 17:00:00 +0900'
tags:
- GitHubActions
- Kubernetes
- kubectl
- kustomize
- helm
- kubeval
- conftest
- yq
---

I've published a new GitHub Action called [action-setup-kube-tools](https://github.com/yokawasa/action-setup-kube-tools) ([View on Marketplace](https://github.com/marketplace/actions/kubernetes-toolset-installer)). The action installs Kubernetes tools (kubectl, kustomize, helm, kubeval, conftest, and yq) and cache them on the runner. This is a typescript version of [stefanprodan/kube-tools](https://github.com/stefanprodan/kube-tools) with no command input param.

## Usage

### Inputs

|Parameter|Required|Default Value|Description|
|:--:|:--:|:--:|:--|
|`kubectl`|`false`|`1.18.2`| kubectl version. kubectl vesion can be found [here](https://github.com/kubernetes/kubernetes/releases)|
|`kustomize`|`false`|`3.5.5`| kustomize version. kustomize vesion can be found [here](https://github.com/kubernetes-sigs/kustomize/releases)|
|`helm`|`false`|`2.16.7`| helm version. helm vesion can be found [here](https://github.com/helm/helm/releases)|
|`helmv3`|`false`|`3.2.1`| helm v3 version. helm v3 vesion can be found [here](https://github.com/helm/helm/releases)|
|`kubeval`|`false`|`0.15.0`| kubeval version. kubeval vesion can be found [here](https://github.com/instrumenta/kubeval/releases)|
|`conftest`|`false`|`0.19.0`| conftest version. conftest vesion can be found [here](https://github.com/open-policy-agent/conftest/releases)|

> Supported Environments: Linux

### Outputs

|Parameter|Description|
|:--:|:--|
|`kubectl_path`| kubectl command path |
|`kustomize_path`| kustomize command path |
|`helm_path`| helm command path |
|`helmv3_path`| helm v3 command path |
|`kubeval_path`| kubeval command path |
|`conftest_path`| conftest command path |
|`yq_path`| yq command path |

### Sample Workflow

Specific versions for the commands can be setup by adding inputs parameters like this:

{% raw %}
```yaml
  test: 
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - uses: yokawasa/action-setup-kube-tools@v0.1.0
      with:
        kubectl: '1.17.1'
        kustomize: '3.7.0'
        helm: '2.16.7'
        helmv3: '3.2.4'
        kubeval: '0.14.0'
        conftest: '0.18.2'
      id: setup
    - run: |
        kubectl version --client
        kustomize version
        helm version --client
        helmv3 version
        kubeval --version
        conftest --version
```
{% endraw %}

Default versions for the commands will be setup if you don't give any inputs like this:

{% raw %}
```yaml
  test: 
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - uses: yokawasa/action-setup-kube-tools@v0.1.0
      id: setup
    - run: |
        kubectl version --client
        kustomize version
        helm version --client
        helmv3 version
        kubeval --version
        conftest --version
```
{% endraw %}


## Developing the action

Install the dependencies  
```bash
npm install
```

Build the typescript and package it for distribution by running [ncc](https://github.com/zeit/ncc)
```bash
npm run build && npm run pack
```

Finally push the resutls
```
git add dist
git commit -a -m "prod dependencies"
git push origin releases/v0.1.0
```

Enjoy the action!
