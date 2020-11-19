---
layout: post
title: Java 并发编程
date: 2020-11-19
Author: shope
categories: 
tags: [Java 并发编程, BlockQueue, ThreadPool]
comments: true
---

### 并发编程特性：

1、可见性--volatile解决可见性问题（确保变化及时可见）

2、原子性--加锁

3、有序性

volatile：Java并发中轻量级的锁机制

### 多线程下即使不加volatile也会可见的原因：

1、因为在OS执行程序时，如果的CPU运行时间片结束了，会保存当前运行状态，然后CPU就会切换给其他线程去使用，等到再次拿到时间片后，OS会从继续恢复执行，此时变量已经被改变了。

2、可能是缓存行（cacheline）更新，会从内存中去读取新的值。

3、根据**时间局限性**或者**空间局限性**缓存原理，读取到新的值。

volatile禁止重排优化：

volatile可以禁止指令重排优化，避免多线程环境下出现乱序执行的现象。

### 逃逸分析

分析一个对象动态的作用域，如果一个对象通过方法参数传递到其他方法中，称为方法逃逸；赋值的类变量被其他线程访问到，称为线程逃逸。

防止逃逸：

1、栈上分配	2、同步取消	3、标量替换

### 锁的粗化

为了确保共享数据在共同作用域时，只在实际作用作用域中进行同步，这样是为了使得同步操作数量尽可能变小，如果存在锁的竞争，那么锁在线程就能尽快获得锁。但是如果一系列的连续操作都对同一个对象反复加锁和解锁，JVM就会对这样的代码进行优化成只需要加一次锁。

```java
synchronized (obj) {
  // 语句 1
}
synchronized (obj) {
  // 语句 2
}


//	优化后变成
synchronized (obj) {
  // 语句 1
  // 语句 2
}
```

### 锁的消除

锁削除是指虚拟机即时编译器在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁，对其进行消除。

```java
public void do(){
    Object obj = new Object();
    synchronized(obj){
		//	TODO
    }
}
```

### AQS

Abstract Queued Synchronizer--抽象队列同步器

特性：

- 阻塞等待队列
- 共享/独占
- 公平/非公平
- 可重入
- 可中断

**原理：自旋、LockSuport、CAS、queue队列。**

当线程抢锁的时候会先判断当前线程是不是已经获得锁，如果已经获得过锁，那么**state+1**（将state+1表示当前线程再次获取到锁，释放锁则将state-1，直到state=0表示锁完全被释放），进入到阻塞代码块中，否则让线程进入到阻塞队列中，为确保入队成功，在入队时操作是进行自旋----CAS，直到成功添加到**queue**队列中，如果**Node**是队列中的**第一个Node**，在入队后还要对锁进行一次抢夺：

​	如果获取锁成功：出队，将head设置为当前Node并将thread设置为空----表示head节点；

​	如果获取锁失败：1、修改head的状态（**waitStatus**） 2、使用LockSuport阻塞线程，并判断线程是否是被唤醒的(在非公平模式下，唤醒后可能获取不到锁)

​	修改**head**的**waitStatus**：0->-1，

#### queue队列

是一个双向链表队列，queue的节点信息由Node存储。

```java
class Node {
	Node pre;	// 上一个节点
	Node next;	// 下一个节点
	int waitStatus;
     	/* Status field, taking on only the values:
         *   SIGNAL: 	-1	可被唤醒
         *   CANCELLED:  1	代表出现异常，中断引起的
         *   CONDITION: -2	条件等待
         *   PROPAGATE: -3	传播
         *   0:  			初始状态
         */
        static final int PROPAGATE = -3;
	Thread thread;	// 记录节点中的线程引用
}
```



### ReentrantLock

在java.util.concurrent.locks;

ReentrantLock是一种居于AQS框架的应用实现，是JDK中的一种线程并发访问的同步手段，他的功能类似于synchronized是一种互斥锁，可以保证线程安全。而且它具有比synchronized更多的特性，比如它支持手动加锁与解锁，支持加锁的公平性。

![ReentrantLock类图](image_for_notes\ReentrantLock_Class_Relation.png)

```java
//	使用ReentrantLock进行同步
ReentrantLock lock = new ReentrantLock(false); // false 为非公平锁，true为公平锁
lock.lock();	//	加锁
lock.unlock();	//	解锁
```

