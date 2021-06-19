# Tomcat

## 1. Tomcat和Weblogic的区别

https://blog.csdn.net/ththcc/article/details/78474476

> 功能上：WebLogic相对Tomcat，主要多支持了EJB

WebLogic应该是J2EE Container(Web Container + EJB Container + XXX规范)！ 

Tomcat只能算Web Container （JSP+Servlet）

> 扩展性：WebLogic的集群高可用和负载均衡

WebLogic Server凭借其出色的群集技术，拥有处理关键Web应用系统问题所需的性能、可扩展性和高可用性。 

WebLogic Server既实现了网页群集，也实现了EJB组件 群集，而且不需要任何专门的硬件或操作系统支持。网页群集可以实现透明的复制、负载平衡以及表示内容容错 。    无论是网页群集，还是组件群集，对于电子商务解决方案所要求的可扩展性和可用性都是至关重要的。共享的客户机/服务器和数据库连接以及数据缓存和EJB都增强了性能表现。这是其它Web应用系统所不具备的    所以，在扩展性方面WebLogic是远远超越了Tomcat。 

> 费用： Weblogic不开源不免费

> EJB 

被称为Java企业bean，服务器端组件，核心应用是部署分布式应用程序。用它部署的系统不限定平台。实际上ejb是一种产品，描述了应用组件要解决的标准  

用通俗话说，EJB就是："把你编写的软件中那些需要执行制定的任务的类，不放到客户端软件上了，而是给他打成包放到一个服务器上了"。是的，没错！EJB 就是将那些"类"放到一个服务器上，用C/S 形式的软件客户端对服务器上的"类"进行调用。 

EJB容器中有三种类也称为组件，分别是会话Bean（Session Bean），实体Bean（Entity Bean）和消息驱动Bean（MessageDriven Bean）  Session bean(逻辑)  EntityBean(数据)  messageDrivenbean（消息） 



## 2. Tomcat 历史 

1） Tomcat 最初由Sun公司的软件架构师 James Duncan Davidson 开发，名称为 “JavaWebServer”。 

2） 1999年 ，在 Davidson 的帮助下，该项目于1999年于apache 软件基金会旗下的JServ 项目合并，并发布第一个版本（3.x）, 即是现在的Tomcat，该版本实现了Servlet2.2 和 JSP 1.1 规范 。 

3） 2001年，Tomcat 发布了**4.0版本， 作为里程碑式的版本，Tomcat 完全重新设计了其架构**，并实现了 Servlet 2.3 和 JSP1.2规范。 

目前 Tomcat 已经更新到 9.0.x版本 ， 但是目前企业中的Tomcat服务器， 主流版本还是7.x 和 8.x 

## 3. 目录结构

|   目录   |         目录下文件          |                             说明                             |
| :------: | :-------------------------: | :----------------------------------------------------------: |
|   bin    |              /              |            存放Tomcat的启动、停止等批处理脚本文件            |
|          |  startup.bat , startup.sh   |               用于在windows和linux下的启动脚本               |
|          | shutdown.bat ,  shutdown.sh |               用于在windows和linux下的停止脚本               |
| **conf** |              /              |                 用于存放Tomcat的相关配置文件                 |
|          |          Catalina           |             用于存储针对每个虚拟机的Context配置              |
|          |         context.xml         | 用于定义所有web应用均需加载的Context配 置，**如果web应用指定了自己的context.xml，该文件将被覆盖** |
|          |     catalina.properties     |                    Tomcat 的环境变量配置                     |
|          |       catalina.policy       |                  Tomcat 运行的安全策略配置                   |
|          |     logging.properties      | Tomcat 的日志配置文件， 可以通过该文件修改Tomcat 的日志级别及日志路径等 |
|          |       **server.xml**        |                 Tomcat 服务器的核心配置文件                  |
|          |      tomcat-users.xml       |          **定义Tomcat默认的用户及角色映射信息配置**          |
|          |         **web.xml**         | Tomcat 中所有应用默认的部署描述文件， 主要定义了基础Servlet和MIME映射。 |
|   lib    |              /              |                    Tomcat 服务器的依赖包                     |
|   logs   |              /              |                  Tomcat 默认的日志存放目录                   |
| webapps  |              /              |                 Tomcat 默认的Web应用部署目录                 |
| **work** |              /              |           **Web 应用JSP代码生成和编译的临时目录**            |

## 4. 跑Tomcat源码

### 4.1 下载源码

tomcat-src.zip

### 4.2 增加pom依赖

