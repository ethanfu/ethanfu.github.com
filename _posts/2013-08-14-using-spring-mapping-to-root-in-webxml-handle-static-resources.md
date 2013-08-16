---
layout: post
title: "Using spring mapping to root in web.xml handle static resources"
description: ""
category: Java
tags: [SpringMVC，Spring,java]
---
{% include JB/setup %}

现在来说很多人都已经开始使用 Spring MVC 来做前端框架，这框架和 Spring 无缝集成，并且注解、RestURL等的支持也都非常符合我们的需求，所以在部分工作中我也使用过，这次对一个产品的 URL 进行改造，想改成基于 REST风格的方式，其中遇到了一个问题，在这里记录下，其实也不复杂，只是看你看文档是否够用心。

## 问题

大家在使用 Spring MVC 的时候，肯定需要在 web.xml 中配置一个 servlet 和 Mapping ，如下：

	<servlet>
        <servlet-name>springServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>springServlet</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
    
其中 `url-pattern` 这个地方大家很多人还是遵循了以前 Struts 的命名方式，如果你使用基于 REST 风格的 URL 的时候，这样配置就不合时宜了。之所以以前做这个选择自然是有原因的，毕竟一些静态内容或者 jsp 等其他的资源就不需要你这个 servlet 来处理，交给 Tomcat 等容器来给你完成就可以了。所以加了这个扩展名就可以很轻松的处理。而我现在需要的可能是需要如下配置才可：

	<servlet-mapping>
        <servlet-name>springServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    
当你配置的拦截是 / 的时候这个时候问题就来了，你的静态内容和jsp应该都不能访问了。

## 解决

以前一般都是采用如下方式（我这里只是举例 Java Config的方式，以前还是xml配置比较多）：
	
	@EnableWebMvc	@Configuration	public class WebConfig extends WebMvcConfigurerAdapter {			@Override		public void addResourceHandlers(ResourceHandlerRegistry registry) {	    	registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");	}
这个方式其实类似白名单，就是添加例外，但是jsp文件的话还是有问题的。这个时候按照我以前逻辑如果不考虑REST这种 URL 的话就可以有以下几种处理方式：
1. 上面例子，添加特殊扩展，继续添加.do等扩展名
2. 添加前缀，例如`<url-pattern>/test/</url-pattern>`

这两种方式都不太能完全满足我的需求，幸好我在 Spring 的 reference 中看到`mvc:default-servlet-handler`这个章节。这玩意的出现能完全满足我的需要，只需要你在配置文件中配置好即可了。这个地方原理其实很简单，这个配置其实就是拦截了`/**`，这个拦截比`/`优先级低，也就是说处理不了的继续交还给 default Servlet 。

我这里出的问题是没太仔细看文档，导致配置出现了一点差错，然后还浪费了一点时间在[stackoverflow](http://stackoverflow.com)逛了一会，也没找到啥思路，我都想提问题了，还好又想到了官方文档，仔细看了下就把问题解决了。