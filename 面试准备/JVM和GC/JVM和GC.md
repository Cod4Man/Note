# JVM和GC

## 1. JMM： Java内存模型（Java Memory Model）

- **JMM本身是一种抽象的概念 并不真实存在**,它描述的是一组规则或规范通过规范定制了程序中各个变量(包括实例字段,静态字段和构成数组对象的元素)的访问方式

- **三大特性(线程安全的保证)**

  - **可见性：变量可见。**

    JMM规范规定所有的变量都存储在**主内存**中，主内存是所有线程共享的区域。而JVM运行程序的实体是线程，每个线程创建JVM都会为其开辟一个独立的**工作内存**，而线程对变量的操作都必须在工作内存中，因此各线程需要从主内存中拷贝一份数据到自己的工作内存中，在工作内存中对其进行修改，然后再写回主内存。

    **JMM关于同步规定:**

    **1).线程解锁前,必须把共享变量的值刷新回主内存**

    **2).线程加锁前,必须读取主内存的最新值到自己的工作内存**

    **3).加锁解锁是同一把锁**

    ![1610798333848](E:\SoftwareNote\面试准备\JVM和GC\img\JMM可见性.png)

  - **原子性：**不可分割，完整性。一个线程在做某个业务时，不可被其他线程打断，需要整体完整。要么成功要么失败（像SQL事务）。**完美解决方案AtomicInteger**

  - **有序性：**指令不可重排

    - 计算机在执行程序时,为了提高性能,编译器和处理器常常会做指令重排,一般分为以下3种指令重排计算机在执行程序时,为了提高性能,编译器和处理器常常会做指令重排,一般分为以下3种:

      源代码 -> **编译器优化的重排** -> **指令并行的重排** -> **内存系统的重排** ->  最终执行的命令

    - 处理器在进行**重新排序是必须要考虑指令之间的数据依赖性数据依赖性(没有依赖即可重排)** 

    - 多线程环境中线程交替执行,由于编译器优化重排的存在,两个线程使用的变量能否保持一致性是无法确定的,结果无法预测

    ```java
    public voidpublic void mySort(){
        int x=11;//语句1
        int y=12;//语句2
        x=x+5;//语句3
        y=x*x;//语句4
    }
    // 1234
    // 2134
    // 1324
    问题:
    请问语句4 可以重排后变成第一条码?
    存在数据的依赖性 没办法排到第一个
    ```

    ```java
    int a ,b ,x,y=0;
    
    线程1
    线程2
    x=a;
    y=b;
    b=1;
    a=2;
    
    
    x=0 y=0
    
     如果编译器对这段代码进行执行重排优化后,可能出现下列情况:
    
    线程1
    线程2
    b=1;
    a=2;
    x=a;
    y=b;
    
    
    x=2 y=1
    
     这也就说明在多线程环境下,由于编译器优化重排的存在,两个线程使用的变量能否保持一致是无法确定的.
    
    ```

    

## 2. 内存屏障 /内存栅栏(Memory Barrier) volatile

- 作用:

  - 保证特定操作的**执行顺序**
  - 保证某些变量的**内存可见性**

- 简介

  由于编译器和处理器都能执行指令重排优化。如果在指令间插入一条MemoryBarrier则会告诉编译器和CPU，不管什么指令都不能和这条MemoryBarrier指令重排序，也就是说通过插入内存屏障禁止在内存屏障前后的指令重排序优化。**内存屏障另一个作用是强制刷出各种CPU缓存数据**，因此任何CPU上的线程都能读取到这些数据的最新版本。

  ![1610808463531](E:\SoftwareNote\面试准备\JVM和GC\img\Voatile的禁止指令重排.png)

## 3. JVM内存结构：(细节可以看石杉的架构笔记：我肝了万字的Java垃圾回收https://mp.weixin.qq.com/s/hiw5878tQz0_fbcffaEk1w)

## 3.1 JVM体系概述

![1612713819316](E:\SoftwareNote\面试准备\JVM和GC\img\JVM体系概述.png)



### 3.2 Java8以后的JVM

![1612714108491](E:\SoftwareNote\面试准备\JVM和GC\img\Java8以后的JVM.png)

### 3.3 GC作用域

![1612714223381](E:\SoftwareNote\面试准备\JVM和GC\img\GC作用域.png)

### 3.4 常见的垃圾回收算法

- 引用计数

  ![1612714320054](E:\SoftwareNote\面试准备\JVM和GC\img\GC之引用计数法.png)

- 复制

  ![1612714361180](E:\SoftwareNote\面试准备\JVM和GC\img\GC之复制算法.png)

  ![1614090684931](E:\SoftwareNote\面试准备\JVM和GC\img\GC之复制算法02.png)

