# SpringBoot实现自动配置的原理

## 入口注解类@EnableAutoConfiguration {#入口注解类enableautoconfiguration}

@SpringBootApplication注解中包含了自动配置的入口注解：

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
  // ...
}
```

```
@SuppressWarnings("deprecation")
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
  // ...
}
```

这个注解的Javadoc内容还是不少，所有就不贴在文章里面了，概括一下：

1. 自动配置基于应用的类路径以及你定义了什么Beans
2. 如果使用了@SpringBootApplication注解，那么自动就启用了自动配置
3. 可以通过设置注解的excludeName属性或者通过spring.autoconfigure.exclude配置项来指定不需要自动配置的项目
4. 自动配置的发生时机在用户定义的Beans被注册之后
5. 如果没有和@SpringBootApplication一同使用，最好将@EnableAutoConfiguration注解放在root package的类上，这样就能够搜索到所有子packages中的类了
6. 自动配置类就是普通的Spring @Configuration类，通过SpringFactoriesLoader机制完成加载，实现上通常使用@Conditional\(比如@ConditionalOnClass或者@ConditionalOnMissingBean\)

### @AutoConfigurationPackage {#autoconfigurationpackage}

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {

}
```

这个注解的职责就是引入了另外一个配置类：AutoConfigurationPackages.Registrar。

```
/**
 * ImportBeanDefinitionRegistrar用来从导入的Config中保存base package
 */
@Order(Ordered.HIGHEST_PRECEDENCE)
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata,
            BeanDefinitionRegistry registry) {
        register(registry, new PackageImport(metadata).getPackageName());
    }

    @Override
    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.<Object>singleton(new PackageImport(metadata));
    }

}
```

这个注解实现的功能已经比较底层了，调试看看上面的register方法什么会被调用：

![](/assets/import-register-01.png)调用参数中的packageNames数组中仅包含一个值：com.example.demo，也就是项目的root package名。

从调用栈来看的话，调用register方法的时间在容器刷新期间：

refresh -&gt; invokeBeanFactoryPostProcessors -&gt; invokeBeanDefinitionRegistryPostProcessors -&gt; postProcessBeanDefinitionRegistry -&gt; processConfigBeanDefinitions\(开始处理配置Bean的定义\) -&gt; loadBeanDefinitions -&gt; loadBeanDefinitionsForConfigurationClass\(读取配置Class中的Bean定义\) -&gt; loadBeanDefinitionsFromRegistrars\(这里开始准备进入上面的register方法\) -&gt; registerBeanDefinitions\(即上述方法\)

这个过程已经比较复杂了，目前暂且不深入研究了。它的功能简单说就是将应用的root package给注册到Spring容器中，供后续使用。

相比而言，下面要讨论的几个类型才是实现自动配置的关键。

### @Import\(EnableAutoConfigurationImportSelector.class\) {#importenableautoconfigurationimportselectorclass}

@EnableAutoConfiguration注解的另外一个作用就是引入了EnableAutoConfigurationImportSelector：

它的类图如下所示：

![](/assets/import-selector-01.png)可以发现它除了实现几个Aware类接口外，最关键的就是实现了DeferredImportSelector\(继承自ImportSelector\)接口。

所以我们先来看看ImportSelector以及DeferredImportSelector接口的定义：

```
public interface ImportSelector {

    /**
     * 基于被引入的Configuration类的AnnotationMetadata信息选择并返回需要引入的类名列表
     */
    String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```

这个接口的Javadoc比较长，还是捡重点说明一下：

1. 主要功能通过selectImports方法实现，用于筛选需要引入的类名
2. 实现了ImportSelector的类也可以实现一系列Aware接口，这些Aware接口中的相应方法会在selectImports方法之前被调用\(这一点通过上面的类图也可以佐证，EnableAutoConfigurationImportSelector确实实现了四个Aware类型的接口\)
3. ImportSelector的实现和通常的@Import在处理方式上是一致的，然而还是可以在所有@Configuration类都被处理后再进行引入筛选\(具体看下面即将介绍的DeferredImportSelector\)

```
public interface DeferredImportSelector extends ImportSelector {

}
```

这个接口是一个标记接口，它本身没有定义任何方法。那么这个接口的含义是什么呢：

