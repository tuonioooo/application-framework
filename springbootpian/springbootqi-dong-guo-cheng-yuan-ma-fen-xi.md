# SpringBoot启动过程源码分析

关于Spring Boot，已经有很多介绍其如何使用的文章了，本文从源代码\(基于Spring-boot 1.5.6\)的角度来看看Spring Boot的启动过程到底是怎么样的，为何以往纷繁复杂的配置到如今可以这么简便。

## 1. 入口类 {#1-入口类}

```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

以上的代码就是通过Spring Initializr配置生成的一个最简单的Web项目\(只引入了Web功能\)的入口方法。这个想必只要是接触过Spring Boot都会很熟悉。简单的方法背后掩藏的是Spring Boot在启动过程中的复杂性，本文的目的就是一探这里面的究竟。

### 1.1 注解@SpringBootApplication {#11-注解springbootapplication}

而在看这个方法的实现之前，需要看看@SpringBootApplication这个注解的功能：

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
  // ...
}
```

很明显的，这个注解就是三个常用在一起的注解@SpringBootConfiguration，@EnableAutoConfiguration以及@ComponentScan的组合，并没有什么高深的地方。

### 1.1.1 @SpringBootConfiguration {#111-springbootconfiguration}

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}
```

这个注解实际上和@Configuration有相同的作用，配备了该注解的类就能够以JavaConfig的方式完成一些配置，可以不再使用XML配置。

### 1.1.2 @ComponentScan {#112-componentscan}

顾名思义，这个注解完成的是自动扫描的功能，相当于Spring XML配置文件中的：

```
<context:component-scan>
```

可以使用basePackages属性指定要扫描的包，以及扫描的条件。如果不设置的话默认扫描@ComponentScan注解所在类的同级类和同级目录下的所有类，所以对于一个Spring Boot项目，一般会把入口类放在顶层目录中，这样就能够保证源码目录下的所有类都能够被扫描到。

### 1.1.3 @EnableAutoConfiguration {#113-enableautoconfiguration}

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

这个注解是让Spring Boot的配置能够如此简化的关键性注解。目前知道这个注解的作用就可以了，关于自动配置不再本文讨论范围内，后面如果有机会另起文章专门分析这个自动配置的实现原理。

## 2. 入口方法 {#2-入口方法}

### 2.1 SpringApplication的实例化 {#21-springapplication的实例化}

介绍完了入口类，下面开始分析关键方法：

```
SpringApplication.run(DemoApplication.class, args);
```

相应实现：

```
// 参数对应的就是DemoApplication.class以及main方法中的args
public static ConfigurableApplicationContext run(Class<?> primarySource,
        String... args) {
    return run(new Class<?>[] { primarySource }, args);
}

// 最终运行的这个重载方法
public static ConfigurableApplicationContext run(Class<?>[] primarySources,
        String[] args) {
    return new SpringApplication(primarySources).run(args);
}
```

它实际上会构造一个SpringApplication的实例，然后运行它的run方法：

```
// 构造实例
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    this.webApplicationType = deduceWebApplicationType();
    setInitializers((Collection) getSpringFactoriesInstances(
            ApplicationContextInitializer.class));
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

在构造函数中，主要做了4件事情：

#### 2.1.1 推断应用类型是Standard还是Web {#211-推断应用类型是standard还是web}

```
private WebApplicationType deduceWebApplicationType() {
    if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
            && !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)) {
        return WebApplicationType.REACTIVE;
    }
    for (String className : WEB_ENVIRONMENT_CLASSES) {
        if (!ClassUtils.isPresent(className, null)) {
            return WebApplicationType.NONE;
        }
    }
    return WebApplicationType.SERVLET;
}
// 相关常量
private static final String REACTIVE_WEB_ENVIRONMENT_CLASS = "org.springframework."
        + "web.reactive.DispatcherHandler";
private static final String MVC_WEB_ENVIRONMENT_CLASS = "org.springframework."
        + "web.servlet.DispatcherServlet";
private static final String[] WEB_ENVIRONMENT_CLASSES = { "javax.servlet.Servlet",
        "org.springframework.web.context.ConfigurableWebApplicationContext" };
```

