### Java中有哪几种方式来创建线程执行任务

**1.继承Thread类**

​	重写run()方法，占用继承

**2.实现Runnable接口**

​	实现run()方法，更常用，可使用匿名内部类或Lambda表达式方式

**3.实现Callable接口**

​	实现call()方法，使用Thread+FutureTask配合，可获取异步执行任务的结果

**4.利用线程池来创建线程**

​	实现Callable接口或者Runnable接口，由ExecutorService来创建线程

### 为什么不建议使用Executors来创建线程池

1.FixedThreadPool

2.SingleThreadExecutor

总结：队列LinkedBlockingQueue是一个无界阻塞队列，任务会不断添加到队列中，内存不断占用，会导致OOM

### 线程池有哪几种状态？每种状态分别表示什么

![image-20250705111221852](F:\LearnDoc\1-架构师-20250406\Java八股文\Java.assets\image-20250705111221852.png)

1.Running-运行状态：线程池新建或调用execute()方法后，处于运行状态，能够接受新的任务。

2.Shutdown-关闭状态：调用shutdown()方法后，线程池的状态会变为Shutdown,此时线程池不再接收新的任务，但会执行已提交的等待任务队列中的任务。

3.Stop-停止状态：调用shutdownNow()方法后，此时线程池的状态会变为Stop，此时线程池不再接受新的任务，并且会中断正在处理中的任务。

4.Tidying-整理状态：中间状态不做任何处理。

5.Terminated：线程池内部的所有线程都已经终止时，线程池进入Terminated状态。

### Sychronized和ReentrantLock有哪些不同点

| sychronized                      | ReentrantLock                                                |
| -------------------------------- | ------------------------------------------------------------ |
| Java中的一个关键字，JVM层面的锁  | JDK提供的一个类，API层面的锁                                 |
| 自动加锁与释放锁                 | 需要手动加锁与释放锁                                         |
| 不可获取当前线程是否上锁         | 可获取当前线程是否上锁，isHeldByCurrentThread                |
| 非公平锁                         | 公平锁或非公平锁                                             |
| 不可中断                         | 可中断：1.调用设置超时方法tryLock(long timeout, timeUnit unit); 2.调用lockInterruptibly()放大到代码块中，然后调用interrupt()方法可以中断 |
| 锁的是对象，锁信息保存在对象头中 | int类型的state标识来标识锁的状态                             |
| 底层有锁升级过程                 | 没有锁升级过程                                               |

### ThreadLocal有哪些应用场景？它底层是如何实现的？

薪资：10k~25k		岗位：中高级开发工程师

1.ThreadLocal是Java中所提供的线程本地存储机制，可以利用该机制将数据缓存在某个线程内部，该线程可以在任意时刻、任意方法中获取缓存的数据

2.ThreadLocal底层是通过ThreadLocalMap来实现的，每个Thread对象（注意不是ThreadLocal对象）中都存在一个ThreadLocalMap，Map的key为ThreadLocal对象，Map的value为需要缓存的值。

3.如果在线程池中使用ThreadLocal会造成内存泄漏，因为当ThreadLocal对象使用完之后，应该要把设置的key，value，也就是Entry对象进行回收，但线程池中的线程不会回收，而线程对象是通过强引用指向ThreadLocalMap，ThreadMap也是通过强引用指向Entry对象，线程不被回收，Entry对象也就不会被回收，从而出现内存泄漏，解决办法是，在使用了ThreadLocal对象之后，手动调用ThreadLocal的remove方法，手动清除Entry对象。

4.当一个共享变量是共享的，但是需要每个线程互不影响，相互隔离，就可以使用ThreadLocal

​	a.跨层传递信息的时候，每个方法都声明一个参数很麻烦，A\B\C\D 3个类相互传递，每个方法都声明参数降低了维护性，可以用一个ThreadLocal共享变量，在A存值，BCD都可以获取。

​	b.隔离线程，存储一些线程不安全的工具对象，如（SimpleDateFormat）

​	c.Spring中的事务管理器就是使用的ThreadLocal

​	d.Springmvc的HttpSession、HttpServletRequest、HttpServletResponse都放在ThreadLocal，因为servlet是单例的，而Springmvc允许在Controller类中通过@Autowired配置request、response以及requestcontext等实例对象。底层就是搭配ThreadLocal才实现线程安全。

### ReentrantLocal分为公平锁和非公平锁，那底层分别是如何实现的？

薪资：15k~25k		岗位：高级开发工程师

首先不管是公平锁和非公平锁，它们的底层实现都是会使用AQS来进行排队，它们的区别在于线程在使用lock()方法加锁时：

​	1.如果是公平锁，会先检查AQS队列中是否存在线程在排队，如果有线程在排队，则当前线程也进行排队。

​	2.如果是非公平锁，则不会去检查是否有线程在排队，而是直接竞争锁。

另外，不管是公平锁还是非公平锁，一旦没竞争到锁，都会进行排队，当锁释放时，都是唤醒排在最前面的线程，所以非公平锁只是体现在了线程加锁阶段，而没有体现在线程被唤醒阶段，ReetrantLocal是可重入锁，不管公平锁还是非公平锁都是可重入的。

#### Sychronized的锁升级过程是怎样的？

薪资：10k~25k		岗位：初中高级开发工程师

1.偏向锁：在锁对象的对象头中记录一下当前获取到该锁的线程ID，该线程下次如果又来获取该锁就可以直接获取到了，也就是支持锁重入。

2.轻量级锁：当两个或以上线程交替获取锁，但并没有在对象上并发的获取锁时，偏向锁升级为轻量级锁。在此阶段，线程采取CAS的自旋方式尝试获取锁，避免阻塞线程造成的cpu在用户态和内核态间转换的消耗。

3.两个或以上线程并发的在同一个对象上进行同步时，为了避免无用自旋消耗cpu，轻量级锁会升级成重量级锁。

4.自旋锁：自旋锁就是线程在获取锁的过程中，不会去阻塞线程，也就是无所谓唤醒线程，阻塞和唤醒这两个步骤都是需要操作系统去进行的，比较消耗时间，自旋锁是线程通过CAS获取预...

#### Tomcat中为什么要使用自定义类加载器？

薪资：10k~20k		岗位：中级开发工程师

一个Tomcat中可以部署多个应用，而每个应用中都存在很多类，并且各个应用中的类是对立的，全类名是可以相同的，比如一个订单系统中可能存在com.xushu.User类，一个库存系统中可能也存在com.xishu.User类，一个Tomcat,不管内部部署了多少应用，Tomcat启动之后就是一个Java进程，也就是一个JVM，所以如果Tomcat中只存在一个类加载器，比如默认的AppClassLoader，那么就只能加载一个com.xushu.User类，这是由问题的，而在Tomcat中，会为部署的每个应用都生成一个类加载器实例，名字叫做WebAppClassLoader，这样Tomcat中每个应用就可以使用自己的类加载器去加载自己的类，从而达到应用之间的类隔离，不出现冲突。另外Tomcat还可以利用自定义加载器实现了热加载功能。