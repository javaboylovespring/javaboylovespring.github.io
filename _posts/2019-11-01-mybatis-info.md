---
sidebar:
  nav: docs-zh
title: 1. MyBatis 简介
tags: MyBatis
categories: MyBatis
abbrlink: mybatis-info
date: 2019-11-01 22:28:52
---

## 1. MyBatis 简介

### 1.1 JDBC 存在的问题

<!--more-->


1. 数据库连接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库连接池可解决此问题。
2. Sql 语句在代码中硬编码，造成代码不易维护，实际应用 sql 变化的可能较大，sql 变动需要改变 java 代码。
3. 使用 preparedStatement 向占位符号传参数存在硬编码，因为 sql 语句的 where 条件不一定，可能多也可能少，修改 sql 还要修改代码，系统不易维护。
4. 对结果集解析存在硬编码（查询列名），sql 变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成 pojo 对象解析比较方便。

上面的问题，借助于第三方工具如 DBUtils 或者 Spring 中自带的数据库操作框架 JdbcTemplate，都可以在一定程度上解决该问题。但是不完美，真正能解决这些问题的框架就是两大类，一种就是 MyBatis，另一种则是 Jpa。

### 1.2 MyBatis 介绍

MyBatis 本是 apache 的一个开源项目 iBatis, 2010 年这个项目由 apache software foundation 迁移到了 google code，并且改名为 MyBatis，实质上 Mybatis 对 ibatis 进行一些改进。MyBatis 是一个优秀的持久层框架，它对 jdbc 的操作数据库的过程进行封装，使开发者只需要关注 SQL 本身，而不需要花费精力去处理例如注册驱动、创建 connection、创建 statement、手动设置参数、结果集检索等 jdbc 繁杂的过程代码。Mybatis 通过 xml 或注解的方式将要执行的各种 statement（statement、preparedStatemnt、CallableStatement）配置起来，并通过 java 对象和 statement 中的 sql 进行映射生成最终执行的 sql 语句，最后由 mybatis 框架执行 sql 并将结果映射成 java 对象并返回。

与其他的对象关系映射框架不同，MyBatis 并没有将 Java 对象与数据库表关联起来，而是将 Java 方法与 SQL 语句关联。MyBatis 允许用户充分利用数据库的各种功能，例如存储过程、视图、各种复杂的查询以及某数据库的专有特性。如果要对遗留数据库、不规范的数据库进行操作，或者要完全控制 SQL 的执行，MyBatis 是一个不错的选择。

MyBatis 官网：https://mybatis.org/mybatis-3/zh/index.html

官网有中文版，可以非常方便的学习。