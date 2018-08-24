# SpringBoot启动类源码分析以及@EnableAutoConfiguration和@SpringBootApplication讲解

原文：[https://blog.csdn.net/tuoni123/article/details/79985950](https://blog.csdn.net/tuoni123/article/details/79985950)

对于任何一个Spring boot项目，都会用到下面的启动类：

```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

从上面代码可以看出，@SpringBootApplication和类SpringApplication.run是我们分析的主要方法

**@SpringBooApplication**源码如下：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
...
}
```

虽然定义使用了多个Annotation进行了原信息标注，但实际上重要的只有三个Annotation：

@Configuration（@SpringBootConfiguration点开查看发现里面还是应用了@Configuration）

@EnableAutoConfiguration

@ComponentScan

所以，如果我们使用如下的SpringBoot启动类，整个SpringBoot应用依然可以与之前的启动类功能对等：

```
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

因此上面三个注解对等于@SpringBootConfiguration，所以直接写@SpringBootConfiguration方便点，下面分别介绍一下这三个Annotation

**@Configuration**

可以参考[@Configuration注解、@Bean注解以及配置自动扫描、bean作用域](https://blog.csdn.net/tuoni123/article/details/79977459)

**@ComponentScan**

@ComponentScan这个注解在Spring中很重要，它对应XML配置中的 元素，@ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。

我们可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。

注：所以SpringBoot的启动类最好是放在root package下，因为默认不指定basePackages

**@EnableAutoConfiguration**

个人感觉@EnableAutoConfiguration这个Annotation最为重要，所以放在最后来解读，大家是否还记得Spring框架提供的各种名字为@Enable开头的Annotation定义？比如@EnableScheduling、@EnableCaching、@EnableMBeanExport等，@EnableAutoConfiguration的理念和做事方式其实一脉相承，简单概括一下就是， 借助@Import的支持，收集和注册特定场景相关的bean定义 。

@EnableScheduling是通过@Import将Spring调度框架相关的bean定义都加载到IoC容器。@EnableMBeanExport是通过@Import将JMX相关的bean定义加载到IoC容器。而@EnableAutoConfiguration也是借助@Import的帮助，将所有符合自动配置条件的bean定义加载到IoC容器，仅此而已！

@EnableAutoConfiguration作为一个复合Annotation,其自身定义关键信息如下

```
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@Import({ EnableAutoConfigurationImportSelector.class,  
        AutoConfigurationPackages.Registrar.class })  
public @interface EnableAutoConfiguration {  

    /**  
     * Exclude specific auto-configuration classes such that they will never be applied.  
     * @return the classes to exclude  
     */  
    Class<?>[] exclude() default {};  

}
```

其中，最关键的要

@Import\(EnableAutoConfigurationImportSelector.class\)，借助EnableAutoConfigurationImportSelector

@EnableAutoConfiguration可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器。就像一只“八爪鱼”一样，借助于Spring框架原有的一个工具类：SpringFactoriesLoader的支持，@EnableAutoConfiguration可以智能的自动配置功效才得以大功告成！

![](/assets/import-springboot-01.png)

自动配置幕后英雄：SpringFactoriesLoader详解

SpringFactoriesLoader属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件META-INF/spring.factories加载配置。

```
public abstract class SpringFactoriesLoader {
    //...
    public static <T> List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader) {
        ...
    }


    public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
        ....
    }
}
```

配合@EnableAutoConfiguration使用的话，它更多是提供一种配置查找的功能支持，即根据@EnableAutoConfiguration的完整类名org.springframework.boot.autoconfigure.EnableAutoConfiguration作为查找的Key,获取对应的一组@Configuration类

![](/assets/import-springboot-02.png)

上图就是从SpringBoot的autoconfigure依赖包中的META-INF/spring.factories配置文件中摘录的一段内容，可以很好地说明问题。所以，@EnableAutoConfiguration自动配置的魔法骑士就变成了： 从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。

EnableAutoConfigurationImportSelector 是一个DeferredImportSelector，由 spring boot autoconfigure 从版本1.3开始,提供用来处理EnableAutoConfiguration自动配置。

EnableAutoConfigurationImportSelector继承自AutoConfigurationImportSelector,从 pring boot 1.5 以后，EnableAutoConfigurationImportSelector已经不再被建议使用，而是推荐使用 AutoConfigurationImportSelector。

[Spring EnableAutoConfigurationImportSelector 是如何工作的 ?](https://blog.csdn.net/andy_zhang2007/article/details/78580980)

[Spring 工具类 ConfigurationClassParser 是如何工作的 ？](https://blog.csdn.net/andy_zhang2007/article/details/78549773)

**深入探索SpringApplication执行流程**

  


SpringApplication的run方法的实现是我们本次旅程的主要线路，该方法的主要流程大体可以归纳如下：

  


  


1） 如果我们使用的是SpringApplication的静态run方法，那么，这个方法里面首先要创建一个SpringApplication对象实例，然后调用这个创建好的SpringApplication的实例方法。在SpringApplication实例初始化的时候，它会提前做几件事情：

  


根据classpath里面是否存在某个特征类（org.springframework.web.context.ConfigurableWebApplicationContext）来决定是否应该创建一个为Web应用使用的ApplicationContext类型。

  


使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationContextInitializer。

  


使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationListener。

  


推断并设置main方法的定义类。

  


2） SpringApplication实例初始化完成并且完成设置后，就开始执行run方法的逻辑了，方法执行伊始，首先遍历执行所有通过SpringFactoriesLoader可以查找到并加载的SpringApplicationRunListener。调用它们的started\(\)方法，告诉这些SpringApplicationRunListener，“嘿，SpringBoot应用要开始执行咯！”。

  


3） 创建并配置当前Spring Boot应用将要使用的Environment（包括配置要使用的PropertySource以及Profile）。

  


4） 遍历调用所有SpringApplicationRunListener的environmentPrepared\(\)的方法，告诉他们：“当前SpringBoot应用使用的Environment准备好了咯！”。

  


5） 如果SpringApplication的showBanner属性被设置为true，则打印banner。

  


6） 根据用户是否明确设置了applicationContextClass类型以及初始化阶段的推断结果，决定该为当前SpringBoot应用创建什么类型的ApplicationContext并创建完成，然后根据条件决定是否添加ShutdownHook，决定是否使用自定义的BeanNameGenerator，决定是否使用自定义的ResourceLoader，当然，最重要的，将之前准备好的Environment设置给创建好的ApplicationContext使用。

  


7） ApplicationContext创建好之后，SpringApplication会再次借助Spring-FactoriesLoader，查找并加载classpath中所有可用的ApplicationContext-Initializer，然后遍历调用这些ApplicationContextInitializer的initialize（applicationContext）方法来对已经创建好的ApplicationContext进行进一步的处理。

  


8） 遍历调用所有SpringApplicationRunListener的contextPrepared\(\)方法。

  


9） 最核心的一步，将之前通过@EnableAutoConfiguration获取的所有配置以及其他形式的IoC容器配置加载到已经准备完毕的ApplicationContext。

  


10） 遍历调用所有SpringApplicationRunListener的contextLoaded\(\)方法。

  


11） 调用ApplicationContext的refresh\(\)方法，完成IoC容器可用的最后一道工序。

  


12） 查找当前ApplicationContext中是否注册有CommandLineRunner，如果有，则遍历执行它们。

  


13） 正常情况下，遍历执行SpringApplicationRunListener的finished\(\)方法、（如果整个过程出现异常，则依然调用所有SpringApplicationRunListener的finished\(\)方法，只不过这种情况下会将异常信息一并传入处理）

  


去除事件通知点后，整个流程如下：

