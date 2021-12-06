---
sidebar:
  nav: docs-zh
title: 12.2 接收 JSON
tags: SpringMVC
categories: SpringMVC
abbrlink: receive-json
date: 2019-11-16 22:28:52
---

## 12.2 接收 JSON

浏览器传来的参数，可以是 key/value 形式的，也可以是一个 JSON 字符串。在 Jsp/Servlet 中，我们接收 key/value 形式的参数，一般是通过 getParameter 方法。如果客户端商户惨的是 JSON 数据，我们可以通过如下格式进行解析：

```java
@RequestMapping("/addbook2")
@ResponseBody
public void addBook2(HttpServletRequest req) throws IOException {
    ObjectMapper om = new ObjectMapper();
    Book book = om.readValue(req.getInputStream(), Book.class);
    System.out.println(book);
}
```

但是这种解析方式有点麻烦，在 SpringMVC 中，我们可以通过一个注解来快速的将一个 JSON 字符串转为一个对象：

```java
@RequestMapping("/addbook3")
@ResponseBody
public void addBook3(@RequestBody Book book) {
    System.out.println(book);
}
```

这样就可以直接收到前端传来的 JSON 字符串了。这也是 HttpMessageConverter 提供的第二个功能。