---
author: Yoichi Kawasaki
date: "2020-06-22T01:00:00Z"
published: true
status: publish
tags:
- GitHubActions
- static-analysis
- sqlcheck
title: GitHub Actions - Elastic Cloud Control (ecctl) tool installer
---

I've published a new GitHub Action called [action-setup-ecctl](https://github.com/yokawasa/action-setup-ecctl) ([View on Marketplace](https://github.com/marketplace/actions/elastic-cloud-control-ecctl-tool-installer)). The action installs a specific version of [ecctl](https://github.com/elastic/ecctl) (Elastic Cloud control tool) and cache it on the runner.

## Usage

### Inputs

|Parameter|Required|Default Value|Description|
|:--:|:--:|:--:|:--|
|`version`|`false`|`latest`|Ecctl tool version such as `v1.0.0-beta3`. Ecctl vesion can be found [here](https://github.com/elastic/ecctl/releases).|

> Supported Environments: Linux and macOS

### Outputs

|Parameter|Description|
|:--:|:--|
|`ecctl-path`| ecctl command path |

### Sample Workflow

A specific version of ecctl can be setup by giving an input - `version` like this:
{% raw %}
```yaml
- uses: yokawasa/action-setup-ecctl@v0.1.0
  with:
    version: 'v1.0.0-beta3'   # default is 'latest'
  id: setup
- run: |
  ecctl version
```
{% endraw %}

The latest version of ecctl will be setup if you don't give an input like this:

{% raw %}
```yaml
- uses: yokawasa/action-setup-ecctl@v0.1.0
  id: setup
- run: |
  ecctl version
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