```xml
<?xml version="1.0" encoding="UTF‐8"?> <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema‐instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven‐4.0.0.xsd"> <modelVersion>4.0.0</modelVersion> <groupId>org.apache.tomcat</groupId> <artifactId>apache‐tomcat‐8.5.42‐src</artifactId> <name>Tomcat8.5</name> <version>8.5</version> <build> <finalName>Tomcat8.5</finalName> <sourceDirectory>java</sourceDirectory> <!‐‐ <testSourceDirectory>test</testSourceDirectory>‐‐> <resources> <resource> <directory>java</directory> </resource> </resources> <!‐‐ <testResources> <testResource> <directory>test</directory> </testResource> </testResources>‐‐> <plugins> <plugin> <groupId>org.apache.maven.plugins</groupId> <artifactId>maven‐compiler‐plugin</artifactId> <version>2.3</version> <configuration> <encoding>UTF‐8</encoding> <source>1.8</source> <target>1.8</target> </configuration> </plugin> </plugins> </build><dependencies> <dependency> <groupId>junit</groupId> <artifactId>junit</artifactId> <version>4.12</version> <scope>test</scope> </dependency> <dependency> <groupId>org.easymock</groupId> <artifactId>easymock</artifactId> <version>3.4</version> </dependency> <dependency> <groupId>ant</groupId> <artifactId>ant</artifactId> <version>1.7.0</version> </dependency> <dependency> <groupId>wsdl4j</groupId> <artifactId>wsdl4j</artifactId> <version>1.6.2</version> </dependency> <dependency> <groupId>javax.xml</groupId> <artifactId>jaxrpc</artifactId> <version>1.1</version> </dependency> <dependency> <groupId>org.eclipse.jdt.core.compiler</groupId> <artifactId>ecj</artifactId> <version>4.5.1</version> </dependency> </dependencies> </project>
```

### 4.3 增加启动VM参数

```shell
‐Dcatalina.home=D:/idea‐workspace/itcast_project_tomcat/apache‐tomcat‐ 8.5.42‐src/home ‐Dcatalina.base=D:/idea‐workspace/itcast_project_tomcat/apache‐tomcat‐ 8.5.42‐src/home ‐Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager ‐Djava.util.logging.config.file=D:/idea‐ workspace/itcast_project_tomcat/apache‐tomcat‐8.5.42‐ src/home/conf/logging.properties
```

### 4.4 配置JSP解析器 JasperInitializer

出现上述异常的原因，是我们直接启动org.apache.catalina.startup.Bootstrap的时候没有加载JasperInitializer，从而无法编译JSP。解决办法是在tomcat的源码ContextConfig中的configureStart函数中手动将JSP解析器初始化：

```java
context.addServletContainerInitializer(new JasperInitializer(), null);
```

## 5. Tomcat架构

### 5.1 servlet处理http请求

加一层servelt和Http请求响应交互，可以解耦业务逻辑直接交互http

![1622857848331](E:\SoftwareNote\Web\img\Servelt处理Http请求.png)

### 5.2 Servlet容器工作流程 

当客户请求某个资源时，HTTP服务器会用一个**ServletRequest**对象把客户的请求信息封装起来，然后调用Servlet容器的service方法，Servlet容器拿到请求后，**根据请求的URL和Servlet的映射关系，找到相应的Servlet**，如果Servlet还没有被加载，就用反射机制创建这个Servlet，并调用Servlet的init方法来完成初始化，接着**调用Servlet的<u>service方法</u> 来处理请求，把ServletResponse对象返回给HTTP服务器**，HTTP服务器会把响应发送给客户端。

### 5.3 Tomcat整体架构  

了Tomcat要实现两个核心功能： 

1） **处理Socket连接，负责网络字节流**与Request和Response对象的转化。 

2） 加载和管理**Servlet**，以及具体处理Request请求。 

因此Tomcat设计了两个核心组件**<u>连接器（Connector）和容器（Container</u>）**来分别做这两件事情。**连接器负责对外交流，容器负责内部处理。**

![1622858459536](E:\SoftwareNote\Web\img\Tomcat两大核心连接器Connector和容器Container.png)

### 5.4 Coyote 连接器

#### 5.4.1 Coyote介绍

Coyote 是Tomcat的连接器框架的名称 , 是**Tomcat服务器提供的供客户端访问的外部接口**。客户端通过Coyote与服务器建立连接、发送请求并接受响应 。 

**Coyote 封装了底层的网络通信（<u>Socket 请求及响应处理</u>）**，为Catalina 容器提供了统一的接口，**使Catalina 容器与具体的请求协议及IO操作方式完全解耦**。Coyote 将Socket 输入转换封装为 Request 对象，交由Catalina 容器进行处理，处理请求完成后, Catalina 通过Coyote 提供的Response 对象将结果写入输出流 。 

**Coyote 作为独立的模块，只负责具体协议和IO的相关操作**， 与Servlet 规范实现没有直接关系，因此即便是 Request 和 Response 对象也并未实现Servlet规范对应的接口， 而是在Catalina 中将他们进一步封装为ServletRequest 和 ServletResponse 。

![1622858992659](E:\SoftwareNote\Web\img\Coyote和Catalina交互过程.png)

#### 5.4.2 IO模型与协议  

在Coyote中 ， Tomcat支持的多种I/O模型和应用层协议，具体包含哪些IO模型和应用层协议，请看下表： 

Tomcat 支持的IO模型（自8.5/9.0 版本起，Tomcat 移除了 对 BIO 的支持）：

- IO模型

| IO模型  |                             描述                             |
| :-----: | :----------------------------------------------------------: |
| **NIO** |              非阻塞I/O，采用Java NIO类库实现。               |
|  NIO2   |            异步I/O，采用JDK 7最新的NIO2类库实现。            |
|   APR   | 采用Apache可移植运行库实现，是C/C++编写的本地库。如果选择该方案，**需要单独安装APR库** |

