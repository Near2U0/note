---
title: Spark运维管理之作业资源调度
categories: spark  
tags: [spark]
---



# 静态资源分配原理

spark提供了许多功能用来在集群中同时调度多个作业。首先，回想一下，每个spark作业都会运行自己独立的一批executor进程，此时集群管理器会为我们提供同时调度多个作业的功能。第二，在每个spark作业内部，多个job也可以并行执行，比如说spark-shell就是一个spark application，但是随着我们输入scala rdd action类代码，就会触发多个job，多个job是可以并行执行的。为这种情况，spark也提供了不同的调度器来在一个application内部调度多个job。

<!--more-->


我们先来看一下多个作业的同时调度

静态资源分配

当一个spark application运行在集群中时，会获取一批独立的executor进程专门为自己服务，比如运行task和存储数据。如果多个用户同时在使用一个集群，并且同时提交多个作业，那么根据cluster manager的不同，有几种不同的方式来管理作业间的资源分配。

最简单的一种方式，是所有cluster manager都提供的，也就是静态资源分配。在这种方式下，每个作业都会被给予一个它能使用的最大资源量的限额，并且可以在运行期间持有这些资源。这是spark standalone集群和YARN集群使用的默认方式。

Standalone集群: 默认情况下，提交到standalone集群上的多个作业，会通过FIFO的方式来运行，每个作业都会尝试获取所有的资源。可以限制每个作业能够使用的cpu core的最大数量（spark.cores.max），或者设置每个作业的默认cpu core使用量（spark.deploy.defaultCores）。最后，除了控制cpu core之外，每个作业的spark.executor.memory也用来控制它的最大内存的使用。

YARN集群管理器: --num-executors属性用来配置作业可以在集群中分配到多少个executor，--executor-memory和--executor-cores可以控制每个executor能够使用的资源。

要注意的是，没有一种cluster manager可以提供多个作业间的内存共享功能。如果你想要通过这种方式来在多个作业间共享数据，我们建议就运行一个spark作业，但是可以接收网络请求，并对相同RDD的进行计算操作。在未来的版本中，内存存储系统，比如Tachyon会提供其他的方式来共享RDD数据。



# 动态资源分配原理


spark 1.2开始，引入了一种根据作业负载动态分配集群资源给你的多个作业的功能。这意味着你的作业在申请到了资源之后，可以在使用完之后将资源还给cluster manager，而且可以在之后有需要的时候再次申请这些资源。这个功能对于多个作业在集群中共享资源是非常有用的。如果部分资源被分配给了一个作业，然后出现了空闲，那么可以还给cluster manager的资源池中，并且被其他作业使用(**如果是今天资源分配,那么属于该作业的空闲资源,要等到作业完成之后才会释放,而不会立即释放空闲的资源**)。在spark中，动态资源分配在executor粒度上被实现，可以通过spark.dynamicAllocation.enabled来启用。

## 资源分配策略

以一个较高的角度来说，当executor不再被使用的时候，spark就应该释放这些executor，并且在需要的时候再次获取这些executor。因为没有一个绝对的方法去预测一个未来可能会运行一个task的executor应该被移除掉，或者一个新的executor应该别加入，我们需要一系列的探索式算法来决定什么应该移除和申请executor。

## 申请策略

一个启用了动态资源分配的spark作业会在它有pending住的task等待被调度时，申请额外的executor。这个条件必要地暗示了，已经存在的executor是不足以同时运行所有的task的，这些task已经提交了，但是没有完成。

driver会轮询式地申请executor。当在一定时间内（spark.dynamicAllocation.schedulerBacklogTimeout）有pending的task时，就会触发真正的executor申请，然后每隔一定时间后（spark.dynamicAllocation.sustainedSchedulerBacklogTimeout），如果又有pending的task了，则再次触发申请操作。此外，每一轮申请到的executor数量都会比上一轮要增加。举例来说，一个作业需要增加一个executor在第一轮申请时，那么在后续的一轮中会申请2个、4个、8个executor。

每轮增加executor数量的原因主要有两方面。第一，一个作业应该在开始谨慎地申请以防它只需要一点点executor就足够了。第二，作业应该会随着时间的推移逐渐增加它的资源使用量，以防突然大量executor被增加进来。

## 移除策略

移除一个executor的策略比较简单。一个spark作业会在它的executor出现了空闲超过一定时间后（spark.dynamicAllocation.executorIdleTimeout），被移除掉。要注意，在大多数环境下，这个条件都是跟申请条件互斥的，因为如果有task被pending住的话，executor是不该是空闲的。

## executor如何优雅地被释放掉

