# SpringBoot启动过程定制化

在上一篇[SpringBoot启动过程源码分析](/springbootpian/springbootqi-dong-guo-cheng-yuan-ma-fen-xi.md)文章中，从源码角度介绍了Spring Boot的启动过程。启动的代码虽然只有短短的一行，但是背后所做的工作还真不少，其中有一些可以定制化的部分，主要分为以下几个方面：

1. 初始化器\(Initializer\)
2. 监听器\(Listener\)
3. 容器刷新后置Runners\(ApplicationRunner或者CommandLineRunner接口的实现类\)
4. 启动期间在Console打印Banner的具体实现类

本文就来看看如何定制上述几个方面。

## 初始化器\(Initializer\) {#初始化器initializer}

### 使用现状 {#使用现状}

在添加一个定制的初始化器之前，还是先来回顾一下初始化器在Spring Boot启动过程中是如何被使用的：

1. 定义：默认的初始化器全部都是定义在每个jar包下的META-INF/spring.factories配置文件中，Key为org.springframework.context.ApplicationContextInitializer
2. 添加：在实例化SpringApplication的时候，会读取定义并进行实例化，保存到SpringApplication实例的initializers成员变量中。
3. 执行：在刷新容器前置动作的applyInitializers方法中会遍历执行所有初始化器。

### 添加定制化的初始化器 {#添加定制化的初始化器}

#### 一个直观的添加方法 {#一个直观的添加方法}

可以在实例化SpringApplication，执行其run方法之前，添加定制的初始化器：

```
public static void main(String[] args) {

    // 创建SpringApplication的实例
    SpringApplication app = new SpringApplication(Application.class);

    // 添加定制的初始化器
    app.addInitializers(new CustomApplicationContextInitializer());

    // 执行SpringApplication实例的run方法
    app.run(args);
}
```

在定制的初始化器中，打印一条日志：

```
public class CustomApplicationContextInitializer implements ApplicationContextInitializer {

    private static final Logger LOGGER = LoggerFactory.getLogger(CustomApplicationContextInitializer.class);

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        LOGGER.info("自定义的初始化器的initialize方法被执行了");
    }
}
```

#### 更符合Spring Boot Style的添加方法 {#更符合spring-boot-style的添加方法}

前面已经讨论过，Spring Boot是通过读取META-INF/spring.factories配置文件中Key为org.springframework.context.ApplicationContextInitializer的内容来依次加载初始化器的，因此我们也可以创建一个配置文件用来定义需要添加的初始化器：

```
org.springframework.context.ApplicationContextInitializer=com.example.demo.CustomApplicationContextInitializer
```

执行入口类，可以看到打印完了Banner后，马上就会输出上面的日志内容。

## 监听器\(Listener\) {#监听器listener}

### 使用现状 {#使用现状-1}

同样地，先回顾一下监听器在Spring Boot启动过程中是如何被使用的：

1. 定义：默认的初始化器全部都是定义在每个jar包下的META-INF/spring.factories配置文件中，Key为org.springframework.context.ApplicationContextInitializer
2. 添加：在实例化SpringApplication的时候，会读取定义并进行实例化，保存到SpringApplication实例的listeners成员变量中。
3. 执行：在容器启动过程中会通过SpringApplicationRunListener发布事件，然后相应的监听器就会被执行。

### 自定义监听器 {#自定义监听器}

下面我们来自定义一个监听器，处理感兴趣的事件类型：

```
public class CustomApplicationListener implements ApplicationListener {

    private static final Logger LOGGER = LoggerFactory.getLogger(CustomApplicationListener.class);

    public void onApplicationEvent(ApplicationEvent event) {
        // 监听ApplicationStartingEvent
        if (event instanceof ApplicationStartedEvent) {
            logInfo("ApplicationStartedEvent listened");
        }

        // 监听ApplicationEnvironmentPreparedEvent
        else if (event instanceof ApplicationEnvironmentPreparedEvent) {
            logInfo("ApplicationEnvironmentPreparedEvent listened");
        }

        // 监听ApplicationPreparedEvent
        else if (event instanceof ApplicationPreparedEvent) {
            logInfo("ApplicationPreparedEvent listened");
        }

        // 监听ApplicationReadyEvent
        else if (event instanceof ApplicationReadyEvent) {
            logInfo("ApplicationReadyEvent listened");
        }

        // 监听ApplicationFailedEvent
        else if (event instanceof ApplicationFailedEvent) {
            logInfo("ApplicationFailedEvent listened");
        }
    }

    private void logInfo(String log) {
        if (LOGGER.isInfoEnabled()) {
            LOGGER.info(log);
        }
    }
}
```

