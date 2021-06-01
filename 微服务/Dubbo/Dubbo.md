# Dubbo

## 1. Web系统发展演变 

![1621502746942](E:\SoftwareNote\微服务\Dubbo\img\Web系统发展演变.png)

- #### *单一应用架构*

  当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。

  适用于小型网站，小型管理系统，将所有功能都部署到一个功能里，简单易用。

  缺点： 1、性能扩展比较难 

  ​            2、协同开发问题

  ​            3、不利于升级维护

- #### *垂直应用架构*

  当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。

  通过切分业务来实现各个模块独立部署，降低了维护和部署的难度，团队各司其职更易管理，性能扩展也更方便，更有针对性。

  缺点： 公用模块无法重复利用，开发性的浪费

- #### *分布式服务架构*

  当垂直应用越来越多，应用之间交互不可避免，将**核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心**，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的**分布式服务框架(RPC)**是关键。

- #### *流动计算架构*

  当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个**调度中心**基于访问压力实时管理集群容量，提高集群利用率。此时，用于**提高机器利用率的资源调度和治理中心(SOA)[ Service Oriented Architecture]**是关键。

## 2. RPC ： Remote Procedure Call

### 2.1 *什么叫RPC*

RPC【Remote Procedure Call】是指远程过程调用，是一种进程间通信方式，他*是一种技术的思想*，而不是规范。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的函数，**本质上编写的调用代码基本相同(就是封装过了，程序员只需向调用本地方法一样调用接口即可)**。

### 2.2 RCP基本原理

**RPC两个核心模块：通讯（与远程通信），序列化（RCP性能的关键）。**

![1621503237651](E:\SoftwareNote\微服务\Dubbo\img\RCP基本原理图1.png)

![1621503270785](E:\SoftwareNote\微服务\Dubbo\img\RCP基本原理图2.png)

## 3. Dubbo的核心概念

### 3.1 简介

Apache Dubbo (incubating) |ˈdʌbəʊ| 是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。（依赖Zookeeper）

### 3.2 Dubbo架构

![1621503575198](E:\SoftwareNote\微服务\Dubbo\img\Dubbo架构图.png)

**服务提供者（Provide）**：暴露服务的服务提供方，服务提供者在启动时，<u>向注册中心注册自己提供的服务</u>。

**服务消费者（Consumer）**: 调用远程服务的服务消费方，服务消费者在启动时，向注册中心**订阅**自己所需的服务，服务消费者，从提供者地址列表中，**基于软负载均衡算法**，选一台提供者进行调用，如果调用失败，再选另一台调用。

**注册中心（Registry）**：注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者

**监控中心（Monitor）**：服务消费者和提供者，在**内存中累计调用次数和调用时间**，定时每分钟发送一次统计数据到监控中心

- Ø 调用关系说明
  - 服务容器负责启动，加载，运行服务提供者。
  - 服务提供者在启动时，向注册中心注册自己提供的服务。
  - 服务消费者在启动时，向注册中心订阅自己所需的服务。
  - 注册中心返回服务提供者地址列表给消费者，**如果有变更，注册中心将基于长连接推送变更数据给消费者。**
  - 服务消费者，从提供者地址列表中，**基于软负载均衡算法，选一台提供者进行调用**，如果调用失败，再选另一台调用。
  - 服务消费者和提供者，在内存中累计调用次数和调用时间，**定时每分钟发送一次统计数据到监控中心**。

## 4. 使用

官方手册： https://dubbo.apache.org/zh/docs/

### 4.1 基于注解

- pom

```xml
<!-- dubbo-start -->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
</dependency>

<!-- 
curator和zookeeper相关 ：
Apache Curator是为ZooKeeper开发的一套Java客户端类库，它是一个分布式协调服务。
Apache Curator解决了哪些问题？

    封装ZooKeeper client与ZooKeeper server之间的连接处理;
    提供了一套Fluent风格的操作API;
    提供ZooKeeper各种应用场景(分布式锁，选举)的抽象封装.
-->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.0.1</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.0.1</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.7</version>
</dependency>
```

