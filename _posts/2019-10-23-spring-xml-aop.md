---
sidebar:
  nav: docs-zh
title: 5.4 XML 配置 Aop
tags: Spring
categories: Spring
abbrlink: spring-xml-aop
date: 2019-10-23 22:28:52
---

## 5.4 XML 配置 Aop

<!--more-->

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.9.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.5</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>1.9.5</version>
</dependency>
```

接下来，定义通知/增强，但是单纯定义自己的行为即可，不再需要注解：

```java
public class LogAspect {

    public void before(JoinPoint joinPoint) {
        Signature signature = joinPoint.getSignature();
        String name = signature.getName();
        System.out.println(name + "方法开始执行了...");
    }
    
    public void after(JoinPoint joinPoint) {
        Signature signature = joinPoint.getSignature();
        String name = signature.getName();
        System.out.println(name + "方法执行结束了...");
    }

    public void returing(JoinPoint joinPoint,Integer r) {
        Signature signature = joinPoint.getSignature();
        String name = signature.getName();
        System.out.println(name + "方法返回："+r);
    }
    
    public void afterThrowing(JoinPoint joinPoint,Exception e) {
        Signature signature = joinPoint.getSignature();
        String name = signature.getName();
        System.out.println(name + "方法抛异常了："+e.getMessage());
    }
    
    public Object around(ProceedingJoinPoint pjp) {
        Object proceed = null;
        try {
            //这个相当于 method.invoke 方法，我们可以在这个方法的前后分别添加日志，就相当于是前置/后置通知
            proceed = pjp.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        return proceed;
    }
}
```

接下来在 spring 中配置 Aop：

```xml
<bean class="org.javaboy.aop.LogAspect" id="logAspect"/>
<aop:config>
    <aop:pointcut id="pc1" expression="execution(* org.javaboy.aop.commons.*.*(..))"/>
    <aop:aspect ref="logAspect">
        <aop:before method="before" pointcut-ref="pc1"/>
        <aop:after method="after" pointcut-ref="pc1"/>
        <aop:after-returning method="returing" pointcut-ref="pc1" returning="r"/>
        <aop:after-throwing method="afterThrowing" pointcut-ref="pc1" throwing="e"/>
        <aop:around method="around" pointcut-ref="pc1"/>
    </aop:aspect>
</aop:config>
```

最后，在 Main 方法中加载配置文件：

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        MyCalculatorImpl myCalculator = ctx.getBean(MyCalculatorImpl.class);
        myCalculator.add(3, 4);
        myCalculator.min(5, 6);
    }
}
```