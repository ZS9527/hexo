---
title: 由lombok引发的一次gradle版本问题
date: 2019-08-09 20:33:27
tags: bug
---

# 由lombok引发的一次gradle版本问题

在编译打包时，出现了找不到方法的错误，机智的更换了lombok版本。然后在运行时使用@Cacheable注解时就报了这个错误：**cannot deserialize from Object value (no delegate- or property-based Creator)**。
<!--more-->
## 错误查询思路
一开始自然是从lombok的版本开始找问题。按照网上的说法从1.6.22换成了最新的版本1.8，可惜不行。

只好呼叫团队大佬出手。一开始的解决思路也是重新build一下项目，删除jar包重新导入。之后因为idea执行时的控制台打印有条数限制，开启了本文件夹中的命令行发现了在执行gradle build时的开头会有一个报错。**lombok.javac.apt.LombokProcessor could not be initialized**

再然后大佬就发现了gradle版本兼容问题，把我的项目gradle设置成了4.8.1，全局gradle依旧不变。成功的解决了这个问题