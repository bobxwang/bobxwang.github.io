---
categories:
  - code
date: 2015-02-03 20:17:59
tags:
  - thrift
title: Apache Thrift 基本类信息
---

[thrift](http://thrift.apache.org/lib/)最初是由Facebook开发并贡献出来的一款PRC库，可以进行跨语言的开发。

# 主要类
* TProtocol 定义传输的是什么内容,客户服务端通过约定的协议来传输消息，编解码
	* TCompactProtocol -- Variable-Length Quantity(VLQ),高效原因之VLQ
	* TBinaryProtocol
	* TJSONProtocol
	* TProtocolDecorator
		* TMultiplexedProtocol
	* TSimpleJSONProtocol
* TTransport 以什么方式来传输，有阻塞跟非阻塞的Tcp/Http实现
	* TmemoryInputTransport
	* AutoExpandingBufferWriterTransport
	* TNonblockingTransport -- 使用非阻塞方式，用于构建异步客户端
		* TNonblockingSocket
	* TFileTransport
	* TSimpleFileTransport
	* TFramedTransport -- 使用非阻塞方式，按块大小传输，类似于Java中的NIO
	* TIOStreamTransport
		* TSocket -- 使用阻塞式I/O传输
		* TZlibTransport 
	* TFastFramedTransport
	* THttpClient
	* AutoExpandingBufferReadTransport
	* TMemoryBuffer
	* TSaslTransport
		* TSaslClientTransport
		* TSaslServerTransport
* TProcessor -- 由编译器编译IDL文件生成
* TServer
	* AbstractNonblockingServer
		* TNonblockingServer -- 多线程服务端使用非阻塞式I/O
			* THsHaServer -- 是一个Half-Sync/Half-Async，即半同步半异步的Server
		* TThreadedSelectorServer
	* TThreadPoolServer -- 多线程服务端使用标准的阻塞式I/O
	* TSimpleServer -- 单线程服务端使用标准阻塞式I/O

# 元数据
* FieldMetaData
* FieldValueMetaData
	* ListMetaData
	* MapMetaData
	* SetMetaData
	* StructMetaData
	* EnumMetaData

# IDL脚本可定义类型
* 基本类型
	* bool
	* byte -- 8位有符号整数，对应java的byte
	* binary -- byte的数组
	* i16 -- 16位有符号整数，对应java的short
	* i32 -- 32位有符号整数，对应java的int
	* i64 -- 64位有符号整数，对应java的long
	* double -- 64位浮点数，对应java的double
	* string -- 未知编码文本或二进制字符串
* 结构体类型
	* struct -- 定义公共类型，即javabean
* 容器类型
	* list 
	* set
	* map
* 异常类型
	* exception 
* 服务类型
	* service -- 对应服务的类
* 其它
	* typedef i32 MyInterger -- 定义别名
	* const i32 AGE_CONST = 1234 -- 定义常量
	* const map<string,string> MAP_CONST = {"hello":"world","goodnight":"moon"}

# 心得体会
通过观察TServer的源码，其中包含有input/output二类TProtocol，体现了数据进来及出去时传输的格式(Binary? Json? ...)；input/output二类Transport，体现了数据进来及出去时如何传输(Socket? File? ...)；另外还有个Processor，其子类是通过IDL文件生成的，运行时必须传递进去具体的实现类信息。这样传递什么数据(what)?用什么方式传输(how)?以及数据如何处理(process)就都有了