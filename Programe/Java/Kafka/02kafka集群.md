# Kafka集群搭建

对于kafka来说，一个单独的broker意味着kafka集群中只有一个节点。要想增加kafka集群中的节点数量，只需要多启动几个broker实例即可。为了有更好的理解，现在我们在一台机器上同时启动三个broker实例。

首先，我们需要建立好其他2个broker的配置文件：

```shell
#复制配置文件
cp config/server.properties config/server-1.properties 
cp config/server.properties config/server-2.properties     
```

配置文件的需要修改的内容分别如下：

config/server-1.properties:

```shell
#broker.id属性在kafka集群中必须要是唯一
broker.id=1 
#kafka部署的机器ip和提供服务的端口号 
listeners=PLAINTEXT://localhost:9091    
log.dir=/usr/local/data/kafka-logs-1 
#kafka连接zookeeper的地址，要把多个kafka实例组成集群，对应连接的zookeeper必须相同 
zookeeper.connect=localhost:2181
```

config/server-2.properties:

```shell
broker.id=2
listeners=PLAINTEXT://localhost:9092
log.dir=/usr/local/data/kafka-logs-2
zookeeper.connect=localhost:2181       
```

目前我们已经有一个zookeeper实例和一个broker实例在运行了，现在我们只需要在启动2个broker实例即可：

```shell
bin/kafka-server-start.sh -daemon config/server-1.properties

bin/kafka-server-start.sh -daemon config/server-2.properties 
```



现在我们创建一个新的topic，副本数设置为3，分区数设置为2：

```shell
#表示运行bin/kafka-topics.sh脚本;
#--create表示创建
#--zookeeper 192.168.124.16:2181 表示指定zookeeper服务host:port
#--replication-factor 3 表示指定副本因子为3(一般建议有几个broker就定几个副本因子)
#--partitions 2 表示指定分区数为2
#--topic hellKafka 表示此次创建topic名称为"kafka-colony"
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 2 --topic kafka-colony
```

**查看下topic的情况**

```shell
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic kafka-colony   
```

![image-20210125232733603](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210125232733603.png)

- 输出结果说明:
  - **topic:kafka-colony：**主题名称为kafka-colony
  - **PartitionCount:2 ：**主题总分区数量为2
  - **ReplicationFactor:3 ：**主题总副本数量为3
  - **Partition:0 ：**当前分区编号为0
  - **Leader:1：**当前分区主节点broker_id为1
  - **replicas:1,2,0：**当前分区所有节点broker_id列表
  - **Isr:1,2,0：**当前分区<span style="color:red">存活并且已同步备份leader节点</span>>的节点broker_id，是replicas的子集

本处笔者消费者发送消息到broker 0中，

```shell
bin/kafka-console-producer.sh --broker-list localhost:9090 --topic kafka-colony
>my test msg 1
>my test msg 2
```

现在开始消费：(现象如下,原理待分析)

```shell
#消费者消费broker 2,发现可以正常消费
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic kafka-colony 
>my test msg 1 
>my test msg 2
#另起一个消费者,同一消费组从头开始消费broker 1,没有消费到信息
bin/kafka-console-consumer.sh --bootstrap-server localhost:9091 --from-beginning --consumer-property group.id=testGroup --topic kafka-colony
#起一个消费者,不同消费组从头开始消费broker 1,消费的消息并不按顺序
bin/kafka-console-consumer.sh --bootstrap-server localhost:9091 --from-beginning --consumer-property group.id=testGroup-1 --topic kafka-colony
>my test msg 2 
>my test msg 1
```

接现在我们先来测试容错性，因为broker1目前是kafka-colony的分区0的leader，所以我们要将其kill

```shell
ps -ef | grep server-1.properties 
kill 14776
```

现在再执行命令：

```shell
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic kafka-colony 
```

![image-20210126000902381](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210126000902381.png)

我们可以看到，分区0的leader节点已经变成了broker 2。要注意的是，在Isr中，已经没有了1号节点。leader的选举也是从ISR(in-sync replica)中进行的。

此时，我们消费消息：

