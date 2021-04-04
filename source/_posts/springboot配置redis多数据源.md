---
title: springboot配置redis多数据源
date: 2020-12-12 19:46:27
tags: redis
---

# springboot配置redis多数据源
> 本文来自[https://iogogogo.github.io/2020/01/10/spring-boot-redis-multi-instance/](https://iogogogo.github.io/2020/01/10/spring-boot-redis-multi-instance/) ，我用了后记录一下。
> 
<!--more-->

## 序章
Spring Boot 2.x版本升级以后，Redis连接库由原来的`Jedis`换成了`Lettuce`，但是提供给上层使用的api没有变化，在日常使用过程中难免会有需要使用多个库的情况，或者使用多个Redis实例，那么这个时候就需要维护两个Redis连接池或者说两个`RedisTemplate` 。

## 实现思路
1. 多数据源最终表现其实就是 `RedisConnectionFactory` 不同
2. Spring Boot 通过`RedisStandaloneConfiguration`维护了一套默认的`RedisConnectionFactory`，要实现多个实例或者使用多个db，理论上只需要自己在维护一套`RedisConnectionFactory`即可。
3. 不管是Jedis或是Lettuce，都是通用的，所以我们可以将两个的实现方式都整理出来，区别如下：
    - `Lettuce`使用的是`LettuceConnectionFactory`
    - `Jedis`使用的是`JedisConnectionFactory`

## 创建Redis服务
为了方便测试，直接使用`docker`创建一个Redis服务，以下是一个完整的启动脚本
1. `--restart=always`设置了Redis服务随着docker进程的启动而启动
2. `-p 6379:6379 `将docker启动的服务端口映射到宿主机，这里是必须的，否则宿主机和redis的端口是不能通信的
3. `--requirepass "redis"` 设置密码为`redis`
4. `--appendonly yes`保存aof持久化文件
5. `-v ~/share/docker/data/redis:/data \` 将docker启动的redis的数据映射到宿主机目录

```
mkdir -p ~/share/docker/data/redis
chmod 755 -R ~/share/docker/data/redis

docker run -dit --restart=always --name redis \
    -v ~/share/docker/data/redis:/data \
    -p 6379:6379 \
    redis:latest redis-server --requirepass "redis" --appendonly yes
```
如果觉得不需要设置这么多，可以使用以下最简单的脚本，仅仅将端口做了映射，其他的都是最简单的配置
```
docker run -dit --name redis -p 6379:6379 redis:latest
```

## 项目pom配置
使用Spring Boot集成Redis，只需要将`spring-boot-starter-data-redis`和`commons-pool2`加到依赖即可
```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-pool2</artifactId>
</dependency>
```

## 自定义RedisConfigure
这里这一步是最重要的，因为Spring Boot默认帮我们维护了一个RedisConnectionFactory，前面说了要使用不同的Redis实例就需要自己在维护一个RedisConnectionFactory，这里就以使用两个redis的database为例

### 配置文件
`spring.redis`开头的都是Spring Boot自动注入需要加载的配置，我们为了在使用一个db2，这里加了一个`spring.redis-db-2`开头的配置
```
spring:
  redis:
    database: 0
    host: localhost
    port: 6379
    password:
    timeout: 60000
    lettuce:
      pool:
        max-active: 8 # 连接池最大连接数（使用负值表示没有限制） 默认 8
        max-wait: -1 # 连接池最大阻塞等待时间（使用负值表示没有限制） 默认 -1
        max-idle: 8 # 连接池中的最大空闲连接 默认 8
        min-idle: 0 # 连接池中的最小空闲连接 默认 0
  redis-db-2:
    database: 2
    host: 127.0.0.1
    port: 6379
    password:
    timeout: 60000
    lettuce:
      pool:
        max-active: 8
        max-wait: 8
        max-idle: 8
        min-idle: 0
```
### RedisConfigure
```java
package com.iogogogo.redis.configure;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.vavr.Tuple;
import io.vavr.Tuple6;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisPassword;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.jedis.JedisClientConfiguration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettucePoolingClientConfiguration;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

/**
 * Created by tao.zeng on 2020-01-10.
 *
 * @Value("${spring.redis-db-2.database}") int database,
 * @Value("${spring.redis-db-2.host}") String host,
 * @Value("${spring.redis-db-2.port}") int port,
 * @Value("${spring.redis-db-2.password}") String password,
 * @Value("${spring.redis-db-2.timeout}") long timeout,
 * @Value("${spring.redis-db-2.lettuce.pool.max-active}") int maxActive,
 * @Value("${spring.redis-db-2.lettuce.pool.max-wait}") int maxWait,
 * @Value("${spring.redis-db-2.lettuce.pool.max-idle}") int maxIdle,
 * @Value("${spring.redis-db-2.lettuce.pool.min-idle}") int minIdle
 */
@EnableCaching
@Configuration
public class RedisConfigure {

    @Bean
    public RedisTemplate redisTemplateDB_2(Tuple6<RedisStandaloneConfiguration, Long, Integer, Integer, Integer, Integer> redisConfigurationDB_2) {

        Long timeout = redisConfigurationDB_2._2();

        int maxActive = redisConfigurationDB_2._3(),
                maxWait = redisConfigurationDB_2._4(),
                maxIdle = redisConfigurationDB_2._5(),
                minIdle = redisConfigurationDB_2._6();


        /* ========= 基本配置 ========= */
        RedisStandaloneConfiguration standaloneConfiguration = redisConfigurationDB_2._1();


        /* ========= 连接池通用配置 ========= */
        GenericObjectPoolConfig genericObjectPoolConfig = new GenericObjectPoolConfig();
        genericObjectPoolConfig.setMaxTotal(maxActive);
        genericObjectPoolConfig.setMaxWaitMillis(maxWait);
        genericObjectPoolConfig.setMaxIdle(maxIdle);
        genericObjectPoolConfig.setMinIdle(minIdle);

        /* ========= jedis pool ========= */
        // jedisConnectionFactory(standaloneConfiguration, genericObjectPoolConfig, timeout);

        /* ========= lettuce pool ========= */
        LettuceConnectionFactory connectionFactory = lettuceConnectionFactory(standaloneConfiguration, genericObjectPoolConfig, timeout);

        // 连接池初始化
        connectionFactory.afterPropertiesSet();

        // 创建 RedisTemplate
        return createRedisTemplate(connectionFactory);
    }

    /**
     * lettuceConnectionFactory
     *
     * @param standaloneConfiguration Redis标准配置
     * @param genericObjectPoolConfig Redis通用配置
     * @param timeout                 超时时间
     * @return
     */
    private LettuceConnectionFactory lettuceConnectionFactory(RedisStandaloneConfiguration standaloneConfiguration, GenericObjectPoolConfig genericObjectPoolConfig, long timeout) {
        LettucePoolingClientConfiguration.LettucePoolingClientConfigurationBuilder builder = LettucePoolingClientConfiguration.builder();
        builder.poolConfig(genericObjectPoolConfig);
        builder.commandTimeout(Duration.ofSeconds(timeout));
        return new LettuceConnectionFactory(standaloneConfiguration, builder.build());
    }

    /**
     * jedisConnectionFactory
     *
     * @param standaloneConfiguration Redis标准配置
     * @param genericObjectPoolConfig Redis通用配置
     * @param timeout                 超时时间
     * @return
     */
    private JedisConnectionFactory jedisConnectionFactory(RedisStandaloneConfiguration standaloneConfiguration, GenericObjectPoolConfig genericObjectPoolConfig, long timeout) {
        JedisClientConfiguration.DefaultJedisClientConfigurationBuilder builder = (JedisClientConfiguration.DefaultJedisClientConfigurationBuilder) JedisClientConfiguration
                .builder();
        builder.connectTimeout(Duration.ofSeconds(timeout));
        builder.usePooling();
        builder.poolConfig(genericObjectPoolConfig);
        return new JedisConnectionFactory(standaloneConfiguration, builder.build());
    }

    /**
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        return createRedisTemplate(redisConnectionFactory);
    }

    /**
     * json 实现 redisTemplate
     * <p>
     * 该方法不能加 @Bean 否则不管如何调用，RedisConnectionFactory 都会是默认配置
     *
     * @param redisConnectionFactory
     * @return
     */
    private RedisTemplate createRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setKeySerializer(stringRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

    /**
     * 自定义Redis的配置加载
     *
     * @param database
     * @param password
     * @param host
     * @param port
     * @param timeout
     * @param maxActive
     * @param maxWait
     * @param maxIdle
     * @param minIdle
     * @return
     */
    @Bean
    public Tuple6<RedisStandaloneConfiguration, Long, Integer, Integer, Integer, Integer>
    redisConfigurationDB_2(@Value("${spring.redis-db-2.database}") int database,
                           @Value("${spring.redis-db-2.password}") String password,
                           @Value("${spring.redis-db-2.host}") String host,
                           @Value("${spring.redis-db-2.port}") int port,
                           @Value("${spring.redis-db-2.timeout}") long timeout,
                           @Value("${spring.redis-db-2.lettuce.pool.max-active}") int maxActive,
                           @Value("${spring.redis-db-2.lettuce.pool.max-wait}") int maxWait,
                           @Value("${spring.redis-db-2.lettuce.pool.max-idle}") int maxIdle,
                           @Value("${spring.redis-db-2.lettuce.pool.min-idle}") int minIdle) {

        RedisStandaloneConfiguration standaloneConfiguration = new RedisStandaloneConfiguration();
        standaloneConfiguration.setDatabase(database);
        standaloneConfiguration.setHostName(host);
        standaloneConfiguration.setPort(port);

        if (StringUtils.isNotEmpty(password)) {
            RedisPassword redisPassword = RedisPassword.of(password);
            standaloneConfiguration.setPassword(redisPassword);
        }

        return Tuple.of(standaloneConfiguration, timeout, maxActive, maxWait, maxIdle, minIdle);

    }
}
```
这个配置有点长，但是都有注释，而且这里也用到了上篇文章中讲到的`tuple`（在方法有多个返回值时，元组真香）,其实就已经可以使用两个不同的`RedisTemplate`了，这时候我们在启动项目时分别注入`redisTemplate`和`redisTemplateDB_2`，别问我为啥名字是这两货。

## 通过debug查看配置是否成功
启动类我们实现`CommandLineRunner`，然后通过debug查看`redisTemplate`和`redisTemplateDB2`的`RedisConnectionFactory`
并且我们在database=0存储了一个string数据，在database=2存储了一个hash数据
```java
package com.iogogogo.redis;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.redis.core.RedisTemplate;

/**
 * Created by tao.zeng on 2020-01-10.
 */
@Slf4j
@SpringBootApplication
public class RedisApplication implements CommandLineRunner {

    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    private RedisTemplate redisTemplateDB_2;

    public static void main(String[] args) {
        SpringApplication.run(RedisApplication.class, args);
    }

    @Override
    public void run(String... args) {
        log.info("redisTemplate:{}", redisTemplate);
        redisTemplate.opsForValue().set("iogogogo", "redisTemplate save value");


        log.info("redisTemplateDB_2:{}", redisTemplateDB2);
        redisTemplateDB2.opsForHash().put("iogogogo", "iogogogo-hash", "redisTemplateDB_2 save value");
    }
}

```
- redisTemplate
![redisTemplate](b1.png)
- redisTemplateDB_2
![redisTemplateDB_2](b2.png)

略过原文的查看存储数据。。。

通过以上日志我们可以看到分别在不同的数据库已经存储了不同的数据，到这里我们redis配置多实例就已经完成。但是我们每次注入的时候都要记得自己创建的Bean的名字，这样对其他人不太友好，那么我们可以进一步封装一下，自定义一个handler进行获取RedisTemplate，这样会不会更便捷呢？

## 自定义RedisHandler和RedisOperations
做这一步主要是为了将下层的RedisTemplate进行统一的封装，对外只是一个`RedisHandler`和`RedisOperations`，`RedisHandler`提供获取`RedisOperations`的方法，`RedisOperations`里面可以封装一些常用的redis操作，这样就只需要和`RedisOperations`进行操作，从而避免同时操作多个`RedisTemplate`

### 项目结构
```
➜  example-redis git:(master) ✗ tree ./
  ./
  ├── example-redis.iml
  ├── pom.xml
  └── src
      ├── main
      │   ├── java
      │   │   └── com
      │   │       └── iogogogo
      │   │           └── redis
      │   │               ├── RedisApplication.java
      │   │               └── configure
      │   │                   ├── RedisConfigure.java
      │   │                   ├── handler
      │   │                   │   └── RedisHandler.java
      │   │                   └── util
      │   │                       └── RedisOperations.java
      │   └── resources
      │       └── application.yml
      └── test
          └── java
```

### RedisOperations
该类可以作为redis的工具类使用，自己添加一些常用的方法
```java
package com.iogogogo.redis.configure.util;

  import lombok.extern.slf4j.Slf4j;
  import org.springframework.data.redis.core.RedisTemplate;

  /**
   * Created by tao.zeng on 2020-01-10.
   */
  @Slf4j
  public class RedisOperations {

      private RedisTemplate redisTemplate;

      private RedisOperations() {
      }

      public RedisOperations(RedisTemplate redisTemplate) {
          this.redisTemplate = redisTemplate;
      }

      public RedisTemplate redisTemplate() {
          return redisTemplate;
      }
  }
```

### RedisHandler
`RedisHandler`提供了统一操作`RedisTemplate`的入口，对用户来说下层的`RedisTemplate`是谁就不需要关心了
```java
package com.iogogogo.redis.configure.handler;

  import com.iogogogo.redis.configure.util.RedisOperations;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.data.redis.core.RedisTemplate;
  import org.springframework.stereotype.Component;

  /**
   * Created by tao.zeng on 2020-01-10.
   */
  @Component
  public class RedisHandler {

      @Autowired
      private RedisTemplate redisTemplate;

      @Autowired
      private RedisTemplate redisTemplateDB_2;

      private RedisOperations redisOperations, redisDB2Operations;

      public RedisOperations redisOperations() {
          if (redisOperations == null) {
              redisOperations = new RedisOperations(redisTemplate);
          }
          return redisOperations;
      }

      public RedisOperations redisDB2Operations() {
          if (redisDB2Operations == null) {
              redisDB2Operations = new RedisOperations(redisTemplateDB_2);
          }
          return redisDB2Operations;
      }
  }
```

## 最终结果
我们在启动时看一下经过RedisHandler保证的RedisTemplate是否和我们想的一致，修改以后的启动类

```java
package com.iogogogo.redis;

import com.iogogogo.redis.configure.handler.RedisHandler;
import com.iogogogo.redis.configure.util.RedisOperations;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.redis.core.RedisTemplate;

/**
 * Created by tao.zeng on 2020-01-10.
 */
@Slf4j
@SpringBootApplication
public class RedisApplication implements CommandLineRunner {

    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    private RedisTemplate redisTemplateDB_2;

    @Autowired
    private RedisHandler redisHandler;

    public static void main(String[] args) {
        SpringApplication.run(RedisApplication.class, args);
    }

    @Override
    public void run(String... args) {
        log.info("redisTemplate:{}", redisTemplate);
        redisTemplate.opsForValue().set("iogogogo", "redisTemplate save value");


        log.info("redisTemplateDB_2:{}", redisTemplateDB_2);
        redisTemplateDB_2.opsForHash().put("iogogogo", "iogogogo-hash", "redisTemplateDB_2 save value");


        // 通过统一的RedisOperations对Redis进行操作
        // 这里可以自己封装一些常用的方法，这样就能把下层的RedisTemplate进行封装，对外仅仅只是一个RedisOperations而已
        // 当然这里也提供对应的方法获取不同RedisTemplate对象最后的封装实例
        RedisOperations redisOperations = redisHandler.redisOperations();
        RedisOperations redisDB2Operations = redisHandler.redisDB2Operations();

        log.info("redisOperations redisTemplate:{}", redisOperations.redisTemplate());
        log.info("redisDB2Operations redisTemplate:{}", redisDB2Operations.redisTemplate());
    }
}
```
启动日志
```
2020-01-10 23:54:39.579  INFO 66673 --- [           main] com.iogogogo.redis.RedisApplication      : Started RedisApplication in 8.672 seconds (JVM running for 10.951)
2020-01-10 23:54:39.580  INFO 66673 --- [           main] com.iogogogo.redis.RedisApplication      : redisTemplate:org.springframework.data.redis.core.RedisTemplate@44924587
2020-01-10 23:54:39.679  INFO 66673 --- [           main] io.lettuce.core.EpollProvider            : Starting without optional epoll library
2020-01-10 23:54:39.682  INFO 66673 --- [           main] io.lettuce.core.KqueueProvider           : Starting without optional kqueue library
2020-01-10 23:54:39.910  INFO 66673 --- [           main] com.iogogogo.redis.RedisApplication      : redisTemplateDB2:org.springframework.data.redis.core.RedisTemplate@7fb66650
2020-01-10 23:54:39.927  INFO 66673 --- [           main] com.iogogogo.redis.RedisApplication      : redisOperations redisTemplate:org.springframework.data.redis.core.RedisTemplate@44924587
2020-01-10 23:54:39.929  INFO 66673 --- [           main] com.iogogogo.redis.RedisApplication      : redisDB2Operations redisTemplate:org.springframework.data.redis.core.RedisTemplate@7fb66650
```
通过日志我们可以看到
1. `redisTemplate`和`redisOperations`` redisTemplate`是同一个对象

2. `redisTemplateDB2` 和`redisDB2Operations`` redisTemplate`也是同一个对象

## 总结
通过本文，我们可以看到一个项目如果需要使用使用多个redis连接获取使用不同的数据库，完全可以使用自定义`RedisConnectionFactory`来完成，第二点就是如果定义的`RedisTemplate`过多，我们可以在上面定义一个`RedisHandler`来进行封装下层的API操作，暴露一个统一的入口进行简化处理。

示例代码：[https://github.com/iogogogo/life-example/tree/master/example-redis](https://github.com/iogogogo/life-example/tree/master/example-redis)
