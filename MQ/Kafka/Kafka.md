# Kafka 9092

## 1. 消息队列

### 1.1 消息队列应用场景

异步处理。客户端和服务端不直接交互，而是都与MQ交互。

![1621385408996](E:\SoftwareNote\MQ\Kafka\img\消息队列异步处理.png)

### 1.2 消息队列的好处

- **解耦** ：

  允许你独立的扩展或修改两边的处理过程，只要确保它们(消费者和生产者)**遵守同样的接口约束**。 

- 可恢复性：消息有持久化，系统崩溃再次上限可以继续处理。

  系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。 

- **缓冲：**有助于控制和优化数据流经过系统的速度，**解决生产消息和消费消息的处理速度不一致**的情况

- 灵活性&**削峰处理能力 **： 

  在**访问量剧增**的情况下，应用仍然需要继续发挥作用，但是这样的**突发流量并不常见(如双11)**。如果为<u>以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费</u>。**使用消息队列能够使关键组件顶住突发的访问压力**，而不会因为突发的超负荷的请求而完全崩溃。 

- 异步通信： 可以把选择权交给消费者，一些不重要的任务等待资源充足再处理。

  很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们

### 1.3 消息队列的两种模式

1. **点对点模式**：一对一，消费者主动拉取数据，消息收到后消息清除 

   消息生产者生产消息发送到Queue中，然后消息消费者从Queue中取出并且消费消息。消息被消费以后，queue 中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue 支持存在多个消费者，但是对**一个消息而言，只会有一个消费者可以消费**。

2. **发布/订阅模式**（一对多，消费者消费数据之后不会清除消息）

   消息生产者（发布）将消息发布到 topic 中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，**发布到 topic 的消息会被所有订阅(了该topic)者消费**。

## 2. Kafka概述

### 2.1 定义

Kafka 是一个**分布式**的基于**发布/订阅模式**的消息队列（Message Queue），主要应用于大数据实时处理领域。 

### 2.2 Kafka基础架构

![1621386610398](E:\SoftwareNote\MQ\Kafka\img\Kafka基础架构.png)

1）Producer ：消息生产者，就是向 kafka broker 发消息的客户端； 

2）Consumer ：消息消费者，向 kafka broker 取消息的客户端； 

3）Consumer Group （CG）：消费者组，由多个 consumer 组成。**<u>消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费(因此最优解应该是消费组数=分区数)</u>**；消费者组之间互不影响。**所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者**。 

4）Broker ：一台 kafka 服务器就是一个 broker。**一个集群由多个 broker 组成**。一个 broker可以容纳多个 topic。 

5）Topic ：可以理解为一个队列，生产者和消费者面向的都是一个 topic； 

6）Partition(分区)：为了实现扩展性，一个非常大的 topic 可以分布到多个 broker（即服务器）上，**一个 topic 可以分为多个 partition，每个 partition 是一个有序的队列**；  

7）Replica：副本，为保证集群中的某个节点发生故障时，该节点上的 partition 数据不丢失，且 kafka 仍然能够继续工作，kafka 提供了副本机制，**一个 topic 的每个分区都有若干个副本，一个 leader 和若干个 follower**

8）leader：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是 leader。 

9）follower：每个分区多个副本中的“从”，实时从 leader 中同步数据，保持和 leader 数据的同步。leader 发生故障时，某个 follower 会成为新的 follower。 

**层次关系：Kafka Cluster<Broker<Topic<Partition<Replica<Leader,Follower>>>>>**

## 3. Kafka 配置

### 3.1 配置文件详解 service.config

```properties
#broker 的全局唯一编号，不能重复
#集群用该id区分
broker.id=0
#删除 topic 功能开启
delete.topic.enable=true
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘 IO 的现成数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的缓冲区大小
socket.request.max.bytes=104857600
#kafka 运行日志（其实是数据+日志）存放的路径
log.dirs=/opt/module/kafka/logs
#topic 在当前 broker 上的分区个数
num.partitions=1
#用来恢复和清理 data 下数据的线程数量
num.recovery.threads.per.data.dir=1
#segment 文件保留的最长时间，超时将被删除（单位为小时，168小时=7天）
log.retention.hours=168
#配置连接 Zookeeper 集群地址
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181
```

