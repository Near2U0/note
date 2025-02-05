---
title: Spark2.0新特性之Structured Streaming介绍
categories: spark  
tags: [spark]
---



Spark Streaming应该说是将离线计算操作和流式计算操作统一起来的大数据计算框架之一(就是DStream和rdd的很多操作都是类似的)。从Spark 0.7开始引入的Spark Streaming，为开发人员提供了很多有用的特性：一次且仅一次的语义支持、容错性、强一致性保证、高吞吐量。


<!--more-->


但是实际上在真正工业界的流式计算项目中，并不仅仅只是需要一个流式计算引擎。这些项目实际上需要深度地使用批处理计算以及流式处理技术，与外部存储系统进行整合，还有应对业务逻辑变更的能力。因此，企业实际上不仅仅只是需要一个流式计算引擎，他们需要的是一个全栈式的技术，让他们能够开发end-to-end的持续计算应用（continuous application）。

Spark 2.0为了解决上述流式计算的痛点和需求，开发了新的模块——Structured Streaming。

Structured Streaming提供了与批处理计算类似的API。要开发一个流式计算应用，开发人员只要使用Dataframe/Dataset API编写与批处理计算一样的代码即可，Structured Streaming会自动将这些类似批处理的计算代码增量式地应用到持续不断进入的新数据上。这样，开发人员就不需要花太多时间考虑状态管理、容错、与离线计算的同步等问题。Structured Streaming可以保证，针对相同的数据，始终与离线计算产出完全一样的计算结果。

Structured Streaming还提供了与存储系统的事务整合。它会进行自动的容错管理以及数据一致性的管理，如果开发人员要写一个应用程序来更新数据库，进而提供一些实时数据服务，与静态数据进行join，或者是在多个存储系统之间移动数据，那么Structured Streaming可以让这些事情更加简单。

Structured Streaming与Spark其余的组件都能够进行完美的整合。比如可以通过Spark SQL对实时数据进行统计分析，与静态数据进行join，还有其他的使用dataframe/dataset的组件，这样就可以让开发人员构建完整的流式计算引用，而不仅仅只是一个流式计算引擎而已。在未来，Spark会将Structured Streaming与Spark MLlib的整合做的更好。


Spark 2.0搭载了一个beta版本的Structured Streaming，目前是作为Dataframe/Dataset的一个小的附加组件。主要是让Spark用户可以先尝试使用一下Structured Streaming，比如做一些实验和测试。Structured Streaming的一些关键特性，比如基于时间的处理，延迟数据的处理，交互式的查询，以及与非流式的数据源和存储进行整合，可能会基于未来的版本来实现。






