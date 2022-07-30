# nginx

## 1. 在linux上使用nginx

### 1.1 yum安装

`yum install nginx`

### 1.2 开启服务

`systemctl  start nginx.service`

### 1.3 开机启动

`systemctl enable nginx.service`

## 2. docker

### 2.1 进入Nignx

`docker exec -it nginx_1129 /bin/bash`

### 2.2. config文件（## 注意upstream和server同级）

https://blog.csdn.net/tjcyjd/article/details/50695922

/etc/nginx/conf.d/default.conf

```xml
## 注意upstream和server同级
## 注意upstream和server同级
## 注意upstream和server同级
upstream redlock{
	server 192.168.1.95:8111 weight=1;
	server 192.168.1.95:8222 weight=1;
}

server {
    listen       80;
    server_name localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
 

location / {
      #  root   /usr/share/nginx/html;
       # index  index.html index.htm;
        #  proxy_pass   http://nacos;
        proxy_pass http://redlock;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
  location / {
      #  root   /usr/share/nginx/html;
       # index  index.html index.htm;
        #  proxy_pass   http://nacos;
        proxy_pass http://redlock;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```

## 3. Nginx简介

### 3.1 Nginx 概述

Nginx ("engine x") 是一个高性能的 **HTTP 和反向代理**服务器,特点是**占有内存少**，**并发能 力强**，事实上 nginx 的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用 nginx 网站用户有：百度、京东、新浪、网易、腾讯、淘宝等

### 3.2 Nginx 作为 web 服务器

Nginx 可以作**为静态页面的 web 服务器**，同时还支持 CGI 协议的动态语言，比如 perl、php等。但是不支持 java。Java 程序只能通过与 tomcat 配合完成。**Nginx 专为性能优化而开发**，性能是其最重要的考量,实现上**非常注重效率 ，能经受高负载的考验**,有报告表明能支持高达 50,000 个并发连接数。

### 3.3 Nginx功能

#### 3.3.1 正向代理

就是通过代理服务器访问网站，而不是直接访问网站。如VPN访问外网等。

![1622088470391](img\Nginx正向代理图.png)

#### 3.3.2 反向代理

访问的是官网，但是该地址并不做实际的业务，而是做一个转发到不同场景的实际服务器。这样**可以隐藏实际服务器地址。**

![1622088671349](img\Nginx反向代理.png)

#### 3.3.3 负载均衡

将原先集中在一个服务器上的请求，**均衡分配**到配置相同的其他服务器上。 比如服务注册发现的Eureka/Zookeeper就是有负载均衡的作用。

![1622088886269](img\Nginx负载均衡.png)

#### 3.3.4 动静分离

为了**加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析**，加快解析速 度。降低原来单个服务器的压力。

  ![1622089055787](img\Nginx动静分离.png)

Nginx 动静分离简单来说就是把动态跟静态请求分开，不能理解成只是单纯的把动态页面和静态页面物理分离。**严格意义上说应该是*动态请求跟静态请求*分开**，可以理解成使用 Nginx 处理静态页面，Tomcat 处理动态页面。动静分离从目前实现角度来讲大致分为两种，**一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案； 另外一种方法就是动态跟静态文件混合在一起发布，通过 nginx 来分开。**

通过 location 指定不同的后缀名实现不同的请求转发。**通过 expires 参数设置，可以使浏览器缓存过期时间**，减少与服务器之前的请求和流量。具体 Expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可， 所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304，如果有修改，则直接从服务器重新下载，返回状态码 200。

```xml
## 本地文件路径 
## /data/web/静态文件(.js/.html等)
## /data/images/静态文件(.png等)
server {
    location /web/ {
        root /data/;
        index index.html;
    }
    location /images/ {
        root /data/;
        index index.html;
    }
}
```

## 4. Nginx常用命令

### 4.1 启动

`在/usr/local/nginx/sbin 目录下执行 ./nginx`

### 4.2 关闭

`在/usr/local/nginx/sbin 目录下执行 ./nginx -s stop `

### 4.3 重新加载配置（热加载）

`在/usr/local/nginx/sbin 目录下执行 ./nginx -s reload `

## 5. 配置文件解析

配置文件分为三大块：

- **全局块 **

  主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

  - worker_processes 工作线程数

    这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约
- **events块 **

  events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process

  下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个word  process 可以同时支持的最大连接数等。

  `worker_connections  1024;` 表示**每个 work process** 支持的**最大连接数**为 1024.