- yml

```yaml
dubbo:
  application:
    name:  dubbo-consumer # 服务名
  registry:
    address: 192.168.1.170:2182
    # 读者请换成自己的zookeeperip
    protocol: zookeeper
    check: false
  monitor:
    protocol: register
  consumer:
    check:  false
    timeout: 3000
    
    
# 提供者配置：
# dubbo.application.name=gmall-user
# dubbo.registry.protocol=zookeeper
# dubbo.registry.address=192.168.67.159:2181
# dubbo.scan.base-package=com.atguigu.gmall
# dubbo.protocol.name=dubbo
# application.name就是服务名，不能跟别的dubbo提供端重复
# registry.protocol 是指定注册中心协议
# registry.address 是注册中心的地址加端口号
# protocol.name 是分布式固定是dubbo,不要改。
# base-package  注解方式要扫描的包
# 消费者配置：
# dubbo.application.name=gmall-order-web
# dubbo.registry.protocol=zookeeper
# dubbo.registry.address=192.168.67.159:2181
# dubbo.scan.base-package=com.atguigu.gmall
# dubbo.protocol.name=dubbo
```

- 注解

```java
@EnableDubbo // 启动类注解，开启Dubbo支持
@DubboReference // 声明该注入为RCP调用，dubbo需要去调用远程服务，@Since2.7.7 原注解为@Reference
private IProducerService producerService;    
@DubboService//声明该service需要注册到dubbo中 @Since2.7.7 原注解为@Sevice
    
```

### 4.2 Dubbo配置

#### 4.2.1 配置优先级原则

JVM 启动 -D 参数优先，这样可以使用户在部署和启动时进行参数重写，比如在启动时需改变协议的端口。

XML 次之，如果在 XML 中有配置，则 dubbo.properties 中的相应配置项无效。

Properties 最后，相当于缺省值，只有 XML 没有配置时，dubbo.properties 的相应配置项才会生效，通常用于共享公共配置，比如应用名。

![1621505086592](E:\SoftwareNote\微服务\Dubbo\img\Dubbo配置优先级原则.png)

#### 4.2.2 重试次数

失败自动切换，当出现失败，重试其它服务器，但**重试会带来更长延迟/重复消费等**。可通过 retries="2" 来设置重试次数(不含第一次)。

因此，在幂等性(调用多次不会影响结果：查询/删除/更新)的方法调用，可以有重试次数。非幂等性调用（新增），不要设置重试

```xml
<!--方式1. 重试次数配置如下：-->
<dubbo:service retries="2" />
<!--方式2. -->
<dubbo:reference retries="2" />
<!--方式3. -->
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>#
```

#### 4.2.3 超时时间 

由于网络或服务端不可靠，会导致调用出现一种不确定的中间状态（超时）。为了**避免超时导致客户端资源（线程）挂起耗尽**，必须设置超时时间。

- 消费端配置

```xml
全局超时配置
<dubbo:consumer timeout="5000" />

指定接口以及特定方法超时配置
<dubbo:reference interface="com.foo.BarService" timeout="2000">
    <dubbo:method name="sayHello" timeout="3000" />
</dubbo:reference>
```

- 服务端配置

```xml
全局超时配置
<dubbo:provider timeout="5000" />

指定接口以及特定方法超时配置
<dubbo:provider interface="com.foo.BarService" timeout="2000">
    <dubbo:method name="sayHello" timeout="3000" />
</dubbo:provider>
```

- 配置原则

dubbo推荐在Provider上尽量多配置Consumer端属性：

1. 作**服务的提供者，比服务使用方更清楚服务性能参数**，如调用的超时时间，合理的重试次数，等等

2. 在Provider配置后，Consumer不配置则会使用Provider的配置值，即**Provider配置可以作为Consumer的缺省值**。否则，Consumer会使用Consumer端的全局设置，这对于Provider不可控的，并且往往是不合理的

