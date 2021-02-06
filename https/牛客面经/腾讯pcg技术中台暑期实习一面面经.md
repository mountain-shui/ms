### 腾讯pcg技术中台暑期实习一面面经 ###

#### 1.内存泄漏相关的问题，ThreadLocal ####

每个Thread 维护一个 ThreadLocalMap 映射表，这个映射表的 key 是 ThreadLocal实例本身，value 是真正需要存储的 Object。
也就是说 ThreadLocal 本身并不存储值，它只是作为一个 key 来让线程从 ThreadLocalMap 获取 value。
ThreadLocalMap 是使用 ThreadLocal 的弱引用作为 Key 的，弱引用的对象在 GC 时会被回收。
ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用来引用它，
那么系统 GC 的时候，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现key为null的Entry，
就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，
这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value永远无法回收，
造成内存泄漏。

#### 2.id生成器的策略，为什么不用uuid，单点、多机都会有啥样的问题？ ####

生成分布式Id的方法主要有以下几种：

1. ##### 数据库自增ID。 #####

   在创建表的时候，指定主键auto_increment（自增）便可以实现。

   但是使用数据库的自增ID，虽然简单，会带来ID重复的问题，并且单机版的ID自增，并且每次生成一个ID都会访问数据库一次，DB的压力也很大，并没有什么并发性能可言。

