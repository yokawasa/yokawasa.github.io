---
author: Yoichi Kawasaki
categories:
- Azure
date: "2017-12-29T17:13:02Z"
date_gmt: 2017-12-29 08:13:02 +0900
published: true
status: publish
tags:
- Python
- AzureFunctions
- Serverless
title: Azure Functions Python Programming  - Experimental
---

今年もあと少し。ほぼ趣味の範囲を超えないレベルで今年取り組んだテーマの１つにAzure Functions with Pythonがある。あまり情報が無い中、興味本位でサンプルコードを作っては動かして試して得られた情報をシコシコとGithubに上げているうちにナレッジが溜まって来た。それほど多くはないと思うがPythonでAzure Functionsアプリを作りたいという人もいると思うのでノウハウをブログにまとめておく。いきなり水を差すようではあるが、現時点（2017年12月）ではAzure FunctionsのPythonサポータビリティはExperimental（実験的サポート）でありプロダクション向きではない状況であるので、ホントにPythonが好きな人がOn your own riskで楽しんでいただければと思う。

## Azure FunctionsのPythonサポート状況

Azure FunctionsのRuntimeには大きく1系と２系の２種類あるが、現時点でPythonは1系でのみExperimentalサポートという状況（ See also [言語サポート状況](https://docs.microsoft.com/en-us/azure/azure-functions/supported-languages)）

![azure-functions-runtime-version](https://farm5.staticflickr.com/4638/38482890335_8c5083192d_z.jpg)

Experimental（実験的サポート）なので本番での利用は非推奨であり、公式サポートはない（ベストエフォートでのサポートは得られるはず）。また、当然ながらGA言語に比べパフォーマンスは悪い。PythonはFunction呼び出し毎にpython.exeが実行される（GA言語はRuntimeと同じプロセスで実行）。

将来的な話をすると、Azure Functions Runtime 1系でのPythonサポートについては今のExperimentalの域を超えることはないだろう。一方、Runtime 2系ではPythonが正式サポートされるように対応が進められている。ただし時期は未定。この対応については下記Github Issueが切られており、ある程度の対応状況であれば確認可能。Pythonを使う利点の１つに、強力な数理計算、自然言語解析、機械学習系モジュールがあるが、早く安定とパフォーマンスが備わったPythonサーバレスアプリ実行環境でこれら強力なモジュールを活用できたらと思うのは私だけではないだろう。今後の進展に期待。

- [Feature planning: first class Python support](https://github.com/Azure/azure-webjobs-sdk-script/issues/335)

## Hosting Planの選択について

### Consumption Plan vs App Service Plan

Azure FunctionsのHosting PlanにはConsumption PlanとApp Service Planの2つがあって、言語に関係なく各プランの特徴は次の通り:

**Consumption Plan**

- コード実行時にコンピューティング割り当て
- リソース使用量（関数実行時間、使用メモリ）で課金
- 自動スケール、各処理は〜10分まで

**App Service Plan**

- 専用VMでリソース確保
- 継続処理：10分以上の処理
- App Service環境でのみ可能な処理: App Service Environment, VNET/VPN接続, より大きなサイズのVM, etc

### Pythonで使う上で気をつけるポイント

- Python 3.XなどRuntimeの変更を行う場合は、専用環境である必要があってApp Service Plan必須
- Consumption Planの場合、Pythonに限らずColdスタート問題という休眠したFunctionの起動が極端に遅くなる問題があるのだが、Pythonの場合は、GA言語に比べてパフォーマンスが悪く、SciPyなど重めのモジュールを利用すると絶望的に遅くなることからConsumption Planでの問題が特に顕著にでてくる。これまでの経験から、小さいインスタンスを並べるConsumption Planよりも比較的大きなサイズのVMが選べるApp Service Planの方が向いていることが多い。Pythonの場合は、予測可能なワークロードに対してApp Service Planで使うほうが問題が少ない。Consumption Planの魅力であるMicro Billing（使った分だけ課金）やリクエストに応じたオートスケーリングといった真のサーバレスに期待される要件は既に正式サポートしているC#、Nodeでやっていただくのがよいかと。

### [参考] Coldスタート問題

- Consumption Planにおける問題
- [Azure Functions Cold Start Workaround](https://blog.kloud.com.au/2017/11/04/azure-functions-cold-start-workaround/)
- The only downside is that the consumption model that keeps the cost so dirt-cheap means that unless you are using your Function constantly (in which case, you might be better off with the non-consumption options anyway), you will often be hit with a long delay as your Function wakes up from hibernation
- 休眠したFunctionをどう起こすかがポイント。事前に空リクエストを送ることが考えられるが問題はタイミング（フォーム開いた時とか）

## Python 3.Xランタイムへの変更方法

2017年12月時点のAzure FunctionsのデフォルトPython Runtimeは2.7.8である。Site ExtensionにPython3.5系とPython3.6系が用意されているので、それを利用してFunctionsで利用するPython Runtimeを変更する方法を下記ページに纏めた。

- [How to change the Python version used in a Function App](https://github.com/yokawasa/azure-functions-python-samples/blob/master/docs/custom-python-version.md)

## モジュールのインストール方法

pipとKudu DebugConsole/UIを利用した２種類のモジュールインストール方法を下記ページに纏めた。

- [How to install Python modules](https://github.com/yokawasa/azure-functions-python-samples/blob/master/docs/install-python-modules.md)

## Pythonサンプルコード

私が試したTriggerとBinding利用サンプルコードは全て下記Githubプロジェクトに追加するようにしている。もしこれを読んでいる皆さんで下記プロジェクトでカバーされていないTrigger/Bindingの組み合わせを試されたら是非ともコントリビュートください。

- [https://github.com/yokawasa/azure-functions-python-samples](https://github.com/yokawasa/azure-functions-python-samples)

## スライドとHands-onマテリアル

[Azure Antenna](https://azure.connpass.com/)にて[2017年11月20日](https://azure.connpass.com/event/72312/)と[11月28日](https://azure.connpass.com/event/72125/)に実施したセッション資料:

[![](https://image.slidesharecdn.com/serverlessappdevelopmentpythonja-171128085250/95/pythonazure-serverless-application-development-with-python-1-1024.jpg)](//www.slideshare.net/yokawasa/pythonazure-serverless-application-development-with-python-82884446)
**[PythonによるAzureサーバレスアプリケーション開発 / Serverless Application Development with Python](//www.slideshare.net/yokawasa/pythonazure-serverless-application-development-with-python-82884446)** from **[Yoichi Kawasaki](https://www.slideshare.net/yokawasa)**

Hands-onマテリアル：

- [Hands-on: Serverless Application Development with Python](https://github.com/yokawasa/azure-functions-python-samples/tree/master/handson)

それでは、Enjoy Serverless Application Development with Python!

### [追記] 上記セッションに関する記事
- [潜入レポート>> Pythonを使ったAzureサーバレスアプリケーション開発](https://japan.zdnet.com/extra/azure_antenna_201712/35111765/)
- [渋谷ヒカリエ Azure Antenna ハンズオン 参加レポート](https://zine.qiita.com/event-report/azure-antenna-report/)
