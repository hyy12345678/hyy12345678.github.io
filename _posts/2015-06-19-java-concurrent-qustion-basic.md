---
layout: post
title:  "java并发总结－基础篇"
date:   2015-06-19 13:21:25
categories: Concurrency
tags: Concurrency
image: /assets/article_images/desktop.JPG
---

# 多线程

### 1. java中有几种方法可以实现一个线程？

继承Thread类，实现Runnable接口
创建一个线程的唯一方法是实例化java.lang.Thread类(或其子类)，并调用其start()方法

### 2. 如何停止一个正在运行的线程？

调用ThreadInstanceA.inerrupt()方法，这样当A线程在Thread的sleep，join方法，或者Object的wait方法的时候会直接抛出InerruptedException，捕捉后便可退出。

{% highlight java %}
public void shutdown() {
stop = true;
this.interrupt();
try {
this.join();
}
catch(InterruptedException ie) {}
}
{% endhighlight %}

停止一个线程的最佳方法是让它执行完毕，没有办法立即停止一个线程，但你可以控制何时或什么条件下让他执行完毕
通过条件变量控制线程的执行，线程内部检查变量状态，外部改变变量值可控制停止执行。为保证线程间的即时通信，需要使用volatile关键字或锁，确保读线程与写线程间变量状态一致。
下面给一个模板：

{% highlight java %}
public class BestPractice extends Thread {
private volatile boolean finished = false; // ① volatile条件变量
public void stopMe() {
finished = true; // ② 发出停止信号
}
@Override
public void run() {
while (!finished) { // ③ 检测条件变量
// do dirty work // ④业务代码
}
}
}
{% endhighlight %}

当④处的代码阻塞于wait()或sleep()时，线程不能立刻检测到条件变量。因此②处的代码最好向上边一样同时调用interrupt()方法。

从源码上看Join内部也是调用wait，那么他是如何做到“在线程B中调用了线程A的Join()方法，直到线程A执行完毕后，才会继续执行线程B“的呢？
wait()会释放对象上的锁的，并且等待获得锁的线程和调用wait的线程有一个本质差别，一旦一个线程调用wait方法，他就进入了等待集合中，当锁可获得时，线程不能立即解除阻塞。他维持阻塞状态一直到另一个线程调用同一个锁上的notifyAll/notify方法为止；在B中调用A.wait,这时a作为提供监视器的对象（锁的提供者），B为真正阻塞的线程（锁的使用者），所以线程B会一直停在执行a.join这一句代码的位置上，又因为没有其他的线程会调用a.notify结束阻塞，所以会一直等到a线程执行完毕，当一个线程结束时，他会立即释放所有他锁住对象上的锁，于是B线程从a.join下一行继续执行。若想打破这种机制，可以调用a.interrupt()，这时，线程b可以不必受刚才的约束

### 3. notify()和notifyAll()有什么区别？

X.notify()方法会唤醒在X对象上wait的其他线程中的一个（只能唤醒等待中的线程中的一个，具体唤醒哪一个线程，由JVM决定）；
X.notify()方法会唤醒所有 在X对象上wait的线程，但是由于要继续执行wait后面的代码，必须在sychronized块中获得相应X对象的锁，所以一旦一个线程获得锁后，其他的线程就会等待其他线程释放锁并自己获得锁后才能继续执行。
notify容易导致死锁，当仅剩一个线程处于活动状态并且也要转入wait的时候，如果调用notify唤醒其他阻塞的线程，因为只会唤醒一个线程，如果唤醒后的线程发现还是不满足条件，继续调用wait，有可能所有线程都阻塞了，造成死锁。

### 4. sleep()和 wait()有什么区别?

sleep不会让出锁，wait会让出锁。

### 5. 什么是Daemon线程？它有什么意义？

用户线程：就是我们平时创建的普通线程.
守护线程：主要是用来服务用户线程. 定期检查系统的状态等。
当线程只剩下守护线程的时候，JVM就会退出.但是如果还有其他的任意一个用户线程还在，JVM就不会退出.

### 6. java如何实现多线程之间的通讯和协作？

