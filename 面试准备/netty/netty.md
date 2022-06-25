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

![1656147090525](image/netty/1656147090525.png)

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

![1656171493257](image/netty/传统IO模型拷贝.png )

**DMA** ：`direct memory access` 直接内存拷贝（不使用 `CPU`）

#### 4.3.3 mmap内存映射优化

1. `mmap` 通过**内存映射**，将文件映射到内核缓冲区，同时，用户空间可以共享内核空间的数据。这样，在进行网络传输时，就可以减少内核空间到用户空间的拷贝次数。如下图

![1656171692466](image/netty/mmap内存映射优化.png )

#### 4.3.4 sendFile优化

1. `Linux2.1` 版本提供了 `sendFile` 函数，其基本原理如下：数据根本**不经过用户态**，直接从内核缓冲区进入到 `SocketBuffer`，同时，由于和用户态完全无关，就减少了一次上下文切换

![1656172167891](image/netty/拷贝sendFile优化.png )

2. 零拷贝从操作系统角度，**是没有 `cpu` 拷贝**

3. `Linux在2.4` 版本中，做了一些修改，避免了从内核缓冲区拷贝到 `Socketbuffer` 的操作，直接拷贝到协议栈，从而再一次减少了数据拷贝。具体如下图和小结：

这里其实只有一个cpu拷贝，但是拷贝的信息比较少，就是一些desc描述信息，比如length/offset等，消耗低，可忽略

![1656172275068](image/netty/拷贝sendFile优化2.png )

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
