---
layout: post
title:  "Netflix Eureka 服务发现之手动注册"
date:   2016-04-03 20:17:59
categories: 
- code 
tags:
- scala
- eureka
- finatra

---
[Eureka](https://github.com/Netflix/eureka/)是[netflix](https://github.com/Netflix/)开源出来的一款服务发现注册框架，就像[etcd](https://github.com/coreos/etcd)，在前面关于SpringCloud的文章中有讲述，此文主要想讲下脱离Cloud在其它框架，如[finatra](https://github.com/twitter/finatra/)中如何来使用它。

假设您已经架设好一个eureka服务，然后想把自己的服务注册上去，就像[zookeeper](http://zookeeper.apache.org/)，您需要自己在应用启来的时候告诉eureka，我已经做好准备可以进行服务了，然后在应用退出的时候，同样需要告诉eureka，我准备退出不继续参与服务了。

一般服务必须告诉对方服务地址，端口号，服务名称等等，这在eureka中体现为
<pre>com.netflix.appinfo.EurekaInstanceConfig</pre>上面，默认实现在告诉服务地址的时候是以域名形式告诉eureka，另外也在这里，eureka帮我们定义了状态健康检查路由，如果是spring cloud则由spring-boot-starter-actuator库帮我们实现了，那么在这里我们可以重写它
<pre>class EurekaDataCenterInstanceConfig extends MyDataCenterInstanceConfig {
  /**
   * 由域名改成IP
   * @param refresh
   * @return
   */
  override def getHostName(refresh: Boolean): String = InetAddress.getLocalHost.getHostAddress
  override def getStatusPageUrlPath: String = "/info"
  override def getHealthCheckUrlPath: String = "/health"
} </pre>
然后在应用启动的main方法中添加如下代码
<pre>def initEurekaClient: Unit = {
    val dataCenterInstanceConfig = new EurekaDataCenterInstanceConfig()
    val defaultEurekaClientConfig = new DefaultEurekaClientConfig()
    DiscoveryManager.getInstance().initComponent(
      dataCenterInstanceConfig, defaultEurekaClientConfig)
    ApplicationInfoManager.getInstance.setInstanceStatus(InstanceStatus.UP)
}</pre>
用我们自己写的EurekaDataCenterInstanceConfig类来进行实例化。
最后在应用退出时做如下处理
<pre>def stopEurekaClient: Unit = {
    Runtime.getRuntime.addShutdownHook(new Thread(new Runnable {
      override def run(): Unit = {
        DiscoveryManager.getInstance().shutdownComponent()
      }
    }))
}</pre>
当然服务在跟eureka通信的时候，是根据状态来处理的，上面的代码也可以看出来，这样我们就可以添加自己的处理器来进行服务的治理。
<pre>post("/pause", swagger(o => {
  o.summary("暂停服务").description("注册到Eureka,但不参与后续请求处理").tag("服务治理")
  })) { request: Request =>
    ApplicationInfoManager.getInstance.setInstanceStatus(InstanceStatus.DOWN)
    val builder = response.status(200)
    val map = Map("msg" -> "the service is down")
    builder.json(map)
  }
</pre>
如果想重新进行服务，则将状态重置为UP即可，这是一个枚举类，可以通过查看其定义知道有哪些状态。

最后初始化的时候会读取应用程序目录下的eureka-client.properties，当然这是在默认的情况下，我们可以进行个性，eureka这个在此处是一个namespace，如果您在初始化EurekaDataCenterInstanceConfig传入的参数是abcd，那么应用就会去读取abcd-client.properties这个文件。就是在这个文件里面配置了端口名，eureka服务发现所在地等等信息，一个基本配置可以参考如下。
<pre>eureka.registration.enabled=true
eureka.region=default
eureka.shouldUseDns=false
eureka.decoderName=JacksonJson
eureka.name=serviceName
eureka.port=8080
eureka.serviceUrl.default=http://url/eureka/
eureka.metadata.instanceId=serviceName:8080</pre>