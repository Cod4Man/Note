# Netty

## 1. Netty的介绍

* Netty是由JBOSS提供的一个Java开源框架
* Netty是一个**异步、基于事件驱动**的网络应用框架，用以快速高性能、高可靠性的网络IO程序
* Netty主要针对在TCP协议下，面向Client端的**高并发**应用，或者Peer-to-Peer场景下的大量数据继续传输的应用
* **Netty本质是一个NIO框架**，适用于服务器通讯相关的多种应用场景
* 要透彻理解Netty，需要先掌握NIO

## 2. Netty的应用

### 2.1 互联网

- 服务间的远程调用，高性能的RPC框架，Netty作为异步高性能的通信框架，往往被RPC框架作基础通信组件
- 典型的应用有：Dubbo

### 2.2 游戏

### 2.3 大数据

## 3. 学习资料

Netty in action

## 4. Java IO

- IO模型的简单理解： 用某种通道进行数据的发送O和接收I，很大程度上决定了程序通信的性能
- JAVA支持3中网络编程IO模式：BIO / NIO / AIO

### 4.1 Java BIO

#### 4.1.1 基本介绍

1. Java BIO 是传统的java io编程，BIO即 Blocking I/O ，同步阻塞，服务器实现模式为**一个连接一个线程**，即客户端有连接请求时服务器就需要启动一个线程进行处理，如果连接不做任何事情会造成不必要的的线程开销(可通过线程池记置改善，实现多个客户连接服务器)
2. BIO模式适用于连接数目比较小且固定的架构，这种方式对服务器的资源要求比较高，并发局限于应用内，在JDK4之只有BIO，程序简单易理解。

#### 4.1.2 工作机制

![1656147090525](image/netty/JavaBIO工作机制图.png)

1. 服务器启动一个ServerSocket
2. 客户端启动Socket对服务器进行通信，默认情况下服务器需要对每一个客户建立一个线程与之通信
3. 客户端发出请求吼，先咨询服务器是否有线程响应，如果没有则会等待，或者被拒绝
4. 如果有响应，客户端线程会等待请求结束吼，在继续执行

#### 4.1.3 实例

```java
package com.codeman.netty.bio;

import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class BIOServer {
    public static void main(String[] args) throws Exception {

        //线程池机制

        //思路
        //1. 创建一个线程池
        //2. 如果有客户端连接，就创建一个线程，与之通讯(单独写一个方法)

        ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();

        //创建ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);

        System.out.println("服务器启动了");

        while (true) {

            System.out.println("线程信息1 id =" + Thread.currentThread().getId() + " 名字=" + Thread.currentThread().getName());
            //监听，等待客户端连接
            System.out.println("等待连接....");
	    // 等待客户端会阻塞
            final Socket socket = serverSocket.accept();
            System.out.println("连接到一个客户端");

            //就创建一个线程，与之通讯(单独写一个方法)
            newCachedThreadPool.execute(new Runnable() {
                public void run() { //我们重写
                    //可以和客户端通讯
                    handler(socket);
                }
            });

        }
  
        // //  客户端1 连接但未输入
        //连接到一个客户端
        //线程信息1 id =1 名字=main
        //等待连接....
        //线程信息2 id =12 名字=pool-1-thread-1
        //线程信息3 id =12 名字=pool-1-thread-1
        //read....
        //
        //// 客户端1 输入
        //zz
        //线程信息3 id =12 名字=pool-1-thread-1
        //read....
        //
        //// 客户端2 连接但未输入
        //连接到一个客户端
        //线程信息1 id =1 名字=main
        //等待连接....
        //线程信息2 id =13 名字=pool-1-thread-2
        //线程信息3 id =13 名字=pool-1-thread-2
        //read....


    }

    //编写一个handler方法，和客户端通讯
    public static void handler(Socket socket) {

        try {
            System.out.println("线程信息2 id =" + Thread.currentThread().getId() + " 名字=" + Thread.currentThread().getName());
            byte[] bytes = new byte[1024];
            //通过socket 获取输入流
            InputStream inputStream = socket.getInputStream();

            //循环的读取客户端发送的数据
            while (true) {

                System.out.println("线程信息3 id =" + Thread.currentThread().getId() + " 名字=" + Thread.currentThread().getName());

                System.out.println("read....");
		// 等待客户端输入会阻塞
               int read =  inputStream.read(bytes);
               if(read != -1) {
                   System.out.println(new String(bytes, 0, read
                   )); //输出客户端发送的数据
               } else {
                   break;
               }
            }


        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            System.out.println("关闭和client的连接");
            try {
                socket.close();
            }catch (Exception e) {
                e.printStackTrace();
            }

        }
    }
}

```

#### 4.1.4 缺陷

1. 每个请求都需要创建独立的线程，与对应的客户端机械能数据Read/业务处理/数据Write
2. 当并发数比较大时，需要创建大量线程来处理连接，系统资源占用较大
3. 连接建立后，如果当前线程暂时没有数据可读，则线程就阻塞在read操作上，会操作线程资源浪费

### 4.2 Java NIO

#### 4.2.1 基本介绍

1. Java NIO全程 java non-blocking IO ，从JDK4开始，是同步非阻塞的
2. NIO三大核心： **通道Channel / 缓冲区Buffer / 选择器Selector**
3. NIO是面向缓冲区，或者说是面向 块 编程的。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动，增加了处理过程的灵活性，使用它可以提供非阻塞式的高伸缩性网络
4. Java NIO的非阻塞模式，使一个线程从某通道发送请求或者读取数据，但它仅能得到目前可用的数据，如果目前没有数据可用，则不会获取，而不是保持线程阻塞，直至数据可读前，都可继续做其他事情。非阻塞写也是如此。
5. NIO可以做到一个线程处理多个操作
6. HTTP2.0使用了多路复用的技术，做到同一个连接并发处理多个请求，而并发请求的数量比HTTP1.1大了好几个数量级

#### 4.2.2 NIO和BIO对比

1. BIO以流的方式处理数据，而NIO以块的方式处理数据，块I/O的效率比流I/O高很多
2. BIO是阻塞的，NIO是非阻塞的
3. BIO是基于字节流和字符流进行操作，而NIO是基于Channel和Buffer进行操作，数据总是从Channel到Buffer，或者从Buffer到Channel。Selector用于监听多个通道的时间(连接请求isAcceptable/数据到达isReadable等)，因此使用单个线程就可以监听多个客户端通道

#### 4.2.3 NIO三大核心

