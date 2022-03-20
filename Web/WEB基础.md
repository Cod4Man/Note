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

![1622857580163](img\Http工作原理图.png)

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

## 7. HTTP、TCP、Socket 的关系是什么？ 

- **TCP/IP 代表传输控制协议/网际协议**，指的是一系列协议族。 
- HTTP 本身就是一个协议，是从 Web 服务器传输超文本到本地浏览器的**传送协议**。 
- **Socket 是 TCP/IP 网络的 API** ，其实就是一个门面模式，<u>它把复杂的 TCP/IP 协议族隐藏在Socket 接口后面</u>。对用户来说，一组简单的接口就是全部，让 Socket 去组织数据，以符合指定的协议。 

综上所述： 

- 需要 IP 协议来连接网络 
- TCP 是一种允许我们安全传输数据的机制，使用 TCP 协议来传输数据的 HTTP 是 Web 服务器和客户端使用的特殊协议。 
- **HTTP 基于 TCP 协议，所以可以使用 Socket 去建立一个 TCP 连接**。 

## 8. TCP三次握手和四次挥手

![1623636068900](img\TCP三次握手和四次挥手图.png)

## 9. OSI 的七层模型都有哪些 

OSI七层模型一般指**开放系统互连参考模型** (Open System Interconnect 简称OSI)是国际标准化组织(ISO)和国际电报电话咨询委员会(CCITT)联合制定的开放系统互连参考模型,为开放式互连信息系统提供了一种功能结构的框架。 

**应用层**：各种应用程序协议，**比如 HTTP、HTTPS、FTP、SOCKS 安全套接字协**议、DNS 域名 系统、GDP 网关发现协议等等。 

**表示层**：加密解密、转换翻译、压缩解压缩，比如 LPP 轻量级表示协议。 

**会话层**：不同机器上的用户建立和管理会话，比如 SSL 安全套接字层协议、TLS 传输层安全协议、RPC 远程过程调用协议等等。

**传输层**：接受上一层的数据，在必要的时候对数据进行分割，并将这些数据交给网络层，保证这些数据段有效到达对端，比如 **TCP 传输控制协议、UDP 数据报协议**。 

**网络层**：控制子网的运行：逻辑编址、分组传输、路由选择，比如 IP、IPV6、SLIP 等等。 

**数据链路层**：物理寻址，同时将原始比特流转变为逻辑传输路线，比如 XTP 压缩传输协议、PPTP 点对点隧道协议等等。 

**物理层：** 机械、电子、定时接口通信信道上的原始比特流传输，比如 IEEE802.2 等等。 

## 10. Servlet不是线程安全的

servlet的service方法内部没有锁。而servlet是不同请求共用的，如果servlet中定义有共享的全局变量，那么就可能出现共享变量冲突的不安全问题。因此开发时要注意不要定义共享变量

## 11. 接口安全

### 11.1 **Token+AppKey签名验证**

与上面开发平台的验证方式类似，为客户端分配**AppKey**（密钥，用于接口加密，不参与传输），将AppKey和所有请求参数组合成源串，根据**签名算法**生成签名值，发送请求时将签名值一起发送给服务器验证。这样，即使Token被劫持，对方不知道AppKey和签名算法，就无法伪造请求和篡改参数。再结合上述的**重发攻击**解决方案，即使请求参数被劫持也无法伪造二次重复请求。

### 11.2 重放攻击

>  **重放攻击的概念：**

重放攻击(Replay Attacks)又称重播攻击、回放攻击或新鲜性攻击(Freshness Attacks)，是指攻击者发送一个目的主机已接收过的包，来达到欺骗系统的目的，主要用于身份认证过程，破坏认证的正确性。
它是一种攻击类型，这种攻击会不断恶意或欺诈性地重复一个有效的数据传输，重放攻击可以由发起者，也可以由拦截并重发该数据的敌方进行。攻击者利用网络监听或者其他方式盗取认证凭据，之后再把它重新发给认证服务器。从这个解释上理解，加密可以有效防止会话劫持，但是却防止不了重放攻击。重放攻击任何网络通讯过程中都可能发生。重放攻击是计算机世界黑客常用的攻击方式之一，它的书面定义对不了解密码学的人来说比较抽象。
重放攻击的基本原理就是把以前窃听到的数据原封不动地重新发送给接收方。很多时候，网络上传输的数据是加密过的，此时窃听者无法得到数据的准确意义。但如果他知道这些数据的作用，就可以在不知道数据内容的情况下通过再次发送这些数据达到愚弄接收端的目的。例如，有的系统会将鉴别信息进行简单加密后进行传输，这时攻击者虽然无法窃听密码，但他们却可以首先截取加密后的口令然后将其重放，从而利用这种方式进行有效的攻击。再比如，假设网上存款系统中，一条消息表示用户支取了一笔存款，攻击者完全可以多次发送这条消息而偷窃存款。

>  对于重放攻击，上面的解释中有一个词最重要，就是重复发送，解决方案就是打破重复，防御方法一般有三种（来源百度）：

