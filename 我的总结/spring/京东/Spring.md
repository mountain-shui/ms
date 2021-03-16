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