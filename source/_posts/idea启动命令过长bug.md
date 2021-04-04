---
title: idea启动命令过长bug
date: 2020-08-23 11:37:26
tags: bug
---

# idea启动命令过长bug
> 在引用 MeteoInfoLib jar 包后带来的问题，报错内容为：
> Error running 'Application': Command line is too long. Shorten command line for Application or also for Spring Boot default configuration.
> 
<!--more-->

## 解决方案
调整运行项目的配置，将 Configuration 中的 Shorten Command Line 修改为 JAR 就可以了。
具体原因就是和 bug 所说的一样，启动的命令行过长，idea 通过临时的 classpath.jar 传递长的类路径。原始类路径在MANIFEST.MF中定义为classpath.jar中的类路径属性。就完事了。