可能会出现三种结果：

1. WebApplicationType.REACTIVE - 当类路径中存在REACTIVE\_WEB\_ENVIRONMENT\_CLASS并且不存在MVC\_WEB\_ENVIRONMENT\_CLASS时
2. WebApplicationType.NONE - 也就是非Web型应用\(Standard型\)，此时类路径中不包含WEB\_ENVIRONMENT\_CLASSES中定义的任何一个类时
3. WebApplicationType.SERVLET - 类路径中包含了WEB\_ENVIRONMENT\_CLASSES中定义的所有类型时

#### 2.1.2 设置初始化器\(Initializer\) {#212-设置初始化器initializer}

```
setInitializers((Collection) getSpringFactoriesInstances(
            ApplicationContextInitializer.class));
```

这里出现了一个新的概念 - 初始化器。

先来看看代码，再来尝试解释一下它是干嘛的：

```
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    return getSpringFactoriesInstances(type, new Class<?>[] {});
}

// 这里的入参type就是ApplicationContextInitializer.class
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
        Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
    // 使用Set保存names来避免重复元素
    Set<String> names = new LinkedHashSet<>(
            SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    // 根据names来进行实例化
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
            classLoader, args, names);
    // 对实例进行排序
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```

这里面首先会根据入参type读取所有的names\(是一个String集合\)，然后根据这个集合来完成对应的实例化操作：

```
// 入参就是ApplicationContextInitializer.class
public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
  String factoryClassName = factoryClass.getName();

  try {
      Enumeration<URL> urls = classLoader != null?classLoader.getResources("META-INF/spring.factories"):ClassLoader.getSystemResources("META-INF/spring.factories");
      ArrayList result = new ArrayList();

      while(urls.hasMoreElements()) {
          URL url = (URL)urls.nextElement();
          Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
          String factoryClassNames = properties.getProperty(factoryClassName);
          result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
      }

      return result;
  } catch (IOException var8) {
      throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() + "] factories from location [" + "META-INF/spring.factories" + "]", var8);
  }
}
```

这个方法会尝试从类路径的META-INF/spring.factories处读取相应配置文件，然后进行遍历，读取配置文件中Key为：org.springframework.context.ApplicationContextInitializer的value。以spring-boot-autoconfigure这个包为例，它的META-INF/spring.factories部分定义如下所示：

```
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer
```

因此这两个类名会被读取出来，然后放入到集合中，准备开始下面的实例化操作：

```
// 关键参数：
// type: org.springframework.context.ApplicationContextInitializer.class
// names: 上一步得到的names集合
private <T> List<T> createSpringFactoriesInstances(Class<T> type,
        Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args,
        Set<String> names) {
    List<T> instances = new ArrayList<T>(names.size());
    for (String name : names) {
        try {
            Class<?> instanceClass = ClassUtils.forName(name, classLoader);
            Assert.isAssignable(type, instanceClass);
            Constructor<?> constructor = instanceClass
                    .getDeclaredConstructor(parameterTypes);
            T instance = (T) BeanUtils.instantiateClass(constructor, args);
            instances.add(instance);
        }
        catch (Throwable ex) {
            throw new IllegalArgumentException(
                    "Cannot instantiate " + type + " : " + name, ex);
        }
    }
    return instances;
}
```

初始化步骤很直观，没什么好说的，类加载，确认被加载的类确实是org.springframework.context.ApplicationContextInitializer的子类，然后就是得到构造器进行初始化，最后放入到实例列表中。

因此，所谓的初始化器就是org.springframework.context.ApplicationContextInitializer的实现类，这个接口是这样定义的：

