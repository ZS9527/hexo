---
title: 简单记录JPA自定义删除方法
date: 2019-10-20 10:48:44
tags: jpa
---

# 简单记录JPA自定义删除方法
> 其实之前使用过多次的时候很是熟悉，但是长期不写后还是要去百度一下配置。干脆就在这里记录一下，下次直接查询方便一些

<!--more-->
## 添加注解
1. 需要在删除方法上添加@Modifying
2. 使用时注意添加@Transactional事务控制，否则就会报错

## 更新LazyInitializationException
使用懒加载注解时也必须添加@Transactional事务控制，否则会报下面的错误
```
Caused by: org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role:
```