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
  从数据发生器接收数据,并将接收的数据以Flume的event格式传递给一个或者多个通道channal,Flume提供多种数据接收的方式,比如Avro,Thrift,twitter 1%等  
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
  kafka是一个分布式消息队列。具有高性能、持久化、多副本备份、横向扩展能力。生产者往队列里写消息，消费者从队列里取消息进行业务逻辑。一般在架构设计中起到解耦、削峰、异步处理的作用。  
  kafka对外使用topic的概念，生产者往topic里写消息，消费者从读消息。为了做到水平扩展，一个topic实际是由多个partition组成的，遇到瓶颈时，可以通过增加partition的数量来进行横向扩容。单个parition内是保证消息有序。  
  每新写一条消息，kafka就是在对应的文件append写，所以性能非常高。  
  kafka的总体数据流是这样的：  
  
  ![collect_kafka_1](/static/img/post/kafka_1.webp){:height="320" width="720"}  
  
  大概用法就是，Producers往Brokers里面的指定Topic中写消息，Consumers从Brokers里面拉去指定Topic的消息，然后进行业务处理。图中有两个topic，topic 0有两个partition，topic 1有一个partition，三副本备份。可以看到consumer gourp 1中的consumer 2没有分到partition处理，这是有可能出现的，下面会讲到。
关于broker、topics、partitions的一些元信息用zk来存，监控和路由啥的也都会用到zk。
  
##### Kafka生产  
  基本流程是这样的：    
  
  ![collect_kafka_2](/static/img/post/kafka_2.webp){:height="320" width="720"} 
  
  创建一条记录，记录中一个要指定对应的topic和value，key和partition可选。 先序列化，然后按照topic和partition，放进对应的发送队列中。kafka produce都是批量请求，会积攒一批，然后一起发送，不是调send()就进行立刻进行网络发包。如果partition没填，那么情况会是这样的：
  1.key有填  
  按照key进行哈希，相同key去一个partition。（如果扩展了partition的数量那么就不能保证了）  
  2.key没填  
  round-robin来选partition    
  这些要发往同一个partition的请求按照配置，攒一波，然后由一个单独的线程一次性发过去。  
  
###### API  
  有high level api，替我们把很多事情都干了，offset，路由啥都替我们干了，用以来很简单。还有simple api，offset啥的都是要我们自己记录。  
  
###### Partition  
  当存在多副本的情况下，会尽量把多个副本，分配到不同的broker上。kafka会为partition选出一个leader，之后所有该partition的请求，实际操作的都是leader，然后再同步到其他的follower。当一个broker歇菜后，所有leader在该broker上的partition都会重新选举，选出一个leader。（这里不像分布式文件存储系统那样会自动进行复制保持副本数）  
  然后这里就涉及两个细节：怎么分配partition，怎么选leader。  
  关于partition的分配，还有leader的选举，总得有个执行者。在kafka中，这个执行者就叫controller。kafka使用zk在broker中选出一个controller，用于partition分配和leader选举。  
  
###### Partition的分配   
  1.将所有Broker（假设共n个Broker）和待分配的Partition排序  
  2.将第i个Partition分配到第（i mod n）个Broker上 （这个就是leader）  
  3.将第i个Partition的第j个Replica分配到第（(i + j) mode n）个Broker上  
  
  
###### Leader容灾  
  controller会在Zookeeper的/brokers/ids节点上注册Watch，一旦有broker宕机，它就能知道。当broker宕机后，controller就会给受到影响的partition选出新leader。controller从zk的/brokers/topics/[topic]/partitions/[partition]/state中，读取对应partition的ISR（in-sync replica已同步的副本）列表，选一个出来做leader。  
  选出leader后，更新zk，然后发送LeaderAndISRRequest给受影响的broker，让它们改变知道这事。为什么这里不是使用zk通知，而是直接给broker发送rpc请求，我的理解可能是这样做zk有性能问题吧。  
  如果ISR列表是空，那么会根据配置，随便选一个replica做leader，或者干脆这个partition就是歇菜。如果ISR列表的有机器，但是也歇菜了，那么还可以等ISR的机器活过来。  
  
  
