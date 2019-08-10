---
layout|post
status|publish
published|true
title|Finding the lowest latency Azure Region from your place with azping
author:
  display_name|Yoichi Kawasaki
  url|http://github.com/yokawasa
author_login|yoichi
author_email|yokawasa@gmail.com
author_url|http://github.com/yokawasa
date|'2019-08-10 8:32:40 +0900'
tags:
|Azure
|ping
|azping
---

[azping](https://github.com/yokawasa/azping) is a command line tools that help you find the lowest latency Azure Region from your place. It acutally reports median latency to Azure regions. It is fork of [gcping](https://github.com/GoogleCloudPlatform/gcping). 

## What does azping actually evalulate?

azping evalulate the median latecy of http request to Azure blob storage endpoints located in each of Azure reagions from your place. Here are a list of Azure blob storage endpoints:

> NOTE
> All blob storage endpoints are created with the following scripts (Just in case I leave the procedures):
> ```
> git clone https://github.com/yokawasa/azping.git
> cd setup
> # Edit RESOURCE_GROUP and REGION_LIST variables in env.sh
> cat env.sh
> # Read variables as enviroment variables
> source env.sh
> # Execute the following script that execute the following|
> # (1) Create resource group for azping
> # (2) Create blob storage accounts in each of Azure regions
> # (3) Create $root container and upload ping file to the container
> # (4) Check accessibility to all blob storage endpoints
> ping-entrypoint.sh
> ```

## How to install
Linux 64-bit|https://azpingrelease.blob.core.windows.net/azping_linux_amd64
```
$ curl https://azpingrelease.blob.core.windows.net/azping_linux_amd64 > azping && chmod +x azping
```
Mac 64-bit|https://azpingrelease.blob.core.windows.net/azping_darwin_amd64
```
$ curl https://azpingrelease.blob.core.windows.net/azping_darwin_amd64 > azping && chmod +x azping
```
Windows 64-bit|https://azpingrelease.blob.core.windows.net/azping_windows_amd64
```
# use WSL-bash
$ curl https://azpingrelease.blob.core.windows.net/azping_windows_amd64 > azping && chmod +x azping
```

## Usage
```
azping [options...]

Options:
-n   Number of requests to be made to each region.
     By default 5; can't be negative.
-c   Max number of requests to be made at any time.
     By default 10; can't be negative or zero.
-t   Timeout. By default, no timeout.
     Examples|"500ms", "1s", "1s500ms".
-top If true, only the top region is printed.

-csv CSV output; disables verbose output.
-v   Verbose output.
```

## Screenshot

Here is an output of azping when I executed azping from my home in Tokyo

![]({{ site.url }}/assets/20190810-azping.png)


Enjoy azping!
