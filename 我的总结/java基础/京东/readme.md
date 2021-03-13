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

 ## 讲一下string、stringbuffer、stringbuilder的区别 

**可变性**

简单的来说：`String` 类中使用 final 关键字修饰字符数组来保存字符串，`private final char value[]`，所以`String` 对象是不可变的。

> 补充（来自[issue 675](https://github.com/Snailclimb/JavaGuide/issues/675)）：在 Java 9 之后，String 、`StringBuilder` 与 `StringBuffer` 的实现改用 byte 数组存储字符串 `private final byte[] value`

而 `StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串`char[]value` 但是没有用 `final` 关键字修饰，所以这两种对象都是可变的。

`StringBuilder` 与 `StringBuffer` 的构造方法都是调用父类构造方法也就是`AbstractStringBuilder` 实现的，大家可以自行查阅源码。

```
AbstractStringBuilder.java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;

    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }}
```

**线程安全性**

`String` 中的对象是不可变的，也就可以理解为常量，线程安全。**`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。**

**性能**

每次对 `String` 类型进行改变的时候，都会生成一个新的 `String` 对象，然后将指针指向新的 `String` 对象。`StringBuffer` 每次都会对 `StringBuffer` 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 `StringBuilder` 相比使用 `StringBuffer` 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

**对于三者使用的总结：**

1. 操作少量的数据: 适用 `String`
2. 单线程操作字符串缓冲区下操作大量数据: 适用 `StringBuilder`
3. 多线程操作字符串缓冲区下操作大量数据: 适用 `StringBuffer`

##  15、讲一下=、==、equals 

**`=`** : 是赋值符号，a=b就是将b赋给a。

**`==`** : 它的作用是判断两个对象的地址是不是相等。即判断两个对象是不是同一个对象。(**基本数据类型==比较的是值，引用数据类型==比较的是内存地址**)

**`equals()`** : 它的作用也是判断两个对象是否相等，它不能用于比较基本数据类型的变量。`equals()`方法存在于`Object`类中，而`Object`类是所有类的直接或间接父类。

`Object`类`equals()`方法：

```
public boolean equals(Object obj) {
     return (this == obj);
}
```

`equals()` 方法存在两种使用情况：

- 情况 1：类没有覆盖 `equals()`方法。则通过`equals()`比较该类的两个对象时，等价于通过“==”比较这两个对象。使用的默认是 `Object`类`equals()`方法。
- 情况 2：类覆盖了 `equals()`方法。一般，我们都覆盖 `equals()`方法；实现若它们的内容相等，则返回 true(即，认为这两个对象相等)。

 ## 16、讲一下java异常 

异常(Exception)是Java语言提出的一种错误报告模型，这种错误报告模型在程序和客户端之间传递异常问题。
使用异常处理(错误报告模型)的好处显而易见：

* 无论何时，代码都能可靠运行，即使发生异常，程序也能执行而不是停止；
* 异常处理使代码的阅读、编写和调试工作更加方便。

2.异常的种类有哪些？

在Java程序运行期间，由于Java程序导致JVM发生错误的BUG用Error对象来表示，此类BUG发生在运行期且与JVM有关，所以一旦发生将不可恢复，即Java程序停止运行。