```shell
#只指定连接broker 1来消费
bin/kafka-console-consumer.sh --bootstrap-server localhost:9091 --from-beginning --consumer-property group.id=testGroup --topic kafka-colony
#出现如下错误
WARN [Consumer clientId=consumer-1, groupId=testGroup] Connection to node -1 could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
#修改为连接broker 2来消费,可正常消费
bin/kafka-console-consumer.sh --bootstrap-serv localhost:9092 --from-beginning --consumer-property group.id=testGroup --topic kafka-colony 
>my test msg 1 
>my test msg 2
```

可以看到，集群中一台broker故障,可以自动升级leader节点，保证高可用,而我们遇到的连接不上挂掉的broker 1,可以通过连接集群所有broker来解决：

```shell
#集群生产:
bin/kafka-console-producer.sh --broker-list localhost:9090,localhost:9091,localhost:9092 --topic kafka-colony
#集群消费
bin/kafka-console-consumer.sh --bootstrap-server localhost:9090,localhost:9091,localhost:9092 --from-beginning --topic kafka-colony
```

## **集群消费**

------

log的partitions分布在kafka集群中不同的broker上，每个broker可以请求备份其他broker上partition上的数据。kafka集群支持配置一个partition备份的数量。

针对每个partition，都有一个broker起到“leader”的作用，0个或多个其他的broker作为“follwers”的作用。**leader处理所有的针对这个partition的读写请求，而followers被动复制leader的结果，不提供读写(主要是为了保证多副本数据与消费的一致性)**。如果这个leader失效了，其中的一个follower将会自动的变成新的leader。

**Producers**

生产者将消息发送到topic中去，同时负责选择将message发送到topic的哪一个partition中。通过round-robin做简单的负载均衡。也可以根据消息中的某一个关键字来进行区分。通常第二种方式使用的更多。

**Consumers**

传统的消息传递模式有2种：队列( queue) 和（publish-subscribe）

- queue模式：多个consumer从服务器中读取数据，消息只会到达一个consumer。
- publish-subscribe模式：消息会被广播给所有的consumer。

Kafka基于这2种模式提供了一种consumer的抽象概念：consumer group。

- queue模式：所有的consumer都位于同一个consumer group 下。
- publish-subscribe模式：所有的consumer都有着自己唯一的consumer group。

​    ![0](https://note.youdao.com/yws/public/resource/e5863162eca29c2b31e8b59c9707817d/xmlnote/9699992BE45B4C21BFE829B1A4EB34D5/105066)

上图说明：由2个broker组成的kafka集群，某个主题总共有4个partition(P0-P3)，分别位于不同的broker上。这个集群由2个Consumer Group消费， A有2个consumer instances ，B有4个。

通常一个topic会有几个consumer group，每个consumer group都是一个逻辑上的订阅者（ logical subscriber ）。每个consumer group由多个consumer instance组成，从而达到可扩展和容灾的功能。

## **消费顺序**

一个partition同一个时刻在一个consumer group中只能有一个consumer instance在消费，从而保证消费顺序。

**consumer group中的consumer instance的数量不能比一个Topic中的partition的数量多，否则，多出来的consumer消费不到消息。**

Kafka只在partition的范围内保证消息消费的局部顺序性，不能在同一个topic中的多个partition中保证总的消费顺序性。

如果有在总体上保证消费顺序的需求，那么我们可以通过将topic的partition数量设置为1，将consumer group中的consumer instance数量也设置为1，但是这样会影响性能，所以kafka的顺序消费很少用。

# Zookeeper查看集群配置

**进入zookeeper确认集群节点是否都注册成功：**(进入方法可查看初级入门)

​    ![0](https://note.youdao.com/yws/public/resource/e5863162eca29c2b31e8b59c9707817d/xmlnote/E199FB00F97442EA84F299A2576C141B/105210)

在zookeeper中查看主题分区对应的leader信息：

![0](https://note.youdao.com/yws/public/resource/e5863162eca29c2b31e8b59c9707817d/xmlnote/90EDE720A3A04CB596CC302AD0AA4278/82761)

**kafka将很多集群关键信息记录在zookeeper里，保证自己的无状态，从而在水平扩容时非常方便。**