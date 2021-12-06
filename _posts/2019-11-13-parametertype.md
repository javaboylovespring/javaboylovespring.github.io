---
sidebar:
  nav: docs-zh
title: 7.1 parameterType
tags: MyBatis
categories: MyBatis
abbrlink: parametertype
date: 2019-11-13 22:28:52
---

### 7.1 parameterType

这个表示输入的参数类型。

<!--more-->


#### 7.1.1 `$` 和 `#`

这是一个非常非常高频的面试题，虽然很简单。在面试中，如果涉及到 MyBatis，一般情况下，都是这个问题。

在 MyBatis 中，我们在 mapper 引用变量时，默认使用的是 `#`，像下面这样：

```xml
<select id="getUserById" resultType="org.javaboy.mybatis.model.User">
    select * from user where id=#{id};
</select>
```

除了使用 `#` 之外，我们也可以使用 `$` 来引用一个变量：

```xml
<select id="getUserById" resultType="org.javaboy.mybatis.model.User">
    select * from user where id=${id};
</select>
```

在旧的 MyBatis 版本中，如果使用 `$`，变量需要通过 @Param 取别名，在最新的 MyBatis 中，无论是 `#` 还是 `$`，如果只有一个参数，可以不用取别名，如下：

```java
public interface UserMapper {
    User getUserById(Integer id);
}
```

既然 `#` 和 `$` 符号都可以使用，那么他们有什么区别呢？

我们在 resources 目录下，添加 log4j.properties ，将 MyBatis 执行时的 SQL 打印出来：

```properties
log4j.rootLogger=DEBUG,stdout
log4j.logger.org.mybatis=DEBUG
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p %d %C: %m%n 
```

然后添加日志依赖：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.5</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.5</version>
</dependency>
```

然后，我们可以分别观察 `$` 和 `#` 执行时的日志：

![](http://mybatis.javaboy.org/assets/images/img/7-1-1.png "7-1-1.png")

上面这个日志，是 `$` 符号执行的日志，可以看到，SQL 直接就拼接好了，没有参数。

下面这个，是 `#` 执行的日志，可以看到，这个日志中，使用了预编译的方式：

![](http://mybatis.javaboy.org/assets/images/img/7-1-2.png "7-1-2.png")

在 JDBC 调用中，SQL 的执行，我们可以通过字符串拼接的方式来解决参数的传递问题，也可以通过占位符的方式来解决参数的传递问题。当然，这种方式也传递到 MyBatis 中，在 MyBatis 中，`$` 相当于是参数拼接的方式，而 `#` 则相当于是占位符的方式。

一般来说，由于参数拼接的方式存在 SQL 注入的风险，因此我们使用较少，但是在一些特殊的场景下，又不得不使用这种方式。

有的 SQL 拼接实际上可以通过数据库函数来解决，例如模糊查询：

```xml
<select id="getUserByName" resultType="org.javaboy.mybatis.model.User">
    select * from user where username like concat('%',#{name},'%');
</select>
```

但是有的 SQL 无法使用 `#` 来拼接，例如传入一个动态字段进来，假设我想查询所有数据，要排序查询，但是排序的字段不确定，需要通过参数传入，这种场景就只能使用 `$`，例如如下方法：

```java
List<User> getAllUser(String orderBy);
```

定义该方法对应的 XML 文件：

```xml
<select id="getAllUser" resultType="user">
    select * from user order by ${orderBy}
</select>
```

测试一下：

```java
SqlSessionFactory instance = SqlSessionFactoryUtils.getInstance();
SqlSession sqlSession = instance.openSession();
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
List<User> allUser = mapper.getAllUser("id");
System.out.println(allUser);
```

#### 小结

面试中，遇到这个问题，一定要答出来 Statement 和 PreparedStatement 之间的区别，这个问题才算理解到位了。

#### 7.1.2 简单类型

简单数据类型传递比较容易，像前面的根据 id 查询一条记录就算是这一类的。

这里再举一个例子，比如根据 id 修改用户名：

```java
Integer updateUsernameById(String username, Integer id);
```

再定义该方法对应的 mapper：

```xml
<update id="updateUsernameById">
    update user set username = #{username} where id=#{id};
</update>
```

此时，如果直接调用该方法，会抛出异常：

![](http://mybatis.javaboy.org/assets/images/img/7-1-2-1.png "7-1-2-1.png")

这里是说，找不到我们定义的 username 和 id 这两个参数。同时，这个错误提示中指明，可用的参数名是 [arg1, arg0, param1, param2]，相当于我们自己给变量取的名字失效了，要使用系统提供的默认名字，默认名字实际上是两套体系：

第一套就是 arg0、arg1、、、、
第二套就是 param1、param2、、、

注意，这两个的下标是不一样的。

因此，按照错误提示，我们将参数改为下面这样：

```xml
<update id="updateUsernameById">
    update user set username = #{arg0} where id=#{arg1};
</update>
```

或者下面这样：

```xml
<update id="updateUsernameById">
    update user set username = #{param1} where id=#{param2};
</update>
```

这两种方式，都可以使该方法顺利执行。

但是，默认的名字不好记，容易出错，我们如果想要使用自己写的变量的名字，可以通过给参数添加 @Param 来指定参数名（一般在又多个参数的时候，需要加），一旦用 @Param 指定了参数类型之后，可以省略掉参数类型，就是在 xml 文件中，不用定义 parameterType 了：

```java
Integer updateUsernameById(@Param("username") String username, @Param("id") Integer id);
```

这样定义之后，我们在 mapper.xml 文件中，就可以直接使用 username 和 id 来引用变量了。

#### 7.1.3 对象参数

对象参数。

例如添加一个用户：

```java
Integer addUser(User user);
```

对应的 mapper 文件如下：

```xml
<insert id="addUser" parameterType="org.javaboy.mybatis.model.User">
    insert into user (username,address,favorites) values (#{username},#{address},#{favorites,typeHandler=org.javaboy.mybatis.typehandler.List2VarcharHandler});
</insert>
```

我们在引用的时候，直接使用属性名就能够定位到对象了。如果对象存在多个，我们也需要给对象添加 @Param 注解，如果给对象添加了 @Param 注解，那么对象属性的引用，会有一些变化。如下：

```java
Integer addUser(@Param("user") User user);
```

如果对象参数添加了 @Param 注解，Mapper 中的写法就会发生变化：

```xml
<insert id="addUser" parameterType="org.javaboy.mybatis.model.User">
    insert into user (username,address,favorites) values (#{user.username},#{user.address},#{user.favorites,typeHandler=org.javaboy.mybatis.typehandler.List2VarcharHandler});
</insert>
```

注意多了一个前缀，这个前缀不是变量名，而是 @Param 注解中定义名称。

如果对象中还存在对象，用 . 继续取访问就可以了。

#### 7.1.4 Map 参数

一般不推荐在项目中使用 Map 参数。如果想要使用 Map 传递参数，技术上来说，肯定是没有问题的。

```java
Integer updateUsernameById(HashMap<String,Object> map);
```

XML 文件写法如下：

```xml
<update id="updateUsernameById">
    update user set username = #{username} where id=#{id};
</update>
```

引用的变量名，就是 map 中的 key。基本上和实体类是一样的，如果给 map 取了别名，那么在引用的时候，也要将别名作为前缀加上，这一点和实体类也是一样的。
