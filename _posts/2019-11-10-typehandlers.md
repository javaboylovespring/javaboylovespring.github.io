---
sidebar:
  nav: docs-zh
title: 6.4 typeHandlers
tags: MyBatis
categories: MyBatis
abbrlink: typehandlers
date: 2019-11-10 22:28:52
---

### 6.4 typeHandlers

在 MyBatis 映射中，能够自动将 Jdbc 类型映射为 Java 类型。默认的映射规则，如下：

<!--more-->


|类型处理器	           |Java类型	             |JDBC类型                                                        |
|:---|:---|:---|
|BooleanTypeHandler 	   |Boolean，boolean      |	任何兼容的布尔值                                              |
|ByteTypeHandler 	   |Byte，byte 	         | 任何兼容的数字或字节类型                                           |
|ShortTypeHandler 	   |Short，short 	     |任何兼容的数字或短整型                                              |
|IntegerTypeHandler 	   |Integer，int 	     |任何兼容的数字和整型                                            |
|LongTypeHandler 	   |Long，long 	         |任何兼容的数字或长整型                                              |
|FloatTypeHandler 	   |Float，float 	     |任何兼容的数字或单精度浮点型                                        |
|DoubleTypeHandler 	   |Double，double 	     |任何兼容的数字或双精度浮点型                                        |
|BigDecimalTypeHandler 	|BigDecimal 	         |任何兼容的数字或十进制小数类型                                  |
|StringTypeHandler 	   | String 	             |CHAR和VARCHAR类型                                               |
|ClobTypeHandler 	   | String 	             |CLOB和LONGVARCHAR类型                                           |
|NStringTypeHandler 	   | String 	             |NVARCHAR和NCHAR类型                                         |
|NClobTypeHandler 	   | String 	             |NCLOB类型                                                       |
|ByteArrayTypeHandler 	|byte[] 	             |任何兼容的字节流类型                                            |
|BlobTypeHandler 	   |  byte[]              |	BLOB和LONGVARBINARY类型                                           |
|DateTypeHandler 	   |  Date（java.util）   |	TIMESTAMP类型                                                     |
|DateOnlyTypeHandler 	|Date（java.util）   |	DATE类型                                                          |
|TimeOnlyTypeHandler 	|Date（java.util）   |	TIME类型                                                          |
|SqlTimestampTypeHandler| Timestamp（java.sql）|	TIMESTAMP类型                                                 |
|SqlDateTypeHandler 	   | Date（java.sql）	 |  DATE类型                                                      |
|SqlTimeTypeHandler 	   | Time（java.sql）	 |  TIME类型                                                      |
|ObjectTypeHandler 	   | 任意	             |其他或未指定类型                                                    |
|EnumTypeHandler 	   | Enumeration类型	    |  VARCHAR-任何兼容的字符串类型，作为代码存储（而不是索引）。     |

前面案例中，之所以数据能够接收成功，是因为有上面这些默认的类型处理器，处理基本数据类型，这些够用了，特殊类型，需要我们自定义类型处理器。

比如，我有一个用户爱好的字段，这个字段，在对象中，是一个 List 集合，在数据库中，是一个 VARCHAR 字段，这种情况下，就需要我们自定义类型转换器，自定义的类型转换器提供两个功能：

1. 数据存储时，自动将 List 集合，转为字符串（格式自定义）
2. 数据查询时，将查到的字符串再转为 List 集合

首先，在数据表中添加一个 favorites 字段：

![](6-4-1.png "6-4-1.png")

然后，在 User 对象中，添加相应的属性：

```java
public class User {
    private Integer id;
    private String username;
    private String address;
    private List<String> favorites;

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

为了能够将 List 集合中的数据存入到 VARCHAR 中，我们需要自定义一个类型转换器：

```java
@MappedJdbcTypes(JdbcType.VARCHAR)
@MappedTypes(List.class)
public class List2VarcharHandler implements TypeHandler<List<String>> {
    public void setParameter(PreparedStatement ps, int i, List<String> parameter, JdbcType jdbcType) throws SQLException {
        StringBuffer sb = new StringBuffer();
        for (String s : parameter) {
            sb.append(s).append(",");
        }
        ps.setString(i, sb.toString());
    }

    public List<String> getResult(ResultSet rs, String columnName) throws SQLException {
        String favs = rs.getString(columnName);
        if (favs != null) {
            return Arrays.asList(favs.split(","));
        }
        return null;
    }

    public List<String> getResult(ResultSet rs, int columnIndex) throws SQLException {
        String favs = rs.getString(columnIndex);
        if (favs != null) {
            return Arrays.asList(favs.split(","));
        }
        return null;
    }

    public List<String> getResult(CallableStatement cs, int columnIndex) throws SQLException {
        String favs = cs.getString(columnIndex);
        if (favs != null) {
            return Arrays.asList(favs.split(","));
        }
        return null;
    }
}
```

- 首先在这个自定义的类型转换器上添加 @MappedJdbcTypes 注解指定要处理的 Jdbc 数据类型，另外还有一个注解是 @MappedTypes 指定要处理的 Java 类型，这两个注解结合起来，就可以锁定要处理的字段是 favorites 了。
- setParameter 方法看名字就知道是设置参数的，这个设置过程由我们手动实现，我们在这里，将 List 集合中的每一项，用一个 , 串起来，组成一个字符串。
- getResult 方法，有三个重载方法，其实都是处理查询的。

接下来，修改插入的 Mapper：

```xml
<insert id="addUser" parameterType="org.javaboy.mybatis.model.User">
    insert into user (username,address,favorites) values (#{username},#{address},#{favorites,typeHandler=org.javaboy.mybatis.typehandler.List2VarcharHandler});
</insert>
```

然后，在 Java 代码中，调用该方法：

```java
public class Main2 {
    public static void main(String[] args) {
        SqlSessionFactory instance = SqlSessionFactoryUtils.getInstance();
        SqlSession sqlSession = instance.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = new User();
        user.setUsername("风气");
        user.setAddress("上海");
        List<String> favorites = new ArrayList<String>();
        favorites.add("足球");
        favorites.add("篮球");
        favorites.add("乒乓球");
        user.setFavorites(favorites);
        mapper.addUser(user);
        sqlSession.commit();
    }
}
```

这样，List 集合存入到数据库中之后，就变成一个字符串了：

![](6-4-2.png "6-4-2.png")

读取的配置，有两个地方，一个可以在 ResultMap 中做局部配置，也可以在全局配置中进行过配置，全局配置方式如下：

```xml
<configuration>
    <properties resource="db.properties"></properties>
    <typeAliases>
        <package name="org.javaboy.mybatis.model"/>
    </typeAliases>
    <typeHandlers>
        <package name="org.javaboy.mybatis.typehandler"/>
    </typeHandlers>
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

接下来去查询，查询过程中，就会自动将字符串转为 List 集合了。