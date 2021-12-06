---
sidebar:
  nav: docs-zh
title: 9. 全局异常处理
tags: SpringMVC
categories: SpringMVC
abbrlink: exception
date: 2019-11-11 22:28:52
---

## 9. 全局异常处理

项目中，可能会抛出多个异常，我们不可以直接将异常的堆栈信息展示给用户，有两个原因：

1. 用户体验不好
2. 非常不安全

所以，针对异常，我们可以自定义异常处理，SpringMVC 中，针对全局异常也提供了相应的解决方案，主要是通过 @ControllerAdvice 和 @ExceptionHandler 两个注解来处理的。

以第八节的文件上传大小超出限制为例，自定义异常，只需要提供一个异常处理类即可：

```java
@ControllerAdvice//表示这是一个增强版的 Controller，主要用来做全局数据处理
public class MyException {
    @ExceptionHandler(Exception.class)
    public ModelAndView fileuploadException(Exception e) {
        ModelAndView error = new ModelAndView("error");
        error.addObject("error", e.getMessage());
        return error;
    }
}
```

在这里：

- @ControllerAdvice 表示这是一个增强版的 Controller，主要用来做全局数据处理
- @ExceptionHandler 表示这是一个异常处理方法，这个注解的参数，表示需要拦截的异常，参数为 Exception 表示拦截所有异常，这里也可以具体到某一个异常，如果具体到某一个异常，那么发生了其他异常则不会被拦截到。
- 异常方法的定义，和 Controller 中方法的定义一样，可以返回 ModelAndview，也可以返回 String 或者 void

例如如下代码，指挥拦截文件上传异常，其他异常和它没关系，不会进入到自定义异常处理的方法中来。

```java
@ControllerAdvice//表示这是一个增强版的 Controller，主要用来做全局数据处理
public class MyException {
    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public ModelAndView fileuploadException(MaxUploadSizeExceededException e) {
        ModelAndView error = new ModelAndView("error");
        error.addObject("error", e.getMessage());
        return error;
    }
}
```