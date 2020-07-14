---
title: 一个普通的日常bug
date: 2019-07-19 21:14:27
tags: bug
---
# 一个普通的日常bug（ Failed to configure a DataSource: 'url' attribute is not specified ）

> SpringBoot 2.0 报错: Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

<!--more-->

## bug日志

**关键词：** Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

**关键词：** Reason: Failed to determine a suitable driver class.

```

2019-07-15 11:11:27.857  INFO 7604 --- [           main] com.yiring.zjtq.Application              : Starting Application on myhbase with PID 7604 (E:\idea-workspace\zjtq-web\out\production\classes started by Administrator in E:\idea-workspace\zjtq-web)
2019-07-15 11:11:27.867  INFO 7604 --- [           main] com.yiring.zjtq.Application              : The following profiles are active: prod
2019-07-15 11:11:27.922  INFO 7604 --- [           main] ConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@68567e20: startup date [Mon Jul 15 11:11:27 CST 2019]; root of context hierarchy
2019-07-15 11:11:28.667  INFO 7604 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Multiple Spring Data modules found, entering strict repository configuration mode!
2019-07-15 11:11:28.689  INFO 7604 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Multiple Spring Data modules found, entering strict repository configuration mode!
2019-07-15 11:11:29.081  INFO 7604 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration$$EnhancerBySpringCGLIB$$5d98e995] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2019-07-15 11:11:29.543  INFO 7604 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2019-07-15 11:11:29.578  INFO 7604 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-07-15 11:11:29.578  INFO 7604 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.32
2019-07-15 11:11:29.583  INFO 7604 --- [ost-startStop-1] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [C:\Program Files\Java\jdk1.8.0_161\bin;C:\Windows\Sun\Java\bin;C:\Windows\system32;C:\Windows;C:\ProgramData\Oracle\Java\javapath;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Program Files\TortoiseSVN\bin;E:\apache-maven-3.5.2\bin;C:\Program Files (x86)\OpenGrADS\Contents\Cygwin\Versions\2.0.a9.oga.1\i686;C:\Program Files\Git\cmd;C:\Users\Administrator\.gradle\wrapper\dists\gradle-5.2.1-all\bviwmvmbexq6idcscbicws5me\gradle-5.2.1\bin;D:\nodejs\;C:\Program Files\TortoiseGit\bin;C:\Program Files\Microsoft VS Code\bin;D:\hadoop\hadoop-3.1.1\bin;C:\Program Files\Java\jdk1.8.0_161\bin;C:\Program Files\Java\jdk1.8.0_161\jre\bin;D:\hadoop\hbase-2.1.5\bin;C:\Program Files\Microsoft VS Code\bin;C:\Users\Administrator\AppData\Roaming\npm;C:\Users\Administrator\AppData\Local\Programs\Fiddler;C:\Users\Administrator\AppData\Local\Yarn\bin;F:\docker\Docker Toolbox;.]
2019-07-15 11:11:29.749  INFO 7604 --- [ost-startStop-1] o.a.c.c.C.[.[localhost].[/zjtq/v3]       : Initializing Spring embedded WebApplicationContext
2019-07-15 11:11:29.750  INFO 7604 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1828 ms
2019-07-15 11:11:29.862  INFO 7604 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2019-07-15 11:11:29.863  INFO 7604 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2019-07-15 11:11:29.863  INFO 7604 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2019-07-15 11:11:29.863  INFO 7604 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2019-07-15 11:11:29.864  INFO 7604 --- [ost-startStop-1] .s.DelegatingFilterProxyRegistrationBean : Mapping filter: 'springSecurityFilterChain' to: [/*]
2019-07-15 11:11:29.864  INFO 7604 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'webStatFilter' to urls: [/*]
2019-07-15 11:11:29.864  INFO 7604 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Servlet dispatcherServlet mapped to [/]
2019-07-15 11:11:29.865  INFO 7604 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Servlet statViewServlet mapped to [/druid/*]
2019-07-15 11:11:29.900  INFO 7604 --- [           main] c.a.d.s.b.a.DruidDataSourceAutoConfigure : Init DruidDataSource
2019-07-15 11:11:30.037  WARN 7604 --- [           main] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaConfiguration': Unsatisfied dependency expressed through constructor parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dataSource' defined in class path resource [com/alibaba/druid/spring/boot/autoconfigure/DruidDataSourceAutoConfigure.class]: Invocation of init method failed; nested exception is org.springframework.boot.autoconfigure.jdbc.DataSourceProperties$DataSourceBeanCreationException: Failed to determine a suitable driver class
2019-07-15 11:11:30.039  INFO 7604 --- [           main] o.apache.catalina.core.StandardService   : Stopping service [Tomcat]
2019-07-15 11:11:30.074  INFO 7604 --- [           main] ConditionEvaluationReportLoggingListener : 

Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.
2019-07-15 11:11:30.080 ERROR 7604 --- [           main] o.s.b.d.LoggingFailureAnalysisReporter   : 

***************************
APPLICATION FAILED TO START
***************************

Description:

Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class


Action:

Consider the following:
	If you want an embedded database (H2, HSQL or Derby), please put it on the classpath.
	If you have database settings to be loaded from a particular profile you may need to activate it (no profiles are currently active).


Process finished with exit code 1

```

## 尝试解决

