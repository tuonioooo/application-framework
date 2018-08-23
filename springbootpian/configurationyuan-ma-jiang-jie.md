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

所以Spring会回调这两个方法

1. postProcessBeanDefinitionRegistry\(由BeanDefinitionRegistryPostProcessor接口定义\)

2. postProcessBeanFactory\(由BeanFactoryPostProcessor接口定义\)

3. BeanDefinitionRegistryPostProcessor接口又继承自BeanFactoryPostProcessor; 依据注释, 前者所定义方法的被执行时机位于后者所定义方法之前.

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

通过观察上面必将被Spring框架回调的两个方法, 我们可以推测:

1. 这两个方法必将被会调用, 且都只会被调用一次.

2. 在执行顺序是 processConfigBeanDefinitions在先, 而enhanceConfigurationClasses在后.

3. **processConfigBeanDefinitions方法**

```
/**
 * Build and validate a configuration model based on the registry of
 * {@link Configuration} classes.
 */
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    Set<BeanDefinitionHolder> configCandidates = new LinkedHashSet<BeanDefinitionHolder>();
    // 迭代现有容器中所有bean
    for (String beanName : registry.getBeanDefinitionNames()) {
        BeanDefinition beanDef = registry.getBeanDefinition(beanName);
        // 注意下面这个方法会向BeanDefinition类型的beanDef注入一个名ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE为的Attribute
        // 返回true的条件是该beanDef代表的类被@Configuration或@Component或@Bean修饰
        // @Configuration注解上有@Component所修饰, @Bean上可没有
        if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
            configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
        }
    }

    // Return immediately if no @Configuration classes were found
    // 如果没有被任何被上述三注解所修饰的, 则中断接下来的处理, 直接返回.
    if (configCandidates.isEmpty()) {
        return;
    }

    // Detect any custom bean name generation strategy supplied through the enclosing application context
    // 上面的官方注释已经很清楚了
    SingletonBeanRegistry singletonRegistry = null;
    if (registry instanceof SingletonBeanRegistry) {
        singletonRegistry = (SingletonBeanRegistry) registry;
        if (!this.localBeanNameGeneratorSet && singletonRegistry.containsSingleton(CONFIGURATION_BEAN_NAME_GENERATOR)) {
            BeanNameGenerator generator = (BeanNameGenerator) singletonRegistry.getSingleton(CONFIGURATION_BEAN_NAME_GENERATOR);
            this.componentScanBeanNameGenerator = generator;
            this.importBeanNameGenerator = generator;
        }
    }

    // Parse each @Configuration class
    // 构建一个专门处理@Configuration的类, 注意该类的访问级别也是 package 
    ConfigurationClassParser parser = new ConfigurationClassParser(
            this.metadataReaderFactory, this.problemReporter, this.environment,
            this.resourceLoader, this.componentScanBeanNameGenerator, registry);
    for (BeanDefinitionHolder holder : configCandidates) {
        BeanDefinition bd = holder.getBeanDefinition();
        try {
            // 解析每个BeanDefinition
            if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
                parser.parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
            }
            else {

                parser.parse(bd.getBeanClassName(), holder.getBeanName());
            }
        }
        catch (IOException ex) {
            throw new BeanDefinitionStoreException("Failed to load bean class: " + bd.getBeanClassName(), ex);
        }
    }
    // 验证解析的成果
    parser.validate();

    // --------------------------------------------------------------
    // 1. 在经过上面的parser.validate();之后, 代表基本解析工作完毕, 并且通过验证
    // 2. 接下来要做的是提取解析的成果, 进行相应的处理, 例如将解析出来的BeanMethod(代表一个被@Bean标注的方法)集合注册进容器
    // --------------------------------------------------------------

    // Handle any @PropertySource annotations
    // 处理上面的parse解析出来的@PropertySource信息
    // @PropertySource注解的主要功能是引入配置文件，将配置的属性键值对与环境变量中的配置合并
    // @PropertySources仅仅只是包含多个@PropertySource
    Stack<PropertySource<?>> parsedPropertySources = parser.getPropertySources();
    if (!parsedPropertySources.isEmpty()) {
        if (!(this.environment instanceof ConfigurableEnvironment)) {
            logger.warn("Ignoring @PropertySource annotations. " +
                    "Reason: Environment must implement ConfigurableEnvironment");
        }
        else {
            MutablePropertySources envPropertySources = ((ConfigurableEnvironment)this.environment).getPropertySources();
            while (!parsedPropertySources.isEmpty()) {
                envPropertySources.addLast(parsedPropertySources.pop());
            }
        }
    }

    // Read the model and create bean definitions based on its content
    // 创建一个用于加载Bean的reader
    if (this.reader == null) {
        this.reader = new ConfigurationClassBeanDefinitionReader(
                registry, this.sourceExtractor, this.problemReporter, this.metadataReaderFactory,
                this.resourceLoader, this.environment, this.importBeanNameGenerator);
    }
    // 1. 处理解析出来的被@Configuration注解的类信息(被@Configuration修饰的类相关的元数据被使用ConfigurationClass来承载)
    // 2. 例如包括解析ConfigurationClass中的BeanMethod实例集合(每个BeanBeanMethod代表的是被@Bean修饰的方法,也就是一个Bean实例)
    this.reader.loadBeanDefinitions(parser.getConfigurationClasses());

    // Register the ImportRegistry as a bean in order to support ImportAware @Configuration classes
    // 不再赘述
    if (singletonRegistry != null) {
        if (!singletonRegistry.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
            singletonRegistry.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
        }
    }

    if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
        ((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
    }
}
```

