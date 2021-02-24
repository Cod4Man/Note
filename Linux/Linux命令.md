# Linux命令

## 1. 路径

## 1.1 当前路径

```shell
pwd
```

## 2. 文件夹

### 2.1 创建文件夹

```shell
mkdir dirName
```

### 2.2 重命名

```shell
例子：将目录A重命名为B

mv A B

例子：将/a目录移动到/b下，并重命名为c

mv /a /b/c
```

## 3. 磁盘容量

- “df  -h” 可以查看系统的分配，已使用和可用情况 
- “du -sh  *” 可以查看当前路径
- 每个文件夹的大小。此举可以快速定位大文件所存在的位置。 

## 4. 端口查看

```shell
systemctl status nginx //查看nginx状态
netstat -tnlp //查看所有运行端口号
netstat -tnlp | grep:98 //查看98端口号状态

1. 查找占用的程序。

端口号：netstat -apn | grep 4040
```

### 4.1 杀死指定端口线程

```shell
1. 查找占用的程序。

端口号：netstat -apn | grep 4040
最后一项显示的是pid和对应的名称
2. 杀掉对应的进程，彻底杀死进程。

端口号：kill -9 26105
```

### 4.2 查看进程的所有线程

```shell
ps -aux | grep java

java 可以直接用 jps -l 效果类似
```



## 5. 文件搜索-find

- 全盘搜索，也可以指定目录搜bai索。find 搜索目录 -name 目标名字，find / -name file（区分大小写）
- find / -iname file（不区分大小写）
- 我们先使用*通配符来匹配下包含file的相关文件区分大小写的：find / -name *file*。
- find 搜索目录 -size 文件大小。下面我们查找下大于100MB的文件,应该实际是102400KB*2,所有搜索命令为：find / -size +204800。-号是小于，直接写数字就是等于。
- find 搜索目录 -user 用户名。这里是查找属于用户名为user1的文件，linux如何添加删除用户名,可以参考Linux 用户管理命令：find / -user user1。
- find 搜索目录 -type d。查找某个目录下的所有目录：find /tmp -type d。
- find 搜索目录 -cmin -时间(单位分钟)。查找etc下面1小时内被修改的文件,根目录下面太多了,指定一个目录：find /etc -cmin -60。
- 当然find命令是可以多个选项一起添加查询的：-a 是前后条件都要满足，-o 是满足一个条件就好，这样我们可以清除的看到被过滤掉的文件。

## 6. 防火墙

```shell
1:查看防火状态

systemctl status firewalld

service  iptables status

2:暂时关闭防火墙

systemctl stop firewalld

service  iptables stop

3:永久关闭防火墙

systemctl disable firewalld

chkconfig iptables off

4:重启防火墙

systemctl enable firewalld

service iptables restart 
```

## 7. 生产运行性能诊断

### 7.1 整机uptime + top

- uptime是top的精简版

- top(系统性能命令)

  其中，load average平均负载展示的3个值分别为1min内平均，5min内平均，15min内平均。通过这3个值的大小，可以判断最近运行情况。

  ![1614092136585](E:\SoftwareNote\Linux\images\top命令.png)

### 7.2 CPU: vmstat

![1614093051480](E:\SoftwareNote\Linux\images\vmstat命令解析.png)

- 查看所有CPU核信息

  mpstat -P ALL 2

  ![1614093196718](E:\SoftwareNote\Linux\images\mpstat命令-查看所有CPU核情况.png)

- **每个进程**使用cpu的用量分解信息

  pidstat -u 1 -p 进程编号

  ![1614093247415](E:\SoftwareNote\Linux\images\pidstat命令-按进程查询CPU占用.png)

### 7.3 内存:free [-m(MB)/-g(GB)] 

- 应用程序可用内存

  ![1614093625078](E:\SoftwareNote\Linux\images\free命令-应用程序可用内存.png)

- 查看额外 pidstat -p 进程号 -r 采样间隔秒数

  ![1614093772086](E:\SoftwareNote\Linux\images\pidstat命令-查看额外内存信息.png)

### 7.5 硬盘 df [-m(MB)/-h] 

![1614093886679](E:\SoftwareNote\Linux\images\df命令-查看硬盘信息.png)

### 7.6 磁盘IO： iostat -xdk 2(秒) 3(次)

- 磁盘I/O性能评估

  ![1614094221174](E:\SoftwareNote\Linux\images\iostat命令-磁盘IO性能.png)

- 查看额外

  pidstat -d 采样间隔秒数 -p 进程号

  ![1614094355029](E:\SoftwareNote\Linux\images\pidstat命令-查看磁盘IO信息.png)

### 7.7 网络IO：ifstat 

![1614094528047](E:\SoftwareNote\Linux\images\ifstat命令-查看网络IO信息.png)

## 8.出现CPU占用过高，请谈谈你的分析思路和定位(Linux+Java命令)

结合Linux和JDK命令一块分析

### 8.1 先用top命令找出CPU占比最高的

![1614095125195](E:\SoftwareNote\Linux\images\top命令查找CPU占用最高的java进程号PID.png)

### 8.2  ps -ef或者jps进一步定位，得知是一个怎么样的一个后台程序

![1614095208808](E:\SoftwareNote\Linux\images\jps定位问题java.png)

### 8.3 定位到具体线程或者代码 ps -mp 进程PID号 -o THREAD,tid,time

-m 显示所有线程

-p pid进程使用cpu的时间

-o 该参数后是用户自定义格式

![1614095292899](E:\SoftwareNote\Linux\images\ps命令查看进程底下线程.png)

### 8.4 将需要的线程ID转换为16进制格式(英文小写格式) 

printf "%x\n"  有问题的线程TID

printf "%x\n" 24587  =》 600b

###8.5 jstack 进程ID | grep tid(16进制线程ID小写英文) -A60

-A60 ： 前60行

![1614095622600](E:\SoftwareNote\Linux\images\jstack命令-查看进程某线程运行状态.png)



