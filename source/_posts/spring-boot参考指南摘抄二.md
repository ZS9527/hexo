---
title: spring boot参考指南摘抄二
date: 2020-02-01 11:16:35
tags: Spring boot
---

# spring boot参考指南摘抄二
> 把要素分开存放。
> 静态内容、

<!--more-->
## 静态内容（来自27.1.5）
默认情况下，Spring Boot从classpath下的 /static  （ /public  ， /resources  或 /META-INF/resources  ）文件夹，或从 ServletContext  根目录提供静态内容。这是通过Spring MVC的 ResourceHttpRequestHandler  实现的，你可以自定义 WebMvcConfigurerAdapter  并覆写 addResourceHandlers  方法来改变该行为（加载静态文件）。

注： 如果你的应用将被打包成jar，那就不要使用 src/main/webapp  文件夹。尽管
该文件夹是通常的标准格式，但它仅在打包成war的情况下起作用，在打包成jar
时，多数构建工具都会默认忽略它。

## CORS支持（来自27.1.10）
从4.2版本开始，Spring MVC对CORS提供开箱即用的支持。不用添加任何特殊配置，只需要在Spring Boot应用的controller方法上注解 @CrossOrigin  ，并添加CORS配置。通过注册一个自定义 addCorsMappings(CorsRegistry)  方法的 WebMvcConfigurer  bean可以指定全局CORS配置：
```
@Configuration
public class MyConfiguration {
	@Bean
	public WebMvcConfigurer corsConfigurer() {
		return new WebMvcConfigurerAdapter() {
			@Override
			public void addCorsMappings(CorsRegistry registry) {
				registry.addMapping("/api/**");
			}
		};
	}
}
```

## 安全（来自28）
如果添加了Spring Security的依赖，那么web应用默认对所有的HTTP路径（也称为终点，端点，表示API的具体网址）使用'basic'认证。为了给web应用添加方法级别（method-level）的保护，你可以添加 @EnableGlobalMethodSecurity  并使用想要的设置。

默认的安全配置是通过 SecurityAutoConfiguration  ， SpringBootWebSecurityConfiguration（用于web安全）， AuthenticationManagerConfiguration  （可用于非web应用的认证配置）进行管理的。你可添加一个 @EnableWebSecurity  bean来彻底关掉Spring Boot的默认配置。为了对它进行自定义，你需要使用外部的属性配置和 WebSecurityConfigurerAdapter  类型的beans（比如，添加基于表单的登陆）。 想要关闭认证管理的配置，你可以添加一个 AuthenticationManager  类型的bean，或在 @Configuration  类的某个方法里注入 AuthenticationManagerBuilder  来配置全局的 AuthenticationManager  。

可以看看[Spring Boot应用示例](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-samples/)

## 实体类（来自29.3.1）
