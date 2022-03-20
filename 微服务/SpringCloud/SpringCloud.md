# SpringCloud

## 1. 版本选择https://start.spring.io/actuator/info

![1606651696686](img\SpringCloud版本细项选择.png)

![1606651411340](img\SpringCloud版本选择.png)

![1606651789339](img\杨哥学习版本选择.png)

## 2. SpringCloud组件升级

![1606652566614](img\SpringCloud组件升级.png)

## 3. 注解

### 3.1 @RequestBody

- 请求参数为实体类，需要增加参数注解,前端传参应为json串

```java
@PostMapping("/payment/create")
    public CommonResult create(@RequestBody Payment payment) {
        log.info("进入方法PaymentController.create.result=" + result);
        return null;
    }
```

## 4.组件们

### 4.1 服务注册与发现

- 负载均衡     @LoadBalanced

```java
spring:
  application:
    name: cloud-payment-service  # Eureka发现需要
    
// 访问连接用服务名大写
public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";

// 开启负载均衡     @LoadBalanced
@Configuration
public class ApplicationContextConfig {

    @Bean
    //赋予RestTemplate负载均衡的能力
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        RestTemplate rt = new RestTemplate();
        // 可增加拦截器，做一些请求增强操作，比如增加header的token
        rt.setInterceptors(Collections.singletonList(tokenInterceptor));
        return rt;
    }
}


```

- 服务发现 @EnableDiscoveryClient

```java
// 服务发现
@EnableDiscoveryClient
SpringBootApplication
@Resource
private DiscoveryClient discoveryClient;

List<String> services = discoveryClient.getServices();
List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
```

#### 4.1.1 Eureka

```yaml
## server   @EnableEurekaServer
eureka:
  instance:
    hostname: eureka7002.com  #eureka服务端的实例名字
  client:
    register-with-eureka: false    #表识不向注册中心注册自己
    fetch-registry: false   #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务
    service-url:
      #设置与eureka server交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7003.com:7003/eureka/
      

## client   @EnableEurekaServer 
eureka:
  client:
    register-with-eureka: true
    fetchRegistry: true
    service-url:
      # 不全部注册也可以， Eureka集群会自动拷贝
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: payment8001
    
```



##### 4.1.1.1 Eureka系统架构

- 客户服务+Eureka服务集群+提供者服务集群
- 提供者服务集群会注册在Eureka集群中，客户服务调用前会先向Eureka查询注册信息，发现提供者URL再进行调用
- eureka继承了Ribbon
- @LoadBalanced // 注释了这个，服务间才可以发现彼此，用服务名调用，而不需要ip+port

![1606749404796](img\Eureka系统架构.png)

##### 4.1.1.2 Eureka组成

- Eureka Server(@EnableEurekaServer)和Eureka Client(@EnableEurekaClient)
- Eureka Server提供服务注册服务
- Eureka Client通过注册中心进行访问。是一个JAVA客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的/使用轮询负载算法的负载均衡器。再应用启动后，将会向Euerka Server发送心跳（默认**30s**）。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除（默认90s）

##### 4.1.1.3 Eureka集群原理 

![1607007154428](img\Eureka集群原理.png)

##### 4.1.1.4 Eureka 自我保护机制

- 注册在Eureka的服务，一定时间间隔向Eureka注册中心发送心跳包，当超过时间间隔注册中心没有收到心跳包(可能是网络阻塞等)，**并不会立即删除该服务，而是触发自我保护机制，仍然保留着失效的服**务。

- Eureka服务端：eureka.server.enable-self-preservation = false

- Eureka客户端：eureka.instance.lease-renewal-interval-in-seconds=30

     			    eureka.instance.lease-expiration-duration-in-seconds=90

#### 4.1.2 Zookeeper :2181(需要客户端)

- 进入容器docker exec -it zookeeper bin/zkCli.sh

- Zookeeper工作机制

  Zookeeper从设计模式角度来理解：是一个基于**观察者模式设计**的分布式服务管理框架，它负 

  责**存储和管理大家都关心的数据**，然后**接受观察者的注册**，一旦这些数据的状态发生变化 ， 

  Zookeeper就将负责**通知已经在Zookeeper上注册的那些观察者**做出相应的反应.（就像消息推送和消息通知?）

![1621258653554](img\Zookeeper工作机制.png)

- Zookeeper特点

  1）Zookeeper：**一个领导者（Leader），多个跟随者（Follower）**组成的集群

  2）集群只要有**半数以上**(>一半，所以Zookeper适合单数台服务器，因此最少需要3台服务器)节点存活，Zookeeper集群就能正常服务

  3）全局数据一致：每个Server保存一份相同数据副本，Client无论连接到哪个Server，**数据都是一致的**

  4）更新请求顺序进行，来自同一个Client的更新请求**按其发送顺序依次执行**

  5）**数据更新原子性**，一次数据更新要么成功，要么失败

  6）**实时性**，在一定时间范围内，Client能读到最新数据

- 数据结构

  ZooKeeper数据模型的结构与**Unix文件系统很类似**，整体上可以看作是一棵树，每个节点称做一 

  个ZNode。每一个ZNode默认能够**存储1M B**的数据，每个ZNode都可以通过其路径唯一标识。 

- 应用场景: 统一命名服务/统一配置管理/统一集群管理/服务器节点动态上下线(临时-e)/软负载均衡。

  - 统一配置管理：

    分布式环境下，配置文件同步非常常见。 一般要求一个集群中，所有节点的配置信息是 

    一致的，比如 Kafka 集群。 对配置文件修改后，希望能够快速同步到各个节点上。 

  - 统一集群管理：

    分布式环境中，实时掌握每个节点的状态是必要的 。可以将分布式服务每个节点的信息写入zookeeper

  - 服务器动态上下线

    服务端在Zookeeper上注册的信息都是临时节点，当服务端挂掉后，zookeeper上的临时节点也被删除，zookeeper就会实时通知客户端服务端下线的信息

  - 软负载均衡

    Zoopeeker**会记录每台服务器的访问数**，让访问最少的服务器去处理最新请求。

  - **分布式锁**
    **因为Zookeeper的临时节点+有序编号+监听上一编号节点的特性。**

    分布式抢锁时，都会在相同路径下创建临时节点，临时节点按顺序编号，编号越前的越有机会抢到锁；

    监听锁的释放可在抢锁时判断是否还有比当前编号还靠前的编号(利用监听上一编号机制)，无则抢到锁；

    临时节点可以保证抢到锁的程序崩掉后节点被删除，即释放锁；

    也可在客户端设置超时时间，超时删除节点即释放锁

- zookeeper客户端配置文件zoo.cfg解读

  - tickTime =2000：通信心跳数，Zookeeper 服务器与客户端心跳时间，单位毫秒 
  - initLimit =10：Leader/Follower 初始通信时间限制 ，单位为tickTime ，10initLimit=10*ticktime=20s
  - syncLimit =5：LF 同步通信时限。，假如响应超过syncLimit *  tickTime，**Leader认为Follwer死掉，从服务器列表中删除Follwer**
  - dataDir：数据文件目录+数据持久化路径 
  - clientPort =2181：客户端连接端口 

- 客户端命令

  命令基本语法 

  功能描述 

  help 显示所有操作命令 

  ls path [watch] 使用 ls 命令来查看当前 znode 中所包含的内容 

  ls2 path [watch] 查看当前节点数据并能看到更新次数等数据 

  create 普通创建 

  -s 含有序列 

  -e 临时（重启或者超时消失） 

  get path [watch] 获得节点的值 

  set 设置节点的具体值 

  stat 查看节点状态 

  delete 删除节点 

  rmr 递归删除节点 

- yml配置

```yaml
spring:
  application:
    name: cloud-payment-service  # Eureka发现需要
  cloud:
    zookeeper:
      connect-string: 192.168.1.170:2181,192.168.1.171:2181,192.168.1.172:2181

// 启动类加@EnableDiscoveryClient注解，即可完成
```

- pom依赖

  ```yaml
  服务端，客户端都依赖
  <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-zookeeper-discovery -->
  <dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
  </dependency>
  ```

- 创建客户端

````java
private static String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
private static int sessionTimeout = 2000;
ZooKeeper = zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
    @Override
    public void process(WatchedEvent event) {
    // 收到事件通知后的回调函数（用户的业务逻辑）
    System.out.println(event.getType() + "--" + 
    event.getPath());
    // 再次启动监听
    try {
        zkClient.getChildren("/", true);
    } catch (Exception e) {
        e.printStackTrace();
    } }
});



@Configuration
@EnableFeignClients
@EnableDiscoveryClient
public class FeignService2 {
    @Autowired
    private TheClient theClient;

    @FeignClient(name = "SHOPCART-PRO")
    interface TheClient {

        @RequestMapping(path = "/mall4springcloud/mall-shopcart/getAll", method = RequestMethod.GET)
        @ResponseBody
        List<MallShopcart> getAll();
    }

    public List<MallShopcart> getAll() {
        return theClient.getAll();
    }
}
````

- 无自我保护机制，没有心跳直接干掉服务。因此服务节点是临时性的

- client : zkCli.sh, 可查看注册信息ls /

- Zookeeper内部原理：

  - 节点类型：两个类型都**可选择带序列号的生成规则**。在分布式系统中，顺序号可以被用于 为所有的事件进行全局排序，这样客户端可以**通过顺序号推断事件的顺序 ** 
    - 持久（Persistent）：客户端和服务器端断开连接后，创建的节点不删除 

    - 短暂（Ephemeral）：客户端和服务器端断开连接后，创建的节点自己删除 

    -  **CONTAINER**

      目前ZooKeeper不支持临时节点下挂子节点。虽然社区一直有呼声提议要增加这个功能，但其实临时节点并不适合这么做。因为它绑定了会话，而父节点更适合让persistent node来担任。不过最终社区在3.5.5做了改变，引入了一个CONTAINER类型的节点。这个类型与persitent node作用相同，只是增加了一个这样的功能：**当它包含的最后一个子节点被删除后，该container父节点会被删除**。

    

- **监听器原理**（面试）

  1）Zookeeper客户端，会**创建两个线程，一个负责网络连接通信（connet），一个负责监听（listener）。 ** 

  2）通过connect线程将注册的监听事件发送给Zookeeper。 

  3）在Zookeeper的注册监听器列表中**将注册的监听事件添加到列表中**。 

  4）Zookeeper监听到有数据或路径变化，就会将这个消息发送 给listener线程。 

  5）listener线程内部调用了**process()**方法。（**回调，也就是watch定义的方法** ） 

- 常见的监听

  1）监听节点数据的变化  get path [watch] 

  2）监听子节点增减的变化  ls path [watch]

![1621311035387](img\Zookeeper监听原理.png)

- **Zookeeper选举机制（面试）**

  1. **半数机制**：集群中**半数以上(因此最少需要3台服务器)**机器存活，集群可用。所以 Zookeeper 适合安装奇数台服务器。

  2. Zookeeper 虽然在配置文件中并没有指定 Master 和 Slave。但是，Zookeeper 工作时，是有一个节点为 Leader，其他则为 Follower，Leader 是通过**内部的选举机制临时产生**的。

  3. 选举例子：5台服务器,每台服务器启动都会投自己和别人一票，然后看总票数高的，把票实际给他，在选举结果没出来前，状态都为Looking。选举出来则为Following和Leading。在选举出来后，再启动服务器，状态不为looikng的(即Following和Leading不会再投票)，4投给自己，但票数小于3，因为改变不了结果

     （1）服务器 1 启动，发起一次选举。服务器 1 投自己一票。此时服务器 1 票数一票， 

     不够半数以上（3 票），选举无法完成，服务器 1 状态保持为 LOOKING； 

     （2）服务器 2 启动，再发起一次选举。服务器 1 和 2 分别投自己一票并交换选票信息： 

     此时服务器 1 发现服务器 2 的 ID 比自己目前投票推举的（服务器 1）大，更改选票为推举 

     服务器 2。此时服务器 1 票数 0 票，服务器 2 票数 2 票，没有半数以上结果，选举无法完成， 

     服务器 1，2 状态保持 LOOKING 

     （3）服务器 3 启动，发起一次选举。此时服务器 1 和 2 都会更改选票为服务器 3。此 

     次投票结果：服务器 1 为 0 票，服务器 2 为 0 票，服务器 3 为 3 票。此时服务器 3 的票数已 

     经超过半数，服务器 3 当选 Leader。服务器 1，2 更改状态为 FOLLOWING，服务器 3 更改 

     状态为 LEADING； 

     （4）服务器 4 启动，发起一次选举。此时服务器 1，2，3 已经不是 LOOKING 状态， 

     不会更改选票信息。交换选票信息结果：服务器 3 为 3 票，服务器 4 为 1 票。此时服务器 4 

     服从多数，更改选票信息为服务器 3，并更改状态为 FOLLOWING； 

     （5）服务器 5 启动，同 4 一样当小弟。

- Zookeeper写数据流程:client想server1写数据，如果server1不是leader，那么**serve1会把请求转发给leader**，**leader收到后广播给每个server**，每个server会**将请求加入待写入队列**，并**向leader发送成功信息**。

  当**大于半数的server写入成功**，说明该写操作可以执行，leader**向每个server发送提交信息** ，然后每个server提交队列中的请求，此时写入成功。

  ![1621311616938](img\Zookeeper写数据流程.png)

#### 4.1.5 Zookeeper模拟建行交易交易 

0.项目启动时，OpenFrameWork去加载DB中申请的交易信息，然后把当前host注册到ZK交易节点底下

1.OpenFrameWork提供API(交易号，请求报文映射类)供业务端调用

2.在OpenFrameWork底层，通过交易号去读取ZK节点，读到所有服务器信息，然后自定义负载均衡算法返回一个实际host

3.OpenFrameWork框架将请求报文映射类转换成请求报文，然后根据ZK返回的host去请求服务端

- pom

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    // 3.5.5开始支持节点类型：Container：当子节点全部失效，该节点也失效
    <version>3.5.5</version>
</dependency>
```

- 生产端

```java
package com.codeman.zk;

import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;
import java.io.IOException;
import java.util.Arrays;
import java.util.Scanner;
import java.util.stream.Stream;

// Zookeeper的生产端，负责注册信息
public class ZKServer {

    // ZK服务器地址
    // 集群可用'，'分隔"192.168.1.170:2181,192.168.1.171:2181"
    private static String connectString = "192.168.1.170:2182";
    // 会话超时时间
    private static int sessionTimeout = 200000;
    private ZooKeeper zk = null;
    // 父节点
    private String parentNode = "/serversCP04";

    // 创建到 zk 的客户端连接
    public void getConnect() throws IOException {
        zk = new ZooKeeper(connectString, sessionTimeout, new
                Watcher() {
                    @Override
                    public void process(WatchedEvent event) {
                        System.out.println("watch..." + event);
                    }
                });
    }

    // 注册服务器
    public void registServer() throws
            Exception {
        String createPar = zk.create(parentNode, "localhost".getBytes(), 
                                     ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.CONTAINER);
        System.out.println("localhost is online " + createPar);
        // 项目启动就去数据库加载所有申请过的交易
        String[] jiaoyis = {"A0671D020", "A0671D021", "A0671D022", "A0671D028"};
        String[] hosts = {"192.168.1.1", "192.168.1.2", "192.168.1.3"};
        for (String jiaoyi : jiaoyis) {
            zk.create(parentNode + "/" + jiaoyi, "jiaoyi".getBytes(), 
                      ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.CONTAINER);
            for (String host : hosts) {
                try {
                    // CreateMode.EPHEMERAL_SEQUENTIAL 临时/持久+带序号/不带序号
                    String create = zk.create(parentNode + "/" + jiaoyi + "/" + jiaoyi,
                            host.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,
                            CreateMode.EPHEMERAL_SEQUENTIAL);
                    System.out.println(host + " is online " + create);
                } catch (Exception e) {
                    System.out.println("异常");
                    System.out.println(e);
                }
            }
        }
    }

    public static void main(String[] args) throws Exception {
        // 1 获取 zk 连接
        ZKServer server = new ZKServer();
        server.getConnect();
        // 2 利用 zk 连接注册服务器信息
        server.registServer();
        Thread.sleep(Long.MAX_VALUE);
    }
}

