# JUC多线程和并发包

## 0. JUC ： java.util.concurrent

## 1. volatile

- 是什么

  volatile是java虚拟机提供的轻量级同步机制。

- 三大特性(JMM三大特性) 【详见JVM和并发包笔记】

  - 保证可见性（共享变量修改，各线程都可见）
  - 不保证原子性(比如++操作，实际编译后有三个步骤，无法保证原子性【同时成功/同时失败】，所以多线程下会错乱)
  - 禁止指令重排（内存屏障）

- 运用：

  - DCL(Double Check Lock 双端检锁)单例模式中禁止实例化对象指令重排

  ```java
  public static SingletonWithDCL getInstance() {
          if (instance == null) { // 检查1
              synchronized (SingletonWithDCL.class) {
                  if (instance == null) // 检查2
                      instance = new SingletonWithDCL();
              }
          }
          return instance;
      }
  ```

  

  ```java
  instance=new SingletonDem(); 可以分为以下步骤(伪代码)
  memory=allocate();//1.分配对象内存空间
  instance(memory);//2.初始化对象
  instance=memory;//3.设置instance的指向刚分配的内存地址,此时instance!=null 
  
  步骤2和步骤3不存在数据依赖关系.而且无论重排前还是重排后程序执行的结果在单线程中并没有改变,因此这种重排优化是允许的.
  memory=allocate();//1.分配对象内存空间
  instance=memory;//3.设置instance的指向刚分配的内存地址,此时instance!=null 但对象还没有初始化完.
  instance(memory);//2.初始化对象
  
  所以当一条线程访问instance不为null时,由于instance实例未必完成初始化,也就造成了线程安全问题.
  ```

  

## 2. CAS （Compare And Swap） ： 比较并交换

- 介绍

  CAS的全称为Compare-And-Swap ,它是一条**CPU并发原语**.

  它的功能是**判断内存某个位置的值是否为预期值,如果是则更新为新的值,这个过程是原子的**.

  CAS并发原语提现在Java语言中就是sun.miscUnSaffe类中的各个方法.调用UnSafe类中的CAS方法,JVM会帮我实现CAS汇编指令.这是一种完全依赖于硬件 功能,通过它实现了原子操作,再次强调,由于CAS是一种系统原语,原语属于操作系统用于范畴,是由若干条指令组成,用于完成某个功能的一个过程,并且原语的执行必须是连续的,在执行过程中不允许中断,也即是说CAS是一条原子指令,不会造成所谓的数据不一致的问题.

- 由AtomicInteger.getAndIncrement()引出

- AtomicInteger.java

  ```java
  /**
  * Atomically increments by one the current value.
  * 该Integer++
  * @return the previous value
  */
  public final int getAndIncrement() {
      return unsafe.getAndAddInt(this, valueOffset, 1);
  }
  
  // setup to use Unsafe.compareAndSwapInt for updates
  private static final Unsafe unsafe = Unsafe.getUnsafe();
  private static final long valueOffset;
  
  // 用volatile修饰的对象值。可见性，一发生改变即收到通知
  private volatile int value;
  ```

- Unsafe.java

  1).UnSafe

   是CAS的核心类 由于Java 方法无法直接访问底层 ,需要通过本地(native)方法来访问,UnSafe相当于一个后面,基于该类可以直接操作特额定的内存数据.UnSafe类在于sun.misc包中,其内部方法操作可以向C的指针一样直接操作内存,因为Java中CAS操作的助兴依赖于UNSafe类的方法.UnSafe类在于sun.misc包中, 是CAS的核心类 由于Java 方法无法直接访问底层 ,需要通过本地(native)方法来访问,UnSafe相当于一个后面,基于该类可以直接操作特额定的内存数据.UnSafe类在于sun.misc包中,其内部方法操作可以向C的指针一样直接操作内存,因为Java中CAS操作的助兴依赖于UNSafe类的方法.

  注意UnSafe类中所有的方法都是native修饰的,也就是说UnSafe类中的方法都是直接调用操作底层资源执行响应的任务

   2).变量ValueOffset,便是该变量在内存中的偏移地址,因为UnSafe就是根据内存偏移地址获取数据的偏移地址 

  ```java
  /**
  * 尝试做加法
  */
  public final int getAndAddInt(Object var1, long var2, int var4) {
      int var5;
      do {
          // 从内存var2中拿到该对象var1当前值var5
          var5 = this.getIntVolatile(var1, var2);
      } 
      // 比较并交换
      // 再从内存var2中拿到该对象var1当前值var5_real(实际值)和刚才取到的值var5(期望值)做比较，
      // 如果var5_real==var5，则可以进行更新，返回var5
      while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
  
      return var5;
  }
  
  /**
  * 从内存var2中拿到该对象var1当前值
  */
  public native int getIntVolatile(Object var1, long var2);
  
  /**
  * 比较并交换，
  * return true：符合预计并修改成功； false: 不符合预期值
  */
  public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
  ```

  执行过程

  ![1610898057637](E:\SoftwareNote\面试准备\多线程\img\AtomicInteger操作Unsafe过程.png)

Unsafe底层汇编

![1610898543817](E:\SoftwareNote\面试准备\多线程\img\Unsafe底层汇编.png)

- CAS缺点
  - 循环时间开销很大，如果CAS一直不成功，会给CPU带来很大的开销(一直在循环)
  - 只能保证一个this共享变量的原子性(多个变量就需要加锁了)
  - ABA问题(最终一致，过程发生变化)



## 3. AtomicReference/AtomicStampedReference原子引用

- CAS都是一些包装类，原子引用AtomicReference<T>可以对引用类型进行CAS
- 由于CAS存在ABA问题，只能保证最终一致，而过程可能发生变化。所有就有带时间戳的原子引用AtomicStampedReference。必须版本+引用对象都一致才可CAS

```java
AtomicStampedReference(V initialRef, int initialStamp); // initialStamp 初始版本号

boolean compareAndSet(V   expectedReference,
                      V   newReference,
                      int expectedStamp, // expectedStamp 期望版本号
                      int newStamp);	//  newStamp 更新版本号
```



