# Kubernetes

## 0. 文档

### 0.1 官方文档

[Kubernetes官方文档](https://kubernetes.io/zh/ "[Kubernetes官方文档](https://kubernetes.io/zh/)")

### 0.2 中文社区

[中文社区](https://www.kubernetes.org.cn/ "中文社区")

### 0.3 Kubesphere 文档

[kubesphere中文文档](https://kubesphere.com.cn/docs)

## 1. 集群搭建

### 1.0 搭建步骤

https://www.kubernetes.org.cn/7189.html

修改版本为最新就好，1.21.3

从节点需要跟着做到init之前，然后join

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

> 1. 生成一条永久有效的token

```shell
kubeadm token create --ttl 0
```

> 查看token的list

```shell
# kubeadm token list
TOKEN                     TTL         EXPIRES                     USAGES                   DESCRIPTION   EXTRA GROUPS
dxnj79.rnj561a137ri76ym   <invalid>   2018-11-02T14:06:43+08:00   authentication,signing   <none>        system:bootstrappers:kubeadm:default-node-token
o4avtg.65ji6b778nyacw68   <forever>   <never>                     authentication,signing   <none>        system:bootstrappers:kubeadm:default-node-token
```

> 2.获取ca证书sha256编码hash值

```shell
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

2cc3029123db737f234186636330e87b5510c173c669f513a9c0e0da395515b0
```

> 3. 新节点加入集群

```shell
kubeadm join 10.167.11.153:6443 --token o4avtg.65ji6b778nyacw68 --discovery-token-ca-cert-hash sha256:2cc3029123db737f234186636330e87b5510c173c669f513a9c0e0da395515b0
-v=10 # 可以查看日志，加入失败可以看错
```

- 加入失败

  > 已经加过了，可以重置后再加入
  >

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

### 1.3 master

#### 1.3.1 获取master 的token

```shell
 kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

### 1.4 kubectl命令

#### 1.4.1 查看status

```shell
systemctl status kubelet -l
```

#### 1.4.2 查看节点

```shell
kubectl get nodes
NAME    STATUS   ROLES                  AGE   VERSION
node1   Ready    control-plane,master   57m   v1.21.3
```

#### 1.4.3 查看pod

```shell
kubectl get pod --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-78d6f96c7b-c6ft7   1/1     Running   1          33m
kube-system   calico-node-nwzjs                          1/1     Running   1          33m
kube-system   coredns-59d64cd4d4-lq5q2                   1/1     Running   1          58m
kube-system   coredns-59d64cd4d4-qs86h                   1/1     Running   1          58m
kube-system   etcd-node1                                 1/1     Running   1          59m
kube-system   kube-apiserver-node1                       1/1     Running   2          59m
kube-system   kube-controller-manager-node1              1/1     Running   6          59m
kube-system   kube-proxy-5jtbd                           1/1     Running   1          58m
kube-system   kube-scheduler-node1                       1/1     Running   7          59m
```

#### 1.4.4 查看集群状态

```shell
kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok  
scheduler            Healthy   ok  
etcd-0               Healthy   {"health":"true"}
```

#### 1.4.5 查看运行状态

```shell
kubectl get pods  prometheus-85bdc4bf49-97djt -n ops -o yaml

kubectl describe pod mysql-master-dbm26 -n nacos

status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2021-08-04T14:25:53Z"
    message: '0/3 nodes are available: 3 pod has unbound immediate PersistentVolumeClaims.'
    reason: Unschedulable
    status: "False"
    type: PodScheduled
  phase: Pending
  qosClass: Burstable

```

#### 1.4.6 监听 watch

```shell
watch kubectl get pod -n kube-system -o wide
```

#### 1.4.7 暴露端口 expose

```shell

kubectl expose deployment tomcat6 --port=80 --targer-port=8080 --type=NodePort
Pod的80映射到容器的8080，service会代理Pod的80，即暴露80给外部，映射到内部8080
```

#### 1.4.8 扩容 replicas

```shell
kubectl scale --replicas=3 deployment tomcat6
```

#### 1.4.9 查看/导出yml

```shell
kubectl get pod <pod-name> -o yaml

导出
kubectl get pod `<pod-name>` -o yaml > root/temp/pod-name.yam
```

#### 1.4.10 通过yml新增/更新 create/apply -f yml

```shell
kubectl apply -f xxx.yml
```

#### 1.4.11 执行命令 exec

```shell
执行Pod的data命令，默认是用Pod中的第一个容器执行

kubectl exec <pod-name> data
指定Pod中某个容器执行data命令

kubectl exec <pod-name> -c <container-name> data
通过bash获得Pod中某个容器的TTY，相当于登录容器

kubectl exec -it <pod-name> -c <container-name> bash
```

#### 1.4.12 删除

```shell
基于yaml定义的名称删除资源对象

kubectl delete -f deployment.yaml
删除所有包含某个label的Pod和service

kubectl delete pods,services -l name=<label-name>
删除所有Pod

kubectl delete pods --all
```

#### 1.4.13 描述 describe

```shell
显示Node的详细信息

kubectl describe nodes <node-name>
显示Pod的详细信息

kubectl describe pods/<pod-name>
显示由deployment管理的Pod的信息

kubectl describe pods nginx-deployment
```

#### 1.4.14 日志 logs

```shell
kubectl logs [-f] [-p] POD [-c CONTAINER]

-c, --container="": 容器名
 
-f, --follow[=false]: 指定是否持续输出日志
    --interactive[=true]: 如果为true，当需要时提示用户进行输入。默认为true
    --limit-bytes=0: 输出日志的最大字节数。默认无限制
 
-p, --previous[=false]: 如果为true，输出pod中曾经运行过，但目前已终止的容器的日志
    --since=0: 仅返回相对时间范围，如5s、2m或3h，之内的日志。默认返回所有日志。只能同时使用since和since-time中的一种
    --since-time="": 仅返回指定时间（RFC3339格式）之后的日志。默认返回所有日志。只能同时使用since和since-time中的一种
    --tail=-1: 要显示的最新的日志条数。默认为-1，显示所有的日志

```

### 1.5  recommended.yaml

```yaml
# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

apiVersion: v1
kind: Namespace
metadata:
  name: kubernetes-dashboard

---

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard

---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      nodePort: 30001
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kubernetes-dashboard
type: Opaque

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-csrf
  namespace: kubernetes-dashboard
type: Opaque
data:
  csrf: ""

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-key-holder
  namespace: kubernetes-dashboard
type: Opaque

---

kind: ConfigMap
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-settings
  namespace: kubernetes-dashboard

---

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
rules:
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs", "kubernetes-dashboard-csrf"]
    verbs: ["get", "update", "delete"]
    # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["kubernetes-dashboard-settings"]
    verbs: ["get", "update"]
    # Allow Dashboard to get metrics.
  - apiGroups: [""]
    resources: ["services"]
    resourceNames: ["heapster", "dashboard-metrics-scraper"]
    verbs: ["proxy"]
  - apiGroups: [""]
    resources: ["services/proxy"]
    resourceNames: ["heapster", "http:heapster:", "https:heapster:", "dashboard-metrics-scraper", "http:dashboard-metrics-scraper"]
    verbs: ["get"]

---

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
rules:
  # Allow Metrics Scraper to get metrics from the Metrics server
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods", "nodes"]
    verbs: ["get", "list", "watch"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard

---

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
        - name: kubernetes-dashboard
          image: registry.cn-hangzhou.aliyuncs.com/huxuezheng/k8s-dashboard:v2.0.0-beta8
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              protocol: TCP
          args:
            - --auto-generate-certificates
            - --namespace=kubernetes-dashboard
            # Uncomment the following line to manually specify Kubernetes API server Host
            # If not specified, Dashboard will attempt to auto discover the API server and connect
            # to it. Uncomment only if the default does not work.
            # - --apiserver-host=http://my-address:port
          volumeMounts:
            - name: kubernetes-dashboard-certs
              mountPath: /certs
              # Create on-disk volume to store exec logs
            - mountPath: /tmp
              name: tmp-volume
          livenessProbe:
            httpGet:
              scheme: HTTPS
              path: /
              port: 8443
            initialDelaySeconds: 30
            timeoutSeconds: 30
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsUser: 1001
            runAsGroup: 2001
      volumes:
        - name: kubernetes-dashboard-certs
          secret:
            secretName: kubernetes-dashboard-certs
        - name: tmp-volume
          emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      nodeSelector:
        "beta.kubernetes.io/os": linux
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule

---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: dashboard-metrics-scraper
  name: dashboard-metrics-scraper
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 8000
      targetPort: 8000
  selector:
    k8s-app: dashboard-metrics-scraper

---

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: dashboard-metrics-scraper
  name: dashboard-metrics-scraper
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: dashboard-metrics-scraper
  template:
    metadata:
      labels:
        k8s-app: dashboard-metrics-scraper
      annotations:
        seccomp.security.alpha.kubernetes.io/pod: 'runtime/default'
    spec:
      containers:
        - name: dashboard-metrics-scraper
          image: registry.cn-hangzhou.aliyuncs.com/huxuezheng/metrics-scraper:v1.0.1
          ports:
            - containerPort: 8000
              protocol: TCP
          livenessProbe:
            httpGet:
              scheme: HTTP
              path: /
              port: 8000
            initialDelaySeconds: 30
            timeoutSeconds: 30
          volumeMounts:
          - mountPath: /tmp
            name: tmp-volume
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsUser: 1001
            runAsGroup: 2001
      serviceAccountName: kubernetes-dashboard
      nodeSelector:
        "beta.kubernetes.io/os": linux
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
      volumes:
        - name: tmp-volume
          emptyDir: {}
```

#### 1.6 kubernetes dashboard

获取token

https://blog.csdn.net/weixin_38320674/article/details/107328982

###1.99 异常

#### 1.99.1. k8s controller-manager scheduler Unhealthy

修改以下配置文件

/etc/kubernetes/manifests/kube-controller-manager.yaml

/etc/kubernetes/manifests/kube-scheduler.yaml

将两个文件中的

```
- --port=0
```

这一行注释掉

#### 1.99.2 The connection to the server localhost:8080 was refused

**从节点加入主节点时，提示这个。**

原因: 在没有配置config文件时，kube-apiserver默认使用的是localhost

解决方法：拷贝master节点的/etc/kubernetes/admin.conf配置文件到/etc/kubernetes/目录下，并执行如下命令

```shell
 mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 1.99.3 The connection to the server 192.168.1.181:6443 was refused - did you specify the right host or port?

**主节点重启服务器后提示。**

> 查看kubelet状态

```shell
[root@node1 ~]# systemctl status kubelet -l
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; disabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: inactive (dead)
     Docs: https://kubernetes.io/docs/

```

> dead，重启kubelet，提示node not found

```shell
systemctl restart kubelet

[root@node1 ~]# systemctl status kubelet -l
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; disabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since 三 2021-07-28 20:26:52 CST; 13s ago
     Docs: https://kubernetes.io/docs/
 Main PID: 3122 (kubelet)
    Tasks: 23
   Memory: 208.0M
   CGroup: /system.slice/kubelet.service
           ├─3122 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.4.1
           └─3488 /opt/cni/bin/calico

7月 28 20:27:05 node1 kubelet[3122]: E0728 20:27:05.020803    3122 kubelet.go:2291] "Error getting node" err="node \"node1\" not found"

```

> 再次重启，服务跑起来了，docker images也自动启动了

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

 　总的来说Kubernetes是来可以动态的利用策略解决**集群中资源调度，管理及监控等问题。

### 2.2  **Kubernetes核心概念及架构设计**

> Kubernete总体架构图

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

![image-20210725231445695](img\Node节点组成部分.png)

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

## 3.  dashboard

### 3.1 在 K8S 中安装 Kuboard

```sh
kubectl apply -f https://addons.kuboard.cn/kuboard/kuboard-v3.yaml
```

### 3.2 KubeSphere

## 4. nfs

[搭建nfs](https://www.cnblogs.com/miaoweiye/p/14754375.html)

[nfs原理及安装配置](https://www.cnblogs.com/whych/p/9196537.html)
