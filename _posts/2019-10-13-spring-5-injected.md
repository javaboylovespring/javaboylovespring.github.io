---
sidebar:
  nav: docs-zh
title: 3.3 属性的注入
tags: Spring
categories: Spring
abbrlink: spring-5-injected
date: 2019-10-13 22:28:52
---

## 3.3 属性的注入

在 XML 配置中，属性的注入存在多种方式。

<!--more-->

### 3.3.1 构造方法注入

通过 Bean 的构造方法给 Bean 的属性注入值。

1.第一步首先给 Bean 添加对应的构造方法：

```java
public class Book {
    private Integer id;
    private String name;
    private Double price;

    public Book() {
        System.out.println("-------book init----------");
    }

    public Book(Integer id, String name, Double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
}
```

2.在 xml 文件中注入 Bean

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="org.javaboy.Book" id="book">
        <constructor-arg index="0" value="1"/>
        <constructor-arg index="1" value="三国演义"/>
        <constructor-arg index="2" value="30"/>
    </bean>
</beans>
```

这里需要注意的是，constructor-arg 中的 index 和 Book 中的构造方法参数一一对应。写的顺序可以颠倒，但是 index 的值和 value 要一一对应。

另一种构造方法中的属性注入，则是通过直接指定参数名来注入：

```java
<bean class="org.javaboy.Book" id="book2">
    <constructor-arg name="id" value="2"/>
    <constructor-arg name="name" value="红楼梦"/>
    <constructor-arg name="price" value="40"/>
</bean>
```

**如果有多个构造方法，则会根据给出参数个数以及参数类型，自动匹配到对应的构造方法上，进而初始化一个对象。**

### 3.3.2 set 方法注入

除了构造方法之外，我们也可以通过 set 方法注入值。

```xml
<bean class="org.javaboy.Book" id="book3">
    <property name="id" value="3"/>
    <property name="name" value="水浒传"/>
    <property name="price" value="30"/>
</bean>
```

set 方法注入，有一个很重要的问题，就是属性名。很多人会有一种错觉，觉得属性名就是你定义的属性名，这个是不对的。在所有的框架中，凡是涉及到反射注入值的，属性名统统都不是 Bean 中定义的属性名，而是通过 Java 中的内省机制分析出来的属性名，简单说，就是根据 get/set 方法分析出来的属性名。

### 3.3.3 p 名称空间注入

p 名称空间注入，使用的比较少，它本质上也是调用了 set 方法。

```xml
<bean class="org.javaboy.Book" id="book4" p:id="4" p:bookName="西游记" p:price="33"></bean>
```

### 3.3.4 外部 Bean 的注入

有时候，我们使用一些外部 Bean，这些 Bean 可能没有构造方法，而是通过 Builder 来构造的，这个时候，就无法使用上面的方式来给它注入值了。

例如在 OkHttp 的网络请求中，原生的写法如下：

```java
public class OkHttpMain {
    public static void main(String[] args) {
        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .build();
        Request request = new Request.Builder()
                .get()
                .url("http://b.hiphotos.baidu.com/image/h%3D300/sign=ad628627aacc7cd9e52d32d909032104/32fa828ba61ea8d3fcd2e9ce9e0a304e241f5803.jpg")
                .build();
        Call call = okHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(@NotNull Call call, @NotNull IOException e) {
                System.out.println(e.getMessage());
            }

            @Override
            public void onResponse(@NotNull Call call, @NotNull Response response) throws IOException {
                FileOutputStream out = new FileOutputStream(new File("E:\\123.jpg"));
                int len;
                byte[] buf = new byte[1024];
                InputStream is = response.body().byteStream();
                while ((len = is.read(buf)) != -1) {
                    out.write(buf, 0, len);
                }
                out.close();
                is.close();
            }
        });
    }
}
```

这个 Bean 有一个特点，OkHttpClient 和 Request 两个实例都不是直接 new 出来的，在调用 Builder 方法的过程中，都会给它配置一些默认的参数。这种情况，我们可以使用 静态工厂注入或者实例工厂注入来给 OkHttpClient 提供一个实例。

1.静态工厂注入

首先提供一个 OkHttpClient 的静态工厂：

```java
public class OkHttpUtils {
    private static OkHttpClient OkHttpClient;
    public static OkHttpClient getInstance() {
        if (OkHttpClient == null) {
            OkHttpClient = new OkHttpClient.Builder().build();
        }
        return OkHttpClient;
    }
}
```

在 xml 文件中，配置该静态工厂：

```xml
<bean class="org.javaboy.OkHttpUtils" factory-method="getInstance" id="okHttpClient"></bean>
```

这个配置表示 OkHttpUtils 类中的 getInstance 是我们需要的实例，实例的名字就叫 okHttpClient。然后，在 Java 代码中，获取到这个实例，就可以直接使用了。

```java
public class OkHttpMain {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        OkHttpClient okHttpClient = ctx.getBean("okHttpClient", OkHttpClient.class);
        Request request = new Request.Builder()
                .get()
                .url("http://b.hiphotos.baidu.com/image/h%3D300/sign=ad628627aacc7cd9e52d32d909032104/32fa828ba61ea8d3fcd2e9ce9e0a304e241f5803.jpg")
                .build();
        Call call = okHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(@NotNull Call call, @NotNull IOException e) {
                System.out.println(e.getMessage());
            }

            @Override
            public void onResponse(@NotNull Call call, @NotNull Response response) throws IOException {
                FileOutputStream out = new FileOutputStream(new File("E:\\123.jpg"));
                int len;
                byte[] buf = new byte[1024];
                InputStream is = response.body().byteStream();
                while ((len = is.read(buf)) != -1) {
                    out.write(buf, 0, len);
                }
                out.close();
                is.close();
            }
        });
    }
}
```

2.实例工厂注入

实例工厂就是工厂方法是一个实例方法，这样，工厂类必须实例化之后才可以调用工厂方法。

这次的工厂类如下：

```java
public class OkHttpUtils {
    private OkHttpClient OkHttpClient;
    public OkHttpClient getInstance() {
        if (OkHttpClient == null) {
            OkHttpClient = new OkHttpClient.Builder().build();
        }
        return OkHttpClient;
    }
}
```

此时，在 xml 文件中，需要首先提供工厂方法的实例，然后才可以调用工厂方法：

```java
<bean class="org.javaboy.OkHttpUtils" id="okHttpUtils"/>
<bean class="okhttp3.OkHttpClient" factory-bean="okHttpUtils" factory-method="getInstance" id="okHttpClient"></bean>
```

自己写的 Bean 一般不会使用这两种方式注入，但是，如果需要引入外部 jar，外部 jar 的类的初始化，有可能需要使用这两种方式。