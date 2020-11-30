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

#### 4.1.1 Eureka

##### 4.1.1.1 Eureka系统架构

- 客户服务+Eureka服务集群+提供者服务集群
- 提供者服务集群会注册在Eureka集群中，客户服务调用前会先向Eureka查询注册信息，发现提供者URL再进行调用

![1606749404796](E:\SoftwareNote\微服务\SpringCloud\img\Eureka系统架构.png)

##### 4.1.1.2 Eureka组成

- Eureka Server和Eureka Client
- Eureka Server提供服务注册服务
- Eureka Client通过注册中心进行访问。是一个JAVA客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的/使用轮询负载算法的负载均衡器。再应用启动后，将会向Euerka Server发送心跳（默认30s）。如果Eureka Server在多个心跳周期内美哟接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除（默认90s）