- 后台运行: `bin/kafka-server-start.sh -daemon config/server.properties`

### 3.2 使用Docker

```shell
1. 启动Zookeeper
    docker run -d --name zookeeper -p 2182:2181 -t wurstmeister/zookeeper

2. 启动Kafka
    docker run  -d --name kafka \
    -p 9092:9092 \
    -e KAFKA_BROKER_ID=0 \
    -e KAFKA_ZOOKEEPER_CONNECT=192.168.1.170:2182 \  ## Zookeeper服务发现
    -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.170:9092 \
    -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t docker.io/wurstmeister/kafka 

3. 搭建Kafka集群
    docker run -d --name kafka1 \
    -p 9093:9093 \
    -e KAFKA_BROKER_ID=1 \
    -e KAFKA_ZOOKEEPER_CONNECT=<宿主机IP>:2181 \
    -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://<宿主机IP>:9093 \
    -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9093 -t wurstmeister/kafka
    
4. 命令:可进入容器内部 docker exec -it kafka0 /bin/bash
也可在容器外部
docker exec -it kafka0 /opt/kafka/bin/kafka-topics.sh --list --zookeeper 192.168.1.170:2182
```

## 4. Kafka命令

1）查看当前服务器中的所有 topic :  --list

`kafka-topics.sh --zookeeper  zookeeperhost:2181 --list `

2）创建 topic : --create

`kafka-topics.sh --zookeeper  hadoop102:2181 --create --replication-factor 3 --partitions 1 --topic first `

选项说明： 

--topic 定义 topic 名 

--replication-factor 定义副本数 

--partitions 定义分区数 

3）删除 topic  : --delete

`kafka-topics.sh --zookeeper  hadoop102:2181 --delete --topic first `

需要 server.properties 中设置 delete.topic.enable=true 否则只是标记删除。 

4）发送消息 : kafka-console-producer.sh --broker-list

`kafka-console-producer.sh --broker-list hadoop102:9092 --topic first `

\>hello world 

\>atguigu atguigu 

5）消费消息 

```shell
kafka-console-consumer.sh --zookeeper hadoop102:2181 --topic first

kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic first

# --from-beginning：会把主题中以往所有的数据都读取出来。
kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --from-beginning --topic first
```

6）查看某个 Topic 的详情  : --describe

`kafka-topics.sh --zookeeper  hadoop102:2181 --describe --topic first `

7）修改分区数  :  --alter

`kafka-topics.sh --zookeeper  hadoop102:2181 --alter --topic first --partitions 6`

## 5. Kafka 工作流程及文件存储机制

- 分区会把数据带上offset存在.log文件，可以记录消费位置

Kafka 中消息是以 topic 进行分类的，生产者生产消息，消费者消费消息，都是面向 topic 的。

**topic 是逻辑上的概念，而 partition 是物理上的概念**，每个 partition 对应于一个 log 文件（实际存的是数据），该 **log 文件中存储的就是 producer 生产的数据**。Producer 生产的数据会被不断追加到该log 文件末端，且**每条数据都有自己的 offset**。消费者组中的每个消费者，都会实时记录自己消费到了哪个 offset，**以便出错恢复时，从上次的位置继续消费**。 

- 分片和索引

由于生产者生产的消息会不断追加到 log 文件末尾，为防止 log 文件过大导致数据定位效率低下，Kafka 采取了**分片和索引**机制，将每个 partition 分为多个 segment。每个 segment对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名 

规则为：**topic 名称+分区序号**。例如，first 这个 topic 有三个分区，则其对应的文件夹为 first- 0,first-1,first-2。 

index 和 log 文件以当前 segment 的第一条消息的 offset 命名。下图为 index 文件和 log 文件的结构示意图。

