---
layout: post
status: publish
published: true
title: GitHub Actions - Elastic Cloud Control (ecctl) tool installer
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
date: '2020-04-30 11:00:00 +0900'
tags:
- GitHubActions
- static-analysis
- sqlcheck
---

I've published a new GitHub Action called [action-setup-ecctl](https://github.com/yokawasa/action-setup-ecctl) ([View on Marketplace](https://github.com/marketplace/actions/elastic-cloud-control-ecctl-tool-installer)). The action installs a specific version of [ecctl](https://github.com/elastic/ecctl) (Elastic Cloud control tool) and cache it on the runner.

## Usage

### Inputs

|Parameter|Required|Default Value|Description|
|:--:|:--:|:--:|:--|
|`token`|`false`|`latest`|Ecctl tool version such as `v1.0.0-beta3`. Ecctl vesion can be found [here](https://github.com/elastic/ecctl/releases).|

> Supported Environments: Linux and macOS

### Sample Workflow

A specific version of ecctl can be setup by giving an input - `version` like this:
```yaml
- uses: yokawasa/action-setup-ecctl@v0.1.0
  with:
    version: 'v1.0.0-beta3'   # default is 'latest'
  id: setup
- run: |
  ecctl=${{steps.setup.outputs.ecctl-path}}
  ${ecctl} version
```

The latest version of ecctl will be setup if you don't give an input like this:

```yaml
- uses: yokawasa/action-setup-ecctl@v0.1.0
  id: setup
- run: |
  ecctl=${{steps.setup.outputs.ecctl-path}}
  ${ecctl} version
```

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
