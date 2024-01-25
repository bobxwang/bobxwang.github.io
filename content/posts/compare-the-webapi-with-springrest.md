---
categories:
  - code
date: 2013-10-09 14:17:59
tags:
  - asp.net webapi
  - spring rest
title: Asp.Net WebApi跟Spring Rest的Controller对比
---

自从11年底接触Asp.net Web Api开始到现在转眼也快三年了，做个日志对比下开源框架Spring Rest。

### Asp.net Web Api Controller Select && Spring Rest Controller Select
众所周知，HttpModule跟HttpHandler在Net Framework中是很重要的两个概念，在Web Api中一样，IIS接收到请求后，会查找一个叫做IRouteHandler的实现类，默认是HttpControllerRouterHander，中间会途径一系列的MessageHandler，HttpControllerDispatcher，DefaultHttpControllerActivator，最终会根据我们事先配置好的路由来选择一个Controller接收这个请求。

基于Spring的web应用最开始我们需要配一个DispatcherServlet的servlet-name，这个servlet会负责接收过来的所有请求，然后根据相应的配置<pre>HandlerMapping
--BeanNameUrlHandlerMapping
--SimpleUrlHandlerMapping
--ControllerClassNameHandlerMapping
--DefaultAnnotationHandlerMapping)</pre>选择一个具体的Controller来接收这个请求。

### Parameter Bind
不管什么平台，客户端过来的数据服务端都会处理成一个NameValueCollection这样一个角色，然后我们自己根据相应的name在这个角色中获得value。不过如果需要我们自己去取，那这样也就谈不上框架了，太Low了点。
##### Get
Asp.net Web Api在配置路由的时候，
```c#
GET("BanKa/GF/Activity.ashx/{user_id?}/{true_name?}/{mobile?}"
```
那我们在相应的处理器中就可以
```c#
public HttpResponseMessage Method(string user_id, string true_name, string mobile)
```
此时这个方法中参数的值框架会给我们自动注入进来。

而在Spring中，框架严格区分@RequestParam跟@PathVariable两种类型，举个例子
```java
@ResponseBody
@RequestMapping(value = "NewPage/{id}/link.ashx", method = RequestMethod.GET)
public Object Qlink(@PathVariable String id,@RequestParam String citycode,@RequestParam(required = false) String callback)
```
如果我们此时访问的地址是: NewPage/13/link.ashx?citycode=1234&callback=abcd，那么上面方法的参数值就是13，1234，abcd
##### Post
这种情况下一般在Asp.net Web Api我们都会定义一个POCO，里面的属性就是NameValueCollection中的Name，框架会帮我们进行序列化，借助的ModelBinder这个类。

而Spring中我们一样，是一个POJO，我们会对它标注上@RequestBody，不过此时一般要求客户端过来的content-type是application/json，如果是application/x-www-form-urlencoded，此时方法参数就应该是MultiValueMap<String, String>的一个实例，同样Spring的这个绑定借助的是FormHttpMessageConverter这个转换类。

### Others
目前Asp.net WebApi所有的Controller都是继承自ApiController，而Spring早期一样，不过目前采用了注解的方式，任何一个类都可以充当Controller的角色，只要给他标识上@Controller或@RestController即可，当然，基于习惯，一般会将这种类以Controller结尾。不过最新版本的Asp.net WebApi据说也会去除强制继承模式了，这样看来，各个语言都是殊途同归啊。