---
sidebar:
  nav: docs-zh
title: 6.3 typeAliases
tags: MyBatis
categories: MyBatis
abbrlink: typealias
date: 2019-11-09 22:28:52
---

### 6.3 typeAliases

这个是 MyBatis 中定义的别名，分两种，一种是 MyBatis 自带的别名，另一种是我们自定义的别名。

<!--more-->


#### 6.3.1 MyBatis 自带的别名

|别名	|映射的类型                 |
|:---|:---|
|_byte |	byte                       |
|_long |	long                       |
|_short |	short                  |
|_int |	int                        |
|_integer |	int                    |
|_double |	double                 |
|_float |	float                  |
|_boolean |	boolean                |
|string |	String                 |
|byte |	Byte                       |
|long |	Long                       |
|short |	Short                      |
|int 	|Integer                    |
|integer |	Integer                |
|double |	Double                 |
|float |	Float                      |
|boolean |	Boolean                |
|date |	Date                       |
|decimal |	BigDecimal             |
|bigdecimal |	BigDecimal         |

本来，我们在 Mapper 中定义数据类型时，需要写全路径，如下：

```xml
<select id="getUserCount" resultType="java.lang.Integer">
    select count(*) from user ;
</select>
```

但是，每次写全路径比较麻烦。这种时候，我们可以用类型的别名来代替，例如用 int 做 Integer 的别名：

```xml
<select id="getUserCount" resultType="int">
    select count(*) from user ;
</select>
```

#### 6.3.2 自定义别名

我们自己的对象，在 Mapper 中定义的时候，也是需要写全路径：

```xml
<select id="getAllUser" resultType="org.javaboy.mybatis.model.User">
    select * from user;
</select>
```

这种情况下，写全路径也比较麻烦，我们可以给我们自己的 User 对象取一个别名，在 mybatis-config.xml 中添加  typeAliases 节点：

```xml
<configuration>
    <properties resource="db.properties"></properties>
    <typeAliases>
        <typeAlias type="org.javaboy.mybatis.model.User" alias="javaboy"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${db.driver}"/>
                <property name="url" value="${db.url}"/>
                <property name="username" value="${db.username}"/>
                <property name="password" value="${db.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <package name="org.javaboy.mybatis.mapper"/>
    </mappers>
</configuration>
```

这里，我们给 User 对象取了一个别名叫 javaboy，然后，我们就可以在 Mapper 中直接使用 javaboy 来代替 User 对象了：

```xml
<select id="getAllUser" resultType="javaboy">
    select * from user;
</select>
```

但是，这种一个一个去枚举对象的过程非常麻烦，我们还可以批量给对象定义别名，批量定义主要是利用包扫描来做，批量定义默认的类的别名，是类名首字母小写，例如如下配置：

```xml
<typeAliases>
    <package name="org.javaboy.mybatis.model"/>
</typeAliases>
```

这个配置就表示给 org.javaboy.mybatis.model 包下的所有类取别名，默认的别名就是类名首字母小写。这个时候，我们在 Mapper 中，就可以利用 user 代替 User 全路径了：

```xml
<select id="getAllUser" resultType="user">
    select * from user;
</select>
```

**在最新版中，批量定义的别名，类名首字母也可以不用小写，在实际开发中，我们一般使用第二种方式（批量定义的方式）**