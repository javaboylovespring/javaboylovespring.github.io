---
sidebar:
  nav: docs-zh
title: 1. SpringMVC 简介
tags: SpringMVC
categories: SpringMVC
abbrlink: springmvc-info
date: 2019-11-01 22:28:52
---

## 1. SpringMVC 简介

### 1.1 Spring Web MVC是什么

Spring Web MVC 是一种基于 Java 的实现了 Web MVC 设计模式的请求驱动类型的轻量级 Web 框架，即使用了 MVC 架构模式的思想，将 web 层进行职责解耦，基于请求驱动指的就是使用请求-响应模型，框架的目的就是帮助我们简化开发，Spring Web MVC 也是要简化我们日常 Web 开发的。在 传统的 Jsp/Servlet 技术体系中，如果要开发接口，一个接口对应一个 Servlet，会导致我们开发出许多 Servlet，使用 SpringMVC 可以有效的简化这一步骤。

Spring Web MVC 也是服务到工作者模式的实现，但进行可优化。前端控制器是 DispatcherServlet；应用控制器可以拆为处理器映射器(Handler Mapping)进行处理器管理和视图解析器(View Resolver)进行视图管理；页面控制器/动作/处理器为 Controller 接口（仅包含 ModelAndView handleRequest(request, response) 方法，也有人称作 Handler）的实现（也可以是任何的 POJO 类）；支持本地化（Locale）解析、主题（Theme）解析及文件上传等；提供了非常灵活的数据验证、格式化和数据绑定机制；提供了强大的约定大于配置（惯例优先原则）的契约式编程支持。

### 1.2 Spring Web MVC能帮我们做什么

- 让我们能非常简单的设计出干净的 Web 层和薄薄的 Web 层；
- 进行更简洁的 Web 层的开发；
- 天生与 Spring 框架集成（如 IoC 容器、AOP 等）；
- 提供强大的约定大于配置的契约式编程支持；
- 能简单的进行 Web 层的单元测试；
- 支持灵活的 URL 到页面控制器的映射；
- 非常容易与其他视图技术集成，如 Velocity、FreeMarker 等等，因为模型数据不放在特定的 API 里，而是放在一个 Model 里（Map 数据结构实现，因此很容易被其他框架使用）；
- 非常灵活的数据验证、格式化和数据绑定机制，能使用任何对象进行数据绑定，不必实现特定框架的 API；
- 提供一套强大的 JSP 标签库，简化 JSP 开发；
- 支持灵活的本地化、主题等解析；
- 更加简单的异常处理；
- 对静态资源的支持；
- 支持 RESTful 风格