// localhost is online /serversCP04
// 192.168.1.1 is online /serversCP04/A0671D020/A0671D0200000000000
// 192.168.1.2 is online /serversCP04/A0671D020/A0671D0200000000001
// 192.168.1.3 is online /serversCP04/A0671D020/A0671D0200000000002
// 192.168.1.1 is online /serversCP04/A0671D021/A0671D0210000000000
// 192.168.1.2 is online /serversCP04/A0671D021/A0671D0210000000001
// 192.168.1.3 is online /serversCP04/A0671D021/A0671D0210000000002
// 192.168.1.1 is online /serversCP04/A0671D022/A0671D0220000000000
// 192.168.1.2 is online /serversCP04/A0671D022/A0671D0220000000001
// 192.168.1.3 is online /serversCP04/A0671D022/A0671D0220000000002
// 192.168.1.1 is online /serversCP04/A0671D028/A0671D0280000000000
// 192.168.1.2 is online /serversCP04/A0671D028/A0671D0280000000001
// 192.168.1.3 is online /serversCP04/A0671D028/A0671D0280000000002
```

- 消费端

```java
package com.codeman.zk;

import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.Scanner;

// 消费端
public class DistributeClient {
    private static String connectString = "192.168.1.170:2182"; // 集群可用'，'分隔"192.168.1.170:2181,192.168.1.171:2181"
    private static int sessionTimeout = 200000;
    private ZooKeeper zk = null;
    private String parentNode = "/serversCP04";
    String jiaoyi;

    // 创建到 zk 的客户端连接
    public void getConnect() throws IOException {
        zk = new ZooKeeper(connectString, sessionTimeout, new
                Watcher() {
                    @Override
                    public void process(WatchedEvent event) {
// 再次启动监听
                        try {
//                            getServerList(jiaoyi);
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                });
    }

    // 获取服务器列表信息
    public void getServerList(String jiaoyi) throws Exception {
        // 1 获取服务器子节点信息，并且对父节点进行监听
        List<String> children = zk.getChildren(parentNode + "/" + jiaoyi, true);
        System.out.println("children" + children);
        // 2 存储服务器信息列表
        ArrayList<String> servers = new ArrayList<>();
        // 3 遍历所有节点，获取节点中的主机名称信息
        for (String child : children) {
            byte[] data = zk.getData(parentNode + "/" + jiaoyi + "/" + child, false, null);
            servers.add(new String(data));
        }
        // 4 打印服务器列表信息
        System.out.println("所有可用服务器：" + servers);
        // 这里可以自定义负载均衡算法
        String realHost = servers.get(new Random().nextInt(servers.size()));
        System.out.println("负载均衡算法，最终调用服务器的：" + realHost);
        // 使用apache.httpcomponent调用远程
        HttpClient httpClient = HttpClientBuilder
                            .create()
                            .setConnectionTimeToLive(1, TimeUnit.MINUTES) // 连接时间
                            .build();
        HttpPost httpPost = new HttpPost("http://"+realHost+":4396/redis/http");
        // 组装请求参数Object->xml
        List<NameValuePair> nvps = new ArrayList<NameValuePair>();
        nvps.add(new BasicNameValuePair("text", "username"));
        httpPost.setEntity(new UrlEncodedFormEntity(nvps, Consts.UTF_8));
        HttpResponse response = httpClient.execute(httpPost);
        if (response.getStatusLine().getStatusCode() == 200) {
            HttpEntity entity = response.getEntity();
            // 响应参数
            String responseContent = EntityUtils.toString(entity, "UTF-8");
            System.out.println("responseContent: " + responseContent);
            EntityUtils.consume(entity);
        }
    }

    public static void main(String[] args) throws Exception {
        // 1 获取 zk 连接
        DistributeClient client = new DistributeClient();
        client.getConnect();
        while (true) {
            // 2 获取 servers 的子节点信息，从中获取服务器信息列表
            System.out.println("想调用的交易：");
            Scanner scanner = new Scanner(System.in);
            client.jiaoyi = scanner.next();
            // 该方法可以封装成API，那么交易调用就会获取到host
            client.getServerList(client.jiaoyi);
        }
    }
}

```



#### 4.1.4 Consul: 8500(需要客户端)

- 配置和Zookeeper很像

```yaml
spring:
  application:
    name: cloud-payment-service  # Eureka发现需要
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery: 
      	# hostname: 127.0.0.1
      	service-name: ${spring.application.name}
```

### 4.2 服务注册与发现组件的异同点

- CAP理论：一个分布式系统不可能同时很好的满足一致性(C)/高可用(A)/**分区容错性(P)**

  **P是分布式不同服务，挂了互不影响，因为是独立部署**

  **A是可用性，就是集群，挂了一台还可以用**

  **C是致性，集群就会有一致性问题，一致性难以保持**

  ![1607245896866](img\CAP理论.png)

- Eureka(AP)  

- Zookeeper/Consul(CP)

### 4.3 服务调用

#### 4.3.1 Ribbon负载均衡（RestTemplate）

>  **Nginx和Ribbon做负载均衡的区别：**

**Nginx是服务端的负载均衡**，所有请求都会请求到Nginx做转发

**Ribbon是客户端的负载均衡**，做的事从注册中心获取地址的操作。

- Eureka自带Ribbon依赖
- 使用

```java
@Bean
@LoadBalanced // 注释了这个，服务间才可以发现彼此，用服务名调用，而不需要ip+port
public RestTemplate restTemplate {
    return new RestTemplate();
}
```

- 7大自带负载均衡规则

  - com.netflix.loadbalancer.RoundRobinRule 轮询
  - com.netflix.loadbalancer.RandomRule 随机
  - com.netflix.loadbalancer.RetryRule 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试
  - WeightedResponseTimeRule  对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
  - BestAvailableRule  会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
  - AvailabilityFilteringRule  先过滤掉故障实例，再选择并发较小的实例
  - ZoneAvoidanceRule 默认规则，复合判断server所在区域的性能和server的可用性选择服务器

- 使用及替换负载均衡规则（放在**客户端，服务名写服务端** ）

  ```java
  // 规则Config类
  // ***需要特别注意的是，该配置类不可放在@ComponentScan可扫描到的地方，眼下之意，不可放在主文件路径下，@SpringBootApplication包含了@ComponentScan会扫描到
  @Configuration
  public class RibbonRuleConfig {
  
      @Bean
      public IRule myRule() {
          return new RandomRule();
      }
  }
  
  // 入口指定服务及规则类，这是写在customer的启动类
  @RibbonClient(name = "CLOUD-PAYMENT-SERVICE", configuration = RibbonRuleConfig.class)
  // 多服务规则
  @RibbonClients({
          @RibbonClient(name = "SHOPCART-PRO", configuration = {LoadBalanceRuleConfig.class})
  })
  ```

  

#### 4.3.2 OpenFeign

- Feign: 是一个声明式WebService客户端，使用方法是定义一个服务接口，然后在上面写注释。Feign可以与Ribbon和Eureka组合使用以支持负载均衡

- Feign旨在使编写Java Http客户端变得更容易。不使用Feign，用Ribbon+RestTemplate通常都会针对每个微服务自动封装这些服务依赖的调用。所以Feign在此基础上进一步封装，由他帮助我们定义和实现依赖服务接口的定义，我们只需创建一个接口(映射服务接口)，并且使用注解的方式来配置他，即可完成对服务接口的绑定。

- Feign和OpenFeign的区别

  ![1607432204805](img\Feign和OpenFeign的区别.png)

#####4.3.2.1 使用

主启动@EnableFeignClients

// 服务接口映射类（映射服务类的Controller层）
@FeignClient(name = "CLOUD-PAYMENT-SERVICE")

```java
@SpringBootApplication
@EnableFeignClients
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class, args);
    }
}

// 服务接口映射类
@FeignClient(name = "CLOUD-PAYMENT-SERVICE")
public interface OrderFeignService {

    @GetMapping("/payment/get/{id}")
    CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);

    @PostMapping("/payment/create")
    CommonResult<Payment> create(@RequestBody Payment payment);

    @GetMapping("/payment/discover")
    Object discover();
}
```

#####4.3.2.2 OpenFeign超时控制

```java
java.lang.RuntimeException: com.netflix.client.ClientException: Load  balancer does not have available server for client:  CLOUD-PAYMENT-SERVICE 
```

OpenFeign默认支持Ribbon，因此用Ribbon的超时控制即可

```yaml
ribbon:
  # 指的是建立连接所用的时间，适用于网络状况正常的情况下，两端连接所用的时间
  ReadTimeout: 5000
  # 指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```

#####4.3.2.3 OpenFeign日志打印功能

- 对Feign接口的调用情况进行监控和输出

- Feign日志级别

  ![1607436697440](img\Feign日志级别.png)

- 使用

```java
// 日志配置Bean
@Configuration
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}

// yml配置开启日志的Feign客户端
logging:
  level:
    com.atguigu.springcloud.service.PaymentFeignService: debug
 
```

##### 4.3.2.4 注意事项

1. **@PathVariable形式的参数,则需要使用value=""标明对应的参数,否则会抛出IllegalStateException异常**

#### 4.3.2 OpenFeign+Ribbon+Eureka的配合

- 只有**Eureka**：

  服务调用是通过ip+port+param调用

  `restTemplate.getForObject("http://192.168.1.1:8001/mall4springcloud/mall_goods/get/1")`

- 只有**Eureka+Ribbon**

  Ribbot带了服务查找的作用，配合Eureka的服务集群，可以根据**RibbonRule算法** 调某一台服务器

  SERVICE-A 集群: 192.168.1.1:8001 |  192.168.1.2:8002   |  192.168.1.3:8003

  ``restTemplate.getForObject("http://SERVICE-B/mall4springcloud/mall_goods/get/1")``

