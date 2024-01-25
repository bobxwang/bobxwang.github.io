---
categories:
  - code
date: 2014-01-05 14:17:59
tags:
  - bigdata
  - spark
title: Spark随谈--基础概念
---

>**Application**:基于Spark的用户程序，包含了一个driver program和集群中多个executor
>
>**Driver Program**:运行Application的main()函数并创建SparkContext。通常SparkContext代表driver program
>
>**Executor**:为某Application运行在worker node上的一个进程。该进程负责运行Task，并负责将数据存在内存或者磁盘上。每个Application都有自己独立的executors
>
>**Cluster Manager**:在集群上获得资源的外部服务（例如 Spark Standalon，Mesos、Yarn)
>**Worker Node**:集群中任何可运行Application代码的节点
>
>**Task**:被送到executor上执行的工作单元。
>
>**Job**:可以被拆分成Task并行计算的工作单元，一般由Spark Action触发的一次执行作业。
>
>**Stage**:每个Job会被拆分成很多组Task，每组任务被称为stage，也可称TaskSet。该术语可以经常在日志中看打。
>
>**RDD**:Spark的基本计算单元，通过Scala集合转化、读取数据集生成或者由其他RDD经过算子操作得到。

## SparkConf
对于spark的环境做一些基本设置
```scala
conf = SparkConf().setAppName('APP_NAME')
conf.setMaster('local')
```

## SparkContext 
通过前面设置好的conf,构造出一个spark的上下文环境，作为一个操作和调度的总入口
```scala
sc = SparkContext(conf=conf)
```

## RDD
**Resilient Distributed Datasets**，是一个容错的，并行的数据结构，可以让用户显示地将数据存储到磁盘和内存中，并能控制数据的分区，本质上，RDD是一个数据集。RDD将操作分成两类，*transformation*跟*action*，无论你执行了多少次transformation操作，RDD都不会真正执行运算，只有当action操作被执行时，运算才会触发，在其内部实现机制中是基于迭代器的，从而使数据的访问更高效。

**Transformation**，根据现有的数据集创建一个新的数据集

* map(func):对调用map的RDD数据集中的每个element都使用func，然后返回一个新的RDD

* filter(func):对调用filter的RDD数据集中的每个元素都使用func，然后返回一个包含使func为true的元素构成的RDD

* flatMap(func):和map差不多，但是flatMap对于每个输入元素生成的是0或多个结果，因此func函数返回值不是一个单一元素，而是一个序列

* groupByKey(numTasks):返回(K,Seq[V])，也就是hadoop中reduce函数接受的key-valuelist

* reduceByKey(func,[numTasks]):就是用一个给定的reduce func再作用在groupByKey产生的(K,Seq[V]),比如求和，求平均数

* sortByKey([ascending],[numTasks]):按照key来进行排序，是升序还是降序，ascending是boolean类型

>**narrow dependencies**
>子RDD的每个分区依赖于常数个父分区（与数据规模无关）
输入输出一对一的算子，且结果RDD的分区结构不变。主要是map/flatmap
输入输出一对一的算子，但结果RDD的分区结构发生了变化，如union/coalesce
从输入中选择部分元素的算子，如filter、distinct、substract、sample
>
>**wide dependencies**
>子RDD的每个分区依赖于所有的父RDD分区
对单个RDD基于key进行重组和reduce，如groupByKey，reduceByKey
对两个RDD基于key进行join和重组，如join
经过大量shuffle生成的RDD，建议进行缓存。这样避免失败后重新计算带来的开销。

**Action**，在数据集上运行计算后，返回一个值给驱动程序

* reduce(func):说白了就是聚集，接受传入的函数是两个参数输入返回一个值，这个函数必须是满足交换律和结合律的

* saveAsTextFile(path):把dataset写到一个text file中，或者hdfs，或者hdfs支持的文件系统中，spark把每条记录都转换为一行记录，然后写到file中

* saveAsSequenceFile(path):只能用在key-value对上，然后生成SequenceFile写到本地或者hadoop文件系统
```scala
data = [1,2,3,4,5]
distData = sc.parallelize(data)
lines = sc.textFile('data.txt')
linelengths = lines.map(lambda s: len(s))
totallength = linelengths.reduce(lambda a,b: a+b)
```

## Shared Variables
一般来说，当一个函数被传给Spark操作(map/reduce)，通常是在集群结点上运行，在函数中使用到的所有变量都分别做拷贝供函数操作而不会相互影响，而在这些函数中对变量所做的所有更新都不会被传播回驱动程序。然而Spark提供两种有限的共享变量：广播跟累加。
#####Broadcast Vaiables: 广播变量，可以在每台机器上缓存只读变量而不需要为各个任务发送该变量的拷贝，可以让大的输入数据集的集群拷贝中的节点更加高效
```scala
broadcastVar = sc.broadcase([1,2,3])
```

##### Accumulators: 累加器，只有驱动程序才能获取累加器的值
```scala
accum = sc.accumulator(0)
sc.parallelize([1,2,3,4]).foreach(lambda x:accum.add(x))
```