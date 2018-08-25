# SpringBoot源码分析之Spring上下文refresh（重点）

* **refreshContext\(\)方法**

```
private void refreshContext(ConfigurableApplicationContext context) {
        refresh(context);
        if (this.registerShutdownHook) {
            try {
                context.registerShutdownHook();
            }
            catch (AccessControlException ex) {
                // Not allowed in some environments.
            }
        }
    }
```

* **refresh\(\)方法**

```
protected void refresh(ApplicationContext applicationContext) {
        Assert.isInstanceOf(AbstractApplicationContext.class, applicationContext);
        ((AbstractApplicationContext) applicationContext).refresh();
    }
```

调用的是AbstractApplicationContext中的refresh方法：

```
@Override
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {//refresh过程只能一个线程处理，不允许并发执行
      // 刷新前准备工作
      prepareRefresh();
      // 调用子类refreshBeanFactory()方法，获取bean factory
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
      // 创建bean Factory的通用设置，添加ApplicationContextAwareProcessor,
      // ResourceLoader、ApplicationEventPublisher、ApplicationContext这3个接口对应的bean都设置为当前的Spring容器,注册环境bean
      prepareBeanFactory(beanFactory);
      try {
         // 子类特殊的bean factory设置
         // GenericWebApplicationContext容器会在BeanFactory中添加ServletContextAwareProcessor用于处理ServletContextAware类型的bean初始化的时候调用setServletContext或者setServletConfig方法
         postProcessBeanFactory(beanFactory);
         // 实例化beanFactoryPostProcessor
         // 调用beanFactoryPostProcessor 这里会调用ConfigurationClassPostProcessor，解析@Configuration的类为BeanDefinition，为后面实例化作准备
         invokeBeanFactoryPostProcessors(beanFactory);
         // 注册 beanPostProcessors 包括自定义的BeanPostProcessor
         // 在实例化Bean后处理 比如AutowiredAnnotationBeanPostProcessor(处理被@Autowired注解修饰的bean并注入)、RequiredAnnotationBeanPostProcessor(处理被@Required注解修饰的方法)
         // 这些都是在创建Context时的reader的构造器中的AnnotationConfigUtils的registerAnnotationConfigProcessors方法中注册的
         registerBeanPostProcessors(beanFactory);
         // 初始化信息源，和国际化相关
         initMessageSource();
         // 初始化容器事件传播器
         initApplicationEventMulticaster();
         // 调用子类特殊的刷新逻辑
         // web程序的容器AnnotationConfigEmbeddedWebApplicationContext中会调用createEmbeddedServletContainer方法去创建内置的Servlet容器。
         onRefresh();
         // 为事件传播器注册事件监听器
         registerListeners();
         // 实例化所有非懒加载单例
         finishBeanFactoryInitialization(beanFactory);
         // 初始化容器的生命周期事件处理器，并发布容器的生命周期事件
         finishRefresh();
      }
      catch (BeansException ex) {
         // ...
      }
      finally {
         // ...
      }
   }
}
```

* **prepareRefresh\(\)方法**

  ```
    @Override
  ```

  protected void prepareRefresh\(\) {

  ```
    this.scanner.clearCache\(\);

    super.prepareRefresh\(\);
  ```

  }