- **OpenFeign+Ribbon+Eureka** ： 简单说就是简化上面，并且有负载均衡的能力

  把@FeignClient(name = "SHOPCART-PRO"）注解上的name作为服务名，与接口上面的  @RequestMapping(）拼接成上面的url，然后**借助Ribbon(OpenFeign集成了Ribbon)实现调用** 

  **contextId: 设置分区，FeignClient默认会以name来创建bean，那么多个相同的name创建的bean就重复了，可以采取分区的形式，就可以创建多个feign。这样可以更加灵活，比如有一半的方法想fallback，那么就设置一个分区，另一半方法不想fallback则创建一个新的分区，然后不加fallback即可。** 

  ```java
  @FeignClient(name = "SHOPCART-PRO", path = "/mall4springcloud/mall-shopcart", fallback = ShopcartFeignFallBackService.class, contextId="one" /*分区一*/)
  public interface ShopcartFeignService {
  
      @RequestMapping("/getAll")
      List<MallShopcart> getAll();
  
      @RequestMapping("/update/{uid}/{gid}")
      String updateById(@PathVariable("uid") Integer uid,@PathVariable("gid") Integer gid);
  }
  
  
  @Component
  @Slf4j
  public class ShopcartFeignFallBackService implements ShopcartFeignService {
      @Override
      public List<MallShopcart> getAll() {
          log.warn("getAll查不到数据，进入fallback");
          return null;
      }
  
      @Override
      public String updateById(Integer uid, Integer gid) {
          log.warn("updateById查不到数据，进入fallback");
          return "查不到";
      }
  }
  ```

- **fallback需要开启**

  ```yml
  feign:
      hystrix:
          enabled: true
  ```

![1621239536183](img\OpenFeign+Ribbon+Eureka的配合.png)

### 4.4 服务降级

#### 4.4.1 Hystrix

- 分布式系统面临的问题： 大型项目各个系统都有数十个依赖。一旦一个系统出现问题，会出现服务雪崩
- 服务雪崩：如果某个服务不可用了(响应时间长/异常等)，但是系统无法得知服务不可用的信息，就会一直调用，结果所有请求卡死在其他依赖服务上，把其他服务也带崩。
- Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多服务依赖不可避免会调用失败(超时/异常)等，Hystrix可以保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障（服务雪崩），以提高分布式系统的弹性
- “断路器”本身是一个开关装饰，当某个服务单元发生故障后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的/可处理的备选响应（FallBack），而不是长时间的等待或则抛出调用方无法处理的异常，这样就可以保证服务调用方的线程不会被长时间/不必要的占用，从而便面了故障在分布式系统中的蔓延，乃至雪崩

#####  4.4.1.1 用途：服务降级/服务熔断/接近实时的监控

##### 4.4.1.2 服务降级：服务器忙，不然客户等待太久而是返回一个友好提示fallback。触发场景：

- 程序运行异常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量打满也会导致服务降级

##### 4.4.1.3 服务熔断：访问达到最大量后，直接拒绝访问，然后调用服务降级进行友好提示

##### 4.4.1.4 服务限流：秒杀高并发等操作，严禁一窝蜂同时访问，而是排队进行，一秒钟N个，有序进行

##### 4.4.1.5 服务端降级@HystrixCommand+@EnableCircuitBreaker

```java
@GetMapping("/payment/discoverAfter3Min")
    @HystrixCommand(fallbackMethod = "discoverAfter3MinFallback", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })
    public Object discoverAfter3Min() {
        try {
            int a = 1/0;
            log.info("into discoverAfter3Min, serverPort=" + serverPort);
            Thread.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        List<String> services = discoveryClient.getServices();

        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PROVIDER-HYSTRIX-PAYMENT");

        return new Object[]{services, instances};
    }

    public Object discoverAfter3MinFallback() {
        return "调用失败，fallback";
    }
```

##### 4.4.1.6 客户端降级 @EnableHystrix

- 和服务端一直，只是主启动类改为@EnableHystrix以及yaml

```yaml
feign:
  hystrix:
    enabled: true #如果处理自身的容错就开启。开启方式与生产端不一样。
```

- 默认降级方法
  - 类注释 @DefaultProperties(defaultFallback = "defaultFallBackMethod")
  - 默认@HystrixCommand，不用指定fallbackMethod
  - 特殊方法自己指定fallbackMethod

- 通过实现FeignClient接口的形式，指定实现类为接口方法的fallback

  ```java
  @FeignClient(name = "CLOUD-PROVIDER-HYSTRIX-PAYMENT", fallback = PaymentFallBackServiceImpl.class)
  public interface PaymentFeignService {
  
  //    @GetMapping("/payment/get/{id}")
  //    CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
  
  //    @PostMapping("/payment/create")
  //    CommonResult<Payment> create(@RequestBody Payment payment);
  
      @GetMapping("/payment/discover")
      Object discover();
  
      @GetMapping("/payment/discoverAfter3Min")
      Object discoverAfter3Min();
  }
  
  @Component
  public class PaymentFallBackServiceImpl implements PaymentFeignService {
      @Override
      public String discover() {
          return "PaymentFallBackServiceImpl - 服务降低调用 discover";
      }
  
      @Override
      public String discoverAfter3Min() {
          return "PaymentFallBackServiceImpl - 服务降低调用 discoverAfter3Min";
      }
  }
  
  
  feign:
    hystrix:
      enabled: true #如果处理自身的容错就开启。开启方式与生产端不一样。
  ```

##### 4.4.1.7 服务熔断

- 熔断机制概述：超过了保险丝(系统设定的一些上限参数：入失败率/响应时间/请求数等)触发熔断，**当检测到该节点微服务调用响应正常后，恢复调用链路**。
- 例子：

```java
@HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),  //是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),   //请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),  //时间范围
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"), //失败率达到多少后跳闸
    })
```

- 熔断器概念图

  ![1608308356224](img\熔断器概念图.png)

- 熔断类型：

  - 熔断打开：请求不再进行调用当前服务，内部设置时钟一般为MTTR(平均故障处理时间)，当打开时长达到所设时钟则进入熔断状态
  - 熔断关闭：熔断关闭不会对服务进行熔断
  - 熔断半开：熔断状态时，过了一段时间(睡眠期，默认5s)会尝试一个请求，尝试成功解除熔断，不成功还是熔断。

- 官网断路器流程图

  ![1608309142716](img\官网断路器流程图.png)

- 断路器开启或者关闭的条件

  - 当满足一定阈值的时候（默认10秒内超过20个请求次数）
  - 当失败率达到一定(默认10秒内超过50%的请求失败)
  - 达到以上阈值，断路器将会开启
  - 当开启的时候，所有请求都不会进行转发
  - 一段时间(睡眠期)之后(默认是5秒)，这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，若事变，继续开启

- 准实时的调用监控 Hystrix Dashboard @EnableHystrixDashboard

  - 注意：新版本Hystrix需要在主启动类MainAppHystrix8001中指定监控路径

  pom

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
  </dependency>
  
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
  </dependency>
  ```

  ```java
  public ServletRegistrationBean getServlet(){
      HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
      ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
      registrationBean.setLoadOnStartup(1);
      registrationBean.addUrlMappings("/hystrix.stream");
      registrationBean.setName("HystrixMetricsStreamServlet");
      return registrationBean;
  }
  ```

  url：host:port/hystrix

### 4.5 服务网关

#### 4.5.1 Zuul1.0 不行

- Zuul1.x模型

  ![1608455324110](img\Zuul1.x模型.png)

  模型缺点：

  Servlet是一个简单的网络IO模型，当请求进入servlet container时，servekt container就会为其绑定一个线程，在并发不高的场景下，这种模型是可以适用的。而高并发下，线程数量就会上涨，而线程资源代价是昂贵的(上下文切换，内存消耗大)，严重影响请求的处理时间。在一些简单的业务场景下，不希望为每个requet分配一个线程，只需要1个或几个线程就能应对极大的并发请求，这种业务场景下servlet模型没有优势。

  而zuul1.x是基于servlet之上的一个阻塞式处理模型，即spring实现了处理所有request请求的一个servelt，并由该servelt阻塞式处理。

#### 4.5.2 Zuul2.0 研发中

#### 4.5.3 Gateway

##### 4.5.3.1 概述

- Gateway 是在Spring 生态系统之上构建的API网关服务，基于Spring5， SpringBoot2, Project Reactor等技术。旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能，如：熔断/限流/重试等。

- GateWay目标是替代Zuul。在SpringCloud2.0以上的版本中，没有对新版本的Zuul2.0以上最新最高性能版本进行集成，仍然使用的Zuul1.0非Reactor模式的老版本。而为了提高网关的性能，SpringCloud-**Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。**

- Gateway的目标：提供统一的路由方式且**基于Filter链的方式**提供了网关的基本功能，如：**安全/监控/指标/限流等。**

- Gateway源码架构（模型）

  ![1608451987988](img\Gateway源码架构.png)

  - 传统的web框架，如struts2，springMVC等都是基于servletAPI和Servelt容器基础之上进行的。
  - **在Servelt3.1之后有了异步非阻塞的支持**。而WebFlux式一个典型的非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty，Undertow以及支持Servlet3.1的容器上。**非阻塞+函数式编程(Spring5必须使用JAVA8)**
  - Spring WebFlux式Spring5.0引入的新的响应式框架，区别于SpringMVC，它不依赖ServeltAPI，他是完全异步非阻塞的，并且基于Reactor来实现响应式流规范

- 用途

  - 反向代理：类似nginx，访问的入口是gateway，可以给gateway外再加一层nginx
  - 鉴权
  - 流量控制(可以加权重，相同入境配合权重转发到不同服务上)
  - 熔断
  - 日志监控
  - ...

- 微服务架构中的网关（在服务层外层）

  ![1608452396670](img\微服务架构中的网关.png)

- 为什么选Gateway而不是Zuul

  - neflix不太靠谱，zuul2.0一直跳票,迟迟不发布
    - 虽然说Zuul2.0也是基于异步非阻塞模型上进行开发的，但是SpringCloud貌似没有整合计划，而是想用Gateway代之。而Netflix相关组件都宣布进入维护期，前景不知如何
    - Gateway是SpringCloud团队开发的，多了很多Zuul没有的功能，使用也更加简单便捷
  - Gateway具有如下特性：
    - 基于SpringFramework5，Project Reactor，SpringBoot2.0进行构建
    - **动态路由：能够匹配任何请求属性**
    - 可以对路由进行指定Predicate(断言)和Filter(过滤器)
    - **集成Hystrix的断路器功能**
    - **集成SpringCloud服务发现功能**
    - 易于编写Predicate和Filter
    - 请求限流功能
    - **支持路径重写**

  

- Gateway和zuul的区别

  - Zuul1.x是以及基于阻塞I/O的API Gateway
  - **Zuul1.x基于Servlet2.5使用阻塞架构**，它不支持任何长连接（如WebSocket），Zuul的涉及模式和Nginx比较像，而JVM本身会有第一次加载较慢的情况，使得Zuul的性能相对差
  - Zuul2.x理念更先进，像基于Netty非阻塞和支持长连接，但SpringCloud目前还没有进行整合。Zuul2.x的性能相对1.x有较大的提升。官方Gateway的RPS(每秒请求数)是Zuul的1.6倍
  - Gateway基于SpringFramework5，Project Reactor，SpringBoot2.0进行构建**，非阻塞API**
  - Gateway还支持WebSocket，并与**Spring紧密集成拥有更好的开发体验(毕竟时Spring的东西)**

- Gatway三大核心概念

  - Route(路由)：路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

  - Predicate(断言)：参考的是java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由

  - Filter(过滤)：指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。（拦截了可以做很多事情）

    ![1608462225245](img\Gateway总体图.png)

    Gateway工作过程（核心逻辑：路由转发+执行过滤器链）

    ![1608462363044](img\Gateway工作过程.png)

##### 4.5.3.2 Predicate 

- Gateway 将路由匹配作为Spring WebFlux HandlerMapping基础架构的一部分。Gateway包括许多内置的Route Predicate工厂，所有这些Predicate都与HTTP请求的不同属性匹配，多个Route Predicate工程可以进行组合

  Gateway创建Route对象时，使用RoutePredicateFactory创建Predicate对象，Predicate对象可以赋值给Route。Gateway包含了许多内置的Route Predicate Factories。

##### 4.5.3.3 Filter

- 路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用。

- 生命周期：pre（业务逻辑前）/post（业务逻辑后）

- 种类：GatewayFilter 和 GlobalFilter

- 主要接口：impiemerts   GlobalFilter ，Ordered

- 用途：全局日志记录/统一网关鉴权

- 自定义全局GlobalFilter

  ```java
  package com.codeman.springcloud.config;
  
  import lombok.extern.slf4j.Slf4j;
  import org.apache.commons.lang.StringUtils;
  import org.springframework.cloud.gateway.filter.GatewayFilterChain;
  import org.springframework.cloud.gateway.filter.GlobalFilter;
  import org.springframework.core.Ordered;
  import org.springframework.http.HttpStatus;
  import org.springframework.stereotype.Component;
  import org.springframework.web.server.ServerWebExchange;
  import reactor.core.publisher.Mono;
  
  import java.util.Date;
  
  /**
   * @author: zhanghongjie
   * @description:
   * @date: 2020/12/21 23:55
   * @version: 1.0
   */
  @Component
  @Slf4j
  public class MyLogGatewayFilter implements GlobalFilter, Ordered {
  
      @Override
      public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
          log.info("*********come in MyLogGateWayFilter: "+new Date());
          String token = exchange.getRequest().getQueryParams().getFirst("token");
          if(StringUtils.isEmpty(token)){
              log.info("*****token为Null 非法用户,(┬＿┬)");
              exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);//给人家一个回应
              return exchange.getResponse().setComplete();
          }
          return chain.filter(exchange);
      }
  
      @Override
      public int getOrder() {
          return 0;
      }
  }
  
  ```

##### 4.5.3.4 pom

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-gateway</artifactId>
  </dependency>
  
  <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka-server -->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
      <!-- gateway 和webstarter冲突 -->
      <exclusions>
          <exclusion>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  <!-- gateway 和webstarter冲突，取消webstart后，缺失mvc依赖 -->
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
  </dependency>
  ```

  ##### 4.5.3.5 yml

  这些routes会每次去读取yml配置文件，因此可以动态修改路由

  ```yaml
  server:
    port: 9527
  spring:
    application:
      name: cloud-gateway
    cloud:
      gateway:
        discovery:
          locator:
            enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
        routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
          - Path=/payment/get/**   #断言,路径相匹配的进行路由
          ## - Before=2020-03-08T10:59:34.102+08:00[Asia/Shanghai]
  
        - id: payment_routh2
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
          - Path=/payment/lb/**   #断言,路径相匹配的进行路由
  
  
  eureka:
    instance:
      hostname: cloud-gateway-service
    client:
      service-url:
        register-with-eureka: true
        fetch-registry: true
        defaultZone: http://eureka7001.com:7001/eureka
  
  ```

### 4.6 配置中心

微服务会有一堆服务，每个服务都需要有自己的配置文件，因此需要一套集中式的，动态的配置管理设施。

#### 4.6.1 SpringCloud Config

![1608736024008](img\SpringCloudConfig模型图.png)

- SpringCloud Config分为服务端和客户端。

  服务端也称分布式配置中心，他是一个独立的微服务应用(SpringBoot),用来连接配置服务器(git)并为客户端听过获取配置信息，加密/解密信息等接口。

- 用途：

  - 集中管理配置文件
  - 不同环境不同配置，**动态化的配置更新，分环境部署**比如dev/test/prod/beta/release
  - 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
  - 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
  - 将配置信息以REST接口的形式暴露(post、curl访问刷新均可....)

- yml

  ```yaml
  server:
    port: 3344
  spring:
    application:
      name: cloud-config-center
    cloud:
      config:
        server:
          git:
            uri: git@github.com:Cod4Man/microservice-config.git # git管理路径
            search-paths:
            - microservice-config # 查找文件路径
        label: master # Git 分支
  eureka:
    client:
      service-url:
        defaultZone:  http://eureka7001.com:7001/eureka
  ```

- 服务端pom

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-config-server</artifactId>
  </dependency>
  ```

- 主启动类

  **@EnableConfigServer**

- 配置读取规则

  ![1608957986658](img\SpringCloudConfig配置读取规则.png)

![1608959876710](img\SpringCloudConfig配置读取规则2.png)

- bootstap.yml

  application.yml是用户级的资源配置项，**而bootstrap.yaml是系统级的，优先级更高。**

  SpringCloud会创建一个“Bootstrap Context”, 作为Spring应用的‘Application Context’ 的父上下文。初始化的时候，‘Bootstrap Context’负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的‘Environment’。

  ‘Bootstrap’属性有高优先级，默认情况下，它们不会被本地配置覆盖。‘Bootstrap Conttext’和‘Application Context’有着不同的约定，所以新增了一个'Bootstrap.yml'文件，保证“Bootstrap Context”和‘Application Context’配置分离。

- Config客户端

  Config服务端与远程配置(git)交互，而Config客户端只与Config服务端交互。

  - pom

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    
    或者这个？
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
    </dependency>
    ```

- Config客户端-动态刷新

  就算客户端重启，也不会刷新的，要么就重启服务端；要么动态刷新。

  - pom引入actuator监控

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```

  - 修改yml，暴露监控端口

    ```yaml
    ## Spring相关配置
    spring:
      application:
        name: goods-pro
      profiles:
        active: dev
      cloud:
        config:
          fail-fast: true
          uri: http://localhost:8899,http://localhost:8898 # Config服务端地址
          label: master
    
    management:
      endpoints:
        web:
          exposure:
            include: "*"
    ```

  - @RefreshScope修饰配置文件变量类，做到监控变量。（可能是版本的问题?@RefreshScope修饰controller的话，全部注入都会失效 ）

    ```java
    @RefreshScope
    @Component
    public class ValueComponent {
        @Value("${spring.application.name}")
        private String applicationName;
    
        public String getApplicationName() {
            return applicationName;
        }
    }
    ```

  - 发送post请求刷新

  ```shell
  curl -X POST "http://localhost:3355/actuator/refresh"
  ```

### 4.7 服务总线

#### 4.7.1 bug消息总线（配合Config使用-动态刷新配置）

- SpringCloud Bus 是用来将分布式系统的节点与轻量级消息系统链接起来的框架，它整合了JAVA的事件处理机制和消息中间件的功能。

- SpringCloud Bus**目前支持的消息中间件：RabbitMQ和Kafka**

- 用途：SpringCloud Bus能管理和传播分布式系统间的消息，就像一个分布式执行器，可用于广播状态更改/事件推送等，也可以当作微服务间的通信通道。

  ![1608989250106](img\SpringCloudBus工作过程.png)

- 总线：在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便的广播一些需要让其他连接在该主题上的实例都知道的消息。

- 基本原理：ConfigClient实例都监听MQ中同一个topic(默认是SpringCloudBus)。当一个服务刷新数据的时候，它会把这个信息放入到Topic中，**这样其他监听同一Topic的服务就能得到通知，然后去更新自身的配置**。

- 设计架构：

  - 方案一：**利用消息总线触发一个客户端/bus/refresh,而刷新所有客户端的配置**

    ![1608995149065](img\SpringCloudBus设计架构1.png)

  - 方案二：**利用消息总线触发一个服务端ConfigServer的/bus/refresh端点,而刷新所有客户端的配置（更加推荐）**

    ![1608995201829](img\SpringCloudBus设计架构2.png)

    - 打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新职责
    - 破坏了微服务各节点的对等性
    - 有一定的局限性。例如，微服务在迁移时，它的网络地址常常会发生变化，此时如果想要做到自动刷新，那就会增加更多的修改

- 添加消息总线支持：

  - pom

  ```xml
  rabbitMQ
  <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-bus-amqp</artifactId>
  </dependency>
  
  Kafka
  <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-bus-kafka</artifactId>
          </dependency>
  ```

  - yml

  ```yml
  ##  服务端 start 
  management:
    endpoints:
      web:
        exposure:
          include: 'bus-refresh'
       
  ##  服务端 end
  
  ##  客户端 start 
  management:
    endpoints:
      web:
        exposure:
          include: '*'
       
  ##  客户端 end
  
  
  spring:
    rabbitmq:
      host: 192.168.1.170
      port: 5672
      username: guest
      password: guest
      
  ```

- 发送POST请求刷新配置

  - 全局刷新

  ```shell
  curl -X POST "http://localhost:3344/actuator/bus-refresh"
  ```

  - 定点刷新

  ```shell
  curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"
  ## 加上 “服务名：端口号”
  ```

- bus通知过程图

  ![1608995629898](img\bus通知过程图.png)

### 4.8 消息驱动(SpringCloudStream) 

#### 4.8.1 定义

- 屏蔽底层消息中间件的差异，降低切换版本，统一消息的编程模型

- 官方定义SCS是一个构建消息驱动微服务的框架。

  应用程序通过inputs或outputs来与SpringCloudStream中binder对象交互。通过我们配置来binding(绑定)，而SpringCloudStream的binder对象负责与消息中间件交互。因此，开发人员只需搞清楚如何与SpringCLoudStream交互就可以方便使用消息驱动的方式。

  通过使用Spring Integration来连接消息代理中间件以实现消息事件驱动。SpringCloudStream为一些供应商的消息中间件产品提供了个性化的自动化配置实现，引用了**发布-订阅/消费组/分区**的三大核心概念。

- 目前仅支持**Rabbit MQ和Kafka**

#### 4.8.2 为什么用SpringCloudStream

- 不同消息中间件架构上不同，**如Rabbit MQ有exchange，kafka有Topic和Partitions分区**
- 不同中间件的差异对我们实际项目开发造成了一定的困扰，在使用其他一种后，**后续需求需要往另一种消息中间件迁移**，这是非常麻烦的，因为它和我们的系统耦合了。SpringCloudStream则能达到解耦的效果。

#### 4.8.3 绑定器binder

通过定义绑定器作为中间层，完美的实现了**应用程序与消息中间件细节之间的隔离**。通过向应用程序暴露统一的Channel通道，使得应用程序不需要再考虑各种不同的消息中间件实现。

![1609079227585](img\SpringCloudStream处理架构.png)

#### 4.8.4 SpringCloudStream标准流程套路

![1609079362362](img\SpringCloudStream标准流程套路.png)

- Binder：很方便的连接中间件，屏蔽差异
- Channel：通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过对Channel对队列进行配置
- Source和Sink：简单的可理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接受消息就是输入

#### 4.8.5 编码API和常用注解

![1609079467370](img\SpringCloudStream编码API和常用注解.png)

#### 4.8.6 试用（RabbitMQ） 

- pom： 生产服务相同

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>

二者不可共存
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-kafka</artifactId>
</dependency>
```

- yml

```yml
# ### 服务端 start

server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: 192.168.1.170
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称（生产端用output）
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit  # 设置要绑定的消息服务的具体设置


# ### 生产端 end

# ### 服务端  start
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: 192.168.1.170
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称（服务端用input）
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit  # 设置要绑定的消息服务的具体设置
          group: codemanA  # 分组
eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址


# ### 服务端  end

```

- 生产端binding: @EnableBinding(value = Source.class)

```java
@EnableBinding(value = Source.class)
@Slf4j
public class MessageProviderImpl implements IMessageProvider {

    @Resource
    private MessageChannel output; // 消息发送管道

    @Override
    public String send() {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        log.info("*****serial: "+serial);
        return null;

    }
}
```

- 服务端binding : @EnableBinding(Sink.class)