为什么选择监听这几个事件，而不是其它的。主要是因为这几个事件都是由Spring Boot定义的：

![](/assets/import-springboot-11.png)以上监听的部分事件在容器启动过程中，会由SpringApplicationRunListener的默认实现EventPublishingRunListener完成事件的派发\(关于此部分的详情可以参考前面的一篇文章\)。

有了实现类，同样还是需要将它定义到spring.factories中：

```
# Application Listeners
org.springframework.context.ApplicationListener=\
com.example.demo.CustomApplicationListener
```

启动后，可以看到控制台中输出了相关信息\(只摘录了相关行\)：

    2017-08-07 23:40:36.705  INFO 40733 --- [           main] c.e.demo.CustomApplicationListener       : ApplicationEnvironmentPreparedEvent listened

      .   ____          _            __ _ _
     /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
    ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
     \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
      '  |____| .__|_| |_|_| |_\__, | / / / /
     =========|_|==============|___/=/_/_/_/
     :: Spring Boot ::        (v1.5.6.RELEASE)

     2017-08-07 23:40:41.858  INFO 40733 --- [           main] c.e.demo.CustomApplicationListener       : ApplicationPreparedEvent listened
    2017-08-07 23:40:43.109  INFO 40733 --- [           main] c.e.demo.CustomApplicationListener       : ApplicationReadyEvent listened

可以看到，监听的三个事件得到了响应：

1. ApplicationEnvironmentPreparedEvent
2. ApplicationPreparedEvent
3. ApplicationReadyEvent

没有得到响应的有两个事件：

1. ApplicationStartedEvent
2. ApplicationFailedEvent

我们来看看负责事件中转的EventPublishingRunListener，看能够得到一些线索：

1. ApplicationStartedEvent

```
public void starting() {
  this.initialMulticaster.multicastEvent(new ApplicationStartedEvent(this.application, this.args));
}
```

上述方法确实会派发一个ApplicationStartedEvent事件，但是没有打印出来日志。是因为在尝试打印这条日志的时候LOGGER.isInfoEnabled\(\)调用返回的是false，因此阻止了日志的输出。

这是因为日志系统初始化也是基于事件机制的，监听到ApplicationStartedEvent事件的时候日志对象还没有完成初始化，直到监听到ApplicationEnvironmentPreparedEvent后才会初始化。所以即使我们监听到了ApplicationStartedEvent事件，也没法打印相应日志。如果将LOGGER的输出改成System.out.println，就没有问题了。

1. ApplicationFailedEvent

```
private SpringApplicationEvent getFinishedEvent( ConfigurableApplicationContext context, Throwable exception) {
    if (exception != null) {
        return new ApplicationFailedEvent(this.application, this.args, context,
                exception);
    }
    return new ApplicationReadyEvent(this.application, this.args, context);
}
```

可以发现，只有当启动过程中有异常时，才会创建一个ApplicationFailedEvent事件对象。否则创建的是ApplicationReadyEvent。因此，如果想要人为地构造这条日志，可以再启动过程中抛出一个异常，比如就在自定义的初始化器中：

```
public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
  if (LOGGER.isInfoEnabled()) {
    LOGGER.info("自定义的初始化器的initialize方法被执行了");
  }

  throw new RuntimeException("人为创造的运行时异常");
}
```

然后再次启动，就可以看到这条日志被输出了，同时异常堆栈也会被打印出来。

## 容器刷新后置Runners {#容器刷新后置runners}

