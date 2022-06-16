---
title: SpringBoot 3.0 预览
date: 2022-03-02
tags: [springboot, spring, java]
---

# Spring Boot 3.0 预览

1 月 20 日, Spring 发布了新的 Spring Boot 3.0 里程碑版本 Spring Boot 3.0.0-M1 , 并预计每两月发布一个新的里程碑版本, 11月下旬发布 GA 版本

>https://spring.io/blog/2022/01/20/spring-boot-3-0-0-m1-is-now-available

## 依赖版本变动

> 这听起来虽然有些激进，但请注意，我们讨论的是 2022 年第四季度的发布。到那时，JDK 17 早已取代 JDK 11 成为下一个长期支持版本一年多了，而且它本身也将被 JDK 18 和 JDK  19 所取代，作为特性发布版本，它们已经可用了，而且 JDK 20 也接近其功能冻结期。
>
> 对于 Jakarta EE 9 也是如此。我们预计 Jakarta  EE 10 届时业已发布，并且新一代的 Tomcat、Jetty 和其他的运行时方案将会支持它们。保持上述最低限度的基线，可以在 Spring  Framework 6.x 一代中获得进一步的 Java 进化，而 Java 17 和 Jakarta EE 9 只是一个开始。

### Java 8 → Java 17

java 17 是 2021.09.14 推出的新 LTS 版本 (Long Term Support) 

### Java EE → Jakarta EE

Java EE 在几年前被 Eclipse 基金会改名为 Jakarta EE , 重命名了很多规范

- 需要将命名空间 javax 改为 jakarta
- 一些依赖Java EE API的第三方库，目前还没有得到很好的支持，在Spring Boot 3中暂时会先移除
  ( 见 [暂时移除的依赖](#暂时移除的依赖) )

### Spring Framework 5.3 → 6.0

预计于 2022.08 发布第一个 GA 版本

#### 关注点: 

排除点和变更点

- 可能 XML 配置格式会成为过去式。
- 一些 Java EE API（ EJB、JCA、JAX-WS）过期。
- RPC 支持（不知道怎么翻译 HTTP Invoker ）过期

迁移至Jakarta EE 9+

- `jakarta.servlet`(Tomcat 10、Jetty 11相关)。
- `jakarta.persistence`(Hibernate ORM 6?)。

云原生

改进对**GraalVM**和**Project Leyden**(一个Java静态图项目)的支持。

#### M1 版本特性: 

- 关于 Commons FileUpload and Tiles, 和 FreeMarker JSP 的集成将被移除

- @RequestMapping without @Controller registered as handler

  > https://github.com/spring-projects/spring-framework/issues/22154

- HttpMethod 从枚举升级为一个类

  > https://github.com/spring-projects/spring-framework/issues/27697

> 更多新特性: https://github.com/spring-projects/spring-framework/releases/tag/v6.0.0-M1

### 其他依赖版本升级

Spring Boot 3.0.0-M1 moves to new versions of several Spring projects:

- Micrometer 2.0.0-M1
- Spring AMQP 3.0.0-M1
- Spring Batch 5.0.0-M1
- Spring Data 2022.0.0-M1
- [Spring Framework 6.0.0-M2](https://github.com/spring-projects/spring-framework/releases/tag/v6.0.0-M2)
- Spring Integration 6.0.0-M1
- [Spring HATEOAS 2.0.0-M1](https://github.com/spring-projects/spring-hateoas/releases/tag/2.0.0-M1)
- Spring Kafka 3.0.0-M1
- [Spring LDAP 3.0.0-M1](https://github.com/spring-projects/spring-ldap/releases/tag/3.0.0-M1)
- [Spring REST Docs 3.0.0-M1](https://github.com/spring-projects/spring-restdocs/releases/tag/v3.0.0-M1)
  - 在 M2 版本中升级为 Spring REST Docs 3.0.0-M2
- [Spring Security 6.0.0-M1](https://github.com/spring-projects/spring-security/releases/tag/6.0.0-M1)
- Spring Session 2022.0.0-M1
- Spring Web Services 4.0.0-M1

Numerous third-party dependencies have also been updated, some of the more noteworthy of which are the following:

- Artemis 2.20.0
- Hazelcast 5.0
- Hibernate Validator 7.0
- Jakarta Activation 2.0
- Jakarta Annotation 2.0
- Jakarta JMS 3.0
- Jakarta JSON 2.0
- Jakarta JSON Bind 3.0
- Jakarta Mail 2.0
- Jakarta Persistence 3.0
- Jakarta Servlet 5.0
- Jakarta Servlet JSP JSTL 2.0
- Jakarta Transaction 2.0
- Jakarta Validation 3.0
- Jakarta WebSocket 2.0
- Jakarta WS RS 3.0
- Jakarta XML Bind 3.0
- Jakarta XML Soap 2.0
- Jetty 11
- jOOQ 3.16
- Tomcat 10

## 暂时移除的依赖

- EhCache 3 
- H2’s web console
  - 已在 M2 版本中恢复
- Hibernate’s metrics
- Infinispan
- Jolokia
- Pooled JMS
- REST Assured
  - 已在 M2 版本中恢复: REST Assured 4.5
- SMTP appending with Logback
- SMTP appending with Log4j 2

## Reference

> - Spring Boot 3.0.0 M1 Release Notes : https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0.0-M1-Release-Notes
> - Spring Boot 3.0.0 M2 Release Notes : https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0.0-M2-Release-Notes
> - Upgrading to Spring Framework 6.x : https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-6.x
> - Spring Framework v6.0.0-M1 : https://github.com/spring-projects/spring-framework/releases/tag/v6.0.0-M1
> - Spring Framework v6.0.0-M2 : https://github.com/spring-projects/spring-framework/releases/tag/v6.0.0-M2
> - Spring Framework 6 将使用 Java 17 和 Jakarta EE 9 作为基线版本 : https://www.infoq.cn/article/4JBuzJI4sbFdoYHfF3h8
> - Java 近期新闻综述 : https://www.infoq.cn/article/1c0wxeozk33n3ki36gzb

