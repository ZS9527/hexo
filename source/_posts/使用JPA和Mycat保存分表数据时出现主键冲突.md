---
title: 使用JPA和Mycat保存分表数据时出现主键冲突
date: 2020-05-30 09:14:54
tags: bug
---

# 使用JPA和Mycat保存分表数据时出现主键冲突
> 将一个百万数据级别的表拆分为几十万数据的表后，保存数据时出现了问题

<!--more-->

## bug描述
**A different object with the same identifier value was already associated with the session** 一开始保存时出现了这个相同 id 的问题。之前调用 Jpa 的 save 方法时，有 id 为 update 没有 id 时为 insert 。尝试增加一下事务管理，错误变成了**Sharding column can't be updated 分表字段**。

## 处理方式
1. 将主键的自增方式转换为snowflake。通过添加自增算法类，然后再添加下面的主键注解
```
@GeneratedValue(generator = "snowflake")
    @GenericGenerator(name = "snowflake", strategy = "snowflake.GenerateId")
```

2. 使用Jpa注解自己创建sql语句更新
```java
@Modifying
    @Query("update ClimateEveryYearStationData set totalPreTwenty = ?1, avgTem = ?2, maxTem = ?3, minTem = ?4 where id = ?5")
    
```