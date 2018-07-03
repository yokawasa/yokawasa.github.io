---
layout: post
status: publish
published: true
title: embulk plugins for Microsoft Azure Services
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 88
wordpress_url: http://unofficialism.info/posts/?p=88
date: '2016-09-01 06:05:05 +0900'
date_gmt: '2016-08-31 21:05:05 +0900'
categories:
- Azure
- English
tags:
- embulk
---

Here is a list of embulk plugins that you can leverage to transfer your data between Microsoft Azure Services and various other databases/storages/cloud services.

| Plugin Name   | Target Azure Services | Note | 
| ------------- | ------------- | ------------- |
| <a href="https://github.com/sakama/embulk-output-azure_blob_storage">embulk-output-azure_blob_storage</a> |	Blob Storage | Embulk output plugin that stores files onto Microsoft Azure Blob Storage |
| <a href="https://github.com/sakama/embulk-input-azure_blob_storage">embulk-input-azure_blob_storage</a> |	Blob Storage | Embulk input plugin that reads files stored on Microsoft Azure Blob Storage |
| <a href="https://github.com/embulk/embulk-output-jdbc/tree/master/embulk-output-sqlserver">embulk-output-sqlserver</a> |	SQL Databases, SQL DWH | Embulk output plugin that Inserts or updates records to SQL server type of services like SQL DB/SQL DWH |
| <a href="https://github.com/embulk/embulk-input-jdbc/tree/master/embulk-input-sqlserver">embulk-input-sqlserver</a> |	SQL Databases, SQL DWH | Embulk input plugin that selects records from SQL type of services like SQL DB/SQL DWH |
| <a href="https://github.com/yokawasa/embulk-output-documentdb">embulk-output-documentdb</a> |	Comos DB | Embulk output plugin that dumps records to Azure Cosmos DB |
| <a href="https://github.com/yokawasa/embulk-output-azuresearch">embulk-output-azuresearch</a> |	Azure Search | Embulk output plugin that dumps records to Azure Search |

(as of Aug 30, 2016)

For embulk, check this site: [https://github.com/embulk/embulk](https://github.com/embulk/embulk)

[![embulk-screenshot](https://c1.staticflickr.com/9/8283/29081871880_b9a4338fbf_b.jpg)
](https://github.com/embulk/embulk)