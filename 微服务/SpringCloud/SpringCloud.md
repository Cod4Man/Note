# SpringCloud

## 1. 版本选择https://start.spring.io/actuator/info

![1606651696686](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloud版本细项选择.png)

![1606651411340](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloud版本选择.png)

![1606651789339](E:\SoftwareNote\微服务\SpringCloud\img\杨哥学习版本选择.png)

## 2. SpringCloud组件升级

![1606652566614](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloud组件升级.png)

## 3. 注解

### 3.1 @RequestBody

- 请求参数为实体类，需要增加参数注解

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
        return new RestTemplate();
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

![1606749404796](E:\SoftwareNote\微服务\SpringCloud\img\Eureka系统架构.png)

##### 4.1.1.2 Eureka组成

- Eureka Server(@EnableEurekaServer)和Eureka Client(@EnableEurekaServer)
- Eureka Server提供服务注册服务
- Eureka Client通过注册中心进行访问。是一个JAVA客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的/使用轮询负载算法的负载均衡器。再应用启动后，将会向Euerka Server发送心跳（默认30s）。如果Eureka Server在多个心跳周期内美哟接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除（默认90s）

##### 4.1.1.3 Eureka集群原理 

![1607007154428](E:\SoftwareNote\微服务\SpringCloud\img\Eureka集群原理.png)

##### 4.1.1.4 Eureka 自我保护机制

- 注册在Eureka的服务，一定时间间隔向Eureka注册中心发送心跳包，当超过时间间隔注册中心没有收到心跳包(可能是网络阻塞等)，并不会立即删除该服务，而是触发自我保护机制，仍然保留着失效的服务。

- Eureka服务端：eureka.server.enable-self-preservation = false

- Eureka客户端：eureka.instance.lease-renewal-interval-in-seconds=30

     			    eureka.instance.lease-expiration-duration-in-seconds=90

#### 4.1.2 Zookeeper :2181(需要客户端)

```yaml
spring:
  application:
    name: cloud-payment-service  # Eureka发现需要
  cloud:
    zookeeper:
      connect-string: 192.168.1.170:2181
```

- 无自我保护机制，没有心跳直接干掉服务。因此服务节点是临时性的
- client : zkCli.sh, 可查看注册信息ls /

#### 4.1.3 Consul: 8500(需要客户端)

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

  ![1607245896866](E:\SoftwareNote\微服务\SpringCloud\img\CAP理论.png)

- Eureka(AP)  

- Zookeeper/Consul(CP)

### 4.3 服务调用

#### 4.3.1 Ribbon

- Eureka自带Ribbon依赖

- 7大自带负载均衡规则

  - com.netflix.loadbalancer.RoundRobinRule 轮询
  - com.netflix.loadbalancer.RandomRule 随机
  - com.netflix.loadbalancer.RetryRule 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试
  - WeightedResponseTimeRule  对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
  - BestAvailableRule  会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
  - AvailabilityFilteringRule  先过滤掉故障实例，再选择并发较小的实例
  - ZoneAvoidanceRule 默认规则，复合判断server所在区域的性能和server的可用性选择服务器

