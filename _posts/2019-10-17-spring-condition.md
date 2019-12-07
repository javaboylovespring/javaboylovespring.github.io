---
sidebar:
  nav: docs-zh
title: 3.7 条件注解
tags: Spring
categories: Spring
abbrlink: spring-condition
date: 2019-10-17 22:28:52
---

## 3.7 条件注解

条件注解就是在满足某一个条件的情况下，生效的配置。

<!--more-->

### 3.7.1 条件注解

首先在 Windows 中如何获取操作系统信息？Windows 中查看文件夹目录的命令是 dir，Linux 中查看文件夹目录的命令是 ls，我现在希望当系统运行在 Windows 上时，自动打印出 Windows 上的目录展示命令，Linux 运行时，则自动展示 Linux 上的目录展示命令。

首先定义一个显示文件夹目录的接口：

```java
public interface ShowCmd {
    String showCmd();
}
```

然后，分别实现  Windows 下的实例和 Linux 下的实例：

```java
public class WinShowCmd implements ShowCmd {
    @Override
    public String showCmd() {
        return "dir";
    }
}
public class LinuxShowCmd implements ShowCmd {
    @Override
    public String showCmd() {
        return "ls";
    }
}
```

接下来，定义两个条件，一个是 Windows 下的条件，另一个是 Linux 下的条件。

```java
public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getEnvironment().getProperty("os.name").toLowerCase().contains("windows");
    }
}
public class LinuxCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getEnvironment().getProperty("os.name").toLowerCase().contains("linux");
    }
}
```

接下来，在定义 Bean 的时候，就可以去配置条件注解了。

```java
@Configuration
public class JavaConfig {
    @Bean("showCmd")
    @Conditional(WindowsCondition.class)
    ShowCmd winCmd() {
        return new WinShowCmd();
    }

    @Bean("showCmd")
    @Conditional(LinuxCondition.class)
    ShowCmd linuxCmd() {
        return new LinuxShowCmd();
    }
}
```

这里，一定要给两个 Bean 取相同的名字，这样在调用时，才可以自动匹配。然后，给每一个 Bean 加上条件注解，当条件中的 matches 方法返回 true 的时候，这个 Bean 的定义就会生效。

```java
public class JavaMain {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(JavaConfig.class);
        ShowCmd showCmd = (ShowCmd) ctx.getBean("showCmd");
        System.out.println(showCmd.showCmd());
    }
}
```

条件注解有一个非常典型的使用场景，就是多环境切换。

### 3.7.2 多环境切换

开发中，如何在 开发/生产/测试 环境之间进行快速切换？Spring 中提供了 Profile 来解决这个问题，Profile 的底层就是条件注解。这个从 @Profile 注解的定义就可以看出来：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {

	/**
	 * The set of profiles for which the annotated component should be registered.
	 */
	String[] value();

}
class ProfileCondition implements Condition {

	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
		if (attrs != null) {
			for (Object value : attrs.get("value")) {
				if (context.getEnvironment().acceptsProfiles(Profiles.of((String[]) value))) {
					return true;
				}
			}
			return false;
		}
		return true;
	}

}
```


我们定义一个 DataSource：

```java
public class DataSource {
    private String url;
    private String username;
    private String password;

    @Override
    public String toString() {
        return "DataSource{" +
                "url='" + url + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
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
```

然后，在配置 Bean 时，通过 @Profile 注解指定不同的环境：

```java
@Bean("ds")
@Profile("dev")
DataSource devDataSource() {
    DataSource dataSource = new DataSource();
    dataSource.setUrl("jdbc:mysql://127.0.0.1:3306/dev");
    dataSource.setUsername("root");
    dataSource.setPassword("123");
    return dataSource;
}
@Bean("ds")
@Profile("prod")
DataSource prodDataSource() {
    DataSource dataSource = new DataSource();
    dataSource.setUrl("jdbc:mysql://192.158.222.33:3306/dev");
    dataSource.setUsername("jkldasjfkl");
    dataSource.setPassword("jfsdjflkajkld");
    return dataSource;
}
```

最后，在加载配置类，注意，需要先设置当前环境，然后再去加载配置类：

```java
public class JavaMain {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
        ctx.getEnvironment().setActiveProfiles("dev");
        ctx.register(JavaConfig.class);
        ctx.refresh();
        DataSource ds = (DataSource) ctx.getBean("ds");
        System.out.println(ds);
    }
}
```

这个是在 Java 代码中配置的。环境的切换，也可以在 XML 文件中配置，如下配置在 XML 文件中，必须放在其他节点后面。

```xml
<beans profile="dev">
    <bean class="org.javaboy.DataSource" id="dataSource">
        <property name="url" value="jdbc:mysql:///devdb"/>
        <property name="password" value="root"/>
        <property name="username" value="root"/>
    </bean>
</beans>
<beans profile="prod">
    <bean class="org.javaboy.DataSource" id="dataSource">
        <property name="url" value="jdbc:mysql://111.111.111.111/devdb"/>
        <property name="password" value="jsdfaklfj789345fjsd"/>
        <property name="username" value="root"/>
    </bean>
</beans>
```

启动类中设置当前环境并加载配置：

```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext();
        ctx.getEnvironment().setActiveProfiles("prod");
        ctx.setConfigLocation("applicationContext.xml");
        ctx.refresh();
        DataSource dataSource = (DataSource) ctx.getBean("dataSource");
        System.out.println(dataSource);
    }
}
```