转自：https://kukudi.blog.csdn.net/article/details/106027405



设置消费者properties的两个参数

```shell
consumer.group.id

properties.setProperty("auto.offset.reset", "earliest”) // latest
```

> 注意：
>
> **只要不更改group.id，每次重新消费kafka，都是从上次消费结束的地方继续开始，不论"auto.offset.reset”属性设置的是什么**

 

* 场景一：Kafka上在实时被灌入数据，但kafka上已经积累了两天的数据，如何从最新的offset开始消费？

（最新指相对于当前系统时间最新）

​	1.将group.id换成新的名字(相当于加入新的消费组)

​	2.网上文章写还要设置 properties.setProperty("auto.offset.reset", "latest”)

> 实验发现即使不设置这个，只要group.id是全新的，就会从最新的的offset开始消费



* 场景二：kafka在实时在灌入数据，kafka上已经积累了两天的数据，如何从两天前最开始的位置消费？

1. 将group.id换成新的名字

2. properties.setProperty("auto.offset.reset", "earliest”)

* 场景三：不更改group.id，只是添加了properties.setProperty("auto.offset.reset", "earliest”)，consumer会从两天前最开始的位置消费吗？

  不会，只要不更改消费组，只会从上次消费结束的地方继续消费

* 场景四：不更改group.id，只是添加了properties.setProperty("auto.offset.reset", "latest”)，consumer会从距离现在最近的位置消费吗？

  不会，只要不更改消费组，只会从上次消费结束的地方继续消费

 ![img](..\..\..\images\bigdata\kafka\kafak_offset.png)