* **ConfigurationClassParser类**

* Parse each @Configuration class — 官方注释

* 访问级别级别为 package;
* 位于org.springframework.context.annotation package下

```
// Parse the specified {@link Configuration @Configuration} class.
public void parse(String className, String beanName) throws IOException {
    MetadataReader reader = this.metadataReaderFactory.getMetadataReader(className);
    // 注意这里使用ConfigurationClass进行了包装
    // 即在Spring中每个ConfigurationClass代表一个被@Configuration修饰的配置类
    processConfigurationClass(new ConfigurationClass(reader, beanName));
}

protected void processConfigurationClass(ConfigurationClass configClass) throws IOException {
    // 略过开始部分的代码

    // Recursively process the configuration class and its superclass hierarchy.
    do {
        metadata = doProcessConfigurationClass(configClass, metadata);
    }
    while (metadata != null);

    this.configurationClasses.add(configClass);
}

/**
 * @return annotation metadata of superclass, {@code null} if none found or previously processed
 */
protected AnnotationMetadata doProcessConfigurationClass(ConfigurationClass configClass, AnnotationMetadata metadata) throws IOException {
    //----------- 虽然看着比较多, 但还是比较清晰; 空格分开的段落之间, 彼此各自处理各自的内容

    // recursively process any member (nested) classes first
    processMemberClasses(metadata);

    // process any @PropertySource annotations
    AnnotationAttributes propertySource = MetadataUtils.attributesFor(metadata,
            org.springframework.context.annotation.PropertySource.class);
    if (propertySource != null) {
        processPropertySource(propertySource);
    }

    // process any @ComponentScan annotations
    // @ComponentScan定义自动扫描的包, 其最关键的方法为doScan方法，会注册BeanDefinition到容器中。
    AnnotationAttributes componentScan = MetadataUtils.attributesFor(metadata, ComponentScan.class);
    if (componentScan != null) {
        // the config class is annotated with @ComponentScan -> perform the scan immediately
        // 这里的this.componentScanParser, 其类型为ComponentScanAnnotationParser(访问级别也是package)
        // 也就是说ComponentScanAnnotationParser负责了@ComponentScan的解析工作
        Set<BeanDefinitionHolder> scannedBeanDefinitions =
                this.componentScanParser.parse(componentScan, metadata.getClassName());

        // check the set of scanned definitions for any further config classes and parse recursively if necessary
        for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
            if (ConfigurationClassUtils.checkConfigurationClassCandidate(holder.getBeanDefinition(), this.metadataReaderFactory)) {
                this.parse(holder.getBeanDefinition().getBeanClassName(), holder.getBeanName());
            }
        }
    }

    // process any @Import annotations
    // @Import注解可以配置需要引入的class
    Set<Object> imports = new LinkedHashSet<Object>();
    Set<Object> visited = new LinkedHashSet<Object>();
    collectImports(metadata, imports, visited);
    if (!imports.isEmpty()) {
        processImport(configClass, metadata, imports, true);
    }

    // process any @ImportResource annotations
    // @ImportResource的主要功能为引入资源文件。
    if (metadata.isAnnotated(ImportResource.class.getName())) {
        AnnotationAttributes importResource = MetadataUtils.attributesFor(metadata, ImportResource.class);
        String[] resources = importResource.getStringArray("value");
        Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
        for (String resource : resources) {
            String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
            configClass.addImportedResource(resolvedResource, readerClass);
        }
    }

    // process individual @Bean methods
    // 这里就是解析每个@Bean元信息的
    // 一个BeanMethod代表一个被@Bean修饰的方法,也就是一个Bean实例
    Set<MethodMetadata> beanMethods = metadata.getAnnotatedMethods(Bean.class.getName());
    for (MethodMetadata methodMetadata : beanMethods) {
        // 使用BeanMethod封装解析出来的结果
        configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
    }

    // process superclass, if any
    if (metadata.hasSuperClass()) {
        String superclass = metadata.getSuperClassName();
        if (!superclass.startsWith("java") && !this.knownSuperclasses.containsKey(superclass)) {
            this.knownSuperclasses.put(superclass, configClass);
            // superclass found, return its annotation metadata and recurse
            if (metadata instanceof StandardAnnotationMetadata) {
                Class<?> clazz = ((StandardAnnotationMetadata) metadata).getIntrospectedClass();
                return new StandardAnnotationMetadata(clazz.getSuperclass(), true);
            }
            else {
                MetadataReader reader = this.metadataReaderFactory.getMetadataReader(superclass);
                return reader.getAnnotationMetadata();
            }
        }
    }

    // no superclass, processing is complete
    return null;
}
```