## 4. ArrayList线程不安全

- 不安全原因：读写操作都没加锁

```java
 public void add(int index, E element);
 public boolean add(E e);
```

- 并发修改异常 java.util.concurrentModificationException

- 解决方案：

  - new Vector();

  ```java
  public void add(E e) {
      int i = cursor;
      synchronized (Vector.this) {
          checkForComodification();
          Vector.this.add(i, e);
          expectedModCount = modCount;
      }
      cursor = i + 1;
      lastRet = -1;
  }
  
  public synchronized E get(int index) {
      if (index >= elementCount)
          throw new ArrayIndexOutOfBoundsException(index);
  
      return elementData(index);
  }
  ```

  - Collections.synchronizedList

  ```java
  public E set(int index, E element) {
      synchronized (mutex) {return list.set(index, element);}
  }
  public void add(int index, E element) {
      synchronized (mutex) {list.add(index, element);}
  }
  ```

  - CopyOnWriteArrayList

  ```java
  public boolean add(E e) {
      final ReentrantLock lock = this.lock;
      lock.lock();
      try {
          Object[] elements = getArray();
          int len = elements.length;
          Object[] newElements = Arrays.copyOf(elements, len + 1); // 扩容一位
          newElements[len] = e; // 扩容出来的给新元素
          setArray(newElements); // 指针指向新数组
          return true;
      } finally {
          lock.unlock();
      }
  }
  ```

- 同理，有Set和Map

  - set用CopyOnWriteArraySet，底层也是CopyOnWriteArrayList

  ```java
  public CopyOnWriteArraySet() {
      al = new CopyOnWriteArrayList<E>();
  }
  
  public boolean add(E e) {
      return al.addIfAbsent(e);
  }
  
  // 去重添加
  public boolean addIfAbsent(E e) {
      Object[] snapshot = getArray();
      return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
      addIfAbsent(e, snapshot);
  }
  ```

  - map用ConcurrentHashMap



## 5. 公平锁/非公平锁/可重入锁/递归锁/自旋锁

### 5.1 公平锁/非公平锁

- 概念公平锁： 是指多个线程按照申请锁的顺序来获取锁类似排队打饭 先来后到

  非公平锁：是指在多线程获取锁的顺序并不是按照申请锁的顺序,有可能后申请的线程比先申请的线程优先获取到锁,在高并发的情况下,有可能造成**优先级反转**或者**饥饿现象**(运气不好的，从头等到尾都没执行)

- 实例化

```java
/**
 * Creates an instance of {@code ReentrantLock}.
 * This is equivalent to using {@code ReentrantLock(false)}.
 */
public ReentrantLock() { // 默认是非公平锁
    sync = new NonfairSync();
}

/**
 * Creates an instance of {@code ReentrantLock} with the
 * given fairness policy.
 *
 * @param fair {@code true} if this lock should use a fair ordering policy
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

- 对比

Java ReentrantLock而言,

通过构造哈数指定该锁是否是公平锁 默认是非公平锁 非公平锁的优点在于吞吐量比公平锁大.

对于synchronized而言 也是一种非公平锁.



### 5.2 可重入锁(又名递归锁) ReentrantLock

- 概念： 指同一线程再外层函数获得锁之后，内层递归(调用)函数仍然可以获取该锁的代码。也就是说同一线程可以进入到它已获得锁的方法的全部调用方法。
- 典型可重入锁：ReentrantLock/synchronized
- 最大优势：避免死锁
- demo

```java
private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
//        new Thread(ReentrantLockDemo::test02, "t2").start();
//        new Thread(ReentrantLockDemo::test01, "t1").start(); // t2先拿到锁，t1会等待t2

        new Thread(ReentrantLockDemo::test04, "t4").start();
        new Thread(ReentrantLockDemo::test03, "t3").start(); // t4先拿到锁，t3会等待t4

    }


    public static void test01() {
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + ", test01");
            test02();
        } finally {
            lock.unlock();
//            lock.unlock(); // 释放过多锁 java.lang.IllegalMonitorStateException
        }
    }

    public static void test02() {
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + ", test02  into");
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getName() + ", test02");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static synchronized void test03() {
        System.out.println(Thread.currentThread().getName() + ", test03");
        test04();
    }

    public static synchronized void test04() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ", test04");
    }	
```

### 5.3 自旋锁 SpinLock

- 概念:  尝试获取锁的先成功不会立即阻塞，而是采用**循环的方式取尝试获取锁**，好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU
- demo

```java
	private AtomicReference<Thread> atomicReference = new AtomicReference<>();

    private int num = 0;

    public void mySpinLock() {
        Thread thread = Thread.currentThread();
        while (!atomicReference.compareAndSet(null, thread)) {

        }
    }

    public void myUnSpinLock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread, null);
    }

    public static void main(String[] args) {
        SpinLockDemo spinLockDemo = new SpinLockDemo();

        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                spinLockDemo.mySpinLock();
                spinLockDemo.doSometing();
                spinLockDemo.myUnSpinLock();
            }, "t" + i).start();
        }

    }


    public void doSometing() {
        System.out.printf("【%s】线程， do something.. 【%s】 ", Thread.currentThread(), ++num);
        System.out.println();
    }