1. 它是ImportSelector接口的一个变体，在所有的@Configuration被处理之后才会执行。在需要筛选的引入类型具备@Conditional注解的时候非常有用
2. 实现类同样也可以实现Ordered接口，来定义多个DeferredImportSelector的优先级别\(同样地，EnableAutoConfigurationImportSelector也实现了Ordered接口\)

明确了这两个接口的意义，下面来看看是如何实现的：

```
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    try {
      // Step1: 得到注解信息
        AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
                .loadMetadata(this.beanClassLoader);
        // Step2: 得到注解中的所有属性信息
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
        // Step3: 得到候选配置列表
        List<String> configurations = getCandidateConfigurations(annotationMetadata,
                attributes);
        // Step4: 去重
        configurations = removeDuplicates(configurations);
        // Step5: 排序
        configurations = sort(configurations, autoConfigurationMetadata);
        // Step6: 根据注解中的exclude信息去除不需要的
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        configurations = filter(configurations, autoConfigurationMetadata);
        // Step7: 派发事件
        fireAutoConfigurationImportEvents(configurations, exclusions);
        return configurations.toArray(new String[configurations.size()]);
    }
    catch (IOException ex) {
        throw new IllegalStateException(ex);
    }
}
```

很明显，核心就在于上面的步骤3：

```
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
        AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
            getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
    Assert.notEmpty(configurations,
            "No auto configuration classes found in META-INF/spring.factories. If you "
                    + "are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

它将实现委托给了SpringFactoriesLoader的loadFactoryNames方法：

```
// 传入的factoryClass：org.springframework.boot.autoconfigure.EnableAutoConfiguration
public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName();
    try {
        Enumeration<URL> urls = (classLoader != null ? classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
                ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
        List<String> result = new ArrayList<String>();
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
            String factoryClassNames = properties.getProperty(factoryClassName);
            result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
        }
        return result;
    }
    catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() +
                "] factories from location [" + FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}

// 相关常量
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
```

这段代码的意图很明确，在第一篇文章讨论Spring Boot启动过程的时候就已经接触到了。它会从类路径中拿到所有名为META-INF/spring.factories的配置文件，然后按照factoryClass的名称取到对应的值。那么我们就来找一个META-INF/spring.factories配置文件看看。

#### META-INF/spring.factories {#meta-infspringfactories}

比如spring-boot-autoconfigure包：

```
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
# 省略了很多
```

列举了非常多的自动配置候选项，挑一个AOP相关的AopAutoConfiguration看看究竟：

```
// 如果设置了spring.aop.auto=false，那么AOP不会被配置
// 需要检测到@EnableAspectJAutoProxy注解存在才会生效
// 默认使用JdkDynamicAutoProxyConfiguration，如果设置了spring.aop.proxy-target-class=true，那么使用CglibAutoProxyConfiguration
@Configuration
@ConditionalOnClass({ EnableAspectJAutoProxy.class, Aspect.class, Advice.class })
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
public class AopAutoConfiguration {

    @Configuration
    @EnableAspectJAutoProxy(proxyTargetClass = false)
    @ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = true)
    public static class JdkDynamicAutoProxyConfiguration {

    }

    @Configuration
    @EnableAspectJAutoProxy(proxyTargetClass = true)
    @ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = false)
    public static class CglibAutoProxyConfiguration {

    }

}
```

这个自动配置类的作用是判断是否存在配置项：

```
spring.aop.proxy-target-class=true
```

如果存在并且值为true的话使用基于CGLIB字节码操作的动态代理方案，否则使用JDK自带的动态代理机制。

在这个配置类中，使用到了两个全新的注解：

* @ConditionalOnClass
* @ConditionalOnProperty

从这两个注解的名称，就大概能够猜出它们的功能了：

**@ConditionalOnClass**

当类路径上存在指定的类时，满足条件。

**@ConditionalOnProperty**

当配置中存在指定的属性时，满足条件。

其实除了这两个注解之外，还有几个类似的，它们都在org.springframework.boot.autoconfigure.condition这个包下，在具体介绍实现之前，下面先来看看Spring Boot对于@Conditional的扩展。=

## Spring Boot对于@Conditional的扩展 {#spring-boot对于conditional的扩展}

Spring Boot提供了一个实现了Condition接口的抽象类SpringBootCondition。

这个类的主要作用是打印一些用于诊断的日志，告诉用户哪些类型被自动配置了。

它实现Condition接口的方法：

```
@Override
public final boolean matches(ConditionContext context,
        AnnotatedTypeMetadata metadata) {
    String classOrMethodName = getClassOrMethodName(metadata);
    try {
        ConditionOutcome outcome = getMatchOutcome(context, metadata);
        logOutcome(classOrMethodName, outcome);
        recordEvaluation(context, classOrMethodName, outcome);
        return outcome.isMatch();
    }
    catch (NoClassDefFoundError ex) {
        throw new IllegalStateException(
                "Could not evaluate condition on " + classOrMethodName + " due to "
                        + ex.getMessage() + " not "
                        + "found. Make sure your own configuration does not rely on "
                        + "that class. This can also happen if you are "
                        + "@ComponentScanning a springframework package (e.g. if you "
                        + "put a @ComponentScan in the default package by mistake)",
                ex);
    }
    catch (RuntimeException ex) {
        throw new IllegalStateException(
                "Error processing condition on " + getName(metadata), ex);
    }
}