- 标记清除

  ![1612714398714](E:\SoftwareNote\面试准备\JVM和GC\img\GC之标记清楚算法.png)

  ![1614090731013](E:\SoftwareNote\面试准备\JVM和GC\img\GC之标记清楚算法02.png)

- 标记整理

  ![1612714436838](E:\SoftwareNote\面试准备\JVM和GC\img\GC之标记压缩算法.png)

![1614090762766](E:\SoftwareNote\面试准备\JVM和GC\img\GC之标记压缩算法02.png)

- 分代收集

  其实就是整合了上面三种算法，扬长避短。

  之所以叫分代，是因为根据**对象存活周期的不同**将整个 Java 堆切割成为三个部分：

  - Young(年轻代)

  - - Eden(伊利园)：新生对象
    - Survivor(幸存者)：垃圾回收后还活着的对象

  - Tenured(老年代)：对象多次回收都没有被清理，会移到老年代

  - Perm(永久代)：**存放加载的类别还有方法对象，java8 之后移除了永久代，替换为元空间(Metaspace)**

  在新生代中，每次垃圾收集都有大量的对象死去，只有少量的存活，那就选用 **复制算法** ，因为复制成本很小，只需要复制少量存活对象。

  老年代中，存活对象较多，没有额外的空间担保，就得使用 **标记清除** 或者 **标记整理** 。

## 4. JVM垃圾回收的时候如何确定垃圾?是否知道什么是 GC Roots

### 4.1 什么是垃圾

- 简单来说，就是内存中，不再被使用到的空间就是垃圾

### 4.2 如何判断一个对象是否可以回收

- 引用计数法: 无法解决循环引用的问题

![1612787872782](E:\SoftwareNote\面试准备\JVM和GC\img\GC引用计数法识别垃圾对象.png)

- 枚举根节点做可达性分析（根搜索路径）

  ![1612788367921](E:\SoftwareNote\面试准备\JVM和GC\img\可达性分析法识别垃圾对象.png)

