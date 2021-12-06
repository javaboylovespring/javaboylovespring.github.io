---
sidebar:
  nav: docs-zh
title: 10. 服务端数据校验
tags: SpringMVC
categories: SpringMVC
abbrlink: validation
date: 2019-11-12 22:28:52
---

## 10. 服务端数据校验

B/S 系统中对 http 请求数据的校验多数在客户端进行，这也是出于简单及用户体验性上考虑，但是在一些安全性要求高的系统中服务端校验是不可缺少的，实际上，几乎所有的系统，凡是涉及到数据校验，都需要在服务端进行二次校验。为什么要在服务端进行二次校验呢？这需要理解客户端校验和服务端校验各自的目的。

1. 客户端校验，我们主要是为了提高用户体验，例如用户输入一个邮箱地址，要校验这个邮箱地址是否合法，没有必要发送到服务端进行校验，直接在前端用 js 进行校验即可。但是大家需要明白的是，前端校验无法代替后端校验，前端校验可以有效的提高用户体验，但是无法确保数据完整性，因为在 B/S 架构中，用户可以方便的拿到请求地址，然后直接发送请求，传递非法参数。
2. 服务端校验，虽然用户体验不好，但是可以有效的保证数据安全与完整性。
3. 综上，实际项目中，两个一起用。

Spring 支持 JSR-303 验证框架，JSR-303 是 JAVA EE 6  中的一项子规范，叫做 Bean Validation，官方参考实现是 Hibernate Validator（与Hibernate ORM 没有关系），JSR-303 用于对 Java Bean 中的字段的值进行验证。

### 10.1 普通校验

普通校验，是这里最基本的用法。

首先，我们需要加入校验需要的依赖：

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.1.0.Final</version>
</dependency>
```

接下来，在 SpringMVC 的配置文件中配置校验的 Bean：

```xml
<bean class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean" id="validatorFactoryBean">
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
</bean>
<mvc:annotation-driven validator="validatorFactoryBean"/>
```

配置时，提供一个 LocalValidatorFactoryBean 的实例，然后 Bean 的校验使用 HibernateValidator。

这样，配置就算完成了。

接下来，我们提供一个添加学生的页面：

```html
<form action="/addstudent" method="post">
    <table>
        <tr>
            <td>学生编号：</td>
            <td><input type="text" name="id"></td>
        </tr>
        <tr>
            <td>学生姓名：</td>
            <td><input type="text" name="name"></td>
        </tr>
        <tr>
            <td>学生邮箱：</td>
            <td><input type="text" name="email"></td>
        </tr>
        <tr>
            <td>学生年龄：</td>
            <td><input type="text" name="age"></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="提交">
            </td>
        </tr>
    </table>
</form>
```

在这里需要提交的数据中，假设学生编号不能为空，学生姓名长度不能超过 10 且不能为空，邮箱地址要合法，年龄不能超过 150。那么在定义实体类的时候，就可以加入这个判断条件了。

```java
public class Student {
    @NotNull
    private Integer id;
    @NotNull
    @Size(min = 2,max = 10)
    private String name;
    @Email
    private String email;
    @Max(150)
    private Integer age;

