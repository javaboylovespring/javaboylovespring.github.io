---
sidebar:
  nav: docs-zh
title: 9. 一对多查询
tags: MyBatis
categories: MyBatis
abbrlink: one-to-many
date: 2019-11-18 22:28:52
---

## 9. 一对多查询

一对多查询，也是一个非常典型的使用场景。比如用户和角色的关系，一个用户可以具备多个角色。

<!--more-->


首先我们准备三个表：

```sql
/*
Navicat MySQL Data Transfer

Source Server         : localhost
Source Server Version : 50717
Source Host           : localhost:3306
Source Database       : security

Target Server Type    : MYSQL
Target Server Version : 50717
File Encoding         : 65001

Date: 2018-07-28 15:26:51
*/

SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for role
-- ----------------------------
DROP TABLE IF EXISTS `role`;
CREATE TABLE `role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL,
  `nameZh` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of role
-- ----------------------------
INSERT INTO `role` VALUES ('1', 'dba', '数据库管理员');
INSERT INTO `role` VALUES ('2', 'admin', '系统管理员');
INSERT INTO `role` VALUES ('3', 'user', '用户');

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(32) DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  `enabled` tinyint(1) DEFAULT NULL,
  `locked` tinyint(1) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('1', 'root', '$2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq', '1', '0');
INSERT INTO `user` VALUES ('2', 'admin', '$2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq', '1', '0');
INSERT INTO `user` VALUES ('3', 'sang', '$2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq', '1', '0');

-- ----------------------------
-- Table structure for user_role
-- ----------------------------
DROP TABLE IF EXISTS `user_role`;
CREATE TABLE `user_role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `uid` int(11) DEFAULT NULL,
  `rid` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of user_role
-- ----------------------------
INSERT INTO `user_role` VALUES ('1', '1', '1');
INSERT INTO `user_role` VALUES ('2', '1', '2');
INSERT INTO `user_role` VALUES ('3', '2', '2');
INSERT INTO `user_role` VALUES ('4', '3', '3');
SET FOREIGN_KEY_CHECKS=1;

```

这三个表中，有用户表，角色表以及用户角色关联表，其中用户角色关联表用来描述用户和角色之间的关系，他们是一对多的关系。

然后，根据这三个表，创建两个实体类：

```java
public class User {
    private Integer id;
    private String username;
    private String password;
    private List<Role> roles;

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", roles=" + roles +
                '}';
    }

    public List<Role> getRoles() {
        return roles;
    }

    public void setRoles(List<Role> roles) {
        this.roles = roles;
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

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
public class Role {
    private Integer id;
    private String name;
    private String nameZh;

    @Override
    public String toString() {
        return "Role{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", nameZh='" + nameZh + '\'' +
                '}';
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

    public String getNameZh() {
        return nameZh;
    }

    public void setNameZh(String nameZh) {
        this.nameZh = nameZh;
    }
}
```

接下来，定义一个根据 id 查询用户的方法：

```java
User getUserById(Integer id);
```

然后，定义该方法的实现：

```xml
<resultMap id="UserWithRole" type="org.javaboy.mybatis.model.User">
    <id column="id" property="id"/>
    <result column="username" property="username"/>
    <result column="password" property="password"/>
    <collection property="roles" ofType="org.javaboy.mybatis.model.Role">
        <id property="id" column="rid"/>
        <result property="name" column="rname"/>
        <result property="nameZh" column="rnameZH"/>
    </collection>
</resultMap>
<select id="getUserById" resultMap="UserWithRole">
    SELECT u.*,r.`id` AS rid,r.`name` AS rname,r.`nameZh` AS rnameZh FROM USER u,role r,user_role ur WHERE u.`id`=ur.`uid` AND ur.`rid`=r.`id` AND u.`id`=#{id}
</select>
```

在 resultMap 中，通过 collection 节点来描述集合的映射关系。在映射时，会自动将一的一方数据集合并，然后将多的一方放到集合中，能实现这一点，靠的就是 id 属性。

当然，这个一对多，也可以做成懒加载的形式，那我们首先提供一个角色查询的方法：

```java
List<Role> getRolesByUid(Integer id);
```

然后，在 XML 文件中，处理懒加载：

```xml
<resultMap id="UserWithRole" type="org.javaboy.mybatis.model.User">
    <id column="id" property="id"/>
    <result column="username" property="username"/>
    <result column="password" property="password"/>
    <collection property="roles" select="org.javaboy.mybatis.mapper.UserMapper.getRolesByUid" column="id" fetchType="lazy">
    </collection>
</resultMap>
<select id="getUserById" resultMap="UserWithRole">
    select * from user  where id=#{id};
</select>
<select id="getRolesByUid" resultType="org.javaboy.mybatis.model.Role">
    SELECT r.* FROM role r,user_role ur WHERE r.`id`=ur.`rid` AND ur.`uid`=#{id}
</select>
```

定义完成之后，我们的查询操作就实现了懒加载功能。