首先使出一招百度一下，搜到了很多的同类报错解决方法。以下四种方法出自[https://www.jianshu.com/p/836d455663da](https://www.jianshu.com/p/836d455663da)

### 方案一：
排除此类的autoconfig。启动以后就可以正常运行。
```
@SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})

```

### 方案二：
在application.properties/或者application.yml文件中没有添加数据库配置信息.

```
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/read_data?useUnicode=true&characterEncoding=UTF-8&useSSL=false
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
```

### 方案三：
在spring xml配置文件中引用了数据库地址 所以需要对:等进行转义处理.但是在application.properties/或者application.yml文件并不需要转义,错误和正确方法写在下面了.

```
//错误示例
spring.datasource.url = jdbc:mysql\://192.168.0.20\:1504/f_me?setUnicode=true&characterEncoding=utf8
```

```
//正确示例
spring.datasource.url = jdbc:mysql://192.168.0.20:1504/f_me?setUnicode=true&characterEncoding=utf8
```
### 方案四：

yml或者properties文件没有被扫描到,需要在pom文件中<build></build>添加如下.来保证文件都能正常被扫描到并且加载成功.
```
<!-- 如果不添加此节点mybatis的mapper.xml文件都会被漏掉。 -->
<resources>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.yml</include>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
    </resource>
    <resource>
        <directory>src/main/resources</directory>
        <includes>
            <include>**/*.yml</include>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
    </resource>
</resources>

```

### 方案五（最终解决版）：
问题是出在了新导入jar包时，出现报错。
```
repositories {
    maven { url 'http://download.osgeo.org/webdav/geotools/' }
    maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    maven { url 'https://repo.spring.io/libs-snapshot' }
    maven { url 'https://repo.spring.io/milestone' }
    mavenLocal()
    mavenCentral()
}
```
![](b2.png)

此时的setting界面为：
![](b3.png)

经过查询，找到下图的这个选项javac。将其修改为eclipse后，便解决了这个错误。
![](b1.png)

### 方案六（问题再次诞生过程）：
项目在导入jar包前可以正常运行。导入jar包为：
```
repositories {
    maven { url 'http://download.osgeo.org/webdav/geotools/' }
    maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    maven { url 'https://repo.spring.io/libs-snapshot' }
    maven { url 'https://repo.spring.io/milestone' }
    mavenLocal()
    mavenCentral()
}
```

```
dependencies {
    compile('org.springframework.boot:spring-boot-starter-cache')
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('org.springframework.boot:spring-boot-starter-jdbc')
    compile('org.springframework.boot:spring-boot-starter-data-redis')
    compile('org.springframework.boot:spring-boot-starter-security')
    compile('org.springframework.boot:spring-boot-starter-validation')
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.springframework.boot:spring-boot-starter-websocket')


    runtime('mysql:mysql-connector-java')
    compileOnly('org.projectlombok:lombok')
    compileOnly('org.springframework.boot:spring-boot-configuration-processor')
    asciidoctor('org.springframework.restdocs:spring-restdocs-asciidoctor')
    testCompile('org.springframework.restdocs:spring-restdocs-mockmvc')
    testCompile('org.springframework.security:spring-security-test')
    testCompile('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'junit', module: 'junit;'
    }
    testCompile('org.testng:testng:6.14.3')

    compile('com.alibaba:druid-spring-boot-starter:1.1.10')
    compile('org.apache.httpcomponents:httpclient:4.5.6')
    compile('org.apache.httpcomponents:httpmime:4.5.6')
    compile('com.google.guava:guava:26.0-jre')
    compile('commons-net:commons-net:3.6')
    compile('com.alibaba:fastjson:1.2.49')
    compile('io.jsonwebtoken:jjwt:0.9.1')
    compile('com.aliyun:aliyun-java-sdk-core:4.0.8')
    compile('com.aliyun:aliyun-java-sdk-dysmsapi:1.1.0')
    compile('jcifs:jcifs:1.3.17') {
        exclude module: 'servlet-api'
    }
    compile fileTree(dir: 'lib', includes: ['*jar'])

    compile ('commons-fileupload:commons-fileupload:1.3.1')

    compile ('org.apache.commons:commons-pool2:2.6.0')

	//新导入jar包开始
    implementation group: 'org.geotools', name: 'gt-shapefile', version: '21.0'
    implementation group: 'org.geotools', name: 'gt-geojson', version: '21.0'
    implementation group: 'org.meteothink', name: 'wContour', version: '1.6.1'
    implementation group: 'com.vividsolutions', name: 'jts', version: '1.13'
    implementation group: 'org.locationtech.jts', name: 'jts-core', version: '1.16.1'
    //新导入jar包结束

}
```

初步怀疑是新jar包和之前的jar包冲突，或者是新导入jar包中有错误。方案五的解决办法在这种操作场景下并不能解决问题了。

最终删除后重新git导入了整个项目，删除了gradle全局配置。重新导入一遍这个jar包，选择了一下idea->setting->Build,Execution,Deployment->complier->java Compiler中的use compiler为javac（在导入时注意提示。）

## 解决bug途中产生的新bug

**关键字：**错误: 找不到或无法加载主类 com.yiring.zjtq.Application
当edit configurations显示下面的错误时，可以重新rebuild一下项目。或者更改一下jar包管理文件，删除jar包再导入。
```
Run Configuration Error: Not a valid Spring Boot application class
```
![](b4.png)