3. 配置的覆盖规则：

   1) 方法级配置别优于接口级别，即小Scope优先 

   2) Consumer端配置 优于 Provider配置 优于 全局配置，

   3) 最后是Dubbo Hard Code的配置值（见配置文档）

   ![1621505712149](E:\SoftwareNote\微服务\Dubbo\img\Dubbo配置的覆盖规则.png)

#### 4.2.4 版本号

当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用。

可以按照以下的步骤进行版本迁移：（灰度发布）

在低压力时间段，先升级一半提供者为新版本

再将所有消费者升级为新版本

然后将剩下的一半提供者升级为新版本

```xml
老版本服务提供者配置：
<dubbo:service interface="com.foo.BarService" version="1.0.0" />

新版本服务提供者配置：
<dubbo:service interface="com.foo.BarService" version="2.0.0" />

老版本服务消费者配置：
<dubbo:reference id="barService" interface="com.foo.BarService" version="1.0.0" />

新版本服务消费者配置：
<dubbo:reference id="barService" interface="com.foo.BarService" version="2.0.0" />

如果不需要区分版本，可以按照以下的方式配置：
<dubbo:reference id="barService" interface="com.foo.BarService" version="*" />
```

## 5. 高可用

### 5.1 zookeeper宕机与dubbo直连

现象：**zookeeper注册中心宕机，还可以消费dubbo暴露的服务**。

原因：

健壮性

l 监控中心宕掉不影响使用，只是丢失部分采样数据

l 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务

l 注册中心对等集群，任意一台宕掉后，将自动切换到另一台

l **注册中心全部宕掉后，服务提供者和服务消费者仍能通过*本地缓存通讯***

l 服务提供者无状态，任意一台宕掉后，不影响使用

l 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

高可用：通过设计，减少系统不能提供服务的时间；

### 5.2 集群下dubbo负载均衡配置

在集群负载均衡时，Dubbo 提供了多种均衡策略，缺省为 random 随机调用。

- 负载均衡策略

  - **Random LoadBalance**

  随机，按权重设置随机概率。

  在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

  - **RoundRobin LoadBalance**

  轮循，按公约后的权重设置轮循比率。

  存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。

  - **LeastActive LoadBalance**

  最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。

  使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。

  - **ConsistentHash LoadBalance**

  一致性 Hash，相同参数的请求总是发到同一提供者。

  当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。算法参见：http://en.wikipedia.org/wiki/Consistent_hashing

  缺省只对第一个参数 Hash,如果要修改，请配置 <dubbo:parameter key="hash.arguments" value="0,1" />缺省用 160 份虚拟节点，如果要修改，请配置 <dubbo:parameter key="hash.nodes" value="320" />

### 5.3 Dubbo服务降级

**当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心交易正常运作或高效运作。**

可以通过服务降级功能临时屏蔽某个出错的非关键服务，并定义降级后的返回策略。