所谓的线程通信，指的是通过发送信号唤醒另一个线程，一般用在存在线程互斥的情况下。
使用上可以选用Object.wait,Object.notify/notifyall,配合sychronized关键字。也可以选用concurrent包中提供的，Lock(ReetrantLock)，Condition(Lock.newCondition())，在Lock.lock获得锁后通过Condition.await(),Condition.signal/signalAll实现线程间的协作

# 锁

### 7. 什么是可重入锁（ReentrantLock）？

concurrent包中提供的替代Object中的监视器的一个显示的锁对象。继承子Lock接口，用来保护从lock()方法，到unlock()的临界区域内的代码。配合条件对象（Condition）使用。condition通过获得reentrantlock得到对临界代码的访问权。

### 8. 当一个线程进入某个对象的一个synchronized的实例方法后，其它线程是否可进入此对象的其它方法？

一个对象的synchronize实例方法，就相当于在一个普通的实力方法中sychronized（this），会占有当前对象的锁，此时其他线程不可以访问这个对象另外的synchronize实例方法，但是可以访问其他非同步方法。

### 9. synchronized和java.util.concurrent.locks.Lock的异同？

基本作用相同，都是为了解决线程间互斥的问题。

隐式的锁和条件（synchronized，wait，notify）方式，虽然写起代码来比较简洁，但他也存在一些缺点：
- 你不能中断一个正在试图获得锁的线程。
- 试图获得锁时不能设定超时
- 每个锁只有一个条件，有时显得不够用（比如读写锁问题）

显示的锁和条件（ReetrantLock，Condition）解决了上面隐式锁的几个问题：
- lock方法不能被中断，所以提供了trylock方法进行锁测试和超时
- 通过调用带超时的tryLock方法，如果在线程等待获取一个锁时被中断，将抛出一个InterruptException异常，以此打破死锁。
- 也可以调用lockInterruptibly方法，相当于一个超时设为无限的trylock方法。
- await方法也提供超时设置

读写锁问题：
在读多写少的情况下保证数据的ACID的同时提高系统的处理能力，只有“读－读”可以不加锁。“读－写”，“写－读”，“写－写”还是需要加锁。

concurrent包中提供ReentrantReadWriteLock类，帮助创建读锁和写锁。
readLock（）得到一个可被多个读操作共享的读锁，但他会排斥所有的写操作。
writeLock（）得到一个写锁，他会排斥其他所有的读操作和写操作。
另外显示锁还提供了公平等特性。

### 10. 乐观锁和悲观锁的理解及如何实现，有哪些实现方式？

悲观锁认为每次拿数据的时候数据都有可能被其他人修改，所以每次拿数据都需要加锁，这样别的人想拿数据就会被block，直到自己释放锁。实现可以利用前面介绍的Object.wait/notify配合synchronized，或者concurrent包提供的Lock.lock配合Condition.await/signal。

乐观锁认为拿数据的时候不会被人修改，所以在拿去数据的时候不加锁，只是在回写数据的时候确认一下这期间是否有人更新这个数据，可以使用版本号等机制。适合读多写少的场合，可以提高程序的吞吐量。如果写比较多就容易发生数据不一致导致retry，反而效率低。实现方式可以用CAS（check-and-set）方式，或者使用volitate变量。

Java demo:
{% highlight java %}
AtomicInteger atom = new AtomicInteger(1);
boolean r = atom.compareAndSet(1, 2);
{% endhighlight %}

# 并发框架

### 11. SynchronizedMap和ConcurrentHashMap有什么区别？

SynchronizedMap时HashMap的加锁版本，效率不高，而且Map接口本身提供keySet，values，entrySet，这些迭代器提供的是视图而不是副本，所以当一个线程正在迭代Map中的元素时，另一个线程可能正在修改其中的元素。此时，在迭代元素时就可能会抛出 ConcurrentModificationException异常。

ConcurrentHashMap实现ConcurrentMap接口，ConcurrentHashMap采用分段加锁的设计，不同的线程不会造成阻塞。只有在size等操作时才需要锁住整个表，大大提高了效率。

### 12. CopyOnWriteArrayList可以用于什么应用场景？

适合于读很多写很少的场景，比如一些人员信息，参数配置等会经常被读到，但是很少修改的情况。

# 线程安全

### 13. 什么叫线程安全？servlet是线程安全吗?

