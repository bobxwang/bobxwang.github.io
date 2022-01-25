---
layout: post
title:  "spring-cloud中服务发现相关的类"
date:   2020-04-16 20:17:59
categories: 
- code 
tags:
- springcloud
- discovery
- microservice

---

服务拆分后离不开服务间的调用，那么怎么确定你要调用的一个服务，它所在的地址呢。

![Channel inherit tree](/media/pic/discovery-problem.jpeg)

一般来讲这里涉及到服务发现，常见的有如下几种方案。

- Client-side discovery

  ![Channel inherit tree](/media/pic/client-side-discovery.jpeg)

- Server-side discovery

  ![Channel inherit tree](/media/pic/server-side-discovery.jpeg)

在SpringCloud体系中，SC抽象了如下的一个接口

```java
package org.springframework.cloud.client.discovery {
  public interface DiscoveryClient {}
}
```

取决于所采用的注册中心，会有不同的实现，

``` java
public class NacosDiscoveryClient implements DiscoveryClient {}
// 以上是Nacos的实现
public class EurekaDiscoveryClient implements DiscoveryClient {}
// 以上是Eureka的实现
```

下面我们用一个图，示例下Eureka或Nacos的相关类。

![Channel inherit tree](/media/pic/sc_discovery.png)