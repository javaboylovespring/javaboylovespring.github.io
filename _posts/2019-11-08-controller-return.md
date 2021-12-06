---
sidebar:
  nav: docs-zh
title: 7.2 Controller 方法的返回值
tags: SpringMVC
categories: SpringMVC
abbrlink: controller-return
date: 2019-11-08 22:28:52
---

## 7.2 Controller 方法的返回值

### 7.2.1 返回 ModelAndView

如果是前后端不分的开发，大部分情况下，我们返回 ModelAndView，即数据模型+视图：

```java
@Controller
@RequestMapping("/user")
public class HelloController {
    @RequestMapping("/hello")
    public ModelAndView hello() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("username", "javaboy");
        return mv;
    }
}
```

Model 中，放我们的数据，然后在 ModelAndView 中指定视图名称。

### 7.2.2 返回 Void


没有返回值。没有返回值，并不一定真的没有返回值，只是方法的返回值为 void，我们可以通过其他方式给前端返回。**实际上，这种方式也可以理解为 Servlet 中的那一套方案。**

> 注意，由于默认的 Maven 项目没有 Servlet，因此这里需要额外添加一个依赖：

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
</dependency>
```

- 通过 HttpServletRequest 做服务端跳转

```java
@RequestMapping("/hello2")
public void hello2(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    req.getRequestDispatcher("/jsp/hello.jsp").forward(req,resp);//服务器端跳转
}
```

- 通过 HttpServletResponse 做重定向

```java
@RequestMapping("/hello3")
public void hello3(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    resp.sendRedirect("/hello.jsp");
}
```

也可以自己手动指定响应头去实现重定向：

```java
@RequestMapping("/hello3")
public void hello3(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    resp.setStatus(302);
    resp.addHeader("Location", "/jsp/hello.jsp");
}
```

- 通过 HttpServletResponse 给出响应

```java
@RequestMapping("/hello4")
public void hello4(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    resp.setContentType("text/html;charset=utf-8");
    PrintWriter out = resp.getWriter();
    out.write("hello javaboy!");
    out.flush();
    out.close();
}
```

这种方式，既可以返回 JSON，也可以返回普通字符串。

### 7.2.3 返回字符串

- 返回逻辑视图名

前面的 ModelAndView 可以拆分为两部分，Model 和 View，在 SpringMVC 中，Model 我们可以直接在参数中指定，然后返回值是逻辑视图名：

```java
@RequestMapping("/hello5")
public String hello5(Model model) {
    model.addAttribute("username", "javaboy");//这是数据模型
    return "hello";//表示去查找一个名为 hello 的视图
}
```

- 服务端跳转

```java
@RequestMapping("/hello5")
public String hello5() {
    return "forward:/jsp/hello.jsp";
}
```

forward 后面跟上跳转的路径。

- 客户端跳转

```java
@RequestMapping("/hello5")
public String hello5() {
    return "redirect:/user/hello";
}
```

这种，本质上就是浏览器重定向。

- 真的返回一个字符串

上面三个返回的字符串，都是由特殊含义的，如果一定要返回一个字符串，需要额外添加一个注意：@ResponseBody ，这个注解表示当前方法的返回值就是要展示出来返回值，没有特殊含义。

```java
@RequestMapping("/hello5")
@ResponseBody
public String hello5() {
    return "redirect:/user/hello";
}
```

上面代码表示就是想返回一段内容为 `redirect:/user/hello` 的字符串，他没有特殊含义。注意，这里如果单纯的返回一个中文字符串，是会乱码的，可以在 @RequestMapping 中添加 produces 属性来解决：

```java
@RequestMapping(value = "/hello5",produces = "text/html;charset=utf-8")
@ResponseBody
public String hello5() {
    return "Java 语言程序设计";
}
```