- 使用及替换负载均衡规则

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
  
  // 入口指定服务及规则类
  @RibbonClient(name = "CLOUD-PAYMENT-SERVICE", configuration = RibbonRuleConfig.class)
  ```

  

#### 4.3.2 OpenFeign

- Feign: 是一个声明式WebService客户端，使用方法是定义一个服务接口，然后在上面写注释。Feign可以与Ribbon和Eureka组合使用以支持负载均衡

- Feign旨在使编写Java Http客户端变得更容易。不使用Feign，用Ribbon+RestTemplate通常都会针对每个微服务自动封装这些服务依赖的调用。所以Feign在此基础上进一步封装，由他帮助我们定义和实现依赖服务接口的定义，我们只需创建一个接口(映射服务接口)，并且使用注解的方式来配置他，即可完成对服务接口的绑定。

- Feign和OpenFeign的区别

  ![1607432204805](E:\SoftwareNote\微服务\SpringCloud\img\Feign和OpenFeign的区别.png)

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

  ![1607436697440](E:\SoftwareNote\微服务\SpringCloud\img\Feign日志级别.png)

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

- 熔断机制概述： 服务降级触发熔断，**当检测到该节点微服务调用响应正常后，恢复调用链路**。
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

  ![1608308356224](E:\SoftwareNote\微服务\SpringCloud\img\熔断器概念图.png)

- 熔断类型：

  - 熔断打开：请求不再进行调用当前服务，内部设置时钟一般为MTTR(平均故障处理时间)，当打开时长达到所设时钟则进入熔断状态
  - 熔断关闭：熔断关闭不会对服务进行熔断
  - 熔断半开：请求不再进行调用当前服务，内部设置时钟一般为MTTR(平均故障处理时间)，当打开时长达到所设时钟则进入熔断状态

- 官网断路器流程图

  ![1608309142716](E:\SoftwareNote\微服务\SpringCloud\img\官网断路器流程图.png)

- 断路器开启或者关闭的条件

  - 当满足一定阈值的时候（默认10秒内超过20个请求次数）
  - 当失败率达到一定(默认10秒内超过50%的请求失败)
  - 达到以上阈值，断路器将会开启
  - 当开启的时候，所有请求都不会进行转发
  - 一段时候之后(默认是5秒)，这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，若事变，继续开启

- 准实时的调用监控 Hystrix Dashboard

  - 注意：新版本Hystrix需要在主启动类MainAppHystrix8001中指定监控路径

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

  - 

### 4.5 服务网关

#### 4.5.1 Zuul1.0 不行

- Zuul1.x模型

  ![1608455324110](E:\SoftwareNote\微服务\SpringCloud\img\Zuul1.x模型.png)

  模型缺点：

  Servlet是一个简单的网络IO模型，当请求进入servlet container时，servekt container就会为其绑定一个线程，在并发不高的场景下，这种模型是可以适用的。而高并发下，线程数量就会上涨，而线程资源代价是昂贵的(上下文切换，内存消耗大)，严重影响请求的处理时间。在一些简单的业务场景下，不希望为每个requet分配一个线程，只需要1个或几个线程就能应对极大的并发请求，这种业务场景下servlet模型没有优势。

  而zuul1.x是基于servlet之上的一个阻塞式处理模型，即spring实现了处理所有request请求的一个servelt，并由该servelt阻塞式处理。

#### 4.5.2 Zuul2.0 研发中

#### 4.5.3 Gateway

##### 4.5.3.1 概述

- Gateway 是在Spring 生态系统之上构建的API网关服务，基于Spring5， SpringBoot2, Project Reactor等技术。旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能，如：熔断/限流/重试等。

- GateWay目标是替代Zuul。在SpringCloud2.0以上的版本中，没有对新版本的Zuul2.0以上最新最高性能版本进行集成，仍然使用的Zuul1.0非Reactor模式的老版本。而为了提高网关的性能，SpringCloud-Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。

- Gateway的目标：提供统一的路由方式且基于Filter链的方式提供了网关的基本功能，如：安全/监控/指标/限流等。

- Gateway源码架构（模型）

  ![1608451987988](E:\SoftwareNote\微服务\SpringCloud\img\Gateway源码架构.png)

  - 传统的web框架，如struts2，springMVC等都是基于servletAPI和Servelt容器基础之上进行的。
  - 在Servelt3.1之后有了异步非阻塞的支持。而WebFlux式一个典型的非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty，Undertow以及支持Servlet3.1的容器上。非阻塞+函数式变成(Spring5必须使用JAVA8)
  - Spring WebFlux式Spring5.0引入的新的响应式框架，区别于SpringMVC，它不依赖ServeltAPI，他是完全异步非阻塞的，并且基于Reactor来实现响应式流规范

- 用途

  - 反向代理
  - 鉴权
  - 流量控制
  - 熔断
  - 日志监控
  - ...

- 微服务架构中的网关（在服务层外层）

  ![1608452396670](E:\SoftwareNote\微服务\SpringCloud\img\微服务架构中的网关.png)

- 为什么选Gateway而不是Zuul

  - neflix不太靠谱，zuul2.0一直跳票,迟迟不发布
    - 虽然说Zuul2.0也是基于异步非阻塞模型上进行开发的，但是SpringCloud貌似没有整合计划，而是想用Gateway代之。而Netflix相关组件都宣布进入维护期，前景不知如何
    - Gateway是SpringCloud团队开发的，多了很多Zuul没有的功能，使用也更加简单便捷
  - Gateway具有如下特性：
    - 基于SpringFramework5，Project Reactor，SpringBoot2.0进行构建
    - 动态路由：能够匹配任何请求属性
    - 可以对路由进行指定Predicate(断言)和Filter(过滤器)
    - 集成Hystrix的断路器功能
    - 集成SpringCloud服务发现功能
    - 易于编写Predicate和Filter
    - 请求限流功能
    - 支持路径重写

  

- Gateway和zuul的区别

  - Zuul1.x是以及基于阻塞I/O的API Gateway
  - Zuul1.x基于Servlet2.5使用阻塞架构，它不支持任何长连接（如WebSocket），Zuul的涉及模式和Nginx比较像，而JVM本身会有第一次加载较慢的情况，使得Zuul的性能相对差
  - Zuul2.x理念更先进，像基于Netty非阻塞和支持长连接，但SpringCloud目前还没有进行整合。Zuul2.x的性能相对1.x有较大的提升。官方Gateway的RPS(每秒请求数)是Zuul的1.6倍
  - Gateway基于SpringFramework5，Project Reactor，SpringBoot2.0进行构建，非阻塞API
  - Gateway还支持WebSocket，并与Spring紧密集成拥有更好的开发体验

- Gatway三大核心概念

  - Route(路由)：路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

  - Predicate(断言)：参考的是java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由

  - Filter(过滤)：指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

    ![1608462225245](E:\SoftwareNote\微服务\SpringCloud\img\Gateway总体图.png)

    Gateway工作过程（核心逻辑：路由转发+执行过滤器链）

    ![1608462363044](E:\SoftwareNote\微服务\SpringCloud\img\Gateway工作过程.png)

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
          String uname = exchange.getRequest().getQueryParams().getFirst("username");
          if(StringUtils.isEmpty(uname)){
              log.info("*****用户名为Null 非法用户,(┬＿┬)");
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

  

### 4.6 配置中心

微服务会有一堆服务，每个服务都需要有自己的配置文件，因此需要一套集中式的，动态的配置管理设施。

#### 4.6.1 SpringCloud Config

![1608736024008](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloudConfig模型图.png)

- SpringCloud Config分为服务端和客户端。

  服务端也称分布式配置中心，他是一个独立的微服务应用(SpringBoot),用来连接配置服务器(git)并为客户端听过获取配置信息，加密/解密信息等接口。

- 用途：

  - 集中管理配置文件
  - 不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release
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
            uri: git@github.com:Cod4Man/microservice-config.git
            search-paths:
            - microservice-config
        label: master
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

  @EnableConfigServer

- 配置读取规则

  ![1608957986658](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloudConfig配置读取规则.png)

![1608959876710](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloudConfig配置读取规则2.png)

- bootstap.yml

  application.yml是用户级的资源配置项，而bootstrap.yaml是系统级的，优先级更高。

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
    ```

