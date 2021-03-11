 ##  1.Java反射机制 

 ### 1.调用反射的几种方法 

```java
//1.源头：获取Class对象，用三种方式
		Phone iPhone=new Phone();
		//1.1.对象.getClass();获取对象
		Class<?> clazz1 = iPhone.getClass();
		//1.2.类.class
		 clazz1=Phone.class;
		//1.3.Class.forName("包名.类名");
		clazz1 = Class.forName("test.Phone");

//2.创建对象
		//2.1通过newInstence()
		Phone instance1 = (Phone) clazz1.newInstance();
		//2.2先调用构造器，再通过newInstence()创建
		Object instance2 = clazz1.getConstructor().newInstance();
```

 ### 2.与new的区别 

  new和反射本质上的区别，new属于静态编译，而反射属于动态编译，意思就说只有到运行时他才会去获得该对象的实例.

反射可以不用import就生成对象

一般会配合接口使用

如果不用反射，interface myobj=new classimpl（）

这样接口和实现类都要import进来

这样的话，如果实现类发生变动，需要重新编译，那这段代码也需要重新编译

如果用反射的话，实现类可以不用import进来

也能生成一个满足接口的对象出来

这样当实现类改动的时候，这段代码可以不用重新编译



##  抽象与接口，重写与重载 

  ### 1.默认构造函数能否重载重写 

首先，构造器是不能被继承的，因为每个类的类名都不相同，而构造器名称与类名相同，所以根本谈不上继承。
 又由于构造器不能继承，所以就不能被重写。但是，在同一个类中，构造器是可以被重载的。

 ## 异常 

  ### 1.异常的分类  

Exception:是程序本身的异常，可分成两类。

RuntimeException:主要由虚拟机抛出，是运行时异常。NullpointerException、ArithmeticException、ArrayIndexOutOfBoundsException。

IOException:IO异常。

  如何捕获异常 

  ### 2.OOM相关

1）什么是OOM？ OOM，全称“Out Of Memory”，翻译成中文就是“内存用完了”，来源于java.lang.OutOfMemoryError。

2）为什么会OOM？

为什么会没有内存了呢？原因不外乎有两点：

1）分配的少了：比如虚拟机本身可使用的内存（一般通过启动时的VM参数指定）太少。

2）应用用的太多，并且用完没释放，浪费了。此时就会造成内存泄露或者内存溢出。

**内存泄露：申请使用完的内存没有释放**，导致虚拟机不能再次使用该内存，此时这段内存就泄露了，因为申请者不用了，而又不能被虚拟机分配给别人用。

**内存溢出：申请的内存超出了JVM能提供的内存大小**，此时称之为溢出。

3）OOM的类型

按照JVM规范，除了程序计数器不会抛出OOM外，其他各个内存区域都可能会抛出OOM。

最常见的OOM情况有以下三种：

    java.lang.OutOfMemoryError: Java heap space ------>java堆内存溢出，此种情况最常见，一般由于内存泄露或者堆的大小设置不当引起。对于内存泄露，需要通过内存监控软件查找程序中的泄露代码，而堆大小可以通过虚拟机参数-Xms,-Xmx等修改。
    java.lang.OutOfMemoryError: PermGen space ------>java永久代溢出，即方法区溢出了，一般出现于大量Class或者jsp页面，或者采用cglib等反射机制的情况，因为上述情况会产生大量的Class信息存储于方法区。此种情况可以通过更改方法区的大小来解决，使用类似-XX:PermSize=64m -XX:MaxPermSize=256m的形式修改。另外，过多的常量尤其是字符串也会导致方法区溢出。
    java.lang.StackOverflowError ------> 不会抛OOM error，但也是比较常见的Java内存溢出。JAVA虚拟机栈溢出，一般是由于程序中存在死循环或者深度递归调用造成的，栈大小设置太小也会出现此种溢出。可以通过虚拟机参数-Xss来设置栈的大小。

4）OOM分析--heapdump

要dump堆的内存镜像，可以采用如下两种方式：

    设置JVM参数-XX:+HeapDumpOnOutOfMemoryError，设定当发生OOM时自动dump出堆信息。不过该方法需要JDK5以上版本。
    使用JDK自带的jmap命令。"jmap -dump:format=b,file=heap.bin <pid>"   其中pid可以通过jps获取。

dump堆内存信息后，需要对dump出的文件进行分析，从而找到OOM的原因。