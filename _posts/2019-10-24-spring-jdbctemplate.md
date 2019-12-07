---
sidebar:
  nav: docs-zh
title: 6. JdbcTemplate
tags: Spring
categories: Spring
abbrlink: spring-jdbctemplate
date: 2019-10-24 22:28:52
---

## 6. JdbcTemplate

JdbcTemplate 是 Spring 利用 Aop 思想封装的 JDBC 操作工具。

<!--more-->

### 6.1 准备

创建一个新项目，添加如下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.17</version>
    </dependency>
</dependencies>
```

准备数据库：

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
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

准备一个实体类：

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

### 6.2 Java 配置

提供一个配置类，在配置类中配置 JdbcTemplate：

```java
@Configuration
public class JdbcConfig {
    @Bean
    DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUsername("root");
        dataSource.setPassword("123");
        dataSource.setUrl("jdbc:mysql:///test01");
        return dataSource;
    }
    @Bean
    JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource());
    }
}
```

这里，提供两个 Bean，一个是 DataSource 的 Bean，另一个是 JdbcTemplate 的 Bean，JdbcTemplate 的配置非常容易，只需要 new 一个 Bean 出来，然后配置一下 DataSource 就i可以。

```java
public class Main {

    private JdbcTemplate jdbcTemplate;

    @Before
    public void before() {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(JdbcConfig.class);
        jdbcTemplate = ctx.getBean(JdbcTemplate.class);
    }

    @Test
    public void insert() {
        jdbcTemplate.update("insert into user (username,address) values (?,?);", "javaboy", "www.javaboy.org");
    }
    @Test
    public void update() {
        jdbcTemplate.update("update user set username=? where id=?", "javaboy123", 1);

    }
    @Test
    public void delete() {
        jdbcTemplate.update("delete from user where id=?", 2);
    }

    @Test
    public void select() {
        User user = jdbcTemplate.queryForObject("select * from user where id=?", new BeanPropertyRowMapper<User>(User.class), 1);
        System.out.println(user);
    }
}
```

在查询时，如果使用了 BeanPropertyRowMapper，要求查出来的字段必须和 Bean 的属性名一一对应。如果不一样，则不要使用 BeanPropertyRowMapper，此时需要自定义 RowMapper 或者给查询的字段取别名。

1. 给查询出来的列取别名：

```java
@Test
public void select2() {
    User user = jdbcTemplate.queryForObject("select id,username as name,address from user where id=?", new BeanPropertyRowMapper<User>(User.class), 1);
    System.out.println(user);
}
```

2.自定义 RowMapper

```java
@Test
public void select3() {
    User user = jdbcTemplate.queryForObject("select * from user where id=?", new RowMapper<User>() {
        public User mapRow(ResultSet resultSet, int i) throws SQLException {
            int id = resultSet.getInt("id");
            String username = resultSet.getString("username");
            String address = resultSet.getString("address");
            User u = new User();
            u.setId(id);
            u.setName(username);
            u.setAddress(address);
            return u;
        }
    }, 1);
    System.out.println(user);
}
```

查询多条记录，方式如下：

```java
@Test
public void select4() {
    List<User> list = jdbcTemplate.query("select * from user", new BeanPropertyRowMapper<>(User.class));
    System.out.println(list);
}
```

### 6.3 XML 配置

以上配置，也可以通过 XML 文件来实现。通过 XML 文件实现只是提供 JdbcTemplate 实例，剩下的代码还是 Java 代码，就是 JdbcConfig 被 XML 文件代替而已。

```xml
<bean class="org.springframework.jdbc.datasource.DriverManagerDataSource" id="dataSource">
    <property name="username" value="root"/>
    <property name="password" value="123"/>
    <property name="url" value="jdbc:mysql:///test01?serverTimezone=Asia/Shanghai"/>
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
</bean>
<bean class="org.springframework.jdbc.core.JdbcTemplate" id="jdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

配置完成后，加载该配置文件，并启动：

```java
public class Main {

    private JdbcTemplate jdbcTemplate;

    @Before
    public void before() {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        jdbcTemplate = ctx.getBean(JdbcTemplate.class);
    }

    @Test
    public void insert() {
        jdbcTemplate.update("insert into user (username,address) values (?,?);", "javaboy", "www.javaboy.org");
    }
    @Test
    public void update() {
        jdbcTemplate.update("update user set username=? where id=?", "javaboy123", 1);

    }
    @Test
    public void delete() {
        jdbcTemplate.update("delete from user where id=?", 2);
    }

    @Test
    public void select() {
        User user = jdbcTemplate.queryForObject("select * from user where id=?", new BeanPropertyRowMapper<User>(User.class), 1);
        System.out.println(user);
    }
    @Test
    public void select4() {
        List<User> list = jdbcTemplate.query("select * from user", new BeanPropertyRowMapper<>(User.class));
        System.out.println(list);
    }

    @Test
    public void select2() {
        User user = jdbcTemplate.queryForObject("select id,username as name,address from user where id=?", new BeanPropertyRowMapper<User>(User.class), 1);
        System.out.println(user);
    }

    @Test
    public void select3() {
        User user = jdbcTemplate.queryForObject("select * from user where id=?", new RowMapper<User>() {
            public User mapRow(ResultSet resultSet, int i) throws SQLException {
                int id = resultSet.getInt("id");
                String username = resultSet.getString("username");
                String address = resultSet.getString("address");
                User u = new User();
                u.setId(id);
                u.setName(username);
                u.setAddress(address);
                return u;
            }
        }, 1);
        System.out.println(user);
    }

}
```