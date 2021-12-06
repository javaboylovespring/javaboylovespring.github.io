---
sidebar:
  nav: docs-zh
title: 7.3 resultMap
tags: MyBatis
categories: MyBatis
abbrlink: resultmap
date: 2019-11-15 22:28:52
---

### 7.3 resultMap

在实际开发中，resultMap 是使用较多的返回数据类型配置。因为实际项目中，一般的返回数据类型比较丰富，要么字段和属性对不上，要么是一对一、一对多的查询，等等，这些需求，单纯的使用 resultType 是无法满足的，因此我们还需要使用 resultMap，也就是自己定义映射的结果集。

<!--more-->


先来看一个基本用法：

首先在 mapper.xml 中定义一个 resultMap：

```xml
<resultMap id="MyResultMap" type="org.javaboy.mybatis.model.User">
    <id column="id" property="id"/>
    <result column="username" property="username"/>
    <result column="address" property="address"/>
</resultMap>
```
在这个 resultMap 中，id 用来描述主键，column 是数据库查询出来的列名，property 则是对象中的属性名。

然后在查询结果中，定义返回值时使用这个 ResultMap：

```xml
<select id="getUserById" resultMap="MyResultMap">
    select * from user where id=#{id};
</select>
```

**注意，在旧版的 MyBatis 中，要求实体类一定要有一个无参构造方法，新版的 MyBatis 没有这个要求。**

当然，我们也可以在 resultMap 中，自己指定要调用的构造方法，指定方式如下：

```xml
<resultMap id="MyResultMap" type="org.javaboy.mybatis.model.User">
    <constructor>
        <idArg column="id" name="id"/>
        <arg column="username" name="username"/>
    </constructor>
</resultMap>
```

这个就表示使用两个参数的构造方法取构造一个 User 实例。注意，name 属性表示构造方法中的变量名，默认情况下，变量名是 arg0、arg1、、、、或者 param1、param2、、、，如果需要自定义，我们可以在构造方法中，手动加上 @Param 注解。

```java
public class User {
    private Integer id;
    private String username;
    private String address;
    private List<String> favorites;

    public User(@Param("id") Integer id, @Param("username") String username) {
        this.id = id;
        this.username = username;
        System.out.println("--------------------");
    }

    public User(Integer id, String username, String address, List<String> favorites) {
        this.id = id;
        this.username = username;
        this.address = address;
        this.favorites = favorites;
        System.out.println("-----------sdfasfd---------");
    }

    public List<String> getFavorites() {
        return favorites;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", address='" + address + '\'' +
                ", favorites=" + favorites +
                '}';
    }

    public void setFavorites(List<String> favorites) {
        this.favorites = favorites;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```