![1656153067259](image/netty//NIO三大核心.png)

- 每个channel都会对应一个Buffer
- Selector对应一个线程，一个线程对应多个Channel
- 改图反映了有三个channel注册到该selector程序
- 程序切换到哪个channel是由事件决定的，Event就是一个重要的概念
- selector会根据不同的事件，在各个通道上切换
- Buffer就是一个内存块，底层是一个数组
- 数据的读写是通过Buffer。在BIO中要么是输入流/要么是输出流，不能双向。但是NIO的Buffer是可以读也可以写，需要用flip方法切换。channel是双向的，可以返回底层操作系统的情况，比如Linux，底层的操作系统通道就是双向的。

##### 4.2.3.1 缓冲区（Buffer）

###### 4.2.3.1.1 基本介绍

缓冲区（Buffer）：缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个容器对象（含数组），该对象提供了一组方法，可以更轻松地使用内存块，，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel 提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由 Buffer，如图:【后面举例说明】

![1656154235058](image/netty/NIO-Buffer.png)

###### 4.2.3.1.2 Buffer主要属性和方法

- 属性

![1656154456571](image/netty/Buffer属性.png)

- 方法

![1656154556030](image/netty/Buffer方法.png)

- ByteBuffer主要方法

![1656154692245](image/netty/ByteBuffer主要方法.png)

##### 4.2.3.2 通道 (Channel)

###### 4.2.3.2.1 基本介绍

1. NIO 的通道类似于流，但有些区别如下：
   - **通道可以同时进行读写**，而流只能读或者只能写
   - 通道可以实现**异步读写**数据
   - 通道可以从缓冲读数据，也可以写数据到缓冲（**双向**）:
2. BIO 中的 Stream 是单向的，例如 FileInputStream 对象只能进行读取数据的操作，而 NIO 中的通道（Channel）是**双向**的，可以读操作，也可以写操作。
3. Channel 在 NIO 中是一个接口 public interface Channel extends Closeable{}
4. 常用的 Channel 类有：FileChannel、DatagramChannel、ServerSocketChannel 和 SocketChannel。【ServerSocketChanne 类似 ServerSocket、SocketChannel 类似 Socket】
5. FileChannel 用于文件的数据读写，DatagramChannel 用于 UDP 的数据读写，ServerSocketChannel 和 SocketChannel 用于 TCP 的数据读写。

##### 4.2.3.3 选择器 （Selector）

###### 4.2.3.3.1 基本介绍

1. Java 的 NIO，用非阻塞的 IO 方式。可以用一个线程，处理多个的客户端连接，就会使用到 Selector（选择器）。
2. Selector 能够检测多个注册的通道上是否有事件发生（注意：多个 Channel 以事件的方式可以注册到同一个 Selector），如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个连接和请求。
3. 只有在连接/通道真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程。
4. 避免了多线程之间的上下文切换导致的开销。

###### 4.2.3.3.2 Selector示意图

![1656165268170](image/netty/Selector示意图.png)

1. Netty 的 IO 线程 NioEventLoop 聚合了 Selector（选择器，也叫多路复用器），可以同时并发处理成百上千个客户端连接。
2. 当线程从某客户端 Socket 通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务。
3. 线程通常将非阻塞 IO 的空闲时间用于在其他通道上执行 IO 操作，所以单独的线程可以管理多个输入和输出通道。
4. 由于读写操作都是非阻塞的，这就可以充分提升 IO 线程的运行效率，避免由于频繁 I/O 阻塞导致的线程挂起。
5. 一个 I/O 线程可以并发处理 N 个客户端连接和读写操作，这从根本上解决了传统同步阻塞 I/O 一连接一线程模型，架构的性能、弹性伸缩能力和可靠性都得到了极大的提升。

###### 4.2.3.3.3 主要方法

* `Selector.open();`  // 获取一个Selector对象
* `selector.keys();` // 所有SelectionKeyke
* `selector.selectedKeys();` // 有事件的所有SelectionKeyke
* `selector.select();` //阻塞
* `selector.select(1000);` //阻塞 1000 毫秒，在 1000 毫秒后返回
* `selector.wakeup();` //唤醒 selector
* `selector.selectNow();` //不阻塞，立马返还

###### 4.2.3.3.3 NIO非阻塞网络编程原理

![1656166366016](image/netty/NIO非阻塞网络编程原理.png)

1. 当客户端连接时，会通过 ServerSocketChannel 得到 SocketChannel。
2. Selector 进行监听 select 方法，返回有事件发生的通道的个数。
3. 将 socketChannel 注册到 Selector 上，register(Selector sel, int ops)，一个 Selector 上可以注册多个 SocketChannel。
4. 注册后返回一个 SelectionKey，会和该 Selector 关联（集合）。
5. 进一步得到各个 SelectionKey（有事件发生）。
6. 在通过 SelectionKey 反向获取 SocketChannel，方法 channel()。
7. 可以通过得到的 channel，完成业务处理。

#### 4.2.4 SelectionKey

1. SelectionKey，表示 Selector 和网络通道的注册关系，共四种：

- int OP_ACCEPT：有新的网络连接可以 accept，值为 16
- int OP_CONNECT：代表连接已经建立，值为 8
- int OP_READ：代表读操作，值为 1
- int OP_WRITE：代表写操作，值为 4

2. 主要方法

- Selector  selector() // 得到关联的Selector对象
- SelectableChannel cannel() // 得到关联的通道
- Object attachment() // 得到关联的共享数据
- SelectionKey interestOps(int ops) // 设置或改变监听事件
- boolean isAcceptable() // 是否有连接请求
- boolean isReadable() // 是否有读操作
- boolean isWritable() // 是否有写操作

#### 4.2.5 ServerSocketChannel

1. `ServerSocketChannel` 在服务器端监听新的客户端 `Socket` 连接，负责监听，不负责实际的读写操作
2. 主要方法：

- ServerSocketChannel open() // 得到一个SSC通道
- ServerSocketChannel bind(SocketAddress local) // 设置监控的地址
- SelectableChannel configureBlocking(boolean block) // 设置是否阻塞， 默认true为阻塞
- SelectionKey register(Selector sel, int ops) // 注册到一个选择器上并设置监听事件

#### 4.2.6 SocketChannel

1. `SocketChannel`，网络 `IO` 通道， **具体负责进行读写操作** 。`NIO` 把缓冲区的数据写入通道，或者把通道里的数据读到缓冲区。
2. 主要方法：

- SocketChannel open() // 得到一个SocketChannel通道
- SelectableChannel configureBlocking(boolean block) // 设置是否阻塞， 默认true为阻塞
- boolean connect(SocketAddress remote) // 连接服务器
- boolean finishConnect() // 如果connect()连接失败，则通过这个方法完成连接
- int write(ByteBuffer src) // 往通道写数据
- in read(ByteBuffer src) // 从通道读数据
- SelectionKey register(Selector sel, int ops, Object att) // 注册一个选择器并设置监听事件，最后一个参数可以设置共享数据
- void close() // 关闭通道

#### 4.2.7 实例：多人聊天室

- Server

```java
package com.codeman.netty.nio;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.Channel;
import java.nio.channels.SelectableChannel;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

/**
 * @author: zhanghongjie
 * @description:
 * @date: 2022/6/24 22:36
 * @version: 1.0
 */
public class SocketServer {
    private final static String HOST = "127.0.0.1";
    private final static int PORT = 6677;
    private static ServerSocketChannel serverSocketChannel;
    private static Selector selector;

    public static void main(String[] args) throws Exception {
        //得到选择器
        selector = Selector.open();
        serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.socket().bind(new InetSocketAddress(HOST, PORT));
        // 非阻塞
        serverSocketChannel.configureBlocking(false);
        // 注册
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        // 有连接通道
        while (true) {
            if (selector.select(2000) > 0) {
                Iterator<SelectionKey> selectionKeyIterator = selector.selectedKeys().iterator();

                while (selectionKeyIterator.hasNext()) {
                    SelectionKey selectionKey = selectionKeyIterator.next();
                    if (selectionKey.isAcceptable()) {
                        // 建立连接
                        ServerSocketChannel serverSocketChannel = (ServerSocketChannel) selectionKey.channel();
                        SocketChannel channel = serverSocketChannel.accept();
                        // 非阻塞
                        channel.configureBlocking(false);
                        // 注册
                        channel.register(selector, SelectionKey.OP_READ);
                        System.out.println(channel.getRemoteAddress() + " 注册了...");
                    }
                    // 可读
                    if (selectionKey.isReadable()) {
                        // 选择到的通道
                        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                        SocketChannel channel = (SocketChannel) selectionKey.channel();
                        if (channel.read(byteBuffer) > 0) {
                            String msg = new String(byteBuffer.array());
                            System.out.println("读取信息：" + msg);
                            // 服务器向客户广播递信息
                            Iterator<SelectionKey> iterator = selector.keys().iterator();
                            while (iterator.hasNext()) {
                                Channel channel1 = iterator.next().channel();
                                if (channel1 instanceof SocketChannel && channel1 != channel) {
                                    // 排除服务器
                                    ByteBuffer byteBuffer1 = ByteBuffer.wrap(msg.getBytes());
                                    ((SocketChannel)channel1).write(byteBuffer1);
                                }
                            }
                        }
                    }


                    // 移除key，避免重复读取
                    selectionKeyIterator.remove();
                }
            }
        }
    }
}

```

- Client

```java
package com.codeman.netty.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Scanner;

/**
 * @author: zhanghongjie
 * @description:
 * @date: 2022/6/24 22:44
 * @version: 1.0
 */
public class SocketClient {
    private final static String HOST = "127.0.0.1";
    private final static int PORT = 6677;
    private static SocketChannel socketChannel;
    private static Selector selector;

    public static void main(String[] args) throws Exception {
        selector = Selector.open();
        socketChannel = SocketChannel.open(new InetSocketAddress(HOST, PORT));
        socketChannel.configureBlocking(false);
        socketChannel.register(selector, SelectionKey.OP_READ);

        // 监听channel
        new Thread(() -> {
            while (true) {
                try {
                    // 重要，查找通道，否则selector.selectedKeys()没东西
                    if (selector.select() > 0) {//有可以用的通道

                        Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                        while (iterator.hasNext()) {
                            SelectionKey selectionKey = iterator.next();
                            if (selectionKey.isReadable()) {
                                ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                                SocketChannel channel = (SocketChannel) selectionKey.channel();
                                System.out.println(channel.read(byteBuffer));
                                String msg = new String(byteBuffer.array());
                                System.out.println("读取信息：" + msg);
                            }
                            // 重要！
                            iterator.remove();
                        }
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }

        }).start();


        // 监听输入
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()) {
            String msg = scanner.nextLine();
            // 发送给服务器，服务器再广播
            socketChannel.write(ByteBuffer.wrap(msg.getBytes()));
        }
    }
}

```

### 4.3 NIO与零拷贝

#### 4.3.1 基本介绍

1. 零拷贝是网络编程的关键，很多性能优化都离不开。
2. 在 `Java` 程序中，常用的零拷贝有 `mmap`（内存映射）和 `sendFile`。那么，他们在 `OS` 里，到底是怎么样的一个的设计？我们分析 `mmap` 和 `sendFile` 这两个零拷贝

#### 4.3.2 传统 IO 模型

![1656171493257](image/netty/传统IO模型拷贝.png)

**DMA** ：`direct memory access` 直接内存拷贝（不使用 `CPU`）

#### 4.3.3 mmap内存映射优化

1. `mmap` 通过**内存映射**，将文件映射到内核缓冲区，同时，用户空间可以共享内核空间的数据。这样，在进行网络传输时，就可以减少内核空间到用户空间的拷贝次数。如下图

![1656171692466](image/netty/mmap内存映射优化.png)

#### 4.3.4 sendFile优化

1. `Linux2.1` 版本提供了 `sendFile` 函数，其基本原理如下：数据根本**不经过用户态**，直接从内核缓冲区进入到 `SocketBuffer`，同时，由于和用户态完全无关，就减少了一次上下文切换

![1656172167891](image/netty/拷贝sendFile优化.png)

2. 零拷贝从操作系统角度，**是没有 `cpu` 拷贝**
3. `Linux在2.4` 版本中，做了一些修改，避免了从内核缓冲区拷贝到 `Socketbuffer` 的操作，直接拷贝到协议栈，从而再一次减少了数据拷贝。具体如下图和小结：

这里其实只有一个cpu拷贝，但是拷贝的信息比较少，就是一些desc描述信息，比如length/offset等，消耗低，可忽略

![1656172275068](image/netty/拷贝sendFile优化2.png)

#### 4.3.5 零拷贝的再次理解

1. mmap 适合小数据量读写，sendFile 适合大文件传输。
2. mmap 需要 4 次上下文切换，3 次数据拷贝；sendFile 需要 3 次上下文切换，最少 2 次数据拷贝。
3. sendFile 可以利用 DMA 方式，减少 CPU 拷贝，mmap 则不能（必须从内核拷贝到 Socket缓冲区）。

### 4.4. Java AIO

#### 4.4.1 基本介绍

1. JDK7 引入了 AsynchronousI/O，即 AIO。在进行 I/O 编程中，常用到两种模式：Reactor 和 Proactor。Java 的 NIO 就是 Reactor，当有事件触发时，服务器端得到通知，进行相应的处理
2. AIO 即 NIO2.0，叫做**异步不阻塞**的 IO。AIO 引入异步通道的概念，采用了 Proactor 模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用
3. 目前 AIO 还没有广泛应用，Netty 也是基于 NIO，而不是 AIO，因此我们就不详解 AIO 了，有兴趣的同学可以参考《Java新一代网络编程模型AIO原理及Linux系统AIO介绍》http://www.52im.net/thread-306-1-1.html

#### 4.4.2 BIO、NIO、AIO 对比表

|          |   BIO   |          NIO          |    AIO    |
| :------: | :------: | :--------------------: | :--------: |
|  IO模型  | 同步阻塞 | 同步非阻塞（多路复用） | 异步非阻塞 |
| 编程难度 |   简单   |          复杂          |    复杂    |
|  可靠性  |    差    |           好           |     好     |
|  吞吐量  |    低    |           高           |     高     |

1. 同步阻塞：到理发店理发，就一直等理发师，直到轮到自己理发。
2. 同步非阻塞：到理发店理发，发现前面有其它人理发，给理发师说下，先干其他事情，一会过来看是否轮到自己.
3. 异步非阻塞：给理发师打电话，让理发师上门服务，自己干其它事情，理发师自己来家给你理发

## 5. Netty概述

### 5.1 原生NIO存在的问题

1. NIO 的类库和 API 繁杂，使用麻烦：需要熟练掌握 Selector、ServerSocketChannel、SocketChannel、ByteBuffer等。
2. 需要具备其他的额外技能：要熟悉 Java 多线程编程，因为 NIO 编程涉及到 Reactor 模式，你必须对多线程和网络编程非常熟悉，才能编写出高质量的 NIO 程序。
3. 开发工作量和难度都非常大：例如客户端面**临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常流的处理**等等。
4. JDK NIO 的 Bug：例如臭名昭著的 Epoll Bug，它会导致 Selector 空轮询，最终导致 CPU100%。直到 JDK1.7 版本该问题仍旧存在，没有被根本解决。

### 5.2 Netty的优点

*Netty 对 JDK 自带的 NIO 的 API 进行了封装，解决了上述NIO问题。*

1. 设计优雅：适用于各种传输类型的统一 API 阻塞和非阻塞 Socket；基于灵活且可扩展的事件模型，可以清晰地分离关注点；高度可定制的线程模型-单线程，一个或多个线程池。
2. 使用方便：详细记录的 Javadoc，用户指南和示例；没有其他依赖项，JDK5（Netty3.x）或 6（Netty4.x）就足够了。
3. **高性能、吞吐量更高：延迟更低；减少资源消耗；最小化不必要的内存复制**。
4. 安全：完整的 SSL/TLS 和 StartTLS 支持。
5. 社区活跃、不断更新：社区活跃，版本迭代周期短，发现的 Bug 可以被及时修复，同时，更多的新功能会被加入。

### 5.3 Netty版本

1. Netty 版本分为 Netty 3.x 和 Netty 4.x、Netty 5.x
   因为 Netty 5 出现重大 bug，已经被官网废弃了，目前推荐使用的是 Netty 4.x的稳定版本
2. 目前在官网可下载的版本 Netty 3.x、Netty 4.0.x 和 Netty 4.1.x

## 6. Netty高性能框架设计

### 6.1 线程模型

不同的线程模式，对程序的性能有很大影响。目前存在的线程模型有：传统阻塞 I/O 服务模型 和Reactor 模式。
根据 Reactor 的数量和处理资源池线程的数量不同，有 3 种典型的实现

1. 单 Reactor 单线程；
2. 单 Reactor多线程；
3. 主从 Reactor多线程

Netty 线程模式（Netty 主要基于主从 Reactor 多线程模型做了一定的改进，其中主从 Reactor 多线程模型有多个 Reactor）

### 6.2传统阻塞 I/O 服务模型

1. 工作原理图

* 黄色的框表示对象，蓝色的框表示线程
* 白色的框表示方法（`API`）

  ![1657204961226](image/netty/传统阻塞IO模型.png)

2. 模型特点

* 采用阻塞 `IO` 模式获取输入的数据
* **每个连接都需要独立的线程**完成数据的输入，业务处理，数据返回

3. 问题分析

* 当并发数很大，就会创建**大量的线程**，占用很大**系统资源**
* 连接创建后，如果当前线程暂时没有数据可读，该线程会**阻塞在 Handler对象中的 `read` 操作，导致上面的处理线程资源浪费**

### 6.3 Reactor 模式

#### 6.3.1 针对传统阻塞 I/O 服务模型的 2 个缺点，解决方案：

1. 基于 I/O 复用模型：**多个连接共用一个阻塞对象ServiceHandler**，应用程序只需要在一个阻塞对象等待，无需阻塞等待所有连接。当某个连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理。

   Reactor 在不同书中的叫法：

   反应器模式

   分发者模式（Dispatcher）

   通知者模式（notifier）
2. 基于线程池复用线程资源：不必再为每个连接创建线程，将连接完成后的业务处理任务分配给线程进行处理，一个线程可以处理多个连接的业务。（解决了当并发数很大时，会创建大量线程，占用很大系统资源）
3. 基于 I/O 复用模型：多个客户端进行连接，先把连接请求给ServiceHandler。多个连接共用一个阻塞对象ServiceHandler。假设，当C1连接没有数据要处理时，C1客户端只需要阻塞于ServiceHandler，C1之前的处理线程便可以处理其他有数据的连接，不会造成线程资源的浪费。当C1连接再次有数据时，ServiceHandler根据线程池的空闲状态，将请求分发给空闲的线程来处理C1连接的任务。（解决了线程资源浪费的那个问题）

![1657355095087](image/netty/Reactor IO 复用模型.png)

#### 6.3.2 I/O 复用结合线程池，就是 Reactor 模式基本设计思想

![1657355256233](image/netty/Reactor IO 复用模型2.png )

1. Reactor 模式，通过一个或多个输入同时传递给服务处理器（ServiceHandler）的模式（基于事件驱动）
2. 服务器端程序处理传入的多个请求,并将它们**同步分派**到相应的处理线程，因此 Reactor 模式也叫 Dispatcher 模式
3. Reactor 模式使用 **IO 复用监听事件**，收到事件后，分发给某个线程（进程），这点就是网络服务器高并发处理关键

#### 6.3.3 Reactor 模式中核心组成

1. Reactor（也就是那个ServiceHandler）：Reactor 在一个**单独的线程**中运行，**负责监听和分发事件**，分发给适当的处理线程来对 IO 事件做出反应。它就像公司的电话接线员，它接听来自客户的电话并将线路转移到适当的联系人；
2. Handlers（处理线程EventHandler）：处理线程**执行 I/O 事件要完成的实际事件**，类似于客户想要与之交谈的公司中的实际官员。Reactor 通过调度适当的处理线程来响应 I/O 事件，处理程序执行非阻塞操作。

#### 6.3.4 Reactor 模式分类

根据 `Reactor` 的数量和处理资源池线程的数量不同，有 `3` 种典型的实现

1. 单 `Reactor` 单线程
2. 单 `Reactor` 多线程
3. 主从 `Reactor` 多线程

**三种模式用生活案例来理解**

1. 单 `Reactor` 单线程，前台接待员和服务员是同一个人，全程为顾客服
2. 单 `Reactor` 多线程，`1` 个前台接待员，多个服务员，接待员只负责接待
3. 主从 `Reactor` 多线程，多个前台接待员，多个服务生

**Reactor 模式具有如下的优点**

1. 响应快，不必为单个同步时间所阻塞，虽然 Reactor 本身依然是同步的（比如你第一个SubReactor阻塞了，我可以调下一个 SubReactor为客户端服务）
2. 可以最大程度的避免复杂的多线程及同步问题，并且避免了多线程/进程的切换开销
3. 扩展性好，可以方便的通过增加 Reactor 实例个数来充分利用 CPU 资源
4. 复用性好，Reactor 模型本身与具体事件处理逻辑无关，具有很高的复用性

### 6.4 单 Reactor 单线程

![1657355863394](image/netty/单 Reactor 单线程.png )

**1. 方案说明**

- Select 是前面 I/O 复用模型介绍的标准网络编程 API，可以实现应用程序通过一个阻塞对象监听多路连接请求
- Reactor 对象通过 Select 监控客户端请求事件，收到事件后通过 Dispatch 进行分发
- 如果是建立连接请求事件，则由 Acceptor 通过 Accept 处理连接请求，然后创建一个 Handler 对象处理连接完成后的后续业务处理
- 如果不是建立连接事件，则 Reactor 会分发调用连接对应的 Handler 来响应
  Handler 会完成 Read → 业务处理 → Send 的完整业务流程

结合实例：服务器端用一个线程通过多路复用搞定所有的 `IO` 操作（包括连接，读、写等），编码简单，清晰明了，但是如果客户端连接数量较多，将无法支撑，前面的 `NIO` 案例就属于这种模型。

**2. 方案优缺点分析**

- 优点：模**型简单**，没有多线程、进程通信、竞争的问题，全部都在一个线程中完成
- 缺点：**性能问题**，只有一个线程，无法完全发挥多核 CPU 的性能。Handler在处理某个连接上的业务时，整个进程无法处理其他连接事件，很容易导致性能瓶颈
- 缺点：可靠性问题，线程意外终止，或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障
- 使用场景：**客户端的数量有限，业务处理非常快速**，比如 Redis 在业务处理的时间复杂度 O(1) 的情况

### 6.5 单Reactor多线程

![1657357271728](image/netty/单Reactor多线程模型.png)

**1. 方案说明**

- Reactor 对象通过 Select 监控客户端请求事件，收到事件后，通过 Dispatch 进行分发
- 如果是建立连接请求，则由 Acceptor 通过 accept 处理连接请求，然后创建一个 Handler 对象处理完成连接后的各种事件
- 如果不是连接请求，则由 Reactor 分发调用连接对应的 handler 来处理（也就是说连接已经建立，后续客户端再来请求，那基本就是数据请求了，直接调用之前为这个连接创建好的handler来处理）
- handler 只负责响应事件，不做具体的业务处理（这样不会使handler阻塞太久），通过 read 读取数据后，会分发给后面的 worker 线程池的某个线程处理业务。**【业务处理是最费时的，所以将业务处理交给线程池去执行】**
- worker 线程池会分配独立线程完成真正的业务，并将结果返回给 handler
- handler 收到响应后，通过 send 将结果返回给 client

**2.方案优缺点**

* 优点：可以充分的利用多核 `cpu` 的处理能力
* 缺点：多线程数据共享和访问比较复杂。`Reactor` 承担所有的事件的监听和响应，它是单线程运行，在高并发场景容易出现性能瓶颈。也就是说 **`Reactor`主线程承担了过多的事**

### 6.6 主从 Reactor 多线程

![1657418702126](image/netty/主从 Reactor 多线程.png)

**1.工作原理**

- Reactor 主线程 MainReactor 对象通过 select 监听连接事件，收到事件后，通过 Acceptor 处理连接事件
- 当 Acceptor 处理连接事件后，**MainReactor 将连接分配给 SubReactor**
- subreactor 将连接加入到连接队列进行监听，并创建 handler 进行各种事件处理
- 当有新事件发生时，subreactor 就会调用对应的 handler 处理
- handler 通过 read 读取数据，分发给后面的 **worker 线程**处理
- worker 线程池分配独立的 worker 线程进行业务处理，并返回结果
- handler 收到响应的结果后，再通过 send 将结果返回给 client
- **Reactor 主线程可以对应多个 Reactor 子线程，即 MainRecator 可以关联多个 SubReactor**

**2.优缺点**

- 优点：父线程与子线程的数据交互简单职责明确，**父线程只需要接收新连接**，**子线程完成后续的业务处理**。
- 优点：父线程与子线程的数据交互简单，**`Reactor` 主线程只需要把新连接传给子线程，子线程无需返回数据**。
- 缺点：编程复杂度较高

结合实例：这种模型在许多项目中广泛使用，**包括 `Nginx` 主从 `Reactor` 多进程模型，`Memcached` 主从多线程，`Netty` 主从多线程模型的支持**

## 7. Netty模型

### 7.1 工作原理示意图1 - 简单版

![1657420808916](image/netty/Netty 工作原理示意图1 - 简单版.png )

`Netty` 主要基于主从 `Reactors` 多线程模型（如图）做了一定的改进，其中主从 `Reactor` 多线程模型有多个 `Reactor`

1. BossGroup 线程维护 Selector，**只关注 Accecpt**
2. 当接收到 Accept 事件，获取到对应的 SocketChannel，封装成 NIOScoketChannel 并注册到 Worker 线程（事件循环），并进行维护
3. 当 Worker 线程监听到 Selector 中通道发生自己感兴趣的事件后，就进行处理（就由 handler），注意 handler 已经加入到通道

### 7.2 工作原理示意图2 - 进阶版

![1657421109043](image/netty/Netty 工作原理示意图2 - 进阶版.png )

`BossGroup`有点像主 `Reactor` 可以有多个，`WorkerGroup`则像 `SubReactor`一样可以有多个。

### 7.3 工作原理示意图3 - 详细版

![1657421190846](image/netty/Netty 工作原理示意图3 - 详细版.png )

1. Netty 抽象出两组线程池 ，**BossGroup 专门负责接收客户端的连接，WorkerGroup 专门负责网络的读写**
2. BossGroup 和 WorkerGroup 类型都是 NioEventLoopGroup
3. NioEventLoopGroup 相当于一个事件循环组，这个组中含有多个事件循环，每一个事件循环是 NioEventLoop
4. NioEventLoop 表示一个不断循环的执行处理任务的线程，**每个 NioEventLoop 都有一个 Selector，用于监听绑定在其上的 socket 的网络通讯**
5. NioEventLoopGroup 可以有多个线程，即可以含有多个 NioEventLoop
6. 每个 BossGroup下面的NioEventLoop 循环执行的步骤有 3 步
   - 轮询 accept 事件
   - **处理 accept 事件**，与 client 建立连接，生成 NioScocketChannel，并将其注册到某个 workerGroup NIOEventLoop 上的 Selector
   - 继续处理任务队列的任务，即 runAllTasks
7. 每个 WorkerGroup NIOEventLoop 循环执行的步骤
   - 轮询 read，write 事件
   - **处理 I/O 事件，即 read，write 事件**，在对应 NioScocketChannel 处理
   - 处理任务队列的任务，即 runAllTasks
8. 每个 Worker NIOEventLoop 处理业务时，会使用 pipeline（管道），pipeline 中包含了 channel（通道），即通过 pipeline 可以获取到对应通道，管道中维护了很多的处理器。（这个点目前只是简单的讲，后面重点说）

### 7.4 Netty快速入门

- NettyServer

  ```java
  package com.codeman.netty.netty.simple;

  import io.netty.bootstrap.ServerBootstrap;
  import io.netty.channel.ChannelFuture;
  import io.netty.channel.ChannelInitializer;
  import io.netty.channel.ChannelOption;
  import io.netty.channel.EventLoopGroup;
  import io.netty.channel.nio.NioEventLoopGroup;
  import io.netty.channel.socket.SocketChannel;
  import io.netty.channel.socket.nio.NioServerSocketChannel;
  import io.netty.util.concurrent.GenericFutureListener;

  import java.net.InetSocketAddress;

  /**
   * @author: zhanghongjie
   * @description:
   * @date: 2022/7/10 11:17
   * @version: 1.0
   */
  public class SimpleNettyServer {

      public static void main(String[] args) {
          //1. 创建两个线程组 bossGroup 和 workerGroup
          //2. bossGroup 只是处理连接请求 , 真正的和客户端业务处理，会交给 workerGroup完成
          //3. 两个都是无限循环
          //4. bossGroup 和 workerGroup 含有的子线程(NioEventLoop)的个数
          //   默认实际 cpu核数 * 2 SystemPropertyUtil.getInt("io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2)
          EventLoopGroup bossGroup = new NioEventLoopGroup(1);
          EventLoopGroup workGroup = new NioEventLoopGroup();

          try {
              //创建服务器端的启动对象，配置参数
              ServerBootstrap bootstrap = new ServerBootstrap();
              bootstrap.group(bossGroup, workGroup) //设置两个线程组
                      .channel(NioServerSocketChannel.class) // 使用NioServerSocketChannel 作为服务器的通道实现
                      .option(ChannelOption.SO_BACKLOG, 128) // 设置线程队列得到连接个数
                      .childOption(ChannelOption.SO_KEEPALIVE, true) //设置保持活动连接状态
                      .childHandler(new ChannelInitializer<SocketChannel>(){
                          //给pipeline 设置处理器
                          @Override
                          protected void initChannel(SocketChannel ch) throws Exception {
                              ch.pipeline().addLast(new CusServerChannleHandler());
                          }
                      });

              System.out.println(".....服务器 is ready...");

              //绑定一个端口并且同步, 生成了一个 ChannelFuture 对象
              //启动服务器(并绑定端口)
              ChannelFuture channelFuture = bootstrap.bind(new InetSocketAddress("localhost", 6677));
              System.out.println("future " + channelFuture);

              //给cf 注册监听器，监控我们关心的事件
              channelFuture.addListener((GenericFutureListener) future -> {
                  System.out.println("future inner " + future); // 和外面是同一个
                  if (future.isSuccess()) {
                      System.out.println("监听端口 6668 成功");
                  } else {
                      System.out.println("监听端口 6668 失败");
                  }
              });
              //给关闭通道进行监听
              channelFuture.channel().closeFuture().sync();

          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              bossGroup.shutdownGracefully();
              workGroup.shutdownGracefully();
          }
      }
  }

  ```
- NettyServerHandler

  ```java
  package com.codeman.netty.netty.simple;

  import io.netty.buffer.ByteBuf;
  import io.netty.buffer.Unpooled;
  import io.netty.channel.Channel;
  import io.netty.channel.ChannelHandlerContext;
  import io.netty.channel.ChannelInboundHandlerAdapter;
  import io.netty.channel.ChannelPipeline;
  import io.netty.util.CharsetUtil;

  /**
   * @author: zhanghongjie
   * @description:
   * @date: 2022/7/10 11:31
   * @version: 1.0
   */
  public class CusServerChannleHandler extends ChannelInboundHandlerAdapter {

      @Override
      public void channelActive(ChannelHandlerContext ctx) throws Exception {
          System.out.println("CusServerChannleHandler  ==  channelActive");
  //        super.channelActive(ctx);
      }

      @Override
      public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
          ByteBuf byteBuf = (ByteBuf) msg;
          System.out.println("CusServerChannleHandler  ==  channelRead， msg=" + byteBuf.toString(CharsetUtil.UTF_8));
          Channel channel = ctx.channel();
          ChannelPipeline pipeline = ctx.pipeline(); //本质是一个双向链接, 出站入站
          System.out.println("客户端地址:" + channel.remoteAddress());


  	// 比如这里我们有一个非常耗时长的业务-> 异步执行 -> 提交该channel 对应的
          // NIOEventLoop 的 taskQueue中,

          // 解决方案1 用户程序自定义的普通任务

          ctx.channel().eventLoop().execute(new Runnable() {
              @Override
              public void run() {

                  try {
                      Thread.sleep(5 * 1000);
                      ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵2", CharsetUtil.UTF_8));
                      System.out.println("channel code=" + ctx.channel().hashCode());
                  } catch (Exception ex) {
                      System.out.println("发生异常" + ex.getMessage());
                  }
              }
          });

          ctx.channel().eventLoop().execute(new Runnable() {
              @Override
              public void run() {

                  try {
                      Thread.sleep(5 * 1000);
                      ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵3", CharsetUtil.UTF_8));
                      System.out.println("channel code=" + ctx.channel().hashCode());
                  } catch (Exception ex) {
                      System.out.println("发生异常" + ex.getMessage());
                  }
              }
          });

          //解决方案2 : 用户自定义定时任务 -》 该任务是提交到 scheduleTaskQueue中

          ctx.channel().eventLoop().schedule(new Runnable() {
              @Override
              public void run() {

                  try {
                      Thread.sleep(5 * 1000);
                      ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵4", CharsetUtil.UTF_8));
                      System.out.println("channel code=" + ctx.channel().hashCode());
                  } catch (Exception ex) {
                      System.out.println("发生异常" + ex.getMessage());
                  }
              }
          }, 5, TimeUnit.SECONDS);

          ctx.writeAndFlush(Unpooled.copiedBuffer("这里是服务端，你好客户端，消息已收到：" + byteBuf.toString(CharsetUtil.UTF_8), CharsetUtil.UTF_8));
  //        super.channelRead(ctx, msg);
      }

      @Override
      public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
          System.out.println("CusServerChannleHandler  ==  channelReadComplete");
  //        super.channelReadComplete(ctx);
      }

      @Override
      public void channelWritabilityChanged(ChannelHandlerContext ctx) throws Exception {
  //        super.channelWritabilityChanged(ctx);
      }

      @Override
      public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
  //        super.exceptionCaught(ctx, cause);
          ctx.close();
      }
  }

  ```
- NettyClient

  ```java
  package com.codeman.netty.netty.simple;

  import io.netty.bootstrap.Bootstrap;
  import io.netty.channel.Channel;
  import io.netty.channel.ChannelFuture;
  import io.netty.channel.ChannelInitializer;
  import io.netty.channel.EventLoopGroup;
  import io.netty.channel.nio.NioEventLoopGroup;
  import io.netty.channel.socket.nio.NioSocketChannel;

  import java.net.InetSocketAddress;

  /**
   * @author: zhanghongjie
   * @description:
   * @date: 2022/7/10 11:48
   * @version: 1.0
   */
  public class SimpleNettyClient {

      public static void main(String[] args) {
          //客户端需要一个事件循环组
          EventLoopGroup workGroup = new NioEventLoopGroup();

          try {
              //创建客户端启动对象
              //注意客户端使用的不是 ServerBootstrap 而是 Bootstrap
              Bootstrap bootstrap = new Bootstrap();
              bootstrap.group(workGroup)
                      .channel(NioSocketChannel.class) // 设置客户端通道的实现类(反射)，和服务端不一样
                      .handler(new ChannelInitializer() {
                          @Override
                          protected void initChannel(Channel ch) throws Exception {
                              ch.pipeline().addLast(new CusClientChannelHandler()); //加入自己的处理器
                          }
                      });

              System.out.println("客户端ok");
              //启动客户端去连接服务器端
              //关于 ChannelFuture 要分析，涉及到netty的异步模型
              ChannelFuture channelFuture = bootstrap.connect(new InetSocketAddress("localhost", 6677));
              //给关闭通道进行监听
              channelFuture.channel().closeFuture().sync();
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              workGroup.shutdownGracefully();
          }
      }
  }

  ```
- NettyClientHandler

  ```java
  package com.codeman.netty.netty.simple;

  import io.netty.buffer.ByteBuf;
  import io.netty.buffer.Unpooled;
  import io.netty.channel.ChannelHandlerContext;
  import io.netty.channel.ChannelInboundHandlerAdapter;
  import io.netty.util.CharsetUtil;

  /**
   * @author: zhanghongjie
   * @description:
   * @date: 2022/7/10 11:55
   * @version: 1.0
   */
  public class CusClientChannelHandler extends ChannelInboundHandlerAdapter {

      @Override
      public void channelActive(ChannelHandlerContext ctx) throws Exception {
          System.out.println("CusClientChannelHandler === channelActive");
          ctx.writeAndFlush(Unpooled.copiedBuffer("你好，这里是客户端", CharsetUtil.UTF_8));
  //        super.channelActive(ctx);
      }

      @Override
      public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
          ByteBuf byteBuf = (ByteBuf) msg;
          System.out.println("CusClientChannelHandler === channelRead  === msg = " + byteBuf.toString(CharsetUtil.UTF_8));
  //        super.channelRead(ctx, msg);
      }

      @Override
      public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
          System.out.println("CusClientChannelHandler === channelReadComplete");
  //        super.channelReadComplete(ctx);
      }

      @Override
      public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
  //        super.exceptionCaught(ctx, cause);
          ctx.close();
      }
  }

  ```

1. Netty 抽象出两组线程池，BossGroup 专门负责接收客户端连接，WorkerGroup 专门负责网络读写操作。
2. NioEventLoop 表示一个不断循环执行处理任务的线程，每个 NioEventLoop 都有一个 Selector，用于监听绑定在其上的 socket网络通道。
3. NioEventLoop 内部采用串行化设计，从消息的 读取->解码->处理->编码->发送，始终由 IO 线程 NioEventLoop 负责

- NioEventLoopGroup 下包含多个 NioEventLoop
- 每个 NioEventLoop 中包含有一个 Selector，一个 taskQueue
- 每个 NioEventLoop 的 Selector 上可以注册监听多个 NioChannel
- 每个 NioChannel 只会绑定在唯一的 NioEventLoop 上
- 每个 NioChannel 都绑定有一个自己的 ChannelPipeline

### 7.5 异步模型

#### 7.5.1 基本介绍

1. 异步的概念和同步相对。当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的组件在完成后，通过状态、通知和回调来通知调用者。
2. Netty 中的 I/O 操作是异步的，包括 Bind、Write、Connect 等操作会首先简单的返回一个 ChannelFuture。
3. 调用者并不能立刻获得结果，而是通过 Future-Listener 机制，用户可以方便的主动获取或者通过通知机制获得 IO 操作结果。
4. **Netty 的异步模型是建立在 future 和 callback 的之上的**。callback 就是回调。重点说 Future，它的核心思想是：假设一个方法 fun，计算过程可能非常耗时，等待 fun 返回显然不合适。那么可以在调用 fun 的时候，立马返回一个 Future，后续可以通过 Future 去监控方法 fun 的处理过程（即：Future-Listener 机制）

#### 7.5.2 Future说明

1. 表示异步的执行结果,可以通过它提供的方法来检测执行是否完成，比如检索计算等等。
2. `ChannelFuture` 是一个接口：`public interface ChannelFuture extends Future<Void>` 我们可以添加监听器，当监听的事件发生时，就会通知到监听器。

#### 7.5.3 工作原理说明

![1657548853970](image/netty/工作原理工作原理1.png)

![1657548891863](image/netty/工作原理工作原理2.png)

1. 在使用 `Netty` 进行编程时，拦截操作和转换出入站数据**只需要您提供 `callback` 或利用 `future` **即可。这使得链式操作简单、高效，并有利于编写可重用的、通用的代码。
2. `Netty` 框架的目标就是让你的业务逻辑从网络基础应用编码中分离出来、解脱出来。

#### 7.5.4 Future-Listener 机制

1. 当 Future 对象刚刚创建时，处于非完成状态，调用者可以通过返回的 ChannelFuture 来获取操作执行的状态，注册监听函数来执行完成后的操作。
2. 常见有如下操作
   - 通过 **isDone** 方法来判断当前操作是否完成；
   - 通过 **isSuccess** 方法来判断已完成的当前操作是否成功；
   - 通过 **getCause** 方法来获取已完成的当前操作失败的原因；
   - 通过 **isCancelled** 方法来判断已完成的当前操作是否被取消；
   - 通过 **addListener** 方法来注册监听器，当操作已完成（isDone方法返回完成），将会通知指定的监听器；如果 Future 对象已完成，则通知指定的监听器

```java
//绑定一个端口并且同步,生成了一个ChannelFuture对象
//启动服务器(并绑定端口)
ChannelFuture cf = bootstrap.bind(6668).sync();
//给cf注册监听器，监控我们关心的事件
cf.addListener(new ChannelFutureListener() {
   @Override
   public void operationComplete (ChannelFuture future) throws Exception {
      if (cf.isSuccess()) {
         System.out.println("监听端口6668成功");
      } else {
         System.out.println("监听端口6668失败");
      }
   }
});
```

#### 7.5.4 实例 - http服务

```java
package com.codeman.netty.netty.httpserver;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.epoll.EpollEventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

import java.net.InetSocketAddress;

/**
 * @author: zhanghongjie
 * @description:
 * @date: 2022/7/11 22:38
 * @version: 1.0
 */
public class HttpServer {

    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            ChannelFuture channelFuture
                    = serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new HttpServerInitializar())
                    .bind(new InetSocketAddress("localhost", 8878));
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

```java
package com.codeman.netty.netty.httpserver;

import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
<<<<<<< HEAD
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.ssl.SslContext;
import io.netty.handler.ssl.SslContextBuilder;
import io.netty.handler.ssl.SslHandler;

import javax.net.ssl.KeyManagerFactory;
import javax.net.ssl.SSLEngine;
import java.io.FileInputStream;
import java.security.KeyStore;
=======
import io.netty.handler.codec.http.HttpServerCodec;
>>>>>>> 5457daad7a4833b765005ec3df770ce3c852d97d


/**
 * @author: zhanghongjie
 * @description:
 * @date: 2022/7/11 22:52
 * @version: 1.0
 */
public class HttpServerInitializar extends ChannelInitializer<SocketChannel> {

<<<<<<< HEAD
    private final SslContext sslContext;

    public HttpServerInitializar() {
        String keyStoreFilePath = "/root/.ssl/test.pkcs12";
        String keyStorePassword = "Password@123";

        try {
            KeyStore keyStore = KeyStore.getInstance("PKCS12");
            keyStore.load(new FileInputStream(keyStoreFilePath), keyStorePassword.toCharArray());

            KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
            keyManagerFactory.init(keyStore, keyStorePassword.toCharArray());

            sslContext = SslContextBuilder.forServer(keyManagerFactory).build();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

=======
>>>>>>> 5457daad7a4833b765005ec3df770ce3c852d97d
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();

        //加入一个netty 提供的httpServerCodec codec =>[coder - decoder]
        //HttpServerCodec 说明
        //1. HttpServerCodec 是netty 提供的处理http的 编-解码器
        pipeline.addLast("HttpServerCodec", new HttpServerCodec());
        pipeline.addLast("HttpServerHandler", new HttpServerHandler());
<<<<<<< HEAD
        // aggregator，消息聚合器（重要）。为什么能有FullHttpRequest这个东西，就是因为有他，HttpObjectAggregator，如果没有他，就不会有那个消息是FullHttpRequest的那段Channel，同样也不会有FullHttpResponse。
        //如果我们将z'h
        //HttpObjectAggregator(512 * 1024)的参数含义是消息合并的数据大小，如此代表聚合的消息内容长度不超过512kb。
        pipeline.addLast("aggregator", new HttpObjectAggregator(512 * 1024));
        // 构建HTTPS服务
        SSLEngine sslEngine = sslContext.newEngine(ch.alloc());
        pipeline.addLast(new SslHandler(sslEngine));
=======
>>>>>>> 5457daad7a4833b765005ec3df770ce3c852d97d
    }
}

```

```java
package com.codeman.netty.netty.httpserver;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.handler.codec.http.DefaultFullHttpResponse;
import io.netty.handler.codec.http.FullHttpResponse;
import io.netty.handler.codec.http.HttpHeaderNames;
import io.netty.handler.codec.http.HttpObject;
import io.netty.handler.codec.http.HttpRequest;
import io.netty.handler.codec.http.HttpResponseStatus;
import io.netty.handler.codec.http.HttpVersion;
import io.netty.util.CharsetUtil;

import java.nio.charset.Charset;

/**
 * @author: zhanghongjie
 * @description:
 * @date: 2022/7/11 22:57
 * @version: 1.0
 */
public class HttpServerHandler extends SimpleChannelInboundHandler<HttpObject> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {

        if (msg instanceof HttpRequest) {
            System.out.println("客户端地址" + ctx.channel().remoteAddress());
            HttpRequest httpRequest = (HttpRequest) msg;
            System.out.println("httpRequest url: " + httpRequest.uri());

            //回复信息给浏览器 [http协议]
            ByteBuf content = Unpooled.copiedBuffer("hello, 我是服务器",  Charset.forName("GBK"));

            //构造一个http的相应，即 httpresponse
            FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, content);
            response.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/plain");
            response.headers().set(HttpHeaderNames.CONTENT_LENGTH, content.readableBytes());

            ctx.writeAndFlush(response);
        }
    }
}

