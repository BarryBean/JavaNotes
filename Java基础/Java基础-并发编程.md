# 1. 基础知识
## 1.1 优缺点
优点：
- 充分利用多核CPU性能，提高运行速度；
- 拆分业务，提高系统处理能力。

缺点：
- 引发的一系列问题，如内存泄漏，上下文切换，线程安全，死锁等等。

## 1.2 三要素
答：原子性、可见性、有序性，也是线程安全的体现。
- 原子性：一组操作要么都执行，要么都不执行。(针对共享变量)
- 可见性：一个线程对共享变量的修改，另一个线程是可见的。
- 有序性：程序执行的顺序按代码先后顺序执行。

## 1.3 线程安全
问题：
- 线程切换->原子性
- 缓存->可见性
- 编译优化->有序性

解决方案：
- Atomic原子类，synchronized，Lock->原子性
- synchronized，volatile，Lock->可见性
- Happens-Before->有序性

## 1.4 并发和并行
答：并发指同一时间段，多个任务按时间片轮转执行。并行指单位时间内，多个任务被多个处理器同时执行。

eg. 8-9点，我洗脸刷牙吃饭->并发，我左手洗脸右手刷牙->并行。


## 1.5 多线程
答：多线程是指一个进程，并发执行多个线程，每个线程有自己的功能处理不同的任务。

优势：
- 使用**CPU轮询时间片**模式，提高资源利用率。

劣势：
- 线程也需要占用内存，线程越大占用内存越大；
- 降低程序执行速度，因为存在线程上下文切换；
- 对共享资源的访问，带来线程死锁等安全问题。


## 1.6 进程通信
- 管道：半双工，只能父子或兄弟进程。
- FIFO：任何进程都能通信，但是慢。
- 消息队列：容量收到系统限制。
- 信号量：只能用来同步，不能传递复杂消息。
- 共享内存。
- socket：同一主机和不同主机的socket通信。


# 2. 进程、线程和协程
## 2.1 区别

||进程 | 线程
|---|---|---
|根本区别|进程是资源分配的基本单位 |线程是CPU调度执行的最小允许单位
|资源开销|进程切换要保存当前CPU环境和建立新的CPU环境 | 线程切换只需要保存虚拟机栈和程序计数器
|包含关系 |一个进程可以有多个线程 |线程是进程的一部分
|内存分配 |进程间的资源和地址相互独立 |同一进程的线程共享本进程的资源和地址
|执行过程 |每个进程都是独立运行 |线程必须依赖应用程序
|影响关系|一个进程崩溃不会影响其他进程|一个线程崩溃容易整个进程挂掉
|系统资源|进程拥有堆和方法区/元空间|线程一般没有系统资源，但有必不可少的ThreadLocal


**协程**是用户态执行的轻量级线程(在一个线程执行)，调度都由用户控制，可以随时中断执行别的子程序再返回接着执行。项目中，协程去读取或者写入本地的文件，这样就是串行。

## 2.2 守护线程和用户线程
- 守护线程：运行在后台，为前台线程服务。如GC线程。
- 用户线程：运行在前台，执行具体的任务。如main()。




# 3. 创建线程的方法
答：java中有四种方法实现线程。实现Runnable接口，实现Callable接口，继承Thread类，Executor创建线程池。

建议采用实现接口的方式，因为继承整个Thread类开销过大且Java不支持多重继承，但支持多接口继承。

## 3.1 实现Runnable接口
继承Runnable接口，**实现run方法，通过Thread调用start()启动线程，无返回值，无法捕获异常处理。**
```java
public class MyRunnable implements Runnable {
 public void run() {
 // ...
 }
}
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```
## 3.2 实现Callable接口
**以Callable做参数创建FutureTask类，通过Thread调用start()启动线程，有返回值(被Future获取)，能捕获异常处理。**
```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
   thread.start();
   System.out.println(ft.get());
}
```
## 3.3 继承Thread类
通过start()启动，因为Thread类也是实现Runnable接口，所以需要重写run()。
```java
public class MyThread extends Thread {
 public void run() {
 // ...
 }
}
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```
## 3.4 线程池
Executors提供方法，实现ExecutorService接口。
```java
public class MyRunnable implements Runnable {
 public void run() {
 // ...
 }
}
public static void main(String[] args) {
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    MyRunnable runnableTest = new MyRunnable();
    for (int i = 0; i < 5; i++) {
        executorService.execute(runnableTest);
    }
    executorService.shutdown();
}
```
## 3.5 run()和start()
- start()用于启动线程让线程进入就绪状态，只能调用一次，会自动执行run()；
- run()用于执行线程内部代码，类似main()下的普通方法，可重复调用，依赖于线程start()。