```java
@Component
@EnableBinding(Sink.class)
public class ReceiveMessageListenerController {

    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message) {
        System.out.println("消费者2号，接受："+message.getPayload()+"\t port:"+serverPort);
    }
}
```

- 分组消费与持久化

  - 分组消费：同组竞争，不同组可以重复消费

    ![1609082910058](img\SpringCloudStream分组.png)

  - 持久化：

    当消费者系统异常(重启等)，生产者会将消息持久化(在这个分组中)。当服务分组改变时，服务正常后也无法接收到异常期间持久化的消息；而当服务分组没有改变，服务恢复正常后，可以重新接收到异常期间持久化的消息。

### 4.9 SpringCloud Sleuth + zipkin分布式请求链路追踪

- 产生原因：在微服务框架中，一个由客户端发起的请求在后端系统中会经过多个不同的服务节点调用来协同产生最后的请求结果，每一个前段请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延迟或错误都会引起整个请求最后的失败。

- 监控平台 java -jar zipkin.jar

  - 下载地址：https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/
  - 监控地址：http://localhost:9411/zipkin/

- 调用链路(类似链表，preNode ≈ parentId)

  ![1609168466292](img\SpringCloudSleuth调用链路.png)

![1609168531376](img\SpringCloudSleuth调用链路2.png)

Trace:类似于树结构的Span集合，表示一条调用链路，存在唯一标识

span:表示调用链路来源，通俗的理解span就是一次请求信息

- pom

  ```xml
  <!--包含了sleuth+zipkin-->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-zipkin</artifactId>
          </dependency>
  ```

  

- yml

  ```yml
  spring:
    application:
      name: cloud-payment-service
    zipkin:
      base-url: http://localhost:9411
    sleuth:
      sampler:
        probability: 1  # 0-1 ， 1表示性能全开
  ```

  

## 5 SpringCloud Alibaba

- 出现原因：Spring Cloud Netflix项目进入维护模式
- SpringCloud alibaba带来了什么？
  - 2018.10.31 SpringCloud Alibaba正式入驻了SpringCloud**官方孵化器**，并在Maven中央库发布了第一个版本
  - 用途：
    - **服务限流降低**：默认支持Servlet/Feign/RestTemplate/Dubbo/Rocket MQ限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降低Metrics监控。
    - **服务注册与发现**：适配SpringCloud服务注册与发现标准，默认集成了Ribbon的支持
    - **分布式配置管理**：支持分布式系统中的外部化配置，**配置更改时自动刷新**。
    - **消息驱动能力**：基于SpringCloud Stream为微服务应用构建消息驱动能力。
    - **阿里云对象存储**：阿里云提供的海量/安全/低成本/高可靠的云存储服务。支持在任何应用/任何时间/任何地点存储和访问任意类型的数据。
    - **分布式任务调度**：提供秒级/精准/高可靠/高可用的定时(基于Cron表达式)任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀的分配到所有Worker(schedulerx-client)上执行。
  - 组件：
    - **Sentinel**：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
    - **Nacos：**一个更易于构建云原生应用的**动态服务发现、配置管理和服务管理平台**。
    - **RocketMQ：**一款开源的分布式消息系统，基于高可用分布式集群技术，提供**低延时的、高可靠**的消息发布与订阅服务。
    - **Dubbo**：Apache Dubbo™ 是一款**高性能 Java RPC 框架**。
    - **Seata**：阿里巴巴开源产品，一个易于使用的**高性能微服务分布式事务解决方案**。
    - Alibaba Cloud ACM：一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。
    - Alibaba Cloud OSS: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
    - Alibaba Cloud SchedulerX: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。
    - Alibaba Cloud SMS: 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。  

### 5.1 Nacos 8848: 服务注册与配置中心 

对标：(Eureka/Zookeeper    Config+Bus)

单机的话，环境变量需要配置为单机。docker运行加上`--env MODE=standalone`

否则注册服务时启动报错 **failed to req API:/nacos/v1/ns/instance after all servers([nacos:8848]) tried**

<http://localhost:8848/nacos/index.html>   默认账号密码是nacos/nacos 

无需EnableDiscoveryClient

#### 5.1.0 docker安装/配置

- 拉取：`docker pull nacos/nacos-server`

- 方式一：修改nacos配置文件

  db.url.0

  db.url.1

  db.user

  db.password

  ```properties
  
  nacos.cmdb.dumpTaskInterval=3600
  nacos.cmdb.eventTaskInterval=10
  nacos.cmdb.labelTaskInterval=300
  nacos.cmdb.loadDataAtStart=false
  db.num=${MYSQL_DATABASE_NUM:1}
  db.url.0=jdbc:mysql://${MYSQL_SERVICE_HOST}:${MYSQL_SERVICE_PORT:3306}/nacos?${MYSQL_SERVICE_DB_PARAM:characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true}
  # db.url.1=jdbc:mysql://${MYSQL_SERVICE_HOST}:${MYSQL_SERVICE_PORT:3306}/${MYSQL_SERVICE_DB_NAME}?${MYSQL_SERVICE_DB_PARAM:characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true}
  db.user=root
  db.password=123456
  
  
  ```

  ```shell
  docker run --env MODE=standalone -itd -p 8868:8848 --name nacosallenv nacos/nacos-server
  
  # 非docker
  sh startup.sh -m standalone
  ```

- 方式二：运行容器时带上参数

  ```shell
  docker run --env MODE=standalone --env SPRING_DATASOURCE_PLATFORM=mysql --env MYSQL_MASTER_SERVICE_DB_NAME=nacos_config --env MYSQL_MASTER_SERVICE_HOST=192.168.1.4 --env MYSQL_MASTER_SERVICE_USER=root --env MYSQL_MASTER_SERVICE_PASSWORD=123456 --env MYSQL_SLAVE_SERVICE_HOST=192.168.1.4 --env MYSQL_MASTER_SERVICE_DB_NAME=nacos_config --name nacosallenv -d -p 8868:8848 nacos/nacos-server
  ```

- Nacos可选配置

  配置项 描述 可选参数 默认值     MODE 模式 cluster/standalone cluster/standalone cluster   PREFER_HOST_MODE 是否支持 hostname hostname/ip ip   NACOS_SERVER_PORT 服务端口号  8848   SPRING_DATASOURCE_PLATFORM 单机模式支持 mysql  mysql / empty empty   MYSQL_MASTER_SERVICE_HOST mysql 主节点 host     MYSQL_MASTER_SERVICE_PORT mysql 主节点 port  3306   MYSQL_MASTER_SERVICE_DB_NAME    mysql 主节点数据库名     MYSQL_MASTER_SERVICE_USER mysql 主节点用户名     MYSQL_MASTER_SERVICE_PASSWORD mysql 主节点密码     MYSQL_SLAVE_SERVICE_HOST mysql 从节点 host     MYSQL_SLAVE_SERVICE_PORT mysql 从节点 port  3306

   ![1609345213243](img\Nacos可选配置.png)

   

- Nacos数据库搭配Mysql使用:(内置derby)

  https://github.com/alibaba/nacos/blob/master/config/src/main/resources/META-INF/nacos-db.sql

- 进入Nacos： `docker exec -it nacos bash`

 

 

#### 5.1.1 简介 

- 含义：前四个字母分别为Naming和Configuration的前两个字母，最后的s为Service

- 是什么:

  - 一个更易于构建云原生应用的动态服务发现，配置管理和服务管理中心
  - Nacos：Dynamic Naming and Configuration Service
  - 注册中心+配置中心的组合。Nacos=Eureka+Config+Bus

- 用途：

  - 替代Eureka做服务注册中心(并且支持负载均衡，因为**内置Ribbon**)
  - 替代Config做服务配置中心
  - 支持AP/CP，可以切换

- 各服务中心对比：

  据说Nacos在阿里巴巴有超过10万实例运行，已经过了类似双十一等各种大型流量的考验

  ![1609341568639](img\Nacos与各服务中心对比.png)

  - Nacos全景图

![1609685702348](img\Nacos全景图.png)

​	-   Nacos和CAP

![1609685812639](img\Nacos和CAP.png)

 - nacos的CP/AP切换：`curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'`

![1609685993812](img\Nacos的CP-AP切换.png)

#### 5.1.2 使用-服务注册中心

- 依赖pom

```xml
<!--父依赖 -->
<!--spring cloud alibaba 2.1.0.RELEASE-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.1.0.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>



<!-- 子工程 -->
<!--SpringCloud ailibaba nacos -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

- 配置yml

```yml
# 服务端
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.170:8848 #配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*'
        
        
        
# 客户端
server:
  port: 83


spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.170:8848


#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider


```

- 请求方式 RestTemplate，也可整合OpenFeign做RPC调用

```java
@Configuration
public class BaseConfiguration {

    @Bean
    @LoadBalanced // 记得加上，否则会找不到服务
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

#### 5.1.3 使用-服务配置中心

- pom

 ```xml
  <!--nacos-config-->
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
  </dependency>
  <!--nacos-discovery-->
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
 ```
- yml

  ![1609777717156](img\Nacos配置规则.png)

```yml
####  bootstrap.yml  start  ####
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.170:8848 #Nacos服务注册中心地址
      config:
        server-addr: 192.168.1.170:8848 #Nacos作为配置中心地址
        file-extension: yml #指定yaml格式的配置
        # group: DEV_GROUP
        # namespace: 7d8f0f5a-6a53-4785-9686-dd460158e5d4


# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
# nacos-config-client-dev.yaml

# nacos-config-client-test.yaml   ----> config.info


####  bootstrap.yml  end  ####

####  application.yml  start  ####
spring:
  profiles:
    active: dev
    
####  application.yml  end  ####
```

- @RefreshScope(Nacos自带动态刷新，**无需发送POST请求**)

```java
@Component
@RefreshScope
public class CommonComponent {

    @Value("${config.info}")
    private String configInfo;

    public String getConfigInfo() {
        return configInfo;
    }
}

```

- nacos配置：Data-ID需和yml配置中规则一致

![1609777293564](img\Nacos发布配置.png)

     - Nacos中的dataid的组成格式与SpringBoot配置文件中的匹配规则


​      

 ![1609777524526](img\Nacos中的dataid的组成格式与SpringBoot配置文件中的匹配规则.png)

- 设置DataId公式

  - `${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}`
  - prefix默认为spring.application.name的值
  - spring.profile.active既为当前环境对应的profile,可以通过配置项spring.profile.active 来配置
  - file-exetension为配置内容的数据格式，可以通过配置项spring.cloud.nacos.config.file-extension配置

    

- 分类配置

  - Nacos结构:NameSpace>Group>Data ID(Service)

    **NameSpace主要用来实现隔离**，比如dev/pro/test

    Group用来做分组，把不同微服务分组

    Service就是微服务，可以包含多个集群Cluster

    ![1609855473548](img\Nacos结构.png)

```yml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.170:8848 #Nacos服务注册中心地址
      config:
        server-addr: 192.168.1.170:8848 #Nacos作为配置中心地址
        file-extension: yml #指定yaml格式的配置
        group: DEFAULT_GROUP  # group
        namespace: c33b9ed5-00c8-4294-b257-332126d40dd9 # namespace
        
        
spring:
  profiles:
    active: test
```

#### 5.1.4 Nacos集群和持久化配置

- 集群官网架构图![1610374262649](img\Nacos集群官网架构图.png)

Nginx+Nacos(3+)+Mysql实现集群

![1610374502767](img\Nginx_Nacos_Mysql实现高可用Nacos集群.png)

最终架构

![1610375881733](img\Nacos集群最终架构.png)

- 数据库支持

  - Nacos默认内嵌derby数据库，如果启动多个默认配置下的nacos节点，数据存储会存在一致性的问题。

  - Nacos采用集中式存储的形式来集群化部署，目前只支持Mysql。 

  - 单机版配置(/nacos/conf/application.properties)

    ![1610375208279](img\Nacos单机版Mysql配置.png)

#### 5.1.5 Nacos共享配置

```yml
# 支持多个共享 Data Id 的配置，优先级小于extension-configs,自定义 Data Id 配置 属性是个集合，内部由 Config POJO 组成。Config 有 3 个属性，分别是 dataId, group 以及 refresh
spring:
  application:
    # 服务名
    name: proj01
  cloud:
    nacos:
      config:
      	extConfig: # springcloud alibaba 2.1.0.RELEASE
          - data-id: common-config.yml
            group: BLOG0703_GROUP
            refresh: true
          - data-id: common-config2.yml
            group: BLOG0703_GROUP
            refresh: true
            
            
# 其他版本写法： 2.2.5.RELEASE
extensionConfigs[0]:
  data-id: common-config.yml
  group: BLOG0703_GROUP
  refresh: true
extensionConfigs[1]:
  data-id: common-config1.yml
  group: BLOG0703_GROUP
  refresh: true
  
# or
sharedConfigs[0]:
  data-id: common-config.yml
  group: BLOG0703_GROUP
  refresh: true
sharedConfigs[1]:
  data-id: common-config1.yml
  group: BLOG0703_GROUP
  refresh: true
```



### 5.2 SpringCloud Alibaba Sentinel实现熔断与限流：8080

使用docker，启动默认端口为8858，需要暴露。也可以进入容器内部启动/bladex/sentinel/app.jar,默认端口为8080，也需要暴露出来

#### 5.2.1 简介

- 轻量级的**流量控制/熔断降级**的Java库

- 替代Hystrix

- 用途：

  ![1610376780126](img\Sentinel用途.png)

- 组成：

  - 核心库（Java客户端）不依赖任何框架和库，能够运行在所有JRE, 同时对Dubbo和SpringCloud有比较好的支持
  - 控制台(Dashboard)基于SpringBoot开发，打包后可以直接运行。

#### 5.2.2 demo

- pom

```xml
<!--SpringCloud ailibaba nacos -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
<!--SpringCloud ailibaba sentinel -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

- yml

```yaml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.170:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: 192.168.1.170:8181 #配置Sentinel dashboard地址
        port: 8719 # #默认8719，假如被占用了会自动从8719开始依次+1扫描。直至找到未被占用的端口
#      datasource:
#        ds1:
#          nacos:
#            server-addr: 192.168.1.170:8848
#            dataId: cloudalibaba-sentinel-service
#            groupId: DEFAULT_GROUP
#            data-type: json
#            rule-type: flow

management:
  endpoints:
    web:
      exposure:
        include: '*'

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持

#feign:
#  hystrix:
#    enabled: true # Openfeign fallback

```

- sentinel有懒加载：调用了才会显示在sentinel监控列表中

#### 5.2.3 流控规则

- 简介

![1610462415449](img\Sentinel流控规则介绍.png)

![1610462761890](img\Sentinel流控规则界面.png)



#### 5.2.4 降级规则

- 介绍

  ![1610546143274](img\Sentinel降级规则介绍.png)

- 用途

  熔断降级避免级联错误。当资源被降级后，接下来的窗口期内，调用都会自动熔断（默认抛出	DegradeException）

  **Sentinel的断路器是没有半开状态的。(区别于Histrix)**



#### 5.2.5 热点key限流

- 介绍

  ![1610549651421](img\Sentinel热点Key规则介绍.png)

- java:  **@SentinelResource, 类似Histrix的@HistrixCommand, 可指定兜底方案<u>blockHandler</u>**

```java
@GetMapping("/sentinel05/{id}/{name}")
    @ResponseBody
    @SentinelResource(value = "sentinel05", blockHandler = "handle_test05")
    public String test05(@PathVariable("id") Long id, @PathVariable("name") String name) {
        log.info("into test05");

        return "test05-" + id + "-" + name ;
    }

    public String handle_test05(Long id, String name, BlockException exception) {
        return "handle test05";
    }
```

#### 5.2.6 系统规则

![1610549846247](img\Sentinel系统规则介绍.png)

#### 5.2.7 @SentinelResource 

- 指定兜底类 **@SentinelResource（blockHandlerClass） 类方法需要static**

```java
@GetMapping("/sentinel06/{id}/{name}")
@ResponseBody
@SentinelResource(value = "sentinel06", blockHandlerClass = CustomerBlockHandler.class, blockHandler = "handler01")
public String test06(@PathVariable("id") Long id, @PathVariable("name") String name) {
    log.info("into test06");

    return "test06-" + id + "-" + name ;
}

public class CustomerBlockHandler {

