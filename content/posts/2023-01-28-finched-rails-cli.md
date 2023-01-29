---
author: Yoichi Kawasaki
date: "2023-01-28T01:00:00Z"
published: true
status: publish
images: ['/assets/20230128-finched-rails.png']
tags:
- finch
- docker
- rails
title: Finched Rails CLI makes it a breeze to try out Rails withough installing anything but Finch
---

![](/assets/20230128-finched-rails.png)

## Preface
In this article, I’d like to introduces [Finched Rails CLI](https://github.com/yokawasa/finched-rails-cli) that makes it a breeze to try out Rails without installing anything but [Finch](https://github.com/runfinch/finch). Finch is an OSS CLI for building, running, and publishing Linux containers create by AWS. As long as you have Finch, you can quick start a Rails app only by copying and pasting a few commands. Finched Rails CLI is a fork of [Docked Rails CLI](https://github.com/rails/docked) and simply a Finch version of it.
> NOTE: As of Jan 2023, Finch only supports macOS (on all Mac CPU architectures). For its future supportability of any other architectures, please watch [Finch repo](https://github.com/runfinch/finch)

Why I created Finched Rails CLI? It's very simple. I don't currenlty use Docker Desktop for managing Linux containers on my Desktop for its complexity and high resource consuming resason. I use Finch instead for it. 


## Quickstart

Please install Finch if you don't have on your desktop

```bash
brew install --cask finch
```

Then copy and paste these below into your terminal:

```bash
finch volume create ruby-bundle-cache

alias finched='finch run --rm -it -v ${PWD}:/rails -v ruby-bundle-cache:/bundle -p 3000:3000 --entrypoint "" ghcr.io/yokawasa/rails/cli'
```

Now you're ready to start develop Rails app using `finched`, the alias you've just created.

Let's create a Rails project named `weblog` and start the Rails server as follows:

```bash
finched rails new weblog
cd weblog
finched rails generate scaffold post title:string body:text
finched rails db:migrate
finched rails server
```

Your Rails app is running, and you can check it on http://localhost:3000/posts.

```
open http://localhost:3000/posts

```

![](/assets/20230128-finched-rails-weblog-posts.png)

That's it!

Enjoy developing Rails app with Finch!