# 4. 线程的生命周期和调度

## 4.1 生命周期状态
答：线程状态包括 新建、运行、阻塞等待和消亡。阻塞等待分为Blocked、Waiting和Time Waiting。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020823060391.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

参照源码中Thread的Enum定义。
- New新建：创建后尚未调用start方法
- Runnable可运行：就绪状态，可能是正在运行或者正在等待CPU资源
- Blocked阻塞：线程进入同步块中，需要申请一个同步锁而进行的等待
- Waiting无限期等待：调用了Object.wait()或Object.notify()或LockSupport.park()方法，无限期等待其他线程来唤醒
- Time Waiting有限期等待：调用Thread.sleep(x)等方法，区别是等待时间是明确的
- Terminated消亡：线程执行结束或产生异常提前结束
 

## 4.2 线程调度

### 4.2.1 调度模型
答：两种模型。
- 分时调度：让所有线程轮流获得CPU使用权，平均分配时间片。
- 抢占式调度：让优先级高的线程抢占CPU，直到有更高优先级线程进入或线程任务运行。JVM默认。

### 4.2.2 相关方法
- sleep()：Thread类的方法，让线程进入有限期等待休眠，之后**自动苏醒**。休眠**不释放锁**。常用于**暂停执行**。
- wait()：Object类的方法，与synchronized一起使用，线程进入有限期或无限期等待，被notify方法调用才能解除阻塞，只有重新占用互斥锁才能进入Runnable。休眠**释放互斥锁**。常用于**线程间通信交互**。wait(long timeout)超时后也会自动苏醒。
- join()：当前线程调用，其他线程全部停止，等待当前线程执行结束再执行。**Stop the world**
- yield()：让线程**放弃当前获得的CPU**，使线程仍处于Runnable，随时可以再获得CPU。只给相同优先级或更高优先级的线程机会。
- notify()：唤醒一个线程。
- notifyAll()：唤醒所有线程，参与锁竞争，失败就留在池中等待下次唤醒。

### 4.2.3 停止运行线程方法
- 退出标志，让线程正常退出；
- stop()/suspend()强制终止；
- interrupt()中断线程，但仅是把逻辑状态设置为中断，不会停止线程，需要后续处理。


### 4.2.4 interrupte()，interrupted()和isInterrupted()
- interrupt()：中断线程。调用该方法后，线程状态被置为中断，不会停止线程，抛出中断异常。
- interrupted()：静态方法，检查当前中断状态，并清除中断状态。如果一个线程被中断了，第一次调用 interrupted 则返回 true，第二次和后面的就返回 false 了。
- isInterrupted()：查看当前线程的中断状态，不清除状态。



### 4.2.5 线程共享数据
- 临界区：单进程的多线程同步。用户态。
- 互斥量：单进程和多进程的多线程访问同步。内核态。
- 事件：多进程的多线程间触发事件实现同步。内核态。
- 信号量：多个线程同时访问公共区域数据。内核态。

### 4.2.6 线程同步方式
答：总共7种。
1. 同步方法。用synchronized修饰的方法。
2. 同步代码块。用synchronized修饰的语句块。
3. 用volatile修饰变量。
4. 可重入锁。
5. ThreadLocal管理变量副本。
6. 阻塞队列。
7. 原子类。

### 4.2.7 线程类的构造方法、静态块是被哪个线程调用的
答：线程类的构造方法、静态块是被new这个线程类所在的线程所调用，run()里的代码才是被线程自身所调用的。