    public static String handler01(Long id, String name, BlockException exception) {
        return "handler01";
    }
    public static String handler02(Long id, String name, BlockException exception) {
        return "handler02";
    }
}

// 相当于兜底走CustomerBlockHandler.handler01()， 但是要static静态，才能直接引用
```

- 注解属性说明

![1610551047067](img\SentinelResource注解属性详解.png)

- 三大核心API
  - SphU定义资源
  - Tracer定义统计
  - ContextUtil定义了上下文

#### 5.2.8 服务熔断功能

- **Sentinel整合了Ribbon+OpenFeign+Fallback**

- @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler", exceptionsToIgnore = {IllegalArgumentException.class})

  - exceptionsToIgnore忽略兜底的异常
  - blockHandler 和fallback 同时配置，会走blockHandler 
  - fallback管运行异常
  - blockHandler管配置违规

- 激活sentinel对Feign熔断的支持

  ```yml
  feign:
    sentinel:
      enabled: true # 激活Sentinel对Feign的支持
  ```

- 熔断框架对比

  ![1610633143086](img\熔断框架对比Sentinel_Hystrix_resillence4j.png)

#### 5.2.9 持久化

- pom

  ```xml
  <dependency>
      <groupId>com.alibaba.csp</groupId>
      <artifactId>sentinel-datasource-nacos</artifactId>
  </dependency>
  ```

- yml

```yaml
spring:
   cloud:
    sentinel:
    datasource:
     ds1:
      nacos:
        server-addr:localhost:8848
        dataid:${spring.application.name}
        groupid:DEFAULT_GROUP
        data-type:json
            rule-type:flow

```

- 需要在nacos手动新增Sentinel配置，注意dataid和groupid需要和yml一致

![1610635046539](img\Sentinel持久化_在Nacos新增配置.png)

Nacos配置json详解

![1610635025274](img\Sentinel持久化_Nacos配置json详解.png)

### 5.3 SpringCloud Alibaba Seata处理分布式事务 8091

- 分布式事务问题：一次业务操作需要跨多个数据源或需要跨多个系统进行远程调用，就会产生分布式事务问题

  ![1610635641899](img\分布式事务问题.png) 

- seata介绍

  - Seata是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务
  - 核心：1 ID + 3 组件
    - Transaction ID （XID）:  全局唯一的事务ID
    - Transaction Coordinator(TC)： 事务协调器，维护全局事务的运行状态，负责**协调**并驱动全局事务的提交或回滚;
    - Transaction  Manager(TM) :  控制全局事务的边界，负责开启一个全局事务，并**最终发起**全局提交或全局回滚的决议;
    - Resource Manager(RM) ： 控制**分支事务，负责分支注册，状态汇报**，并接收事务协调器的指令，驱动**分支（本地）事务的提交和回滚；** 

![1610636230433](img\Seata核心处理过程.png)

- **全局@GlobalTransactional**

  ![1610636348781](img\Seata的分布式事务解决方案.png)

- demo

  - 配置

  ```properties
  ## ============ registry.conf start ============ 
  
  registry {
    # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
    type = "nacos"   #  类型修改为nacos
    loadBalance = "RandomLoadBalance"
    loadBalanceVirtualNodes = 10
  
    nacos {
      application = "seata-server"
      serverAddr = "192.168.1.170:8848"  ## nacos配置响应调整
      group = "SEATA_GROUP"
      namespace = ""
      cluster = "default"
      username = ""
      password = ""
    }
    
    ## ============ registry.conf end  ============ 
    
    ## ============ file.conf start  ==============
    
    # transaction log store, only used in seata-server
  # service 1.0后好像没有了，是自己补上的，好像是迁移到yml里面了？
  service {
    #transaction service group mapping
    #修改事务组名称为：my_test_tx_group，和客户端自定义的名称对应
    vgroupMapping.my_test_tx_group = "default" ## vgroupMapping这里写法好像变了，只能驼峰？
    #only support when registry.type=file, please don't set multiple addresses
    # default.grouplist = "127.0.0.1:8091"
    #disable seata
    disableGlobalTransaction = false
  }
  
  store {
    ## store mode: file、db、redis
    mode = "db" # 修改为db，并响应的更改db内容
    ## rsa decryption public key
    publicKey = ""
    ## file store property
    file {
      ## store location dir
      dir = "sessionStore"
      # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
      maxBranchSessionSize = 16384
      # globe session size , if exceeded throws exceptions
      maxGlobalSessionSize = 512
      # file buffer size , if exceeded allocate new buffer
      fileWriteBufferCacheSize = 16384
      # when recover batch read size
      sessionReloadReadSize = 100
      # async, sync
      flushDiskMode = async
    }
  
    ## database store property
    db {
      ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
      datasource = "druid"
      ## mysql/oracle/postgresql/h2/oceanbase etc.
      dbType = "mysql"
      driverClassName = "com.mysql.jdbc.Driver"
      ## if using mysql to store the data, recommend add rewriteBatchedStatements=true in jdbc connection param
      url = "jdbc:mysql://192.168.1.170:3307/seata?rewriteBatchedStatements=true"
      user = "root"
      password = "123456"
      minConn = 5
      maxConn = 100
      globalTable = "global_table"
      branchTable = "branch_table"
      lockTable = "lock_table"
      queryLimit = 100
      maxWait = 5000
    }
    
    ## ============ file.conf end  ==============
  ```

  - java

    - SpringBootApplication:  排除springboot默认数据源配置

      ```
      @SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
      ```

    - Seata数据源代理

      ```java
      /**
       * @auther zzyy
       * @create 2020-02-26 16:24
       * 使用Seata对数据源进行代理
       */
      @Configuration
      public class DataSourceProxyConfig {
      
          @Value("${mybatis.mapperLocations}")
          private String mapperLocations;
      
          @Bean
          @ConfigurationProperties(prefix = "spring.datasource")
          public DataSource druidDataSource(){
              return new DruidDataSource();
          }
      
          @Bean
          public DataSourceProxy dataSourceProxy(DataSource dataSource) {
              return new DataSourceProxy(dataSource);
          }
      
          @Bean
          public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
              SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
              sqlSessionFactoryBean.setDataSource(dataSourceProxy);
              sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
              sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
              return sqlSessionFactoryBean.getObject();
          }
      ```

    - 主业务类 @GlobalTransactional，加上注解，配合配置文件，就可以实现

      ```java
      @GlobalTransactional(name = "fsp-create-order",rollbackFor = Exception.class)
      ```

  - pom

  ```xml
   <!--seata-->
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
      <exclusions>
          <exclusion>
              <artifactId>seata-all</artifactId>
              <groupId>io.seata</groupId>
          </exclusion>
      </exclusions>
  </dependency>
  <dependency>
      <groupId>io.seata</groupId>
      <artifactId>seata-all</artifactId>
      <version>1.4.0</version>
  </dependency>
  ```

  - 要用外网启动（看nacos里面注册的）

    ```shell
    docker run -itd --name seata03 -p 8091:8091 -h 192.168.1.170 docker.io/seataio/seata-server
    ```

- 分布式事务的执行流程

  - TM开启分布式事务(TM向TC注册全局事务记录)
  - 换业务场景，编排数据库，服务等事务内资源（RM向TC汇报资源准备状态）
  - TM结束分布式事务，事务一阶段结束（TM通知TC提交/回滚分布式事务）
  - TC汇总事务信息，决定分布式事务是提交还是回滚(当所有微服务都反馈正常，就可提交)
  - TC通知所有RM提交/回滚资源，事务二阶段结束。

- AT模式如何做到对业务的无侵入

  - 是什么

    ![1610726979181](img\Seata的AT模式介绍.png)

  - 一阶段加载

    ![1610727042307](img\Seata的AT模式之一阶段加载.png)

  - 二阶段提交

    ![1610727084709](img\Seata的AT模式之二阶段提交.png)

  - 三阶段回滚

    ![1610727127492](img\Seata的AT模式之三阶段回滚.png)

  - Seata的AT模式执行过程

![1610727172507](img\Seata的AT模式执行过程.png)



#### 5.3.1 Seata源码

1. 服务端

```java
// 1. 拦截器拦截方法，对有GlobalTransactional注解的方法做增强
GlobalTransactionalInterceptor.invoke(final MethodInvocation methodInvocation) {
    Class<?> targetClass =
            methodInvocation.getThis() != null ? AopUtils.getTargetClass(methodInvocation.getThis()) : null;
        Method specificMethod = ClassUtils.getMostSpecificMethod(methodInvocation.getMethod(), targetClass);
        if (specificMethod != null && !specificMethod.getDeclaringClass().equals(Object.class)) {
            final Method method = BridgeMethodResolver.findBridgedMethod(specificMethod);
            final GlobalTransactional globalTransactionalAnnotation =
                getAnnotation(method, targetClass, GlobalTransactional.class);
            final GlobalLock globalLockAnnotation = getAnnotation(method, targetClass, GlobalLock.class);
            boolean localDisable = disable || (degradeCheck && degradeNum >= degradeCheckAllowTimes);
            if (!localDisable) {
                if (globalTransactionalAnnotation != null) {
                    return handleGlobalTransaction(methodInvocation, globalTransactionalAnnotation);
                } else if (globalLockAnnotation != null) {
                    return handleGlobalLock(methodInvocation, globalLockAnnotation);
                }
            }
        }
        return methodInvocation.proceed();
}
    
// 2. 执行
TransactionalTemplate.execute(TransactionalExecutor business) {
   	 beginTransaction(txInfo, tx);

        Object rs;
        try {
            // Do Your Business
            rs = business.execute();
        } catch (Throwable ex) {
            // 3. The needed business exception to rollback.
            completeTransactionAfterThrowing(txInfo, tx, ex);
            throw ex;
        }

        // 4. everything is fine, commit.
    	
        commitTransaction(tx);

        return rs;
    } finally {
        //5. clear
        resumeGlobalLockConfig(previousConfig);
        triggerAfterCompletion();
        cleanUp();
    }
}

// 3. 提交
DefaultGlobalTransaction.commit() {
    while (retry > 0) {
        try {
            status = transactionManager.commit(xid);
            break;
        } catch (Throwable ex) {
            retry--;
            if (retry == 0) {
                throw new TransactionException("Failed to report global commit", ex);
            }
        }
    }
}

// 4. 
DefaultTransactionManager.commit(String xid) throws TransactionException {
    GlobalCommitRequest globalCommit = new GlobalCommitRequest();
    // 全局事务id xid
    globalCommit.setXid(xid);
    GlobalCommitResponse response = (GlobalCommitResponse) syncCall(globalCommit);
    return response.getGlobalStatus();
}

// 5. 使用Netty（TmNettyRemotingClient）请求seata客户端，注册全局事务	
DefaultTransactionManager.syncCall(AbstractTransactionRequest request) throws TransactionException {
        try {
            return (AbstractTransactionResponse) TmNettyRemotingClient.getInstance().sendSyncRequest(request);
        } catch (TimeoutException toe) {
            throw new TmTransactionException(TransactionExceptionCode.IO, "RPC timeout", toe);
        }
    }
```



2. 客户端

## 6. 分布式事务

https://zhuanlan.zhihu.com/p/183753774

### 6.1 2PC (2阶段提交)

![1623592073994](img\分布式事务方案-2PC.png)

> **阶段一：协调者发送事务请求，等待预提交结果**

a) 协调者向所有参与者发送事务内容，询问是否可以提交事务，并等待答复。 

b) 各参与者执行事务操作，将 undo 和 redo 信息记入事务日志中（但不提交事务）。 

c) 如参与者执行成功，给协调者反馈 yes，否则反馈 no。 

>  **阶段二：协调者根据全部结果，决定发送commit/rollback通知** 

如果协调者收到了参与者的失败消息或者超时，直接给每个参与者发送回滚(rollback)消息；否则， 

发送提交(commit)消息。两种情况处理如下： 

- 情况1：当所有参与者均反馈 yes，提交事务 

  a) 协调者向所有参与者发出正式提交事务的请求（即 commit 请求）。 

  b) 参与者执行 commit 请求，并释放整个事务期间占用的资源。 

  c) 各参与者向协调者反馈 ack(应答)完成的消息。 

  d) 协调者收到所有参与者反馈的 ack 消息后，即完成事务提交。 

- 情况2：当有一个参与者反馈 no，回滚事务 

  a) 协调者向所有参与者发出回滚请求（即 rollback 请求）。 

  b) 参与者使用阶段 1 中的 undo 信息执行回滚操作，并释放整个事务期间占用的资源。 

  c) 各参与者向协调者反馈 ack 完成的消息。 

  d) 协调者收到所有参与者反馈的 ack 消息后，即完成事务。

> **2PC存在的问题** 

1**) 性能问题**：所有**参与者在事务提交阶段处于同步阻塞状态**，占用系统资源，容易导致性能瓶颈。 

2) **可靠性问题**：如果协调者存在单点故障问题，或出现故障，提供者将一直处于**锁定状态**。 

3) 数据**一致性**问题：在阶段 2 中，如果出现协调者和参与者都挂了的情况，有可能导致数据不一 

致。 

优点：**尽量**保证了数据的强一致，适合对数据强一致要求很高的关键领域。（其实也不能100%保证 

强一致）。 

缺点：实现复杂，**牺牲了可用性，对性能影响较大**，不适合高并发高性能场景。

### 6.2 3PC (3阶段提交，2阶段的改进版)

![1623592145300](img\分布式事务方案-3PC.png)

三阶段提交是在二阶段提交上的改进版本，**3PC最关键要解决的就是协调者和参与者同时挂掉的问题**，所以3PC把2PC的准备阶段(阶段一)再次一分为二，这样三阶段提交。

>  **阶段一：参与者cancommit信号**

a) 协调者向所有参与者发出包含事务内容的 canCommit 请求，询问是否可以提交事务，并等待所有参与者答复。 

b) 参与者收到 canCommit 请求后，如果认为可以执行事务操作，则反馈 yes 并进入预备状态，否则反馈 no。 

>  **阶段二：参与者resubmit信号 **

协调者根据参与者响应情况，有以下两种可能。 

- 情况1：所有参与者均反馈 yes，协调者预执行事务 

  a) 协调者向所有参与者发出 preCommit 请求，进入准备阶段。 

  b) 参与者收到 preCommit 请求后，执行事务操作，将 undo 和 redo 信息记入事务日志中（但不提交事务）。 

  c) 各参与者向协调者反馈 ack 响应或 no 响应，并等待最终指令。 

- 情况2：只要有一个参与者反馈 no，或者等待超时后协调者尚无法收到所有提供者的反馈，即中断事务 

  a) 协调者向所有参与者发出 abort 请求。 

  b) 无论收到协调者发出的 abort 请求，或者在等待协调者请求过程中出现超时，参与者均会中断事务。 

> **阶段三：通知参与者commit **

**该阶段进行真正的事务提交**，也可以分为以下两种情况。 

- 情况 1：所有参与者均反馈 ack 响应，执行真正的事务提交 

  a) 如果协调者处于工作状态，则向所有参与者发出 do Commit 请求。 

  b) 参与者收到 do Commit 请求后，会正式执行事务提交，并释放整个事务期间占用的资源。 

  c) 各参与者向协调者反馈 ack 完成的消息。 

  d) 协调者收到所有参与者反馈的 ack 消息后，即完成事务提交。 

- 情况2：只要有一个参与者反馈 no，或者等待超时后协调组尚无法收到所有提供者的反馈，即回滚事务。 - 

  a) 如果协调者处于工作状态，向所有参与者发出 rollback 请求。 

  b) 参与者使用阶段 1 中的 undo 信息执行回滚操作，并释放整个事务期间占用的资源。 

  c) 各参与者向协调组反馈 ack 完成的消息。 

  d) 协调组收到所有参与者反馈的 ack 消息后，即完成事务回滚。 

优点：相比二阶段提交，三阶段提交降低了阻塞范围，在等待超时后协调者或参与者会中断事务。避免了协调者单点问题。阶段 3 中协调者出现问题时，参与者会继续提交事务。 

缺点：**数据不一致问题**依然存在，当在参与者收到 preCommit 请求后等待 do commite 指令时，此时如果协调者请求中断事务，而协调者无法与参与者正常通信，会导致参与者继续提交事务，造成数据不一致。 

### 6.3 TCC（补偿事务）

![1623631428137](img\分布式事务方案-TCC.png)

`Try - Confirm - Cancel` 

是一种补偿性事务思想，适用的范围更广，在业务层面实现，**因此对业务的侵入性较大，每一个操作都需要实现对应的三个方法。**

TCC分为三个阶段：

1. **Try** 阶段是做业务检查(一致性)及资源预留(隔离)，此阶段仅是一个初步操作，它和后续的Confirm 一起才能真正构成一个完整的业务逻辑。
2. **Confirm** 阶段是做确认提交，Try阶段所有分支事务执行成功后开始执行 Confirm。通常情况下，采用TCC则认为 Confirm阶段是不会出错的。即：只要Try成功，Confirm一定成功。**若Confirm阶段真的出错了，需引入重试机制或人工处理。**
3. **Cancel** 阶段是在业务执行错误需要回滚的状态下执行分支事务的业务取消，预留资源释放。通常情况下，采用TCC则认为Cancel阶段也是一定成功的。**若Cancel阶段真的出错了，需引入重试机制或人工处理。**
4. TM事务管理器
   TM事务管理器可以实现为独立的服务，也可以让**全局事务发起方**充当TM的角色，TM独立出来是为了成为公用组件，是为了考虑系统结构和软件复用。

　　TM在发起全局事务时生成全局事务记录，**全局事务ID贯穿整个分布式事务调用链条**，用来记录事务上下文，追踪和记录状态，由于Confirm 和cancel失败需进行重试，因此需要实现为幂等，幂等性是指同一个操作无论请求多少次，其结果都相同。 

>  **优点：** 

**性能提升**：具体业务来实现控制资源**锁的粒度变小**，不会锁定整个资源。 

**数据最终一致性：**基于 Confifirm 和 Cancel 的**幂等性**，保证事务最终完成确认或者取消，保证**数据的一致性**。 

**可靠性**：解决了 XA 协议的协调者单点故障问题，由**主业务方发起并控制整个业务活动**，业务活动管理器也变成多点，引入集群。 

>  **缺点：**

TCC 的 Try、Confifirm 和 Cancel 操作功能要按具体业务来实现，**业务耦合度较高，提高了开发成本**。 

### 6.4 **本地消息表MQ**

基于RocketMQ实现 https://zhuanlan.zhihu.com/p/115924952

本地消息表其实就是利用了 **各系统本地的事务**来实现分布式事务。

本地消息表顾名思义就是会有一张存放本地消息的表，一般都是放在数据库中，然后在执行业务的时候 **将业务的执行和将消息放入消息表中的操作放在同一个事务中**，这样就能保证消息放入本地表中业务肯定是执行成功的。

可以看到本地消息表其实实现的是**最终一致性**，容忍了数据暂时不一致的情况。 

### 6.5 **Seata** 

### 6.6 Sagas事务模型 

### 6.6 LCN介绍

官方宣称：**LCN并不生产事务，LCN只是本地事务的协调工**。
 TX-LCN定位于一款事务协调性框架，框架其本身并不操作事务，而是基于对事务的协调从而达到事务一致性的效果。



## 7. Spring Security + JWT/OAuth2.0

Spring Security可以用来做一些用户角色权限(可访问的路径)控制

JWT/OAuth用来做身份(登录)验证（Token）

### 7.1 Spring Security 权限管理 

Spring Security本质是一个过滤器链，两大核心(也是Web安全的2核心)：认证和授权

- pom

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- Entity：需实现UserDetails

```java
package com.elltor.securityjwt2.entity;


import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import javax.persistence.*;
import java.util.Collection;
import java.util.List;

// jpa 相关都导入 javax.persistence.包下的

/**
 * 用户对象  users
 */
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Users implements UserDetails {
    /**
     * 用户ID
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long userId;

    /**
     * 用户账号
     */
    @Column(name = "user_name")
    private String userName;

    /**
     * 密码
     */
    @Column(name = "password")
    private String password;

    /**
     * 帐号状态（0正常  1停用）
     */
    @Column(name = "status")
    private String status;


    /**
     * 用户角色（多角色用逗号间隔）
     */
    @Column(name = "roles")
    private String roles;

    //实体类中使想要添加表中不存在字段，就要使用@Transient这个注解了。
    @Transient
    private List<GrantedAuthority> authorities;

    public void setAuthorities(List<GrantedAuthority> authorities) {
        this.authorities = authorities;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return this.authorities;
    }

    @Override
    public String getUsername() {
        return this.userName;
    }

    /**
     * 账户是否未过期,过期无法验证
     */
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    /**
     * 指定用户是否解锁,锁定的用户无法进行身份验证
     *
     * @return
     */
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    /**
     * 指示是否已过期的用户的凭据(密码),过期的凭据防止认证
     *
     * @return
     */
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    /**
     * 是否可用  ,禁用的用户不能身份验证
     *
     * @return
     */
    @Override
    public boolean isEnabled() {
        return true;
    }

}

```

- SecurityConfig

```java
package com.elltor.securityjwt2.config.security;

import com.elltor.securityjwt2.config.security.handler.MyAuthencationFailureHandler;
import com.elltor.securityjwt2.config.security.handler.MyAuthenticationSuccessHandler;
import com.elltor.securityjwt2.config.security.jwt.JwtAuthTokenFilter;
import com.elltor.securityjwt2.service.impl.UserDetailServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import javax.annotation.Resource;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private MyAuthenticationSuccessHandler successHandler;

    @Autowired
    private MyAuthencationFailureHandler failureHandler;

    @Autowired
    private UserDetailServiceImpl myUserDetailsService;

    @Autowired
    private JwtAuthTokenFilter jwtAuthTokenFilter;

    @Override
    // 权限校验，API访问的控制；以及可添加身份校验
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity
            // 认证失败处理类
            //.exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
            // 基于token，所以不需要session,这里设置STATELESS(无状态)是在请求是不生成session
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
            //配置权限
            .authorizeRequests()
            //对于登录login  验证码captchaImage  允许匿名访问
            .antMatchers("/login").anonymous()
            .antMatchers(
                HttpMethod.GET,
                "/*.html",
                "/**/*.html",
                "/**/*.css",
                "/**/*.js"
            ).permitAll()
            .antMatchers("/order")  //需要对外暴露的资源路径
            .hasAnyAuthority("ROLE_USER", "ROLE_ADMIN")    //user角色和 admin角色都可以访问
            .antMatchers("/system/user", "/system/role", "/system/menu")
            .hasAnyRole("ADMIN")    //admin角色可以访问
            //  除上面外的所有请求全部需要鉴权认证
            .anyRequest().authenticated().and()//authenticated()要求在执 行该请求时，必须已经登录了应用
            //  CRSF禁用，因为不使用session
            .csrf().disable();//禁用跨站csrf攻击防御，否则无法登陆成功
        //登出功能
        httpSecurity.logout().logoutUrl("/logout");
        //  添加JWT  filter, 在每次http请求前进行拦截
        httpSecurity.addFilterBefore(jwtAuthTokenFilter, UsernamePasswordAuthenticationFilter.class);
    }