**“.index”文件存储大量的索引信息，“.log”文件存储大量的数据**，索引文件中的元 数据指向对应数据文件中 message 的物理偏移地址。 

```tex
00000000000000000000.index   # offset 为0
00000000000000000000.log	
00000000000000170410.index	 # offset 为170410
00000000000000170410.log
00000000000000239430.index 	 # offset 为239430
00000000000000239430.log
```

![1621392630185](E:\SoftwareNote\MQ\Kafka\img\index文件和log文件.png)

## 5. Kafka生产者

### 5.1 分区策略

#### 5.1.1 分区的原因

（1）**方便在集群中扩展**，每个 Partition 可以通过调整以适应它所在的机器，而一个 topic 

又可以有多个 Partition 组成，因此整个集群就可以适应任意大小的数据了； 

（2）可以**提高并发**，因为可以以 Partition 为单位读写了。 

#### 5.1.2 分区原则

我们需要将 producer 发送的数据封装成一个 ProducerRecord 对象。  

ProducerRecode(@NotNull String, Integer partition, Long timestamp, String key, String value, @Nullable Iterable<Header> headers)；

（1）指明 partition的情况下，直接将指明的值直接作为 partiton 值； 即将数据存至指定分区。

（2）没有指明partition值但有key 的情况下，将 key 的 hash 值与 topic 的partition数取余得算出分区 

（3）既没有partition值又没有key值的情况下，第一次调用时**随机**生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数**取余(还是取余，只是数字是随机生产而不是key的hash)**得到 partition值，也就是常说的 round-robin 算法。 

### 5.2 数据可靠性保证：发送ack

为保证 producer 发送的数据，能可靠的发送到指定的 topic，topic 的每个 partition 收到producer 发送的数据后，都需要向 producer 发送 **ack（acknowledgement 确认收到）**，如果producer 收到 ack，就会进行下一轮的发送，**否则重新发送数据(因此要确保ack正确返回，否则会消息重复)**。 

#### 5.2.1 **何时发送ack**

确保follower和leader同步完成，这样才能在leader挂了再选出新的leader

![1621393994949](E:\SoftwareNote\MQ\Kafka\img\ack发送机制.png)

#### 5.2.2 副本数据同步策略

|               方案                |                             优点                             |                             缺点                             |
| :-------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 半数 **以上**完成同步，就发送 ack |                            延迟低                            | 选举新的 leader 时,容忍n台节点的故障,需要2n+1个副本(半数以上即>n+1同步完成，最坏的情况就是其余都没同步完成) |
|          全部完成才发送           | 选举新leader时，如果有n台挂掉，只需要n+1个副本(即有一台数据全就行) |                            延迟高                            |

Kafka 选择了第二种方案（全部同步完成才发送ack），然后**在2的基础上增加了ISR**，原因如下： 

1.同样为了容忍 n 台节点的故障，第一种方案需要 2n+1 个副本，而第二种方案只需要 n+1个副本，而 Kafka 的每个分区都有大量的数据，第一种方案会造成大量数据的冗余。 

2.虽然第二种方案的网络延迟会比较高，但网络延迟对 Kafka 的影响较小。

#### 5.2.3 ISR ： in-sysn replica set

采用第二种方案之后，设想以下情景：leader 收到数据，所有 follower 都开始同步数据，但有一个 follower，因为某种故障，迟迟不能与 leader 进行同步，那 leader 就要一直等下去，直到它完成同步，才能发送 ack。这个问题怎么解决呢？ 

**Leader 维护了一个动态的 in-sync replica set (ISR)，意为和 leader 保持同步的 follower 集合。当 ISR 中的 follower 完成数据的同步之后，leader 就会给 follower 发送 ack。如果 follower长时间 未 向 leader 同 步 数 据 ， 则 该 follower 将 被 踢 出 ISR ， 该 时 间 阈 值 由replica.lag.time.max.ms 参数设定。Leader 发生故障之后，就会从 ISR 中选举新的 leader。**