* **ConfigurationClassBeanDefinitionReader类**

这个类的访问级别也是 package.

```
/**
 * Read a particular {@link ConfigurationClass}, registering bean definitions for the
 * class itself, all its {@link Bean} methods
 */
private void loadBeanDefinitionsForConfigurationClass(ConfigurationClass configClass) {
    if (configClass.isImported()) {
        // Register the {@link Configuration} class itself as a bean definition.
        registerBeanDefinitionForImportedConfigurationClass(configClass);
    }
    for (BeanMethod beanMethod : configClass.getBeanMethods()) {
        // 解读每个BeanMethod里的元数据, 将解析得到的bean注册进容器
        loadBeanDefinitionsForBeanMethod(beanMethod);
    }
    loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
}
```

* **enhanceConfigurationClasses方法**

> 唯一想要提及的是 如果我们从容器中取出配置类\(例如通过springConfig作为key取出上面的定义的SpringConfig\)的话, 就会发现, Spring返回给我们的并不是一个原生的SpringConfig实例; 而是类似SpringConfig$$EnhancerBySpringCGLIB$$47a5de53@11935e这样的实例; 这明显就是被CGLIB代理过的.
>
> 而原因就发生在 ConfigurationClassPostProcessor类的enhanceConfigurationClasses方法中.
>
> 其它暂时略.

## Links

1. [http://blog.csdn.net/honghailiang888/article/details/74981445](http://blog.csdn.net/honghailiang888/article/details/74981445)
2. [http://www.mamicode.com/info-detail-1564393.html](http://www.mamicode.com/info-detail-1564393.html)
3. [http://blog.csdn.net/isea533/article/details/78072133](http://blog.csdn.net/isea533/article/details/78072133)
4. [https://www.cnblogs.com/jiaoqq/p/7678037.html](https://www.cnblogs.com/jiaoqq/p/7678037.html)