- **http块  **  **(http包含多个server[server包含多个location]**)

  这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。 需要注意的是：http 块也可以包括 http 全局块、server 块。

  - http块

    http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。
  - server块

    这块和虚拟主机有密切关系，虚拟主机从用户角度看，**和一台独立的硬件主机是完全一样的**，该技术的产生是**为了节省互联网服务器硬件成本**。

    每个 http 块可以包括多个 server 块，而**每个 server 块就相当于一个虚拟主机**。而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。

    1、全局 server 块

    最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。

    2、location 块

    一个 server 块可以配置多个 location 块。

    这块的**主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称 （也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配**，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

```xml
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

###   /etc/nginx/conf.d/defalut.conf  ###
upstream mall4sc{
       server 192.168.1.12:8101 weight=2;
       server 192.168.1.12:8102 weight=1;
    }

server {
    listen       80;
    server_name localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;



    location / {
      #  root   /usr/share/nginx/html;
       # index  index.html index.htm;
        #  proxy_pass   http://nacos;
        proxy_pass http://mall4sc;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

- location 指令说明

  该指令用于匹配 URL。

  语法： location [ = | ~ | ~* | ^~] uri {}

  1、= ：用于不含正则表达式的 uri 前，要求请求字符串与 uri**严格匹配**，如果匹配

  成功，就停止继续向下搜索并立即处理该请求。

  2、~：用于表示 uri**包含正则表达式，并且区分大小写**。

  3、~*：用于表示 uri 包含正则表达式，**并且不区分大小写**。

  4、^~：用于**不含正则表达式的** uri 前，要求 **Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location 块中的正则 uri 和请求字符串做匹配。**

  注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。

### 5.1 配置文件全解析

```xml
######Nginx配置文件nginx.conf中文详解#####

#定义Nginx运行的用户和用户组
user www www;

#nginx进程数，建议设置为等于CPU总核心数。
worker_processes 8;
#nginx默认是没有开启利用多核cpu的配置的。需要通过增加worker_cpu_affinity配置参数来充分利用多核cpu。
#worker_processes最多开启8个，8个以上性能提升不会再提升了，而且稳定性变得更低，所以8个进程够用了。  
#两颗CPU参数配置
#   worker_processes  2;
#   worker_cpu_affinity 0101 1010;
#四颗CPU参数配置
#   worker_processes  4;
#   worker_cpu_affinity 0001 0010 0100 1000;
#八颗CPU参数配置
#   worker_processes  8;
#   worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
#八颗CPU参数配置
#   worker_processes  8;
#   worker_cpu_affinity 0001 0010 0100 1000 0001 0010 0100 1000;

#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
error_log /usr/local/nginx/logs/error.log info;

#进程pid文件
pid /usr/local/nginx/logs/nginx.pid;

#指定进程可以打开的最大描述符：数目
#工作模式与连接数上限
#这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致。
#现在在linux 2.6内核下开启文件打开数为65535，worker_rlimit_nofile就相应应该填写65535。
#这是因为nginx调度时分配请求到进程并不是那么的均衡，所以假如填写10240，总并发量达到3-4万时就有进程可能超过10240了，这时会返回502错误。
worker_rlimit_nofile 65535;


events
{
    #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型
    #是Linux 2.6以上版本内核中的高性能网络I/O模型，linux建议epoll，如果跑在FreeBSD上面，就用kqueue模型。
    #补充说明：
    #与apache相类，nginx针对不同的操作系统，有不同的事件模型
    #A）标准事件模型
    #Select、poll属于标准事件模型，如果当前系统不存在更有效的方法，nginx会选择select或poll
    #B）高效事件模型
    #Kqueue：使用于FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0 和 MacOS X.使用双处理器的MacOS X系统使用kqueue可能会造成内核崩溃。
    #Epoll：使用于Linux内核2.6版本及以后的系统。
    #/dev/poll：使用于Solaris 7 11/99+，HP/UX 11.22+ (eventport)，IRIX 6.5.15+ 和 Tru64 UNIX 5.1A+。
    #Eventport：使用于Solaris 10。 为了防止出现内核崩溃的问题， 有必要安装安全补丁。
    use epoll;

    #单个进程最大连接数（最大连接数=连接数*进程数）
    #根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行。每个进程允许的最多连接数，理论上每台nginx服务器的最大连接数为。
    worker_connections 65535;

    #keepalive超时时间。
    keepalive_timeout 60;

    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。
    #分页大小可以用命令getconf PAGESIZE 取得。
    #[root@web001 ~]# getconf PAGESIZE
    #4096
    #但也有client_header_buffer_size超过4k的情况，但是client_header_buffer_size该值必须设置为“系统分页大小”的整倍数。
    client_header_buffer_size 4k;

    #这个将为打开文件指定缓存，默认是没有启用的，max指定缓存数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。
    open_file_cache max=65535 inactive=60s;

    #这个是指多长时间检查一次缓存的有效信息。
    #语法:open_file_cache_valid time 默认值:open_file_cache_valid 60 使用字段:http, server, location 这个指令指定了何时需要检查open_file_cache中缓存项目的有效信息.
    open_file_cache_valid 80s;

    #open_file_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive时间内一次没被使用，它将被移除。
    #语法:open_file_cache_min_uses number 默认值:open_file_cache_min_uses 1 使用字段:http, server, location  这个指令指定了在open_file_cache指令无效的参数中一定的时间范围内可以使用的最小文件数,如果使用更大的值,文件描述符在cache中总是打开状态.
    open_file_cache_min_uses 1;

    #语法:open_file_cache_errors on | off 默认值:open_file_cache_errors off 使用字段:http, server, location 这个指令指定是否在搜索一个文件是记录cache错误.
    open_file_cache_errors on;
}



