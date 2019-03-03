---
layout: post
title:  "如何设计一个数据收集服务器架构"
date:   2019-03-03 00:18:23 +0700
categories: [bigdata]
---

#### 概述
  前几节中我们讲了数据采集方面的一些知识，这一节我们讲一些数据收集方面的知识。数据采集是如何从设备上获取数据并发送到服务端，服务端如何收集这些数据就是本节
的重点。数据收集本质上我们接口服务没有太大的区别就一个高并发，高吞吐的接口服务。常见的数据收集服务有Nginx/FaceBook的scribe等等。具体这些工具不做过程的
讲解，网上有很详细的资料。我们这节主要讲如何根据自己的业务情况选择相应的数据收集服务。
  
#### 如何分解数据收集业务
  数据收集业务的分解其实主要看数据源，我们之前讲的SDK采集就是一种数据源，通常来说大致我们可以分解成如下几种：  
  
###### 客户端数据上报  
  客户端采集的数据，通常通过实时或者批量方面发送到服务，通过接口的方面将这些数据收集并落地点文件或者数据库中，该块数据的特点一般会随着用户或者采集数据多少不断
增加，一般这些数据特点为非结化，数据量化，上报的高并量，实时性要求高。像一个上千万的平台每天至少产生超过５０ＴＢ以上的用户数据量。这类数据价值相比直接业务系统直接
数据价值小，但是数据全。需要通过数据分析产生更大的数据价值。
###### 业务系统产生LOG数据  
　　业务系统产生的LOG数据一般都在公司内网环境数据收集难度低，数据量大，通常像application log。非结构，用于监控服务的场景较多，对实时要求较高。根据公司部署的服务
情况而定，千万级的平台产生的application log 也在TB级别。当然也有比较服务的业务较小的用户量产生较大的application log,特别像电信这类行业。
###### 业务系统数据  
  业务系统通常我们是指存储在rmdbs中的系统，其特点结构化，数据可更新。一般是业务系统直接产生的结果数据。相较上面两类数据量偏小。对于数据的完整性要求极高
通过这类直接可直接反应数据价值，却直接被业务系统应用。

#### 如何根据业务特性选择收集工具  
  通过上面的业务分解我们可以按网络分类内网环境产生与外网环境产生，按产生者我们可以分为业务系统数据／用户数据；按是否结构化又可以分为结构化与非结构化；按存储方式
我们又可以分为文本数据与数据库数据。那具体我们如何选择收集工具，我们还是根据具体的场景进行选择：
  如何是客户端上报的数据我们如何选择?
  客户端上报的数据一般实时，单条记录不会太大，我们一般选择直接Nginx做为数据收集服务器，或者自己也可以根据情况开发一个log server,以之前SDK上报的数据为例我们如何搭建
```$xslt

```
  如果是内网的业务系统Application LOG我们怎么选择？
  内网的数据收集服务通常是以文件信息批量进行传输，通常来说其吞吐量大，并发小，走内网。一般我们会选择像Apache Flume这类工具。
```aidl

```
    
  如果是业务系统的生产的RMDBS数据我们怎么选择？  
  从RMDBS收集数据一般的步骤是从RMDBS中导出数据，然后将数据以文件的方式进行传输。
```aidl

```
  
#### 包接包送
  当数据收集落地之后，如何进行后面的数据分发工作。通常有二种情况，第一种将数据推送到存储上面。第二种将数据发送到消息队列。假设我们以存储是
HDFS，消息队列以Kafka为组件。
```aidl
将数据推送到存储上面:
 hdfs put
使用方法：hadoop fs -put <localsrc> ... <dst>

从本地文件系统中复制单个或多个源路径到目标文件系统。也支持从标准输入中读取输入写入目标文件系统。
hadoop fs -put localfile /user/hadoop/hadoopfile
hadoop fs -put localfile1 localfile2 /user/hadoop/hadoopdir
hadoop fs -put localfile hdfs://host:port/hadoop/hadoopfile
hadoop fs -put - hdfs://host:port/hadoop/hadoopfile 
从标准输入中读取输入。
返回值：
成功返回0，失败返回-1。

将数据发送到Kafka上面:
我们直接可以通过配置Flume Source配置为Directory
示例:
a1.sources = s1  
a1.sinks = k1  
a1.channels = c1  
   
# Describe/configure the source  
a1.sources.s1.type =spooldir  
a1.sources.s1.spoolDir =/home/hadoop/logs  
a1.sources.s1.fileHeader= true  
a1.sources.s1.channels =c1  
   
# Describe the sink  
a1.sinks.k1.type = logger  
a1.sinks.k1.channel = c1  
   
# Use a channel which buffers events inmemory  
a1.channels.c1.type = memory 

我们直接可以通过配置Flume Sink配置为Kafka
示例:
Flume2KafkaAgent.sources=mysource
Flume2KafkaAgent.channels=mychannel
Flume2KafkaAgent.sinks=mysink

Flume2KafkaAgent.sources.mysource.type=spooldir
Flume2KafkaAgent.sources.mysource.channels=mychannel
Flume2KafkaAgent.sources.mysource.spoolDir=/usr/local/share/applications/tmp/flumetokafka/logs

Flume2KafkaAgent.sinks.mysink.channel=mychannel
Flume2KafkaAgent.sinks.mysink.type=org.apache.flume.sink.kafka.KafkaSink
Flume2KafkaAgent.sinks.mysink.kafka.bootstrap.servers=master:9092,slave1:9092,slave2:9092,slave3:9092
Flume2KafkaAgent.sinks.mysink.kafka.topic=flume-data
Flume2KafkaAgent.sinks.mysink.kafka.batchSize=20
Flume2KafkaAgent.sinks.mysink.kafka.producer.requiredAcks=1

Flume2KafkaAgent.channels.mychannel.type=memory
Flume2KafkaAgent.channels.mychannel.capacity=30000
Flume2KafkaAgent.channels.mychannel.transactionCapacity=100

```
    
#### 收集的整体架构  
  

#### 数据收集过程中的几点思考  


#### 能不能自己造个轮子  


#### 与数据收集相关几个重要事项　　

  