**在 8.0 之前 ， Tomcat 默认采用的I/O方式为 BIO ， 之后改为 NIO**。 无论 NIO、NIO2还是 APR， 在性能方面均优于以往的BIO。 如果采用APR， 甚至可以达到 Apache HTTP Server 的影响性能。

- Tomcat 支持的应用层协议 

| 应用层协议 |                             描述                             |
| :--------: | :----------------------------------------------------------: |
|  HTTP/1.1  |              这是大部分Web应用采用的访问协议。               |
|    AJP     | 用于和Web服务器集成（如Apache），以实现对静态资源的优化以及 集群部署，当前支持AJP/1.3。 |
|   HTTP/2   | HTTP 2.0大幅度的提升了Web性能。下一代HTTP协议 ， 自8.5以及9.0 版本之后支持。 |

- 协议分层

  ![1622859510490](E:\SoftwareNote\Web\img\Tomcat协议分层.png)

Tomcat为了实现支持多种I/O模型和应用层协议，一个容器可能对接多个连接器，就好比一个房间有多个门。但是单独的连接器或者容器都不能对外提供服务，需要把它们组装起来才能工作，组装后这个整体叫作Service组件。这里请你注意，Service本身没有做什么重要的事情，只是在连接器和容器外面多包了一层，把它们组装在一起。Tomcat内可能有多个Service，这样的设计也是出于灵活性的考虑。**通过在Tomcat中配置多个Service，可以实现通过不同的端口号来访问同一台机器上部署的不同应用。**

#### 5.4.3  连接器组件

![1622859676342](E:\SoftwareNote\Web\img\Tomcat连接器组件.png)

①EndPoint负责和客户端对接，接受客户端socket数据

②processor负责和EndPoint对接，将Socket数据转换为Request对象

③adapter和processor对接，将http的request转换为tomcat的request，然后调用httpservelt.service

##### 5.4.3.1 EndPoint 

1） EndPoint ： Coyote 通信端点，即**通信监听的接口**，是具体Socket接收和发送处理器，是对传输层的抽象，因此**EndPoint用来实现TCP/IP协议的**。

2） Tomcat 并没有EndPoint 接口，而是提供了一个抽象类AbstractEndpoint ， 里面定义了两个内部类：**Acceptor和SocketProcessor**。**Acceptor用于监听Socket连接请求。SocketProcessor用于处理接收到的Socket请求**，它实现Runnable接口，在Run方法里调用协议处理组件Processor进行处理。为了提高处理能力，SocketProcessor被提交到线程池来执行。而这个线程池叫作执行器（Executor)

##### 5.4.3.2 Processor 

Processor ： Coyote 协议处理接口 ，如果说**EndPoint是用来实现TCP/IP协议的，那么Processor用来实现HTTP协议，Processor接收来自EndPoint的Socket，读取字节流解析成Tomcat Request和Response对象，并通过Adapter将其提交到容器处理**，Processor是对应用层协议的抽象。 

##### 5.4.3.3 ProtocolHandler 

ProtocolHandler： Coyote 协议接口， 通过Endpoint 和 Processor ， 实现针对具体协议的处理能力。Tomcat 按照协议和I/O 提供了6个实现类 ： AjpNioProtocol ，AjpAprProtocol， AjpNio2Protocol ， Http11NioProtocol ，Http11Nio2Protocol ，Http11AprProtocol。我们在配置tomcat/conf/server.xml 时 ， 至少要指定具体的ProtocolHandler , 当然也可以指定协议名称 ， 如 ： HTTP/1.1 ，如果安装了APR，那么将使用Http11AprProtocol ， 否则使用 Http11NioProtocol 。 

##### 5.4.3.4 Adapter 

由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat定义了自己的Request类来“存放”这些请求信息。ProtocolHandler接口负责解析请求并生成Tomcat Request类。但是这个Request对象不是标准的ServletRequest，也就意味着，不能用Tomcat Request作为参数来调用容器。Tomcat设计者的解决方案是引入**CoyoteAdapter**，这是适配器模式的经典运用，连接器调用CoyoteAdapter的Sevice方法，传入的是TomcatRequest对象，CoyoteAdapter负责将Tomcat Request转成ServletRequest，**再调用容器的Service方法**

### 5.5 Catalina 容器

Tomcat是一个由一系列可配置的组件构成的Web容器，而Catalina是Tomcat的servlet容器。

**Catalina 是Servlet 容器实现**，包含了之前讲到的所有的容器组件，以及后续章节涉及到的安全、会话、集群、管理等Servlet 容器架构的各个方面。它通过松耦合的方式集成Coyote，以完成按照请求协议进行数据读写。同时，它还包括我们的启动入口、Shell程序等。

#### 5.5.1 Catalina的地位

Tomcat 本质上就是一款 Servlet 容器， 因此**Catalina才是 Tomcat 的核心** ， <u>其他模块都是为Catalina提供支</u>撑。 比如: 通过**Coyote 模块提供链接通信**，**Jasper 模块提供JSP引擎**，**Naming 提供JNDI 服务**，**Juli 提供日志服务**

