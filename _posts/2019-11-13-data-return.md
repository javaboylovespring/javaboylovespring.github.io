---
sidebar:
  nav: docs-zh
title: 11.1 数据回显基本用法
tags: SpringMVC
categories: SpringMVC
abbrlink: data-return
date: 2019-11-13 22:28:52
---

## 11.1 数据回显基本用法

数据回显就是当用户数据提交失败时，自动填充好已经输入的数据。一般来说，如果使用 Ajax 来做数据提交，基本上是没有数据回显这个需求的，但是如果是通过表单做数据提交，那么数据回显就非常有必要了。

### 11.1.1 简单数据类型

简单数据类型，实际上框架在这里没有提供任何形式的支持，就是我们自己手动配置。我们继续在第 10 小节的例子上演示 Demo。加入提交的 Student 数据不符合要求，那么重新回到添加 Student 页面，并且预设之前已经填好的数据。

首先我们先来改造一下 student.jsp 页面：

```jsp
<form action="/addstudent" method="post">
    <table>
        <tr>
            <td>学生编号：</td>
            <td><input type="text" name="id" value="${id}"></td>
        </tr>
        <tr>
            <td>学生姓名：</td>
            <td><input type="text" name="name" value="${name}"></td>
        </tr>
        <tr>
            <td>学生邮箱：</td>
            <td><input type="text" name="email" value="${email}"></td>
        </tr>
        <tr>
            <td>学生年龄：</td>
            <td><input type="text" name="age" value="${age}"></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="提交">
            </td>
        </tr>
    </table>
</form>
```

在接收数据时，使用简单数据类型去接收：

```java
@RequestMapping("/addstudent")
public String addStudent2(Integer id, String name, String email, Integer age, Model model) {
    model.addAttribute("id", id);
    model.addAttribute("name", name);
    model.addAttribute("email", email);
    model.addAttribute("age", age);
    return "student";
}
```

这种方式，相当于框架没有做任何工作，就是我们手动做数据回显的。此时访问页面，服务端会再次定位到该页面，而且数据已经预填好。

### 11.1.2 实体类

上面这种简单数据类型的回显，实际上非常麻烦，因为需要开发者在服务端一个一个手动设置。如果使用对象的话，就没有这么麻烦了，因为 SpringMVC 在页面跳转时，会自动将对象填充进返回的数据中。

此时，首先修改一下 student.jsp 页面：

```jsp
<form action="/addstudent" method="post">
    <table>
        <tr>
            <td>学生编号：</td>
            <td><input type="text" name="id" value="${student.id}"></td>
        </tr>
        <tr>
            <td>学生姓名：</td>
            <td><input type="text" name="name" value="${student.name}"></td>
        </tr>
        <tr>
            <td>学生邮箱：</td>
            <td><input type="text" name="email" value="${student.email}"></td>
        </tr>
        <tr>
            <td>学生年龄：</td>
            <td><input type="text" name="age" value="${student.age}"></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="提交">
            </td>
        </tr>
    </table>
</form>
```

注意，在预填数据中，多了一个 student. 前缀。这 student 就是服务端接收数据的变量名，服务端的变量名和这里的 student 要保持一直。服务端定义如下：

```java
@RequestMapping("/addstudent")
public String addStudent(@Validated(ValidationGroup2.class) Student student, BindingResult result) {
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

注意，服务端什么都不用做，就说要返回的页面就行了，student 这个变量会被自动填充到返回的 Model 中。变量名就是填充时候的 key。如果想自定义这个 key，可以在参数中写出来 Model，然后手动加入 Student 对象，就像简单数据类型回显那样。

另一种定义回显变量别名的方式，就是使用 @ModelAttribute 注解。