eg.Thread2 中 new 了Thread1，main 函数中 new 了 Thread2，那么：
- Thread2 的构造方法、静态块是 main 线程调用的，Thread2 的 run()方法是Thread2 自己调用的；
- Thread1 的构造方法、静态块是 Thread2 调用的，Thread1 的 run()方法是Thread1 自己调用的。



# 5. 线程故障

## 5.1 线程死锁
答：定义为多个线程之间相互请求对方占用的资源而被无限期阻塞。

### 5.1.1 四个必要条件
答：OS的基础知识：
- 资源互斥：一个资源任意时刻只能被一个线程使用；
- 请求和保持：一个线程因请求资源而阻塞时，对已获得的资源保持不放；
- 不剥夺：线程已获得的资源，再未使用完之前，不能强行剥夺；
- 循环等待：若干线程间形成头尾相接的循环等待资源状态。

### 5.1.2 避免死锁的方法
答：破坏死锁产生的四个条件中的任一，常用算法是银行家算法。
- 破坏资源互斥：做不到；
- 破坏请求和保持：**一次性申请全部资源**；
- 破坏不剥夺：占用部分资源的线程再申请其他资源时，若申请不到就**主动释放其占有的资源**；
- 破环循环等待：**锁排序法**，指定获取锁的顺序(也可认为指定获取资源的顺序)，比如：只有获得A锁的线程才能获得B锁，只有AB锁都获得的才能操作资源C。。

### 5.1.3 死锁代码
```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//资源 1
    private static Object resource2 = new Object();//资源 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "线程 2").start();
    }
}
```

## 5.2 线程锁死
答：等待线程由于唤醒条件无法成立或其他线程无法唤醒这个线程，而一直处于非运行状态。

### 5.2.1 分类
- 信号丢失锁死：没有对应的线程来唤醒等待线程，导致一直等待。
- 嵌套监视器锁死：由于嵌套锁导致等待线程永远无法被唤醒的故障。比如，线程只释放了内层锁Y.wait()，没有释放外层锁X；但通知线程必须获得外层锁X，才能通过Y.notify()唤醒，这就出现嵌套等待现象。

## 5.3 线程活锁
答：线程一直处于运行状态，但其执行的任务没有任何进展。比如，线程一直在请求其需要的资源，但无法申请成功。(占着茅坑不拉屎)

## 5.4 线程饥饿
答：线程一直无法获得其所需的资源致使任务无法运行的情况。

## 5.5 活性故障间转换
答：线程饥饿发生时，如果线程处于Runnable状态，就转变为活锁。线程死锁也是线程饥饿。




# 6. 并发理论
## 6.1 JVM
### 6.1.1 GC
答：GC是为了识别和丢弃不再使用的对象来释放资源。详细过程和相关见JVM部分。

### 6.1.2 finalize()
答：GC在回收某对象时，会调用该对象的finalize()，让对象处理生前的最后事情或者挣扎一下。

- 自我挣扎。覆写了finalize()重新引用到GC root上。只能调用一次。但只保证被调用，不保证方法内任务执行。
- 做最后的资源回收。

## 6.2 重排序
答：执行程序时，为了最优性能，处理器和编译器会对指令进行重排序。
- 单线程下不会改变运行结果，但会破坏多线程的执行结果。
- 存在数据依赖关系的不允许重排序。

## 6.3 happens-before和as-if-serial
- 都为了在不改变执行结果的前提下，尽可能提高程序并行程度；
- happens-before保证多线程按指定顺序执行，as-if-serial保证单线程按指定顺序执行。


# 7. 关键字
## 7.1 synchronized
### 7.1.1 理解
答：synchronized是Java的一个关键字，用来控制线程同步，保证修饰的方法或代码块在任意时刻只有一个线程执行。在JDK1.6后进行了大量优化，引入了锁升级机制，减少了锁的开销。

synchronized可用来修饰实例方法(锁当前实例)、静态方法(锁当前类的class对象)和代码块(锁当前括号内对象)。

