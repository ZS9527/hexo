---
title: Idea报错 【cannot access com.xxx】的【解决办法】
date: 2020-11-28 17:47:29
tags:
---

# Idea报错 【cannot access com.xxx】的【解决办法】
> 本文来自 [https://blog.csdn.net/qq_27818541/article/details/105078428](https://blog.csdn.net/qq_27818541/article/details/105078428)
> 我也没搞明白为啥会这样，但是这个文章中的方法我尝试后解决了问题
> 
<!--more-->

## 起因
从别的项目中抽取代码，将代码包路径改动后出现。具体表现为工具类正常但是无法调用，实体类正常也无法使用。直接 import 路径也不行。
## 解决方案
 执行下面操作，执行完后项目变为正常。
![1](b1.png)
![2](b2.png)