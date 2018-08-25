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

![](/assets/import-selector-01.png)

## 参考

[https://blog.csdn.net/dm\_vincent/article/details/77619752](https://blog.csdn.net/dm_vincent/article/details/77619752)

