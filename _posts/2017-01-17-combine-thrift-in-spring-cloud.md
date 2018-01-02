---
layout: post
title:  "整合thrift到spring cloud"
date:   2017-01-17 20:17:59
categories: 
- code 
tags:
- sleuth
- spring cloud
- thrift

---
[thrift](http://thrift.apache.org/)是由[facebook](https://github.com/facebook)开源并贡献到[apache](http://www.apache.org/)的一款高效，支持多语言的远程服务调用框架，就像[gRPC](http://www.grpc.io/)，在前面的博文中有对其做一个简单的介绍，包括里面所包含的主要类型。

近一年公司做架构改进，差不多都押宝在了[Spring Cloud](https://github.com/spring-cloud/)上面，但sc中不管三七二十一，统统利用HTTP协议进行服务间的调用，这在面向客户端的网关服务中还可以，但在网关跟其它所谓的内部服务间就想利用rpc来进行调用，由此产生了写一个[thirft-spring-boot-starter](https://github.com/bobxwang/bbspring-thrift-starter)的想法。

服务端利用thrift自带的TServlet，因此主要是添加注解类，仿照**RestController**添加**ThriftController**，然后服务起来后搜索所有的带此注解的类，利用TServlet添加到ServletContext中。

客户端利用thrift自带的THttpClient，同服务端，仿照[Feign](https://github.com/OpenFeign/feign)添加**ThriftClient**注解，但此类是固定的，我们要整合sc，就添加一个类**TLoadBalancerClient**继承**TTransport**，主要目的是利用sc的**LoadBalancerClient**类发现相应的服务集群并做相应负载。同时封装**TServiceClient**利用连接池跟[Hystrix](https://github.com/netflix/hystrix)做断路处理。

当然上面我们只是利用了thrift的二进制编码机制，放弃了它底层NIO传输与服务线程模型。我们也可以利用其它的进行整合，只是这样一来做为服务端就得监听多个端口了，一个是sc自己的端口，一个是thrift的监听端口。

最后我们还可以利用**TMultiplexedProcessor**不同的服务写不同的定义文件，不用全都写在一起。