**ReentrantLock**内部定义了一个**Sync内部类**，该类继承**AbstractQueuedSynchronizer**，对该抽象类的部分方法做了实现：定义了**两个子类**：

​		**1、FairSync 公平锁的实现**

​		**2、NonFairSync 非公平锁的实现**

这些类都继承自Sync，也就是间接继承了AbstractQueueSybchronizer，所以ReentrantLock同时具备公平与非公平特性。

<font color="red">*</font>**interrupt()**方法会将线程的中断信号清楚，并返回中断状态，这也就意味着线程会进入运行状态。

### BlockQueue

BlockingQueue，是java.util.concurrent 包提供的用于解决**并发生产者 - 消费者问题**的最有用的类，它的特性是在任意时刻只有一个线程可以进行take或者put操作，并且 BlockingQueue提供了超时return null的机制，在许多生产场景里都可以看到这个工具的 身影。

#### 常见的4种阻塞队列

1. ArrayBlockingQueue 由数组支持的有界队列 
2. LinkedBlockingQueue 由链接节点支持的可选有界队列 
3. PriorityBlockingQueue 由优先级堆支持的无界优先级队列 
4. DelayQueue 由优先级堆支持的、基于时间的调度队列

### CountDownLatch & Semaphore

#### Semaphore 

字面意思是信号量的意思，它的作用是控制访问特定资源的线程数目，底层依赖**AQS**的状态State，是在生产当中比较常用的一个工具类。

```java
public Semaphore(int permits)
public Semaphore(int permits, boolean fair)
//	permits 表示许可线程的数量
//	fair	如果为true，下次执行等待最久的线程。如果为false，线程间会进行抢夺，谁抢到执行谁
    
    
//	表示阻塞并获取许可
public void acquire() throws InterruptedException
//	表示释放许可
public void release()
```

应用：Hystrix里限流就是基于信号量方式；

#### CountDownLatch

​	CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器的值为0时，表示所有线程执行完任务，然后在闭锁上等待的线程就可以恢复执行任务。

```java
private CountDownLatch countDownLatch = new CountDownLatch(2);

countDownLatch.countDown(); //	每调用一次 计数器的值减1 ，直到为0

countDownLatch.await();	//	等待2个子线程执行完毕，否则一直阻塞

```

#### CyclicBarrier 

​	栅栏屏障，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程 到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。 

​	CyclicBarrier默认的构造方法是**CyclicBarrier（int parties）**，其参数表示屏障拦截的线程数量，每个线程调用await方法告CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。

------

### ThreadPool

**使⽤线程池主要有以下三个原因：** 

1. 创建/销毁线程需要消耗系统资源，线程池可以复⽤已创建的线程。 
2. 控制并发的数量。并发数量过多，可能会导致资源消耗过多，从⽽造成服务器 崩溃。
3. 可以对线程做统⼀管理。

#### ThreadPoolExecutor

```java
// 五个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
 int maximumPoolSize,
 long keepAliveTime,
 TimeUnit unit,
 BlockingQueue<Runnable> workQueue)
// 六个参数的构造函数-1
public ThreadPoolExecutor(int corePoolSize,
 int maximumPoolSize,
 long keepAliveTime,
 TimeUnit unit,
 BlockingQueue<Runnable> workQueue,
 ThreadFactory threadFactory)
// 六个参数的构造函数-2
public ThreadPoolExecutor(int corePoolSize,
 int maximumPoolSize,
 long keepAliveTime,
 TimeUnit unit,
 BlockingQueue<Runnable> workQueue,
 RejectedExecutionHandler handler)
// 七个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
 int maximumPoolSize,
 long keepAliveTime,
 TimeUnit unit,
 BlockingQueue<Runnable> workQueue,
 ThreadFactory threadFactory,
 RejectedExecutionHandler handler)
```

- **int corePoolSize**：该线程池中核⼼线程数最⼤值

  线程池中有两类线程，核心线程与非核心线程。核心线程**默认情况**下回一直存在与线程中，不回被销毁，而非核心线程如果长时间闲置，就会被销毁。

- **int maximumPoolSize**：该线程池中线程总数最⼤值 。
  $$
  maximumPoolSize=核⼼线程数量 + ⾮核⼼线程数量
  $$

