# JVM内存结构

## 1. 运行时数据区

主要包括：

- 堆
- 栈(虚拟机栈+本地方法栈)
- 方法区
- 程序计数器

JVM的优化问题主要在**线程共享的数据区**：**堆/方法区**

![1614075810740](E:\SoftwareNote\面试准备\JVM和GC\img\JVM内存结构图.png)

### 1.1 程序计数器

**程序计数器（Program Counter Register）**是一块较小的内存空间，可以看作是**当前线程**所执行字节码的**行号指示器**，指向下一个将要执行的指令代码，由执行引擎来读取下一条指令。更确切的说，**一个线程的执行，是通过字节码解释器改变当前线程的计数器的值，来获取下一条需要执行的字节码指令，从而确保线程的正确执行**。

为了确保线程切换后（**上下文切换**）能恢复到正确的执行位置，***每个线程都有一个独立的程序计数器***，各个线程的计数器互不影响，独立存储。也就是说程序计数器是***线程私有的内存***。

如果线程执行 Java 方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果执行的是 Native 方法，计数器值为Undefined。

**程序计数器不会发生内存溢出（OutOfMemoryError即OOM）问题。**

### 1.2 栈

- 分类

  - 虚拟机栈：JVM执行Java方法服务
  - 本地方法栈：JVM使用到的**Native方法**服务 

- 定义：**限定仅在表头进行插入和删除操作的线性表**。即**压栈（入栈）和弹栈（出栈）**都是对**栈顶元素**进行操作的。所以栈是**后进先出**的。 

  栈是**线程私有**的，他的生命周期与线程相同。每个线程都会分配一个栈的空间，即**每个线程拥有独立的栈空间**。  

- 栈中存储的是什么

  栈帧是栈的元素。每个方法在执行时都会创建一个栈帧。**栈帧中存储了局部变量表/操作数栈/动态连接/方法出口等信息。** 

  每个方法从调用到执行结束，就相当于一个栈帧压栈到出栈。

  ![1614076678103](E:\SoftwareNote\面试准备\JVM和GC\img\栈的栈帧元素存储的信息.png)

#### 1.2.1 局部变量表

栈帧中，有一个局部变量表存储数据。局部变量表中存储了

- 基本数据类型的局部变量（包括参数）
- 对象的引用

但是**不存储<u>对象</u>的内容**。局部变量表所需的内存空间在编译期间完成分配，因此在方法运行期间不会改变局部变量表的大小。

局部变量的容量以变量槽（Variable Slot）为最小单位，每个变量槽最大存储32位的数据类型，对于64位的double和long，JVM会分配两个slot来存储。

JVM通过局部变量表的索引来使用局部变量。其中普通方法和static方法有个区别，就是**非static方法**的局部变量表第一个slot为**该对象的实例引用this**。 

![1614077381249](E:\SoftwareNote\面试准备\JVM和GC\img\栈帧-局部变量表.png)

- slot可复用

  为了尽可能节省栈帧的空间，slot是可复用的。一个方法中的局部变量，它的作用范围不一定会覆盖整个方法(可能只有一小块代码块)。当一个变量已经过了它的作用范围(即变量失效)，那么该变量的slot可以交给其他变量使用。

  - demo

    ```java
    public void test(boolean flag)
    {
        if(flag)
        {
            int a = 66; // a的作用域为if flag
        }
        int b = 55; // a超出作用域，变量失效，b可复用a的slot
    }
    ```

  - slot复用影响系统垃圾收集行为

  ```java
  public class TestDemo {
   
      public static void main(String[] args){
          
          byte[] placeholder = new byte[64 * 1024 * 1024];
          
          System.gc(); // 此时GC，placeholder作用域还未超出，所以不会被回收
      }
  }
  ```

  ```java
  public class TestDemo {
   
      public static void main(String[] args){
          {
              byte[] placeholder = new byte[64 * 1024 * 1024];
          }
          // 此时GC，placeholder作用域已过，但是placeholder还是没有被回收
          System.gc(); 
      }
  }
  
  
  /**PrintGCDetails****/
  [GC (System.gc()) [PSYoungGen: 71434K->840K(114176K)] 71434K->66384K(375296K)
  [Full GC (System.gc()) [PSYoungGen: 840K->0K(114176K)] [ParOldGen: 65544K->66204K(261120K)] 66384K->66204K(375296K), 
  ```

  ```java
  public class TestDemo {
   
      public static void main(String[] args){
          {
              byte[] placeholder = new byte[64 * 1024 * 1024];
          }
          // 加入了一个新局部变量
          int a = 0;
          // 再GC发现placeholder被回收
          System.gc();
      }
  }
  
  /***PrintGCDetails****/
  [GC (System.gc()) [PSYoungGen: 71434K->872K(114176K)] 71434K->880K(375296K）
  [Full GC (System.gc()) [PSYoungGen: 872K->0K(114176K)] [ParOldGen: 8K->668K(261120K)] 880K->668K(375296K)
  ```

  这是因为，**局部变量是否被回收的关键在于**：**局部变量表中的slot是否还存有关于该变量对象的引用**。当被另一个新局部变量复用后，原变量再局部变量表中再没有引用了(被替换)，GC就会回收。

