---
sidebar:
  nav: docs-zh
title: 5.2 动态代理
tags: Spring
categories: Spring
abbrlink: spring-proxy
date: 2019-10-21 22:28:52
---

## 5.2 动态代理

基于 JDK 的动态代理。

<!--more-->

1.定义一个计算器接口：

```java
public interface MyCalculator {
    int add(int a, int b);
}
```

2.定义计算机接口的实现：

```java
public class MyCalculatorImpl implements MyCalculator {
    public int add(int a, int b) {
        return a+b;
    }
}
```

3.定义代理类

```java
public class CalculatorProxy {
    public static Object getInstance(final MyCalculatorImpl myCalculator) {
        return Proxy.newProxyInstance(CalculatorProxy.class.getClassLoader(), myCalculator.getClass().getInterfaces(), new InvocationHandler() {
            /**
             * @param proxy 代理对象
             * @param method 代理的方法
             * @param args 方法的参数
             * @return
             * @throws Throwable
             */
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println(method.getName()+"方法开始执行啦...");
                Object invoke = method.invoke(myCalculator, args);
                System.out.println(method.getName()+"方法执行结束啦...");
                return invoke;
            }
        });
    }
}
```

Proxy.newProxyInstance 方法接收三个参数，第一个是一个 classloader，第二个是代理多项实现的接口，第三个是代理对象方法的处理器，所有要额外添加的行为都在 invoke 方法中实现。