---
categories:
  - code
date: 2020-04-14 20:17:59
tags:
  - springcloud
  - registry
  - microservice
title: spring-cloud中服务自注册的相关类
---

服务拆分后离不开注册，要不然就得手工维护每个实例所在的地址以及监听的端口，当服务一多，手工维护肯定不现实。

一般来讲有两种选择。

- Self registration pattern
- 3rd party registration pattern 

下面我们用一个图，示例下Eureka或Nacos的相关类。

![Channel inherit tree](/images/sc_register.png)