---
sidebar:
  nav: docs-zh
title: 2. HelloWorld
tags: SpringMVC
categories: SpringMVC
abbrlink: springmvc-helloworld
date: 2019-11-02 22:28:52
---

## 2. HelloWorld

接下来，通过一个简单的例子来感受一下 SpringMVC。

1.利用 Maven 创建一个 web 工程（参考 Maven 教程）。
2.在 pom.xml 文件中，添加 spring-webmvc 的依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>RELEASE</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>javax.servlet.jsp-api</artifactId>
        <version>2.3.3</version>
    </dependency>
</dependencies>
```

添加了 spring-webmvc 依赖之后，其他的 spring-web、spring-aop、spring-context 等等就全部都加入进来了。

3.准备一个 Controller，即一个处理浏览器请求的接口。

```java
public class MyController implements Controller {
    /**
     * 这就是一个请求处理接口
     * @param req 这就是前端发送来的请求
     * @param resp 这就是服务端给前端的响应
     * @return 返回值是一个 ModelAndView，Model 相当于是我们的数据模型，View 是我们的视图
     * @throws Exception
     */
    public ModelAndView handleRequest(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("name", "javaboy");
        return mv;
    }
}
```

这里我们我们创建出来的 Controller 就是前端请求处理接口。

4.创建视图

这里我们就采用 jsp 作为视图，在 webapp 目录下创建 hello.jsp 文件，内容如下：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>hello ${name}!</h1>
</body>
</html>
```

5.在 resources 目录下，创建一个名为 spring-servlet.xml 的 springmvc 的配置文件，这里，我们先写一个简单的 demo ，因此可以先不用添加 spring 的配置。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="org.javaboy.helloworld.MyController" name="/hello"/>
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
</beans>
```

6.加载 springmvc 配置文件

在 web 项目启动时，加载 springmvc 配置文件，这个配置是在 web.xml 中完成的。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-servlet.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

所有请求都将自动拦截下来，拦截下来后，请求交给 DispatcherServlet 去处理，在加载 DispatcherServlet 时，还需要指定配置文件路径。这里有一个默认的规则，如果配置文件放在 webapp/WEB-INF/ 目录下，并且配置文件的名字等于 DispatcherServlet 的名字+ `-servlet`（即这里的配置文件路径是 webapp/WEB-INF/springmvc-servlet.xml），如果是这样的话，可以不用添加 init-param 参数，即不用手动配置 springmvc 的配置文件，框架会自动加载。

7.配置并启动项目（参考 Maven 教程）

8.项目启动成功后，浏览器输入 http://localhost:8080/hello 就可以看到如下页面：

![](http://springmvc.javaboy.org/assets/images/img/2-1.png "2-1.png")