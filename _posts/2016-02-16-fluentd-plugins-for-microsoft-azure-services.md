---
layout: post
status: publish
published: true
title: fluentd plugins for Microsoft Azure Services
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 78
wordpress_url: http://unofficialism.info/posts/?p=78
date: '2016-02-16 09:56:03 +0900'
date_gmt: '2016-02-16 00:56:03 +0900'
categories:
- Azure
- English
tags:
- AzureSearch
- DocumentDB
- fluentd
- AzureTables
- AzureBlobStorage
- EventHubs
- AzureFunctions
- LogAnalytics
---

UPDATED:

- 2016-12-10: Added [fluent-plugin-azure-loganalytics](https://yokawasa.github.io/fluent-plugin-azure-loganalytics/) to the list
- 2016-11-23: Added [fluent-plugin-azurefunctions](https://yokawasa.github.io/fluent-plugin-azurefunctions/) to the list

Here is a list of [fluentd](http://www.fluentd.org/) plugins for Microsoft Azure Services. 

| Plugin Name   | Target Azure Services | Note | 
| ------------- | ------------- | ------------- |
| <a href="https://github.com/htgc/fluent-plugin-azurestorage">fluent-plugin-azurestorage</a> | Blob Storage | Azure Storate output plugin buffers logs in local file and upload them to Azure Storage periodicall | 
| <a href="https://github.com/htgc/fluent-plugin-azureeventhubs">fluent-plugin-azureeventhubs</a>| Event Hubs | Azure Event Hubs buffered output plugin for Fluentd. Currently it supports only HTTPS (not AMQP) | 
| <a href="https://github.com/heocoi/fluent-plugin-azuretables">fluent-plugin-azuretables</a> | Azure Tables | Fluent plugin to add event record into Azure Tables Storage| 
| <a href="https://github.com/yokawasa/fluent-plugin-azuresearch">fluent-plugin-azuresearch</a>| Azure Search | Fluent plugin to add event record into Azure Search| 
| <a href="https://github.com/yokawasa/fluent-plugin-documentdb">fluent-plugin-documentdb</a> | Cosmos DB | Fluent plugin to add event record into Azure Cosmos DB| 
| <a href="https://github.com/yokawasa/fluent-plugin-azurefunctions">fluent-plugin-azurefunctions</a>| Azure Functions| Azure Functions (HTTP Trigger) output plugin for Fluentd. The plugin aggregates semi-structured data in real-time and writes the buffered data via HTTPS request to HTTP Trigger Function | 
| <a href="https://github.com/yokawasa/fluent-plugin-azure-loganalytics">fluent-plugin-azure-loganalytics</a> | Log Analytics | Azure Log Analytics output plugin for Fluentd. The plugin aggregates semi-structured data in real-time and writes the buffered data via HTTPS request to Azure Log Analytics| 

(as of Nov 23, 2016)

[![fluentd](https://farm2.staticflickr.com/1471/24937329432_69aa1c6983_c.jpg)
](http://www.fluentd.org/)
