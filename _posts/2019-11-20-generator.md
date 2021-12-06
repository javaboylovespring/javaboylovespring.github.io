---
sidebar:
  nav: docs-zh
title: 11. 逆向工程
tags: MyBatis
categories: MyBatis
abbrlink: generator
date: 2019-11-20 22:28:52
---

## 11. 逆向工程

由于使用数据表时，我们需要给每一个表都创建对应的实体类，每个实体类都有对应的 Mapper 接口和 Mapper.xml 文件，这些其实都是一些重复的工作，我们可以通过第三方工具来完成。

<!--more-->


MyBatis 逆向工程非常多，我们 IDEA 里边，也有对应的插件可以使用。

这里以 mybatis-generator-core-1.3.1 工具为例，拿到工具后，首先解压目录：

![](http://mybatis.javaboy.org/assets/images/img/11-1.png "11-1.png")

注意，这里的数据库驱动以 MySQL5.7 为例，如果自己的 MySQL 不是 5.7，这个驱动要换成适合自己 MySQL 的版本。

然后，我们打开 generator.xml 文件，配置我们需要逆向处理的表。

![](http://mybatis.javaboy.org/assets/images/img/11-2.png "11-2.png")

配置完成后，双击 123.bat 运行项目，运行完成后，将 src 目录下生成的代码拷贝到我们的项目中。