###### 多副本同步  
  这里的策略，服务端这边的处理是follower从leader批量拉取数据来同步。但是具体的可靠性，是由生产者来决定的。  
  生产者生产消息的时候，通过request.required.acks参数来设置数据的可靠性。  
  acks:0;which means that the producer never waits for an acknowledgement from the broker.发过去就完事了，不关心broker是否处理成功，可能丢数据。  
  acks:1;which means that the producer gets an acknowledgement after the leader replica has received the data. 当写Leader成功后就返回,其他的replica都是通过fetcher去同步的,所以kafka是异步写，主备切换可能丢数据。  
  acks:-1;which means that the producer gets an acknowledgement after all in-sync replicas have received the data. 要等到isr里所有机器同步成功，才能返回成功，延时取决于最慢的机器。强一致，不会丢数据。  
  在acks=-1的时候，如果ISR少于min.insync.replicas指定的数目，那么就会返回不可用。  
  这里ISR列表中的机器是会变化的，根据配置replica.lag.time.max.ms，多久没同步，就会从ISR列表中剔除。以前还有根据落后多少条消息就踢出ISR，在1.0版本后就去掉了，因为这个值很难取，在高峰的时候很容易出现节点不断的进出ISR列表。  
  从ISA中选出leader后，follower会从把自己日志中上一个高水位后面的记录去掉，然后去和leader拿新的数据。因为新的leader选出来后，follower上面的数据，可能比新leader多，所以要截取。这里高水位的意思，对于partition和leader，就是所有ISR中都有的最新一条记录。消费者最多只能读到高水位；  
  从leader的角度来说高水位的更新会延迟一轮，例如写入了一条新消息，ISR中的broker都fetch到了，但是ISR中的broker只有在下一轮的fetch中才能告诉leader。  
  也正是由于这个高水位延迟一轮，在一些情况下，kafka会出现丢数据和主备数据不一致的情况，0.11开始，使用leader epoch来代替高水位。  
  
  
       
  
##### Kafka消费  
  订阅topic是以一个消费组来订阅的，一个消费组里面可以有多个消费者。同一个消费组中的两个消费者，不会同时消费一个partition。换句话来说，就是一个partition，只能被消费组里的一个消费者消费，但是可以同时被多个消费组消费。因此，如果消费组内的消费者如果比partition多的话，那么就会有个别消费者一直空闲。  
  
  ![collect_kafka_3](/static/img/post/kafka_3.webp){:height="320" width="720"}  
  
###### API  
  订阅topic时，可以用正则表达式，如果有新topic匹配上，那能自动订阅上。  
  
###### Offset的保存      
  一个消费组消费partition，需要保存offset记录消费到哪，以前保存在zk中，由于zk的写性能不好，以前的解决方法都是consumer每隔一分钟上报一次。这里zk的性能严重影响了消费的速度，而且很容易出现重复消费。  
  在0.10版本后，kafka把这个offset的保存，从zk总剥离，保存在一个名叫__consumeroffsets topic的topic中。写进消息的key由groupid、topic、partition组成，value是偏移量offset。topic配置的清理策略是compact。总是保留最新的key，其余删掉。一般情况下，每个key的offset都是缓存在内存中，查询的时候不用遍历partition，如果没有缓存，第一次就会遍历partition建立缓存，然后查询返回。
  确定consumer group位移信息写入__consumers_offsets的哪个partition，具体计算公式：  
```aidl
__consumers_offsets partition = Math.abs(groupId.hashCode() % groupMetadataTopicPartitionCount)   
//groupMetadataTopicPartitionCount由offsets.topic.num.partitions指定，默认是50个分区。
``` 
  
###### 分配partition--reblance  
   生产过程中broker要分配partition，消费过程这里，也要分配partition给消费者。类似broker中选了一个controller出来，消费也要从broker中选一个coordinator，用于分配partition。  
   下面从顶向下，分别阐述一下  
   1.怎么选coordinator  
   2.交互流程  
   3.reblance的流程  
   
###### 选coordinator  
  1.看offset保存在那个partition  
  2.该partition leader所在的broker就是被选定的coordinator  
  这里我们可以看到，consumer group的coordinator，和保存consumer group offset的partition leader是同一台机器。  
  
###### 交互流程  
  把coordinator选出来之后，就是要分配了整个流程是这样的：  
  1.consumer启动、或者coordinator宕机了，consumer会任意请求一个broker，发送ConsumerMetadataRequest请求，broker会按照上面说的方法，选出这个consumer对应coordinator的地址  
  2.consumer 发送heartbeat请求给coordinator，返回IllegalGeneration的话，就说明consumer的信息是旧的了，需要重新加入进来，进行reblance。返回成功，那么consumer就从上次分配的partition中继续执行  
  