向注册中心写入动态配置覆盖规则：

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&mock=force:return+null"));
```

其中：

- mock=force:return+null 表示消费方对该服务的方法调用都直接返回 null 值，不发起远程调用。用来屏蔽不重要服务不可用时对调用方的影响。
- 还可以改为 mock=fail:return+null 表示消费方对该服务的方法调用在失败后，再返回 null 值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响。

### 5.4 集群容错

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。

- **集群容错模式**	

  - **Failover Cluster**

  失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 retries="2" 来设置重试次数(不含第一次)。

  

  重试次数配置如下：

  <dubbo:service retries="2" />

  或

  <dubbo:reference retries="2" />

  或

  \<dubbo:reference>

  ​    <dubbo:method name="findFoo" retries="2" />

  \</dubbo:reference>

  - **Failfast Cluster**

  快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

  - **Failsafe Cluster**

  失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

  - **Failback Cluster**

  失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

  - **Forking Cluster**

  并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks="2" 来设置最大并行数。

  - **Broadcast Cluster**

  广播调用所有提供者，逐个调用，任意一台报错则报错 [2]。通常用于通知所有提供者更新缓存或日志等本地资源信息。

  

- **集群模式配置**

  按照以下示例在服务提供方和消费方配置集群模式

  <dubbo:service cluster="failsafe" />

  或

  <dubbo:reference cluster="failsafe" />

## 6. dubbo原理

### 6.1 RPC原理

一次完整的RPC调用流程（同步调用，异步另说）如下： 

**1）服务消费方（client）调用以本地调用方式调用服务；** 

2）client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体； 

3）client stub找到服务地址，并将消息发送到服务端； 

4）server stub收到消息后进行解码； 

5）server stub根据解码结果调用本地的服务； 

6）本地服务执行并将结果返回给server stub； 

7）server stub将返回结果打包成消息并发送至消费方； 

8）client stub接收到消息，并进行解码； 

**9）服务消费方得到最终结果。**

RPC框架的目标就是要2~8这些步骤都封装起来，这些细节对用户来说是透明的，不可见的。

![1621513329699](E:\SoftwareNote\微服务\Dubbo\img\RPC原理.png)

### 6.2 netty通信原理 (NIO)

Netty是一个异步事件驱动的网络应用程序框架， 用于快速开发可维护的高性能协议服务器和客户端。它极大地简化并简化了TCP和UDP套接字服务器等网络编程。

![1621513404950](E:\SoftwareNote\微服务\Dubbo\img\BIO和NIO.png)

Selector 一般称 为**选择器** ，也可以翻译为 **多路复用器，**

Connect（连接就绪）、Accept（接受就绪）、Read（读就绪）、Write（写就绪）

Netty基本原理：

1. 初始化channel
2. 注册channel到selector
3. 轮询accept事件
4. 处理accept建立连接channel
5. 注册channel到selector
6. 轮询读写事件
7. 处理读写事件

![1621513527657](E:\SoftwareNote\微服务\Dubbo\img\Netty基本原理.png)

### 6.3 dubbo框架

![1621513683719](E:\SoftwareNote\微服务\Dubbo\img\Dubbo框架图.png)

- config 配置层：对外配置接口，以 ServiceConfig, ReferenceConfig 为中心，可以直接初始化配置类，也可以通过 spring 解析配置生成配置类
- proxy 服务代理层：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以 ServiceProxy 为中心，扩展接口为 ProxyFactory
- registry 注册中心层：封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 RegistryFactory, Registry, RegistryService
- cluster 路由层：封装多个提供者的路由及负载均衡，并桥接注册中心，以 Invoker 为中心，扩展接口为 Cluster, Directory, Router, LoadBalance
- monitor 监控层：RPC 调用次数和调用时间监控，以 Statistics 为中心，扩展接口为 MonitorFactory, Monitor, MonitorService
- protocol 远程调用层：封装 RPC 调用，以 Invocation, Result 为中心，扩展接口为 Protocol, Invoker, Exporter
- exchange 信息交换层：封装请求响应模式，同步转异步，以 Request, Response 为中心，扩展接口为 Exchanger, ExchangeChannel, ExchangeClient, ExchangeServer
- transport 网络传输层：抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel, Transporter, Client, Server, Codec
- serialize 数据序列化层：可复用的一些工具，扩展接口为 Serialization, ObjectInput, ObjectOutput, ThreadPool

### 6.4 dubbo服务暴露（注册）

![1621513855350](E:\SoftwareNote\微服务\Dubbo\img\dubbo服务暴露.png)

### 6.5 dubbon服务引用

![1621513920716](E:\SoftwareNote\微服务\Dubbo\img\dubbon服务引用.png)

### 6.6 dubbo服务调用

![1621513963313](E:\SoftwareNote\微服务\Dubbo\img\dubbo服务调用.png)