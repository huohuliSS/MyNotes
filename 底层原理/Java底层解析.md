> author : huohuliSS

[TOC]

# Java底层解析

## java设计模型

java的设计模型分为三种类型，共23种：

- 创建型模式

  **单例模式**、**抽象工厂模式**、**建造者模式**、**工厂模式**、原型模式

- 结构型模式

  **适配器模式**、桥接模式、**装饰模式**、组合模式、外观模式、享元模式、**代理模式**

- 行为型模式

  模板方法模式、命令模式、迭代器模式、**观察者模式**、中介者模式、备忘录模式、解释器模式、状态模式、**策略模式**、职责链（责任链）模式、访问模式

### 单例模式

> 每个类只能创建一个实例对象。构造方法为私有，单例模式主要作于java应用程序中，一个类Class只有一个实例存在。使用Singleton的好处在于可以**节省内存**，因为它限制了实例的个数，有利于java垃圾回收。

**饿汉模式**

> 饿汉模式是**典型的空间换时间**，被类加载时，静态变量instance会被初始化，此时类的私有构造子会被调用。这时候，单例类的唯一实例就被创建出来了。

```java
public class EagerSingleton {
    private static EagerSingleton instance = new EagerSingleton();
    /**
     * 私有默认构造子
     */
    private EagerSingleton(){}
    /**
     * 静态工厂方法
     */
    public static EagerSingleton getInstance(){
        return instance;
    }
}
```

**懒汉模式**

> 懒汉模式是**典型的时间换空间**，在创建对象实例的时候就不着急。会一直等到马上要使用对象实例的时候才会去创建。

```java
public class Single {

//   3. 为了确保对象创建的唯一性，添加属性
    private static Single instance = null;
//    4.为了线程安全，加锁
    private static Object lock = new Object();
//   1. 禁止调用构造方法,即类的外部不允许创建对象
    private Single(){
    }
//    2.建立一个方法，为调用者提供对象
    public static Single getInstance(){
//        5.为了节省资源，没必要每次进行加锁（只在第一次访问会出现线程被抢的问题）
        if (null == instance){
            synchronized (lock){
                if (null == instance){
                    instance = new Single();
                }
            }
        }
        return instance;
    }
}
```

### 工厂模式

1、**普通工厂模式**

> 就是建立一个工厂类，对实现了同一接口的一些类进行实例的创建

```java
// 接口
public interface Sender{
    public void send();
}
//实现类
public class MailSender implements Sender{
    @Override  
    public void send() {  
        System.out.println("this is mailsender!"); 
    }  
}
public class SmsSender implements Sender {  
    @Override  
    public void Send() {  
        System.out.println("this is sms sender!"); 
    }  
}

// 工厂类
public class SendFactory {  
    public Sender produce(String type) {  
        if ("mail".equals(type)) {  
            return new MailSender();  
        } else if ("sms".equals(type)) {  
            return new SmsSender();  
        } else {  
            System.out.println("请输入正确的类型!");  
            return null;  
        }  
    }  
}
```

```java
// 测试类
public class FactoryTest {  
    public static void main(String[] args) {  
        SendFactory factory = new SendFactory();  
        Sender sender = factory.produce("sms");  
        sender.Send();  
    }  
}
```

2、**多个工厂方法模式**

> 是对普通工厂方法模式的改进，在普通工厂方法模式中，如果传递的字符串出错误，则不能正确创建对象，而多个工厂方法模式是提供多个工厂方法，分别创建对象

```java
// 改动SendFactory类
public class SendFactory{
    public Sender produceMail(){
        return new MailSender();
    }
    public Sender produceSms(){
        return new SmsSender();
    }
}
```

```java
public class FactoryTest{
    public static void main(String[] args) {
        SendFactory factory = new SendFactory();  
        Sender sender = factory.produceMail();  
        sender.Send(); 
    }
}
```

3、静态工厂方法模式

> 将上面的多个工厂模式里的方法置为静态的，不需要创建实例，直接调用即可。

```java
public class SendFactory {        
    public static Sender produceMail(){  
        return new MailSender();  
    }  
    public static Sender produceSms(){  
        return new SmsSender();  
    }  
}
```

```java
public class FactoryTest {  
    public static void main(String[] args) {      
        Sender sender = SendFactory.produceMail(); 
        sender.Send();  
    }  
}
```

### 抽象工厂模式

