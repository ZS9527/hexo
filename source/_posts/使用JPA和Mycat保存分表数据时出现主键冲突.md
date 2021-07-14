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
    @GenericGenerator(name = "snowflake", strategy = "com.yiring.env.snowflake.GenerateId")
```

2. 使用Jpa注解自己创建sql语句更新
```java
@Modifying
    @Query("update ClimateEveryYearStationData set totalPreTwenty = ?1, avgTem = ?2, maxTem = ?3, minTem = ?4 where id = ?5")
    
```

## 代码
```java

import java.io.Serializable;
import java.util.Properties;
import org.hibernate.HibernateException;
import org.hibernate.MappingException;
import org.hibernate.engine.spi.SharedSessionContractImplementor;
import org.hibernate.id.Configurable;
import org.hibernate.id.IdentifierGenerator;
import org.hibernate.service.ServiceRegistry;
import org.hibernate.type.Type;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * 基于雪花算法的 ID 生成器
 *
 */

@Service
@Component
@Transactional(readOnly = true)
public class GenerateId implements IdentifierGenerator, Configurable {

    private SnowFlakeIdWorker snowFlakeIdWorker;

    @Override
    public Serializable generate(
        SharedSessionContractImplementor session, Object object) throws HibernateException {
        return snowFlakeIdWorker.nextId();
    }

    @Override
    public void configure(
        Type type, Properties params, ServiceRegistry serviceRegistry) throws MappingException {
        snowFlakeIdWorker = new SnowFlakeIdWorker(1L, 0L);
    }
}

```

```java
package com.yiring.env.snowflake;

/**
 * Twitter_Snowflake官网demo格式
 * SnowFlake的结构如下(每部分用-分开):
 * 1位标志位                    41位时间戳                                               5位机器+5位数据标志          12位计数器
 * 0 - 0000000000 0000000000 0000000000 0000000000 0 - 00000 - 00000 - 000000000000
 * 41位时间戳是因为当前时间减去起始时间的时间戳刚好在2^41范围内
 * 1位标识，由于long基本类型在Java中是带符号的，最高位是符号位，正数是0，负数是1，所以id一般是正数，最高位是0
 * 41位时间截(毫秒级)，注意，41位时间截不是存储当前时间的时间截，而是存储时间截的差值（当前时间截 - 开始时间截)
 * 得到的值），这里的的开始时间截，一般是我们的id生成器开始使用的时间，由我们程序来指定的（如下下面程序IdWorker类的startTime属性）。41位的时间截，可以使用69年，年T = (1L << 41) / (1000L * 60 * 60 * 24 * 365) = 69
 * 10位的数据机器位，可以部署在1024个节点，包括5位dataCenterId和5位workerId
 * 12位序列，毫秒内的计数，12位的计数顺序号支持每个节点每毫秒(同一机器，同一时间截)产生4096个ID序号
 * 加起来刚好64位，为一个Long型。
 * SnowFlake的优点是，整体上按照时间自增排序，并且整个分布式系统内不会产生ID碰撞(由数据中心ID和机器ID作区分)，并且效率较高，经测试，SnowFlake每秒能够产生26万ID左右。
 * <p>
 * 算法在的<<表示左移，左移后后面剩下部分会补0，并不是循环移位
 * 二进制转换运算中负数的二进制需要用补码表示
 * 补码 = 反码 + 1 ;   补码 = （原码 - 1）再取反码
 * <p>
 * 本代码对字段进行了调整，采用43位时间戳  3位机器标志、3位数据标志  14位
 *
 */

public class SnowFlakeIdWorker {

    /**
     * 机器id所占的位数
     */
    private final long workerIdBits = 3L;

    /**
     * 数据标识id所占的位数
     */
    private final long dataCenterIdBits = 3L;

    /**
     * 工作机器ID(0~7)
     */
    private long workerId;

    /**
     * 数据中心ID(0~7)
     */
    private long dataCenterId;

    /**
     * 毫秒内序列(0~16383)
     */
    private long sequence = 0L;

    /**
     * 上次生成ID的时间截
     */
    private long lastTimestamp = -1L;

    //private static String wordId = "1";
    //private static String dataId = "0";

    //private static org.hibernate.snowflake.SnowFlakeIdWorker idWorker;


    /**
     * 构造函数
     *
     * @param workerId     工作ID (0~7)
     * @param dataCenterId 数据中心ID (0~7)
     */
    public SnowFlakeIdWorker(long workerId, long dataCenterId) {
        /*
          支持的最大机器id，结果是7 (这个移位算法可以很快的计算出几位二进制数所能表示的最大十进制数)
          -1L ^ (-1L << n)表示占n个bit的数字的最大值是多少。举个栗子：-1L ^ (-1L << 2)等于10进制的3 ，即二进制的11表示十进制3。
         */
        long maxWorkerId = ~(-1L << workerIdBits);
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        /*
          支持的最大数据标识id，结果是7
          -1L 原码 1000 0001   原码是在符号位加标志  正数 0 负数1
          反码1111 1110   正数原码是其本身 负数是符号位不变其余位取反
          补码1111 1111   原码-1再取反
          -1L << 3 1111 1000
          -1L ^ (1111 1000)   1111 1000 ^ 1111 1111 结果为7
         */
        long maxDataCenterId = ~(-1L << dataCenterIdBits);
        if (dataCenterId > maxDataCenterId || dataCenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDataCenterId));
        }
        this.workerId = workerId;
        this.dataCenterId = dataCenterId;
    }

    /**
     * 获得下一个ID (该方法是线程安全的)
     *
     * @return SnowflakeId
     */
    public synchronized long nextId() {
        long timestamp = timeGen();

        //如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(
                    String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }

        //如果是同一时间生成的，则进行毫秒内序列
        //序列在id中占的位数
        long sequenceBits = 14L;
        if (lastTimestamp == timestamp) {
            //序列号 = 上次序列号+1 与 生成序列的掩码
            //生成序列的掩码，这里为16383
            long sequenceMask = ~(-1L << sequenceBits);
            sequence = (sequence + 1) & sequenceMask;
            //毫秒内序列溢出
            if (sequence == 0) {
                //阻塞到下一个毫秒,获得新的时间戳
                timestamp = tilNextMillis(lastTimestamp);
            }
        }
        //时间戳改变，毫秒内序列重置
        else {
            sequence = 0L;
        }

        //上次生成ID的时间截（保存时间戳）
        lastTimestamp = timestamp;

        //移位并通过或运算拼到一起组成64位的ID(或运算)
        //当前时间戳-开始时间戳 左移22位相当于两个差的22位
        //机器ID向左移14位  数据标识id向左移17位(14+3)
        long dataCenterIdShift = sequenceBits + workerIdBits;
        //时间截向左移20位(3+3+14) 机器码+计数器
        long timestampLeftShift = sequenceBits + workerIdBits + dataCenterIdBits;
        //开始时间截 (2020-01-01 00:00:00)毫秒级时间戳
        long startTimestamp = 1577808000000L;
        return ((timestamp - startTimestamp) << timestampLeftShift)
                //数据中心Id左移17位
                | (dataCenterId << dataCenterIdShift)
                // 机器id左移12位
                | (workerId << sequenceBits)
                //毫秒内序列
                | sequence;
    }

    /**
     * 阻塞到下一个毫秒，直到获得新的时间戳
     *
     * @param lastTimestamp 上次生成ID的时间截
     * @return 当前时间戳
     */
    private long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }

    /**
     * 返回以毫秒为单位的当前时间
     *
     * @return 当前时间(毫秒)
     */
    private long timeGen() {
        return System.currentTimeMillis();
    }

}

```