```
protected void prepareRefresh() {
        // 表示在真正做refresh操作之前需要准备做的事情：

        // 设置Spring容器的启动时间，撤销关闭状态，开启活跃状态。
        this.startupDate = System.currentTimeMillis();
        this.closed.set(false);
        this.active.set(true);

        if (logger.isInfoEnabled()) {
            logger.info("Refreshing " + this);
        }

        // Initialize any placeholder property sources in the context environment
        // 初始化属性源信息,初始化占位符
        initPropertySources();

        // Validate that all properties marked as required are resolvable
        // see ConfigurablePropertyResolver#setRequiredProperties
        // 验证环境信息里一些必须存在的属性
        getEnvironment().validateRequiredProperties();

        // Allow for the collection of early ApplicationEvents,
        // to be published once the multicaster is available...
        this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

* **obtainFreshBeanFactory\(\)方法**

```
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
        // 获取刷新Spring上下文的Bean工厂
        refreshBeanFactory();
        ConfigurableListableBeanFactory beanFactory = getBeanFactory();
        if (logger.isDebugEnabled()) {
            logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
        }
        return beanFactory;
}
```

此时BeanFactory中注册的BeanDefinition和Bean实例

![](/assets/import-refresh-01.png)

![](/assets/import-refresh-02.png)

* **prepareBeanFactory\(\)方法**

对BeanFactory\(Spring Bean容器\)进行相关的设置为后续的使用做准备：

```
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// Tell the internal bean factory to use the context's class loader etc.
		// 设置classloader(用于加载bean)，设置表达式解析器(解析bean定义中的一些表达式)，添加属性编辑注册器(注册属性编辑器)
		beanFactory.setBeanClassLoader(getClassLoader());
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));
 
		// Configure the bean factory with context callbacks.
		// 添加ApplicationContextAwareProcessor这个BeanPostProcessor。取消ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware、ApplicationContextAware、EnvironmentAware这5个接口的自动注入。
		// 因为ApplicationContextAwareProcessor把这6个接口的实现工作做了
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
 
		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		// 设置特殊的类型对应的bean。BeanFactory对应刚刚获取的BeanFactory；
		// ResourceLoader、ApplicationEventPublisher、ApplicationContext这3个接口对应的bean都设置为当前的Spring容器
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);
 
		// Register early post-processor for detecting inner beans as ApplicationListeners.
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
 
		// Detect a LoadTimeWeaver and prepare for weaving, if found.
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}
 
		// Register default environment beans.
		// 注入一些其它信息的bean实例，比如environment、systemProperties等
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
}

```

* **postProcessBeanFactory\(\)方法**

BeanFactory设置之后再进行后续的一些BeanFactory操作。

不同的Spring容器做不同的操作。比如GenericWebApplicationContext容器会在BeanFactory中添加ServletContextAwareProcessor用于处理ServletContextAware类型的bean初始化的时候调用setServletContext或者setServletConfig方法\(跟ApplicationContextAwareProcessor原理一样\)。

AnnotationConfigEmbeddedWebApplicationContext对应的postProcessBeanFactory方法：

```
@Override
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
  // 调用父类EmbeddedWebApplicationContext的实现
  super.postProcessBeanFactory(beanFactory);
  // 查看basePackages属性，如果设置了会使用ClassPathBeanDefinitionScanner去扫描basePackages包下的bean并注册
  if (this.basePackages != null && this.basePackages.length > 0) {
    this.scanner.scan(this.basePackages);
  }
  // 查看annotatedClasses属性，如果设置了会使用AnnotatedBeanDefinitionReader去注册这些bean
  if (this.annotatedClasses != null && this.annotatedClasses.length > 0) {
    this.reader.register(this.annotatedClasses);
  }
}

```

父类EmbeddedWebApplicationContext的实现：

```
@Override
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
  beanFactory.addBeanPostProcessor(
      new WebApplicationContextServletContextAwareProcessor(this));//添加ServletContext注入处理器
  beanFactory.ignoreDependencyInterface(ServletContextAware.class);
}

```

* **invokeBeanFactoryPostProcessors\(\)方法**

```
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
		PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
 
		// Detect a LoadTimeWeaver and prepare for weaving, if found in the meantime
		// (e.g. through an @Bean method registered by ConfigurationClassPostProcessor)
		if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}
}

```

getBeanFactoryPostProcessors\(\)方法返回的数据：//不使用dubbo时没有dubbo

由图可知这些BeanFactoryPostProcessor

**实例**

都是在哪添加进context的beanFactoryPostProcessors属性中