    @Override
    // 加载一些用户权限
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        //调用DetailsService完成用户身份验证              设置密码加密方式
 //  userDetailsService为重写UserDetailsService的类    	 
        auth.userDetailsService(myUserDetailsService).passwordEncoder(getBCryptPasswordEncoder());
    }


    // 在通过数据库验证登录的方式中不需要配置此种密码加密方式, 因为已经在JWT配置中指定
    @Bean
    public BCryptPasswordEncoder getBCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }


}


package com.elltor.securityjwt2.service.impl;

import com.elltor.securityjwt2.entity.Users;
import com.elltor.securityjwt2.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;


@Service
public class UserDetailServiceImpl implements UserDetailsService {

    @Autowired
    private UserService userService;

    @Override
    public UserDetails loadUserByUsername(String username) throws
            UsernameNotFoundException {
        Users users = userService.selectUserByUserName(username);
        if (users == null) {
            throw new UsernameNotFoundException("登录用户：" + username + "不存在");
        }
        //将数据库的roles解析为UserDetails的权限集
        //AuthorityUtils.commaSeparatedStringToAuthorityList将逗号分隔的字符集转成权限对象列表
        users.setAuthorities(AuthorityUtils.commaSeparatedStringToAuthorityList(users.getRoles()));
        return users;
    }
}





```

- 注解权限控制

1. **@Secured**  ：判断是否具有角色，另外需要注意的是这里匹配的字符串需要添加前缀“ROLE_“。 

   使用注解先要开启注解功能！ @EnableGlobalMethodSecurity(securedEnabled=true) 

2. **@PreAuthorize ：**

   先开启注解功能： 

   @EnableGlobalMethodSecurity(prePostEnabled = true) 

   @PreAuthorize：注解适合进入方法前的权限验证， @PreAuthorize 可以将登录用 

   户的 roles/permissions 参数传到方法中。

3. **@PostAuthorize** 

   先开启注解功能：  

   @EnableGlobalMethodSecurity(prePostEnabled = true) 

   @PostAuthorize 注解使用并不多，在方法执行后再进行权限验证，适合验证带有返回值的权限

4. **@PostFilter** 

   @PostFilter ：权限验证之后对数据进行过滤 留下用户名是 admin1 的数据表达式中的 filterObject 引用的是方法返回值 List 中的某一个元素

5. **@PreFilter** 

   @PreFilter: 进入控制器之前对数据进行过滤

### 7.2 JWT：Json Web Token 身份认证

> Why JWT

现在，前后端分离和 RESTful API 越来越火热，当后台渐渐开始只负责为客户端提供 API 接口之后，身份校验和接口安全成了难题。在传统的开发模式下，使用 cookie-session 可以保证接口安全，在没有登录的情况下访问关键数据会跳转到登录界面或者请求失败。而使用 REStful API 之后，cookie-session 存在以下 3 个问题：

- 客户端除了浏览器，可能还包括手机端 APP，对于手机端而言，管理 cookie 是一件麻烦的事情。
- RESTful 风格的 API 不建议使用 cookie。
- cookie 本身有一个缺陷，不能跨域。

正是存在上面的几个缺陷，现在 API 开始使用 JWT 代替 cookie-session 来做身份验证。

> What is JWT

JWT 全称 **JSON Web Token**。本质上 JWT 是一串 token 字符串。客户端登录之后，服务端返回一串 token 给客户端，之后每次客户端请求 API 接口都需要携带该 token 进行身份校验。JWT 由三个部分组成：**头部(header)、载荷(payload)、签名(signature)**。这三个部分使用 `.` 连接在一起就是一个完整的 JWT。所以，一个完整的 JWT 应该类似下面这种形式： `xxxxxx.yyyyy.zzzzz`

- **header**: header 是一个 json 数据，用于描述 JWT 的基本信息。一般要由两个部分组成：

  - alg：代表的是**加密所使用的算法**
  - typ：表示该 token 是什么类型的。这里 typ 自然是 JWT

  一个完整的 header 信息。最后使用 Base64 对 header 进行编码，得到 JWT 的第一部分 

  ```txt
  {
      'alg':'HS256',
      'typ':'JWT'
  }
  ```

- **payload**：

   payload 是 JWT 存储信息的部分。payload 也是一个 json 数据，每一个 json 的 key-value 称为一个声明。

  payload 有两种类型的声明：标准声明和自定义声明。

  标准声明一共有 6 个，其名称和对应含义如下：

  -  `iss` : JWT 的签发者。
  -  `iat` : JWT 的签发时间，是一个 unix 时间戳。
  -  `exp` : JWT 的过期时间，是一个 unxi 时间戳。
  -  `aud` : 接受 JWT 的一方。
  -  `sub` : JWT 所面向的用户。
  -  `jti` : 唯一标识一个 JWT。

  自定义声明为用户自己定义的 key-value，可以用来存储一些简单的基本信息。考虑到性能，不应该在 payload 中定义太多自定义声明。

  定义一个 payload ：

  ```txt
  {
      "iss": "jaychen",
      "iat": 1441593502,
      "exp": 1441594722,
      "aud": "jaychen.cc",
      "sub": "chenjiayaooo@gmail.com",
      "jti:" "xxxxxxxx",
  
      "user_id": "1",
      "username": "jaychen"
  }
  ```

  上面这个 payload 中，`id` 和 `username` 为自定义声明。

  有了 payload 只有，将该 payload 进行 Base64 加密，得到一串字符串之后，用 `.` 把 header 和 payload 连接起来。

- **signature: 签名**

 将 header 和 payload 两个部分连接起来之后，得到的字符串类似下面 `xxxx.yyyy`

 接着，使用 `header.alg` 定义的加密算法对 `hader.payload` 的字符串进行加密，并且加密的时候应该有一个密钥。加密之后，得到一串加密字符串，最后把这串加密字符串也是用 `.` 拼接在 `header.payload` 后面，形成完整的 JWT。

**这里签名的目的是为了保证 payload 数据的完整性**。如果 JWT 在传输过程中被第三方劫持，中间人对  `header.payload` 进行修改，并且使用自己的密钥重新签名。服务端收到中间人修改过的 JWT，使用自己的密钥对 `header.payload` 进行再次加密，由于中间人和服务端使用的是不同的密钥签名，所以服务端再次加密的结果肯定和中间人加密的结果不一致，由此可以断定该 JWT 被恶意篡改。

>  基于 JWT 的身份验证的流程

现在已经明白了 JWT 的生成过程，现在来梳理下 JWT 的使用流程。

- 首次登陆系统，向服务端发送 username&&password 进行登录。
- 服务端验证 username&&password,验证合法为客户端生成一串 JWT，这里在 payload 中可以自定义声明 user_id,username 等字段用来保存信息。
- 客户端收到服务端的 JWT 字符串，自行保存。**后续需要请求 API 都要携带该 JWT 到服务端进行身份校验**。
- 服务端收到客户端的 API 请求，先获取 JWT 信息，通过签名判断 JWT 的合法性，如果合法，返回数据。

> 配合Spring Security使用

- pom

```xml
<!--jwt-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>${jjwt.version}</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>${jjwt.version}</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>${jjwt.version}</version>
</dependency>
```

- java

```java
@EnableWebSecurity
// Security配置类
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        // 这里添加jwt过滤器，用来拦截所有请求，然后进行token校验
    	httpSecurity.addFilterBefore(jwtAuthTokenFilter, 	
                                     UsernamePasswordAuthenticationFilter.class);
    }
}

@Component
// JWTFilter过滤器拦截请求，OncePerRequestFilter，每次拦截并只有一次
public class JwtAuthTokenFilter extends OncePerRequestFilter {
    @Autowired
    private JwtTokenUtils jwtTokenUtils;

    @Autowired
    private UserDetailsService userDetailServiceImpl;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //http  请求头中的token,前后端沟通，可读性差一点
        String token = request.getHeader("AuthorizationXSADASDASD");
        if (token!=null && token.length()>0) {
            // 通过token解析payout获取数据username
            String username = jwtTokenUtils.getUsernameFromToken(token);
            if (username != null && SecurityContextHolder.getContext().getAuthentication()==null) {
                UserDetails userDetails = userDetailServiceImpl.loadUserByUsername(username);
                if (jwtTokenUtils.validateToken(token, userDetails)) {
                    //给使用该JWT令牌的用户进行授权
                    UsernamePasswordAuthenticationToken authenticationToken =
                            new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                    //设置用户身份授权
                    SecurityContextHolder.getContext().setAuthentication(authenticationToken);
                }
            }
        }
        filterChain.doFilter(request, response);
    }
}

@Data
@Component
public class JwtTokenUtils {

    @Value("${token.secret}")
    private String secret;

    @Value("${token.expireTime}")
    private Long expiration;

    @Value("${token.header}")
    private String header;


    private static Key KEY = null;

    /**
     * 生成token令牌
     *
     * @param userDetails 用户
     * @return 令token牌
     */
    public String generateToken(UserDetails userDetails) {
        System.out.println("[JwtTokenUtils] generateToken "+userDetails.toString());
        Map<String, Object> claims = new HashMap<>(2);
        claims.put("sub", userDetails.getUsername());
        claims.put("created", new Date());

        return generateToken(claims);
    }


    /**
     * 从令牌中获取用户名
     *
     * @param token 令牌
     * @return 用户名
     */
    public String getUsernameFromToken(String token) {
        String username = null;
        try {
            Claims claims = getClaimsFromToken(token);
            username = claims.get("sub",String.class);
            System.out.println("从令牌中获取用户名:" + username);
        } catch (Exception e) {
            username = null;
        }
        return username;
    }

    /**
     * 判断令牌是否过期
     *
     * @param token 令牌
     * @return 是否过期
     */
    public Boolean isTokenExpired(String token) {
        try {
            Claims claims = getClaimsFromToken(token);
            Date expiration = claims.getExpiration();
            return expiration.before(new Date());
        } catch (Exception e) {
            return false;
        }
    }

    /**
     * 刷新令牌
     *
     * @param token 原令牌
     * @return 新令牌
     */
    public String refreshToken(String token) {
        String refreshedToken;
        try {
            Claims claims = getClaimsFromToken(token);
            claims.put("created", new Date());


            refreshedToken = generateToken(claims);
        } catch (Exception e) {
            refreshedToken = null;
        }
        return refreshedToken;
    }

