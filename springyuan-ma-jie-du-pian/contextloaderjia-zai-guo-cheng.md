# ContextLoader加载过程

ContextLoaderListener实现了ServletContextListener接口，ServletContextListener是Java EE标准接口之一，类似tomcat，jetty的java容器启动时便会触发该接口的contextInitialized。

1java容器启动触发ContextLoaderListener的contextInitialized

2 contextInitialized 方法调用ContextLoader的initWebApplicationContext方法。

3 initWebApplicationContext调用createWebApplicationContext方法

4 createWebApplicationContext 调用determineContextClass方法

5 determineContextClass有如下代码

```
contextClassName = defaultStrategies
                .getProperty(WebApplicationContext.class.getName());
```

显然是从defaultStrategies中加载的ContextLoader 类中有段静态代码

```
static {
        try {
            ClassPathResource resource = new ClassPathResource(
                    "ContextLoader.properties", ContextLoader.class);
            defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
        } catch (IOException ex) {
            throw new IllegalStateException(
                    "Could not load 'ContextLoader.properties': "
                            + ex.getMessage());
        }

        currentContextPerThread = new ConcurrentHashMap(1);
    }
```

ContextLoader.properties 文件内容如下：

```
org.springframework.web.context.WebApplicationContext=org.springframework.web.context.support.XmlWebApplicationContext
```

至此，determineContextClass方法返回的是XmlWebApplicationContext

6 回到 initWebApplicationContext 方法，调用configureAndRefreshWebApplicationContext方法

7 configureAndRefreshWebApplicationContext 调用了AbstractApplicationContext的refresh方法

8 refresh 方法调用了obtainFreshBeanFactory

9 obtainFreshBeanFactory 调用了AbstractRefreshableApplicationContext类的refreshBeanFactory方法

10 refreshBeanFactory调用了XmlWebApplicationContext的loadBeanDefinitions

11 loadBeanDefinitions中加载了对应的applicationContext.xml

## ContextLoaderListener和DispatcherServlet的相互关系

tomcat在加载的时候会先加载listner，然后再加载servlet。

ContextLoaderListener加载的时候会实例化加载了比如DAO、service等Bean的spring context；

DispatcherContext加载的时候会以ContextLoaderListener加载的spring context容器作为parent context容器，

这个spring context里边主要定义的bean一般是和spring mvc相关的controller、页面跳转等；