- Config客户端-动态刷新

  - pom引入actuator监控

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```

  - 修改yml，暴露监控端口

    ```yml
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

- SpringCloud Bus目前支持的消息中间件：RabbitMQ和Kafka

- 用途：SpringCloud Bus能管理和传播分布式系统间的消息，就像一个分布式执行器，可用于广播状态更改/事件推送等，也可以当作微服务间的通信通道。

  ![1608989250106](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloudBus工作过程.png)

- 总线：在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便的广播一些需要让其他连接在该主题上的实例都知道的消息。

- 基本原理：ConfigClient实例都监听MQ中同一个topic(默认是SpringCloudBus)。当一个服务刷新数据的时候，它会把这个信息放入到Topic中，这样其他监听同一Topic的服务就能得到通知，然后去更新自身的配置。

- 设计架构：

  - 方案一：利用消息总线触发一个客户端/bus/refresh,而刷新所有客户端的配置

    ![1608995149065](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloudBus设计架构1.png)

  - 方案二：利用消息总线触发一个服务端ConfigServer的/bus/refresh端点,而刷新所有客户端的配置（更加推荐）

    ![1608995201829](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloudBus设计架构2.png)

    - 打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新职责
    - 破坏了微服务各节点的对等性
    - 有一定的局限性。例如，微服务在迁移时，它的网络地址常常会发生变化，此时如果想要做到自动刷新，那就会增加更多的修改

