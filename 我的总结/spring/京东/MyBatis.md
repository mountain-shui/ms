## mybatis怎么防止sql注入 

Mybatis中使用#{}可以防止SQL注入，${}并不能防止SQL注入

Mybatis实现SQL注入的原理是调用了jdbc中的PreparedStatement来进行预编译。SQL注入只能在编译过程中起作用。

 ## springaop的底层原理，讲了动态代理和cglib代理，以及[项目]()中我主要做什么，怎么做的

spring aop主要包括两种代理模式，一个是动态代理，一个是cglib代理。动态代理使用JDK中的proxy进行代理，被代理对象必须实现接口，如果没有实现接口，则必须使用cglib代理。cglib是通过修改字节码的方式，生成被代理类的子类，完成对被代理类的代理。

aop是一种设计思想，面向切面编程，是为了避免编写大量与核心业务无关的代码。比如日志，权限认定，事务管理等都可以使用aop完成。是利用横切技术，将业务之间共同调用的逻辑提取并且封装成一个可复用的模块。

项目中我使用aop进行日志的记录。每次执行请求之前，在日志文件中都会打印出url，ip，classmethod，args。

## mYbatis的sql注入 

mybatis使用#{}防止sql注入，使用${}不能防止sql注入，

#{}是调用JDBC的PreparedStatement进行预编译，sql注入只在编译过程中起作用

${}是直接进行字符串拼接，会引起SQL注入

## mybatis中sql注入问题？

mybatis提供两种方式进行参数替换（ #{} 和 ${}）
1).使用**#{}** 语法时，MyBatis 会自动生成 PreparedStatement ，使用参数绑定 ( ?) 的方式来设置值，上述两个例子等价的 JDBC 查询代码如下： 可以防止sql注入

    String sql = "SELECT * FROM users WHERE id = ?";
    PreparedStatement ps = connection.prepareStatement(sql);
    ps.setInt(1, id);

2).使用**${}**其实就是进行sql的拼接，先进行sql的拼接再执行sql语句就会出现sql注入。


    <select id="getByName" resultType="org.example.User">    
    SELECT * FROM user WHERE name = '${name}' limit 1
    </select>

如果name 值为 ’ or ‘1’='1，实际执行的语句为

    SELECT * FROM user WHERE name = '' or '1'='1' limit 1 

3).**所以mybatis推荐使用#{}来防止sql注入。**
3.同样的也会出现问题:比如 order by、column name，不能使用参数绑定
其实就是如果需要用数据库表的字段来替换的话就不能用参数绑定
比如：
　　1). sortBy 参数值为 name ，如果使用#{}替换后会成为

    ORDER BY "name"

即以字符串 “name” 来排序，而非按照 name 字段排序
　　2).所以需要使用${}来进行拼接，但是拼接就会出现sql注入：
　　代码层使用白名单的方式，限制 sortBy 允许的值，如只能为 name, email 字段，异常情况则设置为默认值 name

在 XML 配置文件中，使用 if 标签来进行判断


```html
<select id="getUserListSortBy" resultType="org.example.User">  
SELECT * FROM user  
<if test="sortBy == 'name' or sortBy == 'email'">    
order by ${sortBy}  
</if>
</select>
```

因为 Mybatis 不支持 else，需要默认值的情况，可以使用 choose(when,otherwise)


```html
<select id="getUserListSortBy" resultType="org.example.User">  
SELECT * FROM user  
<choose>    
<when test="sortBy == 'name' or sortBy == 'email'">      
order by ${sortBy}    
</when>    
<otherwise>    
  order by name    
</otherwise>    
 </choose>
 </select>
```

4).mybatis更多场景：

```html
除了 orderby之外，还有一些可能会使用到 ${} 情况，可以使用其他方法避免，如
like语句
使用mybatis的bind标签

<select id="getUserListLike" resultType="org.example.User">    
<bind name="pattern" value="'%' + name + '%'" />   
 SELECT * FROM user    WHERE name LIKE #{pattern}
 </select>

或者使用concat标签

<select id="getUserListLikeConcat" resultType="org.example.User">    
SELECT * FROM user WHERE name LIKE concat ('%', #{name}, '%')
</select>


in条件
使用 < foreach> 和 #{}

<select id="selectUserIn" resultType="com.example.User">  
SELECT * FROM user WHERE name in  
<foreach item="name" collection="nameList" open="(" separator="," close=")">        
	#{name}  
</foreach>
</select>


limit语句
直接通过#{}防止sql注入
```

## MyBatis 分页如何实现？ 

* MyBatis 使用 RowBounds 对象进行分页，它**是针对 ResultSet 结果集执行的内存分页**，而非物理分页，在sqlSession执行的时候进行分页

* 可以在 sql 内直接书写带有物理分页的参数来完成**物理分页功能**，使用limit关键字完成分页

* 也可以使用分页插件来完成物理分页，分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。

举例：`select _ from student`，拦截 sql 后重写为：`select t._ from （select \* from student）t limit 0，10`