### 7.1.2 对象头
1. 实例对象在堆中，分为对象头、实例变量和填充数据。
2. synchronized锁对象存储在对象头中，由Mark Word 和 Class Metadata Address组成。
3. Mark Word存储对象的hashCode、锁信息、分代年龄等。

### 7.1.2 底层原理
答：分为修饰代码块和修饰方法。

**修饰代码块：** 显式同步
- 进入时，执行 monitorenter，将计数器+1，释放锁 monitorexit 时，计数器-1；
- 每条 enter 指令都有对应的 monitorexit，多出的一个 monitorexit 是为了异常时释放 monitor，避免死锁。
- 当一个线程判断到计数器为0时，则当前锁空闲，可以占用；反之，当前线程进入等待状态

**修饰方法：** 隐式同步
使用 ACC_SYNCHRONIZED 标识指明方法是一个同步方法，JVM从而执行相应的同步调用(和代码块类似的 monitor管程)


### 7.1.3 锁优化
锁优化主要是synchronized的优化。

Java早期版本中，monitor 依赖于os的 Mutex Lock互斥锁，每次调用都要切换用户态和核心态，效率低下。


1. **自旋锁**：让线程在请求一个共享数据锁时忙循环（自旋）一段时间，若这段时间内能获得锁，则避免进入阻塞状态。总结：**请求锁时被占用先忙循环**
2. **锁粗化**：JVM探测到一组操作都对同一个对象加锁，就会把加锁范围扩展到整个操作的外部（粗化），以避免频繁加锁引起性能损耗。
3. **偏向锁**：主要思想是**先来就是你的，对你不用同步，有竞争就释放锁。**对象头有一个变量专门存储当前线程的id，之后再来线程与这个id比较，相同就不用再进行同步验证，不同就释放锁，锁升级为轻量锁。

### 7.1.4 锁升级
答：目的是为了降低锁的性能消耗。具体流程是:
1. 最开始阶段时无锁状态。
1. 第一次访问时，JVM设置为偏向锁，把threadid设置为当前线程id，再次进入时判断id是否一致；
2. 不一致，升级为轻量锁，进入自旋；
3. 自旋一定次数仍未获取资源，升级为重量锁。


### 7.1.5 等待唤醒机制
答：notify,notifyAll和wait方法，必须拿到当前对象的monitor监视器对象，synchronized可以获取监视器，所以等待唤醒方法必须要synchronized的方法或代码块。


## 7.2 Volatile
### 7.2.1 理解
答：Volatile的主要作用是**保证变量的可见性和有序性，不能保证原子性**。可见性是通过，每次把修改后的值立即更新到主存，其他线程需要时再到主存读取。有序性是通过在适当位置插入内存屏障实现。当多个线程共享一组状态变量，可以替代锁。

### 7.2.2 底层原理
- 生成汇编代码时，Lock前缀会将处理器缓存写回内存；
- 写回内存使其他CPU的缓存失效；
- CPU发现本地缓存失效时，会从内存重读该变量数据，从而实现获得新值。

### 7.2.3 synchronized和volatile的区别
答：比较总结如下：
- volatile关键字是**轻量级**的锁，性能比synchronized好(1.6优化后不一定)；
- volatile只能**修饰变量**，synchronized能修饰方法和代码块；
- volatile保证数据**可见性和有序性**，不保证原子性；synchronized都保证。


### 7.2.4 JMM内存屏障
答：JMM(Java内存模型)通过在适当位置插入内存屏障阻止重排序。

1. volatile写：在前和后插入屏障(先禁上面普通写，再禁止下面可能的volatile读写)；
2. volatile读：在后插入两个屏障(禁止普通读写+volatile读)。

- StoreStore屏障：禁止上面的普通写和下面的volatile写重排序；
- StoreLoad屏障：禁止上面的volatile写和下面的volatile读/写重排序；
- LoadLoad屏障：禁止上面的volatile读重排序和下面的普通读操作；
- LoadStore屏障：禁止上面的volatile读重排序和下面的普通写操作。
![volatile写](A973C9457A9D4E1BA78DD855F744C48E)
![volatile读](61AE59742E444F3185ED083B03E58A66)