![1622860891709](E:\SoftwareNote\Web\img\Tomcat分层结构图.png)

#### 5.5.2 Catalina的组成

![1622861049351](E:\SoftwareNote\Web\img\Catalina的结构图.png)

Catalina负责管理Server，而Server表示着整个服务器。Server下面有多个服务Service，每个服务都包含着多个连接器组件Connector（Coyote 实现）和一个容器组件Container。**在Tomcat 启动的时候， 会初始化一个Catalina的实例。** 

>  Catalina 各个组件的职责 

|   组件    |                             职责                             |
| :-------: | :----------------------------------------------------------: |
| Catalina  | 负责解析Tomcat的配置文件 , 以此来创建服务器Server组件，并根据 命令来对其进行管理 |
|  Server   | 服务器表示整个Catalina Servlet容器以及其它组件，负责组装并启动Servlet引擎,Tomcat连接器。Server通过实现Lifecycle接口，提供了一种优雅的启动和关闭整个系统的方式 |
|  Service  | 服务是Server内部的组件，一个Server包含多个Service。它将若干个 Connector组件绑定到一个Container（Engine）上 |
| Connector | 连接器，处理与客户端的通信，它负责接收客户请求，然后转给相关 的容器处理，最后向客户返回响应结果 |
| Container |  容器，负责处理用户的servlet请求，并返回对象给web用户的模块  |

### 5.6 Container 容器结构

Tomcat设计了4种容器，分别是**Engine、Host、Context和Wrapper**。这4种容器不是平行关系，**而是父子关系**。 Tomcat通过一种分层的架构，使得Servlet容器具有很好的灵活性。

![1622861679341](E:\SoftwareNote\Web\img\Engine-Host-Context-Wrapper的关系.png)

> 各组件的职责

|  容器   |                             描述                             |
| :-----: | :----------------------------------------------------------: |
| Engine  | 表示整个Catalina的Servlet引擎，用来管理多个虚拟站点，一个Service 最多只能有一个Engine，但是一个引擎可包含多个Host |
|  Host   | 代表一个虚拟主机，或者说一个站点，可以给Tomcat配置多个虚拟主机地址，而一个虚拟主机下可包含多个Context |
| Context |    表示**一个Web应用程序**， 一个Web应用可包含多个Wrapper    |
| Wrapper | 表示**一个Servlet**，Wrapper 作为容器中的最底层，不能包含子容器 |

我们也可以再通过Tomcat的server.xml配置文件来加深对Tomcat容器的理解。Tomcat采用了组件化的设计，它的构成组件都是可配置的，其中最外层的是Server，其他组件按照一定的格式要求配置在这个顶层容器中。 

```xml
<Server>
	<Service>
    	<Connector executor="tomcatThreadPool" port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000" redirectPort="8443" />
        <Connector port="8081" protocol="HTTP/1.1"
               connectionTimeout="20000" redirectPort="8443" />
        <Engine name="Catalina" defaultHost="www.codeman.com">
        	<Host name="www.codeman.com"  appBase="catalina-home/webapps"
            	unpackWARs="true" autoDeploy="true">
            	<Context></Context>
            </Host>
            <Host name="oa.codeman.com"  appBase="catalina-home/webapps"
            	unpackWARs="true" autoDeploy="true">
            	<Context></Context>
            </Host>
        </Engine>
    </Service>
</Server>
```

> Tomcat是怎么管理这些容器的呢？

你会发现这些容器具有父子关系，形成一个树形结构，你可能马上就想到了设计模式中的**组合模式**。没错，Tomcat就是用组合模式来管理这些容器的。具体实现方法是，**所有容器组件都实现了Container接口**，因此组合模式可以使得用户对单容器对象和组合容器对象的使用具有一致性。**这里单容器对象指的是最底层的Wrapper，组合容器对象指的是上面的Context、Host或者Engine。**

![1622862441317](E:\SoftwareNote\Web\img\Tomcat的Container架构图.png)

<u>Container接口扩展了LifeCycle接口</u>，**LifeCycle接口用来统一管理各组件的生命周期**

1） init（）：初始化组件 

2） start（）：启动组件 

3） stop（）：停止组件 

4） destroy（）：销毁组件 

## 6. Tomcat启动流程

![1622862930253](E:\SoftwareNote\Web\img\Tomcat启动流程图.png)

步骤 :  各组件的init()+start()

1） 启动tomcat ， 需要调用 bin/startup.bat (在linux 目录下 , 需要调用 bin/startup.sh) ， 在startup.bat 脚本中, 调用了catalina.bat。 

2） 在catalina.bat 脚本文件中，调用了**BootStrap 中的main方法(入口)。** 

3）在BootStrap 的main 方法中调用了 init 方法 ， 来创建Catalina 及 初始化类加载器。 

4）在BootStrap 的main 方法中调用了 load 方法 ， 在其中又调用了Catalina的load方法。

5）在Catalina 的load 方法中 , 需要进行一些初始化的工作, **并需要构造Digester 对象, 用于解析 XML**。 