#### 1.2.2 操作数栈

**操作数栈**是一个**后进先出**栈。操作数栈的**元素可以是任意的Java数据类型**。方法刚开始执行时，操作数栈是**空的**，在方法执行过程中，通过字节码指令对操作数栈进行**压栈和出栈**的操作。通常进行**算数运算**的时候是通过操作数栈来进行的，又或者是在**调用其他方法的时候通过操作数栈进行参数传递**。操作数栈可以理解为栈帧中**用于计算的临时数据存储区**。  

- demo

```java
public class OperandStack{
 
    public static int add(int a, int b){
        int c = a + b;
        return c;
    }
 
    public static void main(String[] args){
        add(100, 98);
    }
}


/* javap -c -p 反编译获得虚拟机指令集  */
 public static int add(int, int);
    Code:
       0: iload_0 // 把局部变量0压栈（a）
       1: iload_1 // 把局部变量1压栈（b）
       2: iadd	  // 弹出2个变量，求和，结果压栈
       3: istore_2// 弹出结果，放于局部变量2
       4: iload_2 // 局部变量2压栈
       5: ireturn // 返回

```

![1614085724615](E:\SoftwareNote\面试准备\JVM和GC\img\栈帧-操作数栈执行过程.png)

- 栈中可能出现哪些异常(错误)
  - StackOverflowError：栈溢出。当一个线程再计算所需栈大小>配置允许的最大栈大小，则抛出该Error
  - OutOfMemoryError：内存不足。栈在动态扩展时无法申请到足够的内存。
  - 栈大小的设置
    - Xss参数设置，通常几百k就够用了。由于栈是线程私有的，线程越多，栈空间就占用越多
    - **栈决定了方法调用的深度**。因此要**慎用递归**。

### 1.3 堆

堆是Java虚拟机所管理的最大一块存储区域。**堆内存被所有线程共享**。主要放置的是new常见的对象(强引用)。所有的对象实例以及数组都要在堆上分配空间。垃圾收集器就是根据GC算法，回收堆上**对象所占用的内存空间**。

![1614086764326](E:\SoftwareNote\面试准备\JVM和GC\img\Java堆的分类.png)

Java堆分为：

- 年轻代（Young Generation）: 存储“新生对象”，当年轻代内存满时，会触发Minor GC，清理年轻代内存空间。

  - 伊甸园（Eden）

    研究表明，有将近 **98%的对象是朝生夕**死，所以针对这一现状，大多数情况下，对象会在新生代 Eden 区中进行分配。

    当 Eden 区没有足够空间进行分配时，虚拟机会发起一次 Minor GC，Minor GC 相比 Major GC 更频繁，回收速度也更快。

    通过 Minor GC 之后，Eden 会被清空，Eden 区中绝大部分对象会被回收，而那些无需回收的存活对象，将会进到 Survivor 的 From 区（若 From 区不够，则直接进入 Old 区）。

  - 幸存区（Survivor）

    Survivor 区相当于是 Eden 区和 Old 区的一个缓冲，Survivor 又分为 2 个区，一个是 From 区，一个是 To 区。每次执行 Minor GC，会将 Eden 区和 From 存活的对象放到 Survivor 的 To 区（如果 To 区不够，则直接进入 Old 区） 

    - From Survivor
    - To Survivor

- 老年代（Old Generation）：存储**长期存活**<u>(虚拟机给每个对象定义了一个对象年龄 Age 计数器。正常情况下对象会不断的在 Survivor 的 From 区与 To 区之间移动，对象在 Survivor 区中每经历一次 Minor GC，年龄就增加 1 岁。当年龄增加到 15 岁时，这时候就会被转移到老年代。)</u>的对象和**大对象**<u>(大对象指需要大量连续内存空间的对象，这部分对象不管是不是“朝生夕死”，都会直接进到老年代。这样做主要是为了避免在 Eden 区及 2 个 Survivor 区之间发生大量的内存复制。当你的系统有非常多“朝生夕死”的大对象时，需要注意。)</u> 和**动态对象年龄** <u>(虚拟机并不重视要求对象年龄必须到 15 岁，才会放入老年区，如果 Survivor 空间中相同年龄所有对象大小的总合大于 Survivor 空间的一半，年龄大于等于该年龄的对象就可以直接进去老年区。)</u>。年轻代的对象，经过多次GC后仍然存活的，就会被转移到老年代。当老年代内存不足时，则会触发FullGC。

  老年代占据着 2/3 的堆内存空间，只有在 Major GC 的时候才会进行清理，每次 GC 都会触发“Stop-The-World”。内存越大，STW 的时间也越长，所以内存也不仅仅是越大就越好。 

  FullGC清理的是整个堆空间（包括年轻代和老年代），Full GC后，堆中仍无法存储对象，则会抛出OOMError。

  **空间分配担保：**

  在发生 Minor GC 之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间。

  如果条件成立的话，Minor GC 是可以确保安全的。

  如果不成立，则虚拟机会查看 HandlePromotionFailure 设置是否担保失败，如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小。

  如果大于，尝试进行一次 Minor GC。

  如果小于或者 HandlePromotionFailure 不允许，则进行一次 Full GC。