#### 5.2.4 ack应答机制

对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等 ISR 中的 follower 全部接收成功。所以 Kafka 为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡， 选择以下的配置。

**acks 参数配置： ** switch (acks)

​	0：producer 不等待 broker 的 ack，这一操作提供了一个**最低的延迟**，broker 一接收到还没有写入磁盘就已	经返回，当 broker 故障时有可能丢失数据； (**只管发送，不管有没有接收到，可能数据丢失**)

​	1：producer 等待 broker 的 ack，partition 的 leader 落盘成功后返回 ack，如果在 follower 同步成功之前 leader 故障，那么将会丢失数据；（**leader落盘成功就返回ack，leader故障+follower没同步，则数据丢失**）

​	-1（all）：producer 等待 broker 的 ack，partition 的 leader 和 follower 全部落盘成功后才返回 ack。但是如果在 follower 同步完成后，broker 发送 ack 之前，leader 发生故障，那么会造**成数据重复** (**同步成功却没发送ack，这样producer会再次发送重复数据**)。 （*也有极端情况的数据丢失：ack机制是配合ISR使用的，如果ISR中只有leader，然后leader故障了，此时follower没有同步过数据，造成数据丢失*）

#### 5.2.5 故障处理细节

![1621396011025](E:\SoftwareNote\MQ\Kafka\img\Log中的LEO和HW.png)

- **Log文件中的HW和LEO ：**

  LEO：指的是每个副本最大的 offset； 

  HW：指的是**消费者能见到的最大的 offset**，即ISR 队列中最小的 LEO。

（1）follower 故障 

follower 发生故障后会被临时踢出 ISR，待该 follower 恢复后，follower 会读取本地磁盘记录的上次的 HW，并将 log 文件高于 HW 的部分截取掉，从 HW 开始向 leader 进行同步。等该 follower 的 LEO 大于等于该 Partition 的 HW，即 follower 追上 leader 之后，就可以重新加入 ISR 了

（2）leader 故障 ：

leader 发生故障之后，会从 ISR 中选出一个新的 leader，之后，为保证多个副本之间的数据一致性，**其余的 follower 会先将各自的 log 文件高于 HW 的部分截掉**，然后**从新的 leader同步数据**。 

***注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复*** 。（数据不丢失和不重复由ack）

### 5.2 Exactly Once 语义

- **At Least Once **：将服务器的 ACK 级别设置为-1，可以保证 Producer 到 Server 之间不会丢失数据。

  可以保证数据不丢失，但是不能保证数据不重复

-  **At Most Once **： 将服务器 ACK 级别设置为 0，可以保证生产者每条消息只会被发送一次

  可以保证数据不重复，但是不能保证数据不丢

- **Exactly Once**： 对于一些非常重要的信息，比如说交易数据，下游数据消费者要求数据既不重复也不丢失

在 <u>0.11 版本以前的 Kafka</u>，对此是无能为力的，只能保证数据不丢失，再**在下游消费者对数据做全局去重**。对于多个下游应用的情况，每个都需要单独做全局去重，这就对性能造成了很大影响。

#### 5.2.1 <u>0.11 版本的 Kafka</u>，引入了一项重大特性：**幂等性**。

所谓的幂等性就是指 Producer 不论向 Server 发送多少次重复数据，Server 端都只会持久化一条。

**At Least Once + 幂等性 = Exactly Once**

要启用幂等性，只需要将 Producer 的参数中 *enable.idompotence 设置为 true* 即可。

**Kafka的幂等性实现其实就是将原来下游需要做的去重放在了数据上游**。开启幂等性的 Producer 在初始化的时候会被分配一个 PID，发往同一 Partition 的消息会附带 Sequence Number。而Broker 端会对<PID, Partition, SeqNumber>做缓存，当具有相同主键的消息提交时，Broker 只会持久化一条。