加随机数。该方法优点是认证双方不需要时间同步，双方记住使用过的随机数，如发现报文中有以前使用过的随机数，就认为是重放攻击。缺点是需要额外保存使用过的随机数，若记录的时间段较长，则保存和查询的开销较大。
加时间戳。该方法优点是不用额外保存其他信息。缺点是认证双方需要准确的时间同步，同步越好，受攻击的可能性就越小。但当系统很庞大，跨越的区域较广时，要做到精确的时间同步并不是很容易。
加流水号。就是双方在报文中添加一个逐步递增的整数，只要接收到一个不连续的流水号报文(太大或太小)，就认定有重放威胁。该方法优点是不需要时间同步，保存的信息量比随机数方式小。缺点是一旦攻击者对报文解密成功，就可以获得流水号，从而每次将流水号递增欺骗认证端。
在实际中，常将方法(1)和方法(2)组合使用，这样就只需保存某个很短时间段内的所有随机数，而且时间戳的同步也不需要太精确。

>  对付重放攻击除了使用以上方法外，还可以使用挑战一应答机制和一次性口令机制，而且似乎后面两种方法在实际中使用得更广泛。

### 11.3 **CSRF** 

https://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html

**一.CSRF是什么？**

　　CSRF（Cross-site request forgery），中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF。

**二.CSRF可以做什么？**

　　你这可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账......造成的问题包括：个人隐私泄露以及财产安全。

**三.CSRF漏洞现状**

　　CSRF这种攻击方式在2000年已经被国外的安全人员提出，但在国内，直到06年才开始被关注，08年，国内外的多个大型社区和交互网站分别爆出CSRF漏洞，如：NYTimes.com（纽约时报）、Metafilter（一个大型的BLOG网站），YouTube和百度HI......而现在，互联网上的许多站点仍对此毫无防备，以至于安全业界称CSRF为“沉睡的巨人”。

**四.CSRF的原理**

　　下图简单阐述了CSRF攻击的思想

![1624201671238](img\CSRF跨域请求伪造原理图.png)

**五.CSRF的防御**

　　我总结了一下看到的资料，CSRF的防御可以从服务端和客户端两方面着手，防御效果是从服务端着手效果比较好，现在一般的CSRF防御也都在服务端进行。

　　**1.服务端进行CSRF防御**

　　服务端的CSRF方式方法很多样，但总的思想都是一致的，就是在客户端页面增加伪随机数。

　　(1).Cookie Hashing(所有表单都包含同一个伪随机值)：

　　这可能是最简单的解决方案了，因为攻击者不能获得第三方的Cookie(理论上)，所以表单中的数据也就构造失败了:>

这个方法个人觉得已经可以杜绝99%的CSRF攻击了，那还有1%呢....由于用户的Cookie很容易由于网站的XSS漏洞而被盗取，这就另外的1%。一般的攻击者看到有需要算Hash值，基本都会放弃了，某些除外，所以如果需要100%的杜绝，这个不是最好的方法。
　　(2).验证码

　　这个方案的思路是：每次的用户提交都需要用户在表单中填写一个图片上的随机字符串，厄....这个方案可以完全解决CSRF，但个人觉得在易用性方面似乎不是太好，还有听闻是验证码图片的使用涉及了一个被称为MHTML的Bug，可能在某些版本的微软IE中受影响。

　　(3).One-Time Tokens(不同的表单包含一个不同的伪随机值)

　　在实现One-Time  Tokens时，需要注意一点：就是“并行会话的兼容”。如果用户在一个站点上同时打开了两个不同的表单，CSRF保护措施不应该影响到他对任何表单的提交。考虑一下如果每次表单被装入时站点生成一个伪随机值来覆盖以前的伪随机值将会发生什么情况：用户只能成功地提交他最后打开的表单，因为所有其他的表单都含有非法的伪随机值。必须小心操作以确保CSRF保护措施不会影响选项卡式的浏览或者利用多个浏览器窗口浏览一个站点。

## 12. apache.httpcomponent

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.4.1</version>
</dependency>
```



```java
public static void main(String[] args) throws IOException {
    HttpClient httpClient = HttpClientBuilder
        .create()
        .setConnectionTimeToLive(1, TimeUnit.MINUTES) // 连接时间
        .build();
    HttpPost httpPost = new HttpPost("http://localhost:4396/redis/http");
    List<NameValuePair> nvps = new ArrayList<NameValuePair>();
    nvps.add(new BasicNameValuePair("text", "username"));
    //        nvps.add(new BasicNameValuePair("IDToken2", "password"));
    //        StringEntity myEntity = new StringEntity("{\"name\":\"zhangsan\", \"age\":20, \"gender\": \"mail\"}",
    //                ContentType.APPLICATION_JSON);// 构造请求数据
    //        httpPost.setEntity(myEntity);// 设置请求体
    httpPost.setEntity(new UrlEncodedFormEntity(nvps, Consts.UTF_8));
    HttpResponse response = httpClient.execute(httpPost);
    if (response.getStatusLine().getStatusCode() == 200) {
        HttpEntity entity = response.getEntity();
        String responseContent = EntityUtils.toString(entity, "UTF-8");
        System.out.println("responseContent: " + responseContent);
        EntityUtils.consume(entity);
    }
}
```

## 13. [WebApplicationInitializer初始化web应用，不需要web.xml](https://www.cnblogs.com/z-test/p/9341370.html)

## 14. Web三大对象：Filter / Servlet / Listener

Filter链过滤请求和响应：统一编码，过滤response，身份认证(JWT)

Servlet处理请求

Listener监听事件，比如HttpSessionListener可以监听sessionCreate和sessionDestory

拦截器Inteceptor [比如说SpringCloud的Resttemplate和Feign调用远程时，拦截请求set header token]

## 15. 跨域

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS

- 跨域流程

![image-20211007181218518](img\跨域流程1.png)

![image-20211007180938627](img\跨域流程.png)