6） 然后在调用后续组件的初始化操作 。。。加载Tomcat的配置文件，初始化容器组件 ，监听对应的端口号， 准备接受客户端请求



### 6.1 各组件的默认实现

上面我们提到的Server、Service、Engine、Host、Context都是接口， 下图中罗列了这些接口的默认实现类。

当前对于 Endpoint组件来说，在Tomcat中没有对应的Endpoint接口， 但是有一个**抽象类 AbstractEndpoint ，其下有三个实现类： NioEndpoint、Nio2Endpoint、AprEndpoint** ， 这三个实现类，分别对应于前面讲解链接器 Coyote时， 提到的链接器支持的三种IO模型：NIO，NIO2，APR ， **Tomcat8.5版本中，默认采用的是 NioEndpoint。**

![1622863558816](E:\SoftwareNote\Web\img\各组件的默认实现.png)

ProtocolHandler ： Coyote协议接口，通过封装Endpoint和Processor ， 实现针对具体协议的处理功能。Tomcat按照协议和IO提供了6个实现类。 

AJP协议： 

1） AjpNioProtocol ：采用NIO的IO模型。 

2） AjpNio2Protocol：采用NIO2的IO模型。 

3） AjpAprProtocol ：采用APR的IO模型，需要依赖于APR库。 

HTTP协议： 

1） Http11NioProtocol ：采用NIO的IO模型，默认使用的协议（如果服务器没有安装APR）。 

2） Http11Nio2Protocol：采用NIO2的IO模型。 

3） Http11AprProtocol ：采用APR的IO模型，需要依赖于APR库。

![1622865557345](E:\SoftwareNote\Web\img\ProtocolHandler的实现类们.png)

### 6.2 总结

从启动流程图中以及源码中，我们可以看出Tomcat的启动过程非常标准化， 统一按照生命周期管理接口Lifecycle的定义进行启动。首先调用init() 方法进行组件的逐级初始化操作，然后再调用start()方法进行启动。 

每一级的组件除了完成自身的处理外，还要负责调用子组件响应的生命周期管理方法，组件与组件之间是松耦合的，因为我们可以很容易的通过配置文件进行修改和替换。

## 7. Tomcat 请求处理流程  

设计了这么多层次的容器，<u>Tomcat是怎么确定每一个请求应该由哪个Wrapper容器里的Servlet来处理的呢</u>？答案是，**Tomcat是用Mapper组件来完成这个任务的。** 

**Mapper组件的功能就是将用户请求的URL定位到一个Servlet**，它的工作原理是： 

Mapper组件里保存了**Web应用的配置信息(如web.xml)**，其实就是容器组件与访问路径的映射关系，比如Host容器里配置的域名、Context容器里的Web应用路径，以及Wrapper容器里Servlet映射的路径，你可以想象这些配置信息就是一个多层次的Map。

当一个请求到来时，Mapper组件通过解析请求URL里的域名和路径，再到自己保存的Map里去查找，就能定位到一个Servlet。请你注意，**一个请求URL最后只会定位到一个Wrapper容器，也就是一个Servlet。** 

下面的示意图中 ， 就描述了 当用户请求链接 http://www.itcast.cn/bbs/findAll 之后, 是如何找到最终处理业务逻辑的servlet 。 

![1622866009834](E:\SoftwareNote\Web\img\url找wrapper(Servlet)流程.png)



> **tomcat是如何处理Http请求流程的？** 

假设来我们在浏览器上输入 

http://localhost:8080/my-web-mave/index.jsp 

在tomcat中是如何处理这个请求流程的： 

1. 我们的请求被发送到本机端口8080，被在那里侦听的Coyote HTTP/1.1 **Connecto**r获得。 

2. Connector把该请求交给它所在的Service的**Engine**来处理，并等待来自Engine的回应 。 

3. Engine获得请求localhost/my-web-maven/index.jsp，**匹配它所拥有的所有虚拟主机Host** ，我们的虚拟主机在**server.xml中默认配置的就是localhost**。 

4. **Engine匹配到name=localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机**）。 

5. localhost Host获得请求/my-web-maven/index.jsp，匹配它所拥有的所有**Context**。 
6. Host匹配到路径为**/my-web-maven的Context**（如果匹配不到就把该请求交给路径名为”"的 Context去处理）。 

7. path=”/my-web-maven”的Context获得请求/index.jsp，在它的**mapping table**中寻找对应的 servlet 。 

8. **Context匹配到URL PATTERN为*.jsp的servlet，对应于JspServlet类**。 
9. 构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet或doPost方法 

10. Context把执行完了之后的HttpServletResponse对象返回给Host 。 
11. Host把HttpServletResponse对象返回给Engine 。 
12. Engine把HttpServletResponse对象返回给Connector 。 
13. Connector把HttpServletResponse对象返回给客户browser

### 7.1 Tomcat请求流程

![1622866059457](E:\SoftwareNote\Web\img\Tomcat请求流程.png)

步骤如下: 

1) Connector组件**Endpoint**中的Acceptor**监听**客户端套接字连接并**接收Socket**。 

