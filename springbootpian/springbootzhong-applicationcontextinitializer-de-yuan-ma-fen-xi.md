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

SpringBoot默认情况下提供了6个initializer，分别由2个jar提供：

　spring-boot-1.5.2.RELEASE.jar

* 　　org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,
* 　　org.springframework.boot.context.ContextIdApplicationContextInitializer,
* 　　org.springframework.boot.context.config.DelegatingApplicationContextInitializer,
* 　　org.springframework.boot.context.embedded.ServerPortInfoApplicationContextInitializer



   spring-boot-autoconfigure-1.5.2.RELEASE.jar

* 　　org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,
* 　　org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer



 Spring Boot对initializer的获取过程如下：

initialize\(Object\[\] sources\)  
　　--&gt;\(Collection\) getSpringFactoriesInstances\( ApplicationContextInitializer.class \)\)  //获取initializer实例  
　　　　　　--&gt;SpringFactoriesLoader.loadFactoryNames\(type, classLoader\)\)  
　　　　　　--&gt;createSpringFactoriesInstances\(type, parameterTypes,classLoader, args, names\)  
　　　　　　　　--&gt;Constructor&lt;?&gt; constructor = instanceClass.getDeclaredConstructor\(parameterTypes\)  
　　　　　　　　--&gt;T instance = \(T\) BeanUtils.instantiateClass\(constructor, args\)  
　　　　　　--&gt;AnnotationAwareOrderComparator.sort\(instances\)

       --&gt; setInitializers\(Collection&lt;? extends ApplicationContextInitializer&lt;?&gt;&gt; initializers\)

　　　--&gt;this.initializers = new ArrayList&lt;ApplicationContextInitializer&lt;?&gt;&gt;\(\);

_　　　--&gt;this.initializers.addAll\(initializers\);   //存入List&lt;ApplicationContextInitializer&lt;?&gt;&gt; initializers_