    public String getEmail() {
        return email;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", age=" + age +
                '}';
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

在这里：

- @NotNull 表示这个字段不能为空
- @Size 中描述了这个字符串长度的限制
- @Email  表示这个字段的值必须是一个邮箱地址
- @Max 表示这个字段的最大值

定义完成后，接下来，在 Controller 中定义接口：

```java
@Controller
public class StudentController {
    @RequestMapping("/addstudent")
    @ResponseBody
    public void addStudent(@Validated Student student, BindingResult result) {
        if (result != null) {
            //校验未通过，获取所有的异常信息并展示出来
            List<ObjectError> allErrors = result.getAllErrors();
            for (ObjectError allError : allErrors) {
                System.out.println(allError.getObjectName()+":"+allError.getDefaultMessage());
            }
        }
    }
}
```

在这里：

- @Validated 表示 Student 中定义的校验规则将会生效
- BindingResult 表示出错信息，如果这个变量不为空，表示有错误，否则校验通过。

接下来就可以启动项目了。访问 jsp 页面，然后添加 Student，查看校验规则是否生效。

默认情况下，打印出来的错误信息时系统默认的错误信息，这个错误信息，我们也可以自定义。自定义方式如下：

由于 properties 文件中的中文会乱码，所以需要我们先修改一下 IDEA 配置，点 File-->Settings->Editor-->File Encodings，如下：

![](http://springmvc.javaboy.org/assets/images/img/10-1.png "10-1.png")

然后定义错误提示文本，在 resources 目录下新建一个 MyMessage.properties 文件，内容如下：

```properties
student.id.notnull=id 不能为空
student.name.notnull=name 不能为空
student.name.length=name 最小长度为 2 ，最大长度为 10
student.email.error=email 地址非法
student.age.error=年龄不能超过 150
```

接下来，在 SpringMVC 配置中，加载这个配置文件：

```xml
<bean class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean" id="validatorFactoryBean">
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
    <property name="validationMessageSource" ref="bundleMessageSource"/>
</bean>
<bean class="org.springframework.context.support.ReloadableResourceBundleMessageSource" id="bundleMessageSource">
    <property name="basenames">
        <list>
            <value>classpath:MyMessage</value>
        </list>
    </property>
    <property name="defaultEncoding" value="UTF-8"/>
    <property name="cacheSeconds" value="300"/>
</bean>
<mvc:annotation-driven validator="validatorFactoryBean"/>
```

最后，在实体类上的注解中，加上校验出错时的信息：

```java
public class Student {
    @NotNull(message = "{student.id.notnull}")
    private Integer id;
    @NotNull(message = "{student.name.notnull}")
    @Size(min = 2,max = 10,message = "{student.name.length}")
    private String name;
    @Email(message = "{student.email.error}")
    private String email;
    @Max(value = 150,message = "{student.age.error}")
    private Integer age;

    public String getEmail() {
        return email;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", age=" + age +
                '}';
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

配置完成后，如果校验再出错，就会展示我们自己的出错信息了。

### 10.2 分组校验

由于校验规则都是定义在实体类上面的，但是，在不同的数据提交环境下，校验规则可能不一样。例如，用户的 id 是自增长的，添加的时候，可以不用传递用户 id，但是修改的时候则必须传递用户 id，这种情况下，就需要使用分组校验。

分组校验，首先需要定义校验组，所谓的校验组，其实就是空接口：

```java
public interface ValidationGroup1 {
}
public interface ValidationGroup2 {
}
```

然后，在实体类中，指定每一个校验规则所属的组：

```java
public class Student {
    @NotNull(message = "{student.id.notnull}",groups = ValidationGroup1.class)
    private Integer id;
    @NotNull(message = "{student.name.notnull}",groups = {ValidationGroup1.class, ValidationGroup2.class})
    @Size(min = 2,max = 10,message = "{student.name.length}",groups = {ValidationGroup1.class, ValidationGroup2.class})
    private String name;
    @Email(message = "{student.email.error}",groups = {ValidationGroup1.class, ValidationGroup2.class})
    private String email;
    @Max(value = 150,message = "{student.age.error}",groups = {ValidationGroup2.class})
    private Integer age;

    public String getEmail() {
        return email;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", age=" + age +
                '}';
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

在 group 中指定每一个校验规则所属的组，一个规则可以属于一个组，也可以属于多个组。

最后，在接收参数的地方，指定校验组：

```java
@Controller
public class StudentController {
    @RequestMapping("/addstudent")
    @ResponseBody
    public void addStudent(@Validated(ValidationGroup2.class) Student student, BindingResult result) {
        if (result != null) {
            //校验未通过，获取所有的异常信息并展示出来
            List<ObjectError> allErrors = result.getAllErrors();
            for (ObjectError allError : allErrors) {
                System.out.println(allError.getObjectName()+":"+allError.getDefaultMessage());
            }
        }
    }
}
```

配置完成后，属于 ValidationGroup2 这个组的校验规则，才会生效。

### 10.3 校验注解

校验注解，主要有如下几种：

- @Null 被注解的元素必须为 null 
- @NotNull 被注解的元素必须不为 null 
- @AssertTrue 被注解的元素必须为 true 
- @AssertFalse 被注解的元素必须为 false 
- @Min(value) 被注解的元素必须是一个数字，其值必须大于等于指定的最小值 
- @Max(value) 被注解的元素必须是一个数字，其值必须小于等于指定的最大值 
- @DecimalMin(value) 被注解的元素必须是一个数字，其值必须大于等于指定的最小值 
- @DecimalMax(value) 被注解的元素必须是一个数字，其值必须小于等于指定的最大值 
- @Size(max=, min=) 被注解的元素的大小必须在指定的范围内 
- @Digits (integer, fraction) 被注解的元素必须是一个数字，其值必须在可接受的范围内 
- @Past 被注解的元素必须是一个过去的日期 
- @Future 被注解的元素必须是一个将来的日期 
- @Pattern(regex=,flag=) 被注解的元素必须符合指定的正则表达式 
- @NotBlank(message =) 验证字符串非 null，且长度必须大于0 
- @Email 被注解的元素必须是电子邮箱地址 
- @Length(min=,max=) 被注解的字符串的大小必须在指定的范围内 
- @NotEmpty 被注解的字符串的必须非空 
- @Range(min=,max=,message=) 被注解的元素必须在合适的范围内