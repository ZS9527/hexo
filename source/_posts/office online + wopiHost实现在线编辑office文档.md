---
title: office online + wopiHost实现在线编辑office文档
date: 2020-04-11 16:00:40
tags: office
---

# office online + wopiHost实现在线编辑office文档
> 跟着博客一步一步走完成的office online server的安装，在此记录一下碰到的问题与解决思路。
> [https://www.netnr.com/home/list/114](https://www.netnr.com/home/list/114) wopi参考文档。
> [https://www.netnr.com/doc/code/4964095842855914510/4688957725858114675](https://www.netnr.com/doc/code/4964095842855914510/4688957725858114675) 配合上面的项目的文档
> [https://blog.csdn.net/jiaqu2177/article/details/81945692](https://blog.csdn.net/jiaqu2177/article/details/81945692) 安装office online server参考文档

<!--more-->

## 问题汇总
按照步骤成功的安装完毕，但是还是会遇到一些文档里没有说的问题。
### 问题一：
文档中的第九步一开始看没有明白是什么意思。请教大佬后，通过服务器管理器 -> 工具 -> （iis）管理器 -> 添加了网站。成功获得文件信息。
![添加网站信息-配合netnr/WopiHost使用](b1.png)

### 问题二：
无法编辑word文件，重启一下服务器后可以进入编辑url

![报错信息](b2.png)

![powershell命令信息](b3.png)
### 问题三：
无法编辑excel文件，是因为文件有宏命令，或者是文件中包含一些不能显示的类型。看来这个线上的office还是很有局限性。

![有宏命令](b4.png)

![发票格式excel](b5.png)
### 问题四：
用office online打开从doc格式转成docx格式的文件会开启兼容模式。提示是 [兼容模式，功能收到限制](https://support.office.com/zh-CN/client/results?Shownav=true&lcid=2052&ns=WDWAEndUser&version=15&omkt=zh-CN&ver=15&apps=WDWAENDUSER%2CXLWAENDUSER%2CPPWAENDUSER%2CONWAENDUSER&HelpID=CompatMode)

### 问题五：
有文字框的doc文档格式错乱，不能正常的显示。添加otf字体后依旧是格式错乱。参考微云后发现文字框被转换成了图片，无法修改其中的内容。并未有解决的方法。