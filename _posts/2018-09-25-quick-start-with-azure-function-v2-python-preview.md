---
layout: post
status: publish
published: true
title: Quick Start with Azure Functions V2 Python (Preview)
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
date: '2018-09-25 19:32:40 +0900'
tags:
- Azure Functions
- Python
---

Today (Sept 25, 2018 JST), Azure Functions supports Python development using Python 3.6 on the Functions v2 (cross-platform) runtime. You can now use your Python code and dependencies on Linux-based Functions. This is an article on quick start with Azure Functions V2 Python (Preview) showing how you can quickly start Python function development on Azure Function V2 runtime.

## 1. Prerequisites for Buidling & Testing Locally
- **Python 3.6** (For Python function apps, you have to be running in a `venv`)
- [Azure Functions Core Tools](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local) **2.0.3 or later**

### Prerequisite 1 - Python version
This is how you switch your python version using `venv`. 

First of all, install [Python 3.6](https://www.python.org/downloads/) if you don't have on your local environemnt. 

```sh
$ python3.6 -V
Python 3.6.1
```
Then, to build and test Python function with `Azure Functions Core Tools` (you'll install after this), you need to setup a Python 3.6 virtual environemnt using `venv` (NOT, pyenv or some other virtual env tools).  

```sh
$ python -V
Python 2.7.1

# create and activate virutal environemnt named "venv3.6"
$ python3.6 -m venv venv3.6
$ source venv3.6/bin/activate

$ Python -V
Python 3.6.1
```
> [NOTE] As virtualenv (venv) was included in Python3.3, you don't need to install venv separately. It's already in your Python 3.6


### Prerequisite 2 - Azure Functions Core Tools

You need to install `azure-functions-core-tools` version `2.0.3` or later for buiding & testing Python functions on Azure Functions V2 runtime. 

```sh
# Check current package version (only if you already have azure-functions-core-tools installed)
$ func --version
2.0.1-beta.35

$ npm list -g  --depth=0 |grep azure-functions-core-tools
├── azure-functions-core-tools@2.0.1-beta.35

# Check available package versions
$ npm info azure-functions-core-tools versions
  '2.0.1-beta.31',
  '2.0.1-beta.32',
  '2.0.1-beta.33',
  '2.0.1-beta.34',
  '2.0.1-beta.35',
  '2.0.1-beta.37',
  '2.0.1-beta.38',
  '2.0.1-beta.39',
  '2.0.1-beta.23-1',
  '2.0.1-beta1.15',
  '2.0.2',
  '2.0.3' ]

# Install speicific package version - 2.0.3
$ npm install -g azure-functions-core-tools@2.0.3

$ func --version
2.0.3
```

## 2. Create Python Function and run locally

### 2-1. Create Python Functions Project
Create a local Python Functions project - `v2projects`

```sh
$ func init v2projects --worker-runtime python

Installing wheel package
Installing azure-functions==1.0.0a4 package
Installing azure-functions-worker==1.0.0a4 package
Running pip freeze
Writing .gitignore
Writing host.json
Writing local.settings.json
Writing /Users/yoichika/dev/github/azure-functions-python-samples/v2projects/.vscode/extensions.json
```

A new folder named `v2projects` is created. Then, change directory to this folder
```sh
$ cd v2projects
```

### 2-2. Create a python function

Check function template list
```sh
$ func templates list

C# Templates:
  Blob trigger
  Cosmos DB trigger
  Durable Functions activity
  Durable Functions HTTP starter
  Durable Functions orchestrator
  Event Grid trigger
  Event Hub trigger
  HTTP trigger
  Outlook message webhook creator
  Outlook message webhook deleter
  Outlook message webhook handler
  Outlook message webhook refresher
  Microsoft Graph profile photo API
  Queue trigger
  SendGrid
  Service Bus Queue trigger
  Service Bus Topic trigger
  Timer trigger

JavaScript Templates:
  Blob trigger
  Cosmos DB trigger
  Event Grid trigger
  HTTP trigger
  Queue trigger
  SendGrid
  Service Bus Queue trigger
  Service Bus Topic trigger
  Timer trigger

Python Templates:
  Blob trigger
  Cosmos DB trigger
  Event Grid trigger
  Event Hub trigger
  HTTP trigger
  Queue trigger
  Service Bus Queue trigger
  Service Bus Topic trigger
  Timer trigger

```

Create a python funciton with "HTTP trigger" template
```
$ func new --language Python --template "HTTP trigger" --name HttpTriggerPython

Select a template: HTTP trigger
Function name: [HttpTriggerPython] Writing /Users/yoichika/dev/github/azure-functions-python-samples/v2projects/HttpTriggerPython/sample.dat
Writing /Users/yoichika/dev/github/azure-functions-python-samples/v2projects/HttpTriggerPython/__init__.py
Writing /Users/yoichika/dev/github/azure-functions-python-samples/v2projects/HttpTriggerPython/function.json
The function "HttpTriggerPython" was created successfully from the "HTTP trigger" template.
```
### 2-3. Local Testing
Run the Functions host locally.

```
$ func host start
```

![]({{ site.url }}/assets/20180925-azfunc-v2-python-host-start.png)


Then, send a test requst to the given endpoint for the HttpTrigger function

```
$ curl http://localhost:7071/api/HttpTriggerPython?name=AzureFunctionV2

Hello AzureFunctionV2!
```

## 3. Create Linux FunctionApps (Preview Consumption App)

Now you create Functions App to deploy and run your Python functions on Azure. You actually need Linux V2 Function Apps as publishing Python functions is only supported for Linux Function Apps. So let's create the Azure Functions Linux Consumption App (which is in preview as of today Sept  25, 2018 JST).

### 3-1. Whitelist request for accessing the Azure Functions Linux Consumption (preview)

As guided in [Azure Functions on Linux Preview](https://github.com/Azure/Azure-Functions/wiki/Azure-Functions-on-Linux-Preview), you need to email linuxazurefunctions at Microsoft dot com with your Azure subscription ID to get your ID whitelisted to access the Azure Functions Linux Consumption (Preview).
[NOTE] This is only needed during its preview preriod. 

### 3-2. Install the Azure CLI extension for the Azure Functions Linux Consumption preview

As a prerequisite, you need to install the Azure CLI extension for the Azure Functions Linux Consumption preview. Here is how you do:

```sh
$ curl "https://functionscdn.azureedge.net/public/docs/functionapp-0.0.1-py2.py3-none-any.whl" \
 -o functionapp-0.0.1-py2.py3-none-any.whl
$ az extension add --source functionapp-0.0.1-py2.py3-none-any.whl
```
For more detail, see [this doc](https://github.com/Azure/Azure-Functions/wiki/Azure-Functions-on-Linux-Preview#prerequisites)

### 3-3. Create a resource group

```sh
RESOURCE_GROUP="RG-azfuncv2"
REGION="eastus"
az group create --name $RESOURCE_GROUP --location $REGION
```

### 3-4. Create an Azure Storage Account

```sh
STORAGE_ACCOUNT="azfuncv2linuxstore"
az storage account create --name $STORAGE_ACCOUNT \
    --location $REGION \
    --resource-group $RESOURCE_GROUP \
    --sku Standard_LRS
```

### 3-5. Create a Empty Function App on Linux (Consumption Plan)

Give your function app name to `APP_NAME` variable, and execute az command like this.

```sh
APP_NAME="yoichikaazfuncv2linux001"
az functionapp createpreviewapp --name $APP_NAME \
    --resource-group $RESOURCE_GROUP \
    --storage-account $STORAGE_ACCOUNT \
    -l $REGION \
    --runtime python \
    --is-linux

(output)
Your Linux, cosumption plan, function app 'yoichikaazfuncv2linux001' has been successfully created but is not active until content is published usingAzure Portal or the Functions Core Tools.
```
> [NOTE] The Function App name needs to be unique across all apps in Azure.

Now you're ready to publish your functions to the App!

## 4. Publish the Python Function

You publish the Python functions to the function app ( named `yoichikaazfuncv2linux001` here )

```sh
# func azure functionapp publish <app_name>
$ func azure functionapp publish yoichikaazfuncv2linux001 --force

Getting site publishing info...
pip download -r /Users/yoichika/dev/github/azure-functions-python-samples/v2projects/requirements.txt --dest /var/folders/mn/xw3hp_854lvb5yr2m_00s7sm0000gn/T/azureworkerry44_fwz
pip download --no-deps --only-binary :all: --platform manylinux1_x86_64 --python-version 36 --implementation cp --abi cp36m --dest /var/folders/mn/xw3hp_854lvb5yr2m_00s7sm0000gn/T/azureworkerpcdk1t4g azure_functions_worker==1.0.0a4
pip download --no-deps --only-binary :all: --platform manylinux1_x86_64 --python-version 36 --implementation cp --abi cp36m --dest /var/folders/mn/xw3hp_854lvb5yr2m_00s7sm0000gn/T/azureworkerpcdk1t4g azure_functions==1.0.0a4
pip download --no-deps --only-binary :all: --platform manylinux1_x86_64 --python-version 36 --implementation cp --abi cp36m --dest /var/folders/mn/xw3hp_854lvb5yr2m_00s7sm0000gn/T/azureworkerpcdk1t4g protobuf==3.6.1
pip download --no-deps --only-binary :all: --platform manylinux1_x86_64 --python-version 36 --implementation cp --abi cp36m --dest /var/folders/mn/xw3hp_854lvb5yr2m_00s7sm0000gn/T/azureworkerpcdk1t4g grpcio_tools==1.14.2
pip download --no-deps --only-binary :all: --platform manylinux1_x86_64 --python-version 36 --implementation cp --abi cp36m --dest /var/folders/mn/xw3hp_854lvb5yr2m_00s7sm0000gn/T/azureworkerpcdk1t4g setuptools==40.4.3
pip download --no-deps --only-binary :all: --platform manylinux1_x86_64 --python-version 36 --implementation cp --abi cp36m --dest /var/folders/mn/xw3hp_854lvb5yr2m_00s7sm0000gn/T/azureworkerpcdk1t4g grpcio==1.14.2
pip download --no-deps --only-binary :all: --platform manylinux1_x86_64 --python-version 36 --implementation cp --abi cp36m --dest /var/folders/mn/xw3hp_854lvb5yr2m_00s7sm0000gn/T/azureworkerpcdk1t4g six==1.11.0

Preparing archive...
Uploading content...
Upload completed successfully.
Deployment completed successfully.
Removing 'WEBSITE_CONTENTSHARE' from 'yoichikaazfuncv2linux001'
Removing 'WEBSITE_CONTENTAZUREFILECONNECTIONSTRING' from 'yoichikaazfuncv2linux001'
Syncing triggers...
```
> [NOTE] 
>  1. if you run `func azure functionapp publish` without `--force` opiton, you'll come up with the following the message:
>  ```
>  Your app is configured with Azure Files for editing from Azure Portal.
>  To force publish use --force. This will remove Azure Files from your app.
>  ```
>  2. Please be noted that when deployging from `func` command( ie, running from a package), the file system in Azure Function App is read-only and no changes can be made to the files. To make any changes update the content in your zip file and WEBSITE_RUN_FROM_PACKAGE app setting.

Looks like pulishing has completed successfully, so let's access to the function on Azure

```sh
$ curl https://yoichikaazfuncv2linux001.azurewebsites.net/api/HttpTriggerPython?name=AzureFunctionV2

Hello AzureFunctionV2!
```

Enjoy Azure Functions V2 Python!

## LINKS
- [azure-functions-python-worker wiki: Create your first Python function](https://github.com/Azure/azure-functions-python-worker/wiki/Create-your-first-Python-function)
- [Azure Functions on Linux Preview](https://github.com/Azure/Azure-Functions/wiki/Azure-Functions-on-Linux-Preview)
- [Create your first function running on Linux using the Azure CLI (preview)](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function-azure-cli-linux#create-an-azure-storage-account)

