---
layout: post
status: publish
published: true
title: GitHub Actions - SQLCheck Action
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

I've published a new GitHub Action called [SQLCheck Action](https://github.com/yokawasa/action-sqlcheck) ([View on Marketplace](https://github.com/marketplace/actions/sqlcheck-action)). The action automatically identifies anti-patterns in SQL queries using [sqlcheck](https://github.com/jarulraj/sqlcheck) when PR is requested  and comment on the PR if risks are found in the queries. 

![](https://raw.githubusercontent.com/yokawasa/action-sqlcheck/master/assets/action-sqlcheck-pr-comment.png)

## Usage

Supports `pull_request` event type.

### Inputs
|Parameter|Required|Default Value|Description|
|:--:|:--:|:--:|:--|
|`post-comment`|false|true|Post comment to PR if it's true|
|`token`|true|""|GitHub Token in order to add comment to PR|
|`risk-level`|false|3|Set of SQL anti-patterns to check: 1,2, or 3<br>- 1 (all anti-patterns, default)<br>- 2 (only medium and high risk anti-patterns)<br> - 3 (only high risk anti-patterns) |
|`verbose`|false|false|Add verbose warnings to SQLCheck analysis result|
|`postfixes`|false|"sql"|List of file postfix to match ( separator: comma )|

## Sample Workflow

> .github/workflows/test.yml

```yml
name: sqlcheck workflow
on: pull_request

jobs:
  sqlcheck:
    name: sqlcheck job 
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - uses: yokawasa/action-sqlcheck@v1.2.1
      with:
        post-comment: true
        risk-level: 3
        verbose: false
        token: ${{ secrets.GITHUB_TOKEN }}
```

Enjoy the action!