```



### 5.4 独占锁(写)/共享锁(读)/互斥锁 ReentrantReadWriteLock

- 概念：

  - 独占锁： 该锁只能被一个线程持有，Reentrant和synchronized都是独占锁。

  - 共享锁：指该锁可以被多个线程持有。

    对于ReentrantReadWriteLock而言，其读锁是共享锁，写锁是独占锁。

  读锁的共享锁可以保证并发读是非常高效的，读写/写读/写写的过程是互斥的。

- demo

```java
    private volatile Map<String, String> map = new HashMap<>();

    private ReentrantLock reentrantLock = new ReentrantLock();

    private ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();

    public void putMap(String key, String value) {
        try {
//            reentrantLock.lock();
            reentrantReadWriteLock.writeLock().lock();
            System.out.printf("【%s】线程， 写【%s】， start ... \r\n", Thread.currentThread().getName(), key);
            map.put(key, value);
            Thread.sleep(10);
            System.out.printf("【%s】线程， 写【%s】， end ... \r\n", Thread.currentThread().getName(), key);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
//            reentrantLock.unlock();
            reentrantReadWriteLock.writeLock().unlock();
        }
    }

    public String getMap(String key) {
        try {
            reentrantReadWriteLock.readLock().lock();
            System.out.printf("【%s】线程， 读【%s】， start ... \r\n", Thread.currentThread().getName(), key);
            return map.get(key);
        } finally {
            System.out.printf("【%s】线程， 读【%s】， end ... \r\n", Thread.currentThread().getName(), key);
            reentrantReadWriteLock.readLock().unlock();
        }
    }

    public static void main(String[] args) {

        ReentrantReadWriteLockDemo reentrantReadWriteLockDemo = new ReentrantReadWriteLockDemo();

        for (int i = 0; i < 5; i++) {
            final int ii = i;
            new Thread(() -> {
                reentrantReadWriteLockDemo.putMap(ii+"", ii + "");
            }).start();
        }



        for (int i = 0; i < 5; i++) {
            final int ii = i;
            new Thread(() -> {
                System.out.println(reentrantReadWriteLockDemo.getMap(ii + ""));
            }).start();
        }
    }

// *************************************
【Thread-0】线程， 写【0】， start ... 
【Thread-0】线程， 写【0】， end ... 
【Thread-2】线程， 写【2】， start ... 
【Thread-2】线程， 写【2】， end ... 
【Thread-1】线程， 写【1】， start ... 
【Thread-1】线程， 写【1】， end ... 
【Thread-3】线程， 写【3】， start ... 
【Thread-3】线程， 写【3】， end ... 
【Thread-4】线程， 写【4】， start ... 
【Thread-4】线程， 写【4】， end ... 
【Thread-5】线程， 读【0】， start ... 
【Thread-5】线程， 读【0】， end ... 
【Thread-7】线程， 读【2】， start ... 
【Thread-7】线程， 读【2】， end ... 
【Thread-6】线程， 读【1】， start ...  // 读是可以并发的
【Thread-8】线程， 读【3】， start ...  // 读是可以并发的
【Thread-8】线程， 读【3】， end ... 
【Thread-9】线程， 读【4】， start ... 
【Thread-9】线程， 读【4】， end ...   
【Thread-6】线程， 读【1】， end ... 
```

## 6. CountDownLatch/CyclicBarrier/Semaphore

### 6.1 CountDownLatch 

- 让一些线程阻塞直到另外一些完成后才被唤醒
- CountDownLatch主要有两个方法,**当一个或多个线程调用await方法时,调用线程会被阻塞** .其他线程调用countDown方法计数器减1(调用countDown方法时线程不会阻塞),**当计数器的值变为0** ,因调用await方法被阻塞的线程会被唤醒,继续执行
- demo

```java
CountDownLatch countDownLatch = new CountDownLatch(6);

for (int i = 0; i < 6; i++) {
    final int ii = i + 1;
    new Thread(() -> {
        System.out.printf("[%s]被灭……", Thread.currentThread().getName());
        System.out.println();
        countDownLatch.countDown();
    }, EnumDemo.getValueById(ii)).start();
}

try {
    countDownLatch.await();
} catch (InterruptedException e) {
    e.printStackTrace();
}
System.out.println("[秦国]统一中国。");
}
// *****************************
[赵国]被灭……
[魏国]被灭……
[齐国]被灭……
[楚国]被灭……
[燕国]被灭……
[韩国]被灭……
[秦国]统一中国。
```



### 6.2 CyclicBarrier

- CyclicBarrier的字面意思是可循环(Cyclic) 使用的屏障(barrier).它要做的事情是,让一组线程到达一个屏障(也可以叫做同步点)时被阻塞,直到最后一个线程到达屏障时,屏障才会开门,所有被屏障拦截的线程才会继续干活,线程进入屏障通过CyclicBarrier的await()方法.
- demo

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> System.out.println("集齐七颗龙珠，准备召唤神龙...."));

for (int i = 0; i < 7; i++) {
    final int ii = i;
    new Thread(() -> {
        try {
            System.out.println("获得" + Thread.currentThread().getName() + "星球，等待其他星球收齐……");
            // 线程屏障
            cyclicBarrier.await();
            System.out.println("全部收集完毕，" + Thread.currentThread().getName());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }, "[" + ii + "]").start();
}

// **********************************
获得[0]星球，等待其他星球收齐……
获得[2]星球，等待其他星球收齐……
获得[1]星球，等待其他星球收齐……
获得[4]星球，等待其他星球收齐……
获得[3]星球，等待其他星球收齐……
获得[6]星球，等待其他星球收齐……
获得[5]星球，等待其他星球收齐……
集齐七颗龙珠，准备召唤神龙....
全部收集完毕，[5]
全部收集完毕，[0]
全部收集完毕，[2]
全部收集完毕，[3]
全部收集完毕，[6]
全部收集完毕，[4]
全部收集完毕，[1]
```

### 6.3 Semaphore 信号量 

- 信号量的主要用户两个目的,一个是用于多个共享资源的相互排斥使用,另一个用于并发资源数的控制
- demo

```java
 // 3个车位
Semaphore semaphore = new Semaphore(3);

for (int i = 0; i < 6; i++) {
    new Thread(() -> {
        try {
            semaphore.acquire(); // 获得，抢到车位
            System.out.println(Thread.currentThread().getName() +", 抢到车位……");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() +", 离开车位……");
            semaphore.release(); // 释放占用
        }
    }, String.valueOf(i)).start();
}

// **********************************************
0, 抢到车位……
3, 抢到车位……
1, 抢到车位……
0, 离开车位……
3, 离开车位……
4, 抢到车位……
2, 抢到车位……
1, 离开车位……
5, 抢到车位……
4, 离开车位……
2, 离开车位……
5, 离开车位……

```

