  ## 3.Java多线程 

### ThreadLocal原理？内存泄漏问题？使用场景？ 

**1.ThreadLocal 是什么？**

ThreadLocal 是一个本地线程副本变量工具类。主要用于将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，适用于各个线程不共享变量值的操作。

**2.ThreadLocal 工作原理是什么？**

ThreadLocal 原理：每个线程的内部都维护了一个 ThreadLocalMap，它是一个 Map（key,value）数据格式，key 是一个弱引用，也就是 ThreadLocal 本身，而 value 存的是线程变量的值。

也就是说 ThreadLocal 本身并不存储线程的变量值，它只是一个工具，用来维护线程内部的 Map，帮助存和取变量。

数据结构，如下图所示：

![img](https://pic2.zhimg.com/80/v2-aaba364d9af11719667ffc9ab6b64dad_720w.jpg)

## 

**内存泄漏**

ThreadLocalMap中的key是弱引用，而value是强引用。

**注意弱引用的对象如果被其他的强引用所引用，GC不会回收。**

所以，如果ThreadLocal没有被外部强引用的情况下，在垃圾回收时，key会被清空掉，而value不会。ThreadLocal中就会出现key为null的Entry。假如我们不采取措施，value永远不会被GC回收，这个时候就可能会产生内存泄漏问题。
ThreadLocalMap实现中已经考虑了这种情况，在调用set()、get()、remove()方法时会处理掉key为null的记录。使用ThreadLocal时，最好手动调用remove()方法。

内存泄露的问题不是因为强引用或者弱引用，不管是什么样的引用方式，只要当前线程的引用存在，这个entry就不会被GC。

内存泄漏的两个必要条件是：

* 栈中thread local引用没断开。
* 栈中当前线程没有结束。

**使用场景**

实际开发中我们真正使用`ThreadLocal`的场景还是比较少的，大多数使用都是在框架里面。

最常见的使用场景的话就是用它来解决数据库连接、`Session`管理等保证每一个线程中使用的数据库连接是同一个。

还有一个用的比较多的场景就是用来解决`SimpleDateFormat`解决线程不安全的问题，不过现在`java8`提供了`DateTimeFormatter`它是线程安全的，感兴趣的同学可以去看看。

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

## 乐观锁与悲观锁？使用场景？乐观锁如何实现？ 

1.悲观锁（一般都是通过锁机制来实现的）

（1）每次去拿数据都会认为别人会修改，所以每次拿数据的时候都会上锁。比如：行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。再比如Java里面的同步原语synchronized关键字的实现也是悲观锁。

2.乐观锁

（1）每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，如果发生冲突了，则返回用户的错误信息，让用户决定如何去做。（适用于多读的类型，并发大的情况。一般基于数据版本号实现）

（2）冲突检测和数据更新(版本号机制实现)

在数据表中加上一个数据版本号version字段，表示数据被修改的次数，数据被修改时version值会加1，当更新数据时，刚才读到的version值和数据库中的version值相等才可以更新。

update table set x=x+1, version=version+1 where id=#{id} and version=#{version};

（3）CAS（compare and swap）实现

多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新值，其他的线程失败，但不会被挂起，而是被告知这次竞争失败，并可以再次尝试。

CAS 操作原理:

CAS 操作中包含三个操作数 —— 需要读写的内存位置（V）、进行比较的预期原值（A）和拟写入的新值(B)。如果内存位置V的值与预期原值A相匹配，那么处理器会自动将该位置值更新为新值B。否则处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。
**乐观锁及悲观锁的应用场景**

1.什么时候使用悲观锁？

**写入频繁使用悲观锁**，如果出现大量的读取操作，每次读取的时候都会进行加锁，这样会增加大量的锁的开销，降低了系统的吞吐量。

一旦通过悲观锁锁定一个资源，那么其他需要操作该资源的使用方，只能等待直到锁被释放，好处在于可以减少并发，但是当并发量非常大的时候，由于锁消耗资源，并且可能锁定时间过长，容易导致系统性能下降，资源消耗严重。

2.什么时候使用乐观锁？

**读取频繁使用乐观锁**，一般乐观锁只用在高并发、多读少写的场景。比如，GIT,SVN,CVS等代码版本控制管理器。

例如：A、B，同时从SVN服务器上下载了hello.java文件，当A完成提交后，此时B再提交，那么会报版本冲突，此时需要B进行版本处理合并后，再提交到服务器。这其实就是乐观锁的实现全过程。如果此时使用的是悲观锁，那么意味者所有程序员都必须一个一个等待操作提交完，才能访问文件，这是难以接受的。

 ## 8.static修饰的作用（一直讲到在jvm的存储） 

static可以修饰成员变量，方法和代码块。

static修饰成员变量：该变量所有的线程都可以访问到，在创建类对象的时候，已经为这个变量分配了内存。如果在多线程中要使用成员变量，需要考虑线程安全的问题，可以配合volatile使用。

static修饰方法：表示该方法是静态的方法，该方法可以通过类名.方法名的方式调用，注意这个方法只能调用类内的静态方法或者静态变量。

static修饰代码块：当加载类的时候就会运行这个代码块，表示静态代码块只会被运行一次，静态代码块主要是完成类加载的各种初始化动作。

 ## 9.多线程相关，让三个线程排序获取锁，讲了Condition接口,问了voilate，讲到防止指令重新排序，内存屏障怎么插入，cpu总线嗅探机制

```java
public class abc_Lock {
    private static Lock lock = new ReentrantLock();
    private static Condition A = lock.newCondition();
    private static Condition B = lock.newCondition();
    private static Condition C = lock.newCondition();

    private static int count = 0;

    static class ThreadA extends Thread{
        @Override
        public void run() {
            try {
                lock.lock();
                for (int i = 0; i < 10; i++) {
                    while (count % 3 != 0){ //当count % 3 !=0 的时候，不让他执行，只有等于0才能执行，也就是刚轮到她执行
                        A.await();
                    }
                    System.out.println("A");
                    count++;
                    B.signal();
                }
            }catch (InterruptedException e){
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }
    }
    static class ThreadB extends Thread{
        @Override
        public void run() {
            try {
                lock.lock();
                for (int i = 0; i < 10; i++) {
                    while (count % 3 != 1){ //当count % 3 !=1 的时候，不让他执行，只有等于1才能执行，也就是刚轮到她执行
                        B.await();
                    }
                    System.out.println("B");
                    count++;
                    C.signal();
                }
            }catch (InterruptedException e){
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }
    }
    static class ThreadC extends Thread{
        @Override
        public void run() {
            try {
                lock.lock();
                for (int i = 0; i < 10; i++) {
                    while (count % 3 != 2){ //当count % 3 !=2 的时候，不让他执行，只有等于2才能执行，也就是刚轮到她执行
                        C.await();
                    }
                    System.out.println("C");
                    count++;
                    A.signal();
                }
            }catch (InterruptedException e){
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new ThreadA().start();
        new ThreadB().start();
        new ThreadC().start();
    }
}
```

 condition一般和reentrantlock结合使用。维护一个资源的等待队列，其底层可以看作一个虚拟节点，这个节点有两部分组成，一部分指向这个等待队列的头节点，一部分指向这个等待队列的尾节点，这个等待队列的头节点拥有这个资源，等待队列中的其他的线程就会进入阻塞状态，只有头节点被移除，新的头节点才能获取到这个资源。condition是由reentrantlock的实现类lock类对象创建出来的。和lock结合使用可以实现线程的阻塞唤醒等功能。

voilate：这个关键字保证了可见性和禁止指令重排，可见性是指多个线程对同一个资源进行操作时，彼此能同时知道。从底层来讲就是在线程完成写操作时，会立刻将值写回主存。禁止指令重排是指在单线程环境下，JVM会将两个毫无联系的语句打乱顺序执行，也是为了执行的效率。但是这种行为在多线程中会引起很多的问题。使用了voilate可以禁止指令重排。加入了voilate关键字的代码，会多出一个lock的前缀，这个前缀相当于一个内存屏障，保证了以下几点：

* 重排序时不会把后面的指令重排序到内存屏障前面。
* 执行完操作之后会迅速的将值写回主存。
* 写回内存的操作会引起其他cpu中缓存的该数据无效化(cpu总线嗅探机制)

cpu总线嗅探机制：cpu会不断地从总线上嗅探是否有修改的voilate修饰的数据，如果有，当前cpu工作内存中的数据无效化。

## 问了threadlocal，讲到内存泄露，采用开放定址法线性探测，二次探测，询问了用什么能够替换锁，这个有点懵逼，讲了cas并发机制 

thradlocal是一个本地线程副本变量工具类。每个threadlocal都维护一个threadlocalmap，这个map的key是threadlocal，value是存放对应threadlocal的变量。在高并发场景下，不同线程下的这些变量不会相互影响。

内存泄露：threadlocalmap中的key是弱引用，弱引用在这个对象有强引用的时候不会被GC。但是当这个key引用的强引用断开之后，这个key会被GC，变为null，但是这个value因为是强引用，不会发生GC，这就导致内存泄露问题的产生。但是jdk官方已经想到了这个问题，在每次调用set或者get方法时，都会将key为null的entry进行清理。但是最好在编程中，使用完threadlocal就调用remove方法。

threadlocalmap使用开放地址法解决hash冲突。就是通过计算得出下标时，发现产生冲突，就会查看下个位置是否可以使用。

二次探测：若当前key与原来key产生相同的哈希地址，则当前key存在该地址后偏移量为（1,2,3...）的二次方地址处

cas：compare and swap的缩写，底层使用unsafe类的compareandset方法，意思是比较并交换，当期望值和当前值相同时，执行交换操作。unsafe类是native修饰的类，底层时使用C或者C++写的，unsafe类相当于一个后面，用来直接操作内存。

## 乐观锁和悲观锁，讲一讲CAS。 

悲观锁：每次拿数据都要认为别人会修改，所以先进行加锁，再拿数据。

乐观锁：每次拿数据都认为别人不会修改，所以不加锁，直接拿数据。

乐观锁是通过cas实现的，多个线程尝试同时更新一个数据，只有一个线程可以成功得到这个数据，其他线程都会被挂起。

cas：compare and swap，底层通过调用unsafe类中的compareandset来实现，意思是比较并交换，这个方法存在三个操作数，内存位置，期望值，更新值。如果内存中值为期望值，则直接更新，否则不做任何操作。

## 讲一下volatile关键字 

就我理解的而言，被volatile修饰的共享变量，就具有了以下两点特性：

- 1.保证了不同线程对该变量操作的内存可见性;
- 2.禁止指令重排序

### 什么是内存可见性，什么又是重排序呢？

由于CPU执行指令的速度是很快的，但是内存访问的速度就慢了很多，相差的不是一个数量级，所以搞处理器的那群大佬们又在CPU里加了好几层高速缓存。 

JMM规定所有变量都是存在主存中的，类似于上面提到的普通内存，每个线程又包含自己的工作内存，方便理解就可以看成CPU上的寄存器或者高速缓存。

所以线程的操作都是以工作内存为主，它们只能访问自己的工作内存，且工作前后都要把值在同步回主内存。 这么说得我自己都有些不清楚了，拿张纸画一下： 

![img](https://cdn.jiler.cn/techug/uploads/2018/01/1.png.jpg)

  在线程执行时，首先会从主存中read变量值，再load到工作内存中的副本中，然后再传给处理器执行，执行完毕后再给工作内存中的副本赋值，随后工作内存再把值传回给主存，主存中的值才更新。 使用工作内存和主存，虽然加快的速度，但是也带来了一些问题。比如看下面一个例子：



```java
i = i + 1;
```

假设i初值为0，当只有一个线程执行它时，结果肯定得到1，当两个线程执行时，会得到结果2吗？这倒不一定了。可能存在这种情况：

```java
线程1： load i from 主存    // i = 0
        i + 1  // i = 1
线程2： load i from主存  // 因为线程1还没将i的值写回主存，所以i还是0
        i +  1 //i = 1
线程1:  save i to 主存
线程2： save i to 主存
```

如果两个线程按照上面的执行流程，那么i最后的值居然是1了。

下面就要提到你刚才问到的问题了，JMM主要就是围绕着如何在并发过程中如何处理原子性、可见性和有序性这3个特征来建立的，通过解决这三个问题，可以解除缓存不一致的问题。而volatile跟可见性和有序性都有关。

### 原子性、可见性和有序性

**1.原子性(Atomicity)：** Java中，对基本数据类型的读取和赋值操作是原子性操作，所谓原子性操作就是指这些操作是不可中断的，要做一定做完，要么就没有执行。 比如：

```java
i = 2;
j = i;
i++;
i = i + 1;
```

上面4个操作中，i=2是读取操作，必定是原子性操作，j=i你以为是原子性操作，其实吧，分为两步，一是读取i的值，然后再赋值给j,这就是2步操作了，称不上原子操作，i++和i = i +  1其实是等效的，读取i的值，加1，再写回主存，那就是3步操作了。所以上面的举例中，最后的值可能出现多种情况，就是因为满足不了原子性。  这么说来，只有简单的读取，赋值是原子操作，还只能是用数字赋值，用变量的话还多了一步读取变量值的操作。

JMM只实现了基本的原子性，像上面i++那样的操作，必须借助于synchronized和Lock来保证整块代码的原子性了。线程在释放锁之前，必然会把i的值刷回到主存的。 

**2. 可见性(Visibility)：** 说到可见性，Java就是利用volatile来提供可见性的。  当一个变量被volatile修饰时，那么对它的修改会立刻刷新到主存，当其它线程需要读取该变量时，会去内存中读取新值。而普通变量则不能保证这一点。

 **3. 有序性（Ordering）** JMM是允许编译器和处理器对指令重排序的，但是规定了as-if-serial语义，即不管怎么重排序，程序的执行结果不能改变。比如下面的程序段：

```java
double pi = 3.14;    //A
double r = 1;        //B
double s= pi * r * r;//C
```

上面的语句，可以按照A->B->C执行，结果为3.14,但是也可以按照B->A->C的顺序执行，因为A、B是两句独立的语句，而C则依赖于A、B，所以A、B可以重排序，但是C却不能排到A、B的前面。JMM保证了重排序不会影响到单线程的执行，但是在多线程中却容易出问题。 比如这样的代码:

```java
int a = 0;
bool flag = false;

public void write() {
    a = 2;              //1
    flag = true;        //2
}

public void multiply() {
    if (flag) {         //3
        int ret = a * a;//4
    }
}
```

假如有两个线程执行上述代码段，线程1先执行write，随后线程2再执行multiply，最后ret的值一定是4吗？结果不一定： 

![img](https://cdn.jiler.cn/techug/uploads/2018/01/2.png.jpg)

另外，JMM具备一些先天的有序性,即不需要通过任何手段就可以保证的有序性，通常称为happens-before原则。**happens-before规则**： 

* 单线程保证语义的串行性（书写在前面的操作先行发生于书写在后面的操作）
* volatile变量的写先于读发生
* 锁规则：解锁必然发生在加锁前（一个unlock操作先行发生于后面对同一个锁的lock操作）
* A先于B，B先于C，那么A必然先于C （先于，先行发生于）
* 线程的 start() 方法先于它的每一个动作
* 线程的所有操作先于线程的终结
* 线程的中断先于被中断线程的代码
* 构造函数的执行和结束 先于 finalize() 方法

### volatile关键字如何满足并发编程的三大特性的？

那就要重提volatile变量规则：就是如果一个变量声明成是volatile的，那么当我读变量时，总是能读到它的最新值，这里最新值是指不管其它哪个线程对该变量做了写操作，都会立刻被更新到主存里

```java
int a = 0;
bool flag = false;

public void write() {
   a = 2;              //1
   flag = true;        //2
}

public void multiply() {
   if (flag) {         //3
       int ret = a * a;//4
   }
}
```

这段代码不仅仅受到重排序的困扰，即使1、2没有重排序。3也不会那么顺利的执行的。假设还是线程1先执行write操作，线程2再执行multiply操作，由于线程1是在工作内存里把flag赋值为1，不一定立刻写回主存，所以线程2执行时，multiply再从主存读flag值，仍然可能为false，那么括号里的语句将不会执行。 如果改成下面这样：

```java
int a = 0;
volatile bool flag = false;

public void write() {
   a = 2;              //1
   flag = true;        //2
}

public void multiply() {
   if (flag) {         //3
       int ret = a * a;//4
   }
}
```

那么线程1先执行write,线程2再执行multiply。根据happens-before原则，这个过程会满足以下3类规则： **程序顺序规则：**1 happens-before 2; 3 happens-before 4; (volatile限制了指令重排序，所以1 在2 之前执行) 

### volatile的两点内存语义能保证可见性和有序性，但是能保证原子性吗？

首先我回答是不能保证原子性，要是说能保证，也只是对单个volatile变量的读/写具有原子性，但是对于类似volatile++这样的复合操作就无能为力了，比如下面的例子：

```java
public class Test {
    public volatile int inc = 0;
 
    public void increase() {
        inc++;
    }
 
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
 
        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
```

按道理来说结果是10000，但是运行下很可能是个小于10000的值。有人可能会说volatile不是保证了可见性啊，一个线程对inc的修改，另外一个线程应该立刻看到啊！可是这里的操作inc++是个复合操作啊，包括读取inc的值，对其自增，然后再写回主存。 假设线程A，读取了inc的值为10，这时候被阻塞了，因为没有对变量进行修改，触发不了volatile规则。  线程B此时也读读inc的值，主存里inc的值依旧为10，做自增，然后立刻就被写回主存了，为11。  此时又轮到线程A执行，由于工作内存里保存的是10，所以继续做自增，再写回主存，11又被写了一遍。所以虽然两个线程执行了两次increase()，结果却只加了一次。 有人说，**volatile不是会使缓存行无效的吗？**但是这里线程A读取到线程B也进行操作之前，并没有修改inc值，所以线程B读取的时候，还是读的10。 又有人说，线程B将11写回主存，**不会把线程A的缓存行设为无效吗？**但是线程A的读取操作已经做过了啊，只有在做读取操作时，发现自己缓存行无效，才会去读主存的值，所以这里线程A只能继续做自增了。  综上所述，在这种复合操作的情景下，原子性的功能是维持不了了。但是volatile在上面那种设置flag值的例子里，由于对flag的读/写操作都是单步的，所以还是能保证原子性的。 要想保证原子性，只能借助于synchronized,Lock以及并发包下的atomic的原子操作类了，即对基本数据类型的  自增（加1操作），自减（减1操作）、以及加法操作（加一个数），减法操作（减一个数）进行了封装，保证这些操作是原子性操作。

### volatile底层的实现机制？

如果把加入volatile关键字的代码和未加入volatile关键字的代码都生成汇编代码，会发现加入volatile关键字的代码会多出一个lock前缀指令。 lock前缀指令实际相当于一个内存屏障，内存屏障提供了以下功能：

 **1.重排序时不能把后面的指令重排序到内存屏障之前的位置** 

**2.会将当前处理器缓存行的数据立即写会系统主存**

**3.这个写回内存的操作会引起在其他cpu里缓存了该内存地址的数据无效（MES协议）**

### 哪里会使用到volatile，举两个例子呢？

**1.状态量标记，就如上面对flag的标记，我重新提一下：**

```java
int a = 0;
volatile bool flag = false;

public void write() {
    a = 2;              //1
    flag = true;        //2
}

public void multiply() {
    if (flag) {         //3
        int ret = a * a;//4
    }
}
```

这种对变量的读写操作，标记为volatile可以保证修改对线程立刻可见。比synchronized,Lock有一定的效率提升。 **2.单例模式的实现，典型的双重检查锁定（DCL）**

```java
class Singleton{
    private volatile static Singleton instance = null;
 
    private Singleton() {
 
    }
 
    public static Singleton getInstance() {
        if(instance==null) {
            synchronized (Singleton.class) {
                if(instance==null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```

这是一种懒汉的单例模式，使用时才创建对象，而且为了避免初始化操作的指令重排序，给instance加上了volatile。

## 4.多线程相关，volatile原理，JMM，synchronize优化（还讲了对象头markword 和类元指针），reentrantlock原理，AQS。 

并发编程由以下3大重要特性：

原子性：保证每个操作要么都成功执行，要么都执行失败。

可见性：保证对每个数据的修改都会立刻被其他的线程看到。

有序性：程序按照代码的先后顺序执行。在JVM中，为了执行效率，有可能会发生指令重排的现象。也就是会发生将程序的不相关操作乱序执行。在单线程中，这样是没有问题的，但是在多线程中，可能产生很严重的问题。



volatile可以保证有序性和可见性，无法保证原子性。

volatile的可见性：

- 当对volatile变量执行写操作后，JMM会把工作内存中的最新变量值强制刷新到主内存
- 写操作会导致其他线程中的缓存无效

这样，其他线程使用缓存时，发现本地工作内存中此变量无效，便从主内存中获取，这样获取到的变量便是最新的值，实现了线程的可见性。

volatile的有序性：

volatile在读写前后都加上了内存屏障，在一定程度上保证了有序性。

JMM：线程独有的区域由：程序计数器、本地方法栈、虚拟机栈。共有的区域有：方法区、堆内存、直接内存。

锁优化：无锁、偏向锁、轻量级锁、重量级锁。随着对资源的竞争激烈程度，进行锁升级。

在synchronized里面，起关键性作用的是对象头和monitor。synchronized通过monitorenter获取锁，通过monitorexit释放锁。

一个对象包括对象头、实例数据和对齐填充。

对象头中分为两部分：markword和klass pointer。markword中存储了对象运行时的状态。

markword是随着运行状态而变化的。

reentrantlock原理：reentrantlock是一个可重入锁，指的是同一线程外层函数获得锁后,再进入该线程的内层方法会自动获取锁。底层是使用CAS+AQS，AQS是AbsractQueuedSynchronized的缩写，是一个队列，是一个FIFO队列，维护想要访问的共享资源的线程队列。CAS是compare and swap的简写，通过调用底层的unsafe类中的compareandset方法进行锁的获取。compareandset方法是对对象的state字段进行检查，如果当前state为0，说明资源没有被上锁，可以使用，如果不为0，则表示已经被上锁。

reentrantlock默认使用非公平锁，也就是线程可以抢夺共享资源的访问权，如果使用公平锁，如果当前资源被上锁，当前线程进入AQS队列，排队等待锁释放，对头节点是正在访问资源的线程，如果释放锁，下个资源就可以获取到锁，并且移除对头节点。

## 12.ThreadLocal原理，里面的map具体怎么实现的，和hashmap的不同，内存泄漏问题，具体应用场景。 

每个线程维护一个threadlocalmap，threadlocal本身并不存储数据，只是一个用来存取数据的工具类。

其中的map类似hashmap，是一个数组链表结构，其中key存储threadlocal的弱引用，value存放具体的信息。

具体可以看下图：

![img](https://pic2.zhimg.com/80/v2-aaba364d9af11719667ffc9ab6b64dad_720w.jpg)

与hashmap的区别：

1.hashmap中的key可以存储任意的对象，threadlocalmap中的key可以存储threadlocal的弱引用。

2.hashmap的hashcode计算方法和threadlocalmap不一样。

3.hashmap中冲突解决使用的是链地址法，threadlocalmap使用开放地址法。

内存泄漏问题：当弱引用的对象的强引用存在时，弱引用不会被GC。当强引用被GC，key就会被置为null，这样的entry永远不会被GC。这就会造成内存泄漏问题。

内存泄漏的两个必要条件：1.threadlocal的强引用已经断开。2.当前线程并未结束。

源码中已经考虑到这个问题，在每次调用set/get方法都会对key为null的entry进行清理。但是这并不能解决问题，在使用中，最好在用完之后进行remove操作。

应用场景：在框架中，使用threadlocal进行数据库连接，保证线程的每次都可以连接同一个数据库。

或者是解决simpledataformat类的线程安全问题。