> 引用概念：如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。
我理解的是线程安全问题都是由全局变量及静态变量引起的，是要么多线程不对同一数据的访问，要么采用机制确保对同一数据访问的安全性进行保障（比如对数据的操作是原子操作，锁机制，包括悲观锁和乐观锁）。

servlet是多线程的，同时一个servlet实现类只会有一个实例对象，也就是它是Singleton的,所以多个线程是可能会访问同一个servlet实例对象的，但是这并不能说明servlet就是线程不安全的，servlet是否线程安全是由它的实现来决定的，如果它内部的属性或方法会被多个线程改变，它就是线程不安全的，反之，就是线程安全的。

### 14. 同步有几种实现方法？

加锁，join，阻塞队列

### 15. volatile有什么用？能否用一句话说明下volatile的应用场景？

> Java语言规范第三版中对volatile的定义如下： java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致的更新，线程应该确保通过排他锁单独获得这个变量。Java语言提供了volatile，在某些情况下比锁更加方便。如果一个字段被声明成volatile，java线程内存模型确保所有线程看到这个变量的值是一致的。

volatile关键字对一个实例的域的同步访问提供了一个免锁（lock-free）机制。如果把域声明为volatile，那么编译器和虚拟机就知道该域可能会被另一个线程并发更新。
对象内需要同步的域值少，使用锁显得浪费和繁琐场景，这时使用volatile。一些并发容器（ConcurrentHashMap，etc）的实现内使用了volatile。利用jvm对volatile承诺的happen-before原则，完成不加锁的并发读，写。


### 16. 请说明下java的内存模型及其工作流程。

内存模型：
Java内存模型在JVM specification, Java SE 7 Edition, and mainly in the chapters “2.5 Runtime Data Areas” and “2.6 Frames”中有详细的说明。对象和类的数据存储在3个不同的内存区域：堆（heap space）、方法区（method area）、本地区（native area）。
堆内存存放对象以及数组的数据，方法区存放类的信息（包括类名、方法、字段）、静态变量、编译器编译后的代码，本地区包含线程栈、本地方法栈等存放线程。方法区有时被称为持久代（PermGen），这个主要是因为早期的hotspot使用了方法区来实现持久代，当然这么做也带来一些问题，本质上两者不是一个概念。
所有的对象在实例化后的整个运行周期内，都被存放在堆内存中。堆内存又被划分成不同的部分：伊甸区(Eden)，幸存者区域(Survivor Sapce)，老年代（Old Generation Space）。
方法的执行都是伴随着线程的。原始类型的本地变量以及引用都存放在线程栈中。而引用关联的对象比如String，都存在在堆中。
堆内存同样被划分成了多个区域：

包含伊甸（Eden）和幸存者区域(Survivor Sapce)的新生代（Young generation）
老年代（Old Generation）
不同区域的存放的对象拥有不同的生命周期：

工作流程：
新建（New）或者短期的对象存放在Eden区域；
幸存的或者中期的对象将会从Eden区域拷贝到Survivor区域；
始终存在或者长期的对象将会从Survivor拷贝到Old Generation；
生命周期来划分对象，可以消耗很短的时间和CPU做一次小的垃圾回收（GC）。原因是跟C一样，内存的释放（通过销毁对象）通过2种不同的GC实现：Young GC、Full GC。
为了检查所有的对象是否能够被销毁，Young GC会标记不能销毁的对象，经过多次标记后，对象将会被移动到老年代中。

### 17. 为什么代码会重排序？

JMM允许编译器、运行库、处理器或缓存可以有特权定时地在变量的指定内存位置存入或取出变量值。例如，编译器为了优化一个循环索引变量，可能会选择把它存储到一个寄存器中，或者缓存会延迟到一个更适合的时间，才把一个新的变量值存入主存。所有的这些优化是为了帮助实现更高的性能。
“重新排序”这个术语用于描述几种对内存操作的类型：

当编译器不会改变程序的语义时，作为一种优化它可以随意地重新排序某些指令。
在某些情况下，可以允许处理器以颠倒的次序执行一些操作。
通常允许缓存以与程序写入变量时所不相同的次序把变量存入主存。

### 参考列表：


- 《JAVA2核心技术－卷2：高级特性》
- 《JAVA并发编程实践》
-  http://ibruce.info/2013/12/17/how-many-ways-to-create-a-thread-in-java/