## 7. 阻塞队列 BlockingQueue

- 阻塞队列

  当队列是空的时，线程task操作会被阻塞；当队列满的时候，线程put操作会被阻塞。

![1611662793087](E:\SoftwareNote\面试准备\多线程\img\阻塞队列图解.png)

- 好处：

  在concurrent包 发布以前,在多线程环境下,我们每个程序员都必须自己去控制这些细节,尤其还要兼顾效率和线程安全,而这会给我们的程序带来不小的复杂度.

  而发布后，我们不需要关心什么时候需要阻塞线程,什么时候需要唤醒线程,因为BlockingQueue都一手给你包办好了

- BlockingQueue核心API

  ![1611663298626](E:\SoftwareNote\面试准备\多线程\img\BlockingQueue核心API.png)

  抛出异常组：

  add(e)  添加， java.lang.IllegalStateException: Queue full

  remove() 先进先出, java.util.NoSuchElementException 

  element() 看首位，java.util.NoSuchElementException

  特殊值组：

  boolean offer(e) : return true-加入成功;false-加入失败

  E poll() ： 加入成功，返回**首位**

  E peek()：返回首位

  阻塞：

  put(e) : 队列满了，插不进去会阻塞

  E take()：队列为空，拿不到阻塞

  超时：

  阻塞一段时间，后不阻塞

- 架构介绍

  ![img](E:\SoftwareNote\面试准备\多线程\img\阻塞队列架构.png)

- 阻塞队列种类

  - ArrayBlockingQueue: 由数组结构组成的有界阻塞队列.

  - LinkedBlockingDeque: 由链表结构组成的有界(但大小默认值Integer>MAX_VALUE)阻塞队列.

  - PriorityBlockingQueue:支持优先级排序的无界阻塞队列.

  - DelayQueue: 使用优先级队列实现的延迟无界阻塞队列.

  - SynchronousQueue:不存储元素的阻塞队列,也即是单个元素的队列.

    SynchronousQueue没有容量

    与其他BlcokingQueue不同,SynchronousQueue是一个不存储元素的BlcokingQueue

    每个put操作必须要等待一个take操作,否则不能继续添加元素,反之亦然.

  - LinkedTransferQueue:由链表结构组成的无界阻塞队列.

  - LinkedBlockingDeque:由了解结构组成的双向阻塞队列.

- 阻塞队列实现生产者消费者

```java
private BlockingQueue<String> blockingQueue = null;
    private volatile boolean FLAG = true;
    private AtomicInteger atomicInteger = new AtomicInteger(1);

    public ImplementWithBlockingQueue() {
        this.blockingQueue = new ArrayBlockingQueue(3);
    }

    public ImplementWithBlockingQueue(BlockingQueue<String> blockingQueue) {
        if (blockingQueue == null) {
            new ImplementWithBlockingQueue();
        } else {
            this.blockingQueue = blockingQueue;
        }
    }

    @Override
    public void product() {
        while (FLAG) {
            try {
                // 尝试加入队列，阻塞2秒
                int andIncrement = atomicInteger.getAndIncrement();
                boolean offer = blockingQueue.offer( andIncrement + "", 2, TimeUnit.SECONDS);
                if (offer) {
                    System.out.println(getCurrThreadName() + "加入队列成功：" + andIncrement);
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void consumer() {
        while (FLAG) {
            try {
                // 尝试加入队列，阻塞2秒
                String str = blockingQueue.poll(2, TimeUnit.SECONDS);
                System.out.println(getCurrThreadName() + "取出队列成功：" + str);
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void STW() {
        this.FLAG = false;
    }

    private String getCurrThreadName() {
        return Thread.currentThread().getName();
    }
```



## 8.  Lock 与synchonized的区别

- 原始构成：

  - synchronized时关键字，属于JVM层面。通过字节码可以看出synchronized会有monitorenter和monitorexit*2(第二次是异常时的中断)关键字。底层是通过monitor对象来完成的，wait/notify等方法也依赖于monitor对象。
  - Lock是具体类(java.util.concurrent.locks.lock)，是API层面的锁

- 使用方法：

  - synchronized不需要手动释放锁，完成synchronized代码块后，会自动释放锁monitorexit
  - lock需要用户主动释放锁unlcok()(要加上finally上，异常也要释放锁)，否则可能出现死锁线程

- 等待是否可中断

  - synchronized不可中断，除非抛出异常或则正常运行结束

  - lock可中断

    - trylock-unlock

      ```java
      try {
          if (reentrantLock.tryLock(2, TimeUnit.SECONDS)) {
              try {
                  // if为true，说明拿到了锁，拿到锁就要解锁
                  System.out.println("获取锁");
              } finally {
                  reentrantLock.unlock();
              }
      
          }
      } catch (InterruptedException e) {
          e.printStackTrace();
      }
      ```

    - lockInterruptibly-interrupt

      ```java
      ReentrantLock reentrantLock = new ReentrantLock();
      
      Thread thread1 = new Thread(() -> {
          try {
              String name = Thread.currentThread().getName();
              System.out.println(name + "0");
              reentrantLock.lockInterruptibly();
              System.out.println(name + "1");
              Thread.sleep(5000);
              System.out.println(name + "2");
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              reentrantLock.unlock();
              System.out.println("解锁");
          }
      });
      
      Thread thread2 = new Thread(() -> {
          try {
              String name = Thread.currentThread().getName();
              reentrantLock.lockInterruptibly();
              System.out.println(name + "1");
              Thread.sleep(5000);
              System.out.println(name + "2");
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              try {
                  reentrantLock.unlock();
              } catch (IllegalMonitorStateException e) {
                  System.out.println("中断异常");
              }
      
          }
      });
      thread1.start();
      try {
          Thread.sleep(1000);
      } catch (InterruptedException e) {
          e.printStackTrace();
      }
      thread2.start();
      thread2.interrupt();
      ```

