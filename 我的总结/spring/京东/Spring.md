### 1.Bean生命周期 

![5496407](images\5496407.jpg)

- Bean 容器找到配置文件中 Spring Bean 的定义。

- Bean 容器利用 Java Reflection API 创建一个Bean的实例。

- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。

- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入Bean的名字。（BeanNameAware让Bean获取自己在BeanFactory配置中的名字）

- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。（BeanClassLoaderAware让受管Bean本身知道它是由哪一类装载器负责装载的）

- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。

- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法

- 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。

- 如果 Bean 在配置文件中的定义包含  init-method 属性，执行指定的方法。

- 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法

- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。

- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

  ### 2.applicationcontext与beanfactory的区别 

（1）BeanFactory：是Spring里面最底层的接口，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。ApplicationContext接口是BeanFactory的子接口，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：

①继承MessageSource，因此支持国际化。

②统一的资源文件访问方式。

③提供在监听器中注册bean的事件。

④同时加载多个配置文件。

⑤载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

（2）①BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常。

（3）BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader。

（4）BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。

## 10.你是怎么实现Bean的？在Spring里，Bean的生命周期知道吗？

实现BEAN可以通过@bean注解，或者xml进行配置。

bean的生命周期：

* 根据配置文件读取bean的定义信息。
* 利用java 反射创建bean实例
* 如果设计一些属性值，使用set（）方法设置属性值
* 如果实现了*Aware接口，则调用相应的方法
* 如果有和加载bean的spring容器相关的beanpostprocessor对象，调用postprocessorbeforeinitilaztion（）方法
* 如果实现了initialazingbean接口，执行afterpropertitesset（）方法
* 如果配置了init-method属性，执行相应的方法
* 如果有何加载bean的spring容器相关的postbeanprocessor对象，调用postprocessorafterinitilaztion（）方法
* 如果实现了destoryablebean接口，执行destory方法
* 如果配置了destory-method属性，需要执行相应的方法

## 讲一下AOP 

AOP：面向切面编程，是spring得一个特性。是将与业务无关却又被业务所调用的模块封装起来，减少代码的重复。降低程序间的耦合，提高代码的复用性，有利于代码的可扩展性和可维护性。

springAOP是基于动态代理的，使用JDK的proxy进行代理，如果类实现了某接口，则可以通过实现接口的方式进行对类的增强。

## 11.那你是怎么实现AOP的？

aop可以在不影响原有功能的条件下，实现横向的扩展功能。比如控制层（Controller）->业务层（Service）->数据层（dao）,那么从这个结构下来的为纵向，它具体的某一层就是我们所说的横向。我们的AOP就是可以作用于这某一个横向模块当中的所有方法。

当我们需要为多个对象引入一个公共行为，比如日志，操作记录等，就需要在每个对象中引用公共行为，这样程序就产生了大量的重复代码，使用AOP可以完美解决这个问题。

 ### 3.如何自动装配

 Spring  Boot启动的时候会通过@EnableAutoConfiguration注解找到META-INF/spring.factories配置文件中的所有自动配置类，并对其进行加载，这些自动配置类都是以AutoConfiguration结尾来命名的，它实际上就是一个JavaConfig形式的Spring容器配置类，通过@Bean导入到Spring容器中，以Properties结尾命名的类是和配置文件进行绑定的。它能通过这些以Properties结尾命名的类中取得在全局配置文件中配置的属性，我们可以通过修改配置文件对应的属性来修改自动配置的默认值，来完成自定义配置

  ### 4.前端发起请求spring交互流程

 ![49790288](images\49790288.png)

  ### 5.常用注解与核心注解 

* 1：@SpringbootBootApplication

@SpringBootApplication是springboot中最核心的注解，写在启动类的上面。它是@Configuration、@EnableAutoConfiguration和@ComponentScan的组合注解。

@Configuration指示一个类声明一个或者多个@Bean 声明的方法并且由Spring容器管理，@EnableAutoConfiguration将SpringBoot中所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器，

@ComponentScan扫描定义路径下的bean。

* 2：@ImportResource

用来加载xml配置文件

* 3：@AutoWired

自动导入依赖的bean。

* 4：@Value

注入springboot中application.properties配置文件中配置的属性值。

* 5：@RestController：

主要作用于Controller的类上，它是@Controller和@ResponseBody的组合注解，主要用于返回json数据。

* 6：@ResponseBody

主要作用于控制层的类上，主要用于返回json数据。

* 7：@Data

主要作用于实体类上，编译后可以自动加上get、set、toString、equals方法等，减少我们实体类代码的书写，增加可阅读性。

* 8：@Bean

用@Bean标注方法等价于XML中配置的bean。放在方法上面，而不是类，产生一个bean，交给spring管理。

* 9：@RequestMapping

