---
title: 使用mycat分表后的A different object with the same identifier value错误
date: 2019-10-26 14:13:40
tags: mycat
---
# 使用mycat分表后的A different object with the same identifier value错误
> 这个问题在之前使用mycat时并没有发生。这次的业务场景是我需要在同一事务中，先查询历史数据，然后再写入新的数据。该新的数据是按时间分表。

<!--more-->


## 错误：
**A different object with the same identifier value was already associated with the session** ：[com.yiring.risk.domain.hiddendangerpoint.HiddenDangerPointNationalScene#776]

## 首次尝试-更换JPA查询
产生这个bug的时候，任务时间很紧张。我就没有百度，直接询问团队大佬。大佬表示有可能是因为同一事务中的查出的数据id和要保存的数据id为同一个。

就把JPA原本的findAllByDateTimeBetween改成了通过@Query(value="原生sql"，nativeQuery = true)来直接查询出List<Map<String, Object>> 

本次尝试失败

## 第二次尝试-更换成JDBC查询
上次的修改失败了之后，再次尝试将原生sql查询，更改为@JdbcTemplate注解后。通过jdbcTemplate.queryForList(sql);查询。

再次尝试失败

## 第三次尝试-减少每次查询时的数据
大佬再次思考过后，认为是saveAll时的数据有不同的时间。在同一事务中的不同时间有可能分到不同的分表里，也就是可能在缓存区里有相同的id。

决定把数据分为按同一时间存储

目前解决了这个bug，但是具体的细节还是没有搞太明白。之后再进行补充