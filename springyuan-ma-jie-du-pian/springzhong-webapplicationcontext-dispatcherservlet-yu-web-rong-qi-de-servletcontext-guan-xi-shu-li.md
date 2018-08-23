# Spring中WebApplicationContext、DispatcherServlet与web容器的ServletContext关系梳理

学习源码过程中，对各种context（上下文）表示很懵逼。特地留此一篇。

1.要了解各个上下文之间的关系。首先走一遍spring在web容器\(tomcat\)中的启动过程

 a\) ServletContext:  tomcat启动会创建一个ServletContext，作为全局上下文以及spring容器的宿主环境。当执行Servlet的init\(\)方法时，会触发ServletContextListener的 public void contextInitialized\(ServletContextEvent sce\);方法

![](/assets/import-web-01.png)