2. ##### 数据库水平拆分，设置初始值和相同的自增步长。 ######

   针对上面的数据库自增ID出现的问题：ID重复、性能不好。就出现了集群版的生成分布式ID方案。**「数据库水平拆分，设置初始值和相同的自增步长」**和**「批量申请自增ID」**。

   **「数据库水平拆分，设置初始值和相同的自增步长」**是指在DB集群的环境下，将数据库进行水平划分，然后每个数据库设置**「不同的初始值」**和**「相同的步长」**，这样就能避免ID重复的情况。

   ![img](https://pic4.zhimg.com/80/v2-8556f8bc7eee56f995e10af59fd553d3_720w.jpg)

   我这里假设有三个数据库，为每一个数据库设置初始值，设置初始值可以通过下面的sql进行设置：

   ```text
   set @@auto_increment_offset = 1;     // 设置初始值
   set @@auto_increment_increment = 2;  // 设置步长
   ```

   三个数据的初始值分别设置为1、2、3，一般步长设置为数据库的数据，这里数据库数量为3，所以步长也设置为3。

   面试官：若是面对再次扩容的情况呢？

   扩容的情况是这种方法的一个缺点，上面我说的步长一般设置为数据库的数量，这是在确保后期不会扩容的情况下，若是确定后期会有扩容情况，在前期设计的的时候可以将步长设置长一点，**「预留一些初始值给后续扩容使用」**。

   总之，这种方案还是优缺点的，但是也有自己的优点，缺点就是：**「后期可能会面对无ID初始值可分的窘境，数据库总归是数据库，抗高并发也是有限的」**。

   它的优点就是算是解决了**「DB单点的问题」**。

3. ##### 批量申请自增ID。 #####

   **「批量申请自增ID」**的解决方案可以解决无ID可分的问题，它的原理就是一次性给对应的数据库上分配一批的id值进行消费，使用完了，再回来申请。

   ![img](https://pic3.zhimg.com/80/v2-0e53d5413a5aeb7d5ca8149e53546742_720w.jpg)

   在设计的初始阶段可以设计一个有初始值字段，并有步长字段的表，当每次要申请批量ID的时候，就可以去该表中申请，每次申请后**「初始值=上一次的初始值+步长」**。

   这样就能保持初始值是每一个申请的ID的最大值，避免了ID的重复，并且每次都会有ID使用，一次就会生成一批的id来使用，这样访问数据库的次数大大减少。

   但是这一种方案依旧有自己的缺点，依然不能抗真正意义上的高并发。

4. ##### UUID生成。 #####

   第四种方式是使用**「UUID生成」**的方式生成分布式ID，UUID的核心思想是使用**「机器的网卡、当地时间、一个随机数」**来生成UUID。

   使用UUID的方式只需要调用UUID.randomUUID().toString()就可以生成，这种方式方便简单，本地生成，不会消耗网络。

   简单的东西，出现的问题就会越多，不利于存储，16字节128位，通常是以36位长度的字符串表示，很多的场景都不适合。

   并且UUID生成的无序的字符串，查询效率低下，没有实际的业务含义，不具备自增特性，所以都不会使用UUID作为分布式ID来使用。

   面试官：恩额，那你知道生成UUID的方式有几种吗？不知道没关系，这个只是作为一个扩展。

   这个我只知道可以通过**「当前的时间戳及机器mac地址」**来生成，可以确保生成的UUID全球唯一，其它的没有了解过。

5. ##### Redis的方式。 #####

   为了解决上面纯关系型数据库生成分布式ID无法抗高并发的问题，可以使用Redis的方式来生成分布式ID。

   Redis本身有incr和increby 这样自增的命令，保证原子性，生成的ID也是有序的。

   Redis基于内存操作，性能高效，不依赖于数据库，数据天然有序，利于分页和排序。

   但是这个方案也会有自己的缺点，因为增加了中间件，需要自己编码实现工作量增大，增加复杂度。

   使用Redis的方式还要考虑持久化，Redis的持久化有两种**「RDB和AOF」**，**「RDB是以快照的形式进行持久化，会丢失上一次快照至此时间的数据」**。

   **「AOF可以设置一秒持久化一次，丢失的数据是秒内的」**，也会存在可能上一次自增后的秒内的ID没有持久化的问题。

   但是这种方法相对于上面的关系型数据库生成分布式ID的方法而言，已经优越了许多。

   若是数据量比较大的话，重启Redis的时间也会比较长，可以采用Redis的集群方式。

   第一种是使用RedisAtomicLong 原子类使用CAS操作来生成ID。

   ```text
   @Service
   public class RedisSequenceFactory {
       @Autowired
       RedisTemplate<String, String> redisTemplate;
   
       public void setSeq(String key, int value, Date expireTime) {
           RedisAtomicLong counter = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
           counter.set(value);
           counter.expireAt(expireTime);
       }
   
       public void setSeq(String key, int value, long timeout, TimeUnit unit) {
           RedisAtomicLong counter = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
           counter.set(value);
           counter.expire(timeout, unit);
       }
   
       public long generate(String key) {
           RedisAtomicLong counter = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
           return counter.incrementAndGet();
       }
   
       public long incr(String key, Date expireTime) {
           RedisAtomicLong counter = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
           counter.expireAt(expireTime);
           return counter.incrementAndGet();
       }
   
       public long incr(String key, int increment) {
           RedisAtomicLong counter = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
           return counter.addAndGet(increment);
       }
   
       public long incr(String key, int increment, Date expireTime) {
           RedisAtomicLong counter = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
           counter.expireAt(expireTime);
           return counter.addAndGet(increment);
       }
   }
   ```

   第二种是使用redisTemplate.opsForHash()和结合UUID的方式来生成生成ID。

   ```text
   public Long getSeq(String key,String hashKey,Long delta) throws BusinessException{
           try {
               if (null == delta) {
                   delta=1L;
               }
               return redisTemplate.opsForHash().increment(key, hashKey, delta);
           } catch (Exception e) {  // 若是redis宕机就采用uuid的方式
               int first = new Random(10).nextInt(8) + 1;
               int randNo=UUID.randomUUID().toString().hashCode();
               if (randNo < 0) {
                   randNo=-randNo;
               }
               return Long.valueOf(first + String.format("%16d", randNo));
           }
       }
   ```

   我把电脑移回给面试官，他很快的扫了一下我的代码，说了一句。

   面试官：小伙子，不写注释哦，这个习惯不好哦。

6. ##### 雪花算法。 #####

   他是采用64bit作为id生成类型，并且将64bit划分为，如下图的几段。

   ![img](https://pic4.zhimg.com/80/v2-ee91bb59ff6c2a627635ce1ebcb5d97f_720w.jpg)

   第一位作为标识位，因为Java中long类型的时代符号的，因为ID位正数，所以第一位位0。

   接着的41bit是时间戳，毫秒级位单位，注意这里的时间戳并不是指当前时间的时间戳，而是值之间差（**「当前时间-开始时间」**）。

   这里的开始时间一般是指ID生成器的开始时间，是由我们程序自己指定的。

   接着后面的10bit：包括5位的**「数据中心标识ID（datacenterId）和5位的机器标识ID（workerId）」**，可以最多标识1024个节点（1<<10=1024）。

   最的12位是序列号，12位的计数顺序支持每个节点每毫秒产生4096序列号（1<<12=4096）。

   雪花算法使用数据中心ID和机器ID作为标识，不会产生ID的重复，并且是在本地生成，不会消耗网络，效率高，有数据显示，每秒能生成26万个ID。

   但是雪花算法也是有自己的缺点，因为雪花算法的计算依赖于时间，若是系统时间回拨，就会产生重复ID的情况。

   面试官：那对于时间回拨产生重复ID的情况，你有什么比较好的解决方案吗？

   在雪花算法的实现中，若是其前置的时间等于当前的时间，就抛出异常，也可以关闭掉时间回拨。

   对于回拨时间比较短的，可以等待回拨时间过后再生成ID。

   面试官：你可以帮我敲一个雪花算法吗？我这键盘给你。

   ```java
   /**
    * 雪花算法
    * @author：黎杜
    */
   public class SnowflakeIdWorker {
   
       /** 开始时间截 */
       private final long twepoch = 1530051700000L;
   
       /** 机器id的位数 */
       private final long workerIdBits = 5L;
   
       /** 数据标识id的位数 */
       private final long datacenterIdBits = 5L;
   
       /** 最大的机器id，结果是31 */
       private final long maxWorkerId = -1L ^ (-1L << workerIdBits);
   
       /** 最大的数据标识id，结果是31 */
       private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
   
       /** 序列的位数 */
       private final long sequenceBits = 12L;
   
       /** 机器ID向左移12位 */
       private final long workerIdShift = sequenceBits;
   
       /** 数据标识id向左移17位 */
       private final long datacenterIdShift = sequenceBits + workerIdBits;
   
       /** 时间截向左移22位*/
       private final long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;
   
       /** 生成序列的掩码 */
       private final long sequenceMask = -1L ^ (-1L << sequenceBits);
   
       /** 工作机器ID(0~31) */
       private long workerId;
   
       /** 数据中心ID(0~31) */
       private long datacenterId;
   
       /** 毫秒内序列(0~4095) */
       private long sequence = 0L;
   
       /** 上次生成ID的时间截 */
       private long lastTimestamp = -1L;
   
       /**
        * 构造函数
        * @param workerId 工作ID (0~31)
        * @param datacenterId 数据中心ID (0~31)
        */
       public SnowflakeIdWorker(long workerId, long datacenterId) {
           if (workerId > maxWorkerId || workerId < 0) {
               throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
           }
           if (datacenterId > maxDatacenterId || datacenterId < 0) {
               throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
           }
           this.workerId = workerId;
           this.datacenterId = datacenterId;
       }
   
       /**
        * 获得下一个ID (该方法是线程安全的)
        * @return SnowflakeId
        */
       public synchronized long nextId() {
           long timestamp = getCurrentTime();
   
           //如果当前时间小于上一次生成的时间戳，说明系统时钟回退过就抛出异常
           if (timestamp < lastTimestamp) {
               throw new BusinessionException("回拨的时间为："+lastTimestamp - timestamp);
           }
   
           //如果是同一时间生成的，则进行毫秒内序列
           if (lastTimestamp == timestamp) {
               sequence = (sequence + 1) & sequenceMask;
               //毫秒内序列溢出
               if (sequence == 0) {
                   //获得新的时间戳
                   timestamp = tilNextMillis(lastTimestamp);
               }
           } else {  //时间戳改变，毫秒内序列重置
               sequence = 0L;
           }
   
           //上次生成ID的时间截
           lastTimestamp = timestamp;
   
           //移位并通过或运算拼到一起组成64位的ID
           return ((timestamp - twepoch) << timestampLeftShift) // 计算时间戳
                   | (datacenterId << datacenterIdShift) // 计算数据中心
                   | (workerId << workerIdShift) // 计算机器ID
                   | sequence; // 序列号
       }
   
       /**
        *获得新的时间戳
        * @param lastTimestamp 上次生成ID的时间截
        * @return 当前时间戳
        */
       protected long tilNextMillis(long lastTimestamp) {
           long timestamp = getCurrentTime();
           // 若是当前时间等于上一次的1时间就一直阻塞，知道获取到最新的时间（回拨后的时间）
           while (timestamp <= lastTimestamp) {
               timestamp = getCurrentTime();
           }
           return timestamp;
       }
   
       /**
        * 获取当前时间
        * @return 当前时间(毫秒)
        */
       protected long getCurrentTime() {
           return System.currentTimeMillis();
       }
   ```

   

7. ##### 百度UidGenerator算法 #####

8.  ##### 美团Leaf算法 #####

   最后两种确实没有深入了解，之前有看网上的资料说美团Leaf算法需要依赖于数据库，ZK，并且也能保证去全局ID的唯一性，单项递增。

   我：而百度UidGenerator算法是基于雪花算法进行实现的，也是需要借助于数据库，与雪花算法不同的是，**「UidGenerator支持自定义时间戳、主句中心ID和机器ID、序列号的位数」**。

#### 3.java的锁，synchronized/lock，相同处和不同处，如何实现可重入的？ ####

​	1. 两者都是可重入锁

​	**“可重入锁”**  指的是自己可以再次获取自己的内部锁。

​	synchronized，比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要	获取这个对象的锁的时候还是可以	获取的，如果不可	锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增 1，所以	要等到锁的计数器下降为 0 时	才能释放锁。

​	ReentrantLock是通过保存的持有锁的线程来判断获取锁的操作是重入的还是竞争的。在获取锁后，设置一个标识变量为当前线程   

​	exclusiveOwnerThread，当线程再次进入判断exclusiveOwnerThread变量是否等于本线程来判断.                    

​	2. synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API

​	`	synchronized` 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 `synchronized` 关键字进行了很多优化，但是这些	优化都是在虚拟机层面实现的，并没有直接暴露给我们。`ReentrantLock` 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和unlock() 方法配合 try/finally 语句块来完成）。

​	3.ReentrantLock 比 synchronized 增加了一些高级功能

​	相比`synchronized`，`ReentrantLock`增加了一些高级功能。主要来说主要有三点：

- **等待可中断** : `ReentrantLock`提供了一种能够中断等待锁的线程的机制，通过 `lock.lockInterruptibly()` 来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
- **可实现公平锁** : `ReentrantLock`可以指定是公平锁还是非公平锁。而`synchronized`只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。`ReentrantLock`默认情况是非公平的，可以通过 `ReentrantLock`类的`ReentrantLock(boolean fair)`构造方法来制定是否是公平的。
- **可实现选择性通知（锁可以绑定多个条件）**: `synchronized`关键字与`wait()`和`notify()`/`notifyAll()`方法相结合可以实现等待/通知机制。`ReentrantLock`类当然也可以实现，但是需要借助于`Condition`接口与`newCondition()`方法。

> `Condition`是 JDK1.5 之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个`Lock`对象中可以创建多个`Condition`实例（即对象监视器），**线程对象可以注册在指定的`Condition`中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用`notify()/notifyAll()`方法进行通知时，被通知的线程是由 JVM 选择的，用`ReentrantLock`类结合`Condition`实例可以实现“选择性通知”** ，这个功能非常重要，而且是 Condition 接口默认提供的。而`synchronized`关键字就相当于整个 Lock 对象中只有一个`Condition`实例，所有的线程都注册在它一个身上。如果执行`notifyAll()`方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而`Condition`实例的`signalAll()`方法 只会唤醒注册在该`Condition`实例中的所有等待线程。

​	如果你想使用上述功能，那么选择 ReentrantLock 是一个不错的选择。性能已不是选择标准



#### 4.写个LinkedBlockingQueue； ####



####  5.java的线程池，核心参数都是啥样的，有没有看过代码； ####

​	`ThreadPoolExecutor`构造函数重要参数分析

​	**`ThreadPoolExecutor` 3 个最重要的参数：**

- **`corePoolSize` :** 核心线程数线程数定义了最小可以同时运行的线程数量。
- **`maximumPoolSize` :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
- **`workQueue`:** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

`ThreadPoolExecutor`其他常见参数:

1. **`keepAliveTime`**:当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
2. **`unit`** : `keepAliveTime` 参数的时间单位。
3. **`threadFactory`** :executor 创建新线程的时候会用到。
4. **`handler`** :饱和策略。关于饱和策略下面单独介绍一下。

**ThreadPoolExecutor 饱和策略**

**`ThreadPoolExecutor` 饱和策略定义:**

如果当前同时运行的线程数量达到最大线程数量并且队列也已经被放满了任时，`ThreadPoolTaskExecutor` 定义一些策略:

- **`ThreadPoolExecutor.AbortPolicy`**：抛出 `RejectedExecutionException`来拒绝新任务的处理。
- **`ThreadPoolExecutor.CallerRunsPolicy`**：调用执行自己的线程运行任务，也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
- **`ThreadPoolExecutor.DiscardPolicy`：** 不处理新任务，直接丢弃掉。
- **`ThreadPoolExecutor.DiscardOldestPolicy`：** 此策略将丢弃最早的未处理的任务请求。

举个例子： Spring 通过 `ThreadPoolTaskExecutor` 或者我们直接通过 `ThreadPoolExecutor` 的构造函数创建线程池的时候，当我们不指定 `RejectedExecutionHandler` 饱和策略的话来配置线程池的时候默认使用的是 `ThreadPoolExecutor.AbortPolicy`。在默认情况下，`ThreadPoolExecutor` 将抛出 `RejectedExecutionException` 来拒绝新来的任务 ，这代表你将丢失对这个任务的处理。 对于可伸缩的应用程序，建议使用 `ThreadPoolExecutor.CallerRunsPolicy`。当最大池被填满时，此策略为我们提供可伸缩队列。（这个直接查看 `ThreadPoolExecutor` 的构造函数源码就可以看出，比较简单的原因，这里就不贴代码了）

#### 6.concurrentHashmap底层原理，1.7\1.8区别，对锁是怎么安排的； ####

Java7 中 ConcruuentHashMap 使用的分段锁，也就是每一个 Segment 上同时只有一个线程可以操作，每一个  Segment 都是一个类似 HashMap 数组的结构，它可以扩容，它的冲突会转化为链表。但是 Segment 的个数一但初始化就不能改变。

Java8 中的 ConcruuentHashMap  使用的 Synchronized 锁加 CAS 的机制。结构也由 Java7 中的 **Segment 数组 + HashEntry 数组 + 链表** 进化成了  **Node 数组 + 链表 / 红黑树**，Node 是类似于一个 HashEntry 的结构。它的冲突再达到一定大小时会转化成红黑树，在冲突小于一定数量时又退回链表。

有些同学可能对 Synchronized 的性能存在疑问，其实 Synchronized 锁自从引入锁升级策略后，性能不再是问题，有兴趣的同学可以自己了解下 Synchronized 的**锁升级**。

#### 7.写个快排； ####

```java
public static void quickSprt(int[] array, int left, int right) {
        //left数组的最小下标，right数组最大下标
        if (left > right) {                    //判断本次是否执行
            return;
        }
        int low = left;
        int high=right;
        int mid=array[low];                    //将数组第一个数定为基准
        while(low<high){
            while(low<high && array[high]>mid) //从后往前，直到找到小于基准的数
                high--;
            array[low]=array[high];            //小数移动到前面
            while(low<high && array[low]<mid)  //从前往后，直到找到大于基准的数
                low++;
            array[high]=array[low];            //大数移动到后面
        }
        array[low]=mid;                        //此时low在中间，将基准放入
        quickSprt(array,left,low-1);      //小数组递归
        quickSprt(array,low+1,right);      //大数组递归
    }
```

#### 8.cap理论； ####

CAP理论作为分布式系统的基础理论,它描述的是一个分布式系统在以下三个特性中：

- 一致性（**C**onsistency）
- 可用性（**A**vailability）
- 分区容错性（**P**artition tolerance）

最多满足其中的两个特性。也就是下图所描述的。分布式系统要么满足CA,要么CP，要么AP。无法同时满足CAP。

　　　　　　　　![img](https://img2018.cnblogs.com/blog/941183/201906/941183-20190614191945691-976367436.png)

 

 

**I.** 什么是 一致性、可用性和分区容错性

**分区容错性**：指的分布式系统中的某个节点或者网络分区出现了故障的时候，整个系统仍然能对外提供满足一致性和可用性的服务。也就是说部分故障不影响整体使用。

事实上我们在设计分布式系统是都会考虑到bug,硬件，网络等各种原因造成的故障，所以即使部分节点或者网络出现故障，我们要求整个系统还是要继续使用的

(不继续使用,相当于只有一个分区,那么也就没有后续的一致性和可用性了)

 

**可用性：** 一直可以正常的做读写操作。简单而言就是客户端一直可以正常访问并得到系统的正常响应。用户角度来看就是不会出现系统操作失败或者访问超时等问题。

 

**一致性**：在分布式系统完成某写操作后任何读操作，都应该获取到该写操作写入的那个最新的值。相当于要求分布式系统中的各节点时时刻刻保持数据的一致性。

 

**II.** 该怎么理解

如果我们**事先保证了分区容错性**，也意味着若某个节点故障了，用户还是可以继续访问。这时用户在访问过程中就会出现一致性和可用性不能同时满足的情况，参考下图：

![img](https://img2018.cnblogs.com/blog/941183/201906/941183-20190614193117074-1830209672.png)![img](https://img2018.cnblogs.com/blog/941183/201906/941183-20190614193138084-1712716947.png)

 

如图假设分布式系统有G1，G2两个节点，初始值都是v0。现在有一个client向系统写入了值v1，这里假设直接写的是节点G1。写完之后client再去读取这个值，这时读到了G2节点,

由于G2节点与G1节点失去连接，这时G1节点上的数据还未同步到G2节点，因此客户端读取到的是修改之前的值v0。 这就出现了不满足一致性的情况了。相当于**满足了可用性，失去了一致性**。

 

类似的，如果系统保证了强的一致性，那么在client 写完G1节点后, 而G1向G2节点同步数据出现了问题，这时如果client再去读取G2节点的数据时，client就会一直处于等待状态，因为系统内各节点

数据为同步上，需要等同步上才能使用。这就相当于**满足了一致性，而失去了可用性**。

![img](https://img2018.cnblogs.com/blog/941183/201906/941183-20190614210523552-1963221130.png)

 

 

考虑多个客户端访问时，一致性和可用性还可以这么理解：假如client1 向G1 修改某个值的时候, 写操作还未完成，client2就发起来对该值的读操作，读的是G2节点，这时如果要满足一致性，

那么就得让client2 暂时无法使用，如果要让client2 使用，那么获取到的数据不是最新的，系统就不满足一致性。

 

 **III.** CAP三者不可兼得，该如何取舍：

(1) CA: 优先保证一致性和可用性，放弃分区容错。 这也意味着放弃系统的扩展性，系统不再是分布式的，有违设计的初衷。

(2) CP: 优先保证一致性和分区容错性，放弃可用性。在数据一致性要求比较高的场合(譬如:zookeeper,Hbase) 是比较常见的做法，一旦发生网络故障或者消息丢失，就会牺牲用户体验，等恢复之后用户才逐渐能访问。

(3) AP: 优先保证可用性和分区容错性，放弃一致性。NoSQL中的Cassandra 就是这种架构。跟CP一样，放弃一致性不是说一致性就不保证了，而是逐渐的变得一致。



#### 9.注册中心一般是ap还是cp； ####

| --              | Nacos                      | Eureka      | Consul            | Zookeeper  |
| --------------- | -------------------------- | ----------- | ----------------- | ---------- |
| 一致性协议      | CP+AP                      | AP          | CP                | CP         |
| 健康检查        | TCP/HTTP/MYSQL/Client Beat | Client Beat | TCP/HTTP/gRPC/Cmd | Keep Alive |
| 负载均衡策略    | 权重/metadata/Selector     | Ribbon      | Fabio             | —          |
| 雪崩保护        | 有                         | 有          | 无                | 无         |
| 自动注销实例    | 支持                       | 支持        | 不支持            | 支持       |
| 访问协议        | HTTP/DNS                   | HTTP        | HTTP/DNS          | TCP        |
| 监听支持        | 支持                       | 支持        | 支持              | 支持       |
| 多数据中心      | 支持                       | 支持        | 支持              | 不支持     |
| 跨注册中心同步  | 支持                       | 不支持      | 支持              | 不支持     |
| SpringCloud集成 | 支持                       | 支持        | 支持              | 支持       |
| Dubbo集成       | 支持                       | 不支持      | 不支持            | 支持       |
| K8S集成         | 支持                       | 不支持      | 支持              | 不支持     |

**Zookeeper**

Zookeeper作为老牌的服务注册中心产品，其遵循的是CP原则，即舍弃了A。即任何时候对 Zookeeper 的访问请求能得到一致的数据结果，同时系统对网络分割具备容错性，但是 Zookeeper 不能保证每次服务请求都是可达的。

从 Zookeeper 的实际应用情况来看，在使用 Zookeeper 获取服务列表时，如果此时的 Zookeeper 集群中的  Leader 宕机了，该集群就要进行 Leader 的选举，又或者 Zookeeper  集群中半数以上服务器节点不可用(例如有三个节点，如果节点一检测到节点三挂了  ，节点二也检测到节点三挂了，那这个节点才算是真的挂了)，那么将无法处理该请求。所以说，Zookeeper 不能保证服务可用性。

**Consul**

Consul 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。Consul 使用 Go 语言编写，因此具有天然可移植性(支持Linux、windows和Mac OS X)。

Consul 内置了服务注册与发现框架、分布一致性协议实现、健康检查、Key/Value 存储、多数据中心方案，不再需要依赖其他工具(比如 ZooKeeper 等)，使用起来也较为简单。

Consul 也是遵循CAP原理中的CP原则，保证了强一致性和分区容错性，且使用的是Raft算法，比Zookeeper使用的Paxos算法更加简单。虽然保证了强一致性，但是可用性就相应下降了。

**Eureka**

Spring Cloud Netflix 在设计 Eureka 时就紧遵AP原则(尽管现在2.0发布了，但是由于其闭源的原因 ，但是目前 Ereka 1.x 任然是比较活跃的)。

Eureka Server 也可以运行多个实例来构建集群，解决单点问题，但不同于 ZooKeeper 的选举 leader  的过程，Eureka Server 采用的是Peer to Peer 对等通信。这是一种去中心化的架构，无 master/slave  之分，每一个 Peer 都是对等的。在这种架构风格中，节点通过彼此互相注册来提高可用性，每个节点需要添加一个或多个有效的 serviceUrl  指向其他节点。每个节点都可被视为其他节点的副本。

在集群环境中如果某台 Eureka Server 宕机，Eureka Client 的请求会自动切换到新的 Eureka Server 节点上，当宕机的服务器重新恢复后，Eureka  会再次将其纳入到服务器集群管理之中。当节点开始接受客户端请求时，所有的操作都会在节点间进行复制(replicate To  Peer)操作，将请求复制到该 Eureka Server 当前所知的其它所有节点中。

当一个新的 Eureka Server 节点启动后，会首先尝试从邻近节点获取所有注册列表信息，并完成初始化。Eureka Server 通过 getEurekaServiceUrls() 方法获取所有的节点，并且会通过心跳契约的方式定期更新。

默认情况下，如果 Eureka Server 在一定时间内没有接收到某个服务实例的心跳(默认周期为30秒)，Eureka Server 将会注销该实例(默认为90秒， eureka.instance.lease-expiration-duration-in-seconds  进行自定义配置)。当 Eureka Server 节点在短时间内丢失过多的心跳时，那么这个节点就会进入自我保护模式。

Eureka的集群中，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：

1. Eureka不再从注册表中移除因为长时间没有收到心跳而过期的服务；
2. Eureka仍然能够接受新服务注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用)；
3. 当网络稳定时，当前实例新注册的信息会被同步到其它节点中；

因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像Zookeeper那样使得整个注册服务瘫痪。

**Nacos**

最后来说Nacos。Nacos是阿里开源的，Nacos 支持基于 DNS 和基于 RPC 的服务发现。在Spring  Cloud中使用Nacos，只需要先下载 Nacos 并启动 Nacos server，Nacos只需要简单的配置就可以完成服务的注册发现。

Nacos除了服务的注册发现之外，还支持动态配置服务。动态配置服务可以让您以中心化、外部化和动态化的方式管理所有环境的应用配置和服务配置。动态配置消除了配置变更时重新部署应用和服务的需要，让配置管理变得更加高效和敏捷。配置中心化管理让实现无状态服务变得更简单，让服务按需弹性扩展变得更容易。

Nacos支持CP+AP模式，即Nacos可以根据配置识别为CP模式或AP模式，默认是AP模式。如果注册Nacos的client节点注册时ephemeral=true，那么Nacos集群对这个client节点的效果就是AP，采用distro协议实现；而注册Nacos的client节点注册时ephemeral=false，那么Nacos集群对这个节点的效果就是CP的，采用raft协议实现。根据client注册时的属性，AP，CP同时混合存在，只是对不同的client节点效果不同。Nacos可以很好的解决不同场景的业务需求。

一句话概括就是Nacos = Spring Cloud注册中心 + Spring Cloud配置中心。

在大多数分布式环境中，尤其是涉及到数据存储的场景，数据一致性应该是首先被保证的，这也是 Zookeeper  设计紧遵CP原则的重要原因。但是对于服务发现来说，情况就不太一样了，针对同一个服务，即使注册中心的不同节点保存的服务提供者信息不尽相同，也并不会造成灾难性的后果。因为对于服务消费者来说，能消费才是最重要的，消费者虽然拿到可能不正确的服务实例信息后尝试消费一下，也要胜过因为无法获取实例信息而不去消费，导致系统异常要好。从这一点上来说，分布式服务注册中心AP模式要更优于CP模式。这也是为什么越来越多的人抛弃Zookeeper的原因所在。

总之，服务注册中心到底选用CP模型还是AP模型，还是要依照业务场景进行决定，如果对数据一致性要求较高，且可以容忍一定时间的不可用，就选用CP模型。反之，如果可以容忍一定时延的数据不一致性，但不能容忍不可用显现发生，则要选用AP模型。

#### 10.雪花算法 具体点； ####

​	SnowFlake 算法，是 Twitter 开源的分布式 id 生成算法。其核心思想就是：使用一个 64 bit 的 long 型的数字作为全局唯一 id。在分布式系统中的应用十分广泛，且ID 引入了时间戳，基本上保持自增的，后面的代码中有详细的注解。

 

这 64 个 bit 中，其中 1 个 bit 是不用的，然后用其中的 41 bit 作为毫秒数，用 10 bit 作为工作机器 id，12 bit 作为序列号。

 

![img](https://img-blog.csdnimg.cn/2019050514590789.png)

给大家举个例子吧，比如下面那个 64 bit 的 long 型数字：

- 第一个部分，是 1 个 bit：0，这个是无意义的。
- 第二个部分是 41 个 bit：表示的是时间戳。
- 第三个部分是 5 个 bit：表示的是机房 id，10001。
- 第四个部分是 5 个 bit：表示的是机器 id，1 1001。
- 第五个部分是 12 个 bit：表示的序号，就是某个机房某台机器上这一毫秒内同时生成的 id 的序号，0000 00000000。

 

**①1 bit：是不用的，为啥呢？**

 

因为二进制里第一个 bit 为如果是 1，那么都是负数，但是我们生成的 id 都是正数，所以第一个 bit 统一都是 0。

 

**②41 bit：表示的是时间戳，单位是毫秒，当前时间 - 开始时间。**

 

41 bit 可以表示的数字多达 2^41 - 1，也就是可以标识 2 ^ 41 - 1 个毫秒值，换算成年就是表示 69 年的时间。

 

**③10 bit：记录工作机器 id，代表的是这个服务最多可以部署在 2^10 台机器上，也就是 1024 台机器。**

 

但是 10 bit 里 5 个 bit 代表机房 id，5 个 bit 代表机器 id。意思就是最多代表 2 ^ 5 个机房（32 个机房），每个机房里可以代表 2 ^ 5 个机器（32 台机器），也可以根据自己公司的实际情况确定。

 

**④12 bit：这个是用来记录同一个毫秒内产生的不同 id。**

 

12 bit 可以代表的最大正整数是 2 ^ 12 - 1 = 4096，也就是说可以用这个 12 bit 代表的数字来区分同一个毫秒内的 4096 个不同的 id。

 

简单来说，你的某个服务假设要生成一个全局唯一 id，那么就可以发送一个请求给部署了 SnowFlake 算法的系统，由这个 SnowFlake 算法系统来生成唯一 id。

 

这个 SnowFlake 算法系统首先肯定是知道自己所在的机房和机器的，比如机房 id = 17，机器 id = 12。

 

接着 SnowFlake 算法系统接收到这个请求之后，首先就会用二进制位运算的方式生成一个 64 bit 的 long 型 id，64 个 bit 中的第一个 bit 是无意义的。

 

接着 41 个 bit，就可以用当前时间戳（单位到毫秒），然后接着 5 个 bit 设置上这个机房 id，还有 5 个 bit 设置上机器 id。

 

最后再判断一下，当前这台机房的这台机器上这一毫秒内，这是第几个请求，给这次生成 id 的请求累加一个序号，作为最后的 12 个 bit。

 

最终一个 64 个 bit 的 id 就出来了，类似于：

 

![img](https://img-blog.csdnimg.cn/20190505145942391.png)

这个算法可以保证说，一个机房的一台机器上，在同一毫秒内，生成了一个唯一的 id。可能一个毫秒内会生成多个 id，但是有最后 12 个 bit 的序号来区分开来。

 

下面我们简单看看这个 SnowFlake 算法的一个代码实现，这就是个示例，大家如果理解了这个意思之后，以后可以自己尝试改造这个算法。

 

```java
public class IdWorker {
	//因为二进制里第一个 bit 为如果是 1，那么都是负数，但是我们生成的 id 都是正数，所以第一个 bit 统一都是 0。

	//机器ID  2进制5位  32位减掉1位 31个
	private long workerId;

	//机房ID 2进制5位  32位减掉1位 31个
	private long datacenterId;

	//代表一毫秒内生成的多个id的最新序号  12位 4096 -1 = 4095 个
	private long sequence;

	//设置一个时间初始值    2^41 - 1   差不多可以用69年
	private long twepoch = 1585644268888L;

	//5位的机器id
	private long workerIdBits = 5L;

	//5位的机房id
	private long datacenterIdBits = 5L;

	//每毫秒内产生的id数 2 的 12次方
	private long sequenceBits = 12L;

	// 这个是二进制运算，就是5 bit最多只能有31个数字，也就是说机器id最多只能是32以内
	private long maxWorkerId = -1L ^ (-1L << workerIdBits);

	// 这个是一个意思，就是5 bit最多只能有31个数字，机房id最多只能是32以内
	private long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
	private long workerIdShift = sequenceBits;
	private long datacenterIdShift = sequenceBits + workerIdBits;
	private long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;
	private long sequenceMask = -1L ^ (-1L << sequenceBits);

	//记录产生时间毫秒数，判断是否是同1毫秒
	private long lastTimestamp = -1L;
	public long getWorkerId(){

		return workerId;
	}


	public long getDatacenterId() {
		return datacenterId;
	}
	public long getTimestamp() {

		return System.currentTimeMillis();

	}
	public IdWorker(long workerId, long datacenterId, long sequence) {
		// 检查机房id和机器id是否超过31 不能小于0
		if (workerId > maxWorkerId || workerId < 0) {
			throw new IllegalArgumentException(
					String.format("worker Id can't be greater than %d or less than 0",maxWorkerId));
		}
		if (datacenterId > maxDatacenterId || datacenterId < 0) {
			throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0",maxDatacenterId));
		}
		this.workerId = workerId;
		this.datacenterId = datacenterId;
		this.sequence = sequence;

	}

	// 这个是核心方法，通过调用nextId()方法，让当前这台机器上的snowflake算法程序生成一个全局唯一的id

	public synchronized long nextId() {

		// 这儿就是获取当前时间戳，单位是毫秒
		long timestamp = timeGen();

		if (timestamp < lastTimestamp) {

			System.err.printf("clock is moving backwards. Rejecting requests until %d.", lastTimestamp);

			throw new RuntimeException(
					String.format("Clock moved backwards. Refusing to generate id for %d milliseconds",lastTimestamp - timestamp));
		}

		// 下面是说假设在同一个毫秒内，又发送了一个请求生成一个id
		// 这个时候就得把seqence序号给递增1，最多就是4096
		if (lastTimestamp == timestamp) {
			// 这个意思是说一个毫秒内最多只能有4096个数字，无论你传递多少进来，
			//这个位运算保证始终就是在4096这个范围内，避免你自己传递个sequence超过了4096这个范围
			sequence = (sequence + 1) & sequenceMask;

			//当某一毫秒的时间，产生的id数 超过4095，系统会进入等待，直到下一毫秒，系统继续产生ID
			if (sequence == 0) {
				timestamp = tilNextMillis(lastTimestamp);
			}

		} else {
			sequence = 0;

		}

		// 这儿记录一下最近一次生成id的时间戳，单位是毫秒
		lastTimestamp = timestamp;
       
		// 这儿就是最核心的二进制位运算操作，生成一个64bit的id
		// 先将当前时间戳左移，放到41 bit那儿；将机房id左移放到5 bit那儿；将机器id左移放到5 bit那儿；将序号放最后12 bit
		// 最后拼接起来成一个64 bit的二进制数字，转换成10进制就是个long型
		return ((timestamp - twepoch) << timestampLeftShift) |
				(datacenterId << datacenterIdShift) |
				(workerId << workerIdShift) | sequence;
	}

	/**
	 * 当某一毫秒的时间，产生的id数 超过4095，系统会进入等待，直到下一毫秒，系统继续产生ID
	 * @param lastTimestamp
	 * @return
	 */
	private long tilNextMillis(long lastTimestamp) {
		long timestamp = timeGen();
		while (timestamp <= lastTimestamp) {
			timestamp = timeGen();
		}
		return timestamp;
	}

	//获取当前时间戳
	private long timeGen(){
		return System.currentTimeMillis();
	}

	/**
	 *  main 测试类
	 * @param args
	 */



	public static void main(String[] args) {

		System.out.println(1&4596);

		System.out.println(2&4596);

		System.out.println(6&4596);

		System.out.println(6&4596);


		System.out.println(6&4596);


		System.out.println(6&4596);
//		IdWorker worker = new IdWorker(1,1,1);

//		for (int i = 0; i < 22; i++) {

//			System.out.println(worker.nextId());

//		}
	}
}
```

总之就是用一个 64 bit 的数字中各个 bit 位来设置不同的标志位，区分每一个 id。

**SnowFlake算法的优点：**


 （1）高性能高可用：生成时不依赖于数据库，完全在内存中生成。

（2）容量大：每秒中能生成数百万的自增ID。

（3）ID自增：存入数据库中，索引效率高。

 

**SnowFlake算法的缺点：**


 依赖与系统时间的一致性，如果系统时间被回调，或者改变，可能会造成id冲突或者重复。

#### 11.rocketmq负载均衡怎么实现的； ####

