---
layout: post
title: "Proxy-ios"
date: 2016-02-29T15:09:17+08:00
categories: iOS
---

#iOS上进行抓包
目前绝大部分的客户端都会有网络请求发生，很多情况下我们需要去分析我们的网络请求的质量，甚至是篡改数据来调试特定的问题。真对这个问题，推荐几个Mac下面的工具。

1. wireShark, 瑞士军刀一样的工具，可以截取各种网络封包，并显示其详细信息。
2. charles, 应用层http或者https的抓包和数据篡改工具。

下面说一下这两个工具的用法。

##WireShark
wireshark是非常流行的网络封包分析软件，功能十分强大。可以截取各种网络封包，显示网络封包的详细信息。
通过[下载地址](https://www.wireshark.org/),下载mac上使用dmg文件。双击安装。

打开WireShark后我们可以看到如下的主界面：
![](http://ww4.sinaimg.cn/large/7df22103jw1f1gdc6fdwlj20uo0knaei.jpg)

红色标记的是当前机器上所有可供监听的网卡。如果要监听ios设备的网络请求，需要一个该ios设备连接的网卡。现在有两种途径可以获得：

###共享WIFI方式
在mac上设置网络共享，生成wifi热点，让ios设备连接到该热点。然后监听该WIFI网卡。



###rvictl工具