工厂模式有一个问题就是，类的创建依赖工厂类，也就是说，如果想扩展程序，必须对工厂类进行修改，这样就违背了闭包原则，所以，从设计角度考虑，有一定的问题，如何解决？就用到了抽象工厂类，创建多个工厂类，这样一旦需要增加新的功能，**只需要做一个实现类，**实现Sender接口，**同时做一个工厂类**，实现Provider接口，就OK了，**不需要修改之前的代码，这样做扩展性好**。

```java
// 功能接口
public interface Sender{
    public void Send();
}
// 两个实现类
public class MailSender implements Sender{
    @Override  
    public void Send() {  
        System.out.println("this is mailsender!");  
    } 
}
public class SmsSender implements Sender {  
    @Override  
    public void Send() {  
        System.out.println("this is sms sender!");  
    }  
}

// 工厂实现接口
public interface Provider{
    public Sender produce();
}
// 两个工厂实现类
public class SendMailFactory implements Provider {  
      
    @Override  
    public Sender produce(){  
        return new MailSender();  
    }
}
public class SendSmsFactory implements Provider{  
  
    @Override  
    public Sender produce() {  
        return new SmsSender();  
    }  
}
```

### 建造者模式

工厂模式是提供创建单个类的模式，而建造者模式则是将各种产品集中起来进行管理，用来创建复合对象，所谓复合对象就是指某个类具有不同的属性。

> 建造者模式和工厂模式到的区别：
>
>    	工厂模式注重创建单个产品，而建造者模式则关注创建复合对象，多个部分。

### 代理模式

代理模式就是给对象提供一个代理对象，并由代理对象控制对原对象的引用。代理可以分为**静态代理**和**动态代理**

通过代理模式，**可以利用代理对象为被代理对象添加额外的功能**，**以此来扩展被代理对象的功能。**

1、**静态代理**

> **静态代理需要我们写出代理类和被代理类，而且一个代理类和一个被代理类一一对应**。代理类和被代理类需要事先同一个接口，通过聚合使得代理对象中有被代理对象的引用，以此实现代理对象控制被代理对象的目的。

```java
// 代理类和被代理类共同实现的接口
interface IService {
    void service();
}
// 被代理的类
class Service implements IService{
    @Override
    public void service() {
        System.out.println("被代理对象执行相关操作");
    }
}

// 代理类
class ProxyService implements IService{
    /**
     * 持有被代理对象的引用
     */
    private IService service;

    /**
     * 默认代理Service类
     */
    public ProxyService() {
        this.service = new Service();
    }
    /**
     * 也可以代理实现相同接口的其他类
     * @param service
     */
    public ProxyService(IService service) {
        this.service = service;
    }

    @Override
    public void service() {
        System.out.println("开始执行service()方法");
        service.service();
        System.out.println("service()方法执行完毕");
    }
}
```

```java
//测试类
public class ProxyPattern {
    public static void main(String[] args) {
        IService service = new Service();
        //传入被代理类的对象
        ProxyService proxyService = new ProxyService(service);
        proxyService.service();
    }
}
```

2、**动态代理**

> JDK 1.3 之后，Java通过java.lang.reflect包中的三个类Proxy、InvocationHandler、Method来支持动态代理。动态代理常用于有若干个被代理的对象，且为每个被代理对象添加的功能是相同的。
>
> **动态代理的代理类不需要我们编写，由java自动产生代理类源代码并进行编译最后生成代理对象。**

```java
interface IService{
    public void service();
}
// 被代理类
class ServiceTest implements IService{
    @Override
    public void service() {
        System.out.println("被代理类运行...");
    }
}
// 实现Invocationhandler接口
class InvocationProxy implements InvocationHandler{
    private Object object;

    public InvocationProxy(Object object){
        this.object = object;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("代理开始");
        //执行原对象的相关操作，容易忘记
        Object o = method.invoke(object, args);
        System.out.println("代理结束");
        return o;
    }
}
```

```java
public class InvocationProxyDemo02 {
    public static void main(String[] args) {

        IService service = new ServiceTest();
        //  使用Proxy.newProxyInstance，创建代理对象
        // 这里注意：jdk动态代理只能转成接口，不能转换为实现类
        IService Proxyinstance = (IService) Proxy.newProxyInstance(service.getClass().getClassLoader(),
                service.getClass().getInterfaces(), new InvocationProxy(service));
        Proxyinstance.service();
    }
}
```

