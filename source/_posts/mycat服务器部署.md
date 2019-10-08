---
title: mycat服务器部署
date: 2019-09-14 14:28:48
tags: mycat
---

# mycat服务器部署

> 记录一下自己将mycat部署到服务器上的步骤和遇到的bug。

<!--more-->

## 前置条件
已经完成在服务器部署多个mysql服务的任务，mysql版本选择为8.0以上。

## 下载mycat
直接从github上面下载最新版本的mycat，(https://github.com/MyCATApache/Mycat-Server)[https://github.com/MyCATApache/Mycat-Server] 。

## 解压后配置文件
> 目前版本为mycat1.6.7，之后的版本更新后配置方式以官方文档为准。

重要的有三个文件需要配置
### rule.xml
这个配置文件定义分片规则，目前只是用到了这些参数：时间格式，分片开始时间，结束时间。

**注意：**这里的这个时间段是要对应之后的schema.xml中的table标签的subTables属性的分表个数。例如下面的按月分表从2019年1月一直分到2024年12月的规则，那么相对应的subTables属性就需要固定好从2019年1月分到2024年12月的表名
```xml
<function name="partbymonthofair"
	class="io.mycat.route.function.PartitionByMonth">
	<property name="dateFormat">yyyy-MM</property>
	<property name="sBeginDate">2019-01</property>
	<property name="sEndDate">2024-12</property>
</function>
```
规定表分片规则
```xml
<tableRule name="env-air-sharding-by-date">
	<rule>
		<columns>date_time</columns>
		<algorithm>partbymonthofair</algorithm>
	</rule>
</tableRule>
```

### schema.xml
在这里定义表的规则。
**schema**标签是用于定义具体的库，之后的server.xml中配置的访问用户需要有对这个库的操作权限。
其中的属性：
- **checkSQLschema**是检查sql语句中是否带有env_v2这个名字，true为自动删除sql语句中的这个名字。
- ** sqlMaxLimit**是指自动在查询语句中限制返回的条数，sql语句中有的话会自动替换。
- **dataNode**是用于定义具体的库，之后的server.xml中配置的访问用户需要有对这个库的操作权限。

**table**标签定义MyCat 中的逻辑表。
其中的属性：
- **name**定义逻辑表的表名，这个名字就如同我在数据库中执行 create table 命令指定的名字一样，同schema 标签中定义的名字必须唯一。
- **dataNode**定义这个逻辑表所属的 dataNode, 该属性的值需要和 dataNode 标签中 name 属性的值相互对应。
- 如果需要定义的 dn 过多 可以使用如下的方法减少配置。（此次出自mysql权威指南）

```xml
<table name="travelrecord" dataNode="multipleDn$0-99,multipleDn2$100-199"rule="auto-sharding-long" ></table>

<dataNode name="multipleDn$0-99" dataHost="localhost1" database="db$0-99"</dataNode>

<dataNode name="multipleDn2$100-199" dataHost="localhost1" database=" db$100-199"></dataNode>
```


- **rule**该属性用于指定逻辑表要使用的规则名字，规则名字在 rule.xml 中定义，必须和tableRule 标签中 name 属性属性值一一对应。
- **needAddLimit**是否需要自动的在每个语句后面加上 limit 限制，默认为true。

#### 实例：
```xml
<schema name="db" checkSQLschema="false" sqlMaxLimit="300" dataNode="dn2">
		<table name="gather_air_data" subTables="gather_air_data_$201901-201912,gather_air_data_$202001-202012,gather_air_data_$202101-202112,gather_air_data_$202201-202212,gather_air_data_$202301-202312,gather_air_data_$202401-202412" dataNode="dn3" rule="env-air-sharding-by-date"  needAddLimit ="false"/>
		<table name="gather_city_aqi_data" subTables="gather_city_aqi_data_$201901-201912,gather_city_aqi_data_$202001-202012,gather_city_aqi_data_$202101-202112,gather_city_aqi_data_$202201-202212,gather_city_aqi_data_$202301-202312,gather_city_aqi_data_$202401-202412" dataNode="dn3" rule="env-air-sharding-by-date"  needAddLimit ="false"/>
		<table name="gather_observation_factor" subTables="gather_observation_factor_$201901-201912,gather_observation_factor_$202001-202012,gather_observation_factor_$202101-202112,gather_observation_factor_$202201-202212,gather_observation_factor_$202301-202312,gather_observation_factor_$202401-202412" dataNode="dn3" rule="env-air-sharding-by-date"  needAddLimit ="false"/>
	</schema>
```

**dataNode** 标签定义了 MyCat 中的数据节点
其中的属性：
- **name**定义数据节点的名字
- **dataHost**该属性用于定义该分片属于哪个数据库实例的，属性值是引用dataHost标签上定义的 name属性
- **database**该属性用于定义该分片属性哪个具体数据库实例上的具体库
#### 实例：
```xml
<dataNode name="dn2" dataHost="localhost" database="db" />
<dataNode name="dn3" dataHost="localhost" database="db_v2" />
```

**dataHost**标签定义了具体的数据库实例、读写分离配置和心跳语句
其中的属性：
- **maxCon**与**minCon**指定每个读写实例连接池的最大与最小连接，以及初始化连接池的大小。
- **balance**是负载均衡类型，有三种取值
	1. balance="0", 不开启读写分离机制，所有读操作都发送到当前可用的 writeHost 上
	2. balance="1"，全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情况下M2,S1,S2 都参与 select 语句的负载均衡。
	3. balance="2"，所有读操作都随机的在 writeHost、readhost 上分发。
	4. balance="3"，所有读请求随机的分发到 wiriterHost 对应的 readhost 执行，writerHost 不负担读压力。
- **writeType**负载均衡类型（具体还是不太明白，套的官方示例）
- **dbType**指定后端连接的数据库类型
- **dbDriver**指定连接后端数据库使用的 Driver，目前可选的值有 native 和 JDBC。

**heartbeat**标签内指明用于和后端数据库进行心跳检查的语句。例如,MYSQL 可以使用 select user()。

**writeHost 标签**、**readHost 标签**两个标签都指定后端数据库的相关配置给 mycat，用于实例化后端连接池。
其中的属性：
- **url**后端实例连接地址
- **user**后端存储实例需要的用户名字。
- **password**后端存储实例需要的密码。


#### 实例：
```xml
<dataHost name="localhost" maxCon="1000" minCon="10" balance="1" writeType="0" dbType="mysql" dbDriver="native" switchType="-1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<writeHost host="hostMaster" url="localhost:3306" user="root" password="root" />
		<writeHost host="hostSalve" url="localhost:3308" user="root" password="root" />
		
</dataHost>
```
### server.xml
server.xml 几乎保存了所有 mycat 需要的系统配置信息。
以下内容出自mycat权威指南:
- server.xml 中的标签本就不多，这个标签主要用于定义登录 mycat 的用户和权限。例如上面的例子中，我定义了一个用户，用户名为 test、密码也为 test，可访问的 schema 也只有 TESTDB 一个。

- 如果我在 schema.xml 中定义了多个 schema，那么这个用户是无法访问其他的 schema。在 mysql 客户端看来则是无法使用 use 切换到这个其他的数据库。

实例：
```xml
<user name="root" defaultAccount="true">
		<property name="password">root</property>
		<property name="schemas">db</property>
</user>
```
## 配置成服务并启动
从./bin目录下打开命令行启动mycat

```
mycat install #配置成服务

mycat start #启动服务

mycat stop #停止服务
```
## 后记：
可以使用在server.xml中配置好的用户去远程访问，记得开启服务器的防火墙8066端口。

### 遇到的bug
在将程序部署到服务器上时遇到了一些问题。
#### 1. jpa的save方法在传入id修改数据时，是全部数据字段修改。而mycat不能修改分片的时间根据字段。
报错：**Sharding column can't be updated GATHER_OBSERVATION_FACTOR->DATE_TIME**
处理方式：将datetime字段加上@Column(updatable=false)注释

#### 2.jpa对一条数据查出来后，直接把原数据的id改为null会报错
报错：**identifier of an instance of .. altered to null**
处理方式：不能直接更新已经查到的数据，要new一个对象实例，进行属性复制（BeanUtils.copyProperties(source, dist)）

#### 3.在rule.xml的tableRule标签中的分片根据表字段columns不能为空，为空无法分片
报错：**columnValue:NULL Please check if the format satisfied.**
处理方式：删除掉为null的数据

#### 4.jpa删除时注意实际执行语句，有时执行语句是按id删除的。mycat会报错