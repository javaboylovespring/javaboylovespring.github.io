---
sidebar:
  nav: docs-zh
title: 15. 拦截器
tags: SpringMVC
categories: SpringMVC
abbrlink: interceptor
date: 2019-11-19 22:28:52
---

## 15. 拦截器

SpringMVC 中的拦截器，相当于 Jsp/Servlet 中的过滤器，只不过拦截器的功能更为强大。

拦截器的定义非常容易：

```java
@Component
public class MyInterceptor1 implements HandlerInterceptor {
    /**
     * 这个是请求预处理的方法，只有当这个方法返回值为 true 的时候，后面的方法才会执行
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor1:preHandle");
        return true;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor1:postHandle");

    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor1:afterCompletion");

    }
}
@Component
public class MyInterceptor2 implements HandlerInterceptor {
    /**
     * 这个是请求预处理的方法，只有当这个方法返回值为 true 的时候，后面的方法才会执行
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor2:preHandle");
        return true;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor2:postHandle");

    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor2:afterCompletion");

    }
}
```

拦截器定义好之后，需要在 SpringMVC 的配置文件中进行配置：

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <ref bean="myInterceptor1"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <ref bean="myInterceptor2"/>
    </mvc:interceptor>
</mvc:interceptors>
```

如果存在多个拦截器，拦截规则如下：

- preHandle 按拦截器定义顺序调用
- postHandler 按拦截器定义逆序调用
- afterCompletion 按拦截器定义逆序调用
- postHandler 在拦截器链内所有拦截器返成功调用
- afterCompletion 只有 preHandle 返回 true 才调用