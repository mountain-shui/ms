### 一面 

 ###  **在浏览器地址栏输入一个地址**  

1. 浏览器解析地址，从浏览器缓存、本地缓存、DNS缓存中查找这个地址所对应的IP地址
2. 本机和对应ip地址的服务器建立连接，发送请求
3. 服务器处理请求
4. 回传html到浏览器
5. 浏览器渲染html
6. 可以选择中断连接

其中使用了DNS协议、ospf协议、tcp/ip协议、arp协议、http协议

DNS协议就是将url地址转换成ip地址的协议

ospf协议是将数据包在路由器之间传递的协议

tcp/IP协议是将主机和服务器建立连接的协议

arp是将ip地址转换成mac地址的协议

http协议是在建立好tcp协议之后，使用这个协议访问网页

   **其余八股文忘了问的什么了，写了两个简单[算法]()和一个sql题目**  


###  **[链表]()反转** 





### **查找[数组中出现次数超过一半的数字]()** 



###   **两个表查询所有成绩都大于80分的课程名，和用户名，用户id**  

​		select coursename, username, userid from tab_1,tab_2 

​		where tab_1.userid = tab_2.userid

   **设计微信发红包的测试用例**



###  二面 

 **自我介绍，介绍[项目]()** 

###  spring IOC和AOP 

spring ioc：控制反转，意思是将创建对象的控制权交给spring来管理，将程序员类与类之间复杂的依赖关系中解放出来，ioc就是一个map，其中key是bean的名称，value是bean。它的好处是可以很方便的解耦，简化操作，使项目的可扩展性和可维护性提高。

spring aop：面向切面编程，对于传统的controller->service->dao是纵向的编程，aop是横向的。将与业务核心逻辑无关的又被核心逻辑所调用的代码封装起来。在需要的时候进行调用，增强了代码的可维护性和可扩展性，进一步对代码进行解耦。spring的aop有两种：如果实现了接口就可以使用jdk自带的proxy，如果没有实现接口，可以使用cglib。

### 依赖注入的方式 

属性注入和构造器注入

**1. 属性注入（最常用的方式）**

1.1. 通过 setter 方法注入Bean 的属性值或依赖的对象
 1.2.  使用<property>元素, 使用 name 属性指定 Bean 的属性名称，value 属性或 <value> 子节                    点指定属性值



```jsx
<bean id="hq1" class="com.chai.demo1.HelloSpring">
    <!--    
        注意:name属性的值  为当前类中setter方法的名称。  
        setName2(String name) 假如方法叫setName2 那么name="name2" 也应该填写name2
    -->
    <property name="name" value="你好..."></property>
</bean>
```

**2. 构造器注入**

2.1 通过构造方法注入Bean 的属性值或依赖的对象，它保证了 Bean 实例在实例化后就可以使用
 2.2 构造器注入在 <constructor-arg> 元素里声明属性, <constructor-arg> 中没有 name 属性



```jsx
    <bean id="p1" class="com.chai.lesson2.Person" scope="prototype">
        <constructor-arg value="小李" index="0"></constructor-arg>
        <constructor-arg value="18"></constructor-arg>
    </bean>
    <bean id="p2" class="com.chai.lesson2.Person">
        <constructor-arg value="jack" index="0"></constructor-arg>
        <constructor-arg type="int">
            <value>14</value>
        </constructor-arg>
    </bean>
```

Tip：

- value：表示你要注入的属性的值
- index 表示参数下标
- type 指定类型
- scope 属性(表示bean的作用域) 可以通过执行springIOC容器来创建对象 的类型
  - prototype 原型模式（多例模式） 每次调用IOC容器的getBean方法，就会返回一个新的对象（地址）
  - singleton  单例模式 每次调用都会返回同一个对象



### wait sleep的区别 

wait表示让线程进入等待状态，并且释放资源，没有notify或者notifiall，线程不会自动苏醒。

sleep表示让线程睡眠，在规定时间之后就会自动醒来。sleep不会释放资源，会一直持有资源。

 ### Lock，synchronized的区别  

lock是api层面的锁，synchronized 是jvm层面的锁。