![ 1612787985539](E:\SoftwareNote\面试准备\JVM和GC\img\以GCRoots做可达性分析.png)

  - Java 可以做GCRoots的对象
    - 虚拟机栈(栈帧中的局部变量区,也叫做局部变量表
    - 方法区中的类静态属性引用的对象
    - 方法区中常量引用的对象
    - 本地方法栈中N( Native方法)引用的对象

## 5. JVM调优和参数配置，如何盘点查看MM系统默认值

### 5.1 JVM的参数类型

#### 5.1.1 标配参数

- -verison
- -help
- java -showversion

#### 5.1.2 X参数（了解）

![1612788850241](E:\SoftwareNote\面试准备\JVM和GC\img\JVM之X参数.png)

- -Xint: 解释执行
- -Xcomp: 第一次使用就编译成本地代码
- -Xmixed: 混合模式

#### 5.1.3 **XX参数**

- Boolean类型：

  - -XX：+或者-  某个属性值（+表示true，-表示false）

  - 如：是否打印GC收集细节： -XX: +/- PrintGCDetails

    ```shell
    Heap
     PSYoungGen      total 114176K, used 7864K [0x0000000740c00000, 0x0000000748b00000, 0x00000007c0000000)
      eden space 98304K, 8% used [0x0000000740c00000,0x00000007413ae2f8,0x0000000746c00000)
      from space 15872K, 0% used [0x0000000747b80000,0x0000000747b80000,0x0000000748b00000)
      to   space 15872K, 0% used [0x0000000746c00000,0x0000000746c00000,0x0000000747b80000)
     ParOldGen       total 261120K, used 0K [0x0000000642400000, 0x0000000652300000, 0x0000000740c00000)
      object space 261120K, 0% used [0x0000000642400000,0x0000000642400000,0x0000000652300000)
     Metaspace       used 3489K, capacity 4498K, committed 4864K, reserved 1056768K
      class space    used 387K, capacity 390K, committed 512K, reserved 1048576K
    ```

- KV类型

  - -XX：属性key=属性值value

  - 如： -XX:MetaspaceSize=128m； -XX:MaxTenuringThreshold=15

  - **经典参数-Xms和-Xmx**

    -Xms 等价于 -XX:InitialHeapSize

    -Xmx 等价于-XX:MaxHeapSize

#### 5.1.4 jinfo查看当前运行进程的配置

- jinfo -flag xxxx(参数名) 1234(进程号，进程号用jps -l查看)

- jinfo -flags 1234

  ```shell
  C:\Users\ASUS>jinfo -flags 12340
  Attaching to process ID 12340, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 25.112-b15
  Non-default VM flags: -XX:CICompilerCount=4 -XX:InitialHeapSize=400556032 -XX:MaxHeapSize=6404702208 -XX:MaxNewSize=2134900736 -XX:MetaspaceSize=134217728 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=133169152 -XX:OldSize=267386880 -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
  Command line:  -XX:+PrintGCDetails -XX:MetaspaceSize=128m -javaagent:C:\Program Files (x86)\IDEA\IntelliJ IDEA 2018.2.2\lib\idea_rt.jar=6608:C:\Program Files (x86)\IDEA\IntelliJ IDEA 2018.2.2\bin -Dfile.encoding=UTF-8
  ```

### 5.2 查看JVM默认值

- 查看初始默认值java -XX:+PrintFlagsInitial -version

  ![1613222561529](E:\SoftwareNote\面试准备\JVM和GC\img\查看初始默认值PrintFlagsInitial.png)

- 主要查看修改更新java -XX:+PirntFlagsFinal -version

  ![1613222638761](E:\SoftwareNote\面试准备\JVM和GC\img\主要查看修改更新PirntFlagsFinal.png)

- 打印一些命令标记-XX:+PrintCommandLineFlags

  ```shell
  E:\Java_Code\IDEA_CODE\ligong\UpUp2021\out\production\UpUp2021\com\codeman\JVMNGC>java -XX:+PrintCommandLineFlags -version
  -XX:InitialHeapSize=400218432 
  -XX:MaxHeapSize=6403494912 
  -XX:+PrintCommandLineFlags 
  -XX:+UseCompressedClassPointers 
  -XX:+UseCompressedOops 
  -XX:-UseLargePagesIndividualAllocation 
  -XX:+UseParallelGC  ## 查看GC回收策略？
  java version "1.8.0_112"
  Java(TM) SE Runtime Environment (build 1.8.0_112-b15)
  Java HotSpot(TM) 64-Bit Server VM (build 25.112-b15, mixed mode)
  
  ```

## 6. 平时工作用过的JVM常用基本配置参数

- -Xms: 等价于-XX:InitialHeapSize	

  初始大小内存，默认为物理内存1/64

- -Xmx: 等价于-XX:MaxHeapSize

  最大分配内存，默认为物理内存1/4

- -Xss: 等价于-XX:ThreadStackSize

  设置单个线程的大小，一般默认为512K~1024K

- -Xmn

  设置年轻代大小

- -XX:MetaspaceSize

  设置元空间大小

  元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受限于本地内存

- -XX:+PrintGCDetails

  输出详细GC收集日志信息

  - GC

    ![1613223653741](E:\SoftwareNote\面试准备\JVM和GC\img\GC收集日志信息_GC.png)

  - FullGC

    ![1613223696598](E:\SoftwareNote\面试准备\JVM和GC\img\GC收集日志信息_FullGC.png)

- -XX:SurvivoRatio

  设置新生代中Eden和s0/s1空间的比例

  默认-XX:SurviviRatio=8, Eden:s0:s1 = 8:1:1

  -XX:SurviviRatio=4, Eden:s0:s1 = 4:1:1

- -XX:NewRatio

  配置年轻代与老年代在堆结构的占比

  默认-XX:NewRatio=2，老年代：新生代= 2：1

  -XX:NewRatio=4，老年代：新生代= 4：1

- -XX:PretenureSizeThreshold 当创建的对象超过指定大小时，直接把对象分配在老年代

- -XX:MaxTenuringThreshold

  设置垃圾最大年龄

  ![1613224562496](E:\SoftwareNote\面试准备\JVM和GC\img\设置垃圾最大年龄MaxTenuringThreshold.png)

- -XX:+HeapDumpOnOutOfMemoryError让JVM在发生内存溢出的时候自动生成内存快照, 

  排查问题用 

- -XX:+DisableExplicitGC禁止系统System.gc()，防止手动误触发FGC造成问题. 

## 7. 强引用、软引用、弱引用、虚引用

### 7.1 整体架构

![1613226725475](E:\SoftwareNote\面试准备\JVM和GC\img\GC引用架构.png)

### 7.2 强引用（默认支持模式）

- 99%都是强引用，对于强引用的对象，就算出现OOM，也不会去回收
- 即使该对象以后都不会被用到，JVM也不会去回收，这是造成OOM的主要原因
- demo

```java
Object obj1 = new Object();
Object obj2 = obj1;

obj1 = null;
System.gc(); // 手动GC

System.out.println(obj1); // null
System.out.println(obj2); // java.lang.Object@1540e19d obj2指向的地址还在
```

### 7.3 软引用

- 软引用时一种相对强引用弱化了一些的引用，需要用java.lang.ref.SoftReference<T>类来实现，可以让对象豁免一些垃圾收集。
- 对于只有软引用的对象来说，当系统内存充足，不会被回收；**内存不足，会被回收**。
- 通常用来堆内存敏感的程序中，比如高速缓存。
- 应用场景：**图片url缓存**，图片一般都会加载在内存中，这样用户体验好，但是加载太多就容易OOM，因此可以用软引用，内存不足时直接回收
- demo

```java
// java -Xms5m -Xmx5m
Object obj1 = new Object();
SoftReference<Object> obj2 = new SoftReference<>(obj1);
obj1 = null;
System.gc();
// 模拟内存不足
try {
    byte[] bytes = new byte[6 * 1024 * 1024]; // java.lang.OutOfMemoryError: Java heap space
} finally {
    System.out.println(obj1);  // null
    System.out.println(obj2.get()); // null ，没有OOM的话，不会被回收
}
```

### 7.4 弱引用

- 需要用java.lang.ref.WeakReference类实现。
- **只要GC就会被回收**
- demo

```java
Object obj1 = new Object();
WeakReference<Object> weakReference = new WeakReference<>(obj1);

obj1 = null;
System.gc();
System.out.println(weakReference.get()); // null
```

### 7.5 弱引用-WeakHashMap

- 内部类Entry

  ```java
  Entry<K,V> extends WeakReference<Object>
  ```

- demo

```java
WeakHashMap<Integer, String> weakHashMap
                = new WeakHashMap<>();
Integer integer = new Integer(1);
String str = "123";
weakHashMap.put(integer, str);
System.out.println(weakHashMap + "" + weakHashMap.size()); // {1=123}1
integer = null;
System.gc();
try {
    Thread.sleep(20);
} catch (InterruptedException e) {
    e.printStackTrace();
}
System.out.println(weakHashMap + "" + weakHashMap.size()); // {}1  {}0,加上延迟才会是0
```

### 7.6 虚引用

- 概念

  需要用PhantomReference来实现。

  形同虚设，虚引用就像没有任何引用一样，随时可能被GC，PhantomReference不能单独使用，也拿不到对象，只能配合引用队列ReferenceQueue联合使用。

  虚引用主要作用：跟踪对象被垃圾回收的状态。

  ![1613233821556](E:\SoftwareNote\面试准备\JVM和GC\img\虚引用概念.png)

- 同理，软引用和弱引用也可以配和ReferenceQueue使用，即能做到软引用和弱引用效果，又能在回收后做通知动作。

- demo

```java
/**
* 虚引用
* 没有什么用(不会应该原对象的回收机制)，
* 配合ReferenceQueue使用，
* 可以在对象被回收时加入引用队列中，
* 这样就可以在对象被回收时做一些特殊动作(比如通知)
*/
private static void fakeReference() throws InterruptedException {
    ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
    Object object = new Object();
    PhantomReference phantomReference = new PhantomReference(object, referenceQueue);
    System.out.println(phantomReference.get());
    System.out.println(referenceQueue.poll());
    System.out.println("====================");
    object = null;
    System.gc();
    Thread.sleep(20);
    System.out.println(phantomReference.get());
    System.out.println(referenceQueue.poll());
}
```

### 7.7 四大引用总结

![1613234813769](E:\SoftwareNote\面试准备\JVM和GC\img\四大引用总结.png)

## 8. OOM Error种类

![1613396002569](E:\SoftwareNote\面试准备\多线程\img\OOMError依赖图.png)

### 8.1 StackOverflowError 栈(方法)溢出

- 产生原因：方法调用过深(常出现于递归)，导致栈溢出
- demo

```java
private static void stackOverflowError() {
    stackOverflowError();
}
```

### 8.2 OOM: java heap space 堆溢出 

- 产生原因： 对象太多，导致堆撑爆

- demo

```java
/**
* 堆溢出. java.lang.OutOfMemoryError: Java heap space
*/
private static void OOMJavaHeapSpace() {
    // -Xms5m -Xmx5m
    byte[] bytes = new byte[6 * 1024 * 1024];
}
```

### 8.3 OOM: GC overhead limit exceeded (GC回收效果不佳)

- 产生原因：程序在垃圾回收上花费了98%的时间，却收集不回2%的空间，通常这样的异常伴随着CPU的冲高。天天GC还干不干活了？

  ![1613397633956](E:\SoftwareNote\面试准备\多线程\img\OOMGCOverheadLimitExceeded.png)

- demo

```java
/**
 * -Xms15m -Xmx15m -XX:MaxDirectMemorySize=3m -XX:+PrintGCDetails
 * java.lang.OutOfMemoryError: GC overhead limit exceeded
 */
private static void OOMGCOverheadLimitExceeded() {
    List<String> list = new ArrayList<>();
    while(true) {
        list.add((new Random().nextInt() + "-" + System.currentTimeMillis()).intern());
    }
}


// Full GC (Ergonomics) [PSYoungGen: 2047K->2047K(3584K)] [ParOldGen: 11154K->11132K(11264K)] 13202K->13180K(14848K), [Metaspace: 3505K->3505K(1056768K)], 0.0503306 secs] [Times: user=0.47 sys=0.00, real=0.05 secs
// 2047K->2047K 回收没有效果，直接抛出OOMError
```

### 8.4 OOM: Direct buffer memory 直接内存不足

- Direct Memory：运行时数据区域——直接内存 

  直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域。但是这部分内存也被频繁的使用，而且也可能导致OutOfMemoryError异常出现。 

  在JDK1.4中新加入了NIO（New  Input/Output）类，引入了一种**基于通道（Channel）与缓冲区（Buffer）的I/O方式**，他可以**使用Native函数库直接分配堆外内存**，然后同故宫一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一场场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。 

  显然，本机直接内存的分配不会受到Java堆大小的限制，但是，既然是内存，肯定还是会受到本机总内存（包括RAM以及SWAP区或者分页文件）大小以及处理器寻址空间的限制。服务器管理员在配置虚拟机参数时，会根据实际内存设置-Xmx等参数信息，但经常忽略直接内存，使得各个内存区域总和大于物理内存限制（包括物理的和操作系统级的限制），从而导致动态扩展时出现OutOfMemoryError异常。 

- 产生原因: 使用NIO时，分配的DirectMemory大于设定的MaxDirectMemorySize。程序中直接或者间接的使用NIO可能会导致此类异常的产生 (ByteBuffer)

- demo

```java
    /**
     * java.lang.OutOfMemoryError: Direct buffer memory
     * 程序中直接或者间接的使用NIO可能会导致此类异常的产生 ByteBuffer
     */
    private static void OOMDirectBufferMemory() {
        System.out.println(sun.misc.VM.maxDirectMemory());

        // -XX:MaxDirectMemorySize=5m  本地
        // Allocates a new direct byte buffer
        ByteBuffer allocate = ByteBuffer.allocateDirect(16 * 1024 * 1024);

    }
```

### 8.5 OOM: unable to create new native thread(超出最大线程数)

- 产生原因: 线程数超出最大线程数

- linux线程数设置

  ![1613402134789](E:\SoftwareNote\面试准备\多线程\img\Linux线程数设置.png)

- demo

```java
	/**
     * java.lang.OutOfMemoryError: unable to create new native thread
     * 线程数超出最大线程数
     */
    private static void OOMUnableCreateNativeThread() {
        while (true) {
            new Thread(() -> {
                try {
                    Thread.sleep(Integer.MAX_VALUE);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }

[roo@192 temp]$ java OOMDemo
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
	at java.lang.Thread.start0(Native Method)
	at java.lang.Thread.start(Thread.java:717)
	at OOMDemo.OOMUnableCreateNativeThread(OOMDemo.java:35)
	at OOMDemo.main(OOMDemo.java:24)
^CJava HotSpot(TM) 64-Bit Server VM warning: Exception java.lang.OutOfMemoryError occurred dispatching signal SIGINT to handler- the VM may need to be forcibly terminated

```

### 8.6 OOM： Metaspace （元空间溢出）

- 当一个类被加载并且它在JVM中的运行时表示正在准备时，它的类加载器会分配元空间来存储类的元数据 

- 超出MaxMetaspaceSize

- demo 

```java
/**
* java.lang.OutOfMemoryError: Metaspace
* -XX:MaxMetaSpaceSize=5m
*/
private static void test5() {
    //        List list = new ArrayList();
    while (true) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(OOMDemo.class);
        enhancer.setCallback((MethodInterceptor) (o, method, objects, methodProxy) -> methodProxy.invoke(o, null));
        System.out.println("this is before method");
        OOMDemo oomDemo = (OOMDemo) enhancer.create();
        oomDemo.staticMethod();  // this is static method
        System.out.println("this is after method");
    }
}
```

## 9. GC垃圾回收算法和垃圾收集器的关系 

GC算法是内存回收的**方法论**，垃圾收集器就是算法**落地实现**。

因为目前为止还没有完美的收集器出现，更加没有万能的收集器，只是针对具体应用最合适的收集器，进行分代收集。

### 9.1 GC算法

- 引用计数
- 复制
- 标记清除
- 标记压缩(整理)

### 9.2 四种主要垃圾收集器

![1613484292842](E:\SoftwareNote\面试准备\JVM和GC\img\垃圾收集器.png)

#### 9.2.1 分类

- **串行垃圾回收器（Serial）**: 它为**单线程**环境设计并且只使用一个线程进行垃圾回收，会**暂停所有的用户线程**。所以不适合服务器环境

- **并行垃圾回收器（Parallel）**: 多个垃圾回收线程并行工作，此时用户线程是**暂停的**，适用于科学计算/大数据处理等**弱交互场景**

- **并发垃圾回收器（CMG: ConcMarkSweep**）: 用户线程和垃圾收集线程**同时执行**（不一定是并行，可能交替执行），**不需要停顿用户线程**

  **互联网公司多用它，适用于对响应时间有要求的场景** 

- **G1垃圾回收器**: G1垃圾回收器将堆内存分割成不同的区域然后并发的对其进行垃圾回收

- **ZGC （Z Garbage Collector）**是一款由Oracle公司研发的，以低延迟为首要目标的一款垃圾收集器。它是基于动态Region内存布局，（暂时）不设年龄分代，使用了读屏障、染色指针和内存多重映射等技术来实现可并发的标记-整理算法的收集器。在 JDK 11 新加入，还在实验阶段，主要特点是：回收TB级内存（最大4T），停顿时间不超过10ms。优点：低停顿，高吞吐 量， ZGC 收集过程中额外耗费的内存小。缺点：浮动垃圾

#### 9.2.2 查看默认垃圾收集器

- -XX:+PrintCommandLineFlags 

```shell
E:\Java_Code\IDEA_CODE\ligong\UpUp2021>java -XX:+PrintCommandLineFlags -version
-XX:InitialHeapSize=400218432 -XX:MaxHeapSize=6403494912 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -X
X:+UseParallelGC
java version "1.8.0_112"
Java(TM) SE Runtime Environment (build 1.8.0_112-b15)
Java HotSpot(TM) 64-Bit Server VM (build 25.112-b15, mixed mode)

```

#### 9.2.3 GC回收类型：６（７）种

- client/server端：
  - 只需掌握server，client几乎不会用
  - 32位Windows操作系统，无论硬件如何，默认都是client的JVM模式
  - 32位的其他操作系统，2G内存并且同时用于2个以上的CPU用的Server模式，低于配置用的都是client模式
  - 64位只有server模式

- 垃圾收集器作用范围

![1613485532865](E:\SoftwareNote\面试准备\JVM和GC\img\垃圾收集器作用范围.png)

- 垃圾收集器对于GC日志简写说明

![1613486653217](E:\SoftwareNote\面试准备\JVM和GC\img\垃圾收集器对于GC日志简写说明.png)

- 垃圾收集器底层源码

  ![1613495401037](img\垃圾收集器底层源码.png)

- 分类

  - UseSerialGC
  - ~~(*UseSerialOldGC*: 过时)~~
  - UseParallelGC
  - UseConcMarkSweepGC
  - UseParNewGC
  - UseParallelOldGC
  - UseG1GC

- 新生代

  - 串行GC（Serial）/（Serial Coping）>>>>> **-XX:+UseSerialGC ** <<<<<<<<

    开启后，会使用新生代Serial+老年代Serial Old收集器组合，均为串行回收器，新生代用的复制算法，来年代用的标记-整理算法

    ![1613491998655](E:\SoftwareNote\面试准备\JVM和GC\img\新生代Serial串行GC.png)

    ![1613492461497](E:\SoftwareNote\面试准备\JVM和GC\img\新生代Serial串行GC日志.png)

  - 并行GC(ParNew)   >>>>> **-XX:+UseParNewGC ** <<<<<<<<

    开启后，新生代用ParNew+老年代用Serial Old收集器组合，新生代用复制算法，老年代用标记-整理算法。但是java8已不再推荐使用

    --XX:ParallelGCThreads=xx 可以限制线程数量，默认开启和CPU数目相同的线程数

    ![1613492564201](E:\SoftwareNote\面试准备\JVM和GC\img\新生代ParNew并行GC.png)

    ![1613492750808](E:\SoftwareNote\面试准备\JVM和GC\img\新生代ParNew并行GC日志.png)

  - 并行回收GC(Parallel)/(Parallel Scavenge)   >>>>> **-XX:+UseParallelGC ** <<<<<<<<

    开启后，新老年代都使用Parallel并行收集器，新生代用复制算法，老年代用标记整理算法

    可控制的吞吐量（Thoughput=运行用户代码时间/运行用户代码时间+GC时间）:高吞吐量以为着高效利用CPU的时间，它用于在后台运算而不需要太多交互的任务

    ![1613493083620](img\新生代Parallel并行GC.png)

    ![1613493267646](img\新生代Parallel并行GC日志.png)

- 老年代

  - 串行回收GC(Serial Old)/(Serial MSC)

    ![1613494421344](img\老年代串行回收SerialOldGC.png)

  - 并行GC(Parallel Old)/(Parallel MSC)

    ![1613494537150](img\老年代并行回收ParallelOldGC.png)

  - 并发标记清除GC(CMS)

    ![1613494602848](img\老年代并发回收CMSGC.png)

    - CMS四步骤

      ![1613494933586](img\CMS回收四步骤.png)

      - 初始标记(CMS initial mark)
      - 并发标记(CMS concurrent mark)和用户线程一起: 进行GCRoots跟踪的过程，和用户线程一起工作，无需STW，主要是标记过程，标记全部对象
      - 重新标记(CMS remark)：修正并发标记期间，因为没有STW而导致的标记产生变动的那一部分对象进行重新标记修正，需要STW
      - 并发清除(CMS concurrent sweep)和用户线程一起： 有了以上操作，并发清除就无需STW，可以并发清除
      - 失败了会Serial Old 串行回收，需要STW，很耗时

    - 优缺点

      - 优点：并发收集低停顿

      - 缺点：

        ①并发执行，对CPU资源压力大：由于并发进行，CMS在收集与应用线程会同时增加对堆内存的占用，也就是说CMS必须要在老年代堆内存用尽之前完成垃圾回收，否则CMS失败会触发担保机制，串行老年代收集器（SerialOldGC）会STW然后进行一次GC，从而造成较大的停顿时间

        ②采用的标记清除算法会导致大量碎片： 标记清除算法无法整理空间碎片，老年代空间会随着时间增加而被逐步耗尽，最后将不得不通过担保机制对堆内存进行压缩。CMS也提供了参数-XX：CMSFullGCsBeForeCOmpaction（默认0，即每次都进行内存整理）来指定多少次CMS收集之后，进行一次压缩的FullGC

#### 9.2.4 垃圾收集器的选择

SerialGC(串行)：单CPU/单机/小内存

ParallelGC(并行)：高吞吐

CMS(并发)：响应时间短的

![1613486258883](img\垃圾收集器的选择.png)

## 10. G1垃圾收集器 (Garbage-First)

7.0出现，9.0默认（替代CMS）

![1613663521765](E:\SoftwareNote\面试准备\JVM和GC\img\各大垃圾回收器的示意图.png)

### 10.1 G1出现前，其他收集器的特点 

- 年轻代和老年代是各自独立且连续的内存块
- 年轻代收集使用单eden+S0 +S进行复制算法
- 老年代收集必须扫描整个老年代区域
- 都是以尽可能少而快速地执行GC为设计原则

### 10.2 G1是什么

- 是一款面向服务端应用的收集器，应用在多处理器和大容量内存环境中，在实现高吞吐量的同时，尽可能的满足垃圾收集暂停时间的要求

- G1收集器可以像CMS收集器一样，能与应用程序线程并发执行

- 整理空闲空间更快

- 需要更多的时间来预测GC停顿时间

- 不希望牺牲大量的吞吐性能

- 不需要更大的Java heap

- G1收集器的设计目标是取代CMS收集器，相比CMS，G1更加突出：

  - G1是一个有整理内存过程的垃圾收集器，不会产生很多的内存碎片
  - G1的STW更**可控**，G1在停顿时间上添加了**预测机制**，用户可以指定期望停顿时间

- CMS垃圾收集器虽然减少了暂停应用程序的运行时间，但是它还是存在着内存碎片问题。为了去除内存碎片的问题，同时又保留了CMS垃圾收集器低暂停时间的有点，JAVA7发布了G1.

  G1在2012年才在jdk1.7u4中可用。Oracle官方计划在jdk9将G1变成默认垃圾回收器以替代CMS。主要应用在多CPU和大内存服务器环境下，极大的减少垃圾收集的停顿时间，全面提升服务器的性能，逐步替代CMS

- 主要改变为Eden/Survivor/Tenured等内存区域不再是连续的了，而是变成了一个个大小一致的region，每个region从1M-32M不等。一个region可能是Eden/Survivor/Tenured

### 10.3 G1的优良特性

- G1能充分利用多CPU/多核环境硬件优势，尽量缩短STW
- G1整体上采用标记-整理算法，局部是通过复制算法，**不会产生内存碎片**
- 宏观上，G1之中不再区分年轻代和老年代。而是**把内存划分成了多个独立的子区域region(就像一个围棋的棋盘，一格一格)**
- G1收集器将整个内存区都混合在一起，但其本身依然在小范围内要进行年轻代和老年代的区分。保留了新生代和老年代
- G1虽然也是分代收集器，但整个内存分区**不存在物理上**的年轻代和老年代，也不需要完全独立的survivor(to space)堆做复制准备。G1只有逻辑上的分代概念，或者说每个分区都可能随G1的运行在不同代之间前后切换

### 10.4 G1底层原理

#### 10.4.1 Region区域化垃圾收集器

最大好处是化整为零，避免全内存扫描，只需要按照区域来进行扫描即可 

![1613663090957](E:\SoftwareNote\面试准备\JVM和GC\img\G1回收器的Region区域化.png)

#### 10.4.2 回收步骤

![1613663222627](E:\SoftwareNote\面试准备\JVM和GC\img\G1回收器回收步骤.png)

#### 10.4.3 G1回收器运行过程

![1613663306282](E:\SoftwareNote\面试准备\JVM和GC\img\G1回收器运行过程.png)

### 10.5 常用配置参数

- -XX:+UseG1GC
- -XX:G1HeapRegionSize=n : 设置G1区域的大小。值是2的幂，范围是1M到32M。目标是根据最小的Java堆大小划分出约2048个区域
- -XX:MaxGCPauseMillis=n : 最大停顿时间，这是个软目标，JVM将尽可能（但不保证）停顿时间小于这个时间
- -XX:InitiatingHeapOccupancyPercent=n  堆占用了多少的时候就触发GC，默认是45
- -XX:ConcGCThreads=n  并发GC使用的线程数
- -XX:G1ReservePercent=n 设置作为空闲空间的预留内存百分比，以降低目标空间溢出的风险，默认值是10%

### 10.6 和CMS相比的优势

- G1不会产生内存碎片

- 可以精确控制停顿。该收集器把整个堆（新生代/老年代）划分成多个固定大小的区域，每次根据允许停顿的时间去收集垃圾最多的区域

  

## 11. Java命令

### 11.1 jps 查看进程端口号pid

```shell
C:\Users\ASUS>jps
8704 Jps
28388
3560 Launcher
10428 Launcher
```

### 11.2 jmap -heap pid 查看堆内存分配情况 

```shell
C:\Users\ASUS>jmap -heap 10428
Attaching to process ID 10428, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.112-b15

using thread-local object allocation.
Parallel GC with 10 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 734003200 (700.0MB)
   NewSize                  = 133169152 (127.0MB)
   MaxNewSize               = 244318208 (233.0MB)
   OldSize                  = 267386880 (255.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 100663296 (96.0MB)
   used     = 6868864 (6.5506591796875MB)
   free     = 93794432 (89.4493408203125MB)
   6.8236033121744795% used
From Space:
   capacity = 16252928 (15.5MB)
   used     = 0 (0.0MB)
   free     = 16252928 (15.5MB)
   0.0% used
To Space:
   capacity = 16252928 (15.5MB)
   used     = 0 (0.0MB)
   free     = 16252928 (15.5MB)
   0.0% used
PS Old Generation
   capacity = 172490752 (164.5MB)
   used     = 8508624 (8.114456176757812MB)
   free     = 163982128 (156.3855438232422MB)
   4.932800107451558% used

6227 interned Strings occupying 530192 bytes.
```

### 11.3 jinfo [-flags] pid 查看VM参数详情 

```java
C:\Users\ASUS>jinfo -flags 12340
Attaching to process ID 12340, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.112-b15
Non-default VM flags: -XX:CICompilerCount=4 -XX:InitialHeapSize=400556032 -XX:MaxHeapSize=6404702208 -XX:MaxNewSize=2134900736 -XX:MetaspaceSize=134217728 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=133169152 -XX:OldSize=267386880 -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
Command line:  -XX:+PrintGCDetails -XX:MetaspaceSize=128m -javaagent:C:\Program Files (x86)\IDEA\IntelliJ IDEA 2018.2.2\lib\idea_rt.jar=6608:C:\Program Files (x86)\IDEA\IntelliJ IDEA 2018.2.2\bin -Dfile.encoding=UTF-8
```

### 11.4 jstack pid 查看栈信息(死锁等)

```shell
 E:\Java_Code\IDEA_CODE\ligong\UpUp2021\out\production\UpUp2021\com\codeman\JUCNThread>jstack 15912
  
  Java stack information for the threads listed above:
  ===================================================
  "线程2":
          at com.codeman.JUCNThread.DeadLock.run(DeadLockDemo.java:36)
          - waiting to lock <0x0000000740d96a60> (a java.lang.Object) // 相互等待
          - locked <0x0000000740d96a70> (a java.lang.Object)
          at java.lang.Thread.run(Thread.java:745)
  "线程1":
          at com.codeman.JUCNThread.DeadLock.run(DeadLockDemo.java:36)
          - waiting to lock <0x0000000740d96a70> (a java.lang.Object) // 相互等待
          - locked <0x0000000740d96a60> (a java.lang.Object)
          at java.lang.Thread.run(Thread.java:745)
  
  Found 1 deadlock.
```

