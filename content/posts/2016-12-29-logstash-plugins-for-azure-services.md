---
author: Yoichi Kawasaki
categories:
- Azure
- English
date: "2016-12-29T23:25:36Z"
date_gmt: 2016-12-29 14:25:36 +0900
published: true
status: publish
tags:
- Ruby
- logstash
- ElasticStack
title: Logstash plugins for Microsoft Azure Services
---

[Logstash](https://www.elastic.co/products/logstash) is an open source, server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then sends it to your favorite destinations. Here is a list of [logstash](https://www.elastic.co/products/logstash) plugins for Microsoft Azure Services. 

| Plugin Name   | Target Azure Services | Note | 
| ------------- | ------------- | ------------- |
| <strong><a href="https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureeventhub">logstash-input-azureeventhub</a></strong> |	EventHub	| Logstash input plugin reads data from specified Azure Event Hubs |
| <strong><a href="https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob">logstash-input-azureblob</a></strong> |	Blob Storage |	Logstash input plugin that reads and parses data from Azure Storage Blobs |
| <strong><a href="https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azuretopic">logstash-input-azuretopic</a></strong> | 	Service Bus Topic | Logstash input plugin reads messages from Azure Service Bus Topics |
| <strong><a href="https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azuretopicthreadable">logstash-input-azuretopicthreadable</a></strong> |	Service Bus Topic |	Logstash input plugin reads messages from Azure Service Bus Topics using multiple threads |
| <strong><a href="https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-output-applicationinsights">logstash-output-applicationinsights</a></strong> |	Application Insights | Logstash output plugin that store events to Application Insights |
| <strong><a href="https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azurewadtable">logstash-input-azurewadtable </a></strong> | Table Storage | Logstash input plugin for Azure Diagnostics. Specifically pulling diagnostics data from Windows Azure Diagnostics tables |
| <strong><a href="https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azuretopicthreadable">logstash-input-azurewadeventhub</a></strong> |	EventHub | Logstash input plugin reads Azure diagnostics data from specified Azure Event Hubs and parses the data for output |
| <strong><a href="https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azurewadtable">logstash-input-azurewadtable </a></strong> | Table Storage	| Logstash input plugin reads Azure diagnostics data from specified Azure Storage Table and parses the data for output |
| <strong><a href="https://github.com/yokawasa/logstash-output-documentdb">logstash-output-documentdb</a></strong> | Cosmos DB	| logstash output plugin that stores events to Azure Cosmos DB |
| <strong><a href="https://github.com/yokawasa/logstash-output-azuresearch">logstash-output-azuresearch</a></strong> |	Azure Search | logstash output plugin that stores events to Azure Search |
| <strong><a href="https://github.com/yokawasa/logstash-output-azure_loganalytics">logstash-output-azure_loganalytics</a></strong> | Log Analytics | logstash output plugin that stores events to Azure Log Analytics |
| <strong><a href="https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html">logstash-input-jdbc</a></strong> | SQL Database, Azure Database for MySQL/PostgreSQL | Input plugin to ingest data in any database with a JDBC interface into Logstash that support most of major RDBMS such as MySQL、PostgreSQL、OracleDB、Microsoft SQL, etc |

(as of Dec 29, 2016)

[
![logstash](https://c3.staticflickr.com/1/328/31922043426_9cc1d85992_c.jpg)
](https://www.elastic.co/products/logstash)