    /**
     * 验证令牌
     *
     * @param token       令牌
     * @param userDetails 用户
     * @return 是否有效
     */
    public Boolean validateToken(String token, UserDetails userDetails) {

        String username = getUsernameFromToken(token);
        return (username.equals(userDetails.getUsername()) &&
                !isTokenExpired(token));
    }


    /**
     * 从claims生成令牌,如果看不懂就看谁调用它
     *
     * @param claims 数据声明
     * @return 令牌
     */
    private String generateToken(Map<String, Object> claims) {
        Date expirationDate = new Date(System.currentTimeMillis() + expiration);
        return Jwts.builder().setClaims(claims)
                .setExpiration(expirationDate)
                .signWith(SignatureAlgorithm.HS256, getKeyInstance())
                .compact();
    }

    /**
     * 从令牌中获取数据声明,如果看不懂就看谁调用它
     *
     * @param token 令牌
     * @return 数据声明
     */
    private Claims getClaimsFromToken(String token) {
        Claims claims = null;

        try {
            claims = Jwts.parser().setSigningKey(getKeyInstance()).parseClaimsJws(token).getBody();
//            claims = Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
        } catch (Exception e) {
            claims = null;
        }
        return claims;
    }


    private Key getKeyInstance() {
        if (KEY == null) {
            synchronized (JwtTokenUtils.class) {
                if (KEY == null) {// 双重锁
                    byte[] apiKeySecretBytes = DatatypeConverter.parseBase64Binary(secret);
                    KEY = new SecretKeySpec(apiKeySecretBytes, SignatureAlgorithm.HS256.getJcaName());
                }
            }
        }
        return KEY;
    }
}

```

> **JWT注销**

有以下几个方法可以做到失效 JWT token：

1. 将 token 存入 DB（如 Redis）中，失效则删除；但增加了一个每次校验时候都要先从 DB 中查询 token 是否存在的步骤，而且违背了 JWT 的无状态原则（这不就和 session 一样了么？）。
2. 维护一个 token 黑名单，失效则加入黑名单中。
3. 在 JWT 中增加一个版本号字段，失效则改变该版本号。
4. 在服务端设置加密的 key 时，为每个用户生成唯一的 key，失效则改变该 key。

最后决定使用Redis建立一个白名单，大体思路如下：

- 1、生成Jwt的时候，将Jwt Token存入redis中
- 2、扩展Jwt的验证功能，验证redis中是否存在数据，如果存在则token有效，否则无效
- 3、实现removeAccessToken功能，在该方法内删除redis对应的key。

 

### 7.3 OAuth2.0 身份认证

> API

- `/oauth/authorize`：授权端点，ABA返回code
- `/oauth/token`：令牌端点，请求token
- `/oauth/confirm_access`：用户确认授权提交端点
- `/oauth/error`：授权服务错误信息端点
- `/oauth/check_token`：用于资源服务访问的令牌解析端点
- `/oauth/token_key`：提供公有密匙的端点，如果你使用 JWT 令牌的话

#### 7.3.1 pom

```xml
<!-- OAuth2 认证服务器-->
<dependency>
    <groupId>org.springframework.security.oauth.boot</groupId>
    <artifactId>spring-security-oauth2-autoconfigure</artifactId>
</dependency>

<!-- 来自-->
<!-- <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-openfeign-dependencies</artifactId>-->

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-jose</artifactId>
</dependency>
<!-- 来自：
org.springframework.cloud:spring-cloud-dependencies-->
```



#### 7.3.2 yml

```yaml
server:
  port: 8000

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/blog0703?serverTimezone=UTC&characterEncoding=utf-8&useSSL=false
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: 
    type: org.springframework.jdbc.datasource.DriverManagerDataSource
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200                                  # 等待连接获取的最大超时时间
      #mysql数据库验证
      test-while-idle: false
      validation-query: SELECT 1
      validation-query-timeout: 10
  redis:
    database: 0
    host: localhost
    port: 6379
  cloud:
    nacos:
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yml #指定yaml格式的配置
        group: BLOG0703_GROUP  # group
        namespace: 9405665c-0bfa-4760-9411-4c15310c0e4a # namespace
     discovery:
       server-addr: localhost:8848 #Nacos作为配置中心地址
       file-extension: yml #指定yaml格式的配置
       group: BLOG0703_GROUP  # group
       namespace: 9405665c-0bfa-4760-9411-4c15310c0e4a # namespace 
# 日志相关
logging:
  #file: logs/blog0703.log # 默认10M切分
  #path: E:\logs\blog0703
  level:
    io:
      seata: info
    com:
      codeman:
        blog0703: debug
#          mapper:
#            MallUsersMapper: debug

# jwt 配置
jwt:
  config:
    enabled: true
    key-location: jwt.jks
    key-alias: jwt
    key-pass: 123456
    iss: youlai.tech
    sub: all
    access-exp-days: 30

feign:
  httpclient:
    enabled: false
  okhttp:
    enabled: true

#weapp:
#  appid: wx99a151dc43d2637b
#  secret: a09605af8ad29ca5d18ff31c19828f37

# 全局参数设置
ribbon:
  ReadTimeout: 120000
  ConnectTimeout: 10000
  SocketTimeout: 10000
  MaxAutoRetries: 0
  MaxAutoRetriesNextServer: 1
```

#### 7.3.3. SQL

```sql
CREATE TABLE `oauth_client_details`  (
                                         `client_id` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                         `resource_ids` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                         `client_secret` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                         `scope` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                         `authorized_grant_types` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                         `web_server_redirect_uri` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                         `authorities` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                         `access_token_validity` int(11) NULL DEFAULT NULL,
                                         `refresh_token_validity` int(11) NULL DEFAULT NULL,
                                         `additional_information` varchar(4096) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                         `autoapprove` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                         PRIMARY KEY (`client_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

INSERT INTO `oauth_client_details` VALUES ('client', NULL, '123456', 'all', 'password,refresh_token', '', NULL, NULL, NULL, NULL, NULL);


CREATE TABLE `sys_oauth_client` (
                                    `client_id` varchar(256) NOT NULL,
                                    `resource_ids` varchar(256) DEFAULT NULL,
                                    `client_secret` varchar(256) DEFAULT NULL,
                                    `scope` varchar(256) DEFAULT NULL,
                                    `authorized_grant_types` varchar(256) DEFAULT NULL,
                                    `web_server_redirect_uri` varchar(256) DEFAULT NULL,
                                    `authorities` varchar(256) DEFAULT NULL,
                                    `access_token_validity` int(11) DEFAULT NULL,
                                    `refresh_token_validity` int(11) DEFAULT NULL,
                                    `additional_information` varchar(4096) DEFAULT NULL,
                                    `autoapprove` varchar(256) DEFAULT NULL,
                                    PRIMARY KEY (`client_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

INSERT INTO `sys_oauth_client` VALUES ('client', NULL, '123456', 'all', 'authorization_code,password,refresh_token,implicit', NULL, NULL, 3600, 7200, NULL, 'true');
INSERT INTO `sys_oauth_client` VALUES ('youlai-admin', '', '123456', 'all', 'password,client_credentials,refresh_token,authorization_code', '', '', 3600, 7200, NULL, 'true');
INSERT INTO `sys_oauth_client` VALUES ('youlai-weapp', '', '123456', 'all', 'authorization_code,password,refresh_token,implicit', NULL, NULL, 3600, 7200, NULL, 'true');

```

#### 7.3.4 JAVA

> 1. ClientDetailsService

```java
import com.codeman.blog0703.api.OAuthClientFeignClient;
import com.codeman.blog0703.entity.SysOauthClient;
import com.codeman.blog0703.enums.PasswordEncoderTypeEnum;
import com.codeman.blog0703.vo.result.Result;
import lombok.SneakyThrows;
import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.security.oauth2.provider.ClientDetails;
import org.springframework.security.oauth2.provider.ClientDetailsService;
import org.springframework.security.oauth2.provider.NoSuchClientException;
import org.springframework.security.oauth2.provider.client.BaseClientDetails;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
public class ClientDetailsServiceImpl implements ClientDetailsService {

    @Resource
    private OAuthClientFeignClient oAuthClientFeignClient;

    @Override
    @SneakyThrows
    public ClientDetails loadClientByClientId(String clientId) {
        try {
            Result<SysOauthClient> result = oAuthClientFeignClient.getOAuthClientById(clientId);
            if (Result.success().getCode().equals(result.getCode())) {
                SysOauthClient client = result.getData();
                BaseClientDetails clientDetails = new BaseClientDetails(
                        client.getClientId(),
                        client.getResourceIds(),
                        client.getScope(),
                        client.getAuthorizedGrantTypes(),
                        client.getAuthorities(),
                        client.getWebServerRedirectUri());
                clientDetails.setClientSecret(PasswordEncoderTypeEnum.NOOP.getPrefix() + client.getClientSecret());
                return clientDetails;
            } else {
                throw new NoSuchClientException("No client with requested id: " + clientId);
            }
        } catch (EmptyResultDataAccessException var4) {
            throw new NoSuchClientException("No client with requested id: " + clientId);
        }
    }
}
```

> 2.  UserDetailsService

```java
import com.codeman.blog0703.api.UserFeignClient;
import com.codeman.blog0703.domain.OAuthUserDetails;
import com.codeman.blog0703.entity.User;
import com.codeman.blog0703.enums.OAuthClientEnum;
import com.codeman.blog0703.util.JwtUtils;
import com.codeman.blog0703.vo.result.Result;
import com.codeman.blog0703.vo.result.ResultCode;
import lombok.AllArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.AccountExpiredException;
import org.springframework.security.authentication.DisabledException;
import org.springframework.security.authentication.LockedException;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
@AllArgsConstructor
@Slf4j
public class UserDetailsServiceImpl implements UserDetailsService {

    private UserFeignClient userFeignClient;
    // private MemberFeignClient memberFeignClient;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        String clientId = JwtUtils.getAuthClientId();
        OAuthClientEnum client = OAuthClientEnum.getByClientId(clientId);

        Result result;
        OAuthUserDetails oauthUserDetails = null;
        switch (client) {
            default:
                result = userFeignClient.user(username);
                if (ResultCode.SUCCESS.getCode().equals(result.getCode())) {
                    User sysUser = (User)result.getData();
                    oauthUserDetails = new OAuthUserDetails(sysUser);
                }
                break;
        }
        if (oauthUserDetails == null || oauthUserDetails.getId() == null) {
            throw new UsernameNotFoundException(ResultCode.USER_NOT_EXIST.getMsg());
        } else if (!oauthUserDetails.isEnabled()) {
            throw new DisabledException("该账户已被禁用!");
        } else if (!oauthUserDetails.isAccountNonLocked()) {
            throw new LockedException("该账号已被锁定!");
        } else if (!oauthUserDetails.isAccountNonExpired()) {
            throw new AccountExpiredException("该账号已过期!");
        }
        return oauthUserDetails;
    }

}
```

> 3. oauth的认证授权配置，token相关 AuthorizationServerConfigurerAdapter

```java
package com.codeman.blog0703.security.config;

import cn.hutool.core.collection.CollectionUtil;
import cn.hutool.http.HttpStatus;
import cn.hutool.json.JSONUtil;
import com.codeman.blog0703.domain.OAuthUserDetails;
import com.codeman.blog0703.security.service.ClientDetailsServiceImpl;
import com.codeman.blog0703.security.service.UserDetailsServiceImpl;
import com.codeman.blog0703.vo.result.Result;
import com.codeman.blog0703.vo.result.ResultCode;
import lombok.AllArgsConstructor;
import lombok.SneakyThrows;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.common.DefaultOAuth2AccessToken;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.provider.token.TokenEnhancer;
import org.springframework.security.oauth2.provider.token.TokenEnhancerChain;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.KeyStoreKeyFactory;
import org.springframework.security.web.AuthenticationEntryPoint;

import java.security.KeyPair;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

/**
 * 认证授权配置
 */
@Configuration
@EnableAuthorizationServer
@AllArgsConstructor
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    private AuthenticationManager authenticationManager;
    private UserDetailsServiceImpl userDetailsService;
    private ClientDetailsServiceImpl clientDetailsService;

    /**
     * OAuth2客户端【数据库加载】
     */
    @Override
    @SneakyThrows
    public void configure(ClientDetailsServiceConfigurer clients) {
        clients.withClientDetails(clientDetailsService);
    }

    /**
     * 配置授权（authorization）以及令牌（token）的访问端点和令牌服务(token services)
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        List<TokenEnhancer> tokenEnhancers = new ArrayList<>();
        tokenEnhancers.add(tokenEnhancer());
        tokenEnhancers.add(jwtAccessTokenConverter());
        tokenEnhancerChain.setTokenEnhancers(tokenEnhancers);
        endpoints
                .authenticationManager(authenticationManager)
                .accessTokenConverter(jwtAccessTokenConverter())
                .tokenEnhancer(tokenEnhancerChain)
                .userDetailsService(userDetailsService)
                // refresh token有两种使用方式：重复使用(true)、非重复使用(false)，默认为true
                //      1 重复使用：access token过期刷新时， refresh token过期时间未改变，仍以初次生成的时间为准
                //      2 非重复使用：access token过期刷新时， refresh token过期时间延续，在refresh token有效期内刷新便永不失效达到无需再次登录的目的
                .reuseRefreshTokens(true);
    }


    /**
     * 使用非对称加密算法对token签名
     */
    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setKeyPair(keyPair());
        return converter;
    }

    /**
     * 从classpath下的密钥库中获取密钥对(公钥+私钥)
     */
    @Bean
    public KeyPair keyPair() {
        KeyStoreKeyFactory factory = new KeyStoreKeyFactory(new ClassPathResource("jwt.jks"), "123456".toCharArray());
        KeyPair keyPair = factory.getKeyPair("jwt", "123456".toCharArray());
        return keyPair;
    }

    /**
     * JWT内容增强
     */
    @Bean
    public TokenEnhancer tokenEnhancer() {
        return (accessToken, authentication) -> {
            Map<String, Object> additionalInfo = CollectionUtil.newHashMap();
            OAuthUserDetails OAuthUserDetails = (OAuthUserDetails) authentication.getUserAuthentication().getPrincipal();
            additionalInfo.put("userId", OAuthUserDetails.getId());
            additionalInfo.put("username", OAuthUserDetails.getUsername());
            ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInfo);
            return accessToken;
        };
    }

    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setHideUserNotFoundExceptions(false); // 用户不存在异常抛出
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    /**
     * 密码编码器
     * <p>
     * 委托方式，根据密码的前缀选择对应的encoder，例如：{bcypt}前缀->标识BCYPT算法加密；{noop}->标识不使用任何加密即明文的方式
     * 密码判读 DaoAuthenticationProvider#additionalAuthenticationChecks
     *
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    /**
     * 自定义认证异常响应数据
     */
    @Bean
    public AuthenticationEntryPoint authenticationEntryPoint() {
        return (request, response, e) -> {
            response.setStatus(HttpStatus.HTTP_OK);
            response.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
            response.setHeader("Access-Control-Allow-Origin", "*");
            response.setHeader("Cache-Control", "no-cache");
            Result result = Result.failed(ResultCode.CLIENT_AUTHENTICATION_FAILED);
            response.getWriter().print(JSONUtil.toJsonStr(result));
            response.getWriter().flush();
        };
    }
}

```

> 4.  Spring Security 访问路由限制，如果只有oauth应该不用吧

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
@Slf4j
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests().antMatchers("/oauth/**").permitAll()
                 // @link https://gitee.com/xiaoym/knife4j/issues/I1Q5X6 (接口文档knife4j需要放行的规则)
                .antMatchers("/webjars/**","/doc.html","/swagger-resources/**","/v2/api-docs").permitAll()
                .anyRequest().authenticated()
                .and()
                .csrf().disable();
    }

    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```

> 4. UserDetails

