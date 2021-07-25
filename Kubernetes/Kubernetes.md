# Kubernetes

## 1. 集群搭建

### 1.1 aliyun Kubernetes 镜像

谷歌用不了：https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64

 **/etc/yum.repos.d/kubernetes.repo**

```shell
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 1.2 从节点加入主节点

#### 1.2.1 加入

```shell
kubeadm join 192.168.1.172:6443 --token 7vhvka.om4xwk67s28wriiw \
        --discovery-token-ca-cert-hash sha256:9856ece67acb29331f765ce4fd7f9dc0d9e18bc4f16dcc352c660f518fc05805
```

- 加入失败

  > 已经加过了，可以重置后再加入

```shell

[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
        [ERROR Port-10250]: Port 10250 is in use
        [ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher

```

#### 1.2.2 重置

```shell
kubeadm reset
```

## 2. Kubernate 介绍

### 2.1 用途

1. 容器自动化部署和复制，随时扩展或者收缩容器规模（资源配额和分配管理），并提供负载均衡。比如当某个容器内存使用不够，会自动扩展出一个副本出来。当多个副本提供服务的时候，能够保证负载均衡。

　2. 健康检查，自愈，容器滚动升级。

　3. 提供容器弹性，方便容器替换，比如有容器失效了Kubernetes就可以很容易将损害的容器从集群中替换掉。

　4. 将容器组织成组，并且提供容器间的负载均衡。

　5. 方便测试。可以将测试服务器集中化，自动化管理。可分配资源将各种平台的服务器加入集群，按需部署或者销毁。

　6. 持续集成时候方便对应用部署。

　7. 提供了对微服务的支撑 ，包括服务发现，服务编排，内部路由支持，服务快速部署，自动负载均衡等。

　8. 面向云原生可移植的新"云平台"。

 　总的来说Kubernetes是来可以动态的利用策略解决集群中资源调度，管理及监控等问题。

### 2.2  **Kubernetes核心概念及架构设计**

>  Kubernete总体架构图

![image-20210725230933704](.\img\Kubernete总体架构图.png)

#### 2.2.1 Master节点

> Master节点组成部分

![image-20210725231309315](.\img\Master节点组成部分.png)

Master是Cluster 的大脑，是控制整个Kubernetes集群的核心，它的主要职责是调度，即决定将应用放在哪里运行。运行着如下一系列Daemon服务：

1. **Api Server：**

k8s集群的接口和通讯总线，用户通过kubectl，dashboard，sdks等操作k8s都是通过Api  Server来与集群进行交互的。Api Server也可以作为一个订阅事件总线，其它外围组件可以订阅在Api  Server上，当有新的事件发生，Api Server可以将相关的事件来通知这些外围组件。

2. **Scheduler：**

负责Kubernetes集群调度和决策，掌握当前集群资源等使用情况。当有新的应用发布到Kubernetes集群中，Scheduler负责决策相应的Pod应该分不到哪些空闲的节点上。Kubernetes上的调度决策算法是可以进行扩展。

3. **Controller Manager：**

保证集群状态最终一致。通过Api  Server来监控集群的状态，比如果然一个应用需要10个Pod，那么Controller会保证实际就会启动10个Pod，如果其中有一个Pod挂了，Controller Manager会协调重新启动Pod，如果启动多了，Controller Manager也会协调关闭多余的Pod。

Controller Manager是集群实现自愈的机制。

4. **etcd：**

负责Kubernetes集群状态，节点数据，配置等信息的存储。可以独立部署，也可以和master部署在一起，etcd被Api Server来操作 。etcd部署一般需要三个节点来保证高可用。

5. **Pod网络。**

#### 2.2.2 **Work** **Node节点**

> Node节点组成部分

![image-20210725231445695](E:\SoftwareNote\Kubernetes\img\Node节点组成部分.png)

Work Node是Pod运行的实际节点，是Kubernetes资源的提供者。Node职责是运行容器应用。Kubernetes本身知道Docker和rkt等容器的运行。 在Node节点中，运行的Kubernetes组件包括：

1. **kubelet：**

Work节点的资源管理者，相当于agent这样的角色，它一直监听在Api Server上，根据Master节点的指示做出相关的动作，比如启动/关闭Pod。同时也会将所管理的节点信息收集并汇报给Master。

2. **kube-proxy：**

管理Kubernetes中service网络。Pod在Kubernetes中是不固定的（ephemeral），Pod的ip可能会变化，为了屏蔽Pod这种变化性，Kubernetes引入了service这样的概念，service可以屏蔽这样的变化，并且可以在调用的时候进行负载均衡，当需要将服务暴露给外部的时候，kube-proxy可以进行转发，kube-proxy就是实现背后这些服务网路的实现机制。

3. **Pod网络**

4. **Container Runtime：**

节点上容器资源的管理者，kubelet并不会直接管理节点上的容器，而是委托Container  Runtime进行管理。Container Runtime在启动容器的时候如果没有本地镜像缓存就会去Docker Hub上去拉取然后缓存在本地。

#### 2.2.3 Pod

Pod是kubernetes中一个非常重要的概念，Pod是放置容器的地方，因为所有的应用最终都是运行在Pod里的。

Pod里可以运行容器，那么Pod运行在哪呢？

Pod运行在一个node上。只要资源足够，一个node上可以有任意个pod。kubelet负责调度pod。

位于Work  node里面，Pod是基本操作单元，也是应用运行的载体，比如可以存放web服务器的容器。整个Kubernetes系统都是围绕着Pod展开，比如如何部署和运行Pod，如何保证Pod数量，如何访问Pod等。而之前所说的副本就是指Pod。另外Pod是有生命周期的。

分离关注点，每个容器只做一件事。

一般一个Pod里面只运行一个容器，当然也有一个Pod里运行多个容器，其中一个容器是主容器，其它的是辅助容器。

一个Pod里多个容器共享Pod的网络栈，存储资源等。

#### 2.2.4 Deployment

Deployment控制器定义了Pod部署信息，并控制Pod的部署并维持其状态。

### 2.3 Kubernetes发布流程

> Kubernetes发布流程

![image-20210725231828035](.\img\Kubernetes发布流程.png)

1. kubectl向API Server提交一个创建副本的应用请求，ReplicaSet副本集是规范副本数量的。

2. Controller Manager监听ReplicaSet创建及修改等事件，它接收到ReplicaSet的通知。

3. Controller Manager比较当前集群的状态和预期集群的状态，这里它会创建新的Pods。

4. Scheduler监听到需要创建Pod，就会根据调度算法来选择合适的Worker节点，然后在使用API  Server来更新Pod定义等状态，这时候应用还没有真正的发布，Controller Manager和Scheduler只是通过API  Server更新了集群所期望的状态。

5. 当Pod被分配到某个具体到worker节点，API Server就会通知相应节点上的Kubelet。

6. Kubelet接受到通知就会告知Container runtime在其节点上去下载，启动，并运行对应的容器，同时Kubelet会监控容器的运行。

| 组件               | 节点                | 作用                                                 |
| ------------------ | ------------------- | ---------------------------------------------------- |
| etcd               | Master 或者独立集群 | 集群状态集中存储                                     |
| API Server         | Master              | 集群接口和通讯总线                                   |
| Controller Manager | Master              | 协调发布状态最终一致性组件                           |
| Scheduler          | Master              | 协调决策组件                                         |
| Kubelet            | Worker              | Worker节点资源管理器                                 |
| Container runtime  | Worker              | 容器资源管理                                         |
| kube-proxy         | Worker              | 实现Service服务抽象组件，屏蔽PodIP的变化和负载均衡   |
| Pod                | Worker              | Kubernetes云平台中提供虚拟机，Kubernetes基本调度单位 |
| Container          | Worker              | 应用跑在容器中，资源隔离单位                         |