**问题 1:为什么需要 Survivor？**

如果没有 Survivor 区，Eden 区每进行一次 Minor GC，存活的对象就会被送到老年代，老年代很快就会被填满。而有很多对象虽然一次 Minor GC 没有消灭，但其实或许第二次，第三次就需要被清除。

这时候移入老年区，很明显不是一个明智的决定。

所以，Survivor 的存在意义就是减少被送到老年代的对象，进而减少老年代 GC 的发生。Survivor 的预筛选保证，只有经历 15 次 Minor GC 还能在新生代中存活的对象，才会被送到老年代。

**问题 2:为什么需要 From 和 To 两个呢？**

这种机制最大的好处就是可以解决**内存碎片化**，整个过程中，永远有一个 Survivor 区是空的，另一个非空的 Survivor 区是无碎片的。

假设只有一个 Survivor 区。

Minor GC 执行后，Eden 区被清空了，存活的对象放到了 Survivor 区，而之前 Survivor 区中的对象，可能也有一些是需要被清除的。

那么问题来了，这时候我们怎么清除它们？

在这种场景下，我们只能标记清除，而我们知道标记清除最大的问题就是内存碎片，在新生代这种经常会消亡的区域，采用标记清除必然会让内存产生严重的碎片化。

因为 Survivor 有 2 个区域，所以每次 Minor GC，会将之前 Eden 区和 From 区中的存活对象复制到 To 区域。第二次 Minor GC 时，To 区 到 From 区 ，以此反复。



| 参数                            | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| -Xms                            | 堆内存初始大小                                               |
| -Xmx（MaxHeapSize）             | 堆内存最大允许大小，一般不要大于物理内存的80%                |
| -XX:NewSize（-Xns）             | 年轻代内存初始大小                                           |
| -XX:MaxNewSize（-Xmn）          | 年轻代内存最大允许大小，也可以缩写                           |
| -XX:NewRatio                    | 新生代和老年代的比值 值为4 表示 新生代:老年代=1:4，即年轻代占堆的1/5 |
| -XX:SurvivorRatio=8             | 年轻代中Eden区与Survivor区的容量比例值，默认为8 表示两个Survivor :Eden=2:8，即一个Survivor占年轻代的2/10,其中From和To各占一份 |
| -XX:+HeapDumpOnOutOfMemoryError | 内存溢出时，导出堆信息到文件                                 |
| -XX:+HeapDumpPath               | 堆Dump路径 -Xmx20m -Xms5m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=d:/a.dump |
| -XX:OnOutOfMemoryError          | 当发生OOM内存溢出时，执行一个脚本 -XX:OnOutOfMemoryError=D:/tools/jdk1.7_40/bin/printstack.bat %p %p表示线程的id pid |
| -XX:MaxTenuringThreshold=7      | 表示如果在幸存区移动多少次没有被垃圾回收，进入老年代         |

### 1.4 方法区（Java8被Metaspace元空间替代） 

方法区同样也是线程间共享的区间。用于存储：

- 已被虚拟机加载的类信息（版本/方法/字段等）
- **常量**
- 静态变量
- 即使编译器编译后的代码（运行时常量池）



元空间并不在JVM中，而是使用的**本地内存 (计算机内存)**。元空间两个参数：

- MetaSpaceSize：初始元空间的大小，控制发生GC的阈值
- MaxMetaspaceSize：限制元空间大小的上限，防止异常占用过多的物理内存。 



常量池中存储遍历器生成的各种字面量和符号引用。

- 常量池

  - 字面量
    - final修饰的常量
    - 基本数据类型的值
    - 字符传
  - 符号引用
    - 类和接口的全限定名
    - 方法名和描述符
    - 字段名和描述符

- 常量池的作用

  **优点：**常量池避免了频繁的创建和销毁对象而影响系统性能，其实现了对象的共享。

  举个栗子： Integer 常量池（缓存池），和字符串常量池