# 8. Lock
## 8.1 Lock接口
答：总体上说Lock接口是synchronized的升级版，支持非公平和公平锁，提供了轮询、定时、中断、多条件的锁操作，更加灵活。

## 8.2 ReentrantLock
答：ReentrantLock可重入锁是Lock接口的一个实现类。可重入锁就是自己可以再次获取自己的内部锁。同一线程每次获取锁，锁的计数器++，计数器为0时再释放锁。

### 8.2.1 synchronized和ReentrantLock的区别
答：总结为：
1. 二者都是可重入锁。
2. synchronized是关键字依赖于JVM，ReentrantLock是类依赖于API接口。
3. synchronized修饰类和方法，ReentrantLock只适用代码块。
4. ReentrantLock比synchronized增加了一些高级功能。
- **等待可中断**。正在等待的线程可以选择放弃等待，执行其他任务。
- ReentrantLock支持**公平和非公平调度**。synchronized只支持非公平锁。
- 支持**选择性通知**。synchronized相当于整个Lock只有一个Condition，所有线程都注册在一个上面，notifyAll()通知所有等待状态线程，效率不高。ReentrantLock的线程对象能注册在指定的Condition中，signalAll()只会唤醒该Condition实例中的等待线程。

### 8.2.2 ReentrantReadWriteLock
答：ReentrantLock在多个线程读数据时也会重复加锁，降低性能，所以诞生了ReentrantReadWriteLock读写重入锁。

- ReentrantReadWriteLock实现了读写分离，读时共享，写时独占，读和读不会互斥。
- 实现锁降级。写-读-释放写-降级为读锁。



## 8.3 AQS
答：AQS(AbstractQueuedSynchronizer)是用来**构建锁和同步器**的框架。

### 8.3.1 底层原理
1. 若被请求的**共享资源空闲**，则将当前请求资源的线程设置为有效的工作线程，**分配后将共享资源设为锁定状态**。
2. 若被请求的**共享资源被占用**，则需要一套线程阻塞等待和唤醒锁分配的机制，这个机制AQS通过CLH队列实现，即**将暂时获取不到锁的线程封装为一个结点加入到队列中**。
3. CLH队列是一个虚拟双向队列，仅存在结点间的关联。AQS就是把线程封装成队列的node。
4. 内部使用volatile int变量的state标识同步状态，FIFO的排队策略，CAS实现值的修改。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200208234023804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

### 8.3.2 资源共享方式
答：两种。
1. Exclusive(独占)：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：
- 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
- 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
2. Share(共享)：多个线程可同时执行，如Semaphore、CountDownLatch、 CyclicBarrier、ReadWriteLock等。


## 8.4 并发组件
1. Semaphore(信号量)：**允许多个线程同时访问某个资源**，通过参数可以设置访问数，synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源。
2. CountDownLatch：**倒计时器，让线程倒计时为0再执行**，用来实现线程等待其余线程完成某一特定操作后，再开始执行。**内部维护一个计数器**，为0则唤醒等待线程，不为0则暂停。
3. CyclicBarrier：**循环栅栏，到地一拦，人齐再走**。让线程到达同步点时被栅栏阻塞，直到最后一个线程到达栅栏，才让所有被拦截的线程继续执行。**内部维护一个参与方个数的计数器**，每个线程到达同步点调用await()方法使count-1，当判断到是最后一个参与方时，调用singalAll唤醒所有线程。


# 9. 并发容器
## 9.1 ConcurrentHashMap
详细见Java-三大集合。

1.8前用分段锁实现，1.8后用数组+链表+红黑树，synchronized和CAS控制并发。

## 9.2 ThreadLocal
### 9.2.1 理解

答：ThreadLocal是**为每个线程提供独立的变量本地副本**，通过get/set方法独立获取/改变自己的副本值，避免线程安全问题。(理解为有100个学生给了100只笔，互不影响)

### 9.2.2 底层原理
答：每个线程内都有一个类HashMap的对象，称为**ThreadLocalMap**，存放以ThreadLocal为key的键值对，get/set/remove基于此实现。

