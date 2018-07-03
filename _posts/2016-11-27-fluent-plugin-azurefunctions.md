---
layout: post
status: publish
published: true
title: Collecting events into Azure Functions and triggering your custom code using
  fluent-plugin-azurefunctions
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 101
wordpress_url: http://unofficialism.info/posts/?p=101
date: '2016-11-27 08:27:46 +0900'
date_gmt: '2016-11-26 23:27:46 +0900'
categories:
- Azure
- English
tags:
- fluentd
- Ruby
- AzureFunctions
- fluent-plugin-azurefunctions
---

In this article, I&rsquo;d like to introduces a solution to collect events from various sources and send them into HTTP Trigger function in Azure Functions using [fluent-plugin-azurefunctions](https://github.com/yokawasa/fluent-plugin-azurefunctions). Triggers in Azure Functions are event responses used to trigger your custom code. [HTTP Trigger functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook) allow you to respond to HTTP events sent from fluentd and cook them into whatever you want! 

![fluent-plugin-azurefunctions](https://c6.staticflickr.com/6/5747/31080973501_83e854eb4a_c.jpg)

[note] [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview) is a ("serverless") solution for easily running small pieces of code, or "functions," in Azure. [Fluentd](http://www.fluentd.org/) is an open source data collector, which lets you unify the data collection and consumption for a better use and understanding of data. [fluent-plugin-azurefunctions](https://github.com/yokawasa/fluent-plugin-azurefunctions) is a fluentd output plugin that enables to collect events into Azure Functions.

## Pre-requisites

- A basic understanding of fluentd - if you're not familiar with fluentd, [fluentd quickstart guide](http://docs.fluentd.org/articles/quickstart) is good starting point
- Azure subscription - you need to have Azure subscription that grants you access to Microsoft Azure services, and under which you can create Azure Functions account. If you don't have yet click [here](https://account.windowsazure.com/Subscriptions) to create it

## Setup: Azure Functions (HTTP Trigger Function) 

Create a function (HTTP Trigger). First, you need to have an function app that hosts the execution of your functions in Azure if you don't already have. Once you have an function app, you can create a function. Here are instructions:

- [Create your first Azure Function](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function)
- [Azure Functions developer reference](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference)

A quick-start HTTP trigger function sample is included under [examples/function-csharp](https://github.com/yokawasa/fluent-plugin-azurefunctions/tree/master/examples/function-csharp) in Github repository. You simply need to save the code ([run.csx](https://github.com/yokawasa/fluent-plugin-azurefunctions/blob/master/examples/function-csharp/run.csx)) and configuration files ([function.json](https://github.com/yokawasa/fluent-plugin-azurefunctions/blob/master/examples/function-csharp/function.json), [project.json](https://github.com/yokawasa/fluent-plugin-azurefunctions/blob/master/examples/function-csharp/project.json)) in the same Azure function folder. Explaining a little bit about each of files, the **function.json** file defines the function bindings and other configuration settings. The runtime uses this file to determine the events to monitor and how to pass data into and return data from function execution. The **project.json** defines packages that the application depends. The **run.csx** is a core application file where you write your code to process Your jobs. Here is a sample run.csx:

```csharp
using System.Net;
using Newtonsoft.Json;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    log.Info("C# HTTP trigger function to process fluentd output request.");
    log.Info( string.Format("Dump request:\n {0}",req.ToString()));
    // parse query parameter
    string payload = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "payload", true) == 0)
        .Value;
    // Get request body
    dynamic data = await req.Content.ReadAsAsync<object>();
    if (data.payload == null) {
        log.Info("Please pass a payload on the query string or in the request body");
        return new HttpResponseMessage(HttpStatusCode.BadRequest);
    }
    // Process Your Jobs!
    dynamic r = JsonConvert.DeserializeObject<dynamic>((string)data.payload);
    if (r.key1!=null) log.Info(string.Format("key1={0}",r.key1));
    if (r.key2!=null) log.Info(string.Format("key2={0}",r.key2));
    if (r.key3!=null) log.Info(string.Format("key3={0}",r.key3));
    if (r.mytime!=null) log.Info(string.Format("mytime={0}",r.mytime));
    if (r.mytag!=null) log.Info(string.Format("mytag={0}",r.mytag));
    return new HttpResponseMessage(HttpStatusCode.OK);
}
```

## Setup: Fluentd

First of all, install Fluentd. The following shows how to install Fluentd using Ruby gem packger but if you are not using Ruby Gem for the installation, please refer to [this installation guide](http://docs.fluentd.org/categories/installation) where you can find many other ways to install Fluentd on many platforms.

```sh
# install fluentd
$ sudo gem install fluentd --no-ri --no-rdoc

# create fluent.conf
$ fluentd --setup 
```

Also, install [fluent-plugin-azurefunctions](https://github.com/yokawasa/fluent-plugin-azurefunctions) for fluentd aggregator to send collected event data into Azure Functions.

```sh
$ sudo gem install fluent-plugin-azurefunctions
```

Next, configure fluent.conf, a fluentd configuration file as follows. Please refer to [this](https://github.com/yokawasa/fluent-plugin-azurefunctions#fluentd---fluentconf) for fluent-plugin-azurefunctions configuration. The following is a sample configuration where the plugin writes only records that are specified by key_names in incoming event stream out to Azure Functions:

```xml
# This is used by event forwarding and the fluent-cat command
<source>
    @type forward
    @id forward_input
</source>

# Send Data to Azure Functions
<match azurefunctions.**>
    @type azurefunctions
    endpoint  AZURE_FUNCTION_ENDPOINT   # ex. https://<accountname>.azurewebsites.net/api/<functionname>
    function_key AZURE_FUNCTION_KEY     # ex. aRVQ7Lj0vzDhY0JBYF8gpxYyEBxLwhO51JSC7X5dZFbTvROs7uNg==
    key_names key1,key2,key3
    add_time_field true
    time_field_name mytime
    time_format %s
    localtime true
    add_tag_field true
    tag_field_name mytag
</match>
```

[note] If **key_names** not specified above, all incoming records are posted to Azure Functions (See also [this](https://github.com/yokawasa/fluent-plugin-azurefunctions#fluentd---fluentconf)).

Finally, run fluentd with the fluent.conf that you configure above.

```sh
$ fluentd -c ./fluent.conf -vv &
```

## TEST

Let's check if test events will be sent to Azure Functions that triggers the HTTP function (let's use [the sample function](https://github.com/yokawasa/fluent-plugin-azurefunctions/tree/master/examples/function-csharp) included in Github repo this time). First, generate test events using fluent-cat like this:

```sh
echo ' { "key1":"value1", "key2":"value2", "key3":"value3"}' | fluent-cat azurefunctions.msg
```

As both **add_time_field** and **add_tag_field** are enabled, time and tag fields are added to the record that are selected by **key_names** before posting to Azure Functions, thus actual HTTP Post request body would be like this:

```json
{
    "payload": '{"key1":"value1", "key2":"value2", "key3":"value3", "mytime":"1480195100", "mytag":"azurefunctions.msg"}'
}
```

If events are sent to the function successfully, a HTTP trigger function handles the events and the following logs can be seen in Azure Functions log stream: 

```
2016-11-26T21:18:55.200 Function started (Id=5392e7ae-3b8e-4f65-9fc1-6ae529cdfe3a)
2016-11-26T21:18:55.200 C# HTTP trigger function to process fluentd output request.
2016-11-26T21:18:55.200 key1=value1
2016-11-26T21:18:55.200 key2=value2
2016-11-26T21:18:55.200 key3=value3
2016-11-26T21:18:55.200 mytime=1480195100
2016-11-26T21:18:55.200 mytag=azurefunctions.msg
2016-11-26T21:18:55.200 Function completed (Success, Id=5392e7ae-3b8e-4f65-9fc1-6ae529cdfe3a)
```

## Advanced Senarios

### 1. Near Real-time processing

Function Apps can output messages to different means or data stores. For example, fluentd collects events generated from IoT devices and send them to Azure Function, and the the HTTP trigger function transforms the events and processes the data to store in a persistent storage or to pass them to different means. Here are some of options available at the time of writing: 

- [Store JSON documents on DocumentDB](https://github.com/Azure/azure-webjobs-sdk-extensions)
- [Send events to Event Hub](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-hubs)
- [Send messages to Azure Service Bus Queues](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-service-bus)
- [Send messages to Azure Storage Queues](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue)
- [Store blobs to Azure Blob Storage](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob)
- [Push notifications to Notification Hub](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-notification-hubs)
- [[Send SMS text messages via Twilio](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-twilio)](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-twilio)
- [Send emails via SendGrid](https://sendgrid.com/)

### 2. Background jobs processing

If the jobs are expected to be large long running ones, it's recommended that you refactor them into smaller function sets that work together and return fast responses. For example, you can pass the HTTP trigger payload into a queue to be processed by a queue trigger function. Or if the payload is too big to pass into the queue, you can store them onto Azure Blob storage at first, then pass only limited amount of the data into a queue just to trigger background workers to process the actual work. These approaches allow you to do the actual work asynchronously and return an immediate response. 

## LINKS

- [fluent-plugin-azurefunctions](https://github.com/yokawasa/fluent-plugin-azurefunctions)
- [Azure Functions Overview](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview)
- [Azure functions Triggers and Bindings](https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings)
- [Azure Functions Best Practices](https://docs.microsoft.com/en-us/azure/azure-functions/functions-best-practices)

END