/**
 * Determine the outcome of the match along with suitable log output.
 * @param context the condition context
 * @param metadata the annotation metadata
 * @return the condition outcome
 */
public abstract ConditionOutcome getMatchOutcome(ConditionContext context,
        AnnotatedTypeMetadata metadata);
```

SpringBootCondition已经提供了基本的实现，将内部的匹配细节定义成抽象方法getMatchOutcome，交给其子类去完成。

另外，还提供了两个可能会被子类使用到的方法：

```
/**
 * 如果指定的conditions中有任意一个匹配，那么就返回true
 * @param context the context
 * @param metadata the annotation meta-data
 * @param conditions conditions to test
 * @return {@code true} if any condition matches.
 */
protected final boolean anyMatches(ConditionContext context,
        AnnotatedTypeMetadata metadata, Condition... conditions) {
    for (Condition condition : conditions) {
        if (matches(context, metadata, condition)) {
            return true;
        }
    }
    return false;
}

/**
 * 检查指定的condition是否匹配
 * @param context the context
 * @param metadata the annotation meta-data
 * @param condition condition to test
 * @return {@code true} if the condition matches.
 */
protected final boolean matches(ConditionContext context,
        AnnotatedTypeMetadata metadata, Condition condition) {
    if (condition instanceof SpringBootCondition) {
        return ((SpringBootCondition) condition).getMatchOutcome(context, metadata)
                .isMatch();
    }
    return condition.matches(context, metadata);
}
```

### org.springframework.boot.autoconfigure.condition包 {#orgspringframeworkbootautoconfigurecondition包}

除了上面已经遇到的@ConditionalOnClass和@ConditionalOnProperty，这个包中还定义了很多条件实现类，下面简单列举几个：

#### @ConditionalOnExpression - 基于SpEL的条件判断 {#conditionalonexpression-基于spel的条件判断}

```
/**
 * Configuration annotation for a conditional element that depends on the value of a SpEL
 * expression.
 *
 * @author Dave Syer
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE, ElementType.METHOD })
@Documented
@Conditional(OnExpressionCondition.class)
public @interface ConditionalOnExpression {

    /**
     * The SpEL expression to evaluate. Expression should return {@code true} if the
     * condition passes or {@code false} if it fails.
     * @return the SpEL expression
     */
    String value() default "true";
```

然后相应的实现类是OnExpressionCondition，它继承自SpringBootCondition。

#### @ConditionalOnMissingClass - 基于类不存在与classpath的条件判断 {#conditionalonmissingclass-基于类不存在与classpath的条件判断}

这一个条件实现正好和@ConditionalOnClass条件相反。

下面列举所有由Spring Boot提供的条件注解：

* @ConditionalOnBean
* @ConditionalOnClass
* @ConditionalOnCloudPlatform
* @ConditionalOnExpression
* @ConditionalOnJava
* @ConditionalOnJndi
* @ConditionalOnMissingBean
* @ConditionalOnMissingClass
* @ConditionalOnNotWebApplication
* @ConditionalOnProperty
* @ConditionalOnResource
* @ConditionalOnSingleCandidate
* @ConditionalOnWebApplication

一般的模式，就是一个条件注解对应一个继承自SpringBootCondition的具体实现类。

## 参考

[https://blog.csdn.net/dm\_vincent/article/details/77619752](https://blog.csdn.net/dm_vincent/article/details/77619752)