```

### 7.6 Netty 核心模块组件

<<<<<<< HEAD

#### 7.6.1 Bootstrap、ServerBootstrap

1. Bootstrap 意思是引导，一个 Netty 应用通常由一个 Bootstrap 开始，主要作用是配置整个 Netty 程序，串联各个组件，Netty 中 Bootstrap 类是客户端程序的启动引导类，ServerBootstrap 是服务端启动引导类。
2. 常见的方法有

- public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)，该方法用于服务器端，用来设置两个 EventLoop
- public B group(EventLoopGroup group)，该方法用于客户端，用来设置一个 EventLoop
- public B channel(Class<? extends C> channelClass)，该方法用来设置一个服务器端的通道实现
- public `<T>` B option(ChannelOption `<T>` option, T value)，用来给 ServerChannel 添加配置
- public `<T>` ServerBootstrap childOption(ChannelOption `<T>` childOption, T value)，用来给接收到的通道添加配置
- public ServerBootstrap childHandler(ChannelHandler childHandler)，该方法用来设置业务处理类（自定义的handler）
- public ChannelFuture bind(int inetPort)，该方法用于服务器端，用来设置占用的端口号
- public ChannelFuture connect(String inetHost, int inetPort)，该方法用于客户端，用来连接服务器端

#### 7.6.2 Future、ChannelFuture

**Netty 中所有的 IO 操作都是异步的**，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 Future 和 ChannelFutures，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件。

常见的方法有

- Channel channel()，返回当前正在进行 IO 操作的通道
- ChannelFuture sync()，等待异步操作执行完毕

#### 7.6.3 Channel

1. Netty 网络通信的组件，能够用于执行网络 I/O 操作。
2. 通过 Channel 可获得当前网络连接的通道的状态
3. 通过 Channel 可获得网络连接的配置参数（例如接收缓冲区大小）
4. Channel 提供异步的网络 I/O 操作(如建立连接，读写，绑定端口)，异步调用意味着任何 I/O 调用都将立即返回，并且不保证在调用结束时所请求的 I/O 操作已完成
5. 调用立即返回一个 ChannelFuture 实例，通过注册监听器到 ChannelFuture 上，可以 I/O 操作成功、失败或取消时回调通知调用方
6. 支持关联 I/O 操作与对应的处理程序
7. 不同协议、不同的阻塞类型的连接都有不同的 Channel 类型与之对应，常用的 Channel 类型：
   - **NioSocketChannel，异步的客户端 TCP Socket 连接。**
   - **NioServerSocketChannel，异步的服务器端 TCP Socket 连接。**
   - NioDatagramChannel，异步的 UDP 连接。
   - NioSctpChannel，异步的客户端 Sctp 连接。
   - NioSctpServerChannel，异步的 Sctp 服务器端连接，这些通道涵盖了 UDP 和 TCP 网络 IO 以及文件 IO。

#### 7.6.4 Selector

1. **Netty 基于 Selector 对象实现 I/O 多路复用**，通过 Selector 一个线程可以监听多个连接的 Channel 事件。
2. 当向一个 Selector 中注册 Channel 后，Selector 内部的机制就可以自动不断地查询（Select）这些注册的 Channel 是否有已就绪的 I/O 事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个线程高效地管理多个 Channel

#### 7.6.5 ChannelHandler 及其实现类

1. ChannelHandler 是一个接口，处理 I/O 事件或拦截 I/O 操作，并将其转发到其 ChannelPipeline（业务处理链）中的下一个处理程序。
2. ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类
3. ChannelHandler 及其实现类一览图

![1657724394216](image/netty/ChannelHandler 及其实现类.png )

4. 我们经常需要自定义一个 `Handler` 类去继承 `ChannelInboundHandlerAdapter`，然后通过重写相应方法实现业务逻辑，我们接下来看看一般都需要重写哪些方法

```java

	// chanel加入
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        super.handlerAdded(ctx);
    }
	// 通道就绪
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        super.channelActive(ctx);
    }

   	// 监听到读事件
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        super.channelRead(ctx, msg);
    }

    	// 读完成
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        super.channelReadComplete(ctx);
    }

  
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        super.exceptionCaught(ctx, cause);
    }