- 是否公平锁

  - synchronized为非公平锁
  - ReentrantLock默认非公平锁，ReentrantLock(true)则为公平锁

- 锁可绑定多个条件

  - synchronized不可
  - lock可实现分组唤醒，condition，每一个condition内的方法都是隔离的

  ```java
  ReentrantLock reentrantLock = new ReentrantLock();
  Condition condition1 = reentrantLock.newCondition();
  Condition condition2 = reentrantLock.newCondition();
  
  new Thread(() -> {
      String name = Thread.currentThread().getName();
      try {
          reentrantLock.lock();
          try {
              System.out.println(name + "await before");
              Thread.sleep(1000);
              condition1.await();
              System.out.println(name + "await after");
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      } finally {
          reentrantLock.unlock();
      }
  }).start();
  
  new Thread(() -> {
      String name = Thread.currentThread().getName();
      try {
          reentrantLock.lock(); // 线程1 condition1.await(); 线程2获取锁
          System.out.println(name + "获取锁"); 
          Thread.sleep(5000);
          condition1.signal(); // 线程2，5秒后唤醒condition1的await
      } catch (InterruptedException e) {
          e.printStackTrace();
      } finally {
          reentrantLock.unlock();
      }
  }).start();
  ```

## 9. Callable

- 
- demo

```java
Callable<String> callable = new Callable() {
            @Override
            public Object call() throws Exception {
                System.out.println("进入");
                Thread.sleep(2000);
                return "返回值";
            }
        };
        FutureTask<String> futureTask = new FutureTask<String>(callable);

        new Thread(futureTask).start();

        try {
            String s = futureTask.get();
            System.out.println("Main线程逻辑");
            System.out.println("result" + s);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
```



## 10. 线程池

### 10.1 线程池的优点

- 线程池做的工作主要是控制运行的线程的数量,处理过程中将任务加入队列,然后在线程创建后启动这些任务,如果先生超过了最大数量,超出的数量的线程排队等候,等其他线程执行完毕,再从队列中取出任务来执行.

  他的主要特点为:线程复用:控制最大并发数:管理线程.

  第一:**降低资源消耗**.通过重复利用自己创建的线程降低线程创建和销毁造成的消耗.

  第二: 提高响应速度.当任务到达时,任务可以不需要等到线程和粗昂就爱你就能立即执行.

  第三: 提高线程的可管理性.线程是稀缺资源,如果无限的创阿金,不仅会消耗资源,还会较低系统的稳定性,使用线程池可以进行统一分配,调优和监控.

### 10.2 线程池架构

![1612538798920](E:\SoftwareNote\面试准备\多线程\img\线程池架构图.png)

### 10.3 线程池创建API 

- Executors.newFixedThreadPool(int)  执行一个长期的任务,性能好很多

  ```java
  public static ExecutorService newFixedThreadPool(int nThreads) {
      return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>());
  }
  
  主要特点如下:
  
  1.创建一个定长线程池,可控制线程的最大并发数,超出的线程会在队列中等待.定长线程池
  
  2.newFixedThreadPool创建的线程池corePoolSize和MaxmumPoolSize是 相等的,它使用的的LinkedBlockingQueue
  ```

- Executors.newSingleThreadExecutor() 一个任务一个线程执行的任务场景

  ```java
  public static ExecutorService newSingleThreadExecutor() {
      return new FinalizableDelegatedExecutorService
          (new ThreadPoolExecutor(1, 1,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>()));
  }
  
  主要特点如下:
  1.创建一个单线程化的线程池,它只会用唯一的工作线程来执行任务,保证所有任务都按照指定顺序执行.
  2.newSingleThreadExecutor将corePoolSize和MaxmumPoolSize都设置为1,它使用的的LinkedBlockingQueue
  ```

- Executors.newCachedThreadPool() 适用:执行很多短期异步的小程序或者负载较轻的服务器l>

  ```java
  public static ExecutorService newCachedThreadPool() {
      return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
  }
  主要特点如下:
  1.创建一个可缓存线程池,如果线程池长度超过处理需要,可灵活回收空闲线程,若无可回收,则创建新线程.
  2.newCachedThreadPool将corePoolSize设置为0,MaxmumPoolSize设置为Integer.MAX_VALUE,它使用的是SynchronousQUeue,也就是说来了任务就创建线程运行,如果线程空闲超过60秒,就销毁线程
  ```

- Executors.newWorkStealingPool(int); 

  java8新增,使用目前机器上可以的处理器作为他的并行级别

- demo

  ```java
  ExecutorService executorService2 = Executors.newFixedThreadPool(3);
  try {
      for (int i = 0; i < 200; i++) {
          executorService2.execute(() -> {
              System.out.println(Thread.currentThread().getName() + "执行任务");
          });
      }
  } finally {
      // 最后shutdow就可以了，就像关闭流一样
      executorService2.shutdown();
  }
  ```

- 前3个底层都是ThreadPoolExecutor

  ```java
  public ThreadPoolExecutor(
      int corePoolSize, // 线程池中的常驻核心线程数
      int maximumPoolSize, // 线程池能够容纳同时执行的最大线程数,此值大于等于1
      long keepAliveTime, // 多余的空闲线程存活时间,当空间时间达到keepAliveTime值时,多余的线程会被销毁直到只剩下corePoolSize个线程为止
      TimeUnit unit, // 空闲工作线程存活时间单位
      BlockingQueue<Runnable> workQueue, // 任务队列,被提交但尚未被执行的任务.
      ThreadFactory threadFactory, // 表示生成线程池中工作线程的线程工厂,用户创建新线程,一般用默认即可
      RejectedExecutionHandler handler) { // 拒绝策略,表示当线程队列满了并且工作线程大于等于线程池的最大显示 数(maxnumPoolSize)时如何来拒绝.
      }
  ```

### 10.4 线程池工作原理

corePool有位置则执行，否则进入阻塞队列等待，阻塞队列满之后则会拓展corePool至MaximumPoolSize

MaximumPoolSize和阻塞队列都满了，则执行拒绝策略

