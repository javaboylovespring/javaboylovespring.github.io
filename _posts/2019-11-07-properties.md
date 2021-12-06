---
sidebar:
  nav: docs-zh
title: 6.1 properties
tags: MyBatis
categories: MyBatis
abbrlink: properties
date: 2019-11-07 22:28:52
---

### 6.1 properties

properties 可以用来引入一个外部配置，最近常见的例子就是引入数据库的配置文件，例如我们在 resources 目录下添加一个 db.properties 文件作为数据库的配置文件，文件内容如下：

<!--more-->


```properties
db.username=root
db.password=123
db.driver=com.mysql.cj.jdbc.Driver
db.url=jdbc:mysql:///test01?serverTimezone=Asia/Shanghai
```

然后，利用 mybatis-config.xml 配置文件中的 properties 属性，引入这个配置文件，然后在 DataSource 中使用这个配置文件，最终配置如下：

```xml
<configuration>
    <properties resource="db.properties"></properties>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${db.driver}"/>
                <property name="url" value="${db.url}"/>
                <property name="username" value="${db.username}"/>
                <property name="password" value="${db.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <package name="org.javaboy.mybatis.mapper"/>
    </mappers>
</configuration>
```