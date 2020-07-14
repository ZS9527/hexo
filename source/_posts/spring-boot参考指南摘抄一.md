---
title: spring boot参考指南摘抄
date: 2020-01-18 14:33:44
tags: Spring boot
---

# spring boot参考指南摘抄一
> 记录摘抄自己阅读文档的部分内容，方便日后记忆。
> 内容有 @RestController和@RequestMapping注解、@EnableAutoConfiguration注解、Starters依赖、放置应用的main类、 自动配置、  Spring Beans和依赖注入、使用@SpringBootApplication注解、Application事件和监听器、使用ApplicationRunner或CommandLineRunner、外部化配置、访问命令行属性、Application属性文件、 Profiles。

<!--more-->
## @RestController和@RequestMapping注解（来自11.3.1）
Example类上使用的第一个注解是 **@RestController ** ，这被称为构造型（stereotype）注解。它为阅读代码的人提供暗示（这是一个支持REST的控制器），对于Spring，该类扮演了一个特角色。在本示例中，我们的类是一个web@Controller  ，所以当web请求进来时，Spring会考虑是否使用它来处理。

**@RequestMapping ** 注解提供路由信息，它告诉Spring任何来自"/"路径的HTTP请求都应该被映射到 home  方法。 @RestController  注解告诉Spring以字符串的形式渲染结果，并直接返回给调用者。

注： @RestController  和 @RequestMapping  是Spring MVC中的注解（它们不是Spring Boot的特定部分）

## @EnableAutoConfiguration注解（来自11.3.2）
第二个类级别的注解是 **@EnableAutoConfiguration**  ，这个注解告诉Spring Boot根据添加的jar依赖猜测你想如何配置Spring。由于 spring-boot-starter-web  添加了Tomcat和Spring MVC，所以auto-configuration将假定你正在开发一个web应用，并对Spring进行相应地设置。

## Starters（来自13.5）
**Starters**是一个依赖描述符的集合，你可以将它包含进项目中，这样添加依赖就非常方便。你可以获取所有Spring及相关技术的一站式服务，而不需要翻阅示例代码，拷贝粘贴大量的依赖描述符。例如，如果你想使用Spring和JPA进行数据库访问，只需要在项目中包含 **spring-boot-starter-data-jpa **依赖，然后你就可以开始了。

该starters包含很多搭建，快速运行项目所需的依赖，并提供一致的，可管理传递性
的依赖集。

## 放置应用的main类（来自14.2）
通常建议将应用的main类放到其他类所在包的顶层**(root package)**，并将 @EnableAutoConfiguration  注解到你的main类上，这样就隐式地定义了一个基础的包搜索路径（search package），以搜索某些特定的注解实体（比如@Service，@Component等） 。例如，如果你正在编写一个JPA应用，Spring将搜索 @EnableAutoConfiguration  注解的类所在包下的 @Entity  实体。

采用root package方式，你就可以使用 @ComponentScan  注解而不需要指定 basePackage  属性，也可以使用 @SpringBootApplication  注解，只要将main类放到root package中。

## 自动配置（来自16）
Spring Boot自动配置（auto-configuration）尝试根据添加的jar依赖自动配置你的
Spring应用。

例如，如果classpath下存在 HSQLDB  ，并且你没有手动配置任何数据库连接的beans，那么Spring Boot将自动配置一个内存型（in-memory）数据库。

实现自动配置有两种可选方式，分别是将 **@EnableAutoConfiguration**  或 @SpringBootApplication 注解到 @Configuration  类上。

注：你应该只添加一个 @EnableAutoConfiguration  注解，通常建议将它添加到
主配置类（primary  @Configuration  ）上。

##  Spring Beans和依赖注入（来自17）
你可以自由地使用任何标准的Spring框架技术去定义beans和它们注入的依赖。简单起见，我们经常使用 **@ComponentScan ** 注解搜索beans，并结合 @Autowired  构造器注入。

如果遵循以上的建议组织代码结构（将应用的main类放到包的最上层，即rootpackage），那么你就可以添加 @ComponentScan  注解而不需要任何参数，所有应用组件（ @Component  ,  @Service  ,  @Repository  ,  @Controller  等）都会自动注册成Spring Beans。

## 使用@SpringBootApplication注解
很多Spring Boot开发者经常使用 @Configuration  ， @EnableAutoConfiguration  ， @ComponentScan  注解他们的main类，由于这些注解如此频繁地一块使用（特别是遵循以上最佳实践的时候），Spring Boot就提供了一个方便的** @SpringBootApplication  **注解作为代替。

@SpringBootApplication  注解等价于以默认属性使用 @Configuration  ， @EnableAutoConfiguration  和 @ComponentScan 。

注：  @SpringBootApplication  注解也提供了用于自定义 @EnableAutoConfiguration  和 @ComponentScan  属性的别名（aliases）。

