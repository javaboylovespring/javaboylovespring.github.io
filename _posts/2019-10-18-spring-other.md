---
sidebar:
  nav: docs-zh
title: 3.8 其他
tags: Spring
categories: Spring
abbrlink: spring-other
date: 2019-10-18 22:28:52
---

## 3.8 其他

### 3.8.1 Bean 的作用域

在 XML 配置中注册的 Bean，或者用 Java 配置注册的 Bean，如果我多次获取，获取到的对象是否是同一个？

<!--more-->

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        User user = ctx.getBean("user", User.class);
        User user2 = ctx.getBean("user", User.class);
        System.out.println(user==user2);
    }
}
```

如上，从 Spring 容器中多次获取同一个 Bean，默认情况下，获取到的实际上是同一个实例。当然我们可以自己手动配置。


```xml
<bean class="org.javaboy.User" id="user" scope="prototype" />
```

通过在 XML 节点中，设置 scope 属性，我们可以调整默认的实例个数。scope 的值为 singleton（默认），表示这个 Bean 在 Spring 容器中，是以单例的形式存在，如果 scope 的值为 prototype，表示这个 Bean 在 Spring 容器中不是单例，多次获取将拿到多个不同的实例。

除了 singleton 和 prototype 之外，还有两个取值，request 和 session。这两个取值在 web 环境下有效。这是在 XML 中的配置，我们也可以在 Java 中配置。

```java
@Configuration
public class JavaConfig {
    @Bean
    @Scope("prototype")
    SayHello sayHello() {
        return new SayHello();
    }
}
```

在 Java 代码中，我们可以通过 @Scope 注解指定 Bean 的作用域。

当然，在自动扫描配置中，也可以指定 Bean 的作用域。

```java
@Repository
@Scope("prototype")
public class UserDao {
    public String hello() {
        return "userdao";
    }
}
```

### 3.8.2 id 和 name 的区别

在 XML 配置中，我们可以看到，即可以通过 id 给 Bean 指定一个唯一标识符，也可以通过 name 来指定，大部分情况下这两个作用是一样的，有一个小小区别：

name 支持取多个。多个 name 之间，用 , 隔开：

```xml
<bean class="org.javaboy.User" name="user,user1,user2,user3" scope="prototype"/>
```

此时，通过 user、user1、user2、user3 都可以获取到当前对象：

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        User user = ctx.getBean("user", User.class);
        User user2 = ctx.getBean("user2", User.class);
        System.out.println(user);
        System.out.println(user2);
    }
}
```

而 id 不支持有多个值。如果强行用 , 隔开，它还是一个值。例如如下配置：

```xml
<bean class="org.javaboy.User" id="user,user1,user2,user3" scope="prototype" />
```

这个配置表示 Bean 的名字为 `user,user1,user2,user3`，具体调用如下：

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        User user = ctx.getBean("user,user1,user2,user3", User.class);
        User user2 = ctx.getBean("user,user1,user2,user3", User.class);
        System.out.println(user);
        System.out.println(user2);
    }
}
```

### 3.8.3 混合配置

混合配置就是 Java 配置+XML 配置。混用的话，可以在 Java 配置中引入 XML 配置。

```java
@Configuration
@ImportResource("classpath:applicationContext.xml")
public class JavaConfig {
}
```

在 Java 配置中，通过 @ImportResource 注解可以导入一个 XML 配置。