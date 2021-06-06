# WEB基础

## 1.Cooking和Session的区别 （WEB基础要复习复习！）

### 1.1 什么是cooking

cooking是保存在本地(浏览器/客户端)的一些数据，请求时会一同发送到服务端

### 1.2 什么是session

session是**服务端和客户端的一次会话**，有过期时间/重开浏览器就过期，用户不同页面之间的数据共享

### 1.3 Cooking和session的区别：

①. **存储位置**：Cooking存储在客户端，Session存储在服务端
②. **存储长度**：Cooing存储的长度较少 4K
③. **存储类型**：Cooking只能存ASCII，Session可以存任意类型数据
④. **有效期**：Cooking可以长期保存在客户端(网站密码保存)，Session一次会话时间（过期或者浏览器关闭）
⑤. **安全性**：Cooking存储在客户端比较不安全

###1.4 Cooking和session关联：

通过sessionID：session会有个sessionID，cooking会缓存sessionID，访问时**通过sessionID**找到session。如果找不到sessionid，服务器端就创建session，生成sessionid对应的cookie，写入到响应头中。
cooking被禁用，可以使用token

>  分布式session(负载均衡到不到服务器，session是不同的)：

① Ngixn 负载均衡有ip_hash策略，可以把相同ip的请求分配到同一个服务器，这样就不会session不同步
② session复制，session发生改变就广播给其他服务器做session同步
③ (**推荐**)共享session将session信息托给中间件管理(redis，key通过sessionID来查找redis中的session对象)

### 1.5 前端跨域问题：

**浏览器的同源策略(为了安全采取的，同ip同port才能访问）**

① 通过代理来避免，Nginx代理服务器，前端只和nginx交互，不会有跨域问题
② jsonp
JSONP 的理念就是，与服务端约定好一个回调函数名，服务端接收到请求后，将返回一段 Javascript，在这段 Javascript 代码中调用了约定好的回调函数，并且将数据作为参数进行传递。当网页接收到这段 Javascript 代码后，就会执行这个回调函数，这时数据已经成功传输到客户端了。

JSONP 的缺点是：它只支持 GET 请求，而不支持 POST 请求等其他类型的 HTTP 请求。



## 2. RESTful

url规范

1. REST描述的是在网络中client和server的一种交互形式；REST本身不实用，实用的是如何设计 RESTful API（REST风格的网络接口）；

\2. Server提供的RESTful API中，**URL中只使用名词来指定资源，原则上不使用动词**。“资源”是REST架构或者说整个网络处理的核心。比如：
[http://api.qc.com/v1/newsfeed](https://link.zhihu.com/?target=http%3A//api.qc.com/v1/newsfeed): 获取某人的新鲜; 
[http://api.qc.com/v1/friends](https://link.zhihu.com/?target=http%3A//api.qc.com/v1/friends): 获取某人的好友列表;
[http://api.qc.com/v1/profile](https://link.zhihu.com/?target=http%3A//api.qc.com/v1/profile): 获取某人的详细信息;

3. **用HTTP协议里的动词来实现资源的添加，修改，删除等操作。即通过HTTP动词来实现资源的状态扭转**：

GET    用来获取资源，
POST  用来新建资源（也可以用于更新资源），
PUT    用来更新资源，
DELETE  用来删除资源。比如：
DELETE [http://api.qc.com/v1/](https://link.zhihu.com/?target=http%3A//api.qc.com/v1/friends)friends: 删除某人的好友 （在http parameter指定好友id）
POST [http://api.qc.com/v1/](https://link.zhihu.com/?target=http%3A//api.qc.com/v1/friends)friends: 添加好友
UPDATE [http://api.qc.com/v1/profile](https://link.zhihu.com/?target=http%3A//api.qc.com/v1/profile): 更新个人资料

## 3. Web概念

>  1）. 软件架构 

\1. C/S： 客户端/服务器端 ‐‐‐‐‐‐‐‐‐‐‐‐> QQ , 360 .... 

\2. B/S： 浏览器/服务器端 ‐‐‐‐‐‐‐‐‐‐‐‐> 京东， 网易 ， 淘宝 ， 传智播客 

官网

>  2）. 资源分类 

\1. 静态资源： 所有用户访问后，得到的结果都是一样的，称为静态资源。静态资源可以直接被浏览器解析。 

\ 如： **html**,css,JavaScript，jpg 

\2. 动态资源: 每个用户访问相同资源后，得到的结果可能不一样 , 称为动态资源。动态资源被访问后，需要先转换为静态资源，再返回给浏览器，通过浏览器进行解析。 

\ 如：**servlet/jsp**,php,asp.... 

>  3）. 网络通信三要素 

\1. IP：电子设备(计算机)在网络中的唯一标识。 

\2. 端口：应用程序在计算机中的唯一标识。 0~65536  (2^16)

\3. 传输协议：规定了数据传输的规则 

\1. 基础协议： 

\1. tcp : 安全协议，**三次握手**。 速度稍慢 

\2. udp：不安全协议。 速度快 

## 5. 常见Web服务器

1). **webLogic**：oracle公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。 

2). **webSphere**：IBM公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。 

3). **JBOSS**：JBOSS公司的，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。 

4). **Tomcat**：Apache基金组织，中小型JavaEE服务器，**仅仅支持少量的JavaEE规范servlet/jsp**。开源免费。 

## 6. Http 工作原理 

HTTP协议是浏览器与服务器之间的**数据传送协议**。<u>作为应用层协议</u>，**HTTP是基于TCP/IP协议**来传递数据的（HTML文件、图片、查询结果等），HTTP协议不涉及数据包（Packet）传输，主要规定了客户端和服务器之间的通信格式。 

![1622857580163](E:\SoftwareNote\Web\img\Http工作原理图.png)

1） 用户通过浏览器进行了一个操作，比如输入网址并回车，或者是点击链接，接着浏览器获取了这个事件。 

2） 浏览器向服务端发出TCP连接请求。 

3） 服务程序接受浏览器的连接请求，并经过**TCP三次握手建立连接**。 

4） 浏览器将请求数据**打包成一个<u>HTTP协议格式</u>的数据包**。 

5） 浏览器将该数据包推入网络，数据包经过网络传输，最终达到端服务程序。 

6） **服务端程序拿到这个数据包后，<u>同样以HTTP协议格式解包</u>**，获取到客户端的意图。 

7） 得知客户端意图后进行处理，比如提供静态文件或者调用服务端程序获得动态结果。 

8） 服务器将响应结果（可能是HTML或者图片等）按照HTTP协议格式打包。 

9） 服务器将响应数据包推入网络，数据包经过网络传输最终达到到浏览器。 

10） 浏览器拿到数据包后，以HTTP协议的格式解包，然后解析数据，假设这里的数据是HTML(服务器拿到的是是静态资源)。 

11） 浏览器将HTML文件展示在页面上。 