```
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {

    /**
     * Initialize the given application context.
     * @param applicationContext the application to configure
     */
    void initialize(C applicationContext);

}
```

根据类文档，这个接口的主要功能是：

在Spring上下文被刷新之前进行初始化的操作。典型地比如在Web应用中，注册Property Sources或者是激活Profiles。Property Sources比较好理解，就是配置文件。Profiles是Spring为了在不同环境下\(如DEV，TEST，PRODUCTION等\)，加载不同的配置项而抽象出来的一个实体。

#### 2.1.3. 设置监听器\(Listener\) {#213-设置监听器listener}

设置完了初始化器，下面开始设置监听器：

```
setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
```

同样地，监听器也是一个新概念，还是从代码入手：

```
// 这里的入参type是：org.springframework.context.ApplicationListener.class
private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type) {
    return getSpringFactoriesInstances(type, new Class<?>[] {});
}

private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type,
        Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
    // Use names and ensure unique to protect against duplicates
    Set<String> names = new LinkedHashSet<String>(
            SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
            classLoader, args, names);
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```

可以发现，这个加载相应的类名，然后完成实例化的过程和上面在设置初始化器时如出一辙，同样，还是以spring-boot-autoconfigure这个包中的spring.factories为例，看看相应的Key-Value：

```
# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer
```

至于ApplicationListener接口，它是Spring框架中一个相当基础的接口了，代码如下：

```
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {

    /**
     * Handle an application event.
     * @param event the event to respond to
     */
    void onApplicationEvent(E event);

}
```

这个接口基于JDK中的EventListener接口，实现了观察者模式。对于Spring框架的观察者模式实现，它限定感兴趣的事件类型需要是ApplicationEvent类型的子类，而这个类同样是继承自JDK中的EventObject类。

#### 2.1.4. 推断应用入口类 {#214-推断应用入口类}

```
this.mainApplicationClass = deduceMainApplicationClass();
```

这个方法的实现有点意思：

```
private Class<?> deduceMainApplicationClass() {
    try {
        StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
        for (StackTraceElement stackTraceElement : stackTrace) {
            if ("main".equals(stackTraceElement.getMethodName())) {
                return Class.forName(stackTraceElement.getClassName());
            }
        }
    }
    catch (ClassNotFoundException ex) {
        // Swallow and continue
    }
    return null;
}
```

它通过构造一个运行时异常，通过异常栈中方法名为main的栈帧来得到入口类的名字。

至此，对于SpringApplication实例的初始化过程就结束了。

### 2.2 SpringApplication.run方法 {#22-springapplicationrun方法}

完成了实例化，下面开始调用run方法：

```
// 运行run方法
public ConfigurableApplicationContext run(String... args) {
  // 计时工具
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();

    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();

    // 设置java.awt.headless系统属性为true - 没有图形化界面
    configureHeadlessProperty();

    // KEY 1 - 获取SpringApplicationRunListeners
    SpringApplicationRunListeners listeners = getRunListeners(args);

    // 发出开始执行的事件
    listeners.starting();
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
                args);

        // KEY 2 - 根据SpringApplicationRunListeners以及参数来准备环境
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
                applicationArguments);
        configureIgnoreBeanInfo(environment);

        // 准备Banner打印器 - 就是启动Spring Boot的时候打印在console上的ASCII艺术字体
        Banner printedBanner = printBanner(environment);

        // KEY 3 - 创建Spring上下文
        context = createApplicationContext();

        // 准备异常报告器
        exceptionReporters = getSpringFactoriesInstances(
                SpringBootExceptionReporter.class,
                new Class[] { ConfigurableApplicationContext.class }, context);

        // KEY 4 - Spring上下文前置处理
        prepareContext(context, environment, listeners, applicationArguments,
                printedBanner);

        // KEY 5 - Spring上下文刷新
        refreshContext(context);

        // KEY 6 - Spring上下文后置处理
        afterRefresh(context, applicationArguments);

        // 发出结束执行的事件
        listeners.finished(context, null);

        // 停止计时器
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                    .logStarted(getApplicationLog(), stopWatch);
        }
        return context;
    }
    catch (Throwable ex) {
        handleRunFailure(context, listeners, exceptionReporters, ex);
        throw new IllegalStateException(ex);
    }
}
```

