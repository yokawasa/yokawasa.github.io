---
layout: post
status: publish
published: true
title: C Sharpで実行ファイルにDLLをスタティックリンクさせたい
author:
  display_name: Yoichi Kawasaki
  url: http://github.com/yokawasa
author_login: yoichi
author_email: yokawasa@gmail.com
author_url: http://github.com/yokawasa
wordpress_id: 45
wordpress_url: http://unofficialism.info/posts/?p=45
date: '2015-09-09 00:12:47 +0900'
date_gmt: '2015-09-08 15:12:47 +0900'
categories:
- TIPS
tags:
- CSharp
- VisualStduio
- ILMerge
---

**Q1. C Sharpでexeに複数DLLをスタティックリンクさせて１ファイルにすることはできますか？**

C、C++ではおなじみのスタティックリンクだが、そもそもC Sharpではスタティックリンクができない。ただし

MS Research謹製の[ILMerge](http://www.microsoft.com/en-us/download/details.aspx?id=17630)を使えば実行ファイル(exe)に対して複数のクラスアセンブリ(DLL)を１つのアセンブリにマージすることはできる。使い方は[こちら](http://www.atmarkit.co.jp/fdotnet/dotnettips/426ilmerge/ilmerge.html)が参考になる。また、[ILMerge-GUI](https://ilmergegui.codeplex.com/)というGUIツールもある。

**Q2. Visual Studioのビルドでも自動的に複数ファイルを１つにまとめることはできますか？**

パッケージマネージャーでILMerge.MSBuild.Tasksをインストールして*.csprojファイルに自動アセンブリ生成するための設定を追記するとVSのビルドで複数ファイルが１つのアセンブリにマージされたファイルが自動的にできあがる。参考: StackOverflow 「[How to Integrate ILMerge into Visual Studio Build Process to Merge Assemblies?](http://stackoverflow.com/questions/2556048/how-to-integrate-ilmerge-into-visual-studio-build-process-to-merge-assemblies)」

1) ILMerge.MSBuild.Tasksをインストール
```powershell
Install-Package ILMerge.MSBuild.Tasks
```

2) *.csprojファイルに下記設定を追記

```xml
  <!-- Code to merge the assemblies into one -->
  <UsingTask TaskName="ILMerge.MSBuild.Tasks.ILMerge" AssemblyFile="$(SolutionDir)\packages\ILMerge.MSBuild.Tasks.1.0.0.3\tools\ILMerge.MSBuild.Tasks.dll" />
  <Target Name="AfterBuild">
    <ItemGroup>
      <MergeAsm Include="$(OutputPath)$(TargetFileName)" />
      <MergeAsm Include="$(OutputPath)追加するDLLファイル名1" />
      <MergeAsm Include="$(OutputPath)追加するDLLファイル名2" />
       ...
      <MergeAsm Include="$(OutputPath)追加するDLLファイル名N" />
    </ItemGroup>
    <PropertyGroup>
      <MergedAssembly>出力する結果アセンブリファイル名（フルパス）</MergedAssembly>
    </PropertyGroup>
    <Message Text="ILMerge @(MergeAsm) -&gt; $(MergedAssembly)" Importance="high" />
    <ILMerge InputAssemblies="@(MergeAsm)" OutputFile="$(MergedAssembly)" TargetKind="SameAsPrimaryAssembly" />
  </Target>
</Project>
```

3) VSビルド実行で、マージされたファイルが生成（*.csprofのMergedAssemblyで指定したファイル名）

尚、[前記事](http://unofficialism.info/posts/azure-media-services-get-all-assets-list/)のコンソールアプリの[サンプルコード](https://github.com/yokawasa/azure-samples/tree/master/ams-list-assets)では&lsquo;*.csprojファイルに[この設定](https://github.com/yokawasa/azure-samples/blob/master/ams-list-assets/ams-list-assets/ams-list-assets.csproj)を追加しており、VSビルドで自動的に1アセンブリにマージされたexeファイルが生成されるようにしている。
