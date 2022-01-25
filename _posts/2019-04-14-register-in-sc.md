---
layout: post
title:  "spring-cloud中服务自注册的相关类"
date:   2019-04-14 20:17:59
categories: 
- code 
tags:
- springcloud
- registry
- microservice
---
服务拆分后离不开注册，要不然就得手工维护每个实例所在的地址以及监听的端口，当服务一多，手工维护肯定不现实。

一般来讲有两种选择。

- Self registration pattern
- 3rd party registration pattern 

下面我们用一个图，示例下Eureka或Nacos的相关Client包。

![Channel inherit tree](/media/pic/sc_register.png)