![1612540179603](E:\SoftwareNote\面试准备\多线程\img\线程池工作原理.png)

### 10.5 线程池拒绝策略RejectExecutionHandler

- AbortPolicy(默认):直接抛出RejectedException异常阻止系统正常运行

  ```java
  java.util.concurrent.RejectedExecutionException: Task com.codeman.JUCNThread.ExecutorsDemo$$Lambda$1/1078694789@3b9a45b3 rejected from java.util.concurrent.ThreadPoolExecutor@7699a589[Running, pool size = 5, active threads = 4, queued tasks = 0, completed tasks = 3]
  ```

- CallerRunPolicy:"调用者运行"一种调节机制,该策略既不会抛弃任务,也不会抛出异常,而是让调用者执行

- DiscardOldestPolicy:抛弃队列中等待最久的任务,然后把当前任务加入队列中尝试再次提交

- DiscardPolicy:直接丢弃任务,不予任何处理也不抛出异常.如果允许任务丢失,这是最好的拒绝策略

### 10.6 线程池的使用

- 不调用API的Executors生成，而是手写底层的ThreadPoolExecutor实现线程池。

  因为API中的阻塞队列都是拥有最大长度，可能出现OOM的情况

  ![1612540566239](E:\SoftwareNote\面试准备\多线程\img\不使用Executors创建线程池的原因.png)

- demo

  ```java
  ExecutorService executorService2 = new ThreadPoolExecutor(3,
                                                            5,
                                                            2,
                                                            TimeUnit.SECONDS, 
                                                            new LinkedBlockingQueue<>(3));
  try {
      for (int i = 0; i < 9; i++) {
          executorService2.execute(() -> {
              System.out.println(Thread.currentThread().getName() + "执行任务");
          });
      }
  } finally {
      executorService2.shutdown();
  }
  
  ```

### 10.7 线程池的选型：线程数

- CPU密集型

  ![1612541886150](E:\SoftwareNote\面试准备\多线程\img\线程池的选型-CPU密集型.png)

- IO密集型

  - 由于IO密集型任务线程不一定一直在执行任务，应尽量多的线程，如CPU核数*2
  - ![1612542009117](E:\SoftwareNote\面试准备\多线程\img\线程池选型-IO密集型.png)

## 11. 死锁编码及定位分析

- 是什么：两个或两个以上的线程在执行过程中，因抢夺资源而造成的一种互相等待的现象，若无外力干涉，永远无法停下。所有系统资源重组，进程的资源请求都能够等到满足，死锁出现的可能性就很低，否则就会因争夺有限的资源而陷入死锁。

  ![1612709384126](E:\SoftwareNote\面试准备\多线程\img\线程死锁.png)

- 产生原因：

  - 系统资源不足
  - 进程运行推进的顺序不合适
  - 资源分配不当

- demo

```java
public class DeadLockDemo {
    public static void main(String[] args) {
        Object lockA = new Object();
        Object lockB = new Object();
        ;;;;;;DeadLock deadLock1 = new DeadLock(lockA, lockB);
        ;;;;;;DeadLock deadLock2 = new DeadLock(lockB, lockA);
        new Thread(deadLock1, "线程1").start();
        new Thread(deadLock2, "线程2").start();
    }
}

class DeadLock implements Runnable{

    private Object lockA;
    private Object lockB;

    public DeadLock(Object lockA, Object lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {
        String name = Thread.currentThread().getName();
        synchronized(lockA) {
            System.out.println(name + ", 抢到锁" + lockA + ", 等待锁" + lockB);
            synchronized (lockB) {
                System.out.println(name + ", 抢到锁" + lockB);
            }
        }
    }
}
```

- 排查问题

  - jps命令定位进程编号: jps -l
  ```shell
  E:\Java_Code\IDEA_CODE\ligong\UpUp2021\out\production\UpUp2021\com\codeman\JUCNThread>jps -l
  15152 sun.tools.jps.Jps
  15912 com.codeman.JUCNThread.DeadLockDemo
  4264
  12364 org.jetbrains.jps.cmdline.Launcher
  ```

  ​	jps命令介绍:

    ```md
    jps 命令类似与 linux 的 ps 命令，但是它只列出系统中所有的 Java 应用程序。 通过 jps 命令可以方便地查看 Java 进程的启动类、传入参数和 Java 虚拟机参数等信息。
    
    如果在 linux 中想查看 java 的进程，一般我们都需要 ps -ef | grep java 来获取进程 ID。
    如果只想获取 Java 程序的进程，可以直接使用 jps 命令来直接查看
    
    -q：只输出进程 ID
    -m：输出传入 main 方法的参数
    -l：输出完全的包名，应用主类名，jar的完全路径名
    -v：输出jvm参数
    -V：输出通过flag文件传递到JVM中的参数
    ```

  ​	jsp原理

    ```md
    java程序在启动以后，会在java.io.tmpdir指定的目录下，就是临时文件夹里，生成一个类似于hsperfdata_User的文件夹，这个文件夹里（在Linux中为/tmp/hsperfdata_{userName}/），有几个文件，名字就是java进程的pid，因此列出当前运行的java进程，只是把这个目录里的文件名列一下而已。 至于系统的参数什么，就可以解析这几个文件获得。
    
    window系统显示如下：
    ```

    ![1612710372159](E:\SoftwareNote\面试准备\多线程\img\jps命令原理.png)





