---
sidebar:
  nav: docs-zh
title: 7.2 resultType
tags: MyBatis
categories: MyBatis
abbrlink: resulttype
date: 2019-11-14 22:28:52
---

### 7.2 resultType

resultType 是返回类型，在实际开发中，如果返回的数据类型比较复杂，一般我们使用 resultMap，但是，对于一些简单的返回，使用 resultType 就够用了。

<!--more-->


resultType 返回的类型可以是简单类型，可以是对象，可以是集合，也可以是一个 hashmap，如果是 hashmap，map 中的 key 就是字段名，value 就是字段的值。

输出 pojo 对象和输出 pojo 列表在 sql 中定义的 resultType 是一样的。
返回单个 pojo 对象要保证 sql 查询出来的结果集为单条，内部使用 sqlSession.selectOne 方法调用，mapper 接口使用 pojo 对象作为方法返回值。返回 pojo 列表表示查询出来的结果集可能为多条，内部使用 sqlSession.selectList 方法，mapper 接口使用 List<pojo> 对象作为方法返回值。