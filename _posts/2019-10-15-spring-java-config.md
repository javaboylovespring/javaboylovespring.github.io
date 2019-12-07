---
sidebar:
  nav: docs-zh
title: 3.5 Java 配置
tags: Spring
categories: Spring
abbrlink: spring-java-config
date: 2019-10-15 22:28:52
---

## 3.5 Java 配置

在 Spring 中，想要将一个 Bean 注册到 Spring 容器中，整体上来说，有三种不同的方式。

<!--more-->

-  XML 注入，如前文所说
- Java 配置（通过 Java 代码将 Bean 注册到 Spring 容器中）
- 自动化扫描

这里我们来看 Java 配置。

Java 配置这种方式在 Spring Boot 出现之前，其实很少使用，自从有了 Spring Boot，Java 配置开发被广泛使用，因为在 Spring Boot 中，不使用一行 XML 配置。

例如我有如下一个 Bean：

```java
public class SayHello {
    public String sayHello(String name) {
        return "hello " + name;
    }
}
```

在 Java 配置中，我们用一个 Java 配置类去代替之前的 applicationContext.xml 文件。

```java
@Configuration
public class JavaConfig {
    @Bean
    SayHello sayHello() {
        return new SayHello();
    }
}
```

首先在配置类上有一个 @Configuration 注解，这个注解表示这个类不是一个普通类，而是一个配置类，它的作用相当于 applicationContext.xml。
然后，定义方法，方法返回对象，方法上添加 @Bean 注解，表示将这个方法的返回值注入的 Spring 容器中去。也就是说，@Bean 所对应的方法，就相当于 applicationContext.xml 中的 bean 节点。

既然是配置类，我们需要在项目启动时加载配置类。

```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(JavaConfig.class);
        SayHello hello = ctx.getBean(SayHello.class);
        System.out.println(hello.sayHello("javaboy"));
    }
}
```

注意，配置的加载，是使用 AnnotationConfigApplicationContext 来实现。

关于 Java 配置，这里有一个需要注意的问题:Bean 的名字是什么？

Bean 的默认名称是方法名。以上面的案例为例，Bean 的名字是 sayHello。
如果开发者想自定义方法名，也是可以的，直接在 @Bean 注解中进行过配置。如下配置表示修改 Bean 的名字为 javaboy：

```java
@Configuration
public class JavaConfig {
    @Bean("javaboy")
    SayHello sayHello() {
        return new SayHello();
    }
}
```