在使用动态分配之前，executor无论是发生了故障失败，还是关联的application退出了，都还是存在的。在所有场景中，executor关联的所有状态都不再被需要，并且可以被安全地抛弃。使用动态分配之后，executor移除之后，作业还是存在的。如果作业尝试获取executor写的中间状态数据，就需要去重新计算哪些数据。因此，spark需要一种机制来优雅地卸载executor，在移除它之前要保护它的状态。

解决方案就是使用一个外部的shuffle服务来保存每个executor的中间写状态，这也是spark 1.2引入的特性。这个服务是一个长时间运行的进程，集群的每个节点上都会运行一个,为你的spark作业和executor服务。如果服务被启用了，那么spark executor会在shuffle write和read时，将数据写入该服务，并从该服务获取数据。这意味着所有executor写的shuffle数据都可以在executor声明周期之外继续使用。

除了写shuffle文件，executor也会在内存或磁盘中持久化数据。当一个executor被移除掉时，所有缓存的数据都会消失。目前还没有有效的方案。在未来的版本中，缓存的数据可能会通过堆外存储来进行保存，就像external shuffle service保存shuffle write文件一样。



# Standalone模式下使用动态资源分配

```

#启用external shuffle service
--conf spark.shuffle.service.enabled=true \
#启用动态资源分配
--conf spark.dynamicAllocation.enabled=true \
#指定external shuffle service的端口号
--conf spark.shuffle.service.port=7337 \
```

1、启动external shuffle service
```
./sbin/.start-shuffle-service.sh

```

2、启动spark-shell，启用动态资源分配

```
spark-shell --master spark://192.168.0.103:7077 --conf spark.shuffle.service.enabled=true --conf spark.dynamicAllocation.enabled=true --conf spark.shuffle.service.port=7337


```

jps可以观察到ExternalShuffleService,并且在spark-shell开始的时候,会启动一个executor进程(CoarseGrainedExecutorBackend)


![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/externalShuffleService.png)



3、过60s，发现打印日志，说executor被removed，executor进程也没了,jps再次查看

![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/externalShuffleService2.png)

而且通过观察spark-shell的日志可以看到
![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/externalShuffleService3.png)



4、然后动手写一个wordcount程序，最后提交job的时候，会动态申请一个新的executor，出来一个新的executor进程

```
scala>val lines = sc.textFile("hdfs://192.168.0.103:9000/test/hello.txt")
scala>val words = lines.flatMap(_.split(" ")).map((_,1))
scala>val counts = words.reduceByKey(_+_)
scala>counts.collect

```

![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/externalShuffleService4.png)

新的executor进程会从external shuffle service进程中读取shuffle write的数据


5、然后整个作业执行完毕，证明external shuffle service+动态资源分配，流程可以走通


6、再等60s，executor又被释放掉

![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/externalShuffleService2.png)


# yarn模式下使用动态资源分配


先停止之前为standalone集群启动的shuffle service
```
./sbin/stop-shuffle-service.sh
```

## 配置

动态资源分配功能使用的所有配置，都是以spark.dynamicAllocation作为前缀的。要启用这个功能，你的作业必须将spark.dynamicAllocation.enabled设置为true。其他相关的配置之后会详细说明。

此外，你的作业必须有一个外部shuffle服务（external shuffle service）。这个服务的目的是去保存executor的shuffle write
文件，从而让executor可以被安全地移除。要启用这个服务，可以将spark.shuffle.service.enabled设置为true。在YARN中，这个
外部shuffle service是由org.apache.spark.yarn.network.YarnShuffleService实现的，在每个NodeManager中都会运行。要启用
这个服务，需要使用以下步骤：

1、首先配置好yarn的shuffle service，然后重启集群
1.1.使用预编译好的spark版本。
1.2.定位到spark-<version>-yarn-shuffle.jar。这个应该在$SPARK_HOME/lib目录下。
1.3.将上面的jar加入到所有NodeManager的classpath中(即在每个NodeManager中都拷贝一份)。


![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/allocat_yarn_resource.png)



1.4.在yarn-site.xml中，将yarn.nodemanager.aux-services设置为spark_shuffle，将yarn.nodemanager.aux-services.spark_shuffle.class设置为org.apache.spark.network.yarn.YarnShuffleService

![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/allocat_yarn_resource2.png)


1.5.重启所有hadoop集群
```
start-all.sh
```

查看进程
![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/allocat_yarn_resource3.png)

在浏览器查看确定:HDFS和yarn是启动OK的
HDFS:192.168.0.103:50070
yarn:192.168.0.103:8088