这个run方法包含的内容也是有点多的，根据上面列举出的关键步骤逐个进行分析：

#### 2.2.1 第一步 - 获取所谓的run listeners： {#221-第一步-获取所谓的run-listeners}

```
private SpringApplicationRunListeners getRunListeners(String[] args) {
    Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
    return new SpringApplicationRunListeners(logger, getSpringFactoriesInstances(
            SpringApplicationRunListener.class, types, this, args));
}
```

这里仍然利用了getSpringFactoriesInstances方法来获取实例：

```
// 这里的入参：
// type: SpringApplicationRunListener.class
// parameterTypes: new Class<?>[] { SpringApplication.class, String[].class };
// args: SpringApplication实例本身 + main方法传入的args
private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type,
        Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
    // Use names and ensure unique to protect against duplicates
    Set<String> names = new LinkedHashSet<String>(
            SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
            classLoader, args, names);
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```

所以这里还是故技重施，从META-INF/spring.factories中读取Key为org.springframework.boot.SpringApplicationRunListener的Values：

比如在spring-boot包中的定义的spring.factories:

```
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```

我们来看看这个EventPublishingRunListener是干嘛的：

```
/**
 * {@link SpringApplicationRunListener} to publish {@link SpringApplicationEvent}s.
 * <p>
 * Uses an internal {@link ApplicationEventMulticaster} for the events that are fired
 * before the context is actually refreshed.
 *
 * @author Phillip Webb
 * @author Stephane Nicoll
 */
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {
  // ...
}
```

从类文档可以看出，它主要是负责发布SpringApplicationEvent事件的，它会利用一个内部的ApplicationEventMulticaster在上下文实际被刷新之前对事件进行处理。至于具体的应用场景，后面用到的时候再来分析。

#### 2.2.2 第二步 - 根据SpringApplicationRunListeners以及参数来准备环境 {#222-第二步-根据springapplicationrunlisteners以及参数来准备环境}

```
private ConfigurableEnvironment prepareEnvironment(
        SpringApplicationRunListeners listeners,
        ApplicationArguments applicationArguments) {
    // Create and configure the environment
    ConfigurableEnvironment environment = getOrCreateEnvironment();
    configureEnvironment(environment, applicationArguments.getSourceArgs());
    listeners.environmentPrepared(environment);
    if (!this.webEnvironment) {
        environment = new EnvironmentConverter(getClassLoader())
                .convertToStandardEnvironmentIfNecessary(environment);
    }
    return environment;
}
```

配置环境的方法：

```
protected void configureEnvironment(ConfigurableEnvironment environment,
        String[] args) {
    configurePropertySources(environment, args);
    configureProfiles(environment, args);
}
```

所以这里实际上也包含了两个步骤：

1. 配置Property Sources
2. 配置Profiles

具体实现这里就不展开了，代码也比较直观。

对于Web应用而言，得到的environment变量是一个StandardServletEnvironment的实例。得到实例后，会调用前面RunListeners中的environmentPrepared方法：

```
@Override
public void environmentPrepared(ConfigurableEnvironment environment) {
    this.initialMulticaster.multicastEvent(new ApplicationEnvironmentPreparedEvent(
            this.application, this.args, environment));
}
```

在这里，定义的广播器就派上用场了，它会发布一个ApplicationEnvironmentPreparedEvent事件。

那么有发布就有监听，在构建SpringApplication实例的时候不是初始化过一些ApplicationListeners嘛，其中的Listener就可能会监听ApplicationEnvironmentPreparedEvent事件，然后进行相应处理。