主要作用于Controller类及方法上，主要作用是请求地址的映射，当然，其中还有method属性等，method属性主要是请求类型，比如post、get等，value = RequestMethod.GET。

* 10：@Mapper

主要作用于DAO接口上，可以自动生成接口的实现类。

* 11：@Id

表示该属性为主键

* 12：@MapperScan

主要作用于启动类上，用于生成DAO接口的实现类，如果DAO接口比较多，推荐使用@MapperScan注解，写法如@MapperScan("com.example.demo.dao").

* 13：@PathVariable

主要是用于取url中的变量的值，比如 @RequestMapping("/student/{studentName}")，那么在对应的方法入参中可以写成：(@PathVariable  String  studentName).

* 14：@RequestParam

将请求参数绑定到Controller的方法上面，@RequestParam(value=”参数名”)。

 1.spring如何保证线程安全 

在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域。就是因为Spring对一些Bean(如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等)中非线程安全状态采用ThreadLocal进行处理，让它们也成为线程安全的状态，因为有状态的Bean就可以在多线程中共享了。

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。

怎么保证spring中controller的线程安全

1、不要在controller中定义成员变量。2、万一必须要定义一个非静态成员变量时候，则通过注解@Scope(“prototype”)，将其设置为多例模式。3、在Controller中使用ThreadLocal变量

 ### 1.spring如何保证线程安全 

在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域。就是因为Spring对一些Bean(如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等)中非线程安全状态采用ThreadLocal进行处理，让它们也成为线程安全的状态，因为有状态的Bean就可以在多线程中共享了。

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。

怎么保证spring中controller的线程安全

1、不要在controller中定义成员变量。2、万一必须要定义一个非静态成员变量时候，则通过注解@Scope(“prototype”)，将其设置为多例模式。3、在Controller中使用ThreadLocal变量

  ### 2.防止注入 

​	解决思路：对用户输入的内容进行合法校验

​	方案一：可以使用PreparedStatement
 	方案二：使用正则表达式

  ### 3.mybatis的作用 

MyBatis是持久层框架，它是支持JDBC的！简化了持久层开发！

使用MyBatis时，只需要通过接口指定数据操作的抽象方法，然后配置与之关联的SQL语句，即可完成！

  ### 4.xml作用 

首先使用xml配置文件的好处是参数配置项与代码分离，便于管理以及日后的维护和修改。
其次，xml是标准化的树节点文档，通用性强。

  ### 5.引申懒汉和饿汉相关 

懒汉：在初始化类的时候，不创建唯一的实例，而是等到真正需要用到的时候才创建。必须加上同步，否则有可能依然创建多个实例。 
 饿汉：在初始化的时候，就创建了唯一的实例，不管是否需要用到。不需要自己加同步，一定产生唯一的实例。 

 ## spring中@Autowire和@Resource的区别 

| 对比项   | @Autowire                | @Resource                          |
| -------- | ------------------------ | ---------------------------------- |
| 注解来源 | Spring注解               | JDK注解(JSR-250标准注解，属于J2EE) |
| 装配方式 | 优先按类型               | 优先按名称                         |
| 属性     | required                 | name、type                         |
| 作用范围 | 字段、setter方法、构造器 | 字段、setter方法                   |

## @RequestBody和@ResponseBody的区别 

**@Responsebody **注解表示该方法的返回的结果直接写入 HTTP 响应正文（ResponseBody）中，一般在异步获取数据时使用，通常是在使用 @RequestMapping 后，返回值通常解析为跳转路径，加上 @Responsebody 后返回结果不会被解析为跳转路径，而是直接写入HTTP 响应正文中。
作用：
该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。
使用时机：
返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用；

**@RequestBody **注解则是将 HTTP 请求正文插入方法中，使用适合的 HttpMessageConverter 将请求体写入某个对象。

作用：

1) 该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上；
2) 再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。

使用时机：

A) GET、POST方式时， 根据request header Content-Type的值来判断:
application/x-www-form-urlencoded， 可选（即非必须，因为这种情况的数据@RequestParam, @ModelAttribute也可以处理，当然@RequestBody也能处理）；
multipart/form-data, 不能处理（即使用@RequestBody不能处理这种格式的数据）；
其他格式， 必须（其他格式包括application/json, application/xml等。这些格式的数据，必须使用@RequestBody来处理）；
B) PUT方式提交时， 根据request header Content-Type的值来判断:

application/x-www-form-urlencoded， 必须；multipart/form-data, 不能处理；其他格式， 必须；

说明：**request的body部分的数据编码格式由header部分的Content-Type指定；**


 ## @Component,@Service和@ Repository @transactional，

