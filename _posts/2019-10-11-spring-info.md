---
sidebar:
  nav: docs-zh
title: 3.1 Ioc 概念
tags: Spring
categories: Spring
abbrlink: spring-info
date: 2019-10-12 20:28:52
---

## 3. Ioc

### 3.1 Ioc 概念

Ioc （Inversion of Control），中文叫做控制反转。这是一个概念，也是一种思想。控制反转，实际上就是指对一个对象的控制权的反转。例如，如下代码：

```java
public class Book {
    private Integer id;
    private String name;
    private Double price;
//省略getter/setter
}
public class User {
    private Integer id;
    private String name;
    private Integer age;

    public void doSth() {
        Book book = new Book();
        book.setId(1);
        book.setName("故事新编");
        book.setPrice((double) 20);
    }
}
```

在这种情况下，Book 对象的控制权在 User 对象里边，这样，Book 和 User 高度耦合，如果在其他对象中需要使用 Book 对象，得重新创建，也就是说，对象的创建、初始化、销毁等操作，统统都要开发者自己来完成。如果能够将这些操作交给容器来管理，开发者就可以极大的从对象的创建中解脱出来。

使用 Spring 之后，我们可以将对象的创建、初始化、销毁等操作交给 Spring 容器来管理。就是说，在项目启动时，所有的 Bean 都将自己注册到 Spring 容器中去（如果有必要的话），然后如果其他 Bean 需要使用到这个 Bean ，则不需要自己去 new，而是直接去 Spring 容器去要。

通过一个简单的例子看下这个过程。

### 3.2 Ioc 初体验

首先创建一个普通的 Maven 项目，然后引入 spring-context 依赖，如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
</dependencies>
```

接下来，在 resources 目录下创建一个 spring 的配置文件（注意，一定要先添加依赖，后创建配置文件，否则创建配置文件时，没有模板选项）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

在这个文件中，我们可以配置所有需要注册到 Spring 容器的 Bean：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="org.javaboy.Book" id="book"/>
</beans>
```

class 属性表示需要注册的 bean 的全路径，id 则表示 bean 的唯一标记，也开可以 name 属性作为 bean 的标记，在超过 99% 的情况下，id 和 name 其实是一样的，特殊情况下不一样。

接下来，加载这个配置文件：

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    }
}
```

执行 main 方法，配置文件就会被自动加载，进而在 Spring 中初始化一个 Book 实例。此时，我们显式的指定 Book 类的无参构造方法，并在无参构造方法中打印日志，可以看到无参构造方法执行了，进而证明对象已经在 Spring 容器中初始化了。

最后，通过 getBean 方法，可以从容器中去获取对象：

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        Book book = (Book) ctx.getBean("book");
        System.out.println(book);
    }
}
```

> 加载方式，除了ClassPathXmlApplicationContext 之外（去 classpath 下查找配置文件），另外也可以使用 FileSystemXmlApplicationContext ，FileSystemXmlApplicationContext 会从操作系统路径下去寻找配置文件。

```java
public class Main {
    public static void main(String[] args) {
        FileSystemXmlApplicationContext ctx = new FileSystemXmlApplicationContext("F:\\workspace5\\workspace\\spring\\spring-ioc\\src\\main\\resources\\applicationContext.xml");
        Book book = (Book) ctx.getBean("book");
        System.out.println(book);
    }
}
```