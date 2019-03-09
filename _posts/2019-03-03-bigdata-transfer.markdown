---
layout: post
title:  "如何规划数据平台的数据传输架构"
date:   2019-03-03 00:18:23 +0700
categories: [bigdata]
---
  
#### 概述
   数据传输是数据平台与数据源之间的桥梁，它负责解决数据在处理过程中进行各种迁移，无论是从RMDBS到存储，还是说从存储到RMDBS，或是说从收集服务器到存储中心。我们需要
靠各种数据传输工具。如何选择我们需要的传输工具确保数据能够正确传输，是今天我们需要解讲的重点。
  
#### 数据传输的架构
  本质上很多数据传输工具与数据采集工具没有太大的区别，类似FLume集成了数据采集与数据传输为一体的。另外一种是像Kafka的消息队列的传输工具。这两种工具一般根据应用场景进行
选择。今天我们主要讲一下Flume与Kafka这两个工具
  
#### Flume
   flume是由cloudera软件公司产出的可分布式日志收集系统，后与2009年被捐赠了apache软件基金会，为hadoop相关组件之一。尤其近几年随着flume的不断被完善以及升级版本的逐一推出，特别是flume-ng;同时flume内部的各种组件不断丰富，用户在开发的过程中使用的便利性得到很大的改善，现已成为apache top项目之一.
apache Flume 是一个从可以收集例如日志，事件等数据资源，并将这些数量庞大的数据从各项数据资源中集中起来存储的工具/服务，或者数集中机制。flume具有高可用，分布式，配置工具，其设计的原理也是基于将数据流，如日志数据从各种网站服务器上汇集起来存储到HDFS，HBase等集中存储器中。其结构如下图所示:  
    
![collect_flume](/static/img/post/flume_1.jpg){:height=""300" width="720"}  
  
##### Flume的特征    
- Flume可以高效率的将多个网站服务器中收集的日志信息存入HDFS/HBase中  
- 使用Flume，我们可以将从多个服务器中获取的数据迅速的移交给Hadoop中  
- 除了日志信息，Flume同时也可以用来接入收集规模宏大的社交网络节点事件数据，比如facebook,twitter,电商网站如亚马逊，flipkart等  
- 支持各种接入资源数据的类型以及接出数据类型
- 支持多路径流量，多管道接入流量，多管道接出流量，上下文路由  
- 可以被水平扩展

##### Flume的结构  
  
![collect_flume_2](/static/img/post/flume_2.jpg){:height=""300" width="720"}  
  如上图所示，数据发生器（如：facebook,twitter）产生的数据被被单个的运行在数据发生器所在服务器上的agent所收集，之后数据收容器从各个agent上汇集数据并将采集到的数据存入到HDFS或者HBase中
    
##### Flume的事件  
  事件作为Flume内部数据传输的最基本单元.它是由一个转载数据的字节数组(该数据组是从数据源接入点传入，并传输给传输器，也就是HDFS/HBase)和一个可选头部构成.典型的Flume 事件如下面结构所示：
    
  ![collect_flume_3](/static/img/post/flume_3.jpg){:height=""80" width="160"}
      
  我们在将event在私人定制插件时比如：flume-hbase-sink插件是，获取的就是event然后对其解析，并依据情况做过滤等，然后在传输给HBase或者HDFS.  
   
##### Flume Agent  
  我们在了解了Flume的外部结构之后,知道了Flume内部有一个或者多个Agent,然而对于每一个Agent来说,它就是一共独立的守护进程(JVM),它从客户端哪儿接收收集,或者从其他的 Agent哪儿接收,然后迅速的将获取的数据传给下一个目的节点sink,或者agent. 如下图所示flume的基本模型:  
  
  ![collect_flume_4](/static/img/post/flume_4.jpg){:height="160" width="160"}  
  
  Agent主要由:source,channel,sink三个组件组成.  
  Source:  
  从数据发生器接收数据,并将接收的数据以Flume的event格式传递给一个或者多个通道channal,Flume提供多种数据接收的方式,比如Avro,Thrift,twitter 1%等  s  
  Channel:  
  channal是一种短暂的存储容器,它将从source处接收到的event格式的数据缓存起来,直到它们被sinks消费掉,它在source和sink间起着一共桥梁的作用,channal是一个完整的事务,这一点保证了数据在收发的时候的一致性. 并且它可以和任意数量的source和sink链接. 支持的类型有: JDBC channel , File System channel , Memort channel等.  
  sink:  
  sink将数据存储到集中存储器比如Hbase和HDFS,它从channals消费数据(events)并将其传递给目标地. 目标地可能是另一个sink,也可能HDFS,HBase.  
  以上介绍的flume的主要组件,下面介绍一下Flume插件:  
  1. Interceptors拦截器  
  用于source和channel之间,用来更改或者检查Flume的events数据  
  2. 管道选择器 channels Selectors  
   在多管道是被用来选择使用那一条管道来传递数据(events). 管道选择器又分为如下两种:  
   默认管道选择器:  每一个管道传递的都是相同的events  
   多路复用通道选择器:  依据每一个event的头部header的地址选择管道.  
  3.sink线程  
   用于激活被选择的sinks群中特定的sink,用于负载均衡.    
  
  
#### Kafka  
  