###### Reblance流程  
  1.consumer给coordinator发送JoinGroupRequest请求  
  2.这时其他consumer发heartbeat请求过来时，coordinator会告诉他们，要reblance了  
  3.其他consumer发送JoinGroupRequest请求  
  4.所有记录在册的consumer都发了JoinGroupRequest请求之后，coordinator就会在这里consumer中随便选一个leader。然后回JoinGroupRespone，这会告诉consumer你是follower还是leader，对于leader，还会把follower的信息带给它，让它根据这些信息去分配partition  
  5.consumer向coordinator发送SyncGroupRequest，其中leader的SyncGroupRequest会包含分配的情况  
  6.coordinator回包，把分配的情况告诉consumer，包括leader  
  当partition或者消费者的数量发生变化时，都得进行reblance。列举一下会reblance的情况：  
  1.增加partition  
  2.增加消费者  
  3.消费者主动关闭  
  4.消费者宕机了  
  5.Coordinator自己也宕机了  
  
###### 消息投递语义
  消息投递语义  
  At most once：最多一次，消息可能会丢失，但不会重复  
  At least once：最少一次，消息不会丢失，可能会重复  
  Exactly once：只且一次，消息不丢失不重复，只且消费一次（0.11中实现，仅限于下游也是kafka）  
  在业务中，常常都是使用At least once的模型，如果需要可重入的话，往往是业务自己实现。  
  
  1.At least once  
  先获取数据，再进行业务处理，业务处理成功后commit offset。  
  > 生产者生产消息异常，消息是否成功写入不确定，重做，可能写入重复的消息  
  > 消费者处理消息，业务处理成功后，更新offset失败，消费者重启的话，会重复消费  
  
  2.At most once  
  先获取数据，再commit offset，最后进行业务处理  
  > 生产者生产消息异常，不管，生产下一个消息，消息就丢了  
  > 消费者处理消息，先更新offset，再做业务处理，做业务处理失败，消费者重启，消息就丢了  
  
  3.Exactly once  
  思路是这样的，首先要保证消息不丢，再去保证不重复。所以盯着At least once的原因来搞。 首先想出来的：  
  > 生产者重做导致重复写入消息----生产保证幂等性
  > 消费者重复消费---消灭重复消费，或者业务接口保证幂等性重复消费也没问题  
  
  由于业务接口是否幂等，不是kafka能保证的，所以kafka这里提供的exactly once是有限制的，消费者的下游也必须是kafka。所以一下讨论的，没特殊说明，消费者的下游系统都是kafka（注:使用kafka conector，它对部分系统做了适配，实现了exactly once）
  
###### 生产幂等性  
  思路是这样的，为每个producer分配一个pid，作为该producer的唯一标识。producer会为每一个<topic,partition>维护一个单调递增的seq。类似的，broker也会为每个<pid,topic,partition>记录下最新的seq。当req_seq == broker_seq+1时，broker才会接受该消息。因为：
  1.消息的seq比broker的seq大超过时，说明中间有数据还没写入，即乱序了。  
  2.消息的seq不比broker的seq小，那么说明该消息已被保存。  
  
  ![collect_kafka_4](/static/img/post/kafka_4.webp){:height="320" width="720"}  
  
###### 事务性/原子性广播  
  场景是这样的：  
  1.先从多个源topic中获取数据  
  2.做业务处理，写到下游的多个目的topic  
  3.更新多个源topic的offset  
  其中第2、3点作为一个事务，要么全成功，要么全失败。这里得益与offset实际上是用特殊的topic去保存，这两点都归一为写多个topic的事务性处理。  
  
  ![collect_kafka_5](/static/img/post/kafka_5.webp){:height="320" width="720"}
  
   基本思路是这样的：  
   引入tid（transaction id），和pid不同，这个id是应用程序提供的，用于标识事务，和producer是谁并没关系。就是任何producer都可以使用这个tid去做事务，这样进行到一半就死掉的事务，可以由另一个producer去恢复  
   同时为了记录事务的状态，类似对offset的处理，引入transaction coordinator用于记录transaction log。在集群中会有多个transaction coordinator，每个tid对应唯一一个transaction coordinator  
   注：transaction log删除策略是compact，已完成的事务会标记成null，compact后不保留。  
   做事务时，先标记开启事务，写入数据，全部成功就在transaction log中记录为prepare commit状态，否则写入prepare abort的状态。之后再去给每个相关的partition写入一条marker（commit或者abort）消息，标记这个事务的message可以被读取或已经废弃。成功后在transaction log记录下commit/abort状态，至此事务结束。
   数据流：  
   
   ![collect_kafka_6](/static/img/post/kafka_6.webp){:height="320" width="720"}
   
   > 首先使用tid请求任意一个broker（代码中写的是负载最小的broker），找到对应的transaction coordinator  
   > 请求transaction coordinator获取到对应的pid，和pid对应的epoch，这个epoch用于防止僵死进程复活导致消息错乱，当消息的epoch比当前维护的epoch小时，拒绝掉。tid和pid有一一对应的关系，这样对于同一个tid会返回相同的pid  
   > client先请求transaction coordinator记录<topic,partition>的事务状态，初始状态是BEGIN，如果是该事务中第一个到达的<topic,partition>，同时会对事务进行计时；client输出数据到相关的partition中；client再请求transaction coordinator记录offset的<topic,partition>事务状态；client发送offset commit到对应offset partition  
   > client发送commit请求，transaction coordinator记录prepare commit/abort，然后发送marker给相关的partition。全部成功后，记录commit/abort的状态，最后这个记录不需要等待其他replica的ack，因为prepare不丢就能保证最终的正确性了  
   
   这里prepare的状态主要是用于事务恢复，例如给相关的partition发送控制消息，没发完就宕机了，备机起来后，producer发送请求获取pid时，会把未完成的事务接着完成  
   当partition中写入commit的marker后，相关的消息就可被读取。所以kafka事务在prepare commit到commit这个时间段内，消息是逐渐可见的，而不是同一时刻可见。  
   