在Java语言中，用Throwable对象作为Exception对象和Error对象的父类。其关系如图所示：
![11](https://img-blog.csdnimg.cn/20190708012034555.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vYm9keV8x,size_16,color_FFFFFF,t_70)
对于Java程序自身导致的BUG用Exception对象表示，Exception对象在Java程序中是可以提前捕获并处理掉的（这个功能由try-catch-finally完成），以此避免因为一个小BUG导致整个系统停止运行。

Java语言把编译时可能产生的异常称为受检查异常，把运行时可能产生的异常称为不受检查异常。
受检查异常是在编译时，由编译器检测出Java程序可能会抛出的异常；



##  17、讲一下基本数据类型和包装类型的关系 

关系：

基本类型并不具有对象的性质，为了让基本类型也具有对象的特征，就出现了包装类型（如我们在使用集合类型Collection时就一定要使用包装类型而非基本类型），它相当于将基本类型“包装起来”，使得它具有了对象的性质，并且为其添加了属性和方法，丰富了基本类型的操作。

另外当需要往ArrayList，HashMap中放东西时，像int，double这种基本类型是放不进去的，因为容器都是装object的，这是就需要这些基本类型的包装器类了。

二者可以相互转换。

区别：

1、包装类是对象，拥有方法和字段，对象的调用都是通过引用对象的地址，基本类型不是
2、包装类型是引用的传递，基本类型是值的传递
3、声明方式不同，基本数据类型不需要new关键字，而包装类型需要new在堆内存中进行new来分配内存空间
4、存储位置不同，基本数据类型直接将值保存在值栈中，而包装类型是把对象放在堆中，然后通过对象的引用来调用他们
5、初始值不同，eg： int的初始值为 0 、 boolean的初始值为false 而包装类型的初始值为null
6、使用方式不同，基本数据类型直接赋值使用就好 ，而包装类型是在集合如 coolection Map时会使用

##  18、讲一下你知道的修饰符 

1、pubic
使用对象: 类、接口、成员。
介绍：无论所属的包定义在哪，该类（接口、成员）都是可访问的。
2、private
使用对象: 成员。
介绍: 成员只可以在定义它的类中被访问。
3、static
使用对象: 类、方法、变量、初始化函数。
介绍：static修辞的内部类是一个项级类，它和类包含的成员是不相关的。静态方法是类方法，被指向到所属的类面不是类的实例。静态变量是类变量，无论该变量所在的类创建了多少实例，该变量只存在一个实例被指向到所属的类而不是类的实例。初始化函数是
在装载类时执行的，面不是在创建实例时执行的。
4、final
使用对象：类、方法、变量。
介绍：被定义成final的类不允许出现子类，不能被覆盖(不应用于动态查询)，变量值不允许被修改。
5、abstract
使用对象：类、接口、方法。
介绍：abstract类中包括没有实现的方法。不能被实例化。abstract 方法的方法体为空
该方法的实现在子类中被定义，并且包含一个abstract方法的类必须是一个abstact类。
6、protected
使用对象: 成员
介绍：protected 成员只能在定义它的包中被访问，如果在其他包中被访问，则实现这个
方法的类必须是该成员所属类的子类。
7、native
使用对象: 成员。
介绍: 与操作平台相关，定义时并不定义其方法，**方法被个外部的库实现。**
8、synchronized
使用对象: 静态方法、非静态方法、代码块、一个对象、一个类。。
介绍: 不管修饰的是谁，锁的都是synchronized 关键字后面的大括号包起来的部分。这一部分是不能被多线程同时访问的。
哪怕修饰的是对象或者类，只要多个线程执行的时候，不同时执行到对象或类中synchronized 修饰的部分，就可以执行。也就是说，给对象或者类加了锁，同时synchronized修饰的代码块和没有修饰的代码块都是可以的。

9、volatile
使用对象：变量。
介绍：因为异步线程可以访问变量，所以有些优化操作是一定不能作用在变量上的。
volatile有时可以代替synchronized.
10、transient
使用对象: 变量。
介绍。变量不是对象持久状态的一部分，不应该把变量和对象一起串起

## 19、final和static的区别 

### **final** 表示最终的，不可变的

final 可以修饰-类，方法，变量

1. 修饰 **类** -表示类不可变，不可继承，比如 **String** 具有不可变性
2. 修饰 **方法** -表示该方法不可重写，比如 **模板方法**，可以用来固定算法
3. 修饰 **变量** -表示该变量在编译后成为一个 **常量**，不可以被修改

**注意：**

修饰**基本数据类型**，**值本身** 不能被改变

修饰的是**引用类型**，**句柄本身** 或者说 **引用的指向** 是不可变，但对象里面的属性可以改变

### **static** 表示全局的 or 静态的

static 可以修饰-方法，变量，代码块

在JVM中，被static修饰的方法和变量存放在方法区中（JDK8称为元空间），它属于全局共享的，而不是某个线程私有的。

也就是独立于该类中的任何对象，它不依赖于类的特定实例（对象），被类的所有实例共享，可以直接用类名调用类中的任何方法和变量。

![img](https://pic3.zhimg.com/80/v2-4e20f3b8e2409828251243b67f5da9e6_720w.jpg)