​	- jstack找到死锁查看: jstack 15912 (jstack详解： https://www.jianshu.com/p/8d5782bc596e)

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

  

## 12. LockSupport

- 介绍：

  LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。

  LockSupport是一个线程阻塞工具类，所有的方法都是静态方法，可以让线程再任意位置阻塞，阻塞之后也有对应的唤醒方法。归根结底，LockSupport底层用的是UnSafe的native方法。

  **与synchronized和Lock不一样的是，LockSupport很灵活，可以用在任意位置。**

  synchronized和Lock的阻塞/唤醒都是不能单独使用的：

  - wait()和notify()需要在synchronized代码块中才能使用
  - condition.await()和condition.signal()需要在lock unlock中使用

- 原理

  LockSupport和每个使用它的线程都有一个许可证(premit)关联。premit相当于1，0的开关，默认是0.调用一个unpark就加1变成1。再调用一次unpark(t)就会消耗该线程的premit，将1变成0，park不进行阻塞，立即返回。

  当线程的premit为0时，调用park()，由于消耗不到premit，因此被阻塞。

  可以提前调用unpark(t)给线程提前增加premit，这样后续调用park()就会直接通过。

  premit最多只有1，多次调用还是1.

- API

  - park() 阻塞线程
  - unpart(Thread t1) 解除t1线程阻塞

- demo

```java
Thread t1 = new Thread(() -> {
    try {
        Thread.sleep(590); // 可以先解除锁，再执行锁
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(Thread.currentThread().getName() + "开启，阻塞中...");
    // 阻塞线程
    LockSupport.park();
    System.out.println(Thread.currentThread().getName() + "运行中结束。");
}, "线程1");
Thread t2 = new Thread(() -> {
    // 阻塞线程
    LockSupport.unpark(t1);
    System.out.println(Thread.currentThread().getName() + "解除【" + t1.getName() + "】 阻塞");
}, "线程2");

t1.start();
t2.start();
```

## 13. AQS ： AbstractQueuedSynchronizer抽象队列同步器

- AQS是各JUC工具的底层基石：ReentrantLock/ReentrantReadWriteLock/CountDownLatch/Semaphore/CuclicBarrier

![1615127961055](E:\SoftwareNote\面试准备\多线程\img\AQS是各JUC工具的底层基石.png)

- 多线程，如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要是用CLH队列的变形实现的。将阻塞的线程加入到队列中等待分配，该队列将各线程封装成队列的节点Node，通过CAS/自旋/LockSupport.park()的方法，维护一个总的**state变量**的状态，从而控制并发达到同步的效果。

![1615128099692](E:\SoftwareNote\面试准备\多线程\img\AQS内部组成以及原理说明.png)

- AQS同步队列的基本结构

![1615128845766](E:\SoftwareNote\面试准备\多线程\img\AQS同步队列的基本结构.png)

- AQS类属性

  AQS使用一个volatile的int类型成员变量state来便是同步状态，通过内置的FIFO队列(CLH的变形)来完成资源获取的排队工作，将每个需要抢占资源的线程封装成Node节点，Node<Thread>，通过CAS完成对state状态修改

![1615128192508](E:\SoftwareNote\面试准备\多线程\img\AQS类属性.png)



- AQS内部体系架构

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    
}
abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer {
    static final class Node {
        static final Node SHARED = new Node();
        static final Node EXCLUSIVE = null;
        volatile int waitStatus;
        volatile Node prev;
        volatile Node next;
        volatile Thread thread;
        Node nextWaiter;
        // waitStatus的状态位
        static final int CANCELLED =  1;
        static final int SIGNAL    = -1;
        static final int CONDITION = -2;
        static final int PROPAGATE = -3;
    }
    private transient volatile Node head;
    private transient volatile Node tail;
    private volatile int state;
    // CAS relation 
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long stateOffset;
    private static final long headOffset;
    private static final long tailOffset;
    private static final long waitStatusOffset;
    private static final long nextOffset;
}
abstract class AbstractOwnableSynchronizer {
	private transient Thread exclusiveOwnerThread;  // 当前处理(获取锁)的线程
}
```

![1615128520435](E:\SoftwareNote\面试准备\多线程\img\AQS内部体系架构.png)

- AQS各属性含义

![1615128793404](E:\SoftwareNote\面试准备\多线程\img\AQS各属性含义.png)





- AQS-lock主要API![1615217173791](E:\SoftwareNote\面试准备\多线程\img\AQS-lock主要API.png)

### 13.1 ReentrantLock.lock底层分析

#### 13.1.1 AQS内部体系架构 

  ```java
  //  Sync类继承的是AQS(AbstractQueuedSynchronizer)
  abstract static class Sync extends AbstractQueuedSynchronizer {
      
  }
  abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer {
      static final class Node {
          static final Node SHARED = new Node();
          static final Node EXCLUSIVE = null;
          volatile int waitStatus;
          volatile Node prev;
          volatile Node next;
          volatile Thread thread;
          Node nextWaiter;
          // waitStatus的状态位
          static final int CANCELLED =  1;
          static final int SIGNAL    = -1;
          static final int CONDITION = -2;
          static final int PROPAGATE = -3;
      }
      private transient volatile Node head;
      private transient volatile Node tail;
      private volatile int state; // 所有线程共享
      // CAS relation 
      private static final Unsafe unsafe = Unsafe.getUnsafe();
      private static final long stateOffset;
      private static final long headOffset;
      private static final long tailOffset;
      private static final long waitStatusOffset;
      private static final long nextOffset;
  }
  abstract class AbstractOwnableSynchronizer {
  	private transient Thread exclusiveOwnerThread;  // 当前处理(获取锁)的线程
  }
  ```

  #### 13.1.2 公平锁与非公平锁

```java
// 1. ReentrantLock.lock底层是调用的内部类Sync
class ReentrantLock {
    private final Sync sync;
    public ReentrantLock() {
       sync = new NonfairSync(); // 默认非公平锁
    }
    public ReentrantLock(boolean fair) {
       sync = fair ? new FairSync() : new NonfairSync();
    }
    // ReentrantLock.lock()底层是Sync.lock()
    public void lock() {
        sync.lock();
	}
}


// 2. ReentrantLock的属性Sync有两个实现，FairSync和NonfairSync
static final class FairSync extends Sync {
	final void lock() {
        acquire(1);
     }
}
static final class NonfairSync extends Sync {
    final void lock() {
        if (compareAndSetState(0, 1)) // 进来先尝试改变状态位0->1，修改成功则抢到锁
            setExclusiveOwnerThread(Thread.currentThread()); // 设置为当前持有锁线程，sync.exclusiveOwnerThread即AbstractOwnableSynchronizer.exclusiveOwnerThread即AQS.exclusiveOwnerThread
        else
            // 没抢到锁的，执行acquire(1)
            acquire(1);
    }
}

