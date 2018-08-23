# @Configuration源码讲解

## 示例

```
// ------------------------ 配置文件
@Configuration
public class SpringConfig {
    @Bean
    public Piano piano() {
        return new Piano();
    }

    @Bean(name = "counter")
    public Counter counter() {
        return new Counter(12, "Shake it Off", piano());
    }
}

// ------------------------ 测试

ApplicationContext annotationContext = new AnnotationConfigApplicationContext("xxxPackagePath");
Counter c = annotationContext.getBean("counter", Counter.class);
Assert.assertEquals("糊涂",c.getName()); //测试
```

## AnnotationConfigApplicationContext

以上的代码非常简单, 一眼就能看出关键点就是这个AnnotationConfigApplicationContext类了.

而下面的AnnotationConfigApplicationContext构造函数中, 相信注释已经很清楚了, 这里我们关注的重点当然就是scan\(basePackages\);方法.至于其它两个方法:

1. this\(\); 主要是为了实例化自身的两个重要字段.

2. refresh\(\); 这个方法至关重要, 不过不是本文的重点. 所以直接略过.如果读者有兴趣, 可以参见本人下面给出的链接, 或者查阅《Spring源码深度解读》—— 作者在其书中对此方法进行了详尽的解读, 个人觉得还是很值得买一本的

```
public AnnotationConfigApplicationContext(String... basePackages) {
    // 调用自身的构造函数
    this();
    // 扫描指定package下的bean
    scan(basePackages);
    // 调用基类AbstractApplicationContext中的refresh()方法.
    refresh();
}
```

* **AnnotationConfigApplicationContext.scan 方法**

```
public void scan(String... basePackages) {
    Assert.notEmpty(basePackages, "At least one base package must be specified");
    // 这个scanner就是上文所说的由this()实例化的字段, 类型为 ClassPathBeanDefinitionScanner
    this.scanner.scan(basePackages);
}
```

* **ClassPathBeanDefinitionScanner.scan 方法**

```
public int scan(String... basePackages) {
    // 统计扫描之前, 容器中的bean数量
    int beanCountAtScanStart = this.registry.getBeanDefinitionCount();

    // 关于这个方法, 请看本人的另外一遍博客 http://blog.csdn.net/lqzkcx3/article/details/78150513 中的第五小节及之后的内容.
    doScan(basePackages);

    // Register annotation config processors, if necessary.
    // 所以这个才是本次关注的重心所在
    // 默认情况下, this.includeAnnotationConfig 为true.所以默认情况下是一定会调用
    if (this.includeAnnotationConfig) {
        AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
    }

    // 返回本次扫描后新增的bean数量
    return this.registry.getBeanDefinitionCount() - beanCountAtScanStart;
}
```

* **AnnotationConfigUtils.registerAnnotationConfigProcessors 方法**

```
/**
 * Register all relevant annotation post processors in the given registry.
 */
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
        BeanDefinitionRegistry registry, Object source) {

    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
    if (beanFactory != null) {
        if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
            beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
        }

        if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
            beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
        }
    }
    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<BeanDefinitionHolder>(4);

    // 增加@Configuration相关的处理器ConfigurationClassPostProcessor
    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    //增加@Autowired、@Value、@Inject相关的处理器AutowiredAnnotationBeanPostProcessor
    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    //增加@Required相关的处理器RequiredAnnotationBeanPostProcessor
    if (!registry.containsBeanDefinition(REQUIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(RequiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, REQUIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    //在支持JSR-250条件下注册javax.annotation包下注解处理器，包括@PostConstruct、@PreDestroy、@Resource注解等
    // Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
    if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    //支持jpa的条件下，注册org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor处理器，处理jpa相关注解
    // Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
    if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition();
        try {
            def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
                    AnnotationConfigUtils.class.getClassLoader()));
        }
        catch (ClassNotFoundException ex) {
            throw new IllegalStateException(
                    "Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
        }
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    // 以下这两个判断是我专门去4.3.11版本里复制过来的

    // 增加@EventListener相关处理器
    if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
    }

    // 注册支持@EventListener注解的处理器, 也就是DefaultEventListenerFactory实例了
    if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
    }

    return beanDefs;
}
```

* **ConfigurationClassPostProcessor类**

既然此次我们关注的重点是@Configuration, 目光当然就落在ConfigurationClassPostProcessor类上了, 观察其继承链, 鼎鼎大名的BeanFactoryPostProcessor赫然在目. 这个类的关键性就不在这里赘述了.

ConfigurationClassPostProcessor 继承链

![](/assets/QQ截图20180823144431.jpg)

```
// --------------------------- BeanDefinitionRegistryPostProcessor接口的实现

/**
 * Derive further bean definitions from the configuration classes in the registry.
 */
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    // 向容器中注册一个 ImportAwareBeanPostProcessor 实例; 用于处理 ImportAware 接口标签
    RootBeanDefinition iabpp = new RootBeanDefinition(ImportAwareBeanPostProcessor.class);
    iabpp.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    registry.registerBeanDefinition(IMPORT_AWARE_PROCESSOR_BEAN_NAME, iabpp);

    // this.registriesPostProcessed 和 this.factoriesPostProcessed : 这两个类级字段用于确保本方法只被调用一次, 
    // 在结合下面的 postProcessBeanFactory 方法, 这两个字段的另外一个作用是确保 processConfigBeanDefinitions 方法只执行一次.
    int registryId = System.identityHashCode(registry);
    if (this.registriesPostProcessed.contains(registryId)) {
        throw new IllegalStateException(
                "postProcessBeanDefinitionRegistry already called for this post-processor against " + registry);
    }
    if (this.factoriesPostProcessed.contains(registryId)) {
        throw new IllegalStateException(
                "postProcessBeanFactory already called for this post-processor against " + registry);
    }
    this.registriesPostProcessed.add(registryId);

    processConfigBeanDefinitions(registry);
}

// --------------------------- BeanFactoryPostProcessor接口的实现

/**
 * Prepare the Configuration classes for servicing bean requests at runtime
 * by replacing them with CGLIB-enhanced subclasses.
 */
public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    int factoryId = System.identityHashCode(beanFactory);
    if (this.factoriesPostProcessed.contains(factoryId)) {
        throw new IllegalStateException(
                "postProcessBeanFactory already called for this post-processor against " + beanFactory);
    }
    this.factoriesPostProcessed.add(factoryId);
    if (!this.registriesPostProcessed.contains(factoryId)) {
        // BeanDefinitionRegistryPostProcessor hook apparently not supported...
        // Simply call processConfigurationClasses lazily at this point then.
        processConfigBeanDefinitions((BeanDefinitionRegistry) beanFactory);
    }
    enhanceConfigurationClasses(beanFactory);
}
```