```

#### 7.6.6 Pipeline 和 ChannelPipeline

**ChannelPipeline 是一个重点：**

1. ChannelPipeline 是一个 **Handler 的集合**，它负责**处理和拦截 inbound 或者 outbound 的事件**和操作，相当于一个贯穿 Netty 的链。（也可以这样理解：ChannelPipeline 是保存 ChannelHandler 的 List，用于处理或拦截 Channel 的入站事件和出站操作）
2. ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 Channel 中各个的 ChannelHandler 如何相互交互
3. 在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下：

![1657724783433](image/netty/Channel ChannelPipeline Handle关系图.png )

- `TestServerInitializer`和 `HttpServerCodec`这些东西本身也是 `handler`

* 一般来说事件从客户端往服务器走我们称为出站，反之则是入站。

#### 7.6.7 ChannelHandlerContext

1. 保存 Channel 相关的所有上下文信息，同时关联一个 ChannelHandler 对象
2. 即 **ChannelHandlerContext 中包含一个具体的事件处理器 ChannelHandler**，同时 **ChannelHandlerContext 中也绑定了对应的 pipeline 和 Channel** 的信息，方便对 ChannelHandler 进行调用。
3. 常用方法
   - ChannelFuture close()，关闭通道
   - ChannelOutboundInvoker flush()，刷新
   - ChannelFuture writeAndFlush(Object msg)，将数据写到
   - ChannelPipeline 中当前 ChannelHandler 的下一个 ChannelHandler 开始处理（出站）

#### 7.6.8 ChannelOption

1. `Netty` 在创建 `Channel` 实例后，一般都需要设置 `ChannelOption` 参数。
2. `ChannelOption` 参数如下：

- ChannelOption.SO_BACKLOG 对应TCP/IP协议listen函数中的backlog参数，用来初始化服务器可连接队列大小。服务端处理客户端连接请求是顺序处理的，所以同一时间只能处理一个客户端连接。多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理，backlog参数指定了队列的大小。
- ChannelOption.SO_KEEPALIVE 一直保持连接活动状态

#### 7.6.9 EventLoopGroup 和其实现类 NioEventLoopGroup

1. EventLoopGroup 是一组 EventLoop 的抽象，Netty 为了更好的利用多核 CPU 资源，一般会有多个 EventLoop 同时工作**，每个 EventLoop 维护着一个 Selector 实例。**
2. EventLoopGroup 提供 next 接口，可以从组里面按照一定规则获取其中一个 EventLoop 来处理任务。在 Netty 服务器端编程中，我们一般都需要提供两个 EventLoopGroup，例如：BossEventLoopGroup 和 WorkerEventLoopGroup。
3. 通常一个服务端口即一个 ServerSocketChannel 对应一个 Selector 和一个 EventLoop 线程。BossEventLoop 负责接收客户端的连接并将 SocketChannel 交给 WorkerEventLoopGroup 来进行 IO 处理，如下图所示

![1657725303578](image/netty/EventLoopGroup示意图.png)

#### 7.6.10 Unpooled 类

1. `Netty` 提供一个专门用来操作缓冲区（即 `Netty` 的数据容器）的工具类
2. 常用方法:

public static ByteBuf copiedBuffer(CharSequence str, Charset charset); 通过给定的数据和字符编码返回一个ByteBuf对象，类似NIO的ByteBuff但有区别

**0 <= readerIndex <= writeIndex <= capacity**

#### 7.6.11 Netty 应用实例-群聊系统

在handlerAdded中，用ChannelGroup管理连接Server的ClientChannel

```java
public class CusServerChannleHandler extends ChannelInboundHandlerAdapter {

