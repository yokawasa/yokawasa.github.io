---
layout: post
status: publish
published: true
title: Moving to Jekyll based Github page
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
date: '2018-07-05 17:32:40 +0900'
date_gmt: '2018-07-05 08:32:40 +0900'
tags:
- jekyll
- wordpress
---

I’ve moved my blog site from Wordpress to Jekyll based Github page. There are a few reasons for this:
- I wanted to manage my blog data on Github
- I wanted to switch from HTML based to Markdown
- I wanted more static approach like generating once, not dynamically rendering for every request (for performance reason)

After a few minutes of googling, I came up with [Jekyll](https://jekyllrb.com/) and found it much easier to manage my blog data with Jekyll than with Wordpress. I love Jekyll's simplicity. For the one who want to do the same migration, I'd like to share memo on how I migrated the blog site from Wordpress to Jekyll based Github page.

## What to do (What I've done)
- [ENV] Wordpress is running on Ubuntu 16.04, and I work on all migration works on MacOS (10.13.5)
- Importing my posts from a self-hosted Wordpress
    - Install jekyll-import ruby gem and all additional packages needed to import my posts from a self-hosted Wordpress
    - Import my posts from the Wordpress 
    - Convert HTML files to Markdown format files (including manual works)
- Publish the site uisng Jekyll and Github Pages
    - Check ruby and github-pages version used in Github page, and install the same github-pages gem on the same ruby version 
    - Run Jekyll locally and verify the site content
    - Create Github project for hosting the site
    - Publish the site to Github uisng git
    - Setup Custom domain
- Tips

## Importing my posts from a self-hosted Wordpress

### Install ruby gem packages to import posts from Wordpress
First, check ruby version as jekyll require > `ruby-2.X`, and upgrade its version if it's not > ruby-2.X (I use `rbenv` to manage multiple ruby versions on MacOS)
```sh
$ ruby --version
ruby 2.4.1p111 (2017-03-22 revision 58053) [x86_64-darwin17]
```

Install `jekyll-import` ruby gem and all additional packages needed to import my posts from a self-hosted Wordpress
```sh
$ gem install jekyll-import
# Additional packages for importing from Wordpress
$ gem install sequel
$ gem install unidecode
$ gem install htmlentities
$ gem install mysql2
```

### Import my posts from the Wordpress 
Run the following command to import my posts from the hosted Wordpress. Make sure that MySQL server used in Wordpress is accessible
```sh
ruby -rubygems -e 'require "jekyll-import";
    JekyllImport::Importers::WordPress.run({
      "dbname"   => "<db name>",
      "user"     => "<db user>",
      "password" => "<db password>",
      "host"     => "<db host or IP>",
      "port"     => "<db port: 3306>",
      "socket"   => "",
      "table_prefix"   => "wp_",
      "site_prefix"    => "",
      "clean_entities" => true,
      "comments"       => false,
      "categories"     => true,
      "tags"           => true,
      "more_excerpt"   => true,
      "more_anchor"    => true,
      "status"         => ["publish"]
    })'
```
All imported blog ports (in HTML) are stored in `_posts` directory. See also [Jekyll documentation](https://import.jekyllrb.com/docs/wordpress/)

### Convert HTML files to Markdown format files (including manual works)

Frist, I converted all HTML files stored in `_posts` directory to markdown using [h2m](https://github.com/island205/h2m) npm package

```sh
# Install h2m npm package
$ npm install h2m -g
# Here is how to conver html to markdown using h2m
# h2m test.html > test.md

# Run the following one liner converted all HTML files to markdown
for h in `ls -1 *.html`;do echo $h; m=$(echo $h | sed s/\.html//g).md; h2m $h > $m;  ;done
```

Then, do some manual modification to `*.md` files. Markdown don't support all HTML components such as embeded, iframe, align, etc. What I did for this modification parts are basically:
- All embeded parts such as Youtube video, slideshare, and gist links (Markdown doesn't support embeded tags)
- Table tags part (I used Wordpress Table plugin that express table with very special tag, which of course can not be automatically convered to Markdown )

[Markdown CheatSheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#videos) was very helpful in this part


## Publish the site uisng Jekyll and Github Pages

### Install the same github-pages gem and ruby as the ones used in Github page

Go to [this page](https://pages.github.com/versions/) to get ruby and github-pages version used in Github page. This time it was
- ruby: 2.4.2
- github-pages: 186

Then, I installed the exactly the same version of ruby and github-pages gem on my local
```sh
# Install Ruby using rbenv
$ rbenv install 2.4.2
$ rbenv global 2.4.2

# Install github-pages ruby gem package
$ gem install github-pages -v 186
```

### Run Jekyll locally and verify the site content

Create Jekyll template site and copy all blog pages (Markdown) on to `_posts` directory under the site template root

```sh
# Check jekyll version
$ jekyll -v
jekyll 3.7.3

# Create new Jekyll template site with jekyll command (here I create template site named `myblog`)
$ jekyll new myblog
$ tree myblog
myblog
├── 404.html
├── Gemfile
├── _config.yml
├── _posts
│   └── 2018-07-06-welcome-to-jekyll.markdown
├── about.md
└── index.md

# Remove Gemfile
# Ruby will use the contents of your Gemfile to build your Jekyll site. So you basically add the same version of github-pages and ruby to Gemfile as the ones in Github Pages, but I removed it to make life simple as I've already synced the local version of github-pages and ruby with the ones in Github Pages but. If you possibly run Jekyll site on multiple environments, it's definitly better to manage package versions with Gemfile
$ rm Gemfile

# Copy all blog pages (Markdown) on to `_posts` directory under the site template root
$ cp *.md _posts/
```

Run the local webserver like this and verify the site content:
```sh
$ jekyll s
$ curl http://localhost:4000
```
See also [Setting up your GitHub Pages site locally with Jekyll](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)

### Create Github project for hosting the site

Create my user site by following instructions in [https://pages.github.com/](https://pages.github.com/). In this case, I've created **yokawasa.github.io** under my account **https://github.com/yokawasa**, by which I can access `*.html|*.md` in **github.com/yokawasa/yokawasa.github.io** with the URL of **https://yokawasa.github.io/**`(*.html|*.md)`


### Publish the site to Github uisng git
Copy all files under Jekyll site content that I've verified onto yokawasa.github.io and push them to Github

```sh
$ git clone https://github.com/yokawasa/yokawasa.github.io
$ cp -pr myblog/* yokawasa.github.io/
# make sure to copy .gitignore file as well

$ cd yokawasa.github.io
$ git add -all
$ git commit -m "Initial commit"
$ git push -u origin master
$ open https://yokawasa.github.io
```

### Setup Custom domain

I added my custom domain (unofficialism.info) to my Github page site by following [Quick start: Setting up a custom domain](https://help.github.com/articles/quick-start-setting-up-a-custom-domain/). Here are what I've actually done:

First, check my current DNS record for my custom domain
```sh
$ dig +noall +answer unofficialism.info
unofficialism.info.     146     IN      A       XX.XXX.XXX.XX
```

Then, follow my DNS provider's instructions to create A records that point my custom domain to the following IP addresses (These are fixed IPs but these IPs can be obtained simply by running dig for username.github.com):
```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

Confirm that the DNS record is set up correctly
```sh
$ dig +noall +answer unofficialism.info

unofficialism.info.     430     IN      A       185.199.108.153
unofficialism.info.     430     IN      A       185.199.109.153
unofficialism.info.     430     IN      A       185.199.110.153
unofficialism.info.     430     IN      A       185.199.111.153
```
In addition, add my custom domain for my GitHub Pages site in Github Page setting page by following [Adding or removing a custom domain for your GitHub Pages site](https://help.github.com/articles/adding-or-removing-a-custom-domain-for-your-github-pages-site/)

Finally, confirm the access to my Github page with my custom domain
```sh
$ curl https://unofficialism.info
```
