# ApplicationContext源码讲解

## 前言

在BeanFactory里只对IOC容器的基本行为作了定义，根本不关心你的bean是如何定义怎样加载的。正如我们只关心工厂里得到什么的产品对象，至于工厂是怎么生产这些对象的，这个基本的接口不关心。

而要知道工厂是如何产生对象的，我们需要看具体的IOC容器实现，spring提供了许多IOC容器的实现。比如XmlBeanFactory，ClasspathXmlApplicationContext等。其中XmlBeanFactory就是针对最基本的ioc容器的实现，这个IOC容器可以读取XML文件定义的BeanDefinition（XML文件中对bean的描述）,如果说XmlBeanFactory是容器中的屌丝，ApplicationContext应该算容器中的高帅富.

ApplicationContext是Spring提供的一个高级的IoC容器，它除了能够提供IoC容器的基本功能外，还为用户提供了以下的附加服务。

## ApplicationContext

* **结构**

![](/assets/import-applicationcontext-01.png)

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

根据接口名可以判决，该接口是可配置的！ApplicationContext 接口本身是 read-only 的，所以子接口 ConfigurableApplicationContext 就提供了如setID\(\)、setParent\(\)、setEnvironment\(\)等方法，用来配置ApplicationContext。

再看其继承的另外两个接口：



Lifecycle：Lifecycle接口中具有start\(\)、stop\(\)等方法，用于对context生命周期的管理；

Closeable：Closeable是标准JDK所提供的一个接口，用于最后关闭组件释放资源等；

public

interface

WebApplicationContext

extends

ApplicationContext

该接口仅仅在原接口基础上提供了getServletContext\(\)，用于给servlet提供上下文信息。

public

interface

ConfigurableWebApplicationContext

extends

WebApplicationContext, ConfigurableApplicationContext

这里 ConfigurableWebApplicationContext 又将上述两个接口结合起来，提供了一个可配置、可管理、可关闭的WebApplicationContext，同时该接口还增加了setServletContext\(\)，setServletConfig\(\)等set方法，用于装配WebApplicationContext。

到这里ApplicationContext相关接口基本上已经讲完了，总结起来就两大接口：

org.springframework.context.ConfigurableApplicationContext

org.springframework.web.context.ConfigurableWebApplicationContext

对于普通应用，使用ConfigurableApplicationContext 接口的实现类作为bean的管理者，对于web应用，使用ConfigurableWebApplicationContext 接口的实现类作为bean的管理者。

这两个接口，从结构上讲他们是继承关系，从应用上讲他们是平级关系

，在不同的领域为Spring提供强大的支撑。  

## ApplicationContext相关实现类设计：

Spring是一个优秀的框架，具有良好的结构设计和接口抽象，它的每一个接口都是其功能具体到各个模块中的高度抽象，实际使用过程中相当于把接口的各个实现类按照接口所提供的组织架构装配起来以提供完整的服务，可以说掌握了Spring的接口就相当于掌握了Spring的大部分功能。

ApplicationContext 的实现类众多，然而

上文中分析了 ApplicationContext 接口的各个功能，下面将分析 ApplicationContext 的实现类对上述接口的各个功能都是怎样实现的（PS. 限于篇幅，这里仅仅指出上述各个功能在实现类中什么位置通过什么方法实现，至于其具体实现过程，每一个功能拿出来都可以单独写一篇文章了，这里不进行详述）。至于实现类又扩展了其他接口或者继承了其他父类，这些只是实现类为了扩展功能或者为了对实现上述接口提供便利而做的事情，对ApplicationContext接口抽象出来的功能没有影响或者没有太大帮助，因此略去。

以ClassPathXmlApplicationContext为例，其主要继承关系如下：

org.springframework.context.support.AbstractApplicationContext

org.springframework.context.support.AbstractRefreshableApplicationContext

org.springframework.context.support.AbstractRefreshableConfigApplicationContext

org.springframework.context.support.AbstractXmlApplicationContext

org.springframework.context.support.ClassPathXmlApplicationContext

而最顶层抽象类 AbstractApplicationContext 又实现了 ConfigurableApplicationContext 接口。

AbstractApplicationContext

extends

DefaultResourceLoader

implements

ConfigurableApplicationContext, DisposableBean

根据上文所述，这里略去其父类 DefaultResourceLoader 和接口 DisposableBean ，只关注接口 ConfigurableApplicationContext，回忆一下该的主要功能：