###### 消费事务  
  前面都是从生产的角度看待事务。还需要从消费的角度去考虑一些问题。
  消费时，partition中会存在一些消息处于未commit状态，即业务方应该看不到的消息，需要过滤这些消息不让业务看到，kafka选择在消费者进程中进行过来，而不是在broker中过滤，主要考虑的还是性能。kafka高性能的一个关键点是zero copy，如果需要在broker中过滤，那么势必需要读取消息内容到内存，就会失去zero copy的特性。
  
###### 文件组织  
  kafka的数据，实际上是以文件的形式存储在文件系统的。topic下有partition，partition下有segment，segment是实际的一个个文件，topic和partition都是抽象概念。  
  在目录/${topicName}-{$partitionid}/下，存储着实际的log文件（即segment），还有对应的索引文件。  
  每个segment文件大小相等，文件名以这个segment中最小的offset命名，文件扩展名是.log；segment对应的索引的文件名字一样，扩展名是.index。有两个index文件，一个是offset index用于按offset去查message，一个是time index用于按照时间去查，其实这里可以优化合到一起，下面只说offset index。总体的组织是这样的：。  
  
  ![collect_kafka_7](/static/img/post/kafka_7.webp){:height="320" width="720"}
  
  为了减少索引文件的大小，降低空间使用，方便直接加载进内存中，这里的索引使用稀疏矩阵，不会每一个message都记录下具体位置，而是每隔一定的字节数，再建立一条索引。 索引包含两部分，分别是baseOffset，还有position  
  baseOffset：意思是这条索引对应segment文件中的第几条message。这样做方便使用数值压缩算法来节省空间。例如kafka使用的是varint。  
  position：在segment中的绝对位置。  
  查找offset对应的记录时，会先用二分法，找出对应的offset在哪个segment中，然后使用索引，在定位出offset在segment中的大概位置，再遍历查找message。  
  
###### 常用配置项  
  1.broker配置  
  > broker.id:broker的唯一标识    
  > auto.create.topics.auto:设置成true，就是遇到没有的topic自动创建topic。    
  > log.dirs:log的目录数，目录里面放partition，当生成新的partition时，会挑目录里partition数最少的目录放。    
  
  2.topic配置  
  > num.partitions:新建一个topic，会有几个partition。    
  > log.retention.ms:对应的还有minutes，hours的单位。日志保留时间，因为删除是文件维度而不是消息维度，看的是日志文件的mtime。    
  > log.retention.bytes:partion最大的容量，超过就清理老的。注意这个是partion维度，就是说如果你的topic有8个partition，配置1G，那么平均分配下，topic理论最大值8G。    
  > log.segment.bytes:一个segment的大小。超过了就滚动。    
  > log.segment.ms:一个segment的打开时间，超过了就滚动。    
  > message.max.bytes:message最大多大  
  
  关于日志清理，默认当前正在写的日志，是怎么也不会清理掉的。  
  还有0.10之前的版本，时间看的是日志文件的mtime，但这个指是不准确的，有可能文件被touch一下，mtime就变了。因此在0.10版本开始，改为使用该文件最新一条消息的时间来判断。  
  按大小清理这里也要注意，Kafka在定时任务中尝试比较当前日志量总大小是否超过阈值至少一个日志段的大小。如果超过但是没超过一个日志段，那么就不会删除。  
  
    
  
   
  