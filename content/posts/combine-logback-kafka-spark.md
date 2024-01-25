---
categories:
  - code
date: 2016-07-05 20:17:59
tags:
  - spark
  - kafka
  - logback
title: 日志监控整合LogBack+Kafka+Spark
---

[kafka](http://kafka.apache.org)是[linkedin](https://github.com/linkedin)开发并开源出来的一个分布式MQ系统，现在是Apache的一个顶级项目。[spark](http://spark.apache.org/)是一个加州大学伯克利分校的AMP实验室所开源出来的一个类Hadoop MapReduce的通用并行框架。[logback](https://logback.qos.ch/)是继log4j后又一个开源日志组件。关于这三个组件在这不做过多介绍，本文描述的是怎么把它们三个组合在一起。

##### Kafka + Logback

* 首先编写一个接口，目的是把ILoggingEvent类转成字符串
```scala
trait Formatter {
  def format(event: ILoggingEvent): String
}
class JsonFormatter extends Formatter {
  private val includeMethodAndLineNumber: Boolean = false
  private val objectMapper: ObjectMapper = new ObjectMapper
  def format(event: ILoggingEvent): String = {
    val maps: java.util.Map[String, AnyRef] = new java.util.HashMap[String, AnyRef]
    maps.put("level", event.getLevel.levelStr)
    maps.put("logger", event.getLoggerName)
    maps.put("timestamp", event.getTimeStamp)
    maps.putAll(event.getMDCPropertyMap)
    maps.put("message", event.getFormattedMessage)
    if (includeMethodAndLineNumber) {
       val callerDataArray: Array[StackTraceElement] = event.getCallerData
       if (callerDataArray != null && callerDataArray.length > 0) {
          val stackTraceElement: StackTraceElement = callerDataArray(0)
	  maps.put("stack", stackTraceElement)
       }
    }
    try
      objectMapper.writeValueAsString(maps)
    catch {
      case e: Exception => event.getFormattedMessage
    }
  }
}
```
* 基于kafka实现一个appender 
```scala
class KafkaAppender extends AppenderBase[ILoggingEvent] {
  private val LOGGER: Logger = LoggerFactory.getLogger(classOf[KafkaAppender])
  private var formatter: Formatter = new JsonFormatter
  private var kafkaProducerProperties: String = null
  private var topic: String = null
  private var producer: Producer[String, String] = null
  override def append(e: ILoggingEvent) = {
    val string: String = this.formatter.format(e)
    val producerRecord: KeyedMessage[String, String] = new KeyedMessage\[String, String](topic, string)
    producer.send(producerRecord)
  }
  override def stop() = {
    super.stop()
    producer.close()
  }
  override def start() = {
    super.start()
    val properties: Properties = new Properties
    properties.load(new StringReader(kafkaProducerProperties))
    val config: ProducerConfig = new ProducerConfig(properties)
    producer = new Producer\[String, String](config)
  }
}
```

##### Kafka + Spark
spark消费kafka数据有两种方案，一种需要自己维护offset，另一种不需要，在这我们采用直连的方式进行消费
```scala
def consumerByDirectSpark(topic: String = "bbtest"): Unit = {
  val sparkConf = new SparkConf().setMaster("local[*]").setAppName("kafka consumer")
  sparkConf.set("auto.offset.reset", "smallest")
  val ssc = new StreamingContext(sparkConf, Seconds(2))
  val topicSet = topic.split(",").toSet
  val kafkaParams = Map\[String, String]("metadata.broker.list" -> "ip:port", "serializer.class" -> "kafka.serializer.StringEncoder")
  val messages = KafkaUtils.createDirectStream\[String, String, StringDecoder, StringDecoder](ssc, kafkaParams, topicSet).map(x=>x._2)
  messages.foreachRDD(rdd => {
    rdd.collect().foreach(x => {
      println(s"${Console.RED} ${x} ${Console.RESET}")
    })
  })
  ssc.start()
  ssc.awaitTermination()
}
```


##### logback 
关于loback，在这只要在logback.xml文件中添加相关的appender即可
```xml
<appender name="KAFKA" class="com.bob.kafka.KafkaAppender">
    <topic>bbtest</topic>
    <kafkaProducerProperties>
        metadata.broker.list=ip:post
        serializer.class=kafka.serializer.StringEncoder
        request.required.acks=1
        request.timeout.ms=5000
        producer.type=async
    </kafkaProducerProperties>
    <logToSystemOut>false</logToSystemOut>
</appender>
<logger name="packagename.classname" level="DEBUG" additivity="false">
    <appender-ref ref="KAFKA"/>
</logger>
```

##### 最后
我们在spark进行消费kafka日志数据的时候，可以直接对某种类型的错误如果频繁发生时，可以进行报警。同时也可以在消费完后入到[elasticsearch](https://www.elastic.co/products/elasticsearch)中，结合[kinaba](https://www.elastic.co/products/kibana)进行相关日志查询。