### 9.2.3 内存泄漏
答：key是弱引用，value是强引用。gc时value不会被清理，长期不清除造成内存泄漏。

### 9.2.4 解决方案
- 每次用完就remove()清除数据。
- 针对key为null的entry，先查有没有哈希冲突，没有就调用cleanSomeSlots检测脏数据；有就向后环形查找，过程中有脏数据就replaceStaleEntry。

### 9.2.5 key使用弱引用原因
答：反过来想。
- key是强引用，业务处理key置null，始终可达，JVM不会自动gc。
- key是软引用，只能等到空间不足才gc，但有些线程比如守护线程是不会关闭的，所以等同不能gc。
- 总之，弱引用即使会出现内存泄漏问题，但在生命周期内只要保证对脏数据处理，就能保证安全。

## 9.3 BlockingQueue
答：BlockingQueue阻塞队列。
- 当队列为空时，获取元素的线程等待队列非空；
- 当队列为满时，存储元素的线程等待队列可用。
- 常用于生产者-消费者模型，socket数据读取和解析。


# 10. 线程池
## 10.1 理解
答：池化思想有利于降低资源消耗，节省创建资源的时间，提高线程的可管理性。线程池，顾名思义，提前创建若干线程放在池中，需要时直接获取不用创建，使用完毕放回池中。

## 10.2 状态
- Running：接收新任务，处理等待队列中的任务；
- Shutdown：不接受新任务，处理等待队列中的任务；
- Stop：不接受新任务，不处理等待队列中的任务，中断正在执行的任务；
- Tidying：所有任务都销毁了，workCount为0，钩子引用terminated()；
- Terminated：terminated()执行后的状态。

## 10.3 execute()和submit()区别
答：总结如下：
- execute()用于提交**不需要返回值**的任务，所以无法判断任务是否被线程池执行成功；Runnable。
- submit()用于提交需要返回值的任务，线程池**返回Future类型对象**，通过get()获得返回值。Callable+Runnable。

## 10.4 ThreadPoolExecutor
答：使用ThreadPoolExecutor创建线程池，客户端调用submit(Runnable task)提交任务。
```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class ThreadPoolExecutorDemo {
    private static final int CORE_POOL_SIZE = 5;
    private static final int MAX_POOL_SIZE = 10;
    private static final int QUEUE_CAPACITY = 100;
    private static final Long KEEP_ALIVE_TIME = 1L;
    public static void main(String[] args) {

        //使用阿里巴巴推荐的创建线程池的方式
        //通过ThreadPoolExecutor构造函数自定义参数创建
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                CORE_POOL_SIZE,
                MAX_POOL_SIZE,
                KEEP_ALIVE_TIME,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(QUEUE_CAPACITY),
                new ThreadPoolExecutor.CallerRunsPolicy());

        for (int i = 0; i < 10; i++) {
            //创建WorkerThread对象（WorkerThread类实现了Runnable 接口）
            Runnable worker = new MyRunnable("" + i);
            //执行Runnable
            executor.execute(worker);
        }
        //终止线程池
        executor.shutdown();
        while (!executor.isTerminated()) {
        }
        System.out.println("Finished all threads");
    }
}
```
具体参数为：
- corePoolSize：核心线程数，即最小可同时运行的线程数量
- maximumPoolSize：最大线程数，即最大可同时运行的线程数量
- keepAliveTime ：线程空闲但是保持不被回收的时间
- unit：时间单位
- workQueue：阻塞队列，存储线程的队列
- threadFactory：创建线程的工厂
- handler：拒绝策略

推荐配置：
- corePoolSize: 核心线程数为 5。
- maximumPoolSize ：最大线程数 10
- keepAliveTime : 等待时间为 1L。
- unit: 等待时间的单位为 TimeUnit.SECONDS。
- workQueue：任务队列为 ArrayBlockingQueue，并且容量为 100;
- handler:拒绝策略为 CallerRunsPolicy。