#设定http服务器，利用它的反向代理功能提供负载均衡支持
http
{
    #文件扩展名与文件类型映射表
    include mime.types;

    #默认文件类型
    default_type application/octet-stream;

    #默认编码
    #charset utf-8;

    #服务器名字的hash表大小
    #保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.
    server_names_hash_bucket_size 128;

    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
    client_header_buffer_size 32k;

    #客户请求头缓冲大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。
    large_client_header_buffers 4 64k;

    #设定通过nginx上传文件的大小
    client_max_body_size 8m;

    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    #sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
    sendfile on;

    #开启目录列表访问，合适下载服务器，默认关闭。
    autoindex on;

    #此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
    tcp_nopush on;

    tcp_nodelay on;

    #长连接超时时间，单位是秒
    keepalive_timeout 120;

    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    #gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k;    #最小压缩文件大小
    gzip_buffers 4 16k;    #压缩缓冲区
    gzip_http_version 1.0;    #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2;    #压缩等级
    gzip_types text/plain application/x-javascript text/css application/xml;    #压缩类型，默认就已经包含textml，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_vary on;

    #开启限制IP连接数的时候需要使用
    #limit_zone crawler $binary_remote_addr 10m;



    #负载均衡配置
    upstream piao.jd.com {

        #upstream的负载均衡，weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。
        server 192.168.80.121:80 weight=3;
        server 192.168.80.122:80 weight=2;
        server 192.168.80.123:80 weight=3;

        #nginx的upstream目前支持4种方式的分配
        #1、轮询（默认）
        #每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
        #2、weight
        #指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
        #例如：
        #upstream bakend {
        #    server 192.168.0.14 weight=10;
        #    server 192.168.0.15 weight=10;
        #}
        #2、ip_hash
        #每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
        #例如：
        #upstream bakend {
        #    ip_hash;
        #    server 192.168.0.14:88;
        #    server 192.168.0.15:80;
        #}
        #3、fair（第三方）
        #按后端服务器的响应时间来分配请求，响应时间短的优先分配。
        #upstream backend {
        #    server server1;
        #    server server2;
        #    fair;
        #}
        #4、url_hash（第三方）
        #按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
        #例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
        #upstream backend {
        #    server squid1:3128;
        #    server squid2:3128;
        #    hash $request_uri;
        #    hash_method crc32;
        #}

        #tips:
        #upstream bakend{#定义负载均衡设备的Ip及设备状态}{
        #    ip_hash;
        #    server 127.0.0.1:9090 down;
        #    server 127.0.0.1:8080 weight=2;
        #    server 127.0.0.1:6060;
        #    server 127.0.0.1:7070 backup;
        #}
        #在需要使用负载均衡的server中增加 proxy_pass http://bakend/;

        #每个设备的状态设置为:
        #1.down表示单前的server暂时不参与负载
        #2.weight为weight越大，负载的权重就越大。
        #3.max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误
        #4.fail_timeout:max_fails次失败后，暂停的时间。
        #5.backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

        #nginx支持同时设置多组的负载均衡，用来给不用的server来使用。
        #client_body_in_file_only设置为On 可以讲client post过来的数据记录到文件中用来做debug
        #client_body_temp_path设置记录文件的目录 可以设置最多3层目录
        #location对URL进行匹配.可以进行重定向或者进行新的代理 负载均衡
    }



    #虚拟主机的配置
    server
    {
        #监听端口
        listen 80;

        #域名可以有多个，用空格隔开
        server_name www.jd.com jd.com;
        index index.html index.htm index.php;
        root /data/www/jd;

        #fastcgi解析php
        location ~ .*.(php|php5)?$
        {
#此处有两种方式去和php-fpm交互,一种是9000端口,另一种是使用socket连接
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
        }

        #图片缓存时间设置
        location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires 10d;
        }

        #JS和CSS缓存时间设置
        location ~ .*.(js|css)?$
        {
            expires 1h;
        }

        #日志格式设定
        #$remote_addr与$http_x_forwarded_for用以记录客户端的ip地址；
        #$remote_user：用来记录客户端用户名称；
        #$time_local： 用来记录访问时间与时区；
        #$request： 用来记录请求的url与http协议；
        #$status： 用来记录请求状态；成功是200，
        #$body_bytes_sent ：记录发送给客户端文件主体内容大小；
        #$http_referer：用来记录从那个页面链接访问过来的；
        #$http_user_agent：记录客户浏览器的相关信息；
        #通常web服务器放在反向代理的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。反向代理服务器在转发请求的http头信息中，可以增加x_forwarded_for信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址。
        log_format access '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" $http_x_forwarded_for';

        #定义本虚拟主机的访问日志
        access_log  /usr/local/nginx/logs/host.access.log  main;
        access_log  /usr/local/nginx/logs/host.access.404.log  log404;

        #对 "/" 启用反向代理
        location / {
            proxy_pass http://127.0.0.1:88;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;

            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            #以下是一些反向代理的配置，可选。
            proxy_set_header Host $host;

            #允许客户端请求的最大单文件字节数
            client_max_body_size 10m;

            #缓冲区代理缓冲用户端请求的最大字节数，
            #如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。如果注释该指令，使用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
            #无论使用firefox4.0还是IE8.0，提交一个比较大，200k左右的图片，都返回500 Internal Server Error错误
            client_body_buffer_size 128k;

            #表示使nginx阻止HTTP应答代码为400或者更高的应答。
            proxy_intercept_errors on;

            #后端服务器连接的超时时间_发起握手等候响应超时时间
            #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_connect_timeout 5;

            #后端服务器数据回传时间(代理发送超时)
            #后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
            proxy_send_timeout 5;

            #连接成功后，后端服务器响应时间(代理接收超时)
            #连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
            proxy_read_timeout 5;

            #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            #设置从被代理服务器读取的第一部分应答的缓冲区大小，通常情况下这部分应答中包含一个小的应答头，默认情况下这个值的大小为指令proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小
            proxy_buffer_size 4k;

            #proxy_buffers缓冲区，网页平均在32k以下的设置
            #设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k
            proxy_buffers 4 32k;

            #高负荷下缓冲大小（proxy_buffers*2）
            proxy_busy_buffers_size 64k;

            #设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
            #设定缓存文件夹大小，大于这个值，将从upstream服务器传
            proxy_temp_file_write_size 64k;
        }


        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status on;
            access_log on;
            auth_basic "NginxStatus";
            auth_basic_user_file confpasswd;
            #htpasswd文件的内容可以用apache提供的htpasswd工具来产生。
        }

        #本地动静分离反向代理配置
        #所有jsp的页面均交由tomcat或resin处理
        location ~ .(jsp|jspx|do)?$ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:8080;
        }

        #所有静态文件由nginx直接读取不经过tomcat或resin
        location ~ .*.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|
        pdf|xls|mp3|wma)$
        {
            expires 15d; 
        }

        location ~ .*.(js|css)?$
        {
            expires 1h;
        }
    }
}
######Nginx配置文件nginx.conf中文详解#####
```

### 5.2 localtion

#### 5.2.1 root 和 alias

```nginx
location ~ ^/weblogs/ {
 	root /www.ttlsa.com;
	# /weblogs/hello.html =>  /www.ttlsa.com/weblogs/hello.html
}

