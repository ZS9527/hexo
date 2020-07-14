---
title: 遇到的bug修复
date: 2019-05-08 19:16:07
tags: gradle
---
# 业务中遇到的bug积累之A bean with that name has already been defined in file

> 这个BUG的全部信息为The bean 'jwtAuthenticationTokenFilter', defined in class path resource [config/SecurityConfig.class], could not be registered. A bean with that name has already been defined in file [E:\idea-workspace\-web\out\production\classes\secruity\JwtAuthenticationTokenFilter.class] and overriding is disabled.
> 
<!--more-->

一开始我只是发现了这里是因为bean的重名导致这种现象，并没有找到原因。
    
还好我们团队的技术大牛伸出援手，指出了问题的所在：在修改Spring Boot Gradle Plugin版本之前并没有出现这种状况。在版本2.0.4之前可以有以下这种技术写法：

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {
	//业务代码
}
```

```java
@Bean
public JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter() {
	return new JwtAuthenticationTokenFilter();
}

```

旧版本支持在两个地方同时注册同名的bean，不会报出今天的这个错误。而2.1.4则是修复了这个问题，必须要在这两个bean中做出选择，保留其一。