lock可以配合condition使用，对符合条件的部分线程进行唤醒，synchronized不行，只可以随机唤醒。

lock可以使用公平锁和非公平锁，synchronized只能使用非公平锁。

lock可以中断等待中的线程，synchronized不行。

lock必须在finally块中手动解锁，synchronized是自动解锁。



### start 和run的区别 

* start表示开启一个线程，可以继续执行之后的代码，不用等待这个线程执行结束。
* run方法是线程执行的线程体，进入run方法，这个线程就进入了运行状态
* 主线程中调用run方法，会将这个run方法当成普通方法执行，并不会启动新的线程。调用start方法才可以启动线程，此时线程是就绪状态，并没有运行

### synchronized和volatile的区别 

synchronized可以用于类，对象，成员变量。synchronized可以保证原子性。

volatile只能作用于成员变量，volatile可以保证有序性和可见性，不能保证原子性。

volatile不会造成线程阻塞，synchronized会造成线程阻塞。

### java IO流

![这里写图片描述](https://img-blog.csdn.net/20180127210359151)

![这里写图片描述](https://img-blog.csdn.net/20180127210359151)

 输入输出流是相对于内存而言的！

1、面试题汇总
（1）java中有几种类型的流？
字符流和字节流。字节流继承inputStream和OutputStream,字符流继承自InputSteamReader和OutputStreamWriter。

（2）谈谈Java IO里面的常见类，字节流，字符流、接口、实现类、方法阻塞
答：输入流就是从外部文件输入到内存，输出流主要是从内存输出到文件。
IO里面常见的类，第一印象就只知道IO流中有很多类，IO流主要分为字符流和字节流。字符流中有抽象类InputStream和OutputStream，它们的子类FileInputStream，FileOutputStream,BufferedOutputStream等。字符流BufferedReader和Writer等。都实现了Closeable, Flushable, Appendable这些接口。程序中的输入输出都是以流的形式保存的，流中保存的实际上全都是字节文件。
java中的阻塞式方法是指在程序调用改方法时，必须等待输入数据可用或者检测到输入结束或者抛出异常，否则程序会一直停留在该语句上，不会执行下面的语句。比如read()和readLine()方法。

（3）字符流和字节流有什么区别？
要把一片二进制数据数据逐一输出到某个设备中，或者从某个设备中逐一读取一片二进制数据，不管输入输出设备是什么，我们要用统一的方式来完成这些操作，用一种抽象的方式进行描述，这个抽象描述方式起名为IO流，对应的抽象类为OutputStream和InputStream ，不同的实现类就代表不同的输入和输出设备，它们都是针对字节进行操作的。

在应用中，经常要完全是字符的一段文本输出去或读进来，用字节流可以吗？
计算机中的一切最终都是二进制的字节形式存在。对于“中国”这些字符，首先要得到其对应的字节，然后将字节写入到输出流。读取时，首先读到的是字节，可是我们要把它显示为字符，我们需要将字节转换成字符。由于这样的需求很广泛，人家专门提供了字符流的包装类。

底层设备永远只接受字节数据，有时候要写字符串到底层设备，需要将字符串转成字节再进行写入。字符流是字节流的包装，字符流则是直接接受字符串，它内部将串转成字节，再写入底层设备，这为我们向IO设别写入或读取字符串提供了一点点方便。

（4）讲讲NIO
答：看了一些文章，传统的IO流是阻塞式的，会一直监听一个ServerSocket，在调用read等方法时，他会一直等到数据到来或者缓冲区已满时才返回。调用accept也是一直阻塞到有客户端连接才会返回。每个客户端连接过来后，服务端都会启动一个线程去处理该客户端的请求。并且多线程处理多个连接。每个线程拥有自己的栈空间并且占用一些 CPU 时间。每个线程遇到外部未准备好的时候，都会阻塞掉。阻塞的结果就是会带来大量的进程上下文切换。
对于NIO，它是非阻塞式，核心类：
1.Buffer为所有的原始类型提供 (Buffer)缓存支持。
2.Charset字符集编码解码解决方案
3.Channel一个新的原始 I/O抽象，用于读写Buffer类型，通道可以认为是一种连接，可以是到特定设备，程序或者是网络的连接。

（5）递归读取文件夹的文件
package test;

import java.io.File;

/**
 * 
 * 递归读取文件夹的文件
 */
public class ListFileDemo {
    public static void listFile(String path) {
        if (path == null) {
            return;// 因为下面的new File如果path为空，回报异常
        }
        File[] files = new File(path).listFiles();
        if (files == null) {
            return;
        }
        for(File file : files) {
            if (file.isFile()) {
                System.out.println(file.getName());
            } else if (file.isDirectory()) {
                System.out.println("Directory:"+file.getName());
                listFile(file.getPath());
            } else {
                System.out.println("Error");
            }
        }
    }

    public static void main(String[] args) {
        ListFileDemo.listFile("D:\\data");

    }
}

运行结果

Hamlet.txt
Directory:ml-1m
movies.dat
ratings.dat
README
users.dat
2、Java IO总结
（1）明确源和目的。
数据source：就是需要读取，可以使用两个体系：InputStream、Reader；
数据destination：就是需要写入，可以使用两个体系：OutputStream、Writer；

（2）操作的数据是否是纯文本数据？

   如果是：
           数据source：Reader
           数据destination：Writer 
   如果不是：
           数据source：InputStream
           数据destination：OutputStream
（3）Java IO体系中有太多的对象，到底用哪个呢？
明确操作的数据设备。
数据source对应的设备：硬盘(File)，内存(数组)，键盘(System.in)
数据destination对应的设备：硬盘(File)，内存(数组)，控制台(System.out)。

记住，只要一读取键盘录入，就用这句话。
BufferedReader bufr = new BufferedReader(new InputStreamReader(System.in));
BufferedWriter bufw = new BufferedWriter(new OutputStreamWriter(System.out));


### vector和ArrayList的区别 

1） Vector的方法都是同步的(Synchronized),是线程安全的(thread-safe)，而ArrayList的方法不是，由于线程的同步必然要影响性能，因此,ArrayList的性能比Vector好。 
2） 当Vector或ArrayList中的元素超过它的初始大小时,Vector会将它的容量翻倍,而ArrayList只增加50%的大小，这样,ArrayList就有利于节约内存空间。

### HashMap和HashTable的区别 

* hashtable是线程安全的，hashmap不是线程安全的。
* hashtable不允许key为null，hashmap允许一个key为null
* hashtable初始容量为11，每次扩容为2n+1，hashmap初始容量为16，每次扩容为2倍
* hashmap底层链表在大于8时，会转换成红黑树结构，hashtable不会

### HashMap和HashSet 

* hashset底层使用hashmap
* hashmap存储键值对，hashset存储对象
* hashmap实现了map接口，hashset实现了set接口

### 线程的状态 

初始->就绪->运行->阻塞->终止

### 数组和[链表]()的区别：

* 插入和查询的时间复杂度 
* 数组使用连续的内存空间，链表使用不连续的内存空间
* 数组支持随机访问，链表不支持随机访问

### 栈和队列的区别 

栈是后进先出，队列是先进先出

### queue.remove和queue.poll的区别 

poll()和remove()都将**移除**并且返回对头，但是在poll()在队列为空时返回null，而remove()会抛出NoSuchElementException异常。

### 接口和抽象类的区别 

抽象类中可以private，protect，final等关键词修饰

接口中只能有public 

抽象类是对多个类共有的属性的抽象，接口更像是一组行为规范，规定可以执行哪些操作。

一个类可以继承一个类，但是可以实现很多接口。

所有方法在接口中不能实现，在抽象类中可以。

### **抽象类可以用final修饰吗** 

不能，抽象类是被用于继承的，final修饰代表不可修改、不可继承的。

### **[算法题]()：一个求[二叉树]()路径和的题目** 



 **设计模式**


###  **三面** 

​    **自我介绍，简单的说一下[项目]()**   

​    **面试时间不常，也没问什么八股文。因为[项目]()比较简单被面试官怼没有实际的工程经验**   

​    **最后写了一个[算法题]()：字符串中第一个出现的不重复字符串**