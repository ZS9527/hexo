---
title: JPA配置多数据源
date: 2020-12-05 18:15:55
tags: jpa
---
# JPA配置多数据源
> 本文来自多个博文的综合。
> 主要为：[https://blog.csdn.net/blowbob_666/article/details/110670713](https://blog.csdn.net/blowbob_666/article/details/110670713)
> 本文顺带解决一个 Jpa 不自动将驼峰（大写字母）命名转下划线的问题。
> 
<!--more-->

## yml配置
```java
spring:
  datasource:
    druid:
      db:
        primary:
          name: primary
          url: jdbc:mysql://127.0.0.1:3307/primary?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
          username: root
          password: 123456
          initialSize: 5
          minIdle: 5
          maxActive: 20
          maxWait: 60000
          timeBetweenEvictionRunsMillis: 60000
          minEvictableIdleTimeMillis: 60000
          validationQuery: SELECT 1
          validationQueryTimeout: 10000
          testWhileIdle: true
          testOnBorrow: false
          testOnReturn: false
          poolPreparedStatements: true
          maxPoolPreparedStatementPerConnectionSize: 20
          filters: stat,wall
          connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
          useGlobalDataSourceStat: true
        test:
          name: test
          url: jdbc:mysql://127.0.0.1:3308/test?useSSL=false
          username: root
          password: 123456
          initialSize: 5
          minIdle: 5
          maxActive: 20
          maxWait: 60000
          timeBetweenEvictionRunsMillis: 60000
          minEvictableIdleTimeMillis: 60000
          validationQuery: SELECT 1
          validationQueryTimeout: 10000
          testWhileIdle: true
          testOnBorrow: false
          testOnReturn: false
          poolPreparedStatements: true
          maxPoolPreparedStatementPerConnectionSize: 20
          filters: stat,wall
          connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
          useGlobalDataSourceStat: true
  jpa:
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    show-sql: false
    open-in-view: true
    hibernate:
      ddl-auto: update
      primary-dialect: org.hibernate.dialect.MySQL5InnoDBDialect
      test-dialect: org.hibernate.dialect.MySQL5InnoDBDialect

```

## 多数据源配置类
```java
import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;
import javax.sql.DataSource;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.transaction.annotation.EnableTransactionManagement;

/**
 * 数据源管理
 * 1. 自动载入多数据源<br>
 * 2. 支持数据源参数重载<br>
 * <br>
 *
 */

@Configuration
@EnableTransactionManagement
public class DataSourceConfig {

    @Primary
    @Bean(name = "dataSource")
    @ConfigurationProperties(prefix = "spring.datasource.druid.db.primary")
    public DataSource dataSource() {
        return DruidDataSourceBuilder.create().build();
    }

	@Bean(name = "dataSourceTest")
	@ConfigurationProperties(prefix = "spring.datasource.druid.db.test")
	public DataSource dataSourceTest() {
		return DruidDataSourceBuilder.create().build();
	}

    @Primary
    @Bean
    public JdbcTemplate jdbcTemplate(@Qualifier("dataSource") DataSource ds) {
        return new JdbcTemplate(ds);
    }

	@Bean
	public JdbcTemplate jdbcTemplateTest(@Qualifier("dataSourceTest") DataSource ds) {
		return new JdbcTemplate(ds);
	}
}
```

## 主数据源的详细配置（其它数据源类比配置即可，不要加@Primary）包含驼峰转下划线配置

```java
import java.util.HashMap;
import java.util.Map;
import javax.persistence.EntityManager;
import javax.sql.DataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy;
import org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.env.Environment;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

/**
 * 主数据源配置
 *
 */
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    entityManagerFactoryRef="entityManagerFactoryPrimary",
    transactionManagerRef="transactionManagerPrimary",
    basePackages= { "com.yiring.primary.repository" })
public class PrimaryConfig {
    @Autowired
    private Environment env;

    @Autowired
    @Qualifier("dataSource")
    private DataSource primaryDataSource;

    @Primary
    @Bean(name = "entityManagerPrimary")
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return entityManagerFactoryPrimary(builder).getObject().createEntityManager();
    }

    @Primary
    @Bean(name = "entityManagerFactoryPrimary")
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryPrimary (EntityManagerFactoryBuilder builder) {
        return builder
            .dataSource(primaryDataSource)
            .properties(getVendorProperties())
            .packages("com.yiring.primary.domain")
            .persistenceUnit("primaryPersistenceUnit")
            .build();
    }



    private Map<String, String> getVendorProperties() {
        Map<String, String> jpaProperties = new HashMap<>(16);
        jpaProperties.put("hibernate.hbm2ddl.auto", "update");
        jpaProperties.put("hibernate.show_sql", env.getProperty("spring.jpa.show-sql"));
        jpaProperties.put("hibernate.format_sql", env.getProperty("spring.jpa.hibernate.format_sql"));
        jpaProperties.put("hibernate.dialect", env.getProperty("spring.jpa.hibernate.primary-dialect"));
        jpaProperties.put("hibernate.current_session_context_class", "org.springframework.orm.hibernate5.SpringSessionContext");
        //设置驼峰转下划线
        jpaProperties.put("hibernate.physical_naming_strategy", SpringPhysicalNamingStrategy.class.getName());
        jpaProperties.put("hibernate.implicit_naming_strategy", SpringImplicitNamingStrategy.class.getName());
        return jpaProperties;
    }

    @Primary
    @Bean(name = "transactionManagerPrimary")
    public PlatformTransactionManager transactionManagerPrimary(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactoryPrimary(builder).getObject());
    }
}

```

## 副数据源
```java

import java.util.HashMap;
import java.util.Map;
import javax.annotation.Resource;
import javax.persistence.EntityManager;
import javax.sql.DataSource;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy;
import org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

/**
 * 副数据源配置
 *
 * @author zhangshuai
 * @date 2021/2/24 15:45
 */
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    //实体管理
    entityManagerFactoryRef="entityManagerFactorySecond",
    //事务管理
    transactionManagerRef="transactionManagerSecond",
    //实体扫描,设置Repository所在位置
    basePackages= { "com.yiring.test.repository" })
public class ZjtqConfig {

    @Resource
    @Qualifier("dataSourceZjtq")
    private DataSource secondDataSource;

    @Resource
    private Environment env;

    @Bean(name = "entityManagerSecond")
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return entityManagerFactorySecond(builder).getObject().createEntityManager();
    }

    @Bean(name = "entityManagerFactorySecond")
    public LocalContainerEntityManagerFactoryBean entityManagerFactorySecond (EntityManagerFactoryBuilder builder) {
        return builder
            .dataSource(secondDataSource)
            .properties(getVendorProperties())
            .packages("com.yiring.test.domain")
            .persistenceUnit("secondPersistenceUnit")
            .build();
    }

    private Map<String, String> getVendorProperties() {
        Map<String, String> jpaProperties = new HashMap<>(16);
        //这里取消对副数据源的修改
        jpaProperties.put("hibernate.show_sql", env.getProperty("spring.jpa.show-sql"));
        jpaProperties.put("hibernate.dialect", env.getProperty("spring.jpa.hibernate.Second-dialect"));
        jpaProperties.put("hibernate.format_sql", env.getProperty("spring.jpa.hibernate.format_sql"));
        jpaProperties.put("hibernate.current_session_context_class", "org.springframework.orm.hibernate5.SpringSessionContext");
        //设置驼峰转下划线
        jpaProperties.put("hibernate.physical_naming_strategy", SpringPhysicalNamingStrategy.class.getName());
        jpaProperties.put("hibernate.implicit_naming_strategy", SpringImplicitNamingStrategy.class.getName());
        return jpaProperties;
    }

    @Bean(name = "transactionManagerSecond")
    PlatformTransactionManager transactionManagerSecond(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactorySecond(builder).getObject());
    }
}

```
## 总结
通过不同的包路径读取来区分不同的数据源。尽量不要用同名类，否则在使用时注入会报错。
使用方法和平时的单数据源使用方法一样。