location ~ ^/weblogs/ {
 	alias /www.ttlsa.com;
	# /weblogs/hello.html =>  /www.ttlsa.com/hello.html
}
```

1、root会直接把location后面配置路径附加到指定目录之后
2、alias会把location后面配置的路径丢弃掉，把当前匹配到的目录指向到指定的目录
3、使用alias时，目录名后面一定要加"/"
4、alias只能位于location块中。（root可以不放在location中）

#### 5.2.2 proxy_pass

```nginx
nginx配置proxy_pass代理转发

假设下面四种情况分别用 http://192.168.1.1/proxy/test.html 进行访问。

第一种：location /proxy/ { proxy_pass http://127.0.0.1/; }

代理到URL：http://127.0.0.1/test.html

第二种(相对于第一种，最后少一个 / )location /proxy/ { proxy_pass http://127.0.0.1; }

代理到URL：http://127.0.0.1/proxy/test.html

第三种：location /proxy/ { proxy_pass http://127.0.0.1/aaa/; }

代理到URL：http://127.0.0.1/aaa/test.html

第四种(相对于第三种，最后少一个 / )location /proxy/ { proxy_pass http://127.0.0.1/aaa; }

代理到URL：http://127.0.0.1/aaatest.html
```

## 6. Nginx 提供了几种均衡分配方式

### 6.1 轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。

### 6.2 weight

weight 代表权,重默认为 1,权重越高被分配的客户端越多。指定轮询几率，weight 和访问比率成正比，用于后端服务器性能不均的情况。 例如： `server 192.168.5.21 weight=10; `

### 6.3  ip_hash

每个请求**按访问 ip(客户端ip) 的 hash 结果分配**，**这样每个访客固定访问一个后端服务器，可以解决 session 的问题。**

```xml
upstream server_pool{  
    ip_hash;  
    server 192.168.5.21:80;  
    server 192.168.5.22:80;  
}

```

### 6.4 fair（第三方）

按**后端服务器的响应时间**来分配请求，响应时间短的优先分配。

```xml
upstream server_pool{  
    server 192.168.5.21:80;  
    server 192.168.5.22:80;  
    fair;
}
```

## 7. Nginx原型和参数设置

### 7.1 master和worker原型

![1622091975567](img\Nginx的master和worker原型.png)

![1622092011640](img\Nginx的master和worker原型2.png)

**master-workers 的机制的好处**

首先，对于**每个 worker 进程来说，独立的进程，不需要加锁，所以省掉了锁带来的开销**，同时在编程以及问题查找时，也会方便很多。其次，**采用独立的进程，可以让互相之间不会影响**，一个进程退出后，其它进程还在工作，服务不会中断，master 进程则很快启动新的worker 进程。当然，worker 进程的异常退出，肯定是程序有 bug 了，异常退出，会导致当前 worker 上的所有请求失败，不过不会影响到所有请求，所以降低了风险。

### 7.2 需要设置多少个worker

**Nginx 同 redis 类似都采用了 io 多路复用机制**，每个 worker 都是一个独立的进程，但每个进程里只有一个主线程，通过异步非阻塞的方式来处理请求， 即使是千上万个请求也不在话下。每个 worker 的线程可以把一个 cpu 的性能发挥到极致。**所以 worker 数和服务器的 cpu数相等是最为适宜**的。设少了会浪费 cpu，设多了会造成 cpu 频繁切换上下文带来的损耗。

- **worker配置**

nginx默认是没有开启利用多核cpu的配置的。需要通过增加**worker_cpu_affinity**配置参数来充分利用多核cpu。

**worker_processes最多开启8个，8个以上性能提升不会再提升了**，而且**稳定性变得更低**，所以8个进程够用了。

> 两颗CPU参数配置

   worker_processes  2;

   worker_cpu_affinity 0101 1010;

> 四颗CPU参数配置

   worker_processes  4;

   worker_cpu_affinity 0001 0010 0100 1000;

> 八颗CPU参数配置

   worker_processes  8;

   worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;

> 八颗CPU参数配置

   worker_processes  8;

   worker_cpu_affinity 0001 0010 0100 1000 0001 0010 0100 1000;

### 7.3 连接数 worker_connection

**静态代理：每次请求需要2个连接数(请求nginx，nginx响应)**

**反向代理：每次请求需要4个连接数（请求nginx，nginx请求tomcat，tomcat响应nginx，nginx响应）**

nginx最大连接数公式： **一个nginx的最大连接数=worker个数(worker_processes) * 每个worker的最大连接数(worker_connections)**

**静态代理最大并发数**： worker_connections * worker_processes /2

**反向代理最大并发数**： worker_connections * worker_processes /4

## 8. Keepalived+ Nginx搭建高可用Nginx

## 9. Nginx+多Nginx搭建集群
