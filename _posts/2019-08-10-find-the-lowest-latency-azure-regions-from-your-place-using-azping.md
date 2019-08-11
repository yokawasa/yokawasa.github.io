---
layout: post
status: publish
published: true
title: Find the lowest latency Azure regions from your place using azping
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
date: '2019-08-10 10:32:40 +0900'
tags:
- Azure
- ping
- azping
---

[azping](https://github.com/yokawasa/azping) is a command line tools that help you find the lowest latency Azure Region from your place. It acutally reports median latency to Azure regions. It is a fork of [gcping](https://github.com/GoogleCloudPlatform/gcping). 

## What does azping actually evalulate?

azping evalulate the median latecy of http requests to Azure blob storage endpoints located in each of Azure reagions from your place. Number of requests to be made to each region is `5` by default, but it can be changed with `-n` parameter that you can give in executing azping command. 

Here is a list of Azure blob storage endpoints that azping evalulates:

| region | endpoint |
|------|------|
| eastasia|            azpingeastasia.blob.core.windows.net/ping |
| southeastasia|       azpingsoutheastasia.blob.core.windows.net/ping |
| centralus|           azpingcentralus.blob.core.windows.net/ping |
| eastus|              azpingeastus.blob.core.windows.net/ping |
| eastus2|             azpingeastus2.blob.core.windows.net/ping |
| westus|              azpingwestus.blob.core.windows.net/ping |
| northcentralus|      azpingnorthcentralus.blob.core.windows.net/ping |
| southcentralus|      azpingsouthcentralus.blob.core.windows.net/ping |
| northeurope|         azpingnortheurope.blob.core.windows.net/ping |
| westeurope|          azpingwesteurope.blob.core.windows.net/ping |
| japanwest|           azpingjapanwest.blob.core.windows.net/ping |
| japaneast|           azpingjapaneast.blob.core.windows.net/ping |
| brazilsouth|         azpingbrazilsouth.blob.core.windows.net/ping |
| australiaeast|       azpingaustraliaeast.blob.core.windows.net/ping |
| australiasoutheast|  azpingaustraliasoutheast.blob.core.windows.net/ping |
| southindia|          azpingsouthindia.blob.core.windows.net/ping |
| centralindia|        azpingcentralindia.blob.core.windows.net/ping |
| westindia|           azpingwestindia.blob.core.windows.net/ping |
| canadacentral|       azpingcanadacentral.blob.core.windows.net/ping |
| canadaeast|          azpingcanadaeast.blob.core.windows.net/ping |
| uksouth|             azpinguksouth.blob.core.windows.net/ping |
| ukwest|              azpingukwest.blob.core.windows.net/ping |
| westcentralus|       azpingwestcentralus.blob.core.windows.net/ping |
| westus2|             azpingwestus2.blob.core.windows.net/ping |
| koreacentral|        azpingkoreacentral.blob.core.windows.net/ping |
| koreasouth|          azpingkoreasouth.blob.core.windows.net/ping |
| francecentral|       azpingfrancecentral.blob.core.windows.net/ping |
| australiacentral|    azpingaustraliacentral.blob.core.windows.net/ping |
| uaenorth|            azpinguaenorth.blob.core.windows.net/ping |
| southafricanorth|    azpingsouthafricanorth.blob.core.windows.net/ping |

> NOTE
> All blob storage endpoints are created with the following scripts (Just in case I leave the procedures):
> ```bash
> $ git clone https://github.com/yokawasa/azping.git
> $ cd setup
> # Edit RESOURCE_GROUP and REGION_LIST variables in env.sh
> $ vi env.sh
> # Read variables as enviroment variables
> $ source env.sh
> # Execute the following script that execute the following|
> # (1) Create resource group for azping
> # (2) Create blob storage accounts in each of Azure regions
> # (3) Create $root container and upload ping file to the container
> # (4) Check accessibility to all blob storage endpoints
> $ ping-entrypoint.sh
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

Or, you can always build the binary from the source code like this:
```
$ git clone https://github.com/yokawasa/azping.git
$ cd azping
$ make
$ tree bin

bin
├── azping_darwin_amd64
├── azping_linux_amd64
└── azping_windows_amd64
```

## Usage

```txt
$ azping [options...]

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
