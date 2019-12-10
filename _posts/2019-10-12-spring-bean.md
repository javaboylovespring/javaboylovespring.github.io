---
sidebar:
  nav: docs-zh
title: 3.2 Bean 的获取
tags: Spring
categories: Spring
abbrlink: spring-bean
date: 2019-10-12 22:28:52
---

## 3.2 Bean 的获取

在上一小节中，我们通过 ctx.getBean 方法来从 Spring 容器中获取 Bean，传入的参数是 Bean 的 name 或者 id 属性。除了这种方式之外，也可以直接通过 Class 去获取一个 Bean。

<!--more-->

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        Book book = ctx.getBean(Book.class);
        System.out.println(book);
    }
}
```

这种方式有一个很大的弊端，如果存在多个实例，这种方式就不可用，例如，xml 文件中存在两个 Bean：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="org.javaboy.Book" id="book"/>
    <bean class="org.javaboy.Book" id="book2"/>
</beans>
```

此时，如果通过 Class 去查找 Bean，会报如下错误：

![](http://spring.javaboy.org/assets/images/img/3-2-1.png "3-2-1.png")

所以，一般建议使用 name 或者 id 去获取 Bean 的实例。