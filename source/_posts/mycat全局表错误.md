---
title: mycat全局表错误
date: 2020-08-30 15:28:41
tags: mycat
---
# mycat全局表错误
> 具体报错内容为：can't find table define in schema DIC_HIDDEN_DANGER_ADM schema:risk

## 缘由
部署时新加入了字典表，在mycat连接的两个库里都写入了一次。但是在查询时，和第一个库的普通表 join 时可以查询。和在另一个库的分表后的表 join 时会显示无法查询出字典表。
## 处理
在 schemal.xml 中将新写入的字典表加入 node 节点，添加  type="global" 标记为全局表
## 二次报错
```sql
SELECT
	r.dictionary_id AS rid,
	r.date_time AS dateTime,
FROM
	bs_mountain r
	INNER JOIN dic_mountain ra ON ra.id = r.dictionary_id
WHERE
	r.date_time BETWEEN '2020-10-14 00:00:00' 
	AND '2020-10-15 00:00:00' ;
```
可以确定的是 bs_mountain 表中是肯定有 dictionary_id 字段和 date_time ，但还是报出了 `Unknown column 'r.dictionary_id' in 'field list'` 错误。
## 二次处理
通过 explain 关键字分析在 mycat 中执行的语句，得出结果
实际执行语句为
```sql
SELECT
	r.dictionary_id AS rid,
	r.date_time AS dateTime,
	
FROM
	bs_mountain_32 
WHERE
	r.date_time BETWEEN '2020-10-14 00:00:00' 
	AND '2020-10-15 00:00:00'
```
mycat 在改写 sql 语句到分表的时候漏下了别名

而且当我的语句是：
```sql
SELECT
	r.dictionary_id AS rid,
	r.date_time AS dateTime,
FROM
	bs_mountain r
	INNER JOIN dic_mountain ra ON ra.id = r.dictionary_id
WHERE
	r.date_time BETWEEN '2020-10-14 00:00:00' 
	AND '2020-10-15 00:00:00' 
	AND ra.id = '1';
```
会去执行一个全分表查询，不再单单只是定位到32分表的查询。


开始百度关键字 `mycat 别名 全局表 join`，发现问题是出在 mycat 在单库分表和未分表关联查询时会出现的别名丢失问题。

并没有去下载源码修改后重新打包，而是在业务中去过滤了。

直接使用sql语句将要关联的表手动查询出来后，再代码中拼接关联查询。


