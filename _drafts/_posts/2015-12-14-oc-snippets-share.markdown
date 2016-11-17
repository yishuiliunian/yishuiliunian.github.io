---
layout: post
title: "代码片段共享"
date: 2015-12-14T18:45:22+08:00
categories: iOS
---

代码片段共享方案

##安装ACCodeSnippetRepository Xcode Plugin

具体方法参见https://github.com/acoomans/ACCodeSnippetRepositoryPlugin

并将ACCodeSnippetRepository的远端地址设置为：

http://gitlab.baidu.com/dongzhao02/Wallet-Snippets-Share.git

使用的gitlab服务器，已经为大家添加了权限。目前内部有一些我之前写的snippet的，后面大家需要共享的snippet或者代码模板可以往这里面放。


##创建一个snippet

每一个snippet都会以一个单独的文件存在，比如我要创建一个async的snippet则创建一个async.m的文件。并填充其中的内容：

~~~
// dispatch_async
// Dispatch to do work in the background, and then to the main queue with the results
//
// IDECodeSnippetCompletionPrefix: dispatch_async
// IDECodeSnippetCompletionScopes: [CodeBlock]
// IDECodeSnippetIdentifier: C86E89FA-6BAE-4BC2-8A98-FD6A9755987F
// IDECodeSnippetLanguage: Xcode.SourceCodeLanguage.C
// IDECodeSnippetUserSnippet: 1
// IDECodeSnippetVersion: 2
dispatch_async(dispatch_get_global_queue(<#dispatch_queue_priority_t priority#>, <#unsigned long flags#>), ^(void) {
        <#code#>
        
        dispatch_async(dispatch_get_main_queue(), ^(void) {
            <#code#>
        });
    });
~~~


其中有几个关键的定义：

|关键字|含义|备注信息|
|:--|:--|:--|
|IDECodeSnippetCompletionPrefix|自动补全使用的关键字||
|IDECodeSnippetCompletionScopes|触发自动补全的作用域||
|IDECodeSnippetIdentifier|该Snippet唯一标识|随便填了，最好找个工具随机生成个GUID|
|IDECodeSnippetLanguage|真对的语言||
|IDECodeSnippetUserSnippet||添1|
|IDECodeSnippetVersion|snippet的版本号||


其中IDECodeSnippetCompletionScopes作用域有以下选项：

|关键字|含义|备注信息|
|:--|:--|:--|
|CodeBlock|代码里面自动补全||
|ClassImplementation|在类的实现体内||
|All|所有区域内有效||


