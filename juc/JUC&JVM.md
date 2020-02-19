> author  :  huohuliSS

# JUC与JVM

[TOC]

1. synchronized 锁定的是当前对象

   当修饰普通方法时锁定的是**this**当前实例对象，方法为static时锁定的是**class**全局锁。

2. 同步和非同步方法是否可以同时调用？   是。 普通方法和同步锁无关

3. 对业务写方法加锁，读方法不加锁，容易产生脏读问题

4. 一个同步的方法可以调用另一个同步方法，一个线程已经拥有某个对象的锁，再次申请的时候仍然会得到该对象的锁，也就是说synchronize获得的锁是可重入的

5. 程序在执行过程中，如果出现了异常，默认情况下会自动释放锁

6. volatile 关键字，使一个共享变量在多个线程之间是可见性的。jvm会将变量的值存在于堆内存（主内存）中，使用volatile会强制所有的线程都去堆内存中读取

7. 多线程的防止虚假唤醒，判断使用**while**

   > while在单线程是用来循环的，if用来做判断的，多线程环境下，只用if判断可能会造成虚假唤醒，而while在多线程环境下如果**当前线程被唤醒时，会再次判断while里条件是否符合**

   当前的线程必须拥有该对象的显示器。  该线程释放此监视器的所有权，并等待另一个线程通知等待该对象监视器的线程通过调用`notify`方法或`notifyAll`方法`notifyAll`  。 然后线程等待，直到它可以重新获得监视器的所有权并恢复执行。 

   像在一个参数版本中，中断和虚假唤醒是可能的，并且该方法应该始终在循环中使用： 

   ```java
     synchronized (obj) {
            while (<condition does not hold>)
                obj.wait();
            ... // Perform action appropriate to condition
        } 
   ```

...............................

## JUC(java.util.concurrent)

1.1 进程/线程

1.2 并发/并行

### 三个包

java.util.concurrent

java.util.concurrent.atomic

java.util.concurrent.locks

### 线程安全集合

```java
List list = new ArrayList();   // 线程不安全
Map map = new HashMap();  // 线程不安全
```

List解决（set同理）

```java
1. List list = Collections.synchronizedList(new ArrayList<>());
2. List list = new CopyOnWriteArrayList(); // 运用ReentrantLock重入锁
```

map解决

```java
Map map = new ConcurrentHashMap<>();
```

### Synchronized底层实现

https://www.cnblogs.com/kubidemanong/p/9520071.html（太牛逼了）

synchronized实现同步主要是：synchronized方法和synchronized代码块，但实际上都是给**对象加锁**，

#### **java对象布局**

> java对象在内存中的存储结构主要有三个部分：
>
> - 对象头
> - 示例数据（类的属性数据）
> - 填充数据（规定对象的起始地址必须为8的整数倍）

对象头中其简单的结构如下：

|   长度   |      内容      |             说明             |
| :------: | :------------: | :--------------------------: |
| 30/64bit |   Mark Word    | 存有hashcode、gc年龄、锁信息 |
| 30/64bit | Class Metadata |    指向对象类型数据的指针    |
| 30/64bit |  Array Length  | 数组的长度(仅当对象为数组时) |

#### 对象的状态

> - 无状态（当对象刚new出来的时候）
> - 偏向锁
> - 轻量锁
> - 重量锁
> - gc标记

在运行期间Mark Word里存储的数据会随着锁状态的变化而变化

