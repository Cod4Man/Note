# Linux基本知识

## 1.[FTP、SFTP、SCP、SSH、OpenSSH关系解密](https://www.cnblogs.com/h-c-g/p/11133667.html)

FTP（File Transfer  Protocol）:是TCP/IP网络上两台计算机传送文件的协议，FTP是在TCP/IP网络和INTERNET上最早使用的协议之一，它属于网络协议组的应用层。FTP客户机可以给服务器发出命令来下载文件，上载文件，创建或改变服务器上的目录。相比于HTTP，FTP协议要复杂得多。复杂的原因，是因为FTP协议要用到两个TCP连接，一个是命令链路，用来在FTP客户端与服务器之间传递命令；另一个是数据链路，用来上传或下载数据。FTP是基于TCP协议的，因此iptables防火墙设置中只需要放开指定端口（21 + PASV端口范围）的TCP协议即可。 

SFTP（Secure File Transfer Protocol）：安全文件传送协议。可以为传输文件提供一种安全的加密方法。sftp 与  ftp  有着几乎一样的语法和功能。SFTP为SSH的一部份，是一种传输文件到服务器的安全方式。在SSH软件包中，已经包含了一个叫作SFTP(Secure File Transfer  Protocol)的安全文件传输子系统，SFTP本身没有单独的守护进程，它必须使用sshd守护进程（端口号默认是22）来完成相应的连接操作，所以从某种意义上来说，SFTP并不像一个服务器程序，而更像是一个客户端程序。SFTP同样是使用加密传输认证信息和传输的数据，所以，使用SFTP是非常安全的。但是，由于这种传输方式使用了加密/解密技术，所以传输效率比普通的FTP要低得多，***如果您对网络安全性要求更高时，可以使用SFTP代替FTP***。 

SSH（Secure Shell）：，由 IETF 的网络工作小组（Network Working Group）所制定；SSH  为建立在应用层和传输层基础上的安全协议。SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH  协议可以有效防止远程管理过程中的信息泄露问题。 
SSH是由客户端和服务端的软件组成的：服务端是一个守护进程(daemon)，他在后台运行并响应来自客户端的连接请求。服务端一般是sshd进程，提供了对远程连接的处理，一般包括公共密钥认证、密钥交换、对称密钥加密和非安全连接；  客户端包含ssh程序以及像scp（远程拷贝）、slogin（远程登陆）、sftp（安全文件传输）等其他的应用程序。 
从客户端来看，SSH提供两种级别的安全验证：第一种级别（基于口令的安全验证）； 第二种级别（基于密匙的安全验证）。 
SSH 主要有三部分组成： 传输层协议 [SSH-TRANS] ；用户认证协议 [SSH-USERAUTH] ；连接协议 [SSH-CONNECT]。

OpenSSH:是SSH（Secure  SHell）协议的免费开源实现。SSH协议族可以用来进行远程控制，或在计算机之间传送文件。而实现此功能的传统方式，如telnet(终端仿真协议)、  rcp ftp、  rlogin、rsh都是极为不安全的，并且会使用明文传送密码。OpenSSH提供了服务端后台程序和客户端工具，用来加密远程控件和文件传输过程的中的数据，并由此来代替原来的类似服务。  OpenSSH是使用SSH透过计算机网络加密通讯的实现。它是取代由SSH Communications  Security所提供的商用版本的开放源代码方案。目前OpenSSH是OpenBSD的子计划。OpenSSH常常被误认以为与OpenSSL有关联，但实际上这两个计划的有不同的目的，不同的发展团队，名称相近只是因为两者有同样的软件发展目标──提供开放源代码的加密通讯软件。

## 2.忘记密码

### 2.1 CentOS7

```shell
CENTOS7
1.重启
 	显示菜单的时候按字母  "e"
2.修改启动菜单内容
 	找到ro  替换为 rw init=/sysroot/bin/sh
 	然后按Ctrl+x
3.依次输入
chroot /sysroot/
passwd root
touch /.autorelabel
4.强制重启服务器
  再输入root账号和修改后的密码即可
```

