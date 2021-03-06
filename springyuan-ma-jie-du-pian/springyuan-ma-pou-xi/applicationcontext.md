# ApplicationContext源码讲解

## 前言

在BeanFactory里只对IOC容器的基本行为作了定义，根本不关心你的bean是如何定义怎样加载的。正如我们只关心工厂里得到什么的产品对象，至于工厂是怎么生产这些对象的，这个基本的接口不关心。

而要知道工厂是如何产生对象的，我们需要看具体的IOC容器实现，spring提供了许多IOC容器的实现。比如XmlBeanFactory，ClasspathXmlApplicationContext等。其中XmlBeanFactory就是针对最基本的ioc容器的实现，这个IOC容器可以读取XML文件定义的BeanDefinition（XML文件中对bean的描述）,如果说XmlBeanFactory是容器中的屌丝，ApplicationContext应该算容器中的高帅富.

ApplicationContext是Spring提供的一个高级的IoC容器，它除了能够提供IoC容器的基本功能外，还为用户提供了以下的附加服务。

## ApplicationContext

* **结构**

![](/assets/import-applicationcontext-02.png)

* **接口定义**

```
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    String getId();

    String getApplicationName();

    String getDisplayName();

    long getStartupDate();

    ApplicationContext getParent();

    AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;
}
```

ApplicationContext接口继承众多接口，集众多接口功能与一身，为Spring的运行提供基本的功能支撑。

根据程序设计的“单一职责原则”，其实每个较顶层接口都是“单一职责的”，只提供某一方面的功能，而ApplicationContext接口继承了众多接口，相当于拥有了众多接口的功能，下面看看它的主要功能：

首先，它是个BeanFactory，可以管理、装配bean，可以有父级BeanFactory实现Bean的层级管理（具体到这里来说它可以有父级的ApplicationContext，因为ApplicationContext本身就是一个BeanFactory。这在web项目中很有用，可以使每个Servlet具有其独立的context, 所有Servlet共享一个父级的context），它还是Listable的，可以枚举出所管理的bean对象。

其次，它是一个ResourceLoader，可以加载资源文件；

再次，它可以管理一些Message实现国际化等功能；

还有，它可以发布事件给注册的Listener，实现监听机制。

## **ApplicationContext 的子接口**

* **ConfigurableApplicationContext **

```
public interface ConfigurableApplicationContext extends ApplicationContext, Lifecycle, Closeable {
    String CONFIG_LOCATION_DELIMITERS = ",; \t\n";
    String CONVERSION_SERVICE_BEAN_NAME = "conversionService";
    String LOAD_TIME_WEAVER_BEAN_NAME = "loadTimeWeaver";
    String ENVIRONMENT_BEAN_NAME = "environment";
    String SYSTEM_PROPERTIES_BEAN_NAME = "systemProperties";
    String SYSTEM_ENVIRONMENT_BEAN_NAME = "systemEnvironment";

    void setId(String var1);

    void setParent(ApplicationContext var1);

    void setEnvironment(ConfigurableEnvironment var1);

    ConfigurableEnvironment getEnvironment();

    void addBeanFactoryPostProcessor(BeanFactoryPostProcessor var1);

    void addApplicationListener(ApplicationListener<?> var1);

    void addProtocolResolver(ProtocolResolver var1);

    void refresh() throws BeansException, IllegalStateException;

    void registerShutdownHook();

    void close();

    boolean isActive();

    ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;
}
```

根据接口名可以判决，该接口是可配置的！ApplicationContext 接口本身是 read-only 的，所以子接ConfigurableApplicationContext 就提供了如setID\(\)、setParent\(\)、setEnvironment\(\)等方法，用来配置ApplicationContext。

接口详解参考

[https://docs.spring.io/spring/docs/5.0.8.RELEASE/javadoc-api/](https://docs.spring.io/spring/docs/5.0.8.RELEASE/javadoc-api/)

* **Lifecycle**

```
public interface Lifecycle {
    void start();

    void stop();

    boolean isRunning();
}
```

Lifecycle接口中具有start\(\)、stop\(\)等方法，用于对context生命周期的管理；

* **Closeable**

```
public interface Closeable extends AutoCloseable {

    /**
     * Closes this stream and releases any system resources associated
     * with it. If the stream is already closed then invoking this
     * method has no effect.
     *
     * <p> As noted in {@link AutoCloseable#close()}, cases where the
     * close may fail require careful attention. It is strongly advised
     * to relinquish the underlying resources and to internally
     * <em>mark</em> the {@code Closeable} as closed, prior to throwing
     * the {@code IOException}.
     *
     * @throws IOException if an I/O error occurs
     */
    public void close() throws IOException;
}
```

Closeable是标准JDK所提供的一个接口，用于最后关闭组件释放资源等；

* **WebApplicationContext**

```
public interface WebApplicationContext extends ApplicationContext {
    String ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = WebApplicationContext.class.getName() + ".ROOT";
    String SCOPE_REQUEST = "request";
    String SCOPE_SESSION = "session";
    String SCOPE_GLOBAL_SESSION = "globalSession";
    String SCOPE_APPLICATION = "application";
    String SERVLET_CONTEXT_BEAN_NAME = "servletContext";
    String CONTEXT_PARAMETERS_BEAN_NAME = "contextParameters";
    String CONTEXT_ATTRIBUTES_BEAN_NAME = "contextAttributes";

    ServletContext getServletContext();
}
```

该接口仅仅在原接口基础上提供了getServletContext\(\)，用于给servlet提供上下文信息。

* **ConfigurableWebApplicationContext**

```
public interface ConfigurableWebApplicationContext extends WebApplicationContext, ConfigurableApplicationContext {
    String APPLICATION_CONTEXT_ID_PREFIX = WebApplicationContext.class.getName() + ":";
    String SERVLET_CONFIG_BEAN_NAME = "servletConfig";

    void setServletContext(ServletContext var1);

    void setServletConfig(ServletConfig var1);

    ServletConfig getServletConfig();

    void setNamespace(String var1);

    String getNamespace();

    void setConfigLocation(String var1);

    void setConfigLocations(String... var1);

    String[] getConfigLocations();
}
```

> 这里 ConfigurableWebApplicationContext 又将上述两个接口结合起来，提供了一个可配置、可管理、可关闭的WebApplicationContext，同时该接口还增加了setServletContext\(\)，setServletConfig\(\)等set方法，用于装配WebApplicationContext。
>
> 到这里ApplicationContext相关接口基本上已经讲完了，总结起来就两大接口：
>
> org.springframework.context.ConfigurableApplicationContext
>
> org.springframework.web.context.ConfigurableWebApplicationContext
>
> 对于普通应用，使用ConfigurableApplicationContext 接口的实现类作为bean的管理者，对于web应用，使用ConfigurableWebApplicationContext 接口的实现类作为bean的管理者。
>
> 这两个接口，从结构上讲他们是继承关系，从应用上讲他们是平级关系，在不同的领域为Spring提供强大的支撑。



