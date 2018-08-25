# SpringBoot中ApplicationContextInitializer的源码分析

在SpringApplication的实例属性中有一个初始器的属性：List&lt;ApplicationContextInitializer&lt;?&gt;&gt; initializers ，这些初始化器\(initializers\)是Spring Boot通过读取每个jar包下的/META-INF/spring.factories文件中的配置获取的。每一个initailizer都是一个实现了ApplicationContextInitializer接口的实例。ApplicationContextInitializer是Spring IOC容器中提供的一个接口：

```
package org.springframework.context;


public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {

    /**
     * Initialize the given application context.
     * @param applicationContext the application to configure
     */
    void initialize(C applicationContext);

}
```

ApplicationContextInitializer是一个回调接口，它会在ConfigurableApplicationContext的refresh\(\)方法调用之前被调用,做一些容器的初始化工作。这一点我们也可以通过SpringApplication的实例run方法的实现代码得到验证，为了说明问题，再次贴一下这段代码,注意下标红的代码和注释就自然理解了。

```
public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        FailureAnalyzers analyzers = null;
        configureHeadlessProperty();
        SpringApplicationRunListeners listeners = getRunListeners(args);
        listeners.starting();
        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(
                    args);
            ConfigurableEnvironment environment = prepareEnvironment(listeners,
                    applicationArguments);
            Banner printedBanner = printBanner(environment);
            context = createApplicationContext();
            analyzers = new FailureAnalyzers(context);
            prepareContext(context, environment, listeners, applicationArguments,
                    printedBanner); // prepareContext方法中将会执行每个initializers的逻辑
            refreshContext(context);  // 执行bean的创建和实例化
            afterRefresh(context, applicationArguments);
            listeners.finished(context, null);
            stopWatch.stop();
            if (this.logStartupInfo) {
                new StartupInfoLogger(this.mainApplicationClass)
                        .logStarted(getApplicationLog(), stopWatch);
            }
            return context;
        }
        catch (Throwable ex) {
            handleRunFailure(context, listeners, analyzers, ex);
            throw new IllegalStateException(ex);
        }
    }
```



Spring Boot对initializer的获取过程如下：

```
setInitializers((Collection) getSpringFactoriesInstances(
            ApplicationContextInitializer.class));
```

这里出现了一个新的概念 - 初始化器。先来看看代码，再来尝试解释一下它是干嘛的：

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



SpringBoot默认情况下提供了6个initializer，分别由2个jar提供：

spring-boot-1.5.2.RELEASE.jar

* 　　org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,
* 　　org.springframework.boot.context.ContextIdApplicationContextInitializer,
* 　　org.springframework.boot.context.config.DelegatingApplicationContextInitializer,
* 　　org.springframework.boot.context.embedded.ServerPortInfoApplicationContextInitializer

spring-boot-autoconfigure-1.5.2.RELEASE.jar

* 　　org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,
* 　　org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer





