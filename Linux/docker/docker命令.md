# docker命令

## 0. docker国内镜像

```shell
创建或修改 /etc/docker/daemon.json 文件

# vi /etc/docker/daemon.json
{
    "registry-mirrors": ["https://registry.docker-cn.com"]
}
systemctl restart docker.service

Docker中国区官方镜像
https://registry.docker-cn.com

网易
http://hub-mirror.c.163.com

ustc 
https://docker.mirrors.ustc.edu.cn

中国科技大学
https://docker.mirrors.ustc.edu.cn

阿里云容器  服务
https://cr.console.aliyun.com/
首页点击“创建我的容器镜像”  得到一个专属的镜像加速地址，类似于“https://1234abcd.mirror.aliyuncs.com”

```

## 1.镜像

### 1.1 拉取远程镜像

```shell
docker pull xxx
```

### 1.2 查看本地镜像库

```shell
docker images
```

### 1.3 运行镜像(mysql)

```shell
docker run -itd --name mysql02 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 docker.io/mysql:5.7
-- 参数详解
--name mysql02 起别名
-p 3307:3306   设置端口，容器内部3306映射运行docker的主机3307(内部3306,如果内部被修改了，那么映射也要改，比如内部改成3309，那么应该 -p 3307:3309)
-e MYSQL_ROOT_PASSWORD=123456 设置密码
-d: 后台运行容器，并返回容器ID；
-i: 以交互模式运行容器，通常与 -t 同时使用；
-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用
-v 本地目录:容器目录。挂载主机的本地目录 /usr/ToolsAPIDir 目录到容器的/ToolsAPIDir1 目录，本地目录的路径必须是绝对路径
 docker.io/mysql:5.7 镜像名:版本tag
```

![1605410214214](E:\SoftwareNote\Linux\docker\images\docker运行镜像.png)

#### 1.3.1 启动镜像

```shell
docker start container_id（id）/names(名称)
```

#### 1.3.2 停止镜像

```shell
docker stop container_id（id）/names(名称)
```

#### 1.3.3 重启镜像

```shell
docker restart container_id（id）/names(名称)
```

#### 1.3.4 移除已安装镜像

```shell
docker rm [-f] container_id（id）/names(名称)
```

### 1.4 查看镜像状态，启动日志

```shell
docker ps 【-a】
不加-a，只展示运行中的。
加-a, 展示全部已安装的

docker logs seata-server #查看启动日志
```

![1605410468485](E:\SoftwareNote\Linux\docker\images\docker-ps.png)

### 1.5 删除本地镜像

```shell
docker rmi -f mall4sb:v1
```



## 2. 报错

### 2.1 IPv4 forwarding is disabled. Networking will not work. 

步骤1：在宿主机上执行echo "net.ipv4.ip_forward=1" >>/usr/lib/sysctl.d/00-system.conf 

步骤2：重启网络和docker。systemctl restart network && systemctl restart docker 

### 2.2 docker内部command not found

```shell
1. 
apt-get update
2. 
apt-get install vim
3. 
vim  ~/.bashrc
4.

```

## 3. 进入镜像内部

### 3.1 进入tomcat

```shell
 docker exec -it 运行的tomcat容器ID /bin/bash 进入到tomcat的目录
```

## 4. 本地文件夹映射到docker内部

- 可以实现自动拷贝/热部署/日志外移等好处

```shell
docker run -d -p 9090:8080 -v /usr/local/mysoftware/testtomcat/webapps/:/usr/local/tomcat/webapps/ --privileged=true --name tomcat_mall4sb docker.io/tomcat:8.5.59

--privileged=true 给docker加上权限，否则war都无法解析

docker run  -v /usr/ToolsAPIDir:/ToolsAPIDir1 -d -p 5005:5004 -it toolsapi:v8 python3 tools_api.py
-v 本地目录:容器目录。挂载主机的本地目录 /usr/ToolsAPIDir 目录到容器的/ToolsAPIDir1 目录，本地目录的路径必须是绝对路径
```

## 5. 定制镜像（新增插件/配置设置等）

### 5.1 编写Dockerfile

```shell
# Docker file for date and locale set
# VERSION 0.0.3
# Author: bolingcavalry
#基础镜像
FROM docker.io/tomcat:8.5.59
#作者
MAINTAINER BolingCavalry <fresh_zz@163.com>
#定义时区参数
ENV TZ=Asia/Shanghai
#设置时区
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo '$TZ' > /etc/timezone
#安装必要应用
RUN yum -y install kde-l10n-Chinese glibc-common
RUN yum -y install vim
#设置编码
RUN localedef -c -f UTF-8 -i zh_CN zh_CN.utf8
#设置环境变量
ENV LC_ALL zh_CN.utf8
~

```

### 5.2 制作镜像

```shell
在Dockerfile文件所在目录执行命令
docker build -t mytomcat8.5:0.0.1 .
即可完成镜像制作, 会出现在docker images列表中
```

## 6. IDEA中连接远程Docker