2、接着呢，启动spark shell，并启用动态资源分配，但是这里跟standalone不一样，上来不会立刻申请executor
```
spark-shell --master spark://192.168.0.103:7077 --conf spark.shuffle.service.enabled=true --conf spark.dynamicAllocation.enabled=true --conf spark.shuffle.service.port=7337


scala>val lines = sc.textFile("hdfs://192.168.0.103:9000/test/hello.txt")
scala>val words = lines.flatMap(_.split(" ")).map((_,1))
scala>val counts = words.reduceByKey(_+_)
scala>counts.collect

```


3、接着执行wordcount，会尝试动态申请executor，并且申请到后，执行job，在spark web ui上，有两个executor
4、过了一会儿，60s过后，executor由于空闲，所以自动被释放掉了，在看spark web ui，没有executor了

![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/allocat_yarn_resource4.png)





# 多个job资源调度原理

在一个spark作业内部，多个并行的job是可以同时运行的。对于job，就是一个spark action操作触发的计算单元。spark的调度器是完全线程安全的，而且支持一个spark application来服务多个网络请求，以及并发执行多个job。

默认情况下，spark的调度会使用FIFO的方式来调度多个job。每个job都会被划分为多个stage，而且第一个job会对所有可用的资源获取优先使用权，并且让它的stage的task去运行，然后第二个job再获取资源的使用权，以此类推。如果队列头部的job不需要使用整个集群资源，之后的job可以立即运行，但是如果队列头部的job使用了集群几乎所有的资源，那么之后的job的运行会被推迟。

从spark 0.8开始，我们是可以在多个job之间配置公平的调度器的。在公平的资源共享策略下，spark会将多个job的task使用一种轮询的方式来分配资源和执行，所以所有的job都有一个基本公平的机会去使用集群的资源。这就意味着，即使运行时间很长的job先提交并在运行了，之后提交的运行时间较短的job，也同样可以立即获取到资源并且运行，而不会等待运行时间很长的job结束之后才能获取到资源。这种模式对于多个并发的job是最好的一种调度方式。



# fair Scheduler使用详解

默认采用的是FIFO,要启用Fair Scheduler，只要简单地将spark.scheduler.mode属性设置为FAIR即可
```
val conf = new SparkConf().setMaster(...).setAppName(...)
conf.set("spark.scheduler.mode", "FAIR")
val sc = new SparkContext(conf)

或者

--conf spark.scheduler.mode=FAIR
```

fair scheduler也支持将job分成多个组并放入多个池中，以及为每个池设置不同的调度优先级。这个feature对于将重要的和不重要的job隔离运行的情况非常有用，可以为重要的job分配一个池，并给予更高的优先级; 为不重要的job分配另一个池，并给予较低的优先级。

默认情况下，新提交的job会进入一个默认池，但是job的池是可以通过spark.scheduler.pool属性来设置的。

如果你的spark application是作为一个服务启动的，SparkContext 7*24小时长时间存在，然后服务每次接收到一个请求，就用一个子线程去服务它在子线程内部，去执行一系列的RDD算子以及代码来触发job的执行在子线程内部，可以调用SparkContext.setLocalProperty("spark.scheduler.pool", "pool1")

在设置这个属性之后，所有在这个线程中提交的job都会进入这个池中。同样也可以通过将该属性设置为null来清空池子。

池的默认行为

默认情况下，每个池子都会对集群资源有相同的优先使用权，但是在每个池内，job会使用FIFO的模式来执行。举例来说，如果要为每个用户创建一个池，这就意味着每个用户都会获得集群的公平使用权，但是每个用户自己的job会按照顺序来执行。

配置池的属性

可以通过配置文件来修改池的属性。每个池都支持以下三个属性: 

1、schedulingMode: 可以是FIFO或FAIR，来控制池中的jobs是否要排队，或者是共享池中的资源
2、weight: 控制每个池子对集群资源使用的权重。默认情况下，所有池子的权重都是1.如果指定了一个池子的权重为2。举例来说，它就会获取其他池子两倍的资源使用权。设置一个很高的权重值，比如1000，也会很有影响，基本上该池子的task会在其他所有池子的task之前运行。
3、minShare: 除了权重之外，每个池子还能被给予一个最小的资源使用量(cpu core)。

池子的配置是通过xml文件来配置的，在spark/conf的fairscheduler.xml中配置
我们自己去设置这个文件的路径，conf.set("spark.scheduler.allocation.file", "/path/to/file")

文件内容大致如下所示
```
<?xml version="1.0"?>
<allocations>
  <pool name="production">
    <schedulingMode>FAIR</schedulingMode>
    <weight>1</weight>
    <minShare>2</minShare>
  </pool>
  <pool name="test">
    <schedulingMode>FIFO</schedulingMode>
    <weight>2</weight>
    <minShare>3</minShare>
  </pool>
</allocations>
```






