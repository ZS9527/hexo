---
title: java连接hbase摸索过程
date: 2019-06-28 22:39:34
tags: hbase
---
# java连接hbase摸索过程
> 记录自己摸索使用javaApi操作hbase的过程

<!--more-->

## 连接hbase
在windows系统中的**C:\Windows\System32\drivers\etc**中找到**hosts**文件，添加
```
192.168.99.100 myhbase
```
**注意：** 192.168.99.100为我本地配置的虚拟机ip，myhbase是我的容器启动时的别名

在虚拟机系统中也同样更改hosts文件
```
vim /etc/hosts
```
添加 虚拟机ip 别名
```
192.168.99.100  myhbase
```

## 启动已经配置好的hbase容器

先用命令查看已经配置好的容器
```
docker ps -a
```

然后在用命令启动容器
```
docker start b8956b1ba7ec
```
**container ID**这里是用上一条命令查出来的要启动的容器id字段，例如我的容器id为：b8956b1ba7ec
停止容器的命令为
```
docker stop b8956b1ba7ec
```
## 未完成

导入jar包：
```
compile (group: 'org.apache.hbase', name: 'hbase-client', version: '2.1.1'){
        exclude module: 'servlet-api'
    }
    compile (group: 'org.apache.hbase', name: 'hbase', version: '2.1.1', ext: 'pom')
```

**这里附加说明：**本来我是导入了1.3.1版本的，后来进入容器查看了docker里面的hbase版本后（进入容器后命令为：**hbase version**）。就改为和镜像版本一致。

编写测试类：
```

import java.io.IOException;
import lombok.extern.slf4j.Slf4j;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.MasterNotRunningException;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.ZooKeeperConnectionException;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.TableDescriptor;
import org.apache.hadoop.hbase.client.TableDescriptorBuilder;

/**
 * TODO
 *
 * @author zhangshuai
 * @date 2019/6/24 15:15
 */

@Slf4j
public class HbaseService {

    Admin admin;
    Configuration conf;

    /**
     * 构造函数 加载配置
     */
    public HbaseService() {
        conf = new Configuration();
        conf.set("hbase.zookeeper.quorum", "192.168.99.100:2181");
        
        try {
            admin = ConnectionFactory.createConnection(conf).getAdmin();
        } catch (MasterNotRunningException e) {
            e.printStackTrace();
        } catch (ZooKeeperConnectionException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 创建一张表
     *
     * @param tableName
     * @param column
     * @throws Exception
     */
    public void createTable(String tableName, String column) throws Exception {
        if (admin.tableExists(TableName.valueOf(tableName))) {
            System.out.println(tableName + "表已经存在！");
        } else {
            TableDescriptor tableDesc = TableDescriptorBuilder.newBuilder(TableName.valueOf(tableName))
                .setColumnFamily(ColumnFamilyDescriptorBuilder.of("t1")).build();
            admin.createTable(tableDesc);
            System.out.println(tableName + "表创建成功！");
        }
    }

    public static void main(String[] args) throws Exception {
        HbaseService hbase = new HbaseService();
        hbase.createTable("t1","cf");
    }
}

```
此次注意：192.168.99.100是我的docker地址。

### 错误内容一

**Connection refused: no further information:**
```
Caused by: org.apache.hbase.thirdparty.io.netty.channel.AbstractChannel$AnnotatedConnectException: Connection refused: no further information: myhbase/192.168.99.100:16020
```

**Not trying to connect to**
```
DEBUG org.apache.hadoop.hbase.ipc.AbstractRpcClient - Not trying to connect to myhbase/192.168.99.100:16020 this server is in the failed servers list
```

#### 解决历程：
首先我检查了本机的hosts文件，文件路径在**C:\Windows\System32\drivers\etc**。发现已经配置好了192.168.99.100 myhbase。

再检查了docker的hosts文件，命令为：**vim /etc/hosts**。增加了192.168.99.100 myhbase。
发现依旧是这个错误。

**更新一下：**docker里的hosts文件映射路径改成了容器ip。命令：docker inspect 容器名称或 id。
hosts文件内容修改为172.17.0.2 myhbase

![](b3.png)

依旧报错，还是不行。

后来仔细研究发现，是容器端口映射问题。

本机可以访问http://192.168.99.100:16010/master-status 。在页面上发现，镜像2.1.1版本的默认Region Servers端口为：16020

![](b1.png)

可以在本页面往下拉发现hbase版本号

![](b2.png)

将端口映射改为16020和16030就可以解决了上面的这个连接不上的错误。接下来，就是对hbase的api操作了。