2) 将**连接交给线程池Executor**处理，开始执行请求响应任务。 

3) **Processo**r组件读取消息报文，解析请求行、请求体、请求头，**封装成Request对象**。 

4) **Mapper**组件根据请求行的**URL值和请求头的Host值匹配由哪个Host容器、Context容器、Wrapper容器处理请求。** 

5) **CoyoteAdaptor**组件负责**将Connector组件和Engine容器关联**起来，把生成的Request对象和响应对象Response传递到**Engine容器中，调用 Pipeline**。 

6) Engine容器的管道开始处理，管道中包含若干个Valve、每个Valve负责部分处理逻 

辑。执行完Valve后会执行基础的 Valve--StandardEngineValve，负责调用Host容器的Pipeline。 

7) Host容器的管道开始处理，流程类似，最后执行 Context容器的Pipeline。 

8) Context容器的管道开始处理，流程类似，最后执行 Wrapper容器的Pipeline。 

9) Wrapper容器的管道开始处理，流程类似，最后执行 Wrapper容器对应的Servlet对象的处理方法service()

![1622866665030](E:\SoftwareNote\Web\img\Tomcat请求流程图.png)

在前面所讲解的Tomcat的整体架构中，我们发现Tomcat中的各个组件各司其职，组件之间松耦合，确保了整体架构的可伸缩性和可拓展性，那么在组件内部，如何增强组件的灵活性和拓展性呢？ 在Tomcat中，每个Container组件采用责任链模式来完成具体的请求处理。 

在Tomcat中定义了Pipeline 和 Valve 两个接口，Pipeline 用于构建责任链， 后者代表责任链上的每个处理器。Pipeline 中维护了一个基础的Valve，它始终位于Pipeline的末端（最后执行），封装了具体的请求处理和输出响应的过程。当然，我们也可以调用addValve()方法， 为Pipeline 添加其他的Valve， 后添加的Valve 位于基础的Valve之前，并按照添加顺序执行。Pipiline通过获得首个Valve来启动整合链条的执行 。

## 8. Jasper  

### 8.1 Jasper 简介

对于基于JSP 的web应用来说，我们可以直接在JSP页面中编写 Java代码，添加第三方的标签库，以及使用EL表达式。但是无论经过何种形式的处理，**最终输出到客户端的都是 标准的HTML页面（包含js ，css...**），并不包含任何的java相关的语法。 也就是说， **我们可以把jsp看做是一种运行在服务端的脚本**。 那么服务器是如何将 JSP页面转换为HTML页面的呢？ 

Jasper模块是Tomcat的JSP核心引擎，我们知道**JSP本质上是一个Servlet**。**Tomcat使用Jasper对JSP语法进行解析，生成Servlet并生成Class字节码，用户在进行访问jsp时，会访问Servlet，最终将访问的结果直接响应在浏览器端 。另外，在运行的时候，Jasper还会检测JSP文件是否修改，如果修改，则会重新编译JSP文件。**

**Jasper会动态的根据jsp内容生成一个servlet内容的java文件然后编译成class文件，请求到url就会执行class文件，然后解析业务动态数据封装成html通过response.getWrite().write()回前端.**

Tomcat 并不会在启动Web应用的时候自动编译JSP文件(当然也可以设置成启动时加载，提高请求速度)， 而是在客户端第一次请求时，才编译需要访问的JSP文件。

- JspServlet处理流程

![1622869000506](E:\SoftwareNote\Web\img\JspServlet处理流程.png)

### 8.2 编译jsp文件后的class文件存放位置

1） 如果在 tomcat/conf/web.xml 中配置了参数scratchdir ， 则jsp编译后的结果，就会存储在该目录下 。默认是在work文件夹内

2） 如果没有配置该选项， 则会将编译后的结果，存储在Tomcat安装目录下的work/Catalina(Engine名称)/localhost(Host名称)/Context名称 。 假设项目名称为jsp_demo 01。 

3） 如果使用的是 IDEA 开发工具集成Tomcat 访问web工程中的jsp ， 编译后的结果， 

存放在 ： 

```tex
C:\Users\Administrator\.IntelliJIdea2019.1\system\tomcat\_project_tomcat\w ork\Catalina\localhost\jsp_demo_01_war_exploded\org\apache\jsp
```

### 8.3 预编译 

除了运行时编译，我们还可以直接在Web应用启动时， 一次性将Web应用中的所有的JSP页面一次性编译完成。在这种情况下，Web应用运行过程中，便可以不必再进行实时编译，而是直接调用JSP页面对应的Servlet 完成请求处理， 从而提升系统性能。 

Tomcat 提供了一个Shell程序JspC，用于支持JSP预编译，而且在Tomcat的安装目录下提供了一个 catalina-tasks.xml 文件声明了Tomcat 支持的Ant任务， 因此，我们很容易使用 Ant 来执行JSP 预编译 。（要想使用这种方式，必须得确保在此之前已经下载并安装了Apache Ant）。

### 8.4 jsp编译过程

![1622869479366](E:\SoftwareNote\Web\img\Jsp编译过程.png)

Compiler 编译工作主要包含代码生成 和 编译两部分 ： 

