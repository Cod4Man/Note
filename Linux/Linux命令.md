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
mkdir -p /1/2/3
```

### 2.2 重命名

```shell
例子：将目录A重命名为B

mv A B

例子：将/a目录移动到/b下，并重命名为c

mv /a /b/c
```

### 2.3 复制文件夹

```shell
# -r 表示递归复制文件夹内的所有
cp -r /1/2 /5/6

# 强制复制，覆盖相同文件
\cp -r /1/2 /5/6
```

### 2.4 ll -h (ls -lh) 格式化列表信息，如大小

```shell
[root@node1 /]# ll -h
总用量 24K
lrwxrwxrwx.   1 root root    7 7月  27 06:38 bin -> usr/bin
dr-xr-xr-x.   5 root root 4.0K 7月  27 22:22 boot
drwxr-xr-x.  20 root root 3.3K 8月   8 09:19 dev


[root@node1 /]# ll
总用量 24
lrwxrwxrwx.   1 root root    7 7月  27 06:38 bin -> usr/bin
dr-xr-xr-x.   5 root root 4096 7月  27 22:22 boot
drwxr-xr-x.  20 root root 3360 8月   8 09:19 dev
drwxr-xr-x. 144 root root 8192 8月   8 09:19 etc

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

### 4.3 查看运行中的服务 ps -ef

```shell

[root@AliyunS6 ~]# ps -ef | grep 8848
root     22225 22156  0 23:20 pts/0    00:00:00 grep --color=auto 8848
```



## 5. 搜索

### 5.1 -find

- 全盘搜索，也可以指定目录搜bai索。find 搜索目录 -name 目标名字，find / -name file（区分大小写）

- find / -iname file（不区分大小写）

- 我们先使用*通配符来匹配下包含file的相关文件区分大小写的：find / -name *file*。

- find 搜索目录 -size 文件大小。下面我们查找下大于100MB的文件,应该实际是102400KB*2,所有搜索命令为：find / -size +204800。-号是小于，直接写数字就是等于。

  单位有k/M/G

  find / -size  200M

- find 搜索目录 -user 用户名。这里是查找属于用户名为user1的文件，linux如何添加删除用户名,可以参考Linux 用户管理命令：find / -user user1。

- find 搜索目录 -type d。查找某个目录下的所有目录：find /tmp -type d。

- find 搜索目录 -cmin -时间(单位分钟)。查找etc下面1小时内被修改的文件,根目录下面太多了,指定一个目录：find /etc -cmin -60。

- 当然find命令是可以多个选项一起添加查询的：-a 是前后条件都要满足，-o 是满足一个条件就好，这样我们可以清除的看到被过滤掉的文件。

### 5.2 locate 快速查找(通过db而不是遍历)

**locate不会遍历文件树，而是读取内置文件树的db**

> 先执行updatedb更新文件树db
>
> 然后 locate xxx

### 5.3 witch 查找指令所在目录

```shell
[root@node1 yml]# which kubectl
/usr/bin/kubectl
```

### 5.4 grep 过滤查找

netstat -apn | grep 3306  ： 在netstat -apn的结果集中，过滤包含3306的

cat test.txt | grep -n -i "keyword" : 在cat test.txt的结果集中，过滤包含keyword的,并显示行号,并忽略大小写

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
5: 生产不可关闭防火墙，开放端口即可
firewall-cmd --zone=public --add-port=8080/tcp --permanent
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

## 13. 压缩和解压

tar命令

> **tar [-cxtzjvfpPN] 文件与目录 .... 参数： **

-c ：建立一个压缩文件的参数指令(create 的意思)， 压缩文件

** -x ：解开一个压缩文件的参数指令！ 

-t ：查看 tarfile 里面的文件！ 特别注意，在参数的下达中， c/x/t 仅能存在一个！不可同时存在！ 因为不可能同时压缩与解压缩。 

-z ：是否同时具有 gzip 的属性？亦即是否需要用 gzip 压缩？ 

-j ：是否同时具有 bzip2 的属性？亦即是否需要用 bzip2 压缩？ 

-v ：压缩的过程中显示文件！这个常用，但不建议用在背景执行过程！ 