## 5种网络I/O模型

### 阻塞I/O

![](http://static.oschina.net/uploads/space/2014/0726/163413_LWxd_220225.gif)

> 当**用户线程**发出IO请求之后，内核会去查看数据是否准备就绪，如果没有就绪就等待数据就绪，而用户线程就会处于阻塞状态，**用户交出CPU。**当数据就绪之后，内核会将数据拷贝到用户线程，并返回结果给用户线程，用户线程才解除block状态。
>
> **blocking IO的特点就是在IO执行的两个阶段（等待数据和拷贝数据两个阶段）都被阻塞了、**
>
> 典型应用： **阻塞Socket、Java BIO**
>
> **进程阻塞挂起不消耗CPU资源、及时响应每个操作**

### 非阻塞I/O

![](http://static.oschina.net/uploads/space/2014/0726/163739_X6pl_220225.gif)

> 当用户线程发起一个 read 操作后，并不需要等待，而是马上就得到了一个结果。如果结果是一个error 时，它就知道数据还没有准备好，于是它可以再次发送 read 操作。
>
> 如果内核中的数据准备好了，并且又再次收到了用户线程的请求，那么它马上就将数据拷贝到了用户线程，然后返回。所以事实上，在非阻塞 IO 模型中，**用户线程需要不断地询问内核数据是否就绪，**也就说非阻塞 IO不会交出 CPU，而会一直占用 CPU。
>
> 典型应用：Socket设置NONBLOCK
>
> **进程轮询（重复）调用、消耗CPU资源**

**所以，在非阻塞式IO中，用户进程其实是需要不断的主动询问kernel数据准备好了没有。**

### 多路复用IO（IO multiplexing）

![](http://static.oschina.net/uploads/space/2014/0726/163932_w5nW_220225.gif)

> 在多路复用IO模型中，会有一个**内核线程不断去轮询多个socket的状态**，只有当socket真正有读写事件时，才真正调用实际的IO读写操作。
>
> 因为在多路复用IO模型中，**只需要使用一个线程就可以管理多个socket，系统不需要创建新的进程或线程，也不必维护这些线程和进程，并且只有真正有socket读写事件进行时，才会使用IO资源，所以大大减少了资源占用。**
>
> 基本原理就是select/epoll这个系统函数会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。
>
> 典型应用：**Java NIO**、Nginx（epoll、poll、select）
>
> 专一进程解决多个进程IO的阻塞问题，性能好，Reactor模式

**注意**：

既然都是轮询，那为什么多路复用IO比非阻塞IO效率高呢？

> 因为在非阻塞IO中，不断询问socket状态时通过**用户线程**去进行的，而在多路复用IO中，轮询每个socket状态时在**内核**中进行的，**内核的效率要比用户线程要高的多！**

### 信号驱动式I/O

> 在信号驱动 IO 模型中，当用户线程发起一个 IO 请求操作，会给对应的 socket **注册一个信号函数**，然后用户线程会继续执行，当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用 IO 读写操作来进行实际的 IO 请求操作。

### 异步IO

> 用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

## 深入理解JAVA NIO

### Java NIO 与 IO 的区别

|          IO           |             NIO             |
| :-------------------: | :-------------------------: |
|    面向流(Stream)     |     面向缓冲区(Buffer)      |
| 阻塞IO（Blocking IO） | 非阻塞IO（non-blocking IO） |
|           /           |      选择器(Selector)       |

### 缓冲区(Buffer)和通道(Channel)

Buffer缓冲区是一个对象，它包含一些要写入或读出的数据，在NIO中，数据是放入Buffer对象中的，而在IO中，数据是直接写入或者读到Stream对象的，应用程序不能直接对Channel进行读写操作，而必须通过Buffer来进行，即Channel是通过Buffer来读写数据的，如下图：

![](https://img-blog.csdnimg.cn/2018121114430439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FpY2h1YW53ZW5kYW5n,size_16,color_FFFFFF,t_70)

> 在NIO中，所有的数据都是用Buffer处理的，它是NIO读写数据的中转池。Buffer实质上是一个数组，通常是一个字节数组，但也可以是其他类型的数组。但一个缓冲区不仅仅是一个数组，更重要的是它提供了对数据的结构化访问，而且还可以跟踪系统的读写进程。

**Buffer读写数据步骤：**

- 写入数据到Buffer；

- **调用flip()方法，将写模式变为读模式**；

- 从Buffer中读取数据

- 调用clear()方法或者compact()方法

  > 缓冲区的管理方式基本上一致，都是通过**allocate(Size)**获取缓存区。当向Buffer写入数据时，Buffer会记录下写了多少数据。一旦要读取数据，需要通过**flip()方法将Buffer从写模式切换到读模式**。在读模式下，可以读取之前写入到Buffer的所有数据。
  >
  > 一旦读完了所有的数据，就需要清空缓存区，让它可以再次被写入。有两种方式能清空缓冲区：调用 clear() 或 compact() 方法。clear() 方法会清空整个缓冲区。compact() 方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

Buffer缓冲区的四个核心属性：

- capacity ： 容量，表示缓冲区中最大存储数据的容量，一旦声明不能改变

- limit ： 界限，表示缓冲区中可以操作数据的大小。（limit后数据不能进行读写）

- position ： 位置，表示缓冲区中正在操作数据的位置

  

### 文件通道

### NIO的非阻塞式网络通信

#### 选择器Selector

**Selector**：通道和缓冲区机制，使得线程无需阻塞的等待IO事件的就绪，但是总是要有人开监管这些IO事件。这个工作就交给了selector来完成，这就是所谓的同步。

Selector允许单线程处理多个Channel。如果你的应用打开了多个连接（通道），但每连接的流量都很低，使用Selector就会很方便。

要使用Selector，得向Selector注册Channel，然后调用它的select()方法，这个方法就一直阻塞到某个注册的通道有事件就绪，这就是所说的轮询。一旦这个方法返回，线程就可以处理这些事件。

#### SocketChannel、ServerSocketChannel、DatagramChannel

### 管道



### NIO个人理解

java NIO使用的是**多路复用IO**模型，**在不考虑多线程的情况下，BIO是不能处理高并发的**。

现在给出一场景，如果单线程有同时1000万个连接（SocketChannel），但真正有数据发送的只有200万，而剩下的800万一直处于连接状态，如果所有socket都交给用户线程轮询去做，会造成很大的资源浪费。

多路复用IO解决的办法是，**将socket交给操作系统内核处理**，**内核会主动感知有数据的socket（在linux下，叫文件描述符fd）**。

- 在window环境下应用系统通过java运行，java再通过轮询去请求操作系统**select()**函数（应用系统   ->   java  ->  轮询  ->  操作系统函数select）；
- linux环境下，NIO模式下应用程序会直接请求操作系统的**epoll()**函数，系统会主动感知线程中有数据发送的**文件描述符**，不用去轮询，从而加大了对资源的高效利用。

## 锁

### 悲观锁和乐观锁

> 悲观锁总是假设最坏的情况，每次去拿数据的时候都会认为别人会修改，所以每次拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁（**共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其他线程**）。比如说行锁，表锁，读锁，写锁等，Java中synchronized和ReentrantLock等独占锁就是悲观锁思想的实现。
>
> 乐观锁总是假设最好的情况，每次去拿数据的时候都认为别人不回去修改数据，所以不会上锁，但是在更新数据的时候会判断一下在此期间别人会有没有去更新这个数据，<u>可以使用**版本号**和**CAS算法**实现。</u>乐观锁使用于多读的应用类型，这样可以提高吞吐量，在Java中java.util.concurrent.atomic包下的原子变量类就是使用了**乐观锁**的一种实现方式CAS实现的

**乐观锁实现的两种实现方式**

- 版本号机制

  > 一般在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，verison会加1，当线程A要更新数据值时，在读取数据的同时也会读取version，在提交更新时，若刚才读到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。

- CAS算法（compare and swap）

  > compare and swap （比较与交换），是一种有名的无锁算法。无锁编程，即**不使用锁的情况下实现多线程之间的变量同步**。当且仅当读入内存的值等于进行比较的值时，CAS通过原子方式用新值来更新读入内存的值。一般情况下是一个**自旋操作**，即不断重试。

**乐观锁的缺点**

- ABA问题

  > 如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，在这段时间它的值可能被改为其他值，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个问题被称为CAS操作的 **"ABA"问题。**

- 循环时间长开销大

  > **自旋CAS（也就是不成功就一直循环执行直到成功）如果长时间不成功，会给CPU带来非常大的执行开销。**

_________________________________________________

还有老多没有写到的地方，请多关照