    //定义一个channle 组，管理所有的channel
    //GlobalEventExecutor.INSTANCE) 是全局的事件执行器，是一个单例
    private static ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        channelGroup.add(channel);
        channelGroup.writeAndFlush(Unpooled.copiedBuffer("【系统】【" + channel.remoteAddress() + "】加入了群聊\n", CharsetUtil.UTF_8));
    }


    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf byteBuf = (ByteBuf) msg;
        Channel channel = ctx.channel();
        channelGroup.forEach( channel1 -> {
           if (channel == channel1) {
               channel1.writeAndFlush(Unpooled.copiedBuffer("【自己】【" + channel.remoteAddress() + "】：" + byteBuf.toString(CharsetUtil.UTF_8), CharsetUtil.UTF_8));
           } else {
               channel1.writeAndFlush(Unpooled.copiedBuffer("【群友】【" + channel.remoteAddress() + "】：" + byteBuf.toString(CharsetUtil.UTF_8), CharsetUtil.UTF_8));
           }
        });

    }
}
```

### 7.7 Netty 心跳检测机制

IdleStateHandler + userEventTriggered

```java
	@Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ChannelPipeline pipeline = ch.pipeline();
                    //加入一个netty 提供 IdleStateHandler
                    /*
                    说明
                    1. IdleStateHandler 是netty 提供的处理空闲状态的处理器
                    2. long readerIdleTime : 表示多长时间没有读, 就会发送一个心跳检测包检测是否连接
                    3. long writerIdleTime : 表示多长时间没有写, 就会发送一个心跳检测包检测是否连接
                    4. long allIdleTime : 表示多长时间没有读写, 就会发送一个心跳检测包检测是否连接

                    5. 文档说明
                    triggers an {@link IdleStateEvent} when a {@link Channel} has not performed
                   read, write, or both operation for a while.
                    6. 当 IdleStateEvent 触发后 , 就会传递给管道 的下一个handler去处理，通过调用(触发)
                   下一个handler 的 userEventTiggered , 在该方法中去处理 IdleStateEvent(读空闲，写空闲，读写空闲)
                    7.handlerRemoved有时候是无法感知连接断掉，所以还是需要心跳包的检测来判断连接是否还有效
                     */
                    pipeline.addLast(new IdleStateHandler(3,5,7, TimeUnit.SECONDS));
                    //加入一个对空闲检测进一步处理的handler(自定义)
                    pipeline.addLast(new MyServerHandler());
                }