## Application事件和监听器（来自23.5）
除了常见的Spring框架事件，比如ContextRefreshedEvent， SpringApplication  也会发送其他的application事件。

注： 有些事件实际上是在 ApplicationContext  创建前触发的，所以你不能在那些事件（处理类）中通过 @Bean  注册监听器，只能通过 SpringApplication.addListeners(…)  或 SpringApplicationBuilder.listeners(…)  方法注册。如果想让监听器自动注册，而不关心应用的创建方式，你可以在工程中添加一个 META-INF/spring.factories  文件，并使用 org.springframework.context.ApplicationListener  作为key指向那些监听器。

应用运行时，事件会以下面的次序发送：
1. 在运行开始，但除了监听器注册和初始化以外的任何处理之前，会发送一
个 ApplicationStartedEvent  。
2. 在Environment将被用于已知的上下文，但在上下文被创建前，会发送一
个 ApplicationEnvironmentPreparedEvent  。
3. 在refresh开始前，但在bean定义已被加载后，会发送一
个 ApplicationPreparedEvent  。
4. 在refresh之后，相关的回调处理完，会发送一个 ApplicationReadyEvent  ，
表示应用准备好接收请求了。
5. 启动过程中如果出现异常，会发送一个 ApplicationFailedEvent  。

注： 通常不需要使用application事件，但知道它们的存在是有用的（在某些场合可能
会使用到），比如，在Spring Boot内部会使用事件处理各种任务。

> 这里需要单独写一篇博客来记录一下，现在还不是太明白这个监听事件的过程。监听的启动顺序，事件的启动顺序，以及这里的一些类的用处。

## 使用ApplicationRunner或CommandLineRunner（来自23.8）
如果需要在 SpringApplication  启动后执行一些特殊的代码，你可以实现 ApplicationRunner  或 CommandLineRunner  接口，这两个接口工作方式相同，都只提供单一的 run  方法，该方法仅在 SpringApplication.run(…)  完成之前调用。

CommandLineRunner  接口能够访问string数组类型的应用参数，而 ApplicationRunner  使用的是上面描述过的 ApplicationArguments  接口：
```
import org.springframework.boot.*
import org.springframework.stereotype.*

@Component
public class MyBean implements CommandLineRunner {
    public void run(String... args) {
    // Do something...
    }
}
```

## 外部化配置（来自24）
Spring Boot设计了一个非常特别的 PropertySource  顺序，以允许对属性值进行
合理的覆盖，属性会以如下的优先级顺序进行设值：
1. home目录下的devtools全局设置属性（ ~/.spring-boot-
devtools.properties  ，如果devtools激活）。
2. 测试用例上的@TestPropertySource注解。
3. 测试用例上的@SpringBootTest#properties注解。
4. 命令行参数
5. 来自 SPRING_APPLICATION_JSON  的属性（环境变量或系统属性中内嵌的内联
JSON）。
6. ServletConfig  初始化参数。
7. ServletContext  初始化参数。
8. 来自于 java:comp/env  的JNDI属性。
9. Java系统属性（System.getProperties()）。
10. 操作系统环境变量。
11. RandomValuePropertySource，只包含 random.\*  中的属性。
12. 没有打进jar包的Profile-specific应用属性（ application-{profile}.properties  和YAML变量）。
13. 打进jar包中的Profile-specific应用属性（ application-{profile}.properties  和YAML变量）。
14. 没有打进jar包的应用配置（ application.properties  和YAML变量）。
15. 打进jar包中的应用配置（ application.properties  和YAML变量）。
16. @Configuration  类上的 @PropertySource  注解。
17. 默认属性（使用 SpringApplication.setDefaultProperties  指定）。

## 访问命令行属性（来自24.2）
默认情况下， SpringApplication  会将所有命令行配置参数（以'--'开头，比如 --server.port=9000  ）转化成一个 property  ，并将其添加到SpringEnvironment  中。正如以上章节提过的，命令行属性总是优先于其他属性源。

## Application属性文件（来自24.3）
SpringApplication  将从以下位置加载 application.properties  文件，并把
它们添加到Spring  Environment  中：
1. 当前目录下的 /config  子目录。
2. 当前目录。
3. classpath下的 /config  包。
4. classpath根路径（root）。
该列表是按优先级排序的（列表中位置高的路径下定义的属性将覆盖位置低的）。

注： 你可以使用YAML（'.yml'）文件替代'.properties'。

如果不喜欢将 application.properties  作为配置文件名，你可以通过指
定 spring.config.name  环境属性来切换其他的名称，也可以使
用 spring.config.location  环境属性引用一个明确的路径（目录位置或文件路
径列表以逗号分割）。

## Profiles（来自25）
Spring Profiles提供了一种隔离应用程序配置的方式，并让这些配置只在特定的环
境下生效。任何 @Component  或 @Configuration  都能注解 @Profile  ，从而限
制加载它的时机：
```
@Configuration
@Profile("production")
public class ProductionConfiguration {
	// ...
}
```