```java
import cn.hutool.core.collection.CollectionUtil;
import com.codeman.blog0703.entity.User;
import com.codeman.blog0703.enums.PasswordEncoderTypeEnum;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.ArrayList;
import java.util.Collection;

import static com.codeman.blog0703.constants.GlobalConstants.STATUS_YES;


/**
 * 登录用户信息
 */
@Data
@NoArgsConstructor
public class OAuthUserDetails implements UserDetails {

    private Long id;

    private String username;

    private String password;

    private Boolean enabled;

    private String clientId;

    private Collection<SimpleGrantedAuthority> authorities;

    public OAuthUserDetails(User user) {
        this.setId(Long.valueOf(user.getUserId()));
        this.setUsername(user.getName());
        this.setPassword(PasswordEncoderTypeEnum.BCRYPT.getPrefix() + user.getPassword());
        this.setEnabled(user.getUseabled());
        if (CollectionUtil.isNotEmpty(user.getRoles())) {
            authorities = new ArrayList<>();
            user.getRoles().forEach(role -> authorities.add(new SimpleGrantedAuthority(role.getRoleCd())));
        }
    }

    /*public OAuthUserDetails(AuthMemberDTO member) {
        this.setId(member.getId());
        this.setUsername(member.getUsername());
        this.setPassword(PasswordEncoderTypeEnum.BCRYPT.getPrefix() + member.getPassword());
        this.setEnabled(STATUS_YES.equals(member.getStatus()));
    }*/


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return this.authorities;
    }

    @Override
    public String getPassword() {
        return this.password;
    }

    @Override
    public String getUsername() {
        return this.username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return this.enabled;
    }
}
```

> 5. AuthController

```java
import cn.hutool.json.JSONObject;
import com.codeman.blog0703.constants.AuthConstants;
import com.codeman.blog0703.enums.OAuthClientEnum;
import com.codeman.blog0703.util.JwtUtils;
import com.codeman.blog0703.vo.result.Result;
import com.nimbusds.jose.jwk.JWKSet;
import com.nimbusds.jose.jwk.RSAKey;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import lombok.AllArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.security.oauth2.provider.endpoint.TokenEndpoint;
import org.springframework.web.HttpRequestMethodNotSupportedException;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import springfox.documentation.annotations.ApiIgnore;

import java.security.KeyPair;
import java.security.Principal;
import java.security.interfaces.RSAPublicKey;
import java.util.Map;
import java.util.concurrent.TimeUnit;

@Api(tags = "认证中心")
@RestController
@RequestMapping("/oauth")
@AllArgsConstructor
@Slf4j
public class OAuthController {

    private TokenEndpoint tokenEndpoint;
    // private IAuthService wechatAuthService;
    private RedisTemplate redisTemplate;
    private KeyPair keyPair;

    @ApiOperation(value = "OAuth2认证", notes = "login")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "grant_type", defaultValue = "password", value = "授权模式", required = true),
            @ApiImplicitParam(name = "client_id", value = "Oauth2客户端ID（新版本需放置请求头）", required = true),
            @ApiImplicitParam(name = "client_secret", value = "Oauth2客户端秘钥（新版本需放置请求头）", required = true),
            @ApiImplicitParam(name = "refresh_token", value = "刷新token"),
            @ApiImplicitParam(name = "username", defaultValue = "admin", value = "登录用户名"),
            @ApiImplicitParam(name = "password", defaultValue = "123456", value = "登录密码")
    })
    @PostMapping("/token")
    public Object postAccessToken(
            @ApiIgnore Principal principal,
            @ApiIgnore @RequestParam Map<String, String> parameters
    ) throws HttpRequestMethodNotSupportedException {

        /**
         * 获取登录认证的客户端ID
         *
         * 兼容两种方式获取Oauth2客户端信息（client_id、client_secret）
         * 方式一：client_id、client_secret放在请求路径中(注：当前版本已废弃)
         * 方式二：放在请求头（Request Headers）中的Authorization字段，且经过加密，例如 Basic Y2xpZW50OnNlY3JldA== 明文等于 client:secret
         */
        String clientId = JwtUtils.getAuthClientId();
        OAuthClientEnum client = OAuthClientEnum.getByClientId(clientId);
        switch (client) {
            case TEST: // knife4j接口测试文档使用 client_id/client_secret : client/123456
                return tokenEndpoint.postAccessToken(principal, parameters).getBody();
            default:
                return Result.success(tokenEndpoint.postAccessToken(principal, parameters).getBody());
        }
    }

    /*@ApiOperation(value = "微信授权登录")
    @ApiImplicitParam(name = "code",  value = "小程序授权code",paramType = "path")
    @PostMapping("/token/{code}")
    public Result wechatLogin(@PathVariable String code, @RequestBody UserInfo userInfo) {
        OAuthToken token = wechatAuthService.login(code, userInfo);
        return Result.success(token);
    }*/

    @ApiOperation(value = "注销", notes = "logout")
    @DeleteMapping("/logout")
    public Result logout() {
        JSONObject payload = JwtUtils.getJwtPayload();
        String jti = payload.getStr(AuthConstants.JWT_JTI); // JWT唯一标识
        Long expireTime = payload.getLong(AuthConstants.JWT_EXP); // JWT过期时间戳(单位：秒)
        if (expireTime != null) {
            long currentTime = System.currentTimeMillis() / 1000;// 当前时间（单位：秒）
            if (expireTime > currentTime) { // token未过期，添加至缓存作为黑名单限制访问，缓存时间为token过期剩余时间
                redisTemplate.opsForValue().set(AuthConstants.TOKEN_BLACKLIST_PREFIX + jti, null, (expireTime - currentTime), TimeUnit.SECONDS);
            }
        } else { // token 永不过期则永久加入黑名单
            redisTemplate.opsForValue().set(AuthConstants.TOKEN_BLACKLIST_PREFIX + jti, null);
        }
        return Result.success("注销成功");
    }

    @ApiOperation(value = "获取公钥", notes = "login")
    @GetMapping("/public-key")
    public Map<String, Object> getPublicKey() {
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        RSAKey key = new RSAKey.Builder(publicKey).build();
        return new JWKSet(key).toJSONObject();
    }
}
```

#### 7.3.5 步骤

>  -1.  测试

```http
http://localhost:9999/oauth/token?username=admin&password=123456&grant_type=password
 Header: Authorization=Basic eW91bGFpLWFkbWluOjEyMzQ1Ng==
 or Authorization: type=Basic Auth ； username=youlai-admin   password=123456
 
 
new String(new BASE64Decoder().decodeBuffer("eW91bGFpLWFkbWluOjEyMzQ1Ng=="), "UTF-8") 
 => youlai-admin:123456
 
 // 返回token数据
 // 然后使用token调用
 http://localhost:9999/api/user/1
 Header: Authorization = bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsInNjb3BlIjpbImFsbCJdLCJleHAiOjE2MjY1NzMwNTMsInVzZXJJZCI6MSwiYXV0aG9yaXRpZXMiOlsiUk9PVCIsIkFETUlOIl0sImp0aSI6IjBmMWI2OTY1LTc1YTItNDIxNC04M2U5LTQxMzdjZTQzZTc2YyIsImNsaWVudF9pZCI6InlvdWxhaS1hZG1pbiIsInVzZXJuYW1lIjoiYWRtaW4ifQ.C2jtBn09ATGPxv_6yFOb6iydmjIcI4FCjK_A-HNkgxANyQATwy2A7sxS_bAYMJYi5CYtQFFeVuTuQ2XDMq2aIaRUXxYLW655ys05YoxBY4xAwaCluDsharGTvtvgcwziKREYMnUI2rgVs2UuLpoPrdmt9FPj7UGLSLyvU1J5iQKGGzf4R_zCkYvQBwYcFjCN3pixlmZvSQbKLMWlE6tnXiIGMcg-5ky70GPkGm729QdqMybGFyS3jSuaG0pwq2yV98GsyQg9ZuFU-LVQJRqGkB9YkJ8NAFgcZb87XYI9YpkZj8WKqgY0h7smljfaMGIjk9L5FKKn7M2u-d17YF_bNw
```



> 0. Security放行

```java
http.authorizeRequests().antMatchers("/oauth/**").permitAll()
```

> 1. 组装客户端详情 ClientDetails  ClientDetailsService.loadClientByClientId(String clientId);

> 2. 执行controller获取token

```java
@ApiImplicitParams({
            @ApiImplicitParam(name = "grant_type", defaultValue = "password", value = "授权模式", required = true),
            @ApiImplicitParam(name = "client_id", value = "Oauth2客户端ID（新版本需放置请求头）", required = true),
            @ApiImplicitParam(name = "client_secret", value = "Oauth2客户端秘钥（新版本需放置请求头）", required = true),
            @ApiImplicitParam(name = "refresh_token", value = "刷新token"),
            @ApiImplicitParam(name = "username", defaultValue = "admin", value = "登录用户名"),
            @ApiImplicitParam(name = "password", defaultValue = "123456", value = "登录密码")
    })
    @PostMapping("/token")
    public Object postAccessToken(
            @ApiIgnore Principal principal,
            @ApiIgnore @RequestParam Map<String, String> parameters
    ) {
        // 本质上，执行的是oauth2提供的POST方法
        tokenEndpoint.postAccessToken(principal, parameters).getBody();
        // @RequestMapping(value = "/oauth/token", method=RequestMethod.POST)
		// public ResponseEntity<OAuth2AccessToken> postAccessToken(Principal principal, @RequestParam
		// Map<String, String> parameters)
        // 获取token，里面包含了loadUserByUsername(username)组装用户信息
        // OAuth2AccessToken token = getTokenGranter().grant(tokenRequest.getGrantType(), tokenRequest)
    }
```

- token response

```json
{
    "code": "00000",
    "data": {
        "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsInNjb3BlIjpbImFsbCJdLCJleHAiOjE2MjY1Mzg4OTAsInVzZXJJZCI6MSwiYXV0aG9yaXRpZXMiOlsiUk9PVCIsIkFETUlOIl0sImp0aSI6IjA5MDAzZmEyLWE1MTQtNDFlNy1iMDNkLTU3NGYyMDE4ODM3ZSIsImNsaWVudF9pZCI6InlvdWxhaS1hZG1pbiIsInVzZXJuYW1lIjoiYWRtaW4ifQ.Ac76jp-k3nKiDihxMHLRBXBTqhceJxkA5buXp3avCOMgdgy9GBBC_cELyn62SHv_Jw4ZZEil6p6gWhcy2dp2vcgrfoav7h1SCmxejzsK9FogPNdI2t73bOFMNuV_lML4fLPpJ5UJWklyDOFqDXoRYV1J9yZ28Llv-VzNUExO_TYJOhQ5RWBiMWlenca9L3vmXVyb074vAR5e_TK5Aw6bvLwb6Ag5LUrrX5MhMILZK-zs3bhgzX8NEYelO1cW7G6Zd1V-yW8nhgsW5zscUwZQeq3BHq8nNV-_jVEKPixMjZY-j1SLT5MtCTleIuZQoInkZ77Q1qhqn9WD4C9mXQ0gyg",
        "token_type": "bearer",
        "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsInNjb3BlIjpbImFsbCJdLCJhdGkiOiIwOTAwM2ZhMi1hNTE0LTQxZTctYjAzZC01NzRmMjAxODgzN2UiLCJleHAiOjE2MjkwODc2ODksInVzZXJJZCI6MSwiYXV0aG9yaXRpZXMiOlsiUk9PVCIsIkFETUlOIl0sImp0aSI6Ijg2MDllMTQwLWQ0OTItNDBhMC1iN2M5LTdhNjg3NDFkYzE0NyIsImNsaWVudF9pZCI6InlvdWxhaS1hZG1pbiIsInVzZXJuYW1lIjoiYWRtaW4ifQ.A2bHQtEA3JXgVVEVqY_TEuc0el471KIbHs4gbOgLpkC_tg_WxqPX1IhUJomLXb9_JMXvIiwGlL32wZYjvT2v7C0bOvemzo-rwYNFRQHz2PLUx0nFm8trA3btQiDQqAoGJ3WbqDiSkmfVB9cRSYQu74LSnY0rn0P47BGf7_7A3zvz34lP7PvdIlMENOEvVH67gwsYcmshD9mq4PKrtHHUMWrPDAhP_Oq6-9wY7eAeuUVtqeI0OIYrysyUi-roKiYphX56piwQ6KBhK3ZkKuusjh5EHuykb8tWSIRm9odIzyPL0awqB8moT5mywkNIN5XwJyOe5W46ceE7T4CN_kjPyg",
        "expires_in": 43183,
        "scope": "all",
        "userId": 1,
        "username": "admin",
        "jti": "09003fa2-a514-41e7-b03d-574f2018837e"
    },
    "msg": "一切ok"
}
```

### 7.4 双token刷新、续期，access_token和refresh_token实效如何设置

https://blog.csdn.net/a704397849/article/details/90216739

## 8. Shiro 权限管理

- pom

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-starter</artifactId>
    <version>${shiro.version}</version>
</dependency>
```

- java

```java
package com.codeman.springbootshiro.config;

import lombok.extern.java.Log;
import lombok.extern.slf4j.Slf4j;
import org.apache.shiro.cache.CacheManager;
import org.apache.shiro.cache.MemoryConstrainedCacheManager;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.filter.authc.AnonymousFilter;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import  org.apache.shiro.mgt.SecurityManager;
import org.springframework.context.annotation.DependsOn;
import org.springframework.core.annotation.Order;

import javax.servlet.Filter;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

/**
 * @author: zhanghongjie
 * @description:
 * @date: 2021/6/18 22:07
 * @version: 1.0
 */
@Configuration
@Slf4j
public class ShiroConfig {


    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setFilters(null);
        shiroFilterFactoryBean.setLoginUrl("/login");
        shiroFilterFactoryBean.setSuccessUrl("/success");
        shiroFilterFactoryBean.setUnauthorizedUrl("/authorize");
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        LinkedHashMap<String, String> map = new LinkedHashMap<>();
        /*
        * anon : 无需认证
        * authc: 必须认证
        * user： 必须拥有”记住我“功能
        * perms：拥有对某个资源的权限
        * role： 拥有某个角色才有权限
        *
        * */
        map.put("/**", "anon");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
        // 添加自己的过滤器并且取名为jwt
        Map<String, Filter> filterMap = new HashMap<String, Filter>(1);
        //如果cloudServer为空 则说明是单体 需要加载跨域配置
        filterMap.put("/**", new AnonymousFilter());
        shiroFilterFactoryBean.setFilters(filterMap);

        return shiroFilterFactoryBean;
    }


    @Bean("securityManager")
    public DefaultWebSecurityManager securityManager(ShiroRealm shiroRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
//        securityManager.login();
//        securityManager.logout();
        securityManager.setCacheManager(cacheManager());
        securityManager.setRealm(shiroRealm);
        return securityManager;
    }

    @Bean
    public CacheManager cacheManager() {
        return new MemoryConstrainedCacheManager();
    }

}


@RestController
public class LoginController {

    @GetMapping("/success")
    public String success() {
        return "success";
    }

    @GetMapping("/failure")
    public String failure() {
        return "failure";
    }

    @GetMapping("/login")
    public String login(@RequestParam String username, @RequestParam String password, Model model) {

        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken auth = new UsernamePasswordToken(username, password);
        try {
            subject.login(auth);
        } catch (AuthenticationException e) {
            // 失败的处理
            return "登录失败！";
        }

        return "登录成功！";
    }
}



package com.codeman.springbootshiro.config;

import lombok.extern.slf4j.Slf4j;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.stereotype.Component;

import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * @author: zhanghongjie
 * @description:
 * @date: 2021/6/18 22:15
 * @version: 1.0
 */
@Component
@Slf4j
public class ShiroRealm extends AuthorizingRealm {

    private final static Map<String, String> USER = Stream.of("user1", "user2")
            .collect(Collectors.toMap(String::toString, String::toString));

    @Override
    // Authorization 授权
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        log.info("===============Shiro权限认证开始============ [ roles、permissions]==========");
        String username = null;
        if (principals != null) {
            LoginUser sysUser = (LoginUser) principals.getPrimaryPrincipal();
            username = sysUser.getUsername();
        }
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();

        // 设置用户拥有的角色集合，比如“admin,test”
        Set<String> roleSet = commonAPI.queryUserRoles(username);
        System.out.println(roleSet.toString());
        info.setRoles(roleSet);

        // 设置用户拥有的权限集合，比如“sys:role:add,sys:user:add”
        Set<String> permissionSet = commonAPI.queryUserAuths(username);
        info.addStringPermissions(permissionSet);
        System.out.println(permissionSet);
        log.info("===============Shiro权限认证成功==============");
        return info;
    }

    @Override
    // Authentication 验证
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken auth) throws AuthenticationException {
        UsernamePasswordToken auth1 = (UsernamePasswordToken) auth;
        if (log.isInfoEnabled()) {
            log.info("doGetAuthenticationInfo");
        }
        String psw = null;
        if ((psw = USER.get(auth1.getUsername())) == null) {
            // 没有账号就return null
            return null;
        }
        return new SimpleAuthenticationInfo("", psw.toCharArray(), "");
    }
}

```





