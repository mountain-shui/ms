  ## 3.Java多线程 

  ### 1.开启线程的方法与各自的优劣

**创建线程的三种方法 **

  * 继承Thread类，重写run()方法
  * 实现Runnable接口，重写run()方法
  * 实现Callable接口，重写call()方法

**优劣：**

继承Thread类

​	优点：编程简单，访问当前线程直接使用this关键字

​	缺点：继承了Thread类就不能继承其他类了

实现Runnable接口

​	优点：可以让多个线程共享一个资源，实现代码和资源的分离，实现这个接口还可以继承其他的类。

​	缺点：访问当前线程需要使用Thread.currentThread()方法。

Runnable和Callable的区别

* Runnable使用run()方法，Callable使用call()方法
* Runnable有返回值，Callable没有返回值
* Runnable不能返回异常，Callable可以返回异常

start和run的区别：

​		运行start表示创建一个新线程，调用run()方法，在main线程中运行run()方法不会创建线程，就相当于普通		的方法调用。

 ### 2.线程池的状态

![08000847-0a9caed4d6914485b2f56048c668251a](images\08000847-0a9caed4d6914485b2f56048c668251a.jpg)

**1.RUNNING**：这是最正常的状态，接受新的任务，处理等待队列中的任务。线程池的初始化状态是RUNNING。线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0。

**2.SHUTDOWN**：不接受新的任务提交，但是会继续处理等待队列中的任务。调用线程池的shutdown()方法时，线程池由RUNNING -> SHUTDOWN。

**3.STOP**：不接受新的任务提交，不再处理等待队列中的任务，中断正在执行任务的线程。调用线程池的shutdownNow()方法时，线程池由(RUNNING or SHUTDOWN ) -> STOP。

**4.TIDYING**：所有的任务都销毁了，workCount 为 0，线程池的状态在转换为 TIDYING 状态时，会执行钩子方法 terminated()。因为terminated()在ThreadPoolExecutor类中是空的，所以用户想在线程池变为TIDYING时进行相应的处理；可以通过重载terminated()函数来实现。 

当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。

当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。

**5.TERMINATED**：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。

  ### 3.线程池的参数 

**最重要的三个参数**

* corePoolSize:线程池正常运行时，存在的最小的线程数量，即使这些线程空闲，也不会被销毁。
* maximunPoolSize:最大线程池数量，一个任务提交到线程池，当前没有线程空闲，就会将这个刚提交的线程放到工作队列的队尾，等待执行，如果队列已经满了，就在线程池中创建一个新的线程，当然不能无限创建，这个参数就是用来限定可以线程池中最多乐意创建多少线程的。
* keepAliveTime:当超出corePoolSize数量之外的线程空闲，并且经过keepAliveTime还没有新的任务，就会自动销毁。

**其他参数**

* unit:keepAliveTime的计时单位。
* workQueue:新任务被提交后，会先进入到此工作队列中，任务调度时再从队列中取出任务
* threadFactory:线程的创建工厂。
* handler:拒绝策略，当工作队列中的任务已经达到最大限制，并且线程池的线程数量也达到了最大数量，就会启动拒绝策略

对于workQueue，jdk提供了四种工作队列：

* ArrayBlockingQueue:有限数组阻塞队列，FIFO队列，有新任务进入时，先放到队尾，如果队列已满，就创建一个新的线程，如果线程已经达到maximunPoolSize就会执行拒绝策略。
* ②LinkedBlockingQuene:有限的链表阻塞队列，FIFO，最大队列长度为integer.MAX，当达到corePoolSize时就会将新任务放到队列，并不会创建新的线程，相当于maximunPoolSize不起作用。
* SynchronousQueue:一个不缓存任务的阻塞队列，新任务不会被缓存，有任务直接会被执行，当线程达到最大线程数时，直接执行拒绝策略。
* PriorityBlockingQueue:具有优先级的无界阻塞队列，按照优先级进行调度。

handler有四种：

* CallRunsPolicy:直接执行被拒绝者的run方法
* AbortPolicy:丢弃任务并且抛出异常。
* DiscardPolicy:直接丢弃，什么都不做。
* DiscardOldPolicy:丢弃最早加入队列的任务，把这个任务放进去。



  ### 4.线程池新建线程的流程 

![68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d372f2545352539422542452545382541372541332545372542412542462545372541382538422545362542312541302545352](images\68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d372f2545352539422542452545382541372541332545372542412542462545372541382538422545362542312541302545352.png)

### 5.保证线程安全的几种方法 

 **1.Synchronized**

线程同步,表示当多个线程同时使用的是一个对象的时候,sync只让一个运行,其他的必须等待

    多个线程在同时运行的时候如果访问的不是不同对象那么他们就是线程安全的,否则就使用sync,来保证线程的同步运行

sync几种方式
1 class 锁
2 方法锁
3 代码块锁

**2.ReentrantLock**

他实现了Lock接口, lock接口是jdk1.5的时候出来的,他实现结果跟sync是一个道理,不过他是finally来释放锁的

    sync是jvm层面,lock是类层面,sync锁状态不可以判断,Lock的锁状态可以判断,sync不可中断, lock可以中断也有公平锁

ReentrantLock 总共实现两种方式,公平锁和非公平锁

**3.Atomic**

原子性

**4.Wait和Notify**

消息通知

## 多个线程操作volatile变量的过程 

 JVM中有一个内存区域是jvm虚拟机栈，每一个线程运行时都有一个线程栈，线程栈保存了线程运行时候变量值信息。当线程访问某一个对象值的时候，首先通过对象的引用找到对应在堆内存的变量的值，

然后把堆内存变量的具体值load到线程本地内存中，建立一个变量副本，之后线程就不再和对象在堆内存变量值有任何关系，而是直接修改副本变量的值，

在修改完之后的某一个时刻（线程退出之前），自动把线程变量副本的值回写到对象在堆中变量。这样在堆中的对象的值就产生变化了。

下面一幅图描述对变量一次写操作的过程。

 

[![java volatile1](https://images.cnblogs.com/cnblogs_com/aigongsi/201204/201204011757235219.jpg)](http://images.cnblogs.com/cnblogs_com/aigongsi/201204/201204011757234696.jpg)

 

 

（1）read and load 从主存复制变量到当前工作内存
（2）use and assign 执行代码，改变共享变量值 
（3）store and write 用工作内存数据刷新主存相关内容

这一些操作并不是原子性，也就是 在read load之后，如果主内存count变量发生修改之后，线程工作内存中的值由于已经加载，不会产生对应的变化，所以计算出来的结果会和预期不一样

对于volatile修饰的变量，**jvm虚拟机只是保证从主内存加载到线程工作内存的值是最新的**

例如假如线程1，线程2 在进行read,load 操作中，发现主内存中count的值都是5，那么都会加载这个最新的值

在线程1堆count进行修改之后，会write到主内存中，主内存中的count变量就会变为6

线程2由于已经进行read,load操作，在进行运算之后，也会更新主内存count的变量值为6

但是，本应该是count的值进行两次递增＋1,要递增到8，可是显示的结果却是6

导致两个线程及时用volatile关键字修改之后，还是会存在并发的情况。volatile修饰的变量，并不能控制并发，只是做到了可见。