---
layout: post
title:  "Netty框架中的几个核心概念"
date:   2013-03-05 14:17:59
categories: 
- code 
tags:
- java
- Netty

--- 
[Netty](http://netty.io/ "netty io") 在java世界中是一个非常知名的通迅框架，目前最新版本应该是5.x,不过3.x版本相对来讲在业界用的比较广泛，主要是从4.x版本开始重写了很多并对项目进行了重新分包。此框架可以让你在不是很熟悉NIO的基础上也可以写出高效的服务端代码。

下面说说在Netty中的几个重要概念。

###Bootstrap
Bootstrap在Netty起到的是一个引导作用，相当于在这上面根据自己的业务场景做好配置。
![Bootstrap inherit tree](/media/pic/netty-bootstrap.PNG)

###Channel
Channel在Netty中是一个通迅载体，我们可以根据自己的业务场景选择它的一个具体类，下图反映的只是其中一小部分。
![Channel inherit tree](/media/pic/netty-channel.PNG)

###ChannelHandler
ChannelHandler在Netty中就是具体的业务逻辑处理实现类，通常我们会新建一个类继承自SimpleChannelInboundHandler，然后在里面通过重写相应的方法来。通过下图可以看出，其实在Netty中各种Decoder/Encoder其实也是一种Handler。
![ChannelHandler inherit tree](/media/pic/netty-channelhandler.PNG)

###ChannelPipeline
从上图中其实我们可以看出，ChannelPipeline的作用就是它会有一个列表，这个列表中存的东西就是我们自己配置进去的各种ChannelHandler的实例，可以说它是一个Netty程序的根本。因为，当有消息到达时，会在这列表中进行一个循环通知让各个handler去处理。

###ChannelInitializer
ChannelInitializer在Netty中是用来配置ChannelHandler的，它通过上面的ChannelPipeline来添加ChannelHandler,ChannelInitializer本身也是一个ChannelHandler,它会在添加完其它handlers后自动从ChannelPipeline中删除自己。

###ByteBuf
既然是通迅框架，肯定离不了数据的传送，在最新版本的Netty中，数据最终都会归为ByteBuf类。同时在框架中也提供了相应的实用类来进行相应的处理。
![Buffer](/media/pic/netty-buf.PNG)

###EventLoopGroup/EventLoop
顾名思义，EventLoopGroup可以包含多个EventLoop，每个Channel会绑定一个EventLoop，很多Channel可能会共享一个EventLoop。我们可以理解EventLoop是一个事件循环线程，而EventLoopGroup是一个事件循环集合。
