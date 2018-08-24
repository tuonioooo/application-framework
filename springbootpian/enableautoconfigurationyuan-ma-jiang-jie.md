# SpringBoot启动类源码分析以及@EnableAutoConfiguration和@SpringBootApplication讲解

原文：https://blog.csdn.net/tuoni123/article/details/79985950

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

  


