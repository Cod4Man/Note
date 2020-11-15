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
-p 3307:3306   设置端口，容器内部3306映射运行docker的主机3307
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

### 1.4 查看镜像状态

```shell
docker ps 【-a】
不加-a，只展示运行中的。
加-a, 展示全部已安装的
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

![1605453626041](E:\SoftwareNote\Linux\docker\images\idea_dockerfile.png) 	- dockerfile配置

![1605453661673](E:\SoftwareNote\Linux\docker\images\idea_dockerfile_edit.png)

- 可视化docker参数

![1605453774572](E:\SoftwareNote\Linux\docker\images\idea_可视化docker数据.png)