public class MyServerHandler extends ChannelInboundHandlerAdapter {

    /**
     *
     * @param ctx 上下文
     * @param evt 事件
     * @throws Exception
     */
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {

        if(evt instanceof IdleStateEvent) {

            //将  evt 向下转型 IdleStateEvent
            IdleStateEvent event = (IdleStateEvent) evt;
            String eventType = null;
            switch (event.state()) {
                case READER_IDLE:
                  eventType = "读空闲";
                  break;
                case WRITER_IDLE:
                    eventType = "写空闲";
                    break;
                case ALL_IDLE:
                    eventType = "读写空闲";
                    break;
            }
            System.out.println(ctx.channel().remoteAddress() + "--超时时间--" + eventType);
            System.out.println("服务器做相应处理..");

            //如果发生空闲，我们关闭通道
           // ctx.channel().close();
        }
    }
}
```

### 7.8 Netty 通过 WebSocket 编程实现服务器和客户端长连接

```java
package com.codeman.netty.netty.websocket;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.codec.http.websocketx.WebSocketServerProtocolHandler;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.handler.stream.ChunkedWriteHandler;

import java.net.InetSocketAddress;

/**
 * @author: zhanghongjie
 * @description:
 * @date: 2022/7/17 14:55
 * @version: 1.0
 */
