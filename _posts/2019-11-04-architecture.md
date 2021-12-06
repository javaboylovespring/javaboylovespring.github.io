---
sidebar:
  nav: docs-zh
title: 4. MyBatis 架构介绍
tags: MyBatis
categories: MyBatis
abbrlink: architecture
date: 2019-11-04 22:28:52
---

## 4. MyBatis 架构介绍

看完前面的 HelloWorld，接下来我们通过一张网络图片来看下 MyBatis 架构：

<!--more-->


![](http://mybatis.javaboy.org/assets/images/img/4-1.png "4-1.png")

1. mybatis 配置:mybatis-config.xml，此文件作为 mybatis 的全局配置文件，配置了 mybatis 的运行环境等信息。另一个 mapper.xml 文件即 sql 映射文件，文件中配置了操作数据库的 sql 语句。此文件需要在 mybatis-config.xml 中加载。
2. 通过 mybatis 环境等配置信息构造 SqlSessionFactory 即会话工厂
3. 由会话工厂创建 sqlSession 即会话，操作数据库需要通过 sqlSession 进行。
4. mybatis 底层自定义了 Executor 执行器接口操作数据库，Executor 接口有两个实现，一个是基本执行器、一个是缓存执行器。
5. Mapped Statement 也是 mybatis 一个底层封装对象，它包装了 mybatis 配置信息及 sql 映射信息等。mapper.xml 文件中一个 sql 对应一个 Mapped Statement 对象，sql 的 id 即是Mapped statement 的 id。
6. Mapped Statement 对 sql 执行输入参数进行定义，包括 HashMap、基本类型、pojo，Executor 通过 Mapped Statement 在执行 sql 前将输入的 java 对象映射至 sql 中，输入参数映射就是 jdbc 编程中对 preparedStatement 设置参数。
7. Mapped Statement 对 sql 执行输出结果进行定义，包括 HashMap、基本类型、pojo，Executor 通过 Mapped Statement 在执行 sql 后将输出结果映射至 java 对象中，输出结果映射过程相当于 jdbc 编程中对结果的解析处理过程。


**MyBatis 所解决的 JDBC 中存在的问题**

1. 数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。解决：在 mybatis-config.xml 中配置数据链接池，使用连接池管理数据库链接。
2. Sql语句写在代码中造成代码不易维护，实际应用 sql 变化的可能较大，sql 变动需要改变 java 代码。解决：将 Sql 语句配置在 XXXXmapper.xml 文件中与 java 代码分离。
3. 向 sql 语句传参数麻烦，因为 sql 语句的 where 条件不一定，可能多也可能少，占位符需要和参数一一对应。解决：Mybatis 自动将 java 对象映射至 sql 语句，通过 statement 中的 parameterType 定义输入参数的类型。
4. 对结果集解析麻烦，sql 变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成 pojo 对象解析比较方便。解决：Mybatis 自动将 sql 执行结果映射至 java 对象，通过 statement 中的 resultType 定义输出结果的类型。