---
title: Java多线程与并发
date: 2019-03-09 21:37:29
tags: [Java,多线程,并发]
---
# 线程池

## 基本组成

1、线程池管理器（ThreadPool）：用于创建并管理线程池，包括 创建线程池，销毁线程池，添加新任务；
2、工作线程（PoolWorker）：线程池中线程，在没有任务时处于等待状态，可以循环的执行任务；
3、任务接口（Task）：每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定了任务的入口，任务执行完后的收尾工作，任务的执行状态等；
4、任务队列（taskQueue）：用于存放没有处理的任务。提供一种缓冲机制。
<!--more-->

- 降低资源消耗。 通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- 提高响应速度。 当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- 提高线程的可管理性。使用线程池可以进行统一的分配，调优和监控，延时执行、定时循环执行的策略等。

![image](http://490.github.io/images/20190310_101501.png)
java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类

构造参数

```java
java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类

构造参数
public ThreadPoolExecutor(
int ==corePoolSize==,该线程池中核心线程数最大值。线程池新建线程的时候，如果当前线程总数小于corePoolSize，则新建的是核心线程，如果超过corePoolSize，则新建的是非核心线程  核心线程默认情况下会一直存活在线程池中，即使这个核心线程啥也不干(闲置状态)。  如果指定ThreadPoolExecutor的allowCoreThreadTimeOut这个属性为true，那么核心线程如果不干活(闲置状态)的话，超过一定时间(时长下面参数决定)，就会被销毁掉  
int ==maximumPoolSize==,（线程不够用时能够创建的最大线程数）线程总数 = 核心线程数 + 非核心线程数
long ==keepAliveTime==,非核心线程闲置超时时长
TimeUnit ==unit==,keepAliveTime的单位，TimeUnit是一个枚举类型
BlockingQueue<Runnable> ==workQueue==,任务队列：维护着等待执行的Runnable对象  当所有的核心线程都在干活时，新添加的任务会被添加到这个队列中等待处理，如果队列满了，则新建非核心线程执行任务                          
ThreadFactory ==threadFactory==, 创建新线程，Executors.defaultThreadFactory()
RejectedExecutionHandler ==handler ==线程池的饱和策略)
```
![image](http://490.github.io/images/20190310_101645.png)

通过ThreadPoolExecutor.execute(Runnable command)方法即可向线程池内添加一个任务

**当一个任务被添加进线程池时**：
![image](http://490.github.io/images/20190310_101722.png)

1、线程数量未达到corePoolSize，则新建一个线程(核心线程)执行任务
2、线程数量达到了corePools，则将任务移入队列BlockingQueue等待
3、如果无法将任务加入BlockingQueue（队列已满），则在非corePool中创建新的线程来处理任务（注意，执行这一步骤需要获取全局锁）。
4、队列已满，总线程数又达到了maximumPoolSize，(RejectedExecutionHandler)抛出异常
![image](http://490.github.io/images/20190310_101759.png)
![image](http://490.github.io/images/20190310_101805.png)

## 四种java实现好的线程池

**CachedThreadPool**()

可缓存线程池：
线程数无限制
有空闲线程则复用空闲线程，若无空闲线程则新建线程
一定程序减少频繁创建/销毁线程，减少系统开销

**FixedThreadPool**()

定长线程池：
可控制线程最大并发数（同时执行的线程数）
超出的线程会在队列中等待

**ScheduledThreadPool**()

定长线程池：
支持定时及周期性任务执行。

**SingleThreadExecutor**()

单线程化的线程池：
有且仅有一个工作线程执行任务
所有任务按照指定顺序执行，即遵循队列的入队出队规则

```
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;public  class ThreadPoolExecutorTest {
public  static  void main(String[] args) {
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
scheduledThreadPool.schedule(new Runnable() {
public  void run() {
System.out.println("delay 3 seconds");
}
}, 3, TimeUnit.SECONDS);
}
}
```

## 线程池的状态

![image](http://490.github.io/images/20190310_102048.png)
![image](http://490.github.io/images/20190310_102239.png)
![image](http://490.github.io/images/20190310_102245.png)
![image](http://490.github.io/images/20190310_102252.png)

## 线程池风险：

死锁、资源不足、并发错误、 线程泄漏、请求过载



## Executor

管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor：

*   CachedThreadPool：一个任务创建一个线程；
*   FixedThreadPool：所有任务只能使用固定大小的线程；
*   SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
```

![image](http://490.github.io/images/20190310_102602.png)
![image](http://490.github.io/images/20190310_102609.png)



## 执行execute()方法和submit()方法的区别是什么呢？

1. execute() 方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；
2. submit()方法用于提交需要返回值的任务。线程池会返回一个future类型的对象，通过这个future对象可以判断任务是否执行成功，并且可以通过future的get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成，而使用`get（long timeout，TimeUnit unit）` 方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。


# 继承Thread、实现Runnable接口和Callable接口

## Thread类

如果一个类继承Thread，则不适合资源共享。但是如果实现了Runable接口的话，则很容易的实现资源共享。实现Runnable接口比继承Thread类所具有的优势：
- 适合多个相同的程序代码的线程去处理同一个资源
- 可以避免java中的单继承的限制
- 增加程序的健壮性，代码可以被多个线程共享，代码和数据独立

```java
//继承 Thread
Thread1 mTh1=new Thread1("A");
Thread1 mTh2=new Thread1("B");
mTh1.start();
mTh2.start();
```

## Runnable接口

```java
先看一下**java.lang.Runnable**吧，它是一个接口，在它里面只声明了一个run()方法：由于run()方法返回值为void类型，所以在执行完任务之后无法返回任何结果。

public interface Runnable {
    public abstract void run();
}

//实现Runnable接口
Thread2 mTh = new Thread2();
new Thread(mTh, "C").start();//同一个mTh，但是在Thread中就不可以，如果用同一个实例化对象mt，就会出现异常   
new Thread(mTh, "D").start();
new Thread(mTh, "E").start();
```

这两种方式都有一个**缺陷**就是：在执行完任务之后无法获取执行结果。如果需要获取执行结果，就必须通过共享变量或者使用线程通信的方式来达到效果，这样使用起来就比较麻烦。

而自从Java 1.5开始，就提供了Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。

## Callable接口

**Callable**接口位于java.util.concurrent包下，在它里面也只声明了一个方法，只不过这个方法叫做call()。
```java
public interface Callable<V> {
    V call() throws Exception;
}
```

可以看到，这是一个泛型接口，call()函数返回的类型就是传递进来的V类型。Callable接口可以看作是Runnable接口的补充，call方法带有返回值，并且可以抛出异常。

## FutureTask类

如何获取Callable的返回结果呢？一般是通过FutureTask这个中间媒介来实现的。整体的流程是这样的：
- 把Callable实例当作参数，生成一个FutureTask的对象
- 然后把这个对象当作一个Runnable，作为参数另起线程。

![image](http://490.github.io/images/20190316_204353.png)

由于FutureTask实现了Runnable，因此它既可以通过Thread包装来直接执行，也可以提交给ExecuteService来执行。下面以Thread包装线程方式启动来说明一下。


```java
import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;
public class Demo 
{
    public static void main(String[] args) throws Exception 
    {
        Callable<Integer> call = new Callable<Integer>() 
        {
            public Integer call() throws Exception 
            {
                System.out.println("计算线程正在计算结果...");
                Thread.sleep(3000);
                return 1;
            }
        };

        FutureTask<Integer> task = new FutureTask<>(call);

        Thread thread = new Thread(task);
        thread.start();

        System.out.println("main线程干点别的...");
        Integer result = task.get();
        System.out.println("从计算线程拿到的结果为：" + result);
    }
}
```

## Future接口

FutureTask继承体系中的核心接口是Future。

**Future的核心思想是**：一个方法，计算过程可能非常耗时，等待方法返回，显然不明智。可以在调用方法的时候，立马返回一个Future，可以通过Future这个数据结构去控制方法f的计算过程。这里的控制包括：

```java
public interface Future<V> 
{
    boolean cancel(boolean mayInterruptIfRunning);//还没计算完，可以取消计算过程
    boolean isCancelled();///判断计算是否被取消
    boolean isDone();//判断是否计算完
    V get() throws InterruptedException, ExecutionException;
    //获取计算结果（如果还没计算完，也是必须等待的）
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```
也就是说Future提供了三种功能：
- 判断任务是否完成；
- 能够中断任务；
- 能够获取任务执行结果。







## 实现Runnable接口和Callable接口的区别

如果想让线程池执行任务的话需要实现的Runnable接口或Callable接口。

 Runnable接口或Callable接口实现类都可以被ThreadPoolExecutor或ScheduledThreadPoolExecutor执行。两者的区别在于 Runnable 接口不会返回结果但是 Callable 接口可以返回结果。

备注： 工具类 Executors 可以实现 Runnable 对象和 Callable 对象之间的相互转换。
（`Executors.callable（Runnable task)也就是说Future提供了三种功能：

　　1）判断任务是否完成；

　　2）能够中断任务；

　　3）能够获取任务执行结果。` 或 
`Executors.callable（Runnable task，Object resule）` ）。

# volatile关键字

如果一个变量在多个CPU中都存在缓存（一般在多线程编程时才会出现），那么就可能存在缓存不一致的问题。为了解决缓存不一致性问题，通常来说有以下2种解决方法：
1. 通过在总线加LOCK#锁的方式
2. 通过缓存一致性协议

这2种方式都是硬件层面上提供的方式。

i如果在执行这段代码的过程中，在总线上发出了LCOK#锁的信号，那么只有等待这段代码完全执行完毕之后，其他CPU才能从变量i所在的内存读取变量，然后进行相应的操作。这样就解决了缓存不一致的问题。

缓存一致性协议。最出名的就是Intel 的MESI协议，MESI协议保证了每个缓存中使用的共享变量的副本是一致的。它核心的思想是：当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。
![image](http://490.github.io/images/20190310_102654.png)

## 原子性问题，可见性问题，有序性问题

　　对于可见性，Java提供了volatile关键字来保证可见性。
　　当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。
　　而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。

另外，通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性。

下面这段话摘自《深入理解Java虚拟机》：
“观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”
　　　
lock前缀指令实际上相当于一个**内存屏障（也成内存栅栏）**，内存屏障会提供3个功能：
1. 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
2. 它会强制将对缓存的修改操作立即写入主存；
3. 如果是写操作，它会导致其他CPU中对应的缓存行无效。

下面列举几个Java中使用volatile的几个场景。
- 状态标记量
- double check防止指令重排序:
 防止指令重排序导致其他线程获取到未初始化完的对象。`instance = new Singleton()`这句，这并非是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情。
   -  给 instance 分配内存
   - 调用 Singleton 的构造函数来初始化成员变量
   - 将instance对象指向分配的内存空间（执行完这步 instance 就为非 null 了）

  但是在 JVM 的即时编译器中存在指令重排序的优化。也就是说上面的第二步和第三步的顺序是不能保证的，最终的执行顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，则在 3 执行完毕、2 未执行之前，被线程二抢占了，这时 instance 已经是非 null 了（但却没有初始化），所以线程二会直接返回 instance，然后使用，然后报错。加了volatile就不会重排序。




## synchronized 关键字和 volatile 关键字的区别

- volatile关键字是线程同步的轻量级实现，所以volatile性能肯定比synchronized关键字要好。但是volatile关键字只能用于变量，而synchronized关键字可以修饰方法以及代码块。synchronized关键字在JavaSE1.6之后，为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁以及其它各种优化，执行效率有了显著提升，实际开发中使用 synchronized 关键字的场景还是更多一些。
- 多线程访问volatile关键字不会发生阻塞，而synchronized关键字可能会发生阻塞。
- volatile关键字能保证数据的可见性，但不能保证数据的原子性。synchronized关键字两者都能保证。
- volatile关键字主要用于解决变量在多个线程之间的可见性，而 synchronized关键字解决的是多个线程之间访问资源的同步性。

# Daemon

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。
当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。
main() 属于非守护线程。
使用 setDaemon() 方法将一个线程设置为守护线程。

```java
public  static  void main(String[] args) {
Thread thread =  new Thread(new MyRunnable());
thread.setDaemon(true);
}
```

* * *

# synchronized关键字

JVM 实现的 synchronized
JDK 实现的 ReentrantLock

- synchronized关键字解决的是多个线程之间访问资源的同步性，synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。
- 在 Java 早期版本中，synchronized属于重量级锁，效率低下，因为监视器锁（monitor）是依赖于底层的操作系统的 Mutex Lock 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。
- Java 6 之后 Java 官方对从 JVM 层面对synchronized 较大优化，JDK1.6对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

## synchronized关键字最主要的三种使用方式

- **修饰实例方法**，作用于当前对象实例加锁，进入同步代码前要获得当前**对象实例**的锁。
- **修饰静态方法**，作用于当前类对象加锁，进入同步代码前要获得当前**类对象**的锁 。也就是给当前类加锁，会作用于类的所有对象实例，因为静态成员不属于任何一个实例对象，是类成员（ static 表明这是该类的一个静态资源，不管new了多少个对象，只有一份，所以对该类的所有对象都加了锁）。所以如果一个线程A调用一个实例对象的非静态 synchronized 方法，而线程B需要调用这个实例对象所属类的静态 synchronized 方法，是允许的，不会发生互斥现象，因为访问静态 synchronized 方法占用的锁是当前类的锁，而访问非静态 synchronized 方法占用的锁是当前实例对象锁。
- **修饰代码块**，指定加锁对象，对给定对象加锁，进入同步代码库前要获得**给定对象**的锁。 和 synchronized 方法一样，synchronized(this)代码块也是锁定当前对象的。synchronized 关键字加到 static 静态方法和 synchronized(class)代码块上都是是给 Class 类上锁。这里再提一下：synchronized关键字加到非 static 静态方法上是给对象实例上锁。另外需要注意的是：尽量不要使用 synchronized(String a) 因为JVM中，字符串常量池具有缓冲功能。

## 双重校验锁实现对象单例（线程安全）

```java
public class Singleton 
{
    private volatile static Singleton uniqueInstance;
    private Singleton() {
    } 
    public static Singleton getUniqueInstance() 
    {
        //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) 
        {
            //类对象加锁
            synchronized (Singleton.class)
            {
                if (uniqueInstance == null) 
                {
                    uniqueInstance = new Singleton();
                }
            }
        } 
        return uniqueInstance;
    }
}
```

另外，需要注意 uniqueInstance 采用 volatile 关键字修饰也是很有必要。 uniqueInstance 采用 volatile 关键字修饰也是很有必要的，`uniqueInstance = new Singleton()` 这段代码其实是分为三步执行：
1. 为 uniqueInstance 分配内存空间 
2. 初始化 uniqueInstance
3. 将 uniqueInstance 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出先问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。

使用 volatile 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。

## synchronized 关键字的底层原理

### synchronized 同步语句块的情况

```java
public class SynchronizedDemo 
{
    public void method() 
    {
      synchronized (this) {
          System.out.println("synchronized 代码块");
      }
    }
}
```

通过 JDK 自带的 javap 命令查看 SynchronizedDemo 类的相关字节码信息：首先切换到类的对应目录执行`javac SynchronizedDemo.java`命令生成编译后的 .class 文件，然后执行
 `javap -c -s -v -l SynchronizedDemo.class` 

![image](http://490.github.io/images/20190314_185500.png)

从上面我们可以看出：
 synchronized 同步语句块的实现使用的是 `monitorenter`和 `monitorexit`指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。 
 
 当执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor(monitor对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权。当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

### synchronized 修饰方法的的情况

```java
public class SynchronizedDemo2 {
    public synchronized void method() {
        System.out.println("synchronized 方法");
    }
}
```

![image](http://490.github.io/images/20190314_192145.png)

synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 `ACC_SYNCHRONIZED `标识，该标识指明了该方法是一个同步方法，JVM 通过该访问标志来
辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

## synchronized和ReenTrantLock 的区别 

- 两者都是可重入锁
 两者都是可重入锁。“可重入锁”概念是：自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。
- synchronized 依赖于 JVM 而 ReenTrantLock 依赖于 API
synchronized 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 synchronized 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。ReenTrantLock 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock 方法配合 try/finally 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。
- ReenTrantLock 比 synchronized 增加了一些高级功能
相比synchronized，ReenTrantLock增加了一些高级功能。主要来说主要有三点：
  - 等待可中断；
  - 可实现公平锁；
  - 可实现选择性通知（锁可以绑定多个条件）

- ReenTrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
- ReenTrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。 ReenTrantLock默认情况是非公平的，可以通过 ReenTrantLock类的ReentrantLock(boolean fair) 构造方法来制定是否是公平的。
- synchronized关键字与wait()和notify/notifyAll()方法相结合可以实现等待/通知机制，ReentrantLock类当然也可以实现，但是需要借助于Condition接口与newCondition() 方法。Condition是JDK1.5之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个Lock对象中可以创建多个Condition实例（即对象监视器），线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用notify/notifyAll()方法进行通知时，被通知的线程是由 JVM 选择的，用ReentrantLock类结合Condition实例可以实现“选择性通知” ，这个功能非常重要，而且是Condition接口默认提供的。而synchronized关键字就相当于整个Lock对象中只有一个Condition实例，所有的线程都注册在它一个身上。如果执行notifyAll()方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而Condition实例的signalAll()方法 只会唤醒注册在该Condition实例中的所有等待线程。

![image](http://490.github.io/images/20190310_103034.png)


**锁的不是代码。是对象**


![image](http://490.github.io/images/20190310_103049.png)
![image](http://490.github.io/images/20190310_103055.png)
![image](http://490.github.io/images/20190310_103102.png)

对象在内存中的布局：对象头、实例数据、对齐填充
![image](http://490.github.io/images/20190310_103127.png)
![image](http://490.github.io/images/20190310_103134.png)




# 锁

## 悲观锁

总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁（共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程）。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先
上锁。Java中 synchronized和 ReentrantLock等独占锁就是悲观锁思想的实现。
## 乐观锁

总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制和CAS算法实现。乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库提供的类似于write_condition机制，其实都是提供的乐观锁。在Java中 java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。

从上面对两种锁的介绍，我们知道两种锁各有优缺点，不可认为一种好于另一种，像乐观锁适用于写比较少的情况下（多读场景），即冲突真的很少发生的时候，这样可以省去了锁的开销，加大了系统的整个吞吐量。但如果是多写的情况，一般会经常产生冲突，这就会导致上层应用会不断的进行retry，这样反倒是降低了性能，所以一般多写的场景下用悲观锁就比较合适。

**乐观锁一般会使用版本号机制或CAS算法实现**。

### 版本号机制

一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读取到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。

举一个简单的例子：假设数据库中帐户信息表中有一个 version 字段，当前值为 1 ；而当前帐户余额字段（ balance ）为 $100 。

```
1、操作员 A 此时将其读出（ version=1 ），并从其帐户余额中扣除 $50（ $100-$50 ）。
2、在操作员 A 操作的过程中，操作员B 也读入此用户信息（ version=1 ），并从其帐户余额中扣除 $20 （ $100-$20 ）。
3、操作员 A 完成了修改工作，将数据版本号加一（ version=2 ），连同帐户扣除后余额（ balance=$50 ），提交至数据库更新，此时由于提交数据版本大于数据库记录当前版本，数据被更新，数据库记录 version 更新为 2 。
4、操作员 B 完成了操作，也将版本号加一（ version=2 ）试图向数据库提交数据（ balance=$80 ），但此时比对数据库记录版本时发现，操作员 B 提交的数据版本号为 2 ，数据库记录当前版本也为 2 ，不满足 “ 提交版本必须大于记录当前版本才能执行更新 “ 的乐观锁策略，因此，操作员 B 的提交被驳回。
```

这样，就避免了操作员 B 用基于 version=1 的旧数据修改的结果覆盖操作员A 的操作结果的可能。

### CAS算法

**CAS算法**
乐观锁的一种表现。 即compare and swap（比较与交换），是一种有名的无锁算法。无锁编程，即不使用锁的情况下实现多线程之间的变量同步，也就是在没有线程被阻塞的情况下实现变量的同步，所以也叫非阻塞同步（Non-blocking Synchronization）。CAS算法涉及到三个操作数

*   需要读写的内存值 V
*   进行比较的值 A
*   拟写入的新值 B

当且仅当 V 的值等于 A时，CAS通过原子方式用新值B来更新V的值，否则不会执行任何操作（比较和替换是一个原子操作）。一般情况下是一个自旋操作，即不断的重试。

**自旋锁、自适应自旋锁**
![image](http://490.github.io/images/20190310_103148.png)
![image](http://490.github.io/images/20190310_103208.png)
自旋锁与互斥锁比较类似，它们都是为了解决对某项资源的互斥使用。无论是互斥锁，还是自旋锁，在任何时刻，最多只能有一个保持者，也就说，在任何时刻最多只能有一个执行单元获得锁。但是两者在调度机制上略有不同。对于互斥锁，如果资源已经被占用，资源申请者只能进入睡眠状态。但是自旋锁不会引起调用者睡眠，如果自旋锁已经被别的执行单元保持，调用者就一直循环在那里看是否该自旋锁的保持者已经释放了锁。

```java
public class SpinLock {
    private AtomicReference<Thread> cas = new AtomicReference<Thread>();
    public void lock() {
        Thread current = Thread.currentThread();
        // 利用CAS
        while (!cas.compareAndSet(null, current)) {
            // DO nothing
        }
    }
    public void unlock() {
        Thread current = Thread.currentThread();
        cas.compareAndSet(current, null);
    }
}
```

### 乐观锁的缺点

**1、 ABA 问题**
如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个问题被称为CAS操作的 "ABA"问题。

JDK 1.5 以后的 AtomicStampedReference类就提供了此种能力，其中的 compareAndSet方法就是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

**2 、循环时间长开销大**
自旋CAS（也就是不成功就一直循环执行直到成功）如果长时间不成功，会给CPU带来非常大的执行开销。 如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

**3、 只能保证一个共享变量的原子操作**
CAS 只对单个共享变量有效，当操作涉及跨多个共享变量时 CAS 无效。但是从 JDK 1.5开始，提供了 AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行 CAS 操作.所以我们可以使用锁或者利用 AtomicReference类把多个共享变量合并成一个共享变量来操作。

## CAS与synchronized的使用情景

简单的来说CAS适用于写比较少的情况下（多读场景，冲突一般较少），synchronized适用于写比较多的情况下（多写场景，冲突一般较多）

对于资源竞争较少（线程冲突较轻）的情况，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户态内核态间的切换操作额外浪费消耗cpu资源；而CAS基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少，因此可以获得更高的性能。

对于资源竞争严重（线程冲突严重）的情况，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于synchronized。

补充： Java并发编程这个领域中synchronized关键字一直都是元老级的角色，很久之前很多人都会称它为 “重量级锁” 。但是，在JavaSE 1.6之后进行了主要包括为了减少获得锁和释放锁带来的性能消耗而引入的 偏向锁 和 轻量级锁 以及其它各种优化之后变得在某些情况下并不是那么重了。synchronized的底层实现主要依靠 Lock-Free 的队列，基本思路是 自旋后阻塞，竞争切换后继续竞争锁，稍微牺牲了公平性，但获得了高吞吐量。在线程冲突较少的情况下，可以获得和CAS类似的性能；而线程冲突严重的情况下，性能远高于CAS。

# Lock锁的使用

## Lock接口简介
锁是用于通过多个线程控制对共享资源的访问的工具。通常，锁提供对共享资源的独占访问：一次只能有一个线程可以获取锁，并且对共享资源的所有访问都要求首先获取锁。 但是，一些锁可能允许并发访问共享资源，如ReadWriteLock的读写锁。

在Lock接口出现之前，Java程序是靠synchronized关键字实现锁功能的。JDK1.5之后并发包中新增了Lock接口以及相关实现类来实现锁功能。

虽然synchronized方法和语句的范围机制使得使用监视器锁更容易编程，并且有助于避免涉及锁的许多常见编程错误，但是有时您需要以更灵活的方式处理锁。例如，用于遍历并发访问的数据结构的一些算法需要使用“手动”或“链锁定”：您获取节点A的锁定，然后获取节点B，然后释放A并获取C，然后释放B并获得D等。在这种场景中synchronized关键字就不那么容易实现了，使用Lock接口容易很多。

## Lock的简单使用

Lock接口的实现类： 
ReentrantLock ， ReentrantReadWriteLock.ReadLock ， ReentrantReadWriteLock.WriteLock

```java
Lock lock=new ReentrantLock()；
lock.lock();
try{
    }finally{
    lock.unlock();
}
```

**Lock接口特性**

| 特性      | 描述    | 
| --------- | -------- |
| 尝试非阻塞地获取锁|当前线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁 |
|能被中断地获取锁|获取到锁的线程能够响应中断，当获取到锁的线程被中断时，中断异常将会被抛出，同时锁会被释放|
|超时获取锁|在指定的截止时间之前获取锁， 超过截止时间后仍旧无法获取则返回|

![image](http://490.github.io/images/20190314_203513.png)

**ReentrantLock类常见方法：**

| 方法名称 | 描述 |
| --- | --- |
| ReentrantLock() | 创建一个 ReentrantLock的实例。 |
| ReentrantLock(boolean fair) | 创建一个特定锁类型（公平锁/非公平锁）的ReentrantLock的实例 |

![image](http://490.github.io/images/20190314_203535.png)

## Condition接口简介

synchronized关键字与wait()和notify/notifyAll()方法相结合可以实现等待/通知机制，ReentrantLock类当然也可以实现，但是需要借助于Condition接口与newCondition() 方法。

Condition是JDK1.5之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个Lock对象中可以创建多个Condition实例（即对象监视器），线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。

在使用notify/notifyAll()方法进行通知时，被通知的线程是有JVM选择的，使用ReentrantLock类结合Condition实例可以实现“选择性通知”，这个功能非常重要，而且是Condition接口默认提供的。

而synchronized关键字就相当于整个Lock对象中只有一个Condition实例，所有的线程都注册在它一个身上。如果执行notifyAll()方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而Condition实例的signalAll()方法 只会唤醒注册在该Condition实例中的所有等待线程

![image](http://490.github.io/images/20190314_203548.png)

## 公平锁与非公平锁

Lock锁分为：公平锁 和 非公平锁。
- 公平锁表示线程获取锁的顺序是按照线程加锁的顺序来分配的，即先来先得的FIFO先进先出顺序。
- 非公平锁就是一种获取锁的抢占机制，是随机获取锁的，和公平锁不一样的就是先来的不一定先的到锁，这样可能造成某些线程一直拿不到锁，结果也就是不公平的了。







# 线程间的协作

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

## wait、notify、notifyAll()

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateExeception。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

* wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
* wait() 会释放锁，sleep() 不会，是让出cpu。

## await、signal、signalAll

```java
线程consumer
synchronize(obj){     
    obj.wait();//没东西了，等待
}

线程producer
synchronize(obj){ 
    obj.notify();//有东西了，唤醒 
}

lock.lock();
condition.await();
lock.unlock();

lock.lock();
condition.signal();
lock.unlock;
```

为了突出区别，省略了若干细节。区别有三点：

*   lock不再用synchronize把同步代码包装起来；
*   阻塞需要另外一个对象condition；
*   同步和唤醒的对象是condition而不是lock，对应的方法是await和signal，而不是wait和notify。

为什么需要使用condition呢？简单一句话，lock更灵活。以前的方式只能有一个等待队列，在实际应用时可能需要多个，比如读和写。为了这个灵活性，lock将同步互斥控制和等待队列分离开来，互斥保证在某个时刻只有一个线程访问临界区（lock自己完成），等待队列负责保存被阻塞的线程（condition完成）。

通过查看ReentrantLock的源代码发现，condition其实是等待队列的一个管理者，condition确保阻塞的对象按顺序被唤醒。

在Lock的实现中，LockSupport被用来实现线程状态的改变，后续将更进一步研究LockSupport的实现机制。

* * *
# 线程的状态

![image](http://490.github.io/images/20190310_133206.png)
![image](http://490.github.io/images/20190310_133216.png)
![image](http://490.github.io/images/20190310_133232.png)
![image](http://490.github.io/images/20190310_133238.png)

**yield**
使当前线程从执行状态（运行状态）变为可执行态（就绪状态）。cpu会从众多的可执行态里选择，也就是说，当前也就是刚刚的那个线程还是有可能会被再次执行到的，并不是说一定会执行其他线程而该线程在下一次中不会执行到了。

用了yield方法后，该线程就会把CPU时间让掉，让其他或者自己的线程执行（也就是谁先抢到谁执行）

**中断线程**
![image](http://490.github.io/images/20190310_133316.png)


# ThreadLocal

ThreadLocal 不继承 Thread，也不实现 Runable 接口， ThreadLocal 类为每一个线程都维护了自己独有的变量拷贝。每个线程都拥有自己独立的变量，其作用在于数据独立。ThreadLocal 采用 hash 表的方式来为每个线程提供一个变量的副本

# 阻塞队列

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

- ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
- LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
- PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
- DelayQueue：一个使用优先级队列实现的无界阻塞队列。
- SynchronousQueue：一个不存储元素的阻塞队列。
- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
- LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

用优先队列实现最大堆最小堆

```java
Comparator<Integer> mycomparator = new Comparator<Integer>() {
                  @Override
                  public int compare(Integer o1, Integer o2) 
                  {
                      return [o2.compareTo](http://o2.compareTo)(o1);
                  }
            };
    maxheap = new PriorityQueue<Integer>(20,mycomparator);
    minheap = new PriorityQueue<Integer>(20);
```



# Atomic 原子类

Atomic 翻译成中文是原子的意思。在化学上，我们知道原子是构成一般物质的最小单位，在化学反应中是不可分割的。在我们这里 Atomic 是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。

所以，所谓原子类说简单点就是具有原子/原子操作特征的类。并发包 java.util.concurrent 的原子类都存放在 java.util.concurrent.atomic 下。

## JUC 包中的原子类是哪4类

- 基本类型
使用原子的方式更新基本类型 
  - AtomicInteger：整形原子类 
  - AtomicLong：长整型原子类
  - AtomicBoolean ：布尔型原子类
- 数组类型
使用原子的方式更新数组里的某个元素
  - AtomicIntegerArray：整形数组原子类
  - AtomicLongArray：长整形数组原子类
  - AtomicReferenceArray ：引用类型数组原子类
- 引用类型 
  - AtomicReference：引用类型原子类 
  - AtomicStampedRerence：原子更新引用类型里的字段原子类 
  - AtomicMarkableReference ：原子更新带有标记位的引用类型
- 对象的属性修改类型
  - AtomicIntegerFieldUpdater:原子更新整形字段的更新器
  - AtomicLongFieldUpdater：原子更新长整形字段的更新器 
  - AtomicStampedReference ：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。

## AtomicInteger 类常用方法

```java
public final int get() //获取当前的值
public final int getAndSet(int newValue)//获取当前的值，并设置新的值
public final int getAndIncrement()//获取当前的值，并自增
public final int getAndDecrement() //获取当前的值，并自减
public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
public final void lazySet(int newValue)//最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
```

使用 AtomicInteger 之后，不用对 increment() 方法加锁也可以保证线程安全

## AtomicInteger 类的原理

```java
// setup to use Unsafe.compareAndSwapInt for updates（更新操作时提供“比较并替换”的作用）
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;
static {
      try {
          valueOffset = unsafe.objectFieldOffset
          (AtomicInteger.class.getDeclaredField("value"));
      } catch (Exception ex) 
      { throw new Error(ex); }
} 
private volatile int value;
```

AtomicInteger 类主要利用 CAS (compare and swap) + volatile 和 native 方法来保证原子操作，从而避免 synchronized 的高开销，执行效率大为提升。 CAS的原理是拿期望的值和原本的一个值作比较，如果相同则更新成新的值。UnSafe 类的 objectFieldOffset() 方法是一个本地方法，这个方法是用来拿到“原来的值”的内存地址，返回值是 valueOffset。另外 value 是一个volatile变量，在内存中可见，因此 JVM 可以保证任何时刻任何线程总能拿到该变量的最新值。


# AQS（AbstractQueuedSynchronizer)

AQS在java.util.concurrent.locks包下面。是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的 ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。


# 并发编程中一些问题

## 多线程就一定好吗？快吗？

并发编程的目的就是为了能提高程序的执行效率提高程序运行速度，但是并发编程并不总是能提高程序运行速度的，而且并发编程可能会遇到很多问题，比如：内存泄漏、上下文切换、死锁还有受限于硬件和软件的资源闲置问题。

多线程就是几乎同时执行多个线程（一个处理器在某一个时间点上永远都只能是一个线程！即使这个处理器是多核的，除非有多个处理器才能实现多个线程同时运行）。CPU通过给每个线程分配CPU时间片来实现伪同时运行，因为CPU时间片一般很短很短，所以给人一种同时运行的感觉。

## 上下文切换

当前任务在执行完CPU时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换会这个任务时，可以再加载这个任务的状态。任务从保存到再加载的过程就是一次上下文切换。

上下文切换通常是计算密集型的。也就是说，它需要相当可观的处理器时间，在每秒几十上百次的切换中，每次切换都需要纳秒量级的时间。所以，上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。 
Linux相比与其他操作系统（包括其他类 Unix 系统）有很多的优点，其中有一项就是，其上下文切换和模式切换的时间消耗非常少。

那么我们现在可能会考虑 ：如何减少上下文切换的次数呢？

## 减少上下文切换

上下文切换又分为2种：让步式上下文切换和抢占式上下文切换。前者是指执行线程主动释放CPU，与锁竞争严重程度成正比，可通过减少锁竞争和使用CAS算法来避免；后者是指线程因分配的时间片用尽而被迫放弃CPU或者被其他优先级更高的线程所抢占，一般由于线程数大于CPU可用核心数引起，可通过适当减少线程数和使用协程来避免。

总结一下：

- 减少锁的使用。因为多线程竞争锁时会引起上下文切换。
- 使用CAS算法。这种算法也是为了减少锁的使用。CAS算法是一种无锁算法。
- 减少线程的使用。人物很少的时候创建大量线程会导致大量线程都处于等待状态。
- 使用协程（微线程或者说是轻量级的线程，它占用的内存更少并且更灵活）。

我们上面提到了两个名词：“CAS算法” 和 “协程”。可能有些人不是很了解这俩东西，所以这里简单说一下。


## 避免死锁

在操作系统中，死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。

在线程中，如果两个线程同时等待对方释放锁也会产生死锁。

锁是一个好东西，但是使用不当就会造成死锁。一旦死锁产生程序就无法继续运行下去。所以如何避免死锁的产生，在我们使用并发编程时至关重要。根据《Java并发编程的艺术》有下面四种避免死锁的常见方法：

- 避免一个线程同时获得多个锁
- 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源
- 尝试使用定时锁，使用lock.tryLock(timeout)来替代使用内部锁机制
- 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况

## 解决资源限制

### 什么是资源限制

所谓资源限制就是我们在进行并发编程时，程序的运行速度受限于计算机硬件资源比如CPU,内存等等或软件资源比如软件的质量、性能等等。举个例子：如果说服务器的带宽只有2MB/s，某个资源的下载速度是1MB/s，系统启动10个线程下载该资源并不会导致下载速度编程10MB/s，所以在并发编程时，需要考虑这些资源的限制。硬件资源限制有：带宽的上传和下载速度、硬盘读写速度和CPU处理速度；软件资源限制有数据库的连接数、socket连接数、软件质量和性能等等。

### 资源限制引发的问题

在并发编程中，程序运行加快的原因是运行方式从串行运行变为并发运行，但是如果如果某段程序的并发执行由于资源限制仍然在串行执行的话，这时候程序的运行不仅不会加快，反而会更慢，因为可能增加了上下文切换和资源调度的时间。

### 如何解决资源限制的问题

对于硬件资源限制，可以考虑使用集群并行执行程序。既然单机的资源有限制，那么就让程序在多机上运行。比如使用Hadoop或者自己搭建服务器集群。

对于软件资源的限制，可以考虑使用资源池将资源复用。比如使用连接池将数据库和Socket复用，或者在调用对方webservice接口获取数据时，只建立一个连接。另外还可以考虑使用良好的开源软件。

### 在资源限制的情况下如何进行并发编程

根据不同的资源限制调整程序的并发度，比如下载文件程序依赖于两个资源-带宽和硬盘读写速度。有数据库操作时，设计数据库练连接数，如果SQL语句执行非常快，而线程的数量比数据库连接数大很多，则某些线程会被阻塞，等待数据库连接。

