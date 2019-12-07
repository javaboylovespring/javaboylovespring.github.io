---
sidebar:
  nav: docs-zh
title: 3.6 自动化配置
tags: Spring
categories: Spring
abbrlink: auto-config
date: 2019-10-16 22:28:52
---

## 3.6 自动化配置

在我们实际开发中，大量的使用自动配置。

<!--more-->

自动化配置既可以通过 Java 配置来实现，也可以通过 xml 配置来实现。

### 3.6.1 准备工作

例如我有一个 UserService，我希望在自动化扫描时，这个类能够自动注册到 Spring 容器中去，那么可以给该类添加一个 @Service，作为一个标记。

和 @Service 注解功能类似的注解，一共有四个：

- @Component
- @Repository
- @Service
- @Controller

这四个中，另外三个都是基于 @Component 做出来的，而且从目前的源码来看，功能也是一致的，那么为什么要搞三个呢？主要是为了在不同的类上面添加时方便。

- 在 Service 层上，添加注解时，使用 @Service
- 在 Dao 层，添加注解时，使用 @Repository
- 在 Controller 层，添加注解时，使用 @Controller
- 在其他组件上添加注解时，使用 @Component

```java
@Service
public class UserService {
    public List<String> getAllUser() {
        List<String> users = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            users.add("javaboy:" + i);
        }
        return users;
    }
}
```

添加完成后，自动化扫描有两种方式，一种就是通过 Java 代码配置自动化扫描，另一种则是通过 xml 文件来配置自动化扫描。

### 3.6.2 Java 代码配置自动扫描

```java
@Configuration
@ComponentScan(basePackages = "org.javaboy.javaconfig.service")
public class JavaConfig {
}
```

然后，在项目启动中加载配置类，在配置类中，通过 @ComponentScan 注解指定要扫描的包（如果不指定，默认情况下扫描的是配置类所在的包下载的 Bean 以及配置类所在的包下的子包下的类），然后就可以获取 UserService 的实例了：

```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(JavaConfig.class);
        UserService userService = ctx.getBean(UserService.class);
        System.out.println(userService.getAllUser());
    }
}
```

这里有几个问题需要注意：

1.Bean 的名字叫什么？

默认情况下，Bean 的名字是类名首字母小写。例如上面的 UserService，它的实例名，默认就是 userService。如果开发者想要自定义名字，就直接在 @Service 注解中添加即可。

2.有几种扫描方式？

上面的配置，我们是按照包的位置来扫描的。也就是说，Bean 必须放在指定的扫描位置，否则，即使你有 @Service 注解，也扫描不到。

除了按照包的位置来扫描，还有另外一种方式，就是根据注解来扫描。例如如下配置：

```java
@Configuration
@ComponentScan(basePackages = "org.javaboy.javaconfig",useDefaultFilters = true,excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class)})
public class JavaConfig {
}
```

这个配置表示扫描 org.javaboy.javaconfig 下的所有 Bean，但是除了 Controller。


### 3.6.3 XML 配置自动化扫描

```xml
<context:component-scan base-package="org.javaboy.javaconfig"/>
```

上面这行配置表示扫描 org.javaboy.javaconfig 下的所有 Bean。当然也可以按照类来扫描。

XML 配置完成后，在 Java 代码中加载 XML 配置即可。

```java
public class XMLTest {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = ctx.getBean(UserService.class);
        List<String> list = userService.getAllUser();
        System.out.println(list);
    }
}
```

也可以在 XML 配置中按照注解的类型进行扫描：

```xml
<context:component-scan base-package="org.javaboy.javaconfig" use-default-filters="true">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

### 3.6.4 对象注入

自动扫描时的对象注入有三种方式：

1. @Autowired
2. @Resources
3. @Injected

@Autowired 是根据类型去查找，然后赋值，这就有一个要求，这个类型只可以有一个对象，否则就会报错。@Resources 是根据名称去查找，默认情况下，定义的变量名，就是查找的名称，当然开发者也可以在 @Resources 注解中手动指定。所以，如果一个类存在多个实例，那么就应该使用 @Resources 去注入，如果非常使用 @Autowired，也是可以的，此时需要配合另外一个注解，@Qualifier，在 @Qualifier 中可以指定变量名，两个一起用（@Qualifier 和 @Autowired）就可以实现通过变量名查找到变量。

```java
@Service
public class UserService {

    @Autowired
    UserDao userDao;
    public String hello() {
        return userDao.hello();
    }

    public List<String> getAllUser() {
        List<String> users = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            users.add("javaboy:" + i);
        }
        return users;
    }
}
```