- 插件: docker integration

![1605453449769](E:\SoftwareNote\Linux\docker\images\idea_plugin.png)

- dockerfile配置

	[1605453626041](E:\SoftwareNote\Linux\docker\images\idea_dockerfile.png) 	- dockerfile配置

![1605453661673](E:\SoftwareNote\Linux\docker\images\idea_dockerfile_edit.png)

- 可视化docker参数

![1605453774572](E:\SoftwareNote\Linux\docker\images\idea_可视化docker数据.png)

## 7. Docker暴漏端口

### 7.1 编辑docker.service文件 

```shell
vim /usr/lib/systemd/system/docker.service

在ExecStart=/usr/bin/dockerd 后插入 -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```

### 7.2 重启虚拟机、重启docker 

```shell
systemctl daemon-reload //重启虚拟机
systemctl restart docker //重启docker
```

### 7.3 检查

```shell
可选如下命令：
systemctl status docker //查看docker状态
netstat -tulp //查看所有运行端口号
netstat -tnlp | grep:2375 //查看2375端口号状态
```

## 8.docker运行参数可自行选择

```shell
Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]   

   
  -d, --detach=false         指定容器运行于前台还是后台，默认为false    
  -i, --interactive=false    打开STDIN，用于控制台交互   
  -t, --tty=false            分配tty设备，该可以支持终端登录，默认为false   
  -u, --user=""              指定容器的用户   
  -a, --attach=[]            登录容器（必须是以docker run -d启动的容器） 
  -w, --workdir=""           指定容器的工作目录  
  -c, --cpu-shares=0         设置容器CPU权重，在CPU共享场景使用   
  -e, --env=[]               指定环境变量，容器中可以使用该环境变量   
  -m, --memory=""            指定容器的内存上限   
  -P, --publish-all=false    指定容器暴露的端口   
  -p, --publish=[]           指定容器暴露的端口  
  -h, --hostname=""          指定容器的主机名   
  -v, --volume=[]            给容器挂载存储卷，挂载到容器的某个目录   
  --volumes-from=[]          给容器挂载其他容器上的卷，挂载到容器的某个目录 
  --cap-add=[]               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities   
  --cap-drop=[]              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities   
  --cidfile=""               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法   
  --cpuset=""                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU   
  --device=[]                添加主机设备给容器，相当于设备直通   
  --dns=[]                   指定容器的dns服务器   
  --dns-search=[]            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件   
  --entrypoint=""            覆盖image的入口点   
  --env-file=[]              指定环境变量文件，文件格式为每行一个环境变量   
  --expose=[]                指定容器暴露的端口，即修改镜像的暴露端口   
  --link=[]                  指定容器间的关联，使用其他容器的IP、env等信息   
  --lxc-conf=[]              指定容器的配置文件，只有在指定--exec-driver=lxc时使用   
  --name=""                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字   
  --net="bridge"             容器网络设置: 
                                bridge 使用docker daemon指定的网桥      
                                host    //容器使用主机的网络   
                                container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源   
                                none 容器使用自己的网络（类似--net=bridge），但是不进行配置  
  --privileged=false         指定容器是否为特权容器，特权容器拥有所有的capabilities ,capabilities可以分为线程capabilities和文件capabilities,capabilities应该为功能的意思
  --restart="no"             指定容器停止后的重启策略: 
                                no：容器退出时不重启   
                                on-failure：容器故障退出（返回值非零）时重启  
                                always：容器退出时总是重启   
  --rm=false                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)   
  --sig-proxy=true           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理
```

如：docker run --env-file=env.list --name nacosallenv2 -d -p 8888:8848 nacos/nacos-server 指定了env.list文件作为环境变量，这样就不用每次运行都携带参数

- 已启动的容器也可更新运行参数

  ```shell
  docker update --restart=always <CONTAINER ID>
  
  ```

  

## 9. Docker容器内部文件与外部文件的替换

首先把容器里面的配置复制出来

```
 docker cp my-nginx:/etc/nginx/conf.d/nginx.conf  /root/Downloads
1
```

修改复制出来目录的文件

```
 cd /root/Downloads
 vim nginx.conf
12
```

修改完成后，替换掉容器里面的配置文件

```
 docker cp nginx.conf my-nginx:/etc/nginx/conf.d/nginx.conf
```

## 10. docker系统命令

```shell
 重启：service docker restart 
```



## 11. 查看docker容器信息

```shell
（1）进入容器内部获取信息；
docker exec -it my_redis /bin/bash

root@ad95adf3a459:/data# hostname
root@ad95adf3a459:/data# ip addr
root@ad95adf3a459:/data# env

（2）执行docker exec命令；
[root@localhost docker]# docker exec my_tomcat hostname
[root@localhost docker]# docker exec my_tomcat ip addr
[root@localhost docker]# docker exec my_tomcat env
（3）执行docker inspect命令（推荐）；
docker inspect 容器名
```

## 12. 进入Redis

docker exec -it redis210420 redis-cli