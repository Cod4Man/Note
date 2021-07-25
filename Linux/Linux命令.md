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

万金油：　netstat -tnlp | grep  端口号/进程号/进程名称
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

  https://www.cnblogs.com/weijiangbao/p/7653475.html

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



## 9. 文本操作 vim

### 9.1 查找

- 向下搜索：  `/关键字`      ， n为下一处
- 向上搜索：  `?关键字`      ， n为上一处

### 9.2 替换

- 当前行出现的第一个

  `:s/源单词/新单词/`

- 当前行所有匹配

  `:s/源单词/新单词/g`

- 文本中所有匹配

  `:%s/源单词/新单词/g`

- 每次替换前要求确认

  `:%s/源单词/新单词/gc`

### 9.3  文本位置跳转

- 跳转文本顶部（第一行）

  `gg`

- 跳转文本底部（最后一行）

  `G`

### 9.4 保存退出

- 退出

  `:q`

- 强制退出

  `:q!`

- 保存并退出

  `:wq`

### 9.5 删除

- 删除光标往后

  `D`

- 删除本行

  `dd`

- 删除多行

  `dnd n表示从光标开始的总行数 `

### 9.6 插入

- 插入： i
- 下一行插入: o (就像Shift+Enter)

### 9.7 全选/全复制/全删除

**全选（高亮显示**）：按esc后，然后ggvG或者ggVG

**全部复制：**按esc后，然后ggyG

**全部删除：**按esc后，然后dG

解析：
gg ：是让光标移到首行，在vim才有效，vi中无效。
v ：是进入Visual（可视）模式。
G ：光标移到最后一行。

选中内容以后就可以其他的操作了，比如：
d ：删除选中内容。
y ：复制选中内容到0号寄存器。
“+y ：复制选中内容到＋寄存器，也就是系统的剪贴板，供其他程序用。

### 9.8 行号

:set number

:set nu

取消行号： :set nonu

### 9.9 复制粘贴

yy: 复制当前行

nyy：复制当前行开始的n行

p: 粘贴

### 9.10 撤销

u

### 9.11 行号跳转

nG/ngg  跳转到第n行

## 10. ntsysv 开启服务开机启动界面

用“空格”选中，“tab”切换，“enter”确认

## 11.  服务启动相关 

![1621992961217](E:\SoftwareNote\Linux\images\服务启动相关1.png)

![1621992940497](E:\SoftwareNote\Linux\images\服务启动相关2.png)



![1621993001443](E:\SoftwareNote\Linux\images\服务启动相关3.png)

## 12. 跨服务器文件拷贝

```shell
scp -r sonarqube-7.3/ root@192.168.0.211:/opt/soft/
```

## 13. 解压

tar命令

**tar [-cxtzjvfpPN] 文件与目录 .... 参数： **

-c ：建立一个压缩文件的参数指令(create 的意思)；解压到指定文件夹

`tar -zxvf jre-8u291-linux-x64.tar.gz -C /usr/local/jre`

** -x ：解开一个压缩文件的参数指令！ 

-t ：查看 tarfile 里面的文件！ 特别注意，在参数的下达中， c/x/t 仅能存在一个！不可同时存在！ 因为不可能同时压缩与解压缩。 

-z ：是否同时具有 gzip 的属性？亦即是否需要用 gzip 压缩？ 

-j ：是否同时具有 bzip2 的属性？亦即是否需要用 bzip2 压缩？ 

-v ：压缩的过程中显示文件！这个常用，但不建议用在背景执行过程！ 

-f ：使用档名，请留意，在 f 之后要立即接档名喔！不要再加参数！ 　　　例如使用『 tar -zcvfP tfile sfile』就是错误的写法，要写成 　　　『 tar -zcvPf tfile sfile』才对喔！

 -p ：使用原文件的原来属性（属性不会依据使用者而变）

 -P ：可以使用绝对路径来压缩！ 

-N ：比后面接的日期(yyyy/mm/dd)还要新的才会被打包进新建的文件中！ 

--exclude FILE：在压缩的过程中，不要将 FILE 打包！**



## 14. java后台运行

`nohup java -jar babyshark-0.0.1-SNAPSHOT.jar  > log.file  2>&1 &`

上面的2 和 1 的意思如下:

0    标准输入（一般是键盘）
 1    标准输出（一般是显示屏，是用户终端控制台）
 2    标准错误（错误信息输出）

## 15. 服务重启以及状态查看

systemctl restart tomcat

systemctl start tomcat

systemctl status tomcat

## 16.  Linux下JRE环境变量配置              

很多时候，我们需要在Linux上部署tomcat，从而搭建web服务器，然JDK/JRE环境是前提，这里就记录一下，在后面的时候直接使用。

下载jre-7u80-linux-x64.tar.gz，并解压，我所解压的路径为：/usr/local/jre1.7.0_80

在Linux的环境变量文件/etc/profile文件中添加如下语句：

```
#set java environment
export JAVA_HOME=/usr/local/jre/jre1.8.0_291

export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export PATH=$JAVA_HOME/bin:$PATH
```

要使环境变量生效，执行如下即可。

source /etc/profile

## 17. 集中文件查找命令 which whereis locate find

![1623847559383](E:\SoftwareNote\Linux\images\文件查找命令对比.png)

## 28. ps命令

### 28.1 通过ps指令查找最占cpu的子<u>线程(线程不是进程)</u>  

ps -mp pid -o THREAD,tid,time

```shell
[root@AliyunS6 ~]# ps -mp 7039 -o THREAD,tid,time
USER     %CPU PRI SCNT WCHAN  USER SYSTEM   TID(线程号)     TIME
root      0.4   -    - -         -      -     - 00:00:10
root      0.0  19    - -         -      -  7039 00:00:00
root      0.1  19    - -         -      -  7040 00:00:03
root      0.0  19    - -         -      -  7041 00:00:00
root      0.0  19    - -         -      -  7042 00:00:00
root      0.0  19    - -         -      -  7043 00:00:00
root      0.0  19    - -         -      -  7044 00:00:00
root      0.0  19    - -         -      -  7045 00:00:00
root      0.0  19    - -         -      -  7046 00:00:00

```

## 29. 系统相关命令

### 29.1 查看服务systemd日志：

```shell
journalctl -xefu kubelet

Jul 25 22:17:35 node1 kubelet[25249]: E0725 22:17:35.698639   25249 kubelet.go:2291] "Error getting node" err="node \"node1\" not found"
```

### 29.2 查看服务状态

```shell
systemctl status kubelet -l

kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Sun 2021-07-25 22:15:20 CST; 4min 7s ago
     Docs: https://kubernetes.io/docs/
 Main PID: 25249 (kubelet)
    Tasks: 18
   CGroup: /system.slice/kubelet.service
           └─25249 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.4.1 --fail-swap-on=false

Jul 25 22:19:26 node1 kubelet[25249]: E0725 22:19:26.677927   25249 kubelet.go:2291] "Error getting node" err="node \"node1\" not found"

```