![](https://images2018.cnblogs.com/blog/1093668/201807/1093668-20180710192342955-285887224.png)

当对象状态为偏向锁（biasable）时，mark word存储的是**偏向的线程ID**；

当状态为轻量级锁（lightweight locked）时，mark word存储的是指向**线程栈中Lock Record的指针**；

当状态为重量级锁（inflated）时，为指向**堆中的monitor对象的指针**。

**偏向锁**

> 偏向锁是jdk1.6引入的一项锁优化，其中的“偏”是偏心的偏。它的意思就是说，这个锁会偏向于第一个获得它的线程，在接下来的执行过程中，假如该锁没有被其他线程所获取，没有其他线程来竞争该锁，那么持有偏向锁的线程将永远不需要进行同步操作。
> 也就是说:
> 在此线程之后的执行过程中，如果再次进入或者退出同一段同步块代码，并不再需要去进行**加锁**或者**解锁**操作，而是会做以下的步骤：
>
> 1. Load-and-test，也就是简单判断一下当前线程id是否与Markword当中的线程id是否一致.
> 2. 如果一致，则说明此线程已经成功获得了锁，继续执行下面的代码.
> 3. 如果不一致，则要检查一下对象是否还是可偏向，即“是否偏向锁”标志位的值。
> 4. 如果还未偏向，则利用CAS操作来竞争锁，也即是第一次获取锁时的操作。
>
> 如果此对象已经偏向了，并且不是偏向自己，则说明存在了**竞争**。此时可能就要根据另外线程的情况，可能是重新偏向，也有可能是做偏向撤销，但大部分情况下就是升级成**轻量级锁**了。
> 可以看出，偏向锁是针对于一个线程而言的，线程获得锁之后就不会再有解锁等操作了，这样可以省略很多开销。假如有两个线程来竞争该锁话，那么偏向锁就失效了，进而升级成轻量级锁了。
> 为什么要这样做呢？因为经验表明，其实大部分情况下，都会是同一个线程进入同一块同步代码块的。这也是为什么会有偏向锁出现的原因。
> 在Jdk1.6中，**偏向锁的开关是默认开启的，适用于只有一个线程访问同步块的场景**。

**轻量级锁**

锁撤销升级为轻量级锁之后，那么对象的Markword也会进行相应的的变化。下面先简单描述下锁撤销之后，升级为轻量级锁的过程：

1. 线程在自己的栈桢中创建锁记录 LockRecord的空间。
2. 将锁对象的对象头中的MarkWord复制到线程的刚刚创建的锁记录中。
3. 将锁记录中的Owner指针指向锁对象。
4. 将锁对象的对象头的MarkWord替换为指向锁记录的指针。

对应的图描述如下(图来自周志明深入java虚拟机)
![图片1](https://user-gold-cdn.xitu.io/2018/8/22/165614893a59be34?w=721&h=344&f=png&s=42033)
![图片2](https://user-gold-cdn.xitu.io/2018/8/22/165614960abf2f0e?w=549&h=343&f=png&s=35522)

**重量级锁**

轻量级锁膨胀之后，就升级为重量级锁了。重量级锁是依赖**对象内部的monitor锁**来实现的，而monitor又依赖**操作系统**的**MutexLock**(互斥锁)来实现的，所以重量级锁也被成为**互斥锁**(阻塞同步、悲观锁)。

补充：**为什么说重量级锁开销大呢？**

> 系统检查到锁是重量级锁之后，会把要等待锁的线程进行**阻塞**（若不阻塞，需要自旋，则会占用大量的cpu资源），被阻塞的线程不会消耗cpu，但是阻塞或者唤醒一个线程时，jvm实现不了，需要借助操作系统来帮忙，这就需要从**用户态**转换到**内核态**，而**转换状态是需要消耗很多时间的**，有可能比用户执行代码的时间还要长。

**synchronized关键字并非一开始就该对象加上重量级锁，也是从偏向锁，轻量级锁，再到重量级锁的过程。**



### Lock

在jdk1.5以后，增加了juc并发包且提供了Lock接口用来实现锁的功能，它除了提供了与synchroinzed关键字类似的同步功能，还提供了比synchronized更灵活api实现。

#### Lock实现类ReentrantLock

> 可重入锁。如果当前线程t1通过调用lock方法获取了锁之后，再次调用lock，是不会再阻塞去获取锁的，直接增加重入次数就行了。与每次lock对应的是unlock，unlock会减少重入次数，重入次数减为0才会释放锁。

代码实例：

```java
Lock lock = new ReentrantLock();

    public void sale() {
        lock.lock();
        try {
            // ....
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
```

#### Condition

> Condition 实际上是 java.util.concurrent.locks 中的一个接口。
>
> Condition 将 Object 监视器方法（wait、notify 和 notifyAll）分解成截然不同的对象，以便通过将这些对象与任意 Lock 实现组合使用，为每个对象提供多个等待 set（wait-set）。其中，Lock 替代了 synchronized 方法和语句的使用，Condition 替代了 Object 监视器方法的使用。

#### 读写锁ReentrantReadWriteLock

```java
class MyData {
    private volatile Map<String, Object> map = new HashMap<>();
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void put(String key, Object value) {
        //put加上写锁
        readWriteLock.writeLock().lock();
        try {

            System.out.println(Thread.currentThread().getName() + "\t---写入数据" + key);
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "\t---写入完成");
        }finally {
            readWriteLock.writeLock().unlock();
        }
    }

    public void get(String key) {
        // get加读锁 
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t读取数据");
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            map.get(key);
            System.out.println(Thread.currentThread().getName() + "\t 读取完成");
        }finally {
            readWriteLock.readLock().unlock();
        }
    }
}
```



### 阻塞队列

**BlockingQueue**是Jdk1.5之后，随着JUC引入的一个接口，常用于**生产者和消费者**的场景。

BlockingQueue继承自Queue接口，提供了一些阻塞方法，主要作用如下：

> - 当线程向队列中插入元素时，如果队列已满，则阻塞线程，直到队列有空闲位置（非满）；
> - 当线程从队列中取元素（删除队列元素）时，如果队列未空，则阻塞线程，直到队列有元素；

#### BlockingQueue核心方法

| 操作类型 | 抛出异常  | 返回特殊值 | 阻塞线程 |        超时        |
| :------: | :-------: | :--------: | :------: | :----------------: |
|   插入   |  add(e)   |  offer(e)  |  put(e)  | offer(e,time,unit) |
|   删除   | remove(e) |   poll()   |  take()  |  poll(time,unit)   |
|   读取   | element() |   peek()   |    /     |         /          |

四套方法对应特点： 

> 1. ThrowsException：如果操作不能马上进行，则抛出异常
> 2. 返回特殊值 ： 如果操作不能马上进行，将会返回一个特殊的值，一般是true或者false
> 3. 阻塞线程： 如果操作不能马上进行，操作就被阻塞
> 4. TimeOut超时： 如果操作不能马上进行，操作就会被阻塞指定的时间，如果指定时间没执行，则就会返回一个特殊值true或false

注意： 不能向BlockingQueue中插入为null，否则会报空指针异常。

#### BlockingQueue的实现类

实现类有7个：**ArrayBlockingQueue**、**LinkedBlockingQueue**、**SynchronizeQueue**、**PriorityBlockingQueue**、DelayQueue、LinkedTransferQueue、LinkedBlockingDeque

1. **ArrayBlockingQueue**

   > ArrayBlockingQueue是一个用数组实现的有界阻塞队列，此队列按照先进先出（FIFO）的原则对元素进行排序，最新插入的对象是尾部，最新移出的是头部，支持公平锁和非公平锁。
   >
   
2. DelayQueue（延迟队列）

   >  DelayQueue是一个实现PriorityBlockingQueue延迟获取的无界队列，在获取元素时，可以指定多久才能从队列中获取当前元素，只有延迟期满后才能从队列中获取元素。加入其中的元素必须实现Delayed接口。
>
   >  **使用场景**：1、缓存系统的设计：用delayQueue保存缓存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从DelayQueue中获取元素时，说明缓存有效期到了。2、定时任务调度：使用DelayQueue保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到任务就开始执行。

3. **LinkedBlockingQueue**

   > LinkedBlockingQueue：一个由链表结构组成的有界队列，如果我们初始值指定一个大小，它就是有边界的，如果不指定大小，就是无边界的，说是无边界，其实是采用了默认大小`Integer.MAX_VALUE`的容量，
   >
   > 和ArrayBlockingQueue一样，也是先进先出的方式存储数据，最新插入的对象是尾部。

4. SynchronousQueue

   > SynchronousQueue队列内部仅允许容纳一个元素。当一个线程插入一个元素后会被阻塞，除非这个元素被另一个线程消费。
   
5. PriorityBlockingQueue

   > 一个支持线程优先级排序的顺序，默认自然进行排序，也可以自定义实现compareTo()方法来指定元素排序规则，不能保证同优先级元素的顺序。

6. 

#### CountDownLatch

> 一个CountDownLatch为一个计数的CountDownLatch用作一个简单的开/关锁存器，或者门：所有线程调用await在门口等待，直到被调用countDown()的线程打开。

#### CyclicBarrir

> 允许一组线程全部等待彼此达到共同屏障点的同步辅助。循环阻塞在涉及固定大小的线程方的程序中很有用，这些线程必须偶尔等待彼此。屏障被称为循环 ，因为它可以在等待的线程被释放之后重新使用

#### Semaphore

> 一个计数信号量。 在概念上，信号量维持一组许可证。
>  *    如果有必要，每个acquire()都会阻塞，直到许可证可用，然后才能使用它。
>  *    每个release()添加许可证，潜在地释放阻塞获取方。 但是，没有使用实际的许可证对象; Semaphore只保留可用数量的计数，并相应地执行。
>  *      主要作用 ： 1.多线程控制并发同时访问资源的数量
>  *                2.用于多个共享资源的互斥使用

### 线程池

为什么使用线程池？

> - 减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务
>
> - 可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为因为消耗过多的内存，而把服务 
>
>   器累趴下

线程池的最上层接口是**Executor**（相当于集合中的Collection），真正的线程池接口是 **ExecutorService**(相当于集合中的list、set)，子接口不仅有父类接口的方法，而且还封装了更多的方法更加强大，这里主要以**ExecutorService**详解

#### 四种常见的线程池介绍

**ExecutorService**介绍

> ExecutorService是Java提供的用于管理线程池的类。该类的两个作用：控制线程数量和重用线程



具体的4种常用的线程池实现如下：（返回值都是ExecutorService）

- Executors.newFixedThreadPool(int n)

  > 创建一个可重用固定个数的线程池，以共享的无界队列方式来运行这些线程。
  >
  > 线程池在执行 execute 方法来执行 Thread 类 中的 run 方法。不管 execute 执行几次，线程池始终都会使用 2 个线程来处理。不会再去创建出其他线程来处理 run 方法执行。这就是固定大小线程池。

  ```java
  public class MyThreadPoolDemo01 {
  
      public static void main(String[] args) {
          ExecutorService executorService = Executors.newFixedThreadPool(3);
  
          try {
  //            模拟有10个人排队打饭，目前线程池只有固定的3个窗口提供服务
              for (int i = 0; i < 10; i++) {
  //                将线程放进线程池中
                  executorService.execute(() -> {
                      System.out.println(Thread.currentThread().getName() + "\t正在办理业务");
  //                    线程工作中
                      try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
                      System.out.println(Thread.currentThread().getName() + "\t办理任务结束离开");
                  });
              }
          } finally {
  //            关闭线程池
              executorService.shutdown();
          }
      }
  }
  ```

- Executors.newCacheThreadPool()

  > 可变线程池，连接池会根据执行的情况，在程序运行时创 建多个线程来处理，这里就是可变连接池的特点。是一个不限线程数上限的线程池，任何提交的任务都将立即执行

- Executors.newSingleThreadExecutor()

  > 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
  >
  > ```java
  > ExecutorService executorService = Executors.newSingleThreadExecutor();
  > ```

- Executors.newScheduledThreadPool(int n)

  > 创建一个定长线程池，支持定时及周期性任务执行

  ```java
  //创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。
  ScheduledExecutorService pool = Executors.newScheduledThreadPool(2);
  ....
  //将线程放入池中进行执行
  pool.execute(t1);
  pool.execute(t2);
  pool.execute(t3);
  //使用定时执行风格的方法
  pool.schedule(t4, 10, TimeUnit.MILLISECONDS); //t4 和 t5 在 10 秒后执行
  pool.schedule(t5, 10, TimeUnit.MILLISECONDS);
  ```

#### 线程池的底层执行者ThreadPoolExecutor

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

// 可变线程池，默认初始大小为Interger.MAX_VALUE
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

> 由底层代码可见，**ThreadPoolExecutor底层是由阻塞队列BlockingQueue实现的**

ThreadPoolExecutor提供了四个构造参数,其中共有7个参数

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
     this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
}
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

| 序号 |      名称       |           类型           |       含义       |
| :--: | :-------------: | :----------------------: | :--------------: |
|  1   |  corePoolSize   |           int            |  核心线程池大小  |
|  2   | maximumPoolSize |           int            |  最大线程池大小  |
|  3   |  keepAliveTIme  |           long           | 线程最大空闲时间 |
|  4   |      unit       |         TimeUnit         |     时间单位     |
|  5   |    workQueue    | BlockingQueue<Ruunable>  |   线程等待队列   |
|  6   |  threadFactory  |      ThreadFactory       |   线程创建工厂   |
|  7   |     handler     | RejectedExecutionHandler |     拒绝策略     |

#### 线程池底层工作流程原理

![](https://img-blog.csdn.net/20180809160432787?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NTIwMjM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

> 1. 在创建了线程池后，开始等待请求。
>
> 2. 当调用execute()方法添加一个请求任务时，线程池会做出如下判断：
>
> ​         2.1 如果正在运行的线程数量小于corePoolSize，那么马上threadFactory创建线程运行这个任务；
>
> ​         2.2 如果正在运行的线程数量大于或等于corePoolSize，那么将这个任务放入队列；
>
> ​         2.3 如果这个时候队列满了且正在运行的线程数量还小于maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；
>
> ​        2.4 如果队列满了且正在运行的线程数量达到了maximumPoolSize，那么线程池会启动**拒绝策略来执行**
>
> 3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。
>
> 4. 当一个线程无事可做超过一定时间keepAliveTIme时，线程会判断：
>
>    ​        4.1 如果当前运行的线程大于corePoolSize，那么这个线程就会被停掉；
>
>    ​        4.2 所以线程池的所有任务完成后，最终都会减少到corePoolSize大小。

#### 线程池如何设置合理参数

**线程池的拒绝策略参数**

JDK自带的拒绝策略：

> - AbortPolicy（默认）：直接抛出RejectedExecutionException异常组织系统正常运行
> - CallerRunPolicy： 即不会抛弃任务，也不会抛出异常，而是将某些人物回调给调用者，从而降低新任务流量
> - DiscardOldestPolicy：抛弃队列中等待最久的任务，然后把当前任务加入到队列中尝试再次提交当前任务
> - DiscardPolicy：直接丢弃任务，不予任何处理也不会抛出异常。如果允许任务丢失，这是最好的解决方案

我们在实际工作中，由Executors创建的单一/固定/可变三种创建线程池均不用，而是通过ThreadPoolExecutor的方式创建，主要以下原因:

> 1. FixedThreadPool和SingleThreadPool：允许的**请求阻塞队列**长度为Interger.MAX_VALUE，可能会堆积大量请求，从而导致OOM。
> 2. CachedThreadPool和ScheduledThreadPool：允许的**线程数量**为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。

**最大线程数参数配置**

maximumPoolSize

> CPU密集型 ： CPU核数 + 1
>
> I/O密集型 ： CPU核数 / (1 - 阻塞系数)

### Java8之流式计算

#### java函数式接口概念

> **有且仅有一个抽象方法的接口。**
>
> 函数式接口，即适用于函数式编程场景的接口。而Java中的函数式编程体现就是Lambda，所以函数式接口就是可以适用于Lambda使用的接口。只有确保接口中有且仅有一个抽象方法，Java中的Lambda才能顺利地进行推导。

#### java四大函数式编程

|          函数式接口           | 参数类型 | 返回类型 |                             用途                             |
| :---------------------------: | :------: | :------: | :----------------------------------------------------------: |
|   Consumer< T>  消费型接口    |    T     |   void   |       接口方法 void accept(T t)：参数类型是T，无返回值       |
|  Supplier< T>    供给型接口   |    无    |    T     |         接口方法 T get()：参数类型是T，返回T类型参数         |
|  Function<T,R>   函数型接口   |    T     |    R     |      接口方法R apply(T)：对类型T参数操作，返回R类型参数      |
| Predicate< T>      段言型接口 |    T     | boolean  | 接口方法 boolean test（T t）：对类型T进行条件筛选操作，返回boolean |

```java
list.stream().filter(user -> {return user.getId() % 2 == 0;})
    //  过滤
                .filter(user -> {return user.getAge() > 24;})
    // 映射
                .map(user ->  user.getName().toUpperCase())
    // 排序
                .sorted((u1,u2) -> {return u2.compareTo(u1);})
                .limit(1)
//                .count()    // 计数
    // 遍历
                .forEach(System.out::println);
```

### 分支合并框架（线程池）

ForkJoinPool、ForkJoinTask、RecursiveTask

> 分支/合并框架 是以**递归**的方式将可以并行的任务切割成更小的任务，然后将每个任务的结果进行合并来生成最终的结果。它是ExecutorService接口的实现，子任务在ForkJoinPool中执行。
>
> 创建了 ForkJoinPool实例，就可以调用 ForkJoinPool的 submit(ForkJoinTask task) 或者invoke(ForkJoinTask task) 方法来执行指定的任务。 其中 ForkJoinTask代表一个可以并行、合并的任务。ForkJoinTask是一个抽象类，它有两个抽象子类：**RecursiveAction**（没有返回值） 和 **RecursiveTask**（有返回值）。

```java
public class ForkJoinPoolDemo02 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyTask myTask = new MyTask(0, 100);

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Integer> forkJoinTask = forkJoinPool.submit(myTask);

        System.out.println(forkJoinTask.get());
        forkJoinPool.shutdown();
    }
}

class MyTask extends RecursiveTask<Integer> {
    private static final Integer ADJUST_VALUE = 10;
    private int begin;
    private int end;
    private int result;

    public MyTask(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        if ((end - begin) <= ADJUST_VALUE) {
            for (int i = begin; i <= end; i++) {
                result = result + i;
            }
        } else {
            int middle = (end + begin) / 2;
            MyTask myTask1 = new MyTask(begin, middle);
            MyTask myTask2 = new MyTask(middle + 1, end);
//            分支计算
            myTask1.fork();
            myTask2.fork();
//            合并结果
            result = myTask1.join() + myTask2.join();
        }
        return result;
    }
}
```



## JVM（Java Virtual Machine）

### jvm体系结构概览

![](https://upload-images.jianshu.io/upload_images/6423761-0e92ccf1175a7962.png?imageMogr2/auto-orient/strip|imageView2/2/w/859/format/webp)

##### 类加载器ClassLoader

> 负责加载class文件，class文件在文件开头有特定的文件标识，将class文件字节码内容加载到内存中。并将这些内容转化成方法区中的运行时数据结构并且ClassLoader只负责class文件的加载，至于它是否可以运行，则由执行引擎(Execution Engine)决定。

#### JVM各个组成部分

##### 本地方法栈

> 线程私有，作用是支撑Native方法的调用、执行和退出

##### PC寄存器(程序计数器)

> （看成一个指针）**每个线程启动的时候，都会创建一个PC（Program Counter，程序计数器）寄存器。**PC寄存器里保存有当前正在执行的JVM指令的地址。 每一个线程都有它自己的PC寄存器，也是该线程启动时创建的。保存下*一条将要执行的指令地址的寄存器是* ：**PC寄存器**。PC寄存器的内容总是指向下一条将被执行指令的地址，这里的地址可以是一个本地指针，也可以是在方法区中相对应于该方法起始指令的偏移量。同样内存也特别小，几乎可以忽略不计，不存在垃圾回收机制

##### 方法区

> method（方法区）又叫**静态区**，**存放类的结构信息**，保存在着被加载过的每一个类的信息；这些信息由类加载器在加载类的时候，从类的源文件中抽取出来；**static变量信息**也保存在方法区中
>
> 1.又叫静态区，跟堆一样，被所有的线程共享。
>
> 2.方法区中存放的都是在整个程序中永远唯一的元素。这也是方法区被所有的线程共享的原因

##### 虚拟机栈

> Java栈的区域很小，只有1M，不存在垃圾回收，特点是存取速度很快，所以在stack中存放的都是快速执行的任务，**基本数据类型的数据+对象的引用（reference）+实例方法**。
>
> Java 虚拟机栈描述的是 Java 方法执行的内存模型，用于**存储栈帧**。线程启动时会创建虚拟机栈，每个方法在执行时会在虚拟机栈中创建一个栈帧，用于存储局部变量表、操作数栈、动态连接、方法返回地址、附加信息等信息。如果栈空间被占满时，会产生栈溢出SOE异常“StackOverflowError”
>
> **栈帧**
>
> 栈帧存在于 Java 虚拟机栈中，是 **Java 虚拟机栈中的单位元素**，每个线程中调用同一个方法或者不同的方法，都会创建不同的栈帧，每个栈帧主要包含四个部分：`局部变量表`、`操作数栈`、`动态链接`、`返回地址`，**每个方法从调用到执行完成的过程，就对应着一个栈帧在虚拟机栈中的入栈（压栈）到出栈（弹栈）的过程。**
>
> - 局部变量表：是一组变量值存储空间，用于存放方法参数和方法参数和方法内部定义的**局部变量**，局部变量表存放了编译期可知的各种基本数据类型
>
> - 操作数栈：存放调用过程中的计算结果的临时存放区域。
>
>   ​        虚拟机把操作数栈作为它的工作区——大多数指令都要从这里弹出数据，执行运算，然后把结果压回操作数栈。比如，iadd指令就要从操作数栈中弹出两个整数，执行加法运算，其结果又压回到操作数栈中，看看下面的示例，它演示了虚拟机是如何把两个int类型的局部变量相加，再把结果保存到第三个局部变量的。
>
>   ```java
>   begin  
>   iload_0    // push the int in local variable 0 onto the stack  
>   iload_1    // push the int in local variable 1 onto the stack  
>   iadd       // pop two ints, add them, push result  
>   istore_2   // pop int, store into local variable 2  
>   end  
>   ```
>
>   ​        在这个字节码序列里，前两个指令iload_0和iload_1将存储在局部变量中索引为0和1的整数压入操作数栈中，其后iadd指令从操作数栈中弹出那两个整数相加，再将结果压入操作数栈。第四条指令istore_2则从操作数栈中弹出结果，并把它存储到局部变量区索引为2的位置。下图详细表述了这个过程中局部变量和操作数栈的状态变化，图中没有使用的局部变量区和操作数栈区域以空白表示。
>
>   ![](http://dl.iteye.com/upload/picture/pic/116487/4032d1b9-43c5-3dc6-95c8-38b9fbb149b0.jpg)
>
> - 动态链接
>
>   ​       由于栈帧是用于方法调用和方法执行的数据结构。所以每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用。在class文件的常量池中有大量的符号引号。这些符号引用有一部分在类加载阶段或者第一次使用的时候就直接转化为直接引用（静态解析）。有的则在每一次运行期间转化为直接引用（动态连接）。
>
> - 返回地址
>
>   ​       当一个方法开始执行后，只有两种方式可以退出这个方法：
>
>   ​       **方法返回指令**：执行引擎遇到一个方法返回的字节码指令，这时候有可能会有返回值传递给上层的方法调用者，这种退出方式成为正常完成出口。
>
>   ​        **异常退出**： 在方法执行过程中遇到异常，并且没有处理这个异常，就会导致方法退出。
>
>   ​        无论采用哪种退出方式，在方法退出之后，都需要返回到方法被调用的位置，程序才能继续执行，方法返回时可能需要在栈帧中保存一些信息。
>

##### 堆（Heap）

> 类的对象放在heap（堆）中，所有的类对象都是通过new方法创建，创建后，在stack（栈）会创建类对象的引用（内存地址）。java堆是JVM管理最大的一块内存空间。
>
> 一个JVM实例只存在一个堆内存，堆内存的大小是可以调节的，类加载器读取了类文件之后，需要把类、方法、常变量放到堆内存中，保存所有引用类型的真实信息，以方便执行器执行，堆内存**逻辑**上分为三部分：`新生代`(Young/New Generation Space) 、 `老年代`(Tenure/Old Generation Space)、`永久代`（Permanent Space） ,物理划分堆中只有新生代和老年代。
>
> ![](https://img-blog.csdnimg.cn/20190715175255576.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tqdzg4OA==,size_16,color_FFFFFF,t_70)
>
> 从JDK8开始，Metaspace(元空间)替代了永久代
>
> ![](https://img-blog.csdnimg.cn/20190715173826559.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tqdzg4OA==,size_16,color_FFFFFF,t_70)
>
> 其中新生代 ( Young ) 又被划分为：Eden（伊甸区）、From Survivor（幸存0区）和To Survivor（幸存1区）三个区域，如图所示：（图中永久代/元空间不属于堆）。所有的类都是在伊甸区被new出来的，如果伊甸区的空间用完时，程序还需new对象，JVM的垃圾回收器会对伊甸区进行垃圾回收（Minor GC—轻度回收），将伊甸区的剩余对象移动到幸存0区中，如果幸存0区也满时，再对该去进行垃圾回收机制（GC），然后移动到幸存1区，1区再满时，移动到老年代空间去，养老区再满时，将产生（Full GC—深度回收）,若老年代空间也满时，就会产生OOM异常“OutOfMemoryError”
>
> 如果出现OOM（java.lang.OutOfMemoryError:Java heap space）异常，说明java虚拟机的堆内存不够而溢出，原因有二：
>
> - java虚拟机的堆内存设置不够，可以通过参数-Xms、-Xmx来调整。
> - 代码中创建了大量的对象，并且长时间不能被垃圾收集器收集（存在被引用）

#### 栈、堆、方法区的交互方式

![](https://img-blog.csdn.net/20180619214704117?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3ljZDUwMDc1Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**类的加载过程**

![](https://img-blog.csdn.net/20170430160610299?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

1. **加载**

   > 加载就是将类的class文件读入到内存中，并为之创建一个Class对象

2. 链接

   > 当类被加载之后，系统为之生成一个对应的Class对象，接着就会进入链接阶段，链接阶段负责把类的二进制数据合并到JRE中。类连接又可分为如下3个阶段。
   >
   > 1> **验证** ： 验证阶段用于检验被加载的类是否有正确的内部结构，并和其他类协调一致
   >
   > 2> **准备** ： 类准备阶段负责为类的静态变量分配内存，并设置默认初始值
   >
   > 3> **解析** ： 将类的二进制数中的符号引用替换成直接引用

3. **初始化**

   > 初始化是为类的静态变量赋予正确的初始值，准备阶段和初始化阶段看似有点矛盾，其实是不矛盾的，如果类中有语句：private static int a = 10，它的执行过程是这样的，首先字节码文件被加载到内存后，先进行链接的验证这一步骤，验证通过后准备阶段，给a分配内存，因为变量a是static的，所以此时a等于int类型的默认初始值0，即a=0,然后到解析（后面在说），到初始化这一步骤时，才把a的真正的值10赋给a,此时a=10。

#### 类的加载时机

> - 创建类的实例，也就是new一个对象
> - 访问某个类或接口的静态变量，或者对该静态变量赋值
> - 调用类的静态方法
> - 反射(Class.forName("com.lji.load"))
> - 初始化一个类的子类（会首先初始化子类的父类）
> - JVM启动时标明的启动类，即文件名和类名相同的那个类

#### 类加载机制的层次结构

Java 中的类加载器大致可以分成两类，一类是系统提供的，另外一类则是由 Java 应用开发人员编写的。系统提供的类加载器主要有下面三个：

> - **启动类加载器**：负责将存放在`<JAVA_HOME>\lib`目录中的，或者被`-Xbootclasspath`参数所指定的路径中的，并且是虚拟机识别的类库加载到虚拟机内存中。启动类加载器无法被Java程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给引导类加载器，那直接使用null代替即可。
> - **扩展类加载器**：这个加载器由`sun.misc.Launcher$ExtClassLoader`实现，负责加载`<JAVA_HOME>\lib\ext`目录下的，或者被`java.ext.dirs`系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。
> - **应用程序类加载器**：这个类加载器是由`sun.misc.Launcher$AppClassLoader`实现的。由于这个类加载器是`ClassLoader`中的`getSystemClassLoader`方法的返回值，所以也叫系统类加载器。它负责加载用户类路径上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

- **自定义类加载器**，继承ClassLoader，父类加载器肯定为AppClassLoader。

#### 加载器的双亲委派机制

![](https://img-blog.csdn.net/20180813145521896?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM4MDc1NDI1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式。

**沙箱安全机制**

简单来说：使用的双亲委派机制来保证自己写的代码不会污染java源代码

### Native 本地接口

使用native关键字说明这个方法是原生函数，也就是这个方法是用C/C++语言实现的，并且被编译成了**DLL**，由java去调用。这些函数的实现体在DLL中，JDK的源代码中并不包含，你应该是看不到的。对于不同的平台它们也是不同的。这也是java的底层机制。

JNI（java Native Interface）调用C流程图

![](https://images2015.cnblogs.com/blog/690102/201607/690102-20160725102547356-2054241629.png)

### 对象作为参数时是值传递还是引用传递？

>  是值传递。**Java编程语言只有值传递参数**。
>
> 当一个对象实例作为一个参数被传递到方法中时，**参数的值**就是该对象的引用一个副本。指向同一个对象，**对象的内容**可以在被调用的方法中**改变**，但**对象的引用**(不是引用的副本)是永远不会改变的。

### java中常量池存储位置

> Java6和6之前，常量池是存放在方法区中的。
>
> Java7，将常量池是存放到了堆中，常量池就相当于是在永久代中，所以永久代存放在堆中。
>
> Java8之后，取消了整个永久代区域，取而代之的是元空间。没有再对常量池进行调整。

### 对象的生命周期与GC

![](https://images2017.cnblogs.com/blog/1187916/201711/1187916-20171130145507089-940285672.png)

**堆内存划分**

> - 堆大小 = 新生代 + 老生代，堆大小可以通过参数-Xms、-Xmx指定
> - 新生代分为Eden（伊甸区）和两个Survivor（幸存）区，这两个 Survivor 区域分别被命名为 from 和 to，以示区分。默认的，Edem : from : to = 8 : 1 : 1
> - JVM 每次只会使用 Eden 和其中的一块 Survivor 区域来为对象服务，所以无论什么时候，总是有一块 Survivor 区域是空闲着的。

**堆的垃圾回收机制**

> java堆是GC垃圾回收的主要区域。 GC分为两种： **Minor GC、Full GC**（也叫做Major GC）.

**Minor GC** 

> Minor GC是发生在**新生代**中的垃圾收集动作， 所采用的是**复制算法**。GC一般为堆空间某个区发生了垃圾回收
>
> **Minor GC回收过程** （复制->清空->互换）
>
> 1.复制  将Eden、Survivor复制到SurvivorTo，年领+1
>
> ​     首先，当Eden区满的时候会触发第一次GC，把还活着的对象拷贝到SurvivorFrom区，当Eden区再次触发GC的时候会扫描Eden区和from区域，对这两个区域进行垃圾回收，经过这次回收后还存活的对象，则直接复制到To区域（如果有对象的年龄已经达到了老年的标准，则赋值到老年区），同时把这些对象的年龄+1
>
> 2.清空  Eden、SurvivorFrom
>
> ​     然后，清空Eden和SurvivorFrom中的对象，也即复制之后又交换，谁空谁是To
>
> 3.SurvivorTo和SurvivorFrom互换
>
> ​    最后，SurvivorTo和SurvivorFrom互换，原SurvivorTo成为下一次GC时的SurvivorFrom区，部分对象会在From和To区域中复制来复制去，如此交换15次（默认是15），最终如果还是存活，就存入老年代。

**Full GC**

> Full GC 基本都是整个堆空间及持久（永久）代发生了垃圾回收，所采用的是**标记-清除算法。**
>
> 缺点：效率慢，容易产生内存碎片

### 堆中的永久代(JDK1.7)和元空间

>  java1.7永久代可以看做在JVM中的方法区中，永久存储区是一个常驻内存区域，用于存放JDK自身所携带的Class，Interface的元数据，也就是说它存储的是运行环境必须的类信息，被转载进此区域的数据是不会被垃圾回收器回收掉的，关闭JVM才会释放区域所占用的内存。

> 在java8中，永久代已经被移除，被元空间的区域所取代，元空间的本质和永久代类似
>
> 元空间与永久代最大的区别在于：
>
> 永久带使用的是JVM的堆内存，但是**java8之后的元空间并不在虚拟机中而是使用本机物理内存。**因此，默认情况下元空间的大小仅受本地内存限制。类的元数据放入native memory本地内存，字符串池和类的静态变量放在java堆内存中，这样加载多少类的元数据就不再受MaxPermsize控制，而由系统的可用空间控制。

### JVM调优

> JVM把内存区分为堆区(heap)、栈区(stack)和方法区(method)。由于本文主要讲解JVM调优，因此我们可以简单的理解为，JVM中的堆区中存放的是实际的对象，是需要被GC的。其他的都无需GC。

![](https://imgconvert.csdnimg.cn/aHR0cDovL3AzLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzJmNjdmYzQzMWE5NzQ4ODk5OWNiNWNlMzQ4ZjNjMzA5)

#### 堆内存调优简介

|           参数           |                   介绍                   |
| :----------------------: | :--------------------------------------: |
|           -Xms           | 设置初始分配大小，默认为物理内存的“1/64” |
|           -Xmx           |   最大分配内存，默认为物理内存的“1/4”    |
|   -XX:+PrintGCDetails    |           输出详细的GC处理日志           |
|      -XX:MaxNewSize      |               新生代最大值               |
|     -XX:MaxPermSize      |               永久代最大值               |
|    -XX:SurvivorRatio     |   新生代内存区域中Eden和Survivor的比例   |
| -XX:MaxTenuringThreshold |   设置对象在新生代中存活的次数（年龄）   |
|                          |                                          |

> ```txt
> 实际生产环境下，-Xms（初始内存）和-Xmx（最大内存）要调成一样大小
> 理由： 避免GC和应用程序争抢内存，理论值内存峰值和风阻忽高忽低
> 例如：-XMs1024m -Xmx1024m -XX:+PrintGCDetails
> ```

#### GC收集日志信息

**Minor GC日志信息**

![](http://img.blog.csdn.net/20131002100114703?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveXhjMTM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

[GC (Allocation Failure)

[PSYoungGen: 1589K->488K(2560K)] 1589K->716K(9728K), 0.0007593 secs]

[Times: user=0.03 sys=0.00, real=0.02 secs] 

**Full GC 日志信息**

![](http://img.blog.csdn.net/20131002100112187?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveXhjMTM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

[Full GC (Allocation Failure) 

[PSYoungGen: 0K->0K(2560K)]

[ParOldGen: 639K->621K(7168K)] 639K->621K(9728K), 

[Metaspace: 3139K->3139K(1056768K)], 0.0050743 secs] 

[Times: user=0.00 sys=0.00, real=0.01 secs]  

### GC是什么

- 次数上频收集Young区
- 次数上较少收集Old区
- 基本不动元空间

#### 如何判断一个对象是否可以被回收

- **引用计数法**

> 给内存中的对象给打上标记，对象被引用一次，计数就加1，引用被释放了，计数就减一，当这个计数为0的时候，这个对象就可以被回收了，java一般不采用这种方式。
>
> **缺点：**
>
> 1. 循环引用的对象是无法被识别出来并且被回收的。
> 2. 每次对象赋值均要维护引用计数器，且计数器本身也有一定的消耗

- **GC Roots**（可达性分析算法）

> 这个算法的基本思路就是通过一系列名为GC Roots的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链(Reference Chain)，当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的

#### GC 4大算法

- **复制算法**

  > 该算法**将内存平均分成两部分**，然后每次只使用其中的一部分，当这部分内存满的时候，将内存中所有存活的对象复制到另一个内存中，然后将之前的内存清空，只使用这部分内存，循环下去。
  >
  > **新生代中使用的是Minor GC，这种算法采用的是复制算法**，在回收时，将 Eden 和 Survivor 中还存活着的对象一次性复制到另一块 Survivor 空间上，最后清理 Eden 和 使用过的那一块 Survivor。
  >
  > **优点：**不会产生内存碎片
  >
  > **缺点：** **需要占用双倍空间**

- **标记清除**

  > 适合在**老年代**进行垃圾回收，它是最基本的收集算法。
  >
  > 分为“标记”和“清除”两个阶段：**首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。**
  >
  > **缺点：**1. 标记和清除的效率都不高
  >
  > ​            2.标记清除会**产生大量不连续的空间碎片**，空间碎片太多可能会导致程序运行过程需要分配较大的对象时候，无法找到足够连续内存而不得不提前触发一次垃圾收集。

- **标记-整理**（压碎）

  > 适合在**老年代**进行垃圾收集
  >
  > 分为标记和整理两个阶段：**首先标记出所有需要回收的对象，让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存**。
  >
  > **特点：**不会产生空间碎片，但是整理会花一定的时间。

- **分代收集算法**

## JMM(Java Memory Model)

java 内存模型，在多线程环境下，线程之间的互相通信

**JMM特征**

- 可见性
- 原子性
- 有序性

**模型**

![](https://img-blog.csdn.net/20180324210652599?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM4ODcwMDg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

JVM运行程序的实体是线程，而每个线程创建时JVM都会创建一个工作内存空间（有些地方称为栈空间），工作内存是每个线程的私有数据区域，而Java内存模型中规定所有变量都存储在**主内存**中，主内存是共享内存区域，所有线程都是可以访问，**但是线程对变量的操作必须在工作内存中进行，首先要将变量从主内存拷贝到的线程自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存**，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存的变量副本拷贝，因此不同的线程间无法访问对方的工作内存，线程间的通信必须通过主内存来完成。