**但是 PID 重启就会变化，同时不同的 Partition 也具有不同主键，所以幂等性无法保证跨分区跨会话(单分区会话)的 Exactly Once。**

## 6. 消费者

同一个消费者组中的消费者，同一时刻只能有一个消费者消费 

### 6.1 消费方式

consumer 采用 pull（拉）模式从 broker 中读取数据（**即consumer主动请求数据** ）。 

push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker 决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成 consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而 pull 模式则可以根据 consumer 的消费能力以适的速率消费消息。

**pull 模式不足之处是，如果 kafka 没有数据，消费者可能会陷入循环中，一直返回空数据**。针对这一点，Kafka 的消费者**在消费数据时会传入一个时长参数 timeout(就像trylock)**，如果当前没有数据可供消费，consumer 会等待一段时间之后再返回，这段时长即为 timeout。 

### 6.2 分区分配策略

**一个 consumer group 中有多个 consumer，一个 topic 有多个 partition**，所以必然会涉及到 partition 的分配问题，即确定那个 partition 由哪个 consumer 来消费。Kafka 有两种分配策略，一是 RoundRobin，一是 Range。

`partition.assignment.strategy`参数选择 range 或 roundrobin。`partition.assignment.strategy`参数默认的值是range。 



在 kafka 中，存在着两种分区分配策略。**一种是 RangeAssignor 分配策略(范围分区)，另一种是 RoundRobinAssignor分配策略(轮询分区)**。默认采用 Range 范围分区。 Kafka提供了消费者客户端参数 partition.assignment.strategy 用来设置消费者与订阅主题之间的分区分配策略。默认情况下，此参数的值为：org.apache.kafka.clients.consumer.RangeAssignor，即采用RangeAssignor分配策略 

    1.1 RangeAssignor 范围分区
    
       Range 范围分区策略是对每个 topic 而言的。首先对同一个 topic 里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序。假如现在有 10 个分区，3 个消费者，排序后的分区将会是0,1,2,3,4,5,6,7,8,9；消费者排序完之后将会是C1-0,C2-0,C3-0。通过 partitions数/consumer数 来决定每个消费者应该消费几个分区。如果除不尽，那么前面几个消费者将会多消费 1 个分区。
    
       例如，10/3 = 3 余 1 ，除不尽，那么 消费者 C1-0 便会多消费 1 个分区，最终分区分配结果如下：
C1-0	消费 0,1,2,3 分区
C2-0	消费 4,5,6 分区
C3-0	消费 7,8,9 分区(如果有11 个分区的话，C1-0 将消费0,1,2,3 分区，C2-0 将消费4,5,6,7分区   C3-0 将消费 8,9,10 分区)

 Range 范围分区的弊端：

       如上，只是针对 1 个 topic 而言，C1-0消费者多消费1个分区影响不是很大。如果有 N 多个 topic，那么针对每个 topic，消费者 C1-0 都将多消费 1 个分区，topic越多，C1-0 消费的分区会比其他消费者明显多消费 N 个分区。这就是 Range 范围分区的一个很明显的弊端了
    
      由于 Range 范围分区存在的弊端，于是有了 RoundRobin 轮询分区策略，如下介绍↓↓↓
    
    1.2 RoundRobinAssignor 轮询分区
    
        RoundRobin 轮询分区策略，是把所有的 partition 和所有的 consumer 都列出来，然后按照 hascode 进行排序，最后通过轮询算法来分配 partition 给到各个消费者。
    
        轮询分区分为如下两种情况：①同一消费组内所有消费者订阅的消息都是相同的    ②同一消费者组内的消费者锁定月的消息不相同    
    
        ①如果同一消费组内，所有的消费者订阅的消息都是相同的，那么 RoundRobin 策略的分区分配会是均匀的。
    
        例如：同一消费者组中，有 3 个消费者C0、C1和C2，都订阅了 2 个主题 t0  和 t1，并且每个主题都有 3 个分区(p0、p1、p2)，那么所订阅的所以分区可以标识为t0p0、t0p1、t0p2、t1p0、t1p1、t1p2。最终分区分配结果如下：