public class WebSocketServer {

    public static void main(String[] args) {

        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap serverBootstrap
                    = new ServerBootstrap()
                    .group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel sc) throws Exception {

                            ChannelPipeline pipeline = sc.pipeline();
                            //因为基于http协议，使用http的编码和解码器
                            pipeline.addLast(new HttpServerCodec());
                            //http是以块方式写，添加ChunkedWriteHandler处理器
                            pipeline.addLast(new ChunkedWriteHandler());

                            /*
                            说明
                            1. http数据在传输过程中是分段, HttpObjectAggregator ，就是可以将多个段聚合
                            2. 这就就是为什么，当浏览器发送大量数据时，就会发出多次http请求
                             */
                            pipeline.addLast(new HttpObjectAggregator(8192));
                            /*
                            说明
                            1. 对应websocket ，它的数据是以 帧(frame) 形式传递
                            2. 可以看到WebSocketFrame 下面有六个子类
                            3. 浏览器请求时 ws://localhost:7000/hello 表示请求的uri
                            4. WebSocketServerProtocolHandler 核心功能是将 http协议升级为 ws协议 , 保持长连接
                            5. 是通过一个 状态码 101
                             */
                            pipeline.addLast(new WebSocketServerProtocolHandler("/hello"));

                            //自定义的handler ，处理业务逻辑
                            pipeline.addLast(new MyTextWebSocketFrameHandler());
                        }
                    });
            ChannelFuture channelFuture = serverBootstrap.bind(new InetSocketAddress("localhost", 7000));

            channelFuture.channel().closeFuture().sync();


        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }
}


package com.codeman.netty.netty.websocket;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;

import java.time.LocalDateTime;

//这里 TextWebSocketFrame 类型，表示一个文本帧(frame)
public class MyTextWebSocketFrameHandler extends SimpleChannelInboundHandler<TextWebSocketFrame>{
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {

        System.out.println("服务器收到消息 " + msg.text());

        //回复消息
        ctx.channel().writeAndFlush(new TextWebSocketFrame("服务器时间" + LocalDateTime.now() + " " + msg.text()));
    }

    //当web客户端连接后， 触发方法
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        //id 表示唯一的值，LongText 是唯一的 ShortText 不是唯一
        System.out.println("handlerAdded 被调用" + ctx.channel().id().asLongText());
        System.out.println("handlerAdded 被调用" + ctx.channel().id().asShortText());
    }


    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {

        System.out.println("handlerRemoved 被调用" + ctx.channel().id().asLongText());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        System.out.println("异常发生 " + cause.getMessage());
        ctx.close(); //关闭连接
    }
}

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script>
    var socket;
    //判断当前浏览器是否支持websocket
    if(window.WebSocket) {
        //go on
        socket = new WebSocket("ws://localhost:7000/hello");
        //相当于channelReado, ev 收到服务器端回送的消息
        socket.onmessage = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value = rt.value + "\n" + ev.data;
        }

        //相当于连接开启(感知到连接开启)
        socket.onopen = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value = "连接开启了.."
        }

        //相当于连接关闭(感知到连接关闭)
        socket.onclose = function (ev) {

            var rt = document.getElementById("responseText");
            rt.value = rt.value + "\n" + "连接关闭了.."
        }
    } else {
        alert("当前浏览器不支持websocket")
    }

    //发送消息到服务器
    function send(message) {
        if(!window.socket) { //先判断socket是否创建好
            return;
        }
        if(socket.readyState == WebSocket.OPEN) {
            //通过socket 发送消息
            socket.send(message)
        } else {
            alert("连接没有开启");
        }
    }
</script>
<form onsubmit="return false">
    <textarea name="message" style="height: 300px; width: 300px"></textarea>
    <input type="button" value="发生消息" onclick="send(this.form.message.value)">
    <textarea id="responseText" style="height: 300px; width: 300px"></textarea>
    <input type="button" value="清空内容" onclick="document.getElementById('responseText').value=''">
</form>
</body>
</html>

```

## 8. 编码和解码

### 8.1 编码和解码的基本介绍

编写网络应用程序时，因为数据在**网络中传输的都是二进制字节码数据**，在发送数据时就需要编码，接收数据时就需要解码
codec（编解码器）的组成部分有两个：decoder（解码器）和 encoder（编码器）。encoder 负责把业务数据转换成字节码数据，decoder 负责把字节码数据转换成业务数据

![1658064789778](image/netty/编码和解码.png)

### 8.2 Netty 本身的编码解码的机制和问题分析

Netty 自身提供了一些 codec(编解码器)

1. Netty 提供的编码器
   - StringEncoder：对字符串数据进行编码。
   - ObjectEncoder：对Java对象进行编码。
2. Netty 提供的解码器
   - StringDecoder,对字符串数据进行解码
   - ObjectDecoder，对 Java 对象进行解码
3. Netty 本身自带的 ObjectDecoder 和 ObjectEncoder 可以用来实现 POJO 对象或各种业务对象的编码和解码，底层使用的仍是Java序列化技术,而Java序列化技术本身效率就不高，存在如下问题
   - 无法跨语言
   - 序列化后的体积太大，是二进制编码的5倍多。
   - 序列化性能太低
4. 引出新的解决方案[Google 的 Protobuf]

### 8.3 Google Protobuf

**Protobuf 基本介绍和使用示意图**

1. Protobuf 是 Google 发布的开源项目，全称 Google Protocol Buffers，是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC [远程过程调用 remote procedure call ]数据交换格式。目前很多公司 从http + json 转向tcp + protobuf，效率会更高。
2. 参考文档：https://developers.google.com/protocol-buffers/docs/proto 语言指南
3. Protobuf 是以 message 的方式来管理数据的.
4. 支持跨平台、跨语言，即[客户端和服务器端可以是不同的语言编写的]（支持目前绝大多数语言，例如 C++、C#、Java、python 等）
5. 高性能，高可靠性
6. 使用 protobuf 编译器能自动生成代码，Protobuf 是将类的定义使用 .proto 文件进行描述。说明，在 idea 中编写 .proto 文件时，会自动提示是否下载 .ptoto 编写插件.可以让语法高亮。然后通过 protoc.exe 编译器根据 .proto 自动生成 .java 文件
   ![1658064928122](image/netty/Protobuf示意图.png)

### 8.4 Netty 编解码器和 Handler调用机制

#### 8.4.1 基本说明

1. Netty 的组件设计：Netty 的主要组件有 Channel、EventLoop、ChannelFuture、ChannelHandler、ChannelPipe 等
2. ChannelHandler 充当了处理入站和出站数据的应用程序逻辑的容器。例如，实现 ChannelInboundHandler 接口（或 ChannelInboundHandlerAdapter），你就可以接收入站事件和数据，这些数据会被业务逻辑处理。当要给客户端发送响应时，也可以从 ChannelInboundHandler 冲刷数据。业务逻辑通常写在一个或者多个 ChannelInboundHandler 中。ChannelOutboundHandler 原理一样，只不过它是用来处理出站数据的
3. ChannelPipeline 提供了 ChannelHandler 链的容器。以客户端应用程序为例，如果事件的运动方向是从客户端到服务端的，那么我们称这些事件为出站的，即客户端发送给服务端的数据会通过 pipeline 中的一系列 ChannelOutboundHandler，并被这些 Handler 处理，反之则称为入站的

#### 8.4.2 编码解码器

1. 当 Netty 发送或者接受一个消息的时候，就将会发生一次数据转换。入站消息会被解码：从字节转换为另一种格式（比如 java 对象）；如果是出站消息，它会被编码成字节。
2. Netty 提供一系列实用的编解码器，他们都实现了 ChannelInboundHadnler 或者 ChannelOutboundHandler 接口。在这些类中，channelRead 方法已经被重写了。以入站为例，对于每个从入站 Channel 读取的消息，这个方法会被调用。随后，它将调用由解码器所提供的 decode() 方法进行解码，并将已经解码的字节转发给 ChannelPipeline 中的下一个 ChannelInboundHandler。

#### 8.4.3 解码器 - ByteToMessageDecoder

#### 8.4.4 解码器 - ReplayingDecoder

1. public abstract class ReplayingDecoder `<S>` extends ByteToMessageDecoder
2. ReplayingDecoder 扩展了 ByteToMessageDecoder 类，使用这个类，我们不必调用 readableBytes() 方法，也就不用判断还有没有足够的数据来读取。参数 S 指定了用户状态管理的类型，其中 Void 代表不需要状态管理\
3. ReplayingDecoder 使用方便，但它也有一些局限性：
   - 并不是所有的 ByteBuf 操作都被支持，如果调用了一个不被支持的方法，将会抛出一个 UnsupportedOperationException。
   - ReplayingDecoder 在某些情况下可能稍慢于 ByteToMessageDecoder，例如网络缓慢并且消息格式复杂时，消息会被拆成了多个碎片，速度变慢

```java
package com.atguigu.netty.inboundhandlerandoutboundhandler;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.ReplayingDecoder;

