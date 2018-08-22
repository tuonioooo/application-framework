# ApplicationContext源码讲解

## 前言

在BeanFactory里只对IOC容器的基本行为作了定义，根本不关心你的bean是如何定义怎样加载的。正如我们只关心工厂里得到什么的产品对象，至于工厂是怎么生产这些对象的，这个基本的接口不关心。

而要知道工厂是如何产生对象的，我们需要看具体的IOC容器实现，spring提供了许多IOC容器的实现。比如XmlBeanFactory，ClasspathXmlApplicationContext等。其中XmlBeanFactory就是针对最基本的ioc容器的实现，这个IOC容器可以读取XML文件定义的BeanDefinition（XML文件中对bean的描述）,如果说XmlBeanFactory是容器中的屌丝，ApplicationContext应该算容器中的高帅富.

ApplicationContext是Spring提供的一个高级的IoC容器，它除了能够提供IoC容器的基本功能外，还为用户提供了以下的附加服务。

## ApplicationContext

**结构**

![](/assets/import-applicationcontext-01.png)**接口**

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

**ApplicationContext 的子接口**

```
org.springframework.context.ConfigurableApplicationContext
org.springframework.web.context.WebApplicationContext
```



