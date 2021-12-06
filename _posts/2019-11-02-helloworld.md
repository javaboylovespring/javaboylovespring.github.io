---
sidebar:
  nav: docs-zh
title: 2. HelloWorld
tags: MyBatis
categories: MyBatis
abbrlink: helloworld
date: 2019-11-02 22:28:52
---

## 2. HelloWorld

我们通过一个简单的 HelloWorld 先来看下 MyBatis 的基本用法。

<!--more-->


首先来准备一个数据库：

```sql
CREATE DATABASE /*!32312 IF NOT EXISTS*/`test01` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;

USE `test01`;

/*Table structure for table `user` */

DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) COLLATE utf8mb4_general_ci DEFAULT NULL,
  `address` varchar(255) COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

/*Data for the table `user` */

insert  into `user`(`id`,`username`,`address`) values (1,'javaboy123','www.javaboy.org'),(3,'javaboy','spring.javaboy.org'),(4,'张三','深圳'),(5,'李四','广州'),(6,'王五','北京');
```

接下来创建一个普通的 Maven 工程，不用创建 Web 工程，JavaSE 工程即可。项目创建完成后，添加 MyBatis 依赖：

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.2</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.17</version>
</dependency>
```

接下来，准备一个 Mapper 文件，Mapper 是用来在 MyBatis 中定义 SQL 的 XML 配置文件，由于在实际开发中，我们经常需要使用到 Mapper，经常需要自己创建 Mapper 文件，因此，我们可以将 Mapper 文件做成一个模板。具体操作如下：

在 IDEA 中，选择 resources 目录，右键单击，New-->Edit File Templates:

![](http://mybatis.javaboy.org/assets/images/img/2-1.png "2-1.png")

然后点击 + ，添加一个新的模板进来，给模板取名，同时设置扩展名，并将如下内容拷贝到模板中：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="#[[$namespace$]]#">
</mapper>
```

如下图：

![](http://mybatis.javaboy.org/assets/images/img/2-2.png "2-2.png")

配置完成后，再次创建 Mapper 文件时，就可以选择 New-->mapper 了，这里，我们创建一个 UserMapper：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.javaboy.mymapper">
    
</mapper>
```

创建一个新的 mapper ，需要首先给它取一个 namespace，这相当于是一个分隔符，因为我们在项目中，会存在很多个 Mapper，每一个 Mapper 中都会定义相应的增删改查方法，为了避免方法冲突，也为了便于管理，每一个 Mapper 都有自己的 namespace，而且这个 namespace 不可以重复。

接下来，在 Mapper 中，定义一个简单的查询方法，根据 id 查询一个用户：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.javaboy.mymapper">

    <select id="getUserById" resultType="org.javaboy.mybatis.model.User">
        select * from user where id=#{id};
    </select>
</mapper>
```

在 Mapper 中，首先定义一个 select ，id 表示查询方法的唯一标识符，resultType 定义了返回值的类型。在 select 节点中，定义查询 SQL，#{id}，表示这个位置用来接收外部传进来的参数。

定义的 User 实体类，如下：

```java
public class User {
    private Integer id;
    private String username;
    private String address;

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", address='" + address + '\'' +
                '}';
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

接下来，创建 MyBatis 配置文件，如果是第一次使用，可以参考官网，拷贝一下配置文件的头信息，如果需要多次使用这个配置文件，可以在 IDEA 中创建该配置文件的模板：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///test01?serverTimezone=Asia/Shanghai"/>
                <property name="username" value="root"/>
                <property name="password" value="123"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>
```

在这个配置文件中，我们只需要配置 environments 和 mapper 即可，environment 就是 MyBatis 所连接的数据库的环境信息，它放在一个 environments 节点中，意味着 environments 中可以有多个 environment，为社么需要多个呢？开发、测试、生产，不同环境各一个 environment，每一个 environment 都有一个 id，也就是它的名字，然后，在 environments 中，通过 default 属性，指定你需要的 environment。每一个 environment 中，定义一个数据的基本连接信息。

在 mappers 节点中，定义 Mapper，也就是指定我们上一步所写的 Mapper 的路径。

最后，我们来加载这个主配置文件：

```java
public class Main {
    public static void main(String[] args) throws IOException {
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
        SqlSession sqlSession = factory.openSession();
        User user = (User) sqlSession.selectOne("org.javaboy.mymapper.getUserById", 3);
        System.out.println(user);
        sqlSession.close();
    }
}
```

首先，我们加载主配置文件，生成一个 SqlSessionFactory，再由 SqlSessionFactory 生成一个 SqlSession，一个 SqlSession 就相当于是我们的一个会话，类似于 JDBC 中的一个连接，在 SQL 操作完成后，这个会话是可以关闭的。

在这里，SqlSessionFactoryBuilder 用于创建 SqlSessionFacoty，SqlSessionFacoty 一旦创建完成就不需要 SqlSessionFactoryBuilder 了，因为 SqlSession 是通过 SqlSessionFactory 生产，所以可以将 SqlSessionFactoryBuilder 当成一个工具类使用，最佳使用范围是方法范围即方法体内局部变量。

SqlSessionFactory 是一个接口，接口中定义了 openSession 的不同重载方法，SqlSessionFactory 的最佳使用范围是整个应用运行期间，一旦创建后可以重复使用，通常以单例模式管理 SqlSessionFactory。

SqlSession 中封装了对数据库的操作，如：查询、插入、更新、删除等。通过 SqlSessionFactory 创建 SqlSession，而 SqlSessionFactory 是通过 SqlSessionFactoryBuilder 进行创建。SqlSession 是一个面向用户的接口， sqlSession 中定义了数据库操作，默认使用 DefaultSqlSession 实现类。每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不能共享使用，它也是线程不安全的。因此最佳的范围是请求或方法范围。绝对不能将 SqlSession 实例的引用放在一个类的静态字段或实例字段中。打开一个  SqlSession；使用完毕就要关闭它。通常把这个关闭操作放到 finally 块中以确保每次都能执行关闭。

基于上面几点，我们可以对 SqlSessionFactory 进行封装：

```java
public class SqlSessionFactoryUtils {
    private static SqlSessionFactory SQL_SESSION_FACTORY = null;

    public static SqlSessionFactory getInstance() {
        if (SQL_SESSION_FACTORY == null) {
            try {
                SQL_SESSION_FACTORY = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return SQL_SESSION_FACTORY;
    }
}
```

这样，在需要使用的时候，通过这个工厂方法来获取 SqlSessionFactory 的实例。