Runner的实现分为两种，可以选择下面的接口之一进行实现：

1. org.springframework.boot.ApplicationRunner
2. org.springframework.boot.CommandLineRunner

它们没有什么本质的区别，除了方法接受的类型不太一样。

### Runners的执行时机 {#runners的执行时机}

这里先回顾一下Runners的执行时机，在容器刷新的后置处理中会找到所有的Runners并依次运行：

```
protected void afterRefresh(ConfigurableApplicationContext context,
            ApplicationArguments args) {
  callRunners(context, args);
}
```

### ApplicationRunner自定义实现 {#applicationrunner自定义实现}

```
@Component
@Order(2)
public class CustomApplicationRunner implements ApplicationRunner {

  private static final Logger LOGGER = LoggerFactory.getLogger(CustomApplicationRunner.class);

  @Override
  public void run(ApplicationArguments arg) {
      if (LOGGER.isInfoEnabled()) {
          LOGGER.info("自定义ApplicationRunner运行了");
      }
  }

}
```

### CommandLineRunner自定义实现 {#commandlinerunner自定义实现}

```
@Component
@Order(1)
public class CustomCommandLineRunner implements CommandLineRunner {

  private static final Logger LOGGER = LoggerFactory.getLogger(CustomCommandLineRunner.class);

  @Override
  public void run(String... strings) throws Exception {
      if (LOGGER.isInfoEnabled()) {
          LOGGER.info("自定义CommandLineRunner运行了");
      }
  }

}
```

自定义Runner的添加方式和上述的初始化器以及监听器有点不同，就是Runners直接通过@Component注解添加即可，借助于包扫描机制，它们会被注册到容器中。至于它们运行的顺序，则可以通过上述的@Order注解来完成标注，数值越小的Runner优先级越高，因此上面代码中，CustomCommandLineRunner将先于CustomApplicationRunner被执行。

### 启动打印Banner {#启动打印banner}

最后，来看看如何定制Banner，默认的Banner Printer会打印出来Spring Boot的ASCII艺术字：

      .   ____          _            __ _ _
     /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
    ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
     \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
      '  |____| .__|_| |_|_| |_\__, | / / / /
     =========|_|==============|___/=/_/_/_/
     :: Spring Boot ::        (v1.5.6.RELEASE)

我们完全根据需求来打印一个更酷炫的图案，作为Gundam爱好者，如果又恰好是做一个相关项目的话，我会选择这样的图案：

![](/assets/import-springboot-12.png)扩展的方式也很简单，将需要输出的内容放到resources目录下的banner.txt文件中即可。

当然，这只是最简单的一种扩展方式，如果需要看看还支持什么扩展方式，看看源代码就清楚了：

```
private Banner printBanner(ConfigurableEnvironment environment) {
    if (this.bannerMode == Banner.Mode.OFF) {
        return null;
    }
    ResourceLoader resourceLoader = this.resourceLoader != null ? this.resourceLoader
            : new DefaultResourceLoader(getClassLoader());
    SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(
            resourceLoader, this.banner);
    if (this.bannerMode == Mode.LOG) {
        return bannerPrinter.print(environment, this.mainApplicationClass, logger);
    }
    return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
}
```

因此Banner的输出模式也有好几种：

1. OFF
2. CONSOLE\(默认\) - 此时对应的输出即为System.out
3. LOG

然后，Banner的输出方式也有Image和Text，我们现在使用的是Text的方式。至于使用Image和Text的高级用法，这里就不展开了，有兴趣的话可以去看看官方文档或者深挖一些相关代码即可。

## 总结 {#总结}

至此，Spring Boot启动过程的定制化就讨论结束了，在本文中，从四个方面介绍了如何定制Spring Boot的启动过程：

1. 初始化器\(Initializer\)
2. 监听器\(Listener\)
3. 容器刷新后置Runners\(ApplicationRunner或者CommandLineRunner接口的实现类\)
4. 启动期间在Console打印Banner的具体实现类

希望能够给大家带来一些帮助和启迪。

## 参考

https://blog.csdn.net/dm\_vincent/article/details/77151122