![3130b79fbe58995cc9cba54a06e89c78.png](https://img-blog.csdnimg.cn/img_convert/3130b79fbe58995cc9cba54a06e89c78.png)

这几个注解几乎可以说是一样的：因为被这些注解修饰的类就会被Spring扫描到并注入到Spring的bean容器中。

这里，有两个注解是不能被其他注解所互换的：

@Controller 注解的bean会被spring-mvc框架所使用。
@Repository 会被作为持久层操作(数据库)的bean来使用
下面三个注解被用于为我们的应用进行分层：

* @Controller注解类进行前端请求的处理，转发，重定向。包括调用Service层的方法
* @Service注解类处理业务逻辑
* @Repository注解类作为DAO对象(数据访问对象，Data Access Objects)，这些类可以直接对数据库进行操



**@transactional**

**1.添加的位置：**

添加在接口的实现类上，或者接口的实现方法上。
 **ps1**   仅限于 public 方法才能生效
 **ps2** 将注解放在需要进行事务管理的地方，只读方法不必进行事务管理----> --- 是因为该注解的底层是aop拦截和事务处理，在一定程度上会影响相应的性能。

**2. 用事务注解 须注意之处：**

**ps1** ：现在有方法a 没有加事务注解， 方法b加类事务注解， 前端请求先调用方法a，方法a中又调用了方法b ---> 此时 ---> 事务不生效 ；
 **ps2** ： 一个方法上加了事务的注解，但在方法体 捕获异常 为抛出 --> 此时 --> 事务不回滚 。（只有抛出 error或者 exception  事务才会进行回滚） ;
 **ps3**: 在方法体中，多线程执行时，事务不生效；

@Transactional(isolation = Isolation.READ_UNCOMMITTED)：

@Transactional(isolation = Isolation.READ_COMMITTED)

@Transactional(isolation = Isolation.REPEATABLE_READ)

@Transactional(isolation = Isolation.SERIALIZABLE) 



在要开启事务的方法上使用`@Transactional`注解即可!

```java
@Transactional(rollbackFor = Exception.class)
public void save() {
  ......
}
Copy to clipboardErrorCopied
```

我们知道 Exception 分为运行时异常 RuntimeException 和非运行时异常。在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事物**只会在遇到`RuntimeException`的时候才会回滚**,加上`rollbackFor=Exception.class`,可以让事物在遇到非运行时异常时也回滚。

`@Transactional` 注解一般用在可以作用在`类`或者`方法`上。

- **作用于类**：当把`@Transactional 注解放在类上时，表示所有该类的`public 方法都配置相同的事务属性信息。
- **作用于方法**：当类配置了`@Transactional`，方法也配置了`@Transactional`，方法的事务会覆盖类的事务配置信息

#### @Transactional注解有哪些属性？

##### propagation属性

`propagation` 代表事务的传播行为，默认值为 `Propagation.REQUIRED`，其他的属性信息如下：

- `Propagation.REQUIRED`：如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。**(** 也就是说如果A方法和B方法都添加了注解，在默认传播模式下，A方法内部调用B方法，会把两个方法的事务合并为一个事务 **）**
- `Propagation.SUPPORTS`：如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。
- `Propagation.MANDATORY`：如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。
- `Propagation.REQUIRES_NEW`：重新创建一个新的事务，如果当前存在事务，暂停当前的事务。**(** 当类A中的 a 方法用默认`Propagation.REQUIRED`模式，类B中的 b方法采用 `Propagation.REQUIRES_NEW`模式，然后在 a 方法中调用 b方法操作数据库，然而 a方法抛出异常后，b方法并没有进行回滚，因为`Propagation.REQUIRES_NEW`会暂停 a方法的事务 **)**
- `Propagation.NOT_SUPPORTED`：以非事务的方式运行，如果当前存在事务，暂停当前的事务。
- `Propagation.NEVER`：以非事务的方式运行，如果当前存在事务，则抛出异常。
- `Propagation.NESTED` ：和 Propagation.REQUIRED 效果一样。

##### isolation 属性

`isolation` ：事务的隔离级别，默认值为 `Isolation.DEFAULT`。

- **TransactionDefinition.ISOLATION_DEFAULT:** 使用后端数据库默认的隔离级别，**Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.**
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **TransactionDefinition.ISOLATION_SERIALIZABLE:** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

##### timeout 属性

`timeout` ：事务的超时时间，默认值为 -1。如果超过该时间限制但事务还没有完成，则自动回滚事务。

##### readOnly 属性

`readOnly` ：指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。

##### rollbackFor 属性

`rollbackFor` ：用于指定能够触发事务回滚的异常类型，可以指定多个异常类型。

##### **noRollbackFor**属性

`noRollbackFor`：抛出指定的异常类型，不回滚事务，也可以指定多个异常类型。

## 还问了spring的事务，mvc里的HandlerMapping和Handler（还好都是[项目]()中使用过的，说了一下其他经常使用的注解还结合自己的[项目]()进行了了一些扩展）

![SpringMVC运行原理](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-10-11/49790288.jpg)

handlermapping是处理器映射器，当dispatcherservlet发给它要处理的任务，handlermapping返回一系列的执行链给dispatcherservlet，dispatcherservlet进行分发，找到handleradapter，handleradapadapter找到对应的handler也就是controller，处理完请求，返回modleandview。

通过DispatcherServlet做分发，通过handlerMapping找到请求处理的方法，再找到handlerAdapter处理完返回ModelAndView,

 ## springboot都有什么优点，为什么可以快速开发，启动器里集成了哪些 

减少开发，测试时间和精力。

使用JavaConfig有助于避免使用XML。

避免大量的Maven导入和各种版本冲突。

通过提供默认值快速开始开发。

没有单独的Web服务器需要。这意味着你不再需要启动Tomcat，Glassfish 或其他任何东西。

需要更少的配置因为没有`web.xml`文件。只需添加用`@Configuration`注释的类，然后添加用`@Bean`注释的方法，Spring将自动加载对象并像以前一样对其进行管理。您甚至可以将`@Autowired`添加到bean方法中，以使Spring自动装入需要的依赖关系中。

基于环境的配置使用这些属性，您可以将您正在使用的环境传递到应用程序：`-Dspring.profiles.active={enviornment}`。在加载主应用程序属性文件后，Spring将在（application（environment}.properties）中加载后续的应用程序属性文件。

在开发过程中，如果对之前的代码做出了修改，不需要重启服务器，只需要重新加载spring boot上的修改即可。

在测试过程中，不需要添加各种的测试库，只需要添加spring-boot-starter-test就可以进行测试。

**starters：启动器**。常见的 Starters 有以下几个：

- spring-boot-starter-test
- spring-boot-starter-web
- spring-boot-starter-data-jpa
- spring-boot-starter-thymeleaf

## 28.AOP原理，IOC原理。循环依赖。CGLIB,JDK proxy，讲了底层缓存，之间的区别。 

  IOC:IOC是一种软件设计思想，将原来程序中创建对象的控制权交给Spring框架来管理。IOC在别的框架中也有使用，并非是spring独有的。IOC实际上就是一个map（key，value）map中存放的是各种对象。

将对象之间的依赖关系交给ioc容器来管理，并由框架完成对象的注入，大大简化了应用开发，把应用从复杂的依赖关系中解放出来。IOC容器就像一个工厂一样，当我们需要对象的时候，只需要配置好配置文件/注解，完全不用考虑对象是如何生成的。在实际项目中，一个service有成百上千个类，如果要实例化这个service，可能需要搞清楚这个service所有底层类的构造函数。现在使用IOC，只需要配置好，然后在需要的地方引用就可以了。

AOP：面向切面编程，将与业务无关的，却被业务模块所共同调用的逻辑或功能封装起来，以减少重复的代码，降低程序间的耦合，并且有利于程序的可扩展性和可维护性。

Spring AOP是基于动态代理的，如果一个类实现了某个接口，就可以可以使用JDK Proxy代理对象，对于没有实现某个接口的类，会使用CGlib去创建这个类的子类，完成对这个类的代理。

当然也可以使用Aspect J进行代理，Aspect J是Java生态圈中最完整的AOP框架。AspectJ 是编译时增强的，Spring aop是运行时增强的。 

循环依赖：循环依赖是指在ioc容器中创建对象时，对象A依赖对象B，对象B依赖对象A，而A,B两个对象都没有实例化出来，在实例化的时候会产生错误。在spring中，使用构造方法完成依赖注入是不支持循环依赖的。使用set方法执行是支持循环依赖的。在创建A过程中发现依赖于B对象，就去调用getbean方法获取B的bean，如果当前的容器中没有B的bean就去创建B的bean，在创建B的bean的过程中，发现依赖于A，此时如果开启支持循环依赖，就会去一级缓存中找A，如果找不到就去二级缓存中找，找不到再去三级缓存中找，因为A已经被实例化，只是没有经过初始化赋值操作，所以会被暴露在三级缓存中，我们可以从三级缓存中拿到A的bean，A就进入了二级缓存，B就继续完成创建，之后返回给A，A完成创建。

CGlib：是一种完成代理的方式，在编译时通过修改字节码完成功能增强，可以应用于实现了接口的类，也可以应用于没有实现接口的类。对于实现接口的类，可以在spring中进行设置，强制其使用CGlib方式进行代理。这个方法是通过生成一个该类的子类的方式完成代理。

JDK proxy：只能作用于实现了接口的类，通过实现接口的方式对该类进行增强。具体的示意如下：

![SpringAOPProcess](https://camo.githubusercontent.com/2948f9b2b5c45eb208990afcac1bf5638783dc3ebc834d3d7d2d4f7cc050981c/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f537072696e67414f5050726f636573732e6a7067)s