import java.util.List;

public class MyByteToLongDecoder2 extends ReplayingDecoder<Void> {
  
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        System.out.println("MyByteToLongDecoder2 被调用");
        //在 ReplayingDecoder 不需要判断数据是否足够读取，内部会进行处理判断
        out.add(in.readLong());
    }
}
```

#### 8.4.5 其他解码器 MessageToByteEncoder子类

- LineBasedFrameDecoder：这个类在 Netty 内部也有使用，它使用行尾控制字符（\n或者\r\n）作为分隔符来解析数据。
- DelimiterBasedFrameDecoder：使用自定义的特殊字符作为消息的分隔符。
- HttpObjectDecoder：一个 HTTP 数据的解码器
- LengthFieldBasedFrameDecoder：通过指定长度来标识整包消息，这样就可以自动的处理黏包和半包消息。

## 9. log

```properties
log4j.rootLogger=DEBUG,stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=[%p]%C{1}-%m%n
```

## 10. TCP 粘包和拆包

### 10.1 基本介绍

1. TCP 是面向连接的，面向流的，提供高可靠性服务。收发两端（客户端和服务器端）都要有一一成对的 socket，因此，发送端为了将多个发给接收端的包，更有效的发给对方，使用了优化方法（Nagle 算法），**将多次间隔较小且数据量小的数据，合并成一个大的数据块**，然后进行封包。这样做虽然**提高了效率，但是接收端就难于分辨出完整的数据包了**，因为**面向流的通信是无消息保护边界**的
2. 由于 TCP 无消息保护边界,需要在接收端处理消息边界问题，也就是我们所说的粘包、拆包问题,看一张图
   TCP 粘包、拆包图解

![1658152850559](image/netty/TCP 粘包、拆包.png )

### 10.2 TCP 粘包和拆包解决方案

1. 常用方案：使用**自定义协议+编解码器**来解决
2. 关键就是要解决**服务器端每次读取数据长度的问题**，这个问题解决，就不会出现服务器多读或少读数据的问题，从而避免的 `TCP` 粘包、拆包。

只需要约定，客户端给的数据增加个读取规则，然后服务端按这个规则读取数据，不会拆包。比如message多一个长度lenght，服务端读取数据时，按定长length读取

- 编码，每次发送数据，多给个length

```java
package com.codeman.netty.netty.tcp.cusprotocol;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.ReplayingDecoder;

import java.util.List;

public class MyMessageDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        System.out.println();
        System.out.println();
        System.out.println("MyMessageDecoder decode 被调用");
        //需要将得到二进制字节码-> MessageProtocol 数据包(对象)
        int length = in.readInt();

        byte[] content = new byte[length];
        in.readBytes(content);

        //封装成 MessageProtocol 对象，放入 out， 传递下一个handler业务处理
        MessageProtocol messageProtocol = new MessageProtocol();
        messageProtocol.setLen(length);
        messageProtocol.setContent(content);

        //放入out传给下一个hanlder进行处理
        out.add(messageProtocol);

    }
}
```

- 解码，按定长length获取字节

```java
package com.codeman.netty.netty.tcp.cusprotocol;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.MessageToByteEncoder;

public class MyMessageEncoder extends MessageToByteEncoder<MessageProtocol> {
    @Override
    protected void encode(ChannelHandlerContext ctx, MessageProtocol msg, ByteBuf out) throws Exception {
        System.out.println("MyMessageEncoder encode 方法被调用");
        out.writeInt(msg.getLen());
        out.writeBytes(msg.getContent());
    }
}
```

## 11. 手写RPC

- RPCClientChannleHandler  - 共享

```java
package com.codeman.netty.netty.rpc;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

import java.lang.reflect.Proxy;
import java.util.concurrent.Callable;

/**
 * @author: zhanghongjie
 * @description:
 * @date: 2022/7/18 23:11
 * @version: 1.0
 */
public class RPCClientChannleHandler extends ChannelInboundHandlerAdapter implements Callable {

    private ChannelHandlerContext context;
    private String msg;
    private Object result;

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelActive");
        context = ctx;
    }

    @Override
    public synchronized  void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("log = 客户端接收到请求：" + msg);
        result = msg;
        notify();
    }


    @Override
    public synchronized Object call() throws Exception {
        System.out.println("进来call");
        context.writeAndFlush(msg);
        wait();
        return result;
    }

    public void setMsg(String msg) {
        System.out.println("client 入参 " + msg);
        this.msg = msg;
    }
}

```

- RPCNettyClient

```java
package com.codeman.netty.netty.rpc;

import com.codeman.netty.netty.rpc.api.APIContants;
import com.codeman.netty.netty.rpc.api.IService;
import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.Unpooled;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;
import io.netty.util.CharsetUtil;

import java.lang.reflect.Proxy;
import java.net.InetSocketAddress;
import java.util.Scanner;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author: zhanghongjie
 * @description:
 * @date: 2022/7/10 11:48
 * @version: 1.0
 */
public class RPCNettyClient {

    private static ExecutorService executorService = Executors.newFixedThreadPool(6);
    private static RPCClientChannleHandler clientHandler = null;
    private static int count = 0;



    public static void main(String[] args) {

        //客户端需要一个事件循环组
        EventLoopGroup workGroup = new NioEventLoopGroup();

        try {
            //创建客户端启动对象
            //注意客户端使用的不是 ServerBootstrap 而是 Bootstrap
            clientHandler = new RPCClientChannleHandler();
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(workGroup)
                    .channel(NioSocketChannel.class) // 设置客户端通道的实现类(反射)，和服务端不一样
                    .handler(new ChannelInitializer() {
                        @Override
                        protected void initChannel(Channel ch) throws Exception {
                            ch.pipeline().addLast(new StringDecoder());
                            ch.pipeline().addLast(new StringEncoder());
                            ch.pipeline().addLast(clientHandler); //加入自己的处理器
                        }
                    });

            System.out.println("客户端ok");
            //启动客户端去连接服务器端
            IService service = (IService) getBean(IService.class, APIContants.FUCTION_HELLO);
            bootstrap.connect(new InetSocketAddress("localhost", 6677)).sync();
            System.out.println("又阻塞吗");
            System.out.println(service.hello("你好鸭"));


        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            workGroup.shutdownGracefully();
        }

        /*clientHandler = new RPCClientChannleHandler();
        //创建EventLoopGroup
        NioEventLoopGroup group = new NioEventLoopGroup();
        Bootstrap bootstrap = new Bootstrap();
        bootstrap.group(group)
                .channel(NioSocketChannel.class)
                .option(ChannelOption.TCP_NODELAY, true)
                .handler(
                        new ChannelInitializer<SocketChannel>() {
                            @Override
                            protected void initChannel(SocketChannel ch) throws Exception {
                                ChannelPipeline pipeline = ch.pipeline();
                                pipeline.addLast(new StringDecoder());
                                pipeline.addLast(new StringEncoder());
                                pipeline.addLast(clientHandler);
                            }
                        }
                );

        try {
            IService service = (IService) getBean(IService.class, APIContants.FUCTION_HELLO);
            bootstrap.connect("127.0.0.1", 6677).sync();
            System.out.println("又阻塞吗");
            System.out.println(service.hello("你好鸭"));
        } catch (Exception e) {
            e.printStackTrace();
        }*/


    }

    public static Object getBean(final Class<?> serivceClass, final String providerName) {

        return Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
                new Class<?>[]{serivceClass}, (proxy, method, args) -> {

                    System.out.println("(proxy, method, args) 进入...." + (++count) + " 次");
                    //{}  部分的代码，客户端每调用一次 hello, 就会进入到该代码

                    //设置要发给服务器端的信息
                    //providerName：协议头，args[0]：就是客户端要发送给服务端的数据

                    //
                    clientHandler.setMsg(providerName + args[0]);
                    return executorService.submit(clientHandler).get();
                });
    }
}

```

- RPCServerChannleHandler

```java
package com.codeman.netty.netty.rpc;

import com.codeman.netty.netty.rpc.api.APIContants;
import com.codeman.netty.netty.rpc.api.ServiceImpl;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * @author: zhanghongjie
 * @description:
 * @date: 2022/7/18 23:11
 * @version: 1.0
 */
public class RPCServerChannleHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("客户端接收到请求：" + msg);
        if (msg.toString().startsWith(APIContants.FUCTION_HELLO)) {
            ctx.writeAndFlush("服务端响应："
                    + new ServiceImpl().hello(msg.toString().substring(msg.toString().indexOf(APIContants.FUCTION_HELLO))));
        } else {
            //
        }
    }

}

```

=======

>>>>>>> 5457daad7a4833b765005ec3df770ce3c852d97d
>>>>>>>
>>>>>>
>>>>>
>>>>
>>>
>>
