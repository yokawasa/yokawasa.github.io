---
author: Yoichi Kawasaki
categories:
- Azure
- English
date: "2018-01-06T23:05:28Z"
date_gmt: 2018-01-06 14:05:28 +0900
published: true
status: publish
tags:
- AzureMediaServices
- TrafficManager
- Resiliency
title: Controlling Azure Media Services traffic with Traffic Manager
---

This is an article on how you can achieve Azure Media Services (AMS) streaming traffic distribution with Traffic Manager.

## The process for a client to find target AMS streaming endpoints

The figure shows how a client find target AMS streaming endpoints with Traffic Manager and requests from video players are distributed to streaming endpoints in AMS:

![Controling Azure Media Services Traffic with Traffic Manager](https://farm5.staticflickr.com/4645/25663550048_4f106e0f5e_c.jpg)

When AMS endpoints are added to an Azure Traffic Manager profile, Azure Traffic Manager keeps track of the status of the endpoints (running, stopped, or deleted) so that it can decide which of those endpoints should receive traffic. You can configure the way to route network traffic to the endpoints by choosing [traffic routing methods](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-routing-methods) available in Traffic Manager.

## Configuration procedure

Suppose that you have 2 AMS accounts (**amsaccount1**, **amsaccount2**), and that you want to distribute requests to your Traffic Manager domain (**myamsstreaming.trafficmanager.net**) from video player clients to streaming endpoints in AMS. When AMS endpoints are added to an Azure Traffic Manager profile (**myamsstreaming**), Azure Traffic Manager keeps track of the status of your AMS streaming endpoints (running, stopped, or deleted) so that it can decide which of those endpoints should receive traffic. However, when it comes to AMS endpoints, it has to be toward a custom domain, NOT simply a Traffic Manager domain for clients' traffic to be distributed into AMS endpoints using Traffic Manager. So let's suppose you prepare a custom domain: **streaming.mydomain.com**.

Example accounts and domains

- AMS Account1: **amsaccount1** (Streaming endpoint: amsaccount1.streaming.mediaservices.windows.net)
- AMS Account2: **amsaccount2** (Streaming endpoint: amsaccount2.streaming.mediaservices.windows.net)
- Traffic Manager Domain: **myamsstreaming.trafficmanager.net**
- Custom Domain: **streaming.mydomain.com**

### (1) Point a custom domain to a Traffic Manager domain

You add the following alias (CNAME) to point the custom domain to the traffic manager domain:
```
streaming.mydomain.com IN CNAME myamsstreaming.trafficmanager.net
```

Useful Link for this step: [Point a company Internet domain to an Azure Traffic Manager domain](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-point-internet-domain)

### (2) Add Azure Media Services origin hosts to the Traffic Manager

Add AMS endpoints to an Azure Traffic Manager profile as "External Endpoint" via either Azure Portal or Azure CLI / PowerShell. Here is an image of adding endpoints in Azure portal: 

![Add endpoints to Traffic Manager](https://farm5.staticflickr.com/4637/38826321084_edb7bcdac6_c.jpg)

Useful Link for this step: [Add, disable, enable, or delete endpoints](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-endpoints)

### (3) Custom domain name ownership verification

First of all, get Media Service Account IDs (GUID) for your AMS accounts in this step. To find the Azure Media Service ID , go to the Azure portal and select your Media Service account. The Azure Media Service ID appears on the right of the DASHBOARD page. Let's suppose you get the following account IDs for AMS accounts (amsaccount1, amsaccount2):

- Account ID: **8dcbe520-59c7-4591-8d98-1e765b7f3729** for AMS account: amsaccount1
- Account ID: **5e0e6784-4ed0-40a0-8444-33da6d4f7171** for AMS account: amsaccount2

Then, create CNAME that maps **(accountId).(parent custom domain)** to **verifydns.(mediaservices-dns-zone)**. This is necessary to proves that the Azure Media Services ID has the ownership of the custom domain. Here are CNAMEs to create in this step:

```
## <MediaServicesAccountID>.<custom parent domain> IN CNAME verifydns.<mediaservices-dns-zone>

## Custom name Ownership verification for amsaccount1
8dcbe520-59c7-4591-8d98-1e765b7f3729.mydomain.com IN CNAME  verifydns.mediaservices.windows.net

## Custom name Ownership verification for amsaccount2
5e0e6784-4ed0-40a0-8444-33da6d4f7171.mydomain.com IN CNAME  verifydns.mediaservices.windows.net
```

Refer to CustomHostNames section of [StreamingEndpoint document](https://docs.microsoft.com/rest/api/media/operations/streamingendpoint) to learn more about the configuration in this step.

### (4) Add Custom host name to each Azure Media Service streaming endpoint

Once you completed "Custom domain name ownership verification" configuration, you then need to configure to add custom host name to each Azure Media Service streaming endpoint. Unfortunately new portal doesn't include capability to add custom domain to the streaming endpoint. You can set it either using [REST API](https://docs.microsoft.com/en-us/rest/api/media/operations/streamingendpoint) directly or using [Azure Media Explorer](https://github.com/Azure/Azure-Media-Services-Explorer). Here is how you add custom host name to the streaming endpoint in Azure Media Explorer:

![azure-media-explorer-custom-host-streaming-endpoint](https://farm5.staticflickr.com/4658/40310439831_0354721aa0_c.jpg)

Choose "Streaming endpoint" in the top menu, right click on your streaming endpoint, and select "Streaming endpoint information and settings". In Streaming endpoint information form, type in your custom host name for the endpoint, and click "Update settings and close". That's it.

### (5) Test video playback with your custom domain

Once all configurations above are completed (+ DNS settings are reflected), you will see the custom domain name lookup points to either of AMS endpoints added to the traffic manager like this:

```
$ dig streaming.mydomain.com

;; ANSWER SECTION:
streaming.mydomain.com. 600 IN CNAME myamsstreaming.trafficmanager.net.
myamsstreaming.trafficmanager.net. 300 IN CNAME amsaccount1.streaming.mediaservices.windows.net.
amsaccount1.streaming.mediaservices.windows.net. 60 IN CNAME wamsorigin-origin-903f1f37105244fba2270ae7b64021bd.cloudapp.net.
wamsorigin-origin-903f1f37105244fba2270ae7b64021bd.cloudapp.net. 60 IN A 104.215.4.76
```

Make sure to check if the custom name lookup points to the other endpoint when one of the endpoints are down.

Finally, check if you can playback video with your custom domain. 

```shell
$ curl http://amsaccount1.streaming.mediaservices.windows.net/ee2e5286-d3fa-40fc-a393-c3c2a3ca5a84/BigBuckBunny.ism/manifest

 <?xml version="1.0" encoding="UTF-8"?><SmoothStreamingMedia MajorVersion="2" MinorVersion="2" Duration="84693333" TimeScale="10000000"><StreamIndex Chunks="2" Type="audio" Url="QualityLevels({bitrate})/Fragments(aac_eng_2_128={start time})" QualityLevels="1" Language="eng" Name="aac_eng_2_128"><QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="128000" FourCC="AACL" CodecPrivateData="1190" Channels="2" PacketSize="4" SamplingRate="48000" /><c t="0" d="60160000" /><c d="24533333" /></StreamIndex><StreamIndex Chunks="2" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="5"><QualityLevel Index="0" Bitrate="2896000" FourCC="H264" MaxWidth="1280" MaxHeight="720" CodecPrivateData="000000016764001FACD9405005BB011000000300100000030300F18319600000000168EBECB22C" /><QualityLevel Index="1" Bitrate="1789000" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FACD940F0117EF011000003000100000300300F1831960000000168EBECB22C" /><QualityLevel Index="2" Bitrate="946000" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EACD940A02FF97011000003000100000300300F162D960000000168EBECB22C" /><QualityLevel Index="3" Bitrate="612000" FourCC="H264" MaxWidth="480" MaxHeight="270" CodecPrivateData="0000000167640015ACD941E08FEB011000000300100000030300F162D9600000000168EBECB22C" /><QualityLevel Index="4" Bitrate="324000" FourCC="H264" MaxWidth="320" MaxHeight="180" CodecPrivateData="000000016764000DACD941419F9F011000000300100000030300F14299600000000168EBECB22C" /><c t="0" d="60000000" /><c d="24583333" /></StreamIndex></SmoothStreamingMedia>
```

```shell
$ curl http://streaming.mydomain.com/ee2e5286-d3fa-40fc-a393-c3c2a3ca5a84/BigBuckBunny.ism/manifest

 <?xml version="1.0" encoding="UTF-8"?><SmoothStreamingMedia MajorVersion="2" MinorVersion="2" Duration="84693333" TimeScale="10000000"><StreamIndex Chunks="2" Type="audio" Url="QualityLevels({bitrate})/Fragments(aac_eng_2_128={start time})" QualityLevels="1" Language="eng" Name="aac_eng_2_128"><QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="128000" FourCC="AACL" CodecPrivateData="1190" Channels="2" PacketSize="4" SamplingRate="48000" /><c t="0" d="60160000" /><c d="24533333" /></StreamIndex><StreamIndex Chunks="2" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="5"><QualityLevel Index="0" Bitrate="2896000" FourCC="H264" MaxWidth="1280" MaxHeight="720" CodecPrivateData="000000016764001FACD9405005BB011000000300100000030300F18319600000000168EBECB22C" /><QualityLevel Index="1" Bitrate="1789000" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FACD940F0117EF011000003000100000300300F1831960000000168EBECB22C" /><QualityLevel Index="2" Bitrate="946000" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EACD940A02FF97011000003000100000300300F162D960000000168EBECB22C" /><QualityLevel Index="3" Bitrate="612000" FourCC="H264" MaxWidth="480" MaxHeight="270" CodecPrivateData="0000000167640015ACD941E08FEB011000000300100000030300F162D9600000000168EBECB22C" /><QualityLevel Index="4" Bitrate="324000" FourCC="H264" MaxWidth="320" MaxHeight="180" CodecPrivateData="000000016764000DACD941419F9F011000000300100000030300F14299600000000168EBECB22C" /><c t="0" d="60000000" /><c d="24583333" /></StreamIndex></SmoothStreamingMedia>
```

## Useful Links

- [traffic routing methods](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-routing-methods)
- [Point a company Internet domain to an Azure Traffic Manager domain](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-point-internet-domain)
- [Add, disable, enable, or delete endpoints](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-endpoints)
- [StreamingEndpoint CustomHostNames Configuration](https://docs.microsoft.com/rest/api/media/operations/streamingendpoint)
