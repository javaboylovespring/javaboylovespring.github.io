---
sidebar:
  nav: docs-zh
title: 4. Aware 接口
tags: Spring
categories: Spring
abbrlink: spring-aware
date: 2019-10-19 22:28:52
---

## 4. Aware 接口

Aware 接口，从字面上理解就是感知捕获。单纯的一个 Bean 是没有知觉的。

<!--more-->

在 3.6.4 节的场景中，之所以 UserDao 能够注入到 UserService ，有一个前提，就是它两个都是被 Spring 容器管理的。如果直接 new 一个 UserService，这是没用的，因为 UserService  没有被 Spring 容器管理，所以也不会给它里边注入 Bean。

在实际开发中，我们可能会遇到一些类，需要获取到容器的详细信息，那就可以通过 Aware 接口来实现。

Aware 是一个空接口，有很多实现类：

![](http://maven.javaboy.org/assets/images/img/4-1.png "4-1.png")

这些实现的接口，有一些公共特性：

1. 都是以 Aware 结尾
2. 都继承自 Aware
3. 接口内均定义了一个 set 方法

每一个子接口均提供了一个 set 方法，方法的参数就是当前 Bean 需要感知的内容，因此我们需要在 Bean 中声明相关的成员变量来接受这个参数。接收到这个参数后，就可以通过这个参数获取到容器的详细信息了。

```java
@Component
public class SayHello implements ApplicationContextAware {
    private ApplicationContext applicationContext;
    public String sayHello(String name) {
        //判断容器中是否存在某个 Bean
        boolean userDao = applicationContext.containsBean("userDao333");
        System.out.println(userDao);
        return "hello " + name;
    }
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