-f ：使用档名，请留意，在 f 之后要立即接档名喔！不要再加参数！ 　　　例如使用『 tar -zcvfP tfile sfile』就是错误的写法，要写成 　　　『 tar -zcvPf tfile sfile』才对喔！（指定解压后的文件名）

 -p ：使用原文件的原来属性（属性不会依据使用者而变）

 -P ：可以使用绝对路径来压缩！ 

-N ：比后面接的日期(yyyy/mm/dd)还要新的才会被打包进新建的文件中！ 

--exclude FILE：在压缩的过程中，不要将 FILE 打包！**

**-C: 指定解压到某路径下**

`tar -zxvf jre-8u291-linux-x64.tar.gz -C /usr/local/jre`

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

## 30. 文本操作

### 30.1 less

分页查看文件，适合大内容的文件

### 30.2 more

先展示一部分，然后再一条一条展示

### 30.3 > 和 >> 覆盖和追加写入

> ls -l > 文件

列表的内容(就是控制台展示的list内容)写入到a.txt

> ls -al >> 文件

列表内容追加到文件中

> cat 文件1 > 文件2

文件1覆盖到文件2中

> echo "内容" >> 文件

追加“内容”到文件中

### 30.4 ln指令：软连接(快捷方式)

ln -s /root/ myroot

## 31. history 查看历史指令

history： 查看全部

history n: 查看最近n条

!n : 执行第n条历史指令

## 32. rwx权限

### 32.1 权限介绍

lrwxrwxrwx. 1 root root          8 May 11  2019  ypdomainname -> hostname
drwxr-xr-x.   3 root root   16 Jun 16 10:53 home
-rwxr-xr-x  1 root root       1983 Nov  9  2019  zcat

> **第0位为文件类型： d / - / l / c / b**

- l是链接(快捷方式)
- d是目录
- \- 是普通文件
- c是字符设备，如鼠标键盘
- b是块设备，如硬盘

> **第1-3位，确定所有者拥有的权限**

> **第4-6位，确定拥有者所属组的权限**

> **第7-9位，确定其他用户拥有的权限**

> rwx

- r表示read，只读。可用数字4代替
- w表示write，可以写入。可用数字2代替
- x表示execute，可执行。可用数字1代替
- **文件的w权限不一定能删除该文件，得看所属目录是否有w权限。文件夹x表示可以进入到该目录，r表示可以ls**

### 32.3 修改权限

- **chmod指令，配合+ / - / =来修改, u(用户) g(组) o(其他用户) a(全部)的权限**
  - chmod u=rwx,g=r,o=w text.txt
  - chmod o+w text.txt
  - chmod ax text.txt

- 通过数字，chmod 751 文件

  相当于 chmod -rwxr-x--x

- 递归修改，chmod -R 751 /home/ap/tep.txt



## 33. 任务调度

### 33.1 crond (比如定期数据库备份)

> 相关指令

-  crontab -e 编写调度指令

  ```shell
  */1 * * 5-8 * ls /root /home/rootpath.txt 
  ```

  分 时 天 月 星期

  ”*“是不作条件，“-”是范围，“，”是and

- crontab -r 终止任务调度

- crontab -l 列出当前任务调度

- service crond restart 重启

### 33.2 at定时任务，一次性

> at [命令] [时间参数]

- -m 当指定任务完成后，将给用户发送邮件，即没有标准输出

- -I atq的别名（稍微执行的任务列表）

- -d atrm的别名（删除已指定的任务）

- -v 显示任务将被执行的时间

- -c 打印任务的内容到标准输出

- -V 显示版本信息

- -q <队列> 使用指定的队列

- -f <文件> 从指定文件读入任务而不是从标准输入读入

- -t <时间参数> 以时间参数的形式提交要运行的任务

- 时间参数

  - hh:mm 当前某时刻执行，当前超过了就第二天执行

  - midnight/noon/teatime等模糊词语

  - 12小时制，am和pm

  - hh:mm mm/dd/yy  或  hh:mm dd.mm.yy

  - 相对时间 now + count + time-units

    now +1 minutes

    5:00 + 5 days

  - today / tomorrow

> Ctrl+D两次结束at命令的输入

