---
layout: post
title: "解决Instruments无法找到调试符号表的问题"
date: 2015-11-11T20:30:53+08:00
categories: iOS
---

在使用Instruments中的time profile调试QQMSF的时候，发现原先有的可以定位到具体函数的功能怎么也掉不出来。Instruments只能定位到一个函数地址，没有具体的函数名。分析应该是调试符号表没有找到的问题。于是去工程设置里面找关于这个选项。有几个地方需要注意：

## 1 Debug information format
这里原先的设置是DWARF，什么是DWARF，他与熟悉的dSYM文件什么关系？查了一下。
"DWARF与dSYM的关系是，DWARF是文件格式，而dSYM往往指一个单独的文件。在Xcode中如果不做特殊制定，debug information是被保存在executable文件中，可以使用dsymutil从executable中提取dSYM文件。"

将选项调整为，DWARF with dSYM File，再次使用Instruments来profile发现能够定位的具体的函数名。改问题解决。

![](http://ww4.sinaimg.cn/large/7df22103jw1exxbpoejnhj20fk057wep.jpg)

分析可能是Instruments工具回去读取调试目标匹配的dsym文件，而当输出调试信息格式使用dwarf时，调试信息输出在了执行文件中，没有输出到dsym文件中，导致Instruments工具无法读取dsym文件，找不到符号表，结果就是无法定位函数名了。

## 2、编译优化选项
在调试的时候尽量，保持零优化的模式，这样能够保证符号表的完整性。关于具体调试选项的描述可以参考gcc的文档。

![](http://ww3.sinaimg.cn/large/7df22103jw1exxbqjncmrj20fk0b3aaq.jpg)

-----
欢迎关注iOS开发公共账号iOS_Tips：扫描下方二维码关注  
![](http://ww4.sinaimg.cn/large/7df22103jw1exx11uhhkoj20by0by3zc.jpg)