```

#### 13.1.3 acquire(1) 获取

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg)  // 尝试获取锁，获取!false则不阻塞自己
        && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt(); // true-阻塞自己
}

```

##### 13.1.3.1 tryAcquire（1）尝试获取锁 （tryAcquire中有ReentrantLock是可重入锁的证据） 

````java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException(); // 子类Sync必须实现，否则抛出异常
}

static final class FairSync extends Sync {
    // 公平锁的实现
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() && // 唯一区别 !hasQueuedPredecessors()
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // ReentrantLock是可重入锁的证据：c!=0，但是该线程为拥有锁线程，还能再次拿到锁
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
    // 返回false为公平锁
    public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        // 反着看
        // !((头=尾[意味着没有等待线程,就本线程一个，也就肯定公平]) or (头.next为当前线程[头为占位线程，头.next为队列第一个线程，第一个线程为本线程，公平！]))
        // return !((h ==t) || (h.next != null && h.next.thread = currThread)
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
}

static final class NonfairSync extends Sync {
    // 非公平锁的实现
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        // 线程被占着，state！=0
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}

````

##### 13.1.3.2 addWaiter(Node.EXCLUSIVE=null) 添加线程node(currThread)到等待队列的tail 

```java
// addWaiter(Node.EXCLUSIVE=null)，将node(currThread)放置在等待队列的tail
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    //  tail不为空，意味着等待队列有线程，node直接接在后面tail
    if (pred != null) {
        // node前节点为原tail节点
        node.prev = pred;
        // 将node置为sync的新tail
        if (compareAndSetTail(pred, node)) {
            // 原tail的后节点指向node，配合上两步，此时，node以及进入queue的tail
            pred.next = node;
            return node;
        }
    }
    //  tail为空，说明是第一个阻塞线程进入，执行enq方法，初始化Head节点，以及添加线程node(currThread)到等待队列的tail
    enq(node);
    // 最终返回的node(currThread)，一定是在等待队列的tail
    return node;
}
private Node enq(final Node node) {
    // this为sync
    for (;;) {
        Node t = this.tail;
        // tail的空说明队列还没创建，因为下面tail=head不会为空
        if (t == null) { // Must initialize
            // head==null.if  setHead = new Node() 设置一个占位(傀儡/哨兵节点)对象为头节点
            if (compareAndSetHead(new Node()))
                // 尾=头，此时头尾的pre/next都为null
                this.tail = this.head;
        } 
        // 这是个for死循环，说明上面已经执行，才会走else
        else {
            // node前节点为原tail节点
            node.prev = t;
            // 设置尾节点尾node
            if (compareAndSetTail(t, node)) {
                // 原tail的后节点指向node，配合上两步，此时，node以及进入queue的tail
                t.next = node;
                return t;
            }
        }
    }
}
```

##### 13.1.3.3 acquireQueued(node(currThread), 1)  

```java
// acquireQueued(node[currThread], 1)
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor(); //node.prev
            // p==head即node.prev=head说明是等待队列唯一线程
            // 当其他线程调用unpark，parkAndCheckInterrupt()结束阻塞，tryAcquire(arg)抢锁
            if (p == head && tryAcquire(arg)) {
                // 抢到锁，则node需要出队列
                // head = node;
                // node.thread = null;
                // node.prev = null; 
                setHead(node); // 当前node为新傀儡节点，原傀儡节点引用取消，等待GC
                p.next = null; // help GC
                failed = false;
                return interrupted; // false=本线程无需阻塞
            }
            // 如果shouldParkAfterFailedAcquire(p, node)则parkAndCheckInterrupt()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt()) // 线程阻塞在if这行
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

final Node predecessor() throws NullPointerException {
    Node p = prev;
    if (p == null)
        throw new NullPointerException();
    else
        return p;
}
private void setHead(Node node) {
    head = node;
    node.thread = null;
    node.prev = null;
}
// 该线程尝试获得锁失败，是否应该被park
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus; // 默认是0
    // 外层自旋，第二次进来 -1=-1，return true，该线程应该被park
    if (ws == Node.SIGNAL)
        /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
        return true;
    if (ws > 0) {
        /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
        // 第一次为0，则被制成-1
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}

private final boolean parkAndCheckInterrupt() {
    // 阻塞当前线程，线程阻塞在这一行，直至其他线程调用LockSuport.unPark(this)
    LockSupport.park(this);
    return Thread.interrupted(); // currentThread().isInterrupted(true);当前线程是否被阻塞
}
```

### 13.2 ReentrantLock.unlock底层分析

```java
class ReentrantLock {
    // 执行lock.unlock的肯定是当前持有锁的线程，其他线程都被lock阻塞
    public void unlock() {
        sync.release(1);
    }
}
abstract class AbstractQueuedSynchronizer {
    // release(1)
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            // 头!=null 头.ws!=0,所有等待队列有线程，否则head.ws会被置成-1，compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    // unparkSuccessor（head)
    private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        // -1
        int ws = node.waitStatus;
        if (ws < 0)
            // head.ws又被置成0
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        // s为第一个等待节点
        Node s = node.next;
        // 等待节点s为异常状态，则倒叙遍历队列，直至找到一个正常状态的node，以其阻塞状态
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            // 取消s的阻塞状态
            LockSupport.unpark(s.thread);
    }
}

abstract static class Sync extends AbstractQueuedSynchronizer{
    // tryRelease(1)
    protected final boolean tryRelease(int releases) {
        // c= 1-1 = 0
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            // currThread=null
            setExclusiveOwnerThread(null);
        }
        // state = 0, 其他线程可以抢锁了
        setState(c);
        return free;
    }
}
```

- AQS多线程出入队列示意图

![1615218875851](E:\SoftwareNote\面试准备\多线程\img\AQS多线程出入队列示意图.png)