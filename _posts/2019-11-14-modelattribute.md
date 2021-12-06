---
sidebar:
  nav: docs-zh
title: 11.2 @ModelAttribute
tags: SpringMVC
categories: SpringMVC
abbrlink: modelattribute
date: 2019-11-14 22:28:52
---

## 11.2 @ModelAttribute

@ModelAttribute 这个注解，主要有两方面的功能：

1. 在数据回显时，给变量定义别名
2. 定义全局数据

### 11.2.1 定义别名

在数据回显时，给变量定义别名，非常容易，直接加这个注解即可：

```java
@RequestMapping("/addstudent")
public String addStudent(@ModelAttribute("s") @Validated(ValidationGroup2.class) Student student, BindingResult result) {
    if (result != null) {
        //校验未通过，获取所有的异常信息并展示出来
        List<ObjectError> allErrors = result.getAllErrors();
        for (ObjectError allError : allErrors) {
            System.out.println(allError.getObjectName()+":"+allError.getDefaultMessage());
        }
        return "student";
    }
    return "hello";
}
```

这样定义完成后，在前端再次访问回显的变量时，变量名称就不是 student 了，而是 s：

```jsp
<form action="/addstudent" method="post">
    <table>
        <tr>
            <td>学生编号：</td>
            <td><input type="text" name="id" value="${s.id}"></td>
        </tr>
        <tr>
            <td>学生姓名：</td>
            <td><input type="text" name="name" value="${s.name}"></td>
        </tr>
        <tr>
            <td>学生邮箱：</td>
            <td><input type="text" name="email" value="${s.email}"></td>
        </tr>
        <tr>
            <td>学生年龄：</td>
            <td><input type="text" name="age" value="${s.age}"></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="提交">
            </td>
        </tr>
    </table>
</form>
```

### 11.2.2 定义全局数据

假设有一个 Controller 中有很多方法，每个方法都会返回数据给前端，但是每个方法返回给前端的数据又不太一样，虽然不太一样，但是没有方法的返回值又有一些公共的部分。可以将这些公共的部分提取出来单独封装成一个方法，用 @ModelAttribute 注解来标记。

例如在一个 Controller 中 ，添加如下代码：

```java
@ModelAttribute("info")
public Map<String,Object> info() {
    Map<String, Object> map = new HashMap<>();
    map.put("username", "javaboy");
    map.put("address", "www.javaboy.org");
    return map;
}
```

当用户访问当前 Controller 中的任意一个方法，在返回数据时，都会将添加了 @ModelAttribute 注解的方法的返回值，一起返回给前端。@ModelAttribute 注解中的 info 表示返回数据的 key。