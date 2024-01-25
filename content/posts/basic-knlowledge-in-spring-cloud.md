---
categories:
  - code
date: 2015-06-05 14:17:59
tags:
  - microservice
  - springcloud
  - netflix
title: Spring Cloud--基础概念
---

当今微服务在博客，社交媒体或会议演讲中获得越来越多关注，同时社区中也有不少怀疑论者，认为其并不是很新东西，而是对SOA架构的重新包装。然而微服务架构模式却正在为敏捷部署及复杂企业应用提供巨大的帮助。如想获得微服务的最新消息，可以关注其[官网](http://microservices.io/)。

跨入微服务领域意味着我们正式迎接分布式系统所带来的诸多挑战，而分布式系统绝不是那种能够"凑合使用"的方案。

jvm平台的著名开源库Spring将[spring-boot](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)与[netflix](http://netflix.github.io/)的库相结合，推出了[spring-cloud](http://projects.spring.io/spring-cloud/spring-cloud.html)。

spring-cloud为开发者提供了在分布式系统(如配置服务，服务发现，断路器，智能路由，控制总线，一次性Token，全局锁)等操作的开发工具，目前已经提供在[Maven](http://mvnrepository.com/search?q=spring+cloud)中央仓库中。

spring cloud包含一大堆子项目，已经开源在github上面，可以在[这里](https://github.com/spring-cloud/)查看。

>**spring cloud cli**
>
>**spring cloud context**
>
>**spring cloud commons**
>
>**spring cloud config**
>
>**spring cloud netflix**：service discovery: Eureka; circuit breaker: Hystrix; client side loadbalancer: Ribbon; declarative rest client: Feign; router and filter: Zuul(Reverse Proxy)
>

当然，服务注册发现这块，spring cloud除了netflix公司的Eureka之外，也可以选择consul或者zookeeper。

下文将会对每个子项目做个说明。