所以这里SpringApplicationRunListeners的用途和目的也比较明显了，它实际上是一个事件中转器，它能够感知到Spring Boot启动过程中产生的事件，然后有选择性的将事件进行中转。为何是有选择性的，看看它的实现就知道了：

```
@Override
public void contextPrepared(ConfigurableApplicationContext context) {

}
```

它的contextPrepared方法实现为空，没有利用内部的initialMulticaster进行事件的派发。因此即便是外部有ApplicationListener对这个事件有兴趣，也是没有办法监听到的。

那么既然有事件的转发，是谁在监听这些事件呢，在这个类的构造器中交待了：

```
public EventPublishingRunListener(SpringApplication application, String[] args) {
    this.application = application;
    this.args = args;
    this.initialMulticaster = new SimpleApplicationEventMulticaster();
    for (ApplicationListener<?> listener : application.getListeners()) {
        this.initialMulticaster.addApplicationListener(listener);
    }
}
```

前面在构建SpringApplication实例过程中设置的监听器在这里被逐个添加到了initialMulticaster对应的ApplicationListener列表中。所以当initialMulticaster调用multicastEvent方法时，这些Listeners中定义的相应方法就会被触发了。

#### 2.2.3 第三步 - 创建Spring上下文 {#223-第三步-创建spring上下文}

```
protected ConfigurableApplicationContext createApplicationContext() {
    Class<?> contextClass = this.applicationContextClass;
    if (contextClass == null) {
        try {
            contextClass = Class.forName(this.webEnvironment
                    ? DEFAULT_WEB_CONTEXT_CLASS : DEFAULT_CONTEXT_CLASS);
        }
        catch (ClassNotFoundException ex) {
            throw new IllegalStateException(
                    "Unable create a default ApplicationContext, "
                            + "please specify an ApplicationContextClass",
                    ex);
        }
    }
    return (ConfigurableApplicationContext) BeanUtils.instantiate(contextClass);
}

// WEB应用的上下文类型
public static final String DEFAULT_WEB_CONTEXT_CLASS = "org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext";
```

这个上下文类型的类图如下所示：

![](/assets/import-springbootapplication-01.png)这也是相当复杂的一个类图了，如果能把这张图中的各个类型的作用弄清楚，估计也是一个Spring大神了 :\)

对于我们的Web应用，上下文类型就是DEFAULT\_WEB\_CONTEXT\_CLASS。

#### 2.2.4 第四步 - Spring上下文前置处理 {#224-第四步-spring上下文前置处理}

```
private void prepareContext(ConfigurableApplicationContext context,
        ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
        ApplicationArguments applicationArguments, Banner printedBanner) {
    // 将环境和上下文关联起来
    context.setEnvironment(environment);

    // 为上下文配置Bean生成器以及资源加载器(如果它们非空)
    postProcessApplicationContext(context);

    // 调用初始化器
    applyInitializers(context);

    // 触发Spring Boot启动过程的contextPrepared事件
    listeners.contextPrepared(context);
    if (this.logStartupInfo) {
        logStartupInfo(context.getParent() == null);
        logStartupProfileInfo(context);
    }

    // 添加两个Spring Boot中的特殊单例Beans - springApplicationArguments以及springBootBanner
    context.getBeanFactory().registerSingleton("springApplicationArguments",
            applicationArguments);
    if (printedBanner != null) {
        context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
    }

    // 加载sources - 对于DemoApplication而言，这里的sources集合只包含了它一个class对象
    Set<Object> sources = getSources();
    Assert.notEmpty(sources, "Sources must not be empty");

    // 加载动作 - 构造BeanDefinitionLoader并完成Bean定义的加载
    load(context, sources.toArray(new Object[sources.size()]));

    // 触发Spring Boot启动过程的contextLoaded事件
    listeners.contextLoaded(context);
}
```



