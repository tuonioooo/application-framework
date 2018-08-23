# Spring中WebApplicationContext、DispatcherServlet与web容器的ServletContext关系梳理

学习源码过程中，对各种context（上下文）表示很懵逼。特地留此一篇。

1.要了解各个上下文之间的关系。首先走一遍spring在web容器\(tomcat\)中的启动过程

* **ServletContext:  tomcat启动会创建一个ServletContext，作为全局上下文以及spring容器的宿主环境。当执行Servlet的init\(\)方法时，会触发ServletContextListener的 public void contextInitialized\(ServletContextEvent sce\);方法**

![](/assets/import-web-01.png)

* **WebApplicationContext:  在web.xml\(上图\)中我们配置了ContextLoaderListener，该listener实现了ServletContextListener的contextInitialized方法用来监听Servlet初始化事件。下图中红框部门的注释解释了该方法的作用。即初始化根上下文（即IOC容器），也就是WebApplicationContext。该类是一个接口类，其默认实现为XmlWebApplicationContext。**

![](/assets/import-web-02.png)在initWebApplicationContext这个方法中进行了创建根上下文，并将该上下文以key-value的方式存储到ServletContext中

![](/assets/import-web-03.png)以WebApplicationContext.ROOT\_WEB\_APPLICATION\_CONTEXT\_ATTRIBUTE为key，this.context则为value。this.context就是刚才创建的根上下文。后面就可以通过这个ServletContext通过这个key获取该上下文了。而在web.xml中还有一对重要的标签

&lt;context-param&gt;该标签内的&lt;param-name&gt;的值是固定的原因在这张图上。该常量的值就是contextConfigLocation。通过该方法去寻找定义spring的xml文件。来初始化IOC容器的相关信息。

![](/assets/import-web-04.png)

* **DispatcherServlet的上下文:  在WebApplicationContext初始化完后。开始初始化web.xml中的servlet。这个servlet可以有多个。默认我们都使用DispatcherServlet。&lt;servlet&gt;标签中可以有&lt;init-param&gt;标签用来配置一些DispatcherServlet的初始化参数。可以参考**[**核心流程剖析及原理分析**](/springyuan-ma-jie-du-pian/springmvcyuan-ma-jie-du/he-xin-liu-cheng-pou-xi.md)

该servlet初始化流程是有tomcat的Servlet的init\(\)方法触发。DispatcherServleet-继承-&gt;FrameworkServlet-继承-&gt;HttpServletBean-继承-GenericServlet- 实现 -&gt;Servlet。这样的一条关系链。其核心方法在FrameworkServlet中的initServletBean\(\)中的initWebApplicationContext\(\)方法中。

![](/assets/import-web-05.png)

![](/assets/import-web-06.png)

