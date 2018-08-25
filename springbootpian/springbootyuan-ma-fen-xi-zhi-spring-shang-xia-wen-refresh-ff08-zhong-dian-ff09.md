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

        @Override

	protected void prepareRefresh\(\) {

		this.scanner.clearCache\(\);

		super.prepareRefresh\(\);

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