代码生成 

1） Compiler 通过一个 PageInfo 对象保存JSP 页面编译过程中的各种配置，这些配置可能来源于 Web 应用初始化参数， 也可能来源于JSP页面的指令配置（如 page ，include）。 

2） 调用ParserController 解析指令节点， 验证其是否合法，同时将配置信息保存到PageInfo 中， 用于控制代码生成。 

3） 调用ParserController 解析整个页面， 由于 JSP 是逐行解析， 所以对于每一行会创建一个具体的Node 对象。如 静态文本（TemplateText）、Java代码（Scriptlet）、定制标签（CustomTag）、Include指令（IncludeDirective）。 

4） 验证除指令外其他所有节点的合法性， 如 脚本、定制标签、EL表达式等。 

5） 收集除指令外其他节点的页面配置信息。 

6） 编译并加载当前 JSP 页面依赖的标签

7） 对于JSP页面的EL表达式，生成对应的映射函数。 

8） 生成JSP页面对应的Servlet 类源代码 

编译

代码生成完成后， Compiler 还会生成 SMAP 信息。 如果配置生成 SMAP 信息，Compiler 则会在编译阶段将SMAP 信息写到class 文件中 。 

在编译阶段， Compiler 的两个实现 AntCompiler 和 JDTCompiler 分别调用先关框架的API 进行源代码编译。 

对于 AntCompiler 来说， 构造一个 Ant 的javac 的任务完成编译。 

对于 JDTCompiler 来说， 调用 org.eclipse.jdt.internal.compiler.Compiler 完成编译。 

## 9. Tomcat配置

Tomcat 服务器的配置主要集中于 tomcat/conf 下的 catalina.policy、catalina.properties、context.xml、server.xml、tomcat-users.xml、web.xml 文件。

## 10. Web配置

web.xml 是web应用的描述文件， 它支持的元素及属性来自于Servlet 规范定义 。 在Tomcat 中， Web 应用的描述信息包括 tomcat/conf/web.xml 中默认配置 以及 Web应用 WEB-INF/web.xml 下的定制配置。

## 11. Tomcat 管理配置 

从早期的Tomcat版本开始，就提供了Web版的管理控制台，他们是两个独立的Web应用，位于webapps目录下。Tomcat 提供的管理应用有用于管理的Host的host-manager和用于管理Web应用的manager。 （像Oracle的界面管理）

### 11.1 host-manager 

### 11.2 web-manager

![1622807307310](E:\SoftwareNote\服务器\Tomcat\img\TomcatManager可以查看应用JVM内存使用情况.png)

## 12. Tomcat的JVM配置

- catalina.bat 

```xml
set JAVA_OPTS=‐server ‐Xms2048m ‐Xmx2048m ‐XX:MetaspaceSize=256m ‐XX:MaxMetaspaceSize=256m ‐XX:SurvivorRatio=8
```

- catalina.sh 

```xml
JAVA_OPTS="‐server ‐Xms1024m ‐Xmx2048m ‐XX:MetaspaceSize=256m ‐ XX:MaxMetaspaceSize=512m ‐XX:SurvivorRatio=8"
```

## 13. Tomcat集群 （Nginx）

### 13.1  session共享方案

#### 13.1.1 ip_hash 策略 

一个用户发起的请求，只会请求到tomcat1上进行操作，另一个用户发起的请求只在tomcat2上进行操作 。那么这个时候，同一个用户发起的请求，都会通过nginx的ip_hash策略，将请求转发到其中的一台Tomcat上。 

#### 13.1.2 Session复制 

1） 在Tomcat的conf/server.xml 配置如下: 

`<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>  `

2） 在Tomcat部署的应用程序 servlet_demo01 的web.xml 中加入如下配置 ： 

`<distributable/> `

3） 配置完毕之后， 再次重启两个 Tomcat服务。 

上述方案，适用于较小的集群环境**（节点数不超过4个**），如果集群的节点数比较多的话，<u>通过这种广播的形式来</u>完成Session的复制，**会消耗大量的网络带宽，影响服务的性能。**

#### 13.1.3 SSO-单点登录 

单点登录（Single Sign On），简称为 SSO，是目前比较流行的企业业务整合的解决方案之一。SSO的定义是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统，也是用来解决集群环境Session共享的方案之一 。 

## 14. Tomcat安全

### 14.1 配置安全  

1） 删除webapps目录下的所有文件，禁用tomcat管理界面； (就是页面manager，可以下线服务，更改session配置等等)

2） 注释或删除tomcat-users.xml文件内的所有用户权限； 

3） 更改关闭tomcat指令或禁用； （8005端口默认执行ShowDown命令，会把服务器down）

tomcat的server.xml中定义了可以直接关闭 Tomcat 实例的管理端口（默认8005）。可以通过 telnet 连接上该端口之后，输入 SHUTDOWN （此为默认关闭指令）即可关闭Tomcat 实例（注意，此时虽然实例关闭了，但是进程还是存在的）。由于默认关闭Tomcat 的端口和指令都很简单。默认端口为8005，指令为SHUTDOWN 。 

### 14.2 应用安全