- 添加消息总线支持：

  - pom

  ```xml
  <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-bus-amqp</artifactId>
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

  ![1608995629898](E:\SoftwareNote\微服务\SpringCloud\img\bus通知过程图.png)

### 4.8 消息驱动(SpringCloudStream) 

#### 4.8.1 定义

- 屏蔽底层消息中间件的差异，降低切换版本，统一消息的编程模型

- 官方定义SCS是一个构建消息驱动微服务的框架。

  应用程序通过inputs或outputs来与SpringCloudStream中binder对象交互。通过我们配置来binding(绑定)，而SpringCloudStream的binder对象负责与消息中间件交互。因此，开发人员只需搞清楚如何与SpringCLoudStream交互就可以方便使用消息驱动的方式。

  通过使用Spring Integration来连接消息代理中间件以实现消息事件驱动。SpringCloudStream为一些供应商的消息中间件产品提供了个性化的自动化配置实现，引用了发布-订阅/消费组/分区的三大核心概念。

- 目前仅支持**Rabbit MQ和Kafka**

#### 4.8.2 为什么用SpringCloudStream

- 不同消息中间件架构上不同，如Rabbit MQ有exchange，kafka有Topic和Partitions分区
- 不同中间件的差异对我们实际项目开发造成了一定的困扰，在使用其他一种后，后续需求需要往另一种消息中间件迁移，这是非常麻烦的，因为它和我们的系统耦合了。SpringCloudStream则能达到解耦的效果。

#### 4.8.3 绑定器binder

通过定义绑定器作为中间层，完美的实现了**应用程序与消息中间件细节之间的隔离**。通过向应用程序暴露统一的Channel通道，使得应用程序不需要再考虑各种不同的消息中间件实现。

![1609079227585](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloudStream处理架构.png)

#### 4.8.4 SpringCloudStream标准流程套路

![1609079362362](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloudStream标准流程套路.png)

- Binder：很方便的连接中间件，屏蔽差异
- Channel：通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过对Channel对队列进行配置
- Source和Sink：简单的可理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接受消息就是输入

#### 4.8.5 编码API和常用注解

![1609079467370](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloudStream编码API和常用注解.png)

#### 4.8.6 试用（RabbitMQ） 

- pom： 生产服务相同

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
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

    ![1609082910058](E:\SoftwareNote\微服务\SpringCloud\img\SpringCloudStream分组.png)

  - 持久化：

    当消费者系统异常(重启等)，生产者会将消息持久化(在这个分组中)。当服务分组改变时，服务正常后也无法接收到异常期间持久化的消息；而当服务分组没有改变，服务恢复正常后，可以重新接收到异常期间持久化的消息。