## 10.5 排队策略
答：向线程池提交任务时，需要遵循排队策略。
- 若运行的线程 < corePoolSize，Executor首选添加线程，不排队；
- 若运行的线程 >= corePoolSize，且队列未满，Executor首选将请求加入队列，不加新线程；
- 若队列已满，创建新线程，若超出maximumPoolSize ，根据拒绝策略处理。

## 10.6 拒绝/饱和策略
答：线程达到max，队列也放满时，使用拒绝策略。
- AbortPolicy：中断，抛出异常 RejectedExecutionException来拒绝新任务的处理。
- CallerRunsPolicy：调用自己所在线程运行任务。会降低对于新任务提交速度，影响程序的整体性能。
- DiscardPolicy：直接丢弃，不处理。
- DiscardOldestPolicy：舍弃最旧任务，丢弃最早的未处理的任务请求。

## 10.7 常见线程池类型
答：四种。
- newCachedThreadPool()：**可缓存**线程池，核心线程池大小为0，最大线程池大小无限，**来一个创建一个线程**。
- newFixedThreadPool()：**固定大小**的线程池。
- newScheduledThreadPool：**定时**线程池，周期执行或者定时执行。
- newSingleThreadExecutor()：**单线程化**的线程池，保证所有任务按指定顺序执行，如FIFO、LRU。不会发生并发执行。

## 10.8 常见阻塞队列
答：三种。
1. ArrayBlockingQueue：基于**预先分配的数组实现**的有界阻塞队列。
- 优点：put和take操作不会增加GC的负担；
- 缺点：put和take**操作使用同一个锁**，可能导致锁争用。
- 适合在生产者线程和消费者线程之间的**并发程序较低**的情况下使用。
2. LinkedBlockingQueue：基于**链表实现**的是无界阻塞队列。(上限是Integer.MAX_VALUE)
- 优点：put和take**操作使用两个显式锁**；
- 缺点：增加GC的负担，因为空间是动态分配的。
- 适合在生产者线程和消费者线程之间的**并发程序较高**的情况下使用。
3. SynchronousQueue：不存储元素的有界阻塞队列。

- 生产者线程生产一个产品之后，会等待消费者线程来取走这个产品，才会接着生产下一个产品。(**put必须有take**)
- 适合在生产者线程和消费者线程之间的处理能力相差不大的情况下使用。




# 11. Atomic原子类
答：简单来说，原子类就是具有原子操作特征的类，即这个类中的操作不可中断。原子类都方法JUC(java.util.concurrent)并发包.atomic下，基本都是**CAS + volatile**实现。
## 11.1 CAS

### 11.1.1 乐观锁
答：CAS乐观锁假设所有线程访问资源不会出现冲突情况，在提交数据时检查完整性，如果出现冲突使用CAS处理。

### 11.1.2 过程
答：总结：**拿期望值和原本值比较，相同就更新为新值**，不同就自旋。

- CAS包含，V：内存中的实际值，A：预期值，B：新值。
- V=A，说明值没有更改，A为当前最新值，可以进行B赋值给V。
- V!=A，说明值已经改变，A不为当前最新值，不能赋值，直接返回V。

### 11.1.3 问题

1. **ABA**：如果一个值初次读取为A，而后被改成B，后来又被改回A，那CAS会误认为其没有改变过。解决方案是**增加变量值的版本号或者引入boolean标志改没改变**，AtomicStampedReference类。
2. **自旋时间过长**：CAS是非阻塞同步，不会挂起线程，而是自旋一段时间进行尝试。自旋时间过长会对性能造成很大的消耗。


## 11.2 synchronized和CAS的区别
答：Synchronized是互斥同步，存在线程阻塞和唤醒锁的性能问题。CAS是非阻塞同步，进行自旋后尝试。

## 11.3 以AtomicInteger为例
常用方法有：
```java
public final int get() //获取当前的值
public final int getAndSet(int newValue)//获取当前的值，并设置新的值
public final int getAndIncrement()//获取当前的值，并自增
public final int getAndDecrement() //获取当前的值，并自减
public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
public final void lazySet(int newValue)//最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
```
