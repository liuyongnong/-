# 多线程 #

## 线程的状态转换 ##
---
![](https://i.imgur.com/rTZzrjW.png)

新建(new)
---
创建后尚未启动

可运行（Runnable）
---
可能正在运行，也可能正在等待时间片  
包括了操作系统中线程状态的Running 和Ready

阻塞（Blocking）
---
等待获取一个排它锁，如果其线程释放了锁就会结束此状态

无限期等待（Waiting）
---
无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

调用 `Thread.sleep()` 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。

调用 `Object.wait()` 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。

睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 `Thread.sleep()` 和 `Object.wait()` 等方法进入。

| 进入方法 | 退出方法 |
| ---- | ---- |
| Thread.sleep()方法 | 时间结束 |
|设置了timeout参数的object.wait方法|时间结束/Object.notify() / Object.notifyAll()|
|设置了 Timeout 参数的 Thread.join() 方法|时间结束 / 被调用的线程执行完毕|
|LockSupport.parkNanos() 方法|----|
|LockSupport.parkUntil() 方法|----|

死亡（Terminated）
---
可以是线程结束任务后自己结束，或者因为异常而退出


## 使用线程 ##
---
使用线程的方法  

- 实现Runnable接口
- 继承Thread类
- 实现Callable接口
- 使用线程池

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。


实现Runnable接口
---
需要实现`run()`方法，通过`Thread.start()`来启动线程
`public class MyRunnable implements Runnable {
    public void run() {
        // ...
    }
}`  
`public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}`  


实现Callable接口
---
与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。  

`public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}`  
`public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}`
  
继承Thread类
---
同样也要实现run方法，因为thread类实现了Runnable接口
当调用start方法启动一个线程的时候，虚拟机会将线程放入到就绪队列中等待调度，当线程被调度的时候会执行`run()`方法


继承和实现的比较
---
实现接口会比继承类好一点，因为java不能实现多继承，但可以实现多个接口  
继承整个类的开销太大

## 基础线程机制 ##
---  

Executor(执行器)
---
![](https://i.imgur.com/g3JJu2q.png)
Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。
  
主要有三种Excutor  
  
- CachedThreadPool:一个任务创建一个线程
- FixedThreadPool:所有任务只能执行固定大小的线程
- SingleThreadPool:相当于大小为一的FixedThreadPool

Daemon(守护线程)
---
守护线程是程序在后台运行时提供服务的线程，不属于程序中不可或缺的部分，当所有的	非守护线程结束时，程序也就终止，同时杀死所有的守护线程  
使用 setDaemon() 方法将一个线程设置为守护线程。  
`public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}`  


sleep()
---
Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒  
sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。  
`public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}`  

yield()
---
Thread.yield()的作用是：暂停当前正在执行的线程对象（及放弃当前拥有的cpu资源），并执行其他线程  
yield()的作用是让当前的运行线程回到可运行状态，以允许具有相同优先级的线程获得运行机会。因此，使用yield的目的是让相同优先级的线程之间能适当的轮转。但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。  
结论：yield()从未导致线程转到等待/睡眠/阻塞状态。在大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果  
`public void run() {
    Thread.yield();
}`  
 

## 中断 ##
---
一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。  

InterruptedException
---
通过调用一个线程的interrupt()来中断线程，如果该线程属于阻塞，限期等待或者无限期等待状态，那么就会抛出一个interruptedExecption,从而提前结束该线程，但是不能提前结束I/O阻塞和synchronized锁阻塞  
  
Executor 的中断操作
---
调用Exection的`shotdown()`方法会等待线程都执行完之后在关闭，但如果调用的是`showdownNow()`相当于每个线程调用`interrupt()`方法  


## 互斥同步 ##
---
java提供了两种锁机制来控制对各线程对资源的互斥访问，第一个是JVM实现的synchronized,而另一个是JDK实现的ReentrantLock.   
synchronized
---
1. 同步一个代码块  
 `public void func() {
    synchronized (this) {
        // ...
    }
}`  
他只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。
1. 同步一个方法，和同步代码快一样，只能作用于同一个对象  
1. 同步一个类：作用于整个类，也就是说两个线程调用同一个类上的不同对象的这种同步语句，也会进行同步。
2. 同步一个静态方法：作用于整个类  
  
ReentrantLock
---
ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。其作用跟synchornized类似但比他更强大，还具有监听对象功能  

synchornized和ReentrantLock的比较
---


1. 锁的实现：synchornized是jvm实现的，RenntrantLock是JDK实现的
2. 性能：新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。
3. 等待可中断：当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。ReentrantLock 可中断，而 synchronized 不行
4. 公平锁：公平锁是指多个线程在等待同一个锁的时候，必须按照申请锁的时间顺序来依次获得锁。synchornized的锁是非公平的，RenntrantLock默认是不公平的，但也可以是公平的。
5. 绑定多个条件：RenntrantLock可以同时绑定多个Condition对象  
  
## 线程之间的协作 ##
---
当多个线程一起去工作解决某个问题时，如果某个部分必须在其他 部分之前完成，那么就要对线程进行协调  
join()
---
在线程中使用另一个线程的Join()方法，会将当前线程挂起，而不是忙等待，直到线程结束  
  
wait(),notify(),notifyAll()
---
调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。  
  
await() signal() signalAll()
---
java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。  

## 线程不安全示例 ## 
---
如果多个线程对同一个共享数据进行访问而不采取同步操作的话，那么操作的结果是不一致的  


## java内存模型 ##
---
java内存模型试图屏蔽各种硬件和操作系统之间的内存访问差异，以实现让java在平台下都能达到内存一致的访问效果  
主内存和工作内存
---
处理器上的寄存器的读写的速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。  
加入高速缓存带来了一个新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题  
![](https://i.imgur.com/Dhe4j9D.png)
  
所有的变量都储存在占内存中，每个线程还有自己的工作内存，工作内存储存在高速缓存或寄存器中，保证了该线程使用的变量主题的主内存副本拷贝  
线程只能直接操作工作内存的变量，不同内存之间的变量值传递需要通过主内存来完成  
  
内存间相互操作
---  
![](https://i.imgur.com/T8TmAZw.png)  

- read:把一个变量的值从主内存传输到工作内存中
- load:在read之后执行，把read得到的值放入工作内存的变量副本中
- use：把工作内存中的一个变量的值传递给执行引擎
- assign:把一个从执行引擎接收到的值付给工作内存的变量
- store:把工作内存的一个变量的值传送到内存中
- write:在store之后执行，把store得到的值放入到主内存的变量中
- lock:作用于主内存的变量
- unlock  
  
 内存的三大模型
---
1. 原子性：  

Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。但是 Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据（long，double）的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。

1. 可见性  

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有有三种实现可见性的方式：

volatile
synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。
对前面的线程不安全示例中的 cnt 变量使用 volatile 修饰，不能解决线程不安全问题，因为 volatile 并不能保证操作的原子性。  
1. 有序性：  
有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。  
  
先行发生原则
---  
#### 单一线程原则 ####
在一个线程内，在程序前面的操作优于后面的操作  

#### 管锁定规则 ####
一个 unlock 操作先行发生于后面对同一个锁的 lock 操作  
#### voliatile ####
   
对于一个volatile变量的写操作先行发生于后面对这个变量的读操作。  
#### 线程启动规则####
线程对象的start（）方法调用先行于发生于此线程的每一个动作
#### 线程加入规则 
线程对象的结束先行发生join()方法的返回  
![](https://i.imgur.com/HP2m2I0.png)  
#### 线程中断规则 ####
对线程interrupt方法的调用先行发生于中断线程的代码检测到中断事件的发生以通过 interrupted() 方法检测到是否有中断发生。
#### 对象终结规则 ####
一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。
#### 传递性 ####
如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。 	
## 线程安全 ##
---
多个线程不管以何种方式访问某个类，并在在主调代码中不需要进行同步，都能表现正确的行为。

线程安全有以下几种实现方式：
不可变
---
不可变的对象一定是安全的，不需要再采取任何线程安全保护措施，只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。  
#### 不可变类型 ####
- final关键字修饰的基本数据类型
- String
- 枚举类型
- Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。 

对于集合类型，可以使用 Collections.unmodifiableXXX() 方法来获取一个不可变的集合。  
互斥同步
---
synchornized和RenntrantLock  

非阻塞同步
---
互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。   
## 锁优化 ##
---
####自旋锁 ####
互斥同步进入阻塞状态的开销都很大，应尽量避免。在许多应用中，共享数据的锁定状态只会持续很短一段时间。自旋锁的思想是让线程请求共享数据时先忙循环（自旋）一段时间，在这段时间内能不能获得锁，能的话就避免机内阻塞状态。  
#### 锁消除 ####
锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。
#### 锁粗化 ####
如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

上一节的示例代码中连续的 append() 方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。







  

