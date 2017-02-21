---
layout: post
title: "iOS架构设计(解耦的尝试)之使用DSL代码生成"
date: 2017-01-13T14:54:23+08:00
categories: iOS
---


iOS架构设计(解耦的尝试)之使用DSL代码生成

> 该系列文章是2016年折腾的一个总结，对于这一年中思考和解决的一些问题做一些梳理和总结

继续我们偷懒的话题，这一次我们探讨另外一种用于节省开发时间的策略：代码生成。一般情况下，提到代码生成就会顺便提到DSL（domain special luanguage）领域特定语言：聚焦一个特定的领域，极易读懂，功能很少，异常简洁。我们通过DSL来降低业务问题的复杂性，通过简化模型等手段提高开发效率等。当然领域特定语言是个很宏大的命题，我们此处所涉及到的代码生成也只是“管中窥豹”，仅仅是一个比较常见的用法。举个例子来讲：[Objective-C模型生成工具Chameleon](https://github.com/yishuiliunian/Chameleon)。去年开发了一个简单的从描述文法生成Objective-C的工具。下面展示的是这个描述文法的示例：

``` 
/*测试协议*/
model Test{
  char                bwchar
  short               bwshort
  array    UserInfo bwobjarray    [bw_objarray_sname]
}
```

运行工具Chameleon，则将其转化成对应的Objective-C模型类：

