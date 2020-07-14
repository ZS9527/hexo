---
title: 使用geotools时遇到的NoClassDefFoundError
date: 2019-07-26 22:33:59
tags: bug
---

# 使用geotools时遇到的NoClassDefFoundError
> 这个问题很常见，但是并没有用常见的解决方式去解决。再次记录一下

<!--more-->

## 错误信息
**关键字：**Handler dispatch failed; nested exception is java.lang.NoClassDefFoundError: org/apache/commons/lang3/JavaVersion

```

org.springframework.web.util.NestedServletException: Handler dispatch failed; nested exception is java.lang.NoClassDefFoundError: org/apache/commons/lang3/JavaVersion
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1006) ~[spring-webmvc-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:925) ~[spring-webmvc-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:974) [spring-webmvc-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:866) [spring-webmvc-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:635) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:851) [spring-webmvc-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:742) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52) [tomcat-embed-websocket-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at com.yiring.zjtq.secruity.SecurityFilter.invoke(SecurityFilter.java:58) [classes/:na]
	at com.yiring.zjtq.secruity.SecurityFilter.doFilter(SecurityFilter.java:37) [classes/:na]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:101) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at com.alibaba.druid.support.http.WebStatFilter.doFilter(WebStatFilter.java:123) [druid-1.1.10.jar:1.1.10]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:320) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.invoke(FilterSecurityInterceptor.java:127) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.doFilter(FilterSecurityInterceptor.java:91) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at com.yiring.zjtq.secruity.SecurityFilter.invoke(SecurityFilter.java:58) [classes/:na]
	at com.yiring.zjtq.secruity.SecurityFilter.doFilter(SecurityFilter.java:37) [classes/:na]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.java:119) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.authentication.AnonymousAuthenticationFilter.doFilter(AnonymousAuthenticationFilter.java:111) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter.doFilter(SecurityContextHolderAwareRequestFilter.java:170) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.savedrequest.RequestCacheAwareFilter.doFilter(RequestCacheAwareFilter.java:63) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.authentication.www.BasicAuthenticationFilter.doFilterInternal(BasicAuthenticationFilter.java:158) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at com.yiring.zjtq.secruity.JwtAuthenticationTokenFilter.doFilterInternal(JwtAuthenticationTokenFilter.java:62) [classes/:na]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:116) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.web.filter.CorsFilter.doFilterInternal(CorsFilter.java:96) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.header.HeaderWriterFilter.doFilterInternal(HeaderWriterFilter.java:66) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter.doFilterInternal(WebAsyncManagerIntegrationFilter.java:56) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:215) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:178) [spring-security-web-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:357) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:270) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.springframework.web.filter.HttpPutFormContentFilter.doFilterInternal(HttpPutFormContentFilter.java:109) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:93) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:198) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:493) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:140) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:81) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:87) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:342) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:800) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:800) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1471) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_161]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_161]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-8.5.32.jar:8.5.32]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_161]
Caused by: java.lang.NoClassDefFoundError: org/apache/commons/lang3/JavaVersion
	at org.geotools.util.NIOUtilities.clean(NIOUtilities.java:207) ~[gt-metadata-21.0.jar:na]
	at org.geotools.util.NIOUtilities.clean(NIOUtilities.java:185) ~[gt-metadata-21.0.jar:na]
	at org.geotools.data.shapefile.dbf.DbaseFileReader.close(DbaseFileReader.java:300) ~[gt-shapefile-21.0.jar:na]
	at org.geotools.data.shapefile.ShapefileFeatureSource.readAttributes(ShapefileFeatureSource.java:586) ~[gt-shapefile-21.0.jar:na]
	at org.geotools.data.shapefile.ShapefileFeatureSource.buildFeatureType(ShapefileFeatureSource.java:479) ~[gt-shapefile-21.0.jar:na]
	at org.geotools.data.shapefile.ShapefileFeatureStore.buildFeatureType(ShapefileFeatureStore.java:137) ~[gt-shapefile-21.0.jar:na]
	at org.geotools.data.store.ContentFeatureSource.getAbsoluteSchema(ContentFeatureSource.java:327) ~[gt-main-21.0.jar:na]
	at org.geotools.data.store.ContentFeatureSource.getSchema(ContentFeatureSource.java:296) ~[gt-main-21.0.jar:na]
	at org.geotools.data.store.ContentFeatureCollection.<init>(ContentFeatureCollection.java:69) ~[gt-main-21.0.jar:na]
	at org.geotools.data.store.ContentFeatureSource.getFeatures(ContentFeatureSource.java:545) ~[gt-main-21.0.jar:na]
	at org.geotools.data.store.ContentFeatureSource.getFeatures(ContentFeatureSource.java:105) ~[gt-main-21.0.jar:na]
	at com.yiring.zjtq.util.GeoUtils.isoLinesCrop(GeoUtils.java:217) ~[classes/:na]
	at com.yiring.zjtq.service.currentmonitor.impl.RainAndTemLiveServiceImpl.getPreTotalStainMap(RainAndTemLiveServiceImpl.java:144) ~[classes/:na]
	at com.yiring.zjtq.service.currentmonitor.impl.RainAndTemLiveServiceImpl$$FastClassBySpringCGLIB$$b4c107d6.invoke(<generated>) ~[classes/:na]
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204) ~[spring-core-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:684) ~[spring-aop-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at com.yiring.zjtq.service.currentmonitor.impl.RainAndTemLiveServiceImpl$$EnhancerBySpringCGLIB$$598b587c.getPreTotalStainMap(<generated>) ~[classes/:na]
	at com.yiring.zjtq.web.currentmonitor.RainAndTemLiveController.stainMap(RainAndTemLiveController.java:33) ~[classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_161]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_161]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_161]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_161]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:209) ~[spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:136) ~[spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:102) ~[spring-webmvc-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:877) ~[spring-webmvc-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:783) ~[spring-webmvc-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) ~[spring-webmvc-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:991) ~[spring-webmvc-5.0.8.RELEASE.jar:5.0.8.RELEASE]
	... 91 common frames omitted

Disconnected from the target VM, address: '127.0.0.1:58378', transport: 'socket'
2019-0
```

## 解决过程
百度搜索时并没有找到javaVersion这个类报错的解决方法。秉着同类问题的心思去实验了一下第一页的解决方法，并没成功。接着就是找冲突jar包。

### 冲突jar包寻找过程
这个项目运用的是gradle管理，使用dependencies导出jar包结构。确定只有gt-shapefile和gt-geojson引入了commons-lang3

尝试屏蔽commons-lang3并换成自己的版本
```
compile (group: 'org.geotools', name: 'gt-shapefile', version: '21.0'){
        exclude module: 'commons-lang3'
}

compile (group: 'org.geotools', name: 'gt-geojson', version: '21.0'){
        exclude module: 'commons-lang3'
}
compile group: 'org.apache.commons', name: 'commons-lang3', version: '3.9'

```
不成功后，导入不同版本（3.9->3.8.1->3.7）的jar包，并且去api查看是否是新的jar包更新了这个枚举类。依旧失败。

呼叫大佬帮助后，大佬修改了jar包下载地址的顺序，重新导入的时候不勾选下图的那个选项，然后就解决了。应该是之前下载的时候下错了jar包
![](b2.png)
