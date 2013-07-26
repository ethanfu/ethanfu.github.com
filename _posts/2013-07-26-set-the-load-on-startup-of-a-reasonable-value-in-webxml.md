---
layout: post
title: "web.xml中load-on-startup引发的讨论"
description: ""
category: java
tags: [java]
---

近期工作需要把公司不少项目把构建方式转移到maven上来，工作量还真不小，遇到各种问题，还算开心的事情就是每个问题都能解决，今天又遇到一问题，深究下来事情还不少，值得说道下。

个人习惯在maven项目使用了jetty的插件来运行项目，就不要问我为啥非使用jetty了，tomcat也可以没啥特殊爱好。项目改造好以后我又熟练的在IntelliJ中执行 `jetty:run` ,这次事情不太妙，提示以下错误：

	DWR can't find a spring config. See following info logs for solutions
	- Option 1. In dwr.xml, <create creator='spring' ...> add <param name='location1' value='beans.xml'/> for each spring config file.
	- Option 2. Use a spring org.springframework.web.context.ContextLoaderListener.
	- Option 3. Call SpringCreator.setOverrideBeanFactory() from your web-app

最后错误的堆栈可以定位到 SpringCreator 这个类上，其实这个问题大家搜下大部分文章都是告诉你添加 Spring的一个 ContextLoadListener 到 web.xml中即可。公司的框架没使用这个东西，加载容器的部分是自己实现的。最后根据堆栈可以得出的信息就是没拿到 Spring 的 BeanFactory ，也就是说容器没加载对。

之所以加 ContextLoadListener 能解决问题，是因为这样 Spring 的容器加载优先于 dwr 的 servlet配置，所以就没问题，我这里加载容器的类放在了一个 servlet 的init-class的配置中了。那问题到这里就可以断定肯定是 servlet 加载顺序出了问题，才导致了这个报错。

这个问题牵扯到了 web.xml 中 servlet 的执行顺序，这也是我之所以想把这事拿出来说的原因之一吧，大家可以在百度或者 Google 去搜索类似的中文文章，基本有两类：

* 告诉你顺序是 `listener >> filter >> servlet ` ，这个好似对我们这个问题没帮助，如果使用了 listener 就没这事了
* 告诉你 servlet 的执行顺序，这个我开始还看到了两种说法，一个是依赖于 `load-on-startup` 的配置（这个是合理的），一个是依赖于 `servlet-mapping` 的书写顺序（这个说法有点坑爹吧？你以为这个是 filter-mapping 啊）

这样基本大体思路有了，就看项目中 `load-on-startup` 的配置就可以了，看了下还真有点问题，本来一定要第一位置执行的 servlet 的值是1 ，dwr 的值是 -1 ，这个时候你肯定要去上网看看这个顺序是怎么回事了，我又搜了点比较坑爹的资料：
> 当是一个负数时或者没有指定时，则指示容器在该servlet被选择时才加载

这个解释有点太坑爹了，估计还真有不少人是因为这句话而选择的 -1 ，我问了下同事当时做的这个选择还真是这个原因，有时候中文资料害死人啊，尼玛。