在大部分的Web应用中，特别是一些后台应用系统，都会实现自己的安全管理模块（权限模块）(如Gateway)，用于控制应用系统的安全访问，基本包含两个部分：**认证（登录/单点登录） 和授权（功能权限、数据权限**）两个部分。对于当前的业务系统，可以自己做一套适用于自己业务系统的权限模块，也有很多的应用系统直接使用一些功能完善的安全框架，将其集成到我们的web应用中，如：**SpringSecurity、Apache Shiro**等。 

### 14.3 传输安全 （Https协议：超文本传输<u>安全</u>传输协议）

> HTTPS介绍 

HTTPS的全称是**超文本传输安全协议**（Hypertext Transfer Protocol **Secure**），是一种网络安全传输协议。**在HTTP的基础上加入SSL/TLS来进行数据加密**，保护交换数据不被泄露、窃取。 

SSL 和 TLS 是用于网络通信安全的加密协议，它允许客户端和服务器之间通过安全链接通信。

>  SSL 协议的3个特性： 

1） **保密**：通过SSL链接传输的数据时加密的。 

2） **鉴别**：通信双方的**身份鉴别**，通常是可选的，单至少有一方需要验证。 

3） **完整性**：**传输数据的完整性检查**。 

从性能角度考虑，**加解密是一项计算昂贵的处理**，因为尽量不要将整个Web应用采用SSL链接， 实际部署过程中， 选择有必要进行安全加密的页面（存在敏感信息传输的页面）采用SSL通信。 

>  HTTPS和HTTP的区别主要为以下四点：

1） HTTPS协议需要到证书颁发机构CA申请SSL证书, 然后与域名进行绑定，HTTP不用申请证书； 

2） HTTP是超文本传输协议，属于应用层信息传输，HTTPS 则是具有SSL加密传安全性传输协议，对数据的传输进行加密，相当于HTTP的升级版； 

3） HTTP和HTTPS使用的是完全不同的连接方式，用的端口也不一样，前者是8080，后者是8443。 

4） HTTP的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比HTTP协议安全。 

> Https协议的优势

1） **提高网站排名**，有利于SEO。谷歌已经公开声明两个网站在搜索结果方面相同，如果一个网站启用了SSL，它可能会获得略高于没有SSL网站的等级，而且百度也表明对安装了SSL的网站表示友好。因此，网站上的内容中启用SSL都有明显的SEO优势。 

2） **隐私信息加密，防止流量劫持**。特别是涉及到隐私信息的网站，互联网大型的数据泄露的事件频发发生，网站进行信息加密势在必行。 

3） **浏览器受信任**。 自从各大主流浏览器大力支持HTTPS协议之后，访问HTTP的网站都会**提示“不安全”的警告信**息。 

## 15. Tomcat性能调用

### 15.1 性能测试 

对于系统性能，用户最直观的感受就是系统的加载和操作时间，即用户执行某项操作的耗时。从更为专业的角度上讲，性能测试可以从以下两个指标量化。 

1). **响应时间**：如上所述，为执行某个操作的耗时。大多数情况下，我们需要针对同一个操作测试多次，以获取操作的平均响应时间。 

2). **吞吐量：**即在给定的时间内，系统支持的事务数量，计算单位为 TPS。

通常情况下，我们需要借助于一些自动化工具来进行性能测试，因为手动模拟大量用户的并发访问几乎是不可行的，而且现在市面上也有很多的性能测试工具可以使用，如：**ApacheBench(免费)、ApacheJMeter（免费）**、WCAT、WebPolygraph、LoadRunner。 

### 15.2 Tomcat调优实际上就是JVM调优

Tomcat是一款Java应用，那么JVM的配置便与其运行性能密切相关，而JVM优化的重点则集中在**内存分配和GC策略**的调整上，因为内存会直接影响服务的运行效率和吞吐量，JVM垃圾回收机制则会不同程度地导致程序运行中断。可以根据应用程序的特点，选择不 同的垃圾回收策略，调整JVM垃圾回收策略，可以极大减少垃圾回收次数，提升垃圾回收效率，改善程序运行性能。

## 16. Tomcat支持WebSocket

HTML5的双向长连接传输协议



## Tomcat进阶

https://www.bilibili.com/video/BV1dJ411N7Um?p=30&spm_id_from=pageDriver

https://blog.csdn.net/ly823260355/article/details/104181278







- JVM调优

  TomcatManager可以查看应用JVM内存使用情况。

  修改catalina.sh 

  ```
  -- windows
  set "JAVA_OPTS=%JAVA_OPTS% %JSSE_OPTS%"
  
  -- linux
  JAVA_OPTS="$JAVA_OPTS -Djava.protocol.handler.pkgs=org.apache.catalina.webresources"
  ```

![1622807307310](E:\SoftwareNote\服务器\Tomcat\img\TomcatManager可以查看应用JVM内存使用情况.png)



- tomcat集群(Nginx)中session问题、
  - Nginx均衡策略用ip_hash
  - tomcat的session复制（适用小于4台集群，否则集群数多了，一直复制session也很消耗资源）
  - SSO（Single Sign On）单点登录
  - session共享（redis）