消费者C0	消费 t0p0 、t1p0 分区
消费者C1	消费 t0p1 、t1p1 分区
消费者C2	消费 t0p2 、t1p2 分区

       ②如果同一消费者组内，所订阅的消息是不相同的，那么在执行分区分配的时候，就不是完全的轮询分配，有可能会导致分区分配的不均匀。如果某个消费者没有订阅消费组内的某个 topic，那么在分配分区的时候，此消费者将不会分配到这个 topic 的任何分区。
    
        例如：同一消费者组中，有3个消费者C0、C1和C2，他们共订阅了 3 个主题：t0、t1 和 t2，这 3 个主题分别有 1、2、3 个分区(即:t0有1个分区(p0)，t1有2个分区(p0、p1)，t2有3个分区(p0、p1、p2))，即整个消费者所订阅的所有分区可以标识为 t0p0、t1p0、t1p1、t2p0、t2p1、t2p2。具体而言，消费者C0订阅的是主题t0，消费者C1订阅的是主题t0和t1，消费者C2订阅的是主题t0、t1和t2，最终分区分配结果如下：
消费者C0	消费 t0p0
消费者C1	消费 t1p0 分区
消费者C2	消费 t1p1、t2p0、t2p1、t2p2 分区

RoundRobin轮询分区的弊端：

       从如上实例，可以看到RoundRobin策略也并不是时分完美，这样分配其实并不是最优解，因为完全可以将分区 t1p1 分配给消费者 C1。
    
      所以，如果想要使用RoundRobin 轮询分区策略，必须满足如下两个条件：
    
      ①每个消费者订阅的主题，必须是相同的
    
      ②每个主题的消费者实例都是相同的。(即：上面的第一种情况，才优先使用 RoundRobin 轮询分区策略)
### 6.3 offset的维护

由于 consumer 在消费过程中可能会出现断电宕机等故障，consumer 恢复后，需要从故障前的位置的继续消费，所以 consumer 需要实时记录自己消费到了哪个 offset，以便故障恢复后继续消费

- Kafka存在Zookeeper中的节点

  ![1621406213157](E:\SoftwareNote\MQ\Kafka\img\Kafka存在Zookeeper中的节点.png)

- 修改配置文件 consumer.properties 

`exclude.internal.topics=false`

- 读取 offset 

  - 0.11.0.0 之前版本: 

    ```shell
    kafka-console-consumer.sh --topic __consumer_offsets --zookeeper hadoop102:2181 \
    --formatter "kafka.coordinator.GroupMetadataManager\$OffsetsMessageFormatter" \
    --consumer.config config/consumer.properties --from-beginning
    ```

  - 0.11.0.0 之后版本(含): 

    ```shell
    kafka-console-consumer.sh --topic __consumer_offsets --zookeeper hadoop102:2181 \
    --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageForm \
    atter" --consumer.config config/consumer.properties --from-beginning
    ```

    

## 7. Kafka的高效读写数据

1）顺序写磁盘 

Kafka 的 producer 生产数据，要写入到 log 文件中，写的过程是一直追加到文件末端， 为顺序写。官网有数据表明，同样的磁盘，顺序写能到 600M/s，而随机写只有 100K/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。 

2）零复制技术：省去用户层面的拷贝，直接由操作系统拷贝

![1621408322267](E:\SoftwareNote\MQ\Kafka\img\Kafka的零拷贝技术.png)

## 8. Zookeeper 在 Kafka 中的作用 

Kafka 集群中有一个 broker 会被选举为 Controller，负责管理集群 broker 的上下线，所 有 topic 的分区副本分配和 leader 选举等工作。 Controller 的管理工作都是依赖于 Zookeeper 的。

- Kafka在Zookeeper中的选举流程

![1621408552487](E:\SoftwareNote\MQ\Kafka\img\Kafka在Zookeeper中的选举流程.png)

