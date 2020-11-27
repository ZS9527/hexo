---
title: jar包冲突后的处理过程
date: 2020-09-26 10:07:37
tags: idea
---

# jar包冲突后的处理过程
> 记录一下发现 jar 包冲突后，怎么用 idea 来排查解决
> 

## 错误
```
***************************
APPLICATION FAILED TO START
***************************

Description:

An attempt was made to call a method that does not exist. The attempt was made from the following location:

    org.hibernate.snowflake.GenerateLongId.configure(GenerateLongId.java:40)

The following method did not exist:

    org.hibernate.snowflake.SnowFlakeIdWorker.getInstance()Lorg/hibernate/snowflake/SnowFlakeIdWorker;

The method's class, org.hibernate.snowflake.SnowFlakeIdWorker, is available from the following locations:

    file:/E:/idea-workspace/web-api/out/production/classes/org/hibernate/snowflake/SnowFlakeIdWorker.class
    jar:file:/E:/idea-workspace/web-api/lib/hibernate-comment-annotation-0.0.2-SNAPSHOT.jar!/org/hibernate/snowflake/SnowFlakeIdWorker.class

It was loaded from the following location:

    file:/E:/idea-workspace/web-api/out/production/classes/


Action:

Correct the classpath of your application so that it contains a single, compatible version of org.hibernate.snowflake.SnowFlakeIdWorker
```

通过错误信息我们可以看出来，是引入的 jar 包和我自己本地配置的发生了同类冲突。需要解决的就是找到引入的地方，并且屏蔽。

## idea jar包关联查询
首先说明我用的是 gradle 管理 jar 包。在右侧的 gradle 工具中，可以查看整体的依赖关系图。
其中有 `Tasks->help->dependencies`这个工具。运行后，就会输出 jar 包依赖的树状图。根据树状图就可以找到是那两个 jar 包中的依赖冲突。

![b1.png](b1.png)

## 屏蔽无关jar包
据别的博客说，可用以下这种方式移除
```
compile(configuration: 'core', group: 'toolbox', name: 'web-tools', version: '2.0.+') {
        exclude(module: 'spring')
        exclude(module: 'commons-codec')
        exclude(module: 'slf4j')
        exclude(module: 'servlet-api')
    }
```
但是我用这种方式发现无效，只好用这种全局屏蔽方式移除
```
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
    all*.exclude group:'javax', module: 'javaee-api'
}
```
**注意：**这个方法是因为我是本地导入了一个 jar 包和 gradle 管理的发生了冲突。
成功解决。