- **long keepAliveTime**：⾮核⼼线程闲置超时时⻓。

  ⾮核⼼线程如果处于闲置状态超过该值，就会被销毁。如果设置 **allowCoreThreadTimeOut(<font color="red">true</font>)**，则会也作⽤于核⼼线程。

- **RejectedExecutionHandler handler** 拒绝处理策略，线程数量⼤于最⼤线程数就会采⽤拒绝处理策略，

  四种拒绝处理的策略为 ：

  1. ThreadPoolExecutor.**AbortPolicy**：默认拒绝处理策略，丢弃任务并抛RejectedExecutionException异常。 

  2. ThreadPoolExecutor.**DiscardPolicy**：丢弃新来的任务，但是不抛出异常。 

  3. ThreadPoolExecutor.**DiscardOldestPolicy**：丢弃队列头部（最旧的） 的任务，然后重新尝试执⾏程序（如果再次失败，重复此过程）。

  4. ThreadPoolExecutor.**CallerRunsPolicy**：由调⽤线程处理该任务。

     

#### ScheduleThreadPoolExecutor

**定时线程池类的类结构图**

![ScheduledExecutorService_Class_Relationship.jpg](https://i.loli.net/2020/11/20/2QoOsk7ZEhapnRc.png)

接收SchduledFutureTask类型的任务，是线程池调度任务的最小单位，有三种提交任务的方式： 

- schedule 提交一次性任务
- cheduledAtFixedRate  提交周期性任务（固定速率提交，延迟时间一到就提交）
- scheduledWithFixedDelay 提交周期性任务（固定延迟提交，等待上一次任务执行完成后在延迟提交）

##### 采用DelayQueue存储等待的任务

1. DelayQueue内部封装了一个PriorityQueue，它会根据time的先后时间排序，若 time相同则根据sequenceNumber排序； 
2. DelayQueue也是一个无界队列；

##### SchduledFutureTask SchduledFutureTask接收的参数(成员变量)： 

1. private long time：任务开始的时间 
2. private final long sequenceNumber：任务的序号 
3. private final long period：任务执行的时间间隔

##### 工作线程的执行过程： 

- 工作线程会从DelayQueue取已经到期的任务去执行； 
- 执行结束后重新设置任务的到期时间，再次放回DelayQueue;

**ScheduledThreadPoolExecutor**会把待执行的任务放到工作队列**DelayQueue**中， DelayQueue封装了一个**PriorityQueue**，PriorityQueue会对队列中的 **ScheduledFutureTask**进行排序，具体的排序算法实现如下：

1. 首先按照**time**排序，time小的排在前面，time大的排在后面； 
2. 如果time相同，按照**sequenceNumber**排序，sequenceNumber小的排在前 面，sequenceNumber大的排在后面，换句话说，如果两个task的执行时间相同， 优先执行先提交的task。


```java
 public int compareTo(Delayed other) {
            if (other == this) // compare zero if same object
                return 0;
            if (other instanceof ScheduledFutureTask) {
                ScheduledFutureTask<?> x = (ScheduledFutureTask<?>)other;
                long diff = time - x.time;
                if (diff < 0)
                    return -1;
                else if (diff > 0)
                    return 1;
                else if (sequenceNumber < x.sequenceNumber)
                    return -1;
                else
                    return 1;
            }
            long diff = getDelay(NANOSECONDS) - other.getDelay(NANOSECONDS);va
            return (diff < 0) ? -1 : (diff > 0) ? 1 : 0;
        }
```

**ScheduledFutureTask之run方法实现 run方法是调度task的核心，task的执行实际上是run方法的执行。**

```java
/**
  * Overrides FutureTask version so as to reset/requeue if periodic.
  */
public void run() {
    boolean periodic = isPeriodic();
    //	如果当前线程池不执行任务，则取消
    if (!canRunInCurrentRunState(periodic))
        cancel(false);
    //	如果不需要周期执行，则直接执行run方法，最后结束
    else if (!periodic)
        ScheduledFutureTask.super.run();
    //	如果需要周期执行，则在执行完任务以后，设置下一次执行时间
    else if (ScheduledFutureTask.super.runAndReset()) {
        //	计算并设置下次执行该任务的时间
        setNextRunTime();
        //	重复执行任务
        reExecutePeriodic(outerTask);
    }
}
```