## 9. Kafka事务

Kafka 从 0.11 版本开始引入了事务支持。事务可以保证 Kafka 在 Exactly Once 语义的基础上，生产和消费可以**跨分区和会话**，要么全部成功，要么全部失败。 

### 9.1 Producer 事务 

为了实现跨分区跨会话的事务，需要引入一个**全局唯一的 Transaction ID**，并将 **Producer 获得的PID 和Transaction ID 绑定**。<u>这样当Producer 重启后就可以通过正在进行的 Transaction ID 获得原来的  PID</u>。为了管理 Transaction，Kafka 引入了一个新的组件 Transaction Coordinator。Producer 就是通过和 Transaction Coordinator 交互获得 Transaction ID 对应的任务状态。Transaction Coordinator 还负责将事务所有写入 Kafka 的一个内部 Topic，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。 

### 9.2 Consumer 事务 

上述事务机制主要是从 Producer 方面考虑，对于 Consumer 而言，事务的保证就会相对较弱，尤其时无法保证 Commit 的信息被精确消费。这是由于 Consumer 可以通过 offset 访问任意信息，而且不同的 Segment File 生命周期不同，同一事务的消息可能会出现重启后被删除的情况

## 10. API

- Kafka 的 Producer 发送消息采用的是异步发送的方式。在消息发送的过程中，涉及到了 

  两个线程——main 线程和 Sender 线程，以及一个线程共享变量——RecordAccumulator。 

  main 线程将消息发送给 RecordAccumulator，Sender 线程不断从 RecordAccumulator 中拉取 

  消息发送到 Kafka broker。

  相关参数：  batch.size和linger.ms二符合一就会发送数据

  batch.size：只有数据积累到 batch.size 之后，sender 才会发送数据。 

  linger.ms：如果数据迟迟未达到 batch.size，sender 等待 linger.time 之后就会发送数据。

- Consumer 消费数据时的可靠性是很容易保证的，因为数据在 Kafka 中是持久化的，故不用担心数据丢失问题,但是需要可靠的offset去查找topic底下某分区里面的数据，因此offset的准确性是consumer的关键

  enable.auto.commit：是否开启自动提交 offset 功能 

  auto.commit.interval.ms：自动提交 offset 的时间间隔

  - 自动提交offset：开发人员难以把握offset 提交的时机。

    - 先处理后提交offset,会造成重读消费（处理后offset还未提交，consumer挂了，offset是旧的）  
    - 先提交offset后处理,会造成数据丢失 （先offset服务器挂了，offset之前的数据还没处理完成）

  - 手动提交offset：是 commitSync（同步提交）和 commitAsync（异步提交）。两者的相同点是，都会将本次 poll 的一批数据最高的偏移量提交；不同点是： 

    - commitSync 阻塞当前线程，一直到提交成功，并且会**自动失败重试**（由不可控因素导致，也会出现提交失败）；

    - 而 commitAsync 则没有失败重试机制，故有可能提交失败

  - 自定义存储offset：重写ConsumerRebalanceListener 

- 自定义拦截器

  重写ProducerInterceptor 

## 11. SpringCloud整合Kafka

- pom

  ```xml
  <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-stream-kafka</artifactId>
  </dependency>
  <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-stream-binder-kafka</artifactId>
  </dependency>
  ```

- yml

  ```yaml
  spring:
    cloud:
      stream:
        kafka:
          binder:
            brokers:  192.168.1.170:9092,192.168.1.170:9093,192.168.1.170:9094  # kafka服务地址和端口
            zk-nodes: 192.168.1.170:2182  # ZK的集群配置地址和端口
  ```

  

## 12. Kafka 监控： Kafka Eagle  



## 13. 面试题

https://blog.csdn.net/C_Xiang_Falcon/article/details/100917145



![1621430230459](C:\Users\ASUS\AppData\Local\Temp\1621430230459.png)

![1621430314772](C:\Users\ASUS\AppData\Local\Temp\1621430314772.png)