由于这个配置是 web.xml 的，所以这个时候第一反应应该是看 JSR 是否有定义这个玩意，看了下 [JSR154-Servlet 2.4](http://download.oracle.com/otn-pub/jcp/7216-servlet-2.4-pfd3-spec-oth-JSpec/servlet-2_4-pfd3-spec.pdf?AuthParam=1374820757_f504e904794456b3d50644ed3e3c8b02) 还真有，人家是这样描述的：
>The element load-on-startup indicates that this servlet should be loaded (instantiated and have its init() called) on the startup of the Web application. The element content of this element must be an integer indicating the order in which the servlet should be loaded. If the value is a negative integer, or the element is not present, the container is free to load the servlet whenever it chooses. If the value is a positive integer or 0, the container must load and initialize the servlet as the application is deployed. The container must guarantee that servlets marked with lower integers are loaded before servlets marked with higher integers. The container may choose the order of loading of servlets with the same load-on-startup value. 

因为加载的时候需要根据这个值来排序，所以这个节点的值得是integer。如果是负值或者没指定的话容器可以自由实现。否则就应该按照值的大小，又小到大排列，如果值相等的话，容器也可以选择自己的排序策略去加载。

看到这个一下子就明白了我的程序为啥出问题了。肯定是因为 -1 ，人家 jetty 自己的策略和其他的不太一样导致的，解决问题就简单了，既然你有个 servlet 需要在第一位，那就把配置的值调整为0，其他的 servlet 都一律调整为大于0的即可。

## 题外话
既然每个容器实现的策略可能不一致，很多程序员这个时候都忍不住看下每个容器的实现方式了，我就属于没忍住的一个。

### Jetty
先看 jetty 的，我这里看的是 maven 的 jetty 插件版本为 `9.0.1-SNAPSHOT` 的代码。首先对所有servlet进行 `Array.sort(servlets)`,看到这个基本可以猜测这个对象实现了 `compareTo` 的方法了，下面是代码：

	/* ------------------------------------------------------------ */
    /** Comparitor by init order.
     */
    @Override
    public int compareTo(ServletHolder sh)
    {
        if (sh==this)
            return 0;
        if (sh._initOrder<_initOrder)
            return 1;
        if (sh._initOrder>_initOrder)
            return -1;

        int c=(_className!=null && sh._className!=null)?_className.compareTo(sh._className):0;
        if (c==0)
            c=_name.compareTo(sh._name);
        if (c==0)
            c=this.hashCode()>sh.hashCode()?1:-1;
            return c;
    }
先根据数字大小来排列，如果在相等的情况下，就再对比 className，name，最后还尼玛比不出来就比 hashCode。 相等的情况就是JSR规定的自己的排序策略了，还有一个就是在负值的时候，策略也很简单，就是看数字大小对比了，这个和接下来的 Tomcat 就完全不一样了。
### Tomcat
Tomcat 的方式就直接先来个代码展示下对负值的处理方式：

	org.apache.catalina.core.StandardWrapper
	/**
     * Return the load-on-startup order value (negative value means
     * load on first call).
     */
    public int getLoadOnStartup() {

        if (isJspServlet && loadOnStartup < 0) {
            /*
             * JspServlet must always be preloaded, because its instance is
             * used during registerJMX (when registering the JSP
             * monitoring mbean)
             */
             return Integer.MAX_VALUE;  //这个值为0x7fffffff
        } else {
            return (this.loadOnStartup);
        }
    }
    
 这个地方足矣说明和 jetty 的不同了，估计对相等的情况和 jetty 应该差不多，这里就暂时不介绍了。
### WebLogic
WebLogic由于是不是开源的项目，代码肯定是看不了了，只能从官方文档来了解了，参照[WebLogic 8.1](http://docs.oracle.com/cd/E13222_01/wls/docs81/webapp/web_xml.html#1039287)，其中如下描述：
>WebLogic Server initializes this servlet when WebLogic Server starts up. The optional content of this element must be a positive integer indicating the order in which the servlet should be loaded. Lower integers are loaded before higher integers. If no value is specified, or if the value specified is not a positive integer, WebLogic Server can load the servlet in any order during application startup.

主要就是当负值的时候可以在应用启动的时候以任意顺序来加载每个servlet。
## 最后
这个问题其实说来说起也不算个什么大问题，并且据我所知除了在浏览器方面的标准执行的极其不理想外，其他的地方都基本按照规范来做的，JSR相对来说还是一个很靠谱的参考资料，当你有困惑的时候还是参照JSR的解释来解决问题比较好。这个问题解决起来很简单，就是你必须要有顺序，就都设置0或者大于0的数来作为顺序就可以了，暂时来说应该是没啥兼容问题的。

本来其实是个挺简单的事，但是由于期间我好奇搜过一点中文文档，的确给我造成了极大的困扰，不知道是否还有其他人被这些人给困扰过没。在出现的问题的时候还是建议使用英文关键词并且使用google，英语会的不需要太多，只需要能看懂基本文章，并且会拼凑几个单词搜就足够了，大部分程序员这方面估计都没太多问题，趁早抛弃中文文档吧。

{% include JB/setup %}
