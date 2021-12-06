---
sidebar:
  nav: docs-zh
title: 5. DispatcherServlet
tags: SpringMVC
categories: SpringMVC
abbrlink: dispatcherservlet
date: 2019-11-05 22:28:52
---

## 5. DispatcherServlet

### 5.1 DispatcherServlet作用

DispatcherServlet 是前端控制器设计模式的实现，提供 Spring Web MVC 的集中访问点，而且负责职责的分派，而且与 Spring IoC 容器无缝集成，从而可以获得 Spring 的所有好处。DispatcherServlet 主要用作职责调度工作，本身主要用于控制流程，主要职责如下：

1. 文件上传解析，如果请求类型是 multipart 将通过 MultipartResolver 进行文件上传解析；
2. 通过 HandlerMapping，将请求映射到处理器（返回一个 HandlerExecutionChain，它包括一个处理器、多个 HandlerInterceptor 拦截器）；
3. 通过 HandlerAdapter 支持多种类型的处理器(HandlerExecutionChain 中的处理器)；
4. 通过 ViewResolver 解析逻辑视图名到具体视图实现；
5. 本地化解析；
6. 渲染具体的视图等；
7. 如果执行过程中遇到异常将交给 HandlerExceptionResolver 来解析

### 5.2 DispathcherServlet配置详解

```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-servlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

- load-on-startup：表示启动容器时初始化该 Servlet；
- url-pattern：表示哪些请求交给 Spring Web MVC 处理， "/" 是用来定义默认 servlet 映射的。也可以如 `*.html` 表示拦截所有以 html 为扩展名的请求
- contextConfigLocation：表示 SpringMVC 配置文件的路径

 其他的参数配置：

|参数|描述|
|:---|:---|
|contextClass|实现WebApplicationContext接口的类，当前的servlet用它来创建上下文。如果这个参数没有指定， 默认使用XmlWebApplicationContext。|
|contextConfigLocation|传给上下文实例（由contextClass指定）的字符串，用来指定上下文的位置。这个字符串可以被分成多个字符串（使用逗号作为分隔符） 来支持多个上下文（在多上下文的情况下，如果同一个bean被定义两次，后面一个优先）。|
|namespace|WebApplicationContext命名空间。默认值是[server-name]-servlet。|

### 5.3 Spring 配置

之前的案例中，只有 SpringMVC，没有 Spring，Web 项目也是可以运行的。在实际开发中，Spring 和 SpringMVC 是分开配置的，所以我们对上面的项目继续进行完善，添加 Spring 相关配置。

首先，项目添加一个 service 包，提供一个 HelloService 类，如下：

```java
@Service
public class HelloService {
    public String hello(String name) {
        return "hello " + name;
    }
}
```

现在，假设我需要将 HelloService 注入到 Spring 容器中并使用它，这个是属于 Spring 层的 Bean，所以我们一般将除了 Controller 之外的所有 Bean 注册到 Spring 容器中，而将 Controller 注册到 SpringMVC 容器中，现在，在 resources 目录下添加 applicationContext.xml 作为 spring 的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.javaboy" use-default-filters="true">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
</beans>
```

但是，这个配置文件，默认情况下，并不会被自动加载，所以，需要我们在 web.xml 中对其进行配置：

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

首先通过 context-param 指定 Spring 配置文件的位置，这个配置文件也有一些默认规则，它的配置文件名默认就叫 applicationContext.xml ，并且，如果你将这个配置文件放在 WEB-INF 目录下，那么这里就可以不用指定配置文件位置了，只需要指定监听器就可以了。这段配置是 Spring 集成 Web 环境的通用配置；一般用于加载除 Web 层的 Bean（如DAO、Service 等），以便于与其他任何Web框架集成。

- contextConfigLocation：表示用于加载 Bean 的配置文件；
- contextClass：表示用于加载 Bean 的 ApplicationContext 实现类，默认 WebApplicationContext。

配置完成之后，还需要修改 MyController，在 MyController 中注入 HelloSerivce:

```java
@org.springframework.stereotype.Controller("/hello")
public class MyController implements Controller {
    @Autowired
    HelloService helloService;
    /**
     * 这就是一个请求处理接口
     * @param req 这就是前端发送来的请求
     * @param resp 这就是服务端给前端的响应
     * @return 返回值是一个 ModelAndView，Model 相当于是我们的数据模型，View 是我们的视图
     * @throws Exception
     */
    public ModelAndView handleRequest(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        System.out.println(helloService.hello("javaboy"));
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("name", "javaboy");
        return mv;
    }
}
```

**注意**

为了在 SpringMVC 容器中能够扫描到 MyController ，这里给 MyController 添加了 @Controller 注解，同时，由于我们目前采用的 HandlerMapping 是 BeanNameUrlHandlerMapping（意味着请求地址就是处理器 Bean 的名字），所以，还需要手动指定 MyController 的名字。

最后，修改 SpringMVC 的配置文件，将 Bean 配置为扫描形式：

```xml
<context:component-scan base-package="org.javaboy.helloworld" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
<!--这个是处理器映射器，这种方式，请求地址其实就是一个 Bean 的名字，然后根据这个 bean 的名字查找对应的处理器-->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" id="handlerMapping">
    <property name="beanName" value="/hello"/>
</bean>
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" id="handlerAdapter"/>
<!--视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="viewResolver">
    <property name="prefix" value="/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

配置完成后，再次启动项目，Spring 容器也将会被创建。访问 /hello 接口，HelloService 中的 hello 方法就会自动被调用。

### 5.4 两个容器

当 Spring 和 SpringMVC 同时出现，我们的项目中将存在两个容器，一个是 Spring 容器，另一个是 SpringMVC 容器，Spring 容器通过 ContextLoaderListener 来加载，SpringMVC 容器则通过 DispatcherServlet 来加载，这两个容器不一样：


![](http://springmvc.javaboy.org/assets/images/img/5-4-1.png "5-4-1.png")

从图中可以看出：

- ContextLoaderListener 初始化的上下文加载的 Bean 是对于整个应用程序共享的，不管是使用什么表现层技术，一般如 DAO 层、Service 层 Bean；
- DispatcherServlet 初始化的上下文加载的 Bean 是只对 Spring Web MVC 有效的 Bean，如 Controller、HandlerMapping、HandlerAdapter 等等，该初始化上下文应该只加载 Web相关组件。

1. 为什么不在 Spring 容器中扫描所有 Bean？

这个是不可能的。因为请求达到服务端后，找 DispatcherServlet 去处理，只会去 SpringMVC 容器中找，这就意味着 Controller 必须在 SpringMVC 容器中扫描。

2.为什么不在 SpringMVC 容器中扫描所有 Bean？
 
 这个是可以的，可以在 SpringMVC 容器中扫描所有 Bean。不写在一起，有两个方面的原因：

1. 为了方便配置文件的管理
2. 在 Spring+SpringMVC+Hibernate 组合中，实际上也不支持这种写法