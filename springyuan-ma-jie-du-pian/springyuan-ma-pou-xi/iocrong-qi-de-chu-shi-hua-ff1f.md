# IoC容器的初始化？

IoC容器的初始化包括BeanDefinition的Resource定位、载入和注册这三个基本的过程。我们以ApplicationContext为例讲解，ApplicationContext系列容器也许是我们最熟悉的，因为web项目中使用的XmlWebApplicationContext就属于这个继承体系，还有ClasspathXmlApplicationContext等，其继承体系如下图所示：

![](/assets/import-ioc-01.png)

**ioc容器的创建过程**

* XmlBeanFactory\(屌丝IOC\)的整个流程

```
/** @deprecated */
@Deprecated
public class XmlBeanFactory extends DefaultListableBeanFactory {
    private final XmlBeanDefinitionReader reader;

    public XmlBeanFactory(Resource resource) throws BeansException {
        this(resource, (BeanFactory)null);
    }

    public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
        super(parentBeanFactory);
        this.reader = new XmlBeanDefinitionReader(this);
        this.reader.loadBeanDefinitions(resource);
    }
}
```

示例

```
//根据Xml配置文件创建Resource资源对象，该对象中包含了BeanDefinition的信息
 ClassPathResource resource =new ClassPathResource("application-context.xml");
//创建DefaultListableBeanFactory
 DefaultListableBeanFactory factory =new DefaultListableBeanFactory();
//创建XmlBeanDefinitionReader读取器，用于载入BeanDefinition。之所以需要BeanFactory作为参数，是因为会将读取的信息回调配置给factory
 XmlBeanDefinitionReader reader =new XmlBeanDefinitionReader(factory);
//XmlBeanDefinitionReader执行载入BeanDefinition的方法，最后会完成Bean的载入和注册。完成后Bean就成功的放置到IOC容器当中，以后我们就可以从中取得Bean来使用
 reader.loadBeanDefinitions(resource);
```

> 注意：XmlBeanFactory在Spring4.0以后的版本已经，不推荐使用了

* FileSystemXmlApplicationContext 的IOC容器流程

```
public class FileSystemXmlApplicationContext extends AbstractXmlApplicationContext {
    public FileSystemXmlApplicationContext() {
    }

    public FileSystemXmlApplicationContext(ApplicationContext parent) {
        super(parent);
    }

    public FileSystemXmlApplicationContext(String configLocation) throws BeansException {
        this(new String[]{configLocation}, true, (ApplicationContext)null);
    }

    public FileSystemXmlApplicationContext(String... configLocations) throws BeansException {
        this(configLocations, true, (ApplicationContext)null);
    }

    public FileSystemXmlApplicationContext(String[] configLocations, ApplicationContext parent) throws BeansException {
        this(configLocations, true, parent);
    }

    public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh) throws BeansException {
        this(configLocations, refresh, (ApplicationContext)null);
    }

    public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {
        super(parent);
        this.setConfigLocations(configLocations);
        if (refresh) {
            this.refresh();
        }

    }

    protected Resource getResourceByPath(String path) {
        if (path != null && path.startsWith("/")) {
            path = path.substring(1);
        }

        return new FileSystemResource(path);
    }
}
```

示例

```
// 用文件系统的路径,默认指项目的根路径
ApplicationContext factory = new FileSystemXmlApplicationContext("src/appcontext.xml");
ApplicationContext factory = new FileSystemXmlApplicationContext("webRoot/WEB-INF/appcontext.xml");
// 使用了classpath:前缀,这样,FileSystemXmlApplicationContext也能够读取classpath下的相对路径
ApplicationContext factory = new FileSystemXmlApplicationContext("classpath:appcontext.xml");
ApplicationContext factory = new FileSystemXmlApplicationContext("file:F:/workspace/example/src/appcontext.xml");
```

初始化流程：

1.调用构造函数

```
public FileSystemXmlApplicationContext(String... configLocations) throws BeansException {
        this(configLocations, true, null);
    }
```

2.实际调用

```
public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {
        super(parent);
        this.setConfigLocations(configLocations);
        if (refresh) {
            this.refresh();
        }

    }
```

3.设置资源加载器和资源定位

通过分析FileSystemXmlApplicationContext的源代码可以知道，在创建FileSystemXmlApplicationContext容器时，构造方法做以下两项重要工作：

首先，调用父类容器的构造方法\(super\(parent\)方法\)为容器设置好Bean资源加载器。

然后，再调用父类AbstractRefreshableConfigApplicationContext的setConfigLocations\(configLocations\)方法设置Bean定义资源文件的定位路径。

通过追踪FileSystemXmlApplicationContext的继承体系，发现其父类的父类AbstractApplicationContext中初始化IoC容器所做的主要源码如下：

```
public abstract class AbstractApplicationContext extends DefaultResourceLoader  
        implements ConfigurableApplicationContext, DisposableBean {  
    //静态初始化块，在整个容器创建过程中只执行一次  
    static {  
        //为了避免应用程序在Weblogic8.1关闭时出现类加载异常加载问题，加载IoC容  
       //器关闭事件(ContextClosedEvent)类  
        ContextClosedEvent.class.getName();  
    }  
    //FileSystemXmlApplicationContext调用父类构造方法调用的就是该方法  
    public AbstractApplicationContext(ApplicationContext parent) {  
        this.parent = parent;  
        this.resourcePatternResolver = getResourcePatternResolver();  
    }  
    //获取一个Spring Source的加载器用于读入Spring Bean定义资源文件  
    protected ResourcePatternResolver getResourcePatternResolver() {  
        // AbstractApplicationContext继承DefaultResourceLoader，也是一个S  
        //Spring资源加载器，其getResource(String location)方法用于载入资源  
        return new PathMatchingResourcePatternResolver(this);  
    }   
……  
}
```

AbstractApplicationContext构造方法中调用PathMatchingResourcePatternResolver的构造方法创建Spring资源加载器：

```
public PathMatchingResourcePatternResolver(ResourceLoader resourceLoader) {  
        Assert.notNull(resourceLoader, "ResourceLoader must not be null");  
        //设置Spring的资源加载器  
        this.resourceLoader = resourceLoader;  
}
```

在设置容器的资源加载器之后，接下来FileSystemXmlApplicationContet执行setConfigLocations方法通过调用其父类AbstractRefreshableConfigApplicationContext的方法进行对Bean定义资源文件的定位，该方法的源码如下：

```
//处理单个资源文件路径为一个字符串的情况  
    public void setConfigLocation(String location) {  
       //String CONFIG_LOCATION_DELIMITERS = ",; /t/n";  
       //即多个资源文件路径之间用” ,; /t/n”分隔，解析成数组形式  
        setConfigLocations(StringUtils.tokenizeToStringArray(location, CONFIG_LOCATION_DELIMITERS));  
    }  

    //解析Bean定义资源文件的路径，处理多个资源文件字符串数组  
     public void setConfigLocations(String[] locations) {  
        if (locations != null) {  
            Assert.noNullElements(locations, "Config locations must not be null");  
            this.configLocations = new String[locations.length];  
            for (int i = 0; i < locations.length; i++) {  
                // resolvePath为同一个类中将字符串解析为路径的方法  
                this.configLocations[i] = resolvePath(locations[i]).trim();  
            }  
        }  
        else {  
            this.configLocations = null;  
        }  
    }
```

4.AbstractApplicationContext的refresh函数载入Bean定义过程：

Spring IoC容器对Bean定义资源的载入是从refresh\(\)函数开始的，refresh\(\)是一个模板方法，refresh\(\)方法的作用是：在创建IoC容器前，如果已经有容器存在，则需要把已有的容器销毁和关闭，以保证在refresh之后使用的是新建立起来的IoC容器。refresh的作用类似于对IoC容器的重启，在新建立好的容器中对容器进行初始化，对Bean定义资源进行载入

FileSystemXmlApplicationContext通过调用其父类AbstractApplicationContext的refresh\(\)函数启动整个IoC容器对Bean定义的载入过程：

```
public void refresh() throws BeansException, IllegalStateException {  
       synchronized (this.startupShutdownMonitor) {  
           //调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识  
           prepareRefresh();  
           //告诉子类启动refreshBeanFactory()方法，Bean定义资源文件的载入从  
          //子类的refreshBeanFactory()方法启动  
           ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();  
           //为BeanFactory配置容器特性，例如类加载器、事件处理器等  
           prepareBeanFactory(beanFactory);  
           try {  
               //为容器的某些子类指定特殊的BeanPost事件处理器  
               postProcessBeanFactory(beanFactory);  
               //调用所有注册的BeanFactoryPostProcessor的Bean  
               invokeBeanFactoryPostProcessors(beanFactory);  
               //为BeanFactory注册BeanPost事件处理器.  
               //BeanPostProcessor是Bean后置处理器，用于监听容器触发的事件  
               registerBeanPostProcessors(beanFactory);  
               //初始化信息源，和国际化相关.  
               initMessageSource();  
               //初始化容器事件传播器.  
               initApplicationEventMulticaster();  
               //调用子类的某些特殊Bean初始化方法  
               onRefresh();  
               //为事件传播器注册事件监听器.  
               registerListeners();  
               //初始化所有剩余的单态Bean.  
               finishBeanFactoryInitialization(beanFactory);  
               //初始化容器的生命周期事件处理器，并发布容器的生命周期事件  
               finishRefresh();  
           }  
           catch (BeansException ex) {  
               //销毁以创建的单态Bean  
               destroyBeans();  
               //取消refresh操作，重置容器的同步标识.  
               cancelRefresh(ex);  
               throw ex;  
           }  
       }  
   }
```

refresh\(\)方法主要为IoC容器Bean的生命周期管理提供条件，Spring IoC容器载入Bean定义资源文件从其子类容器的refreshBeanFactory\(\)方法启动，所以整个refresh\(\)中“ConfigurableListableBeanFactory beanFactory =obtainFreshBeanFactory\(\);”这句以后代码的都是注册容器的信息源和生命周期事件，载入过程就是从这句代码启动。

refresh\(\)方法的作用是：在创建IoC容器前，如果已经有容器存在，则需要把已有的容器销毁和关闭，以保证在refresh之后使用的是新建立起来的IoC容器。refresh的作用类似于对IoC容器的重启，在新建立好的容器中对容器进行初始化，对Bean定义资源进行载入

AbstractApplicationContext的obtainFreshBeanFactory\(\)方法调用子类容器的refreshBeanFactory\(\)方法，启动容器载入Bean定义资源文件的过程，代码如下：

```
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {  
        //这里使用了委派设计模式，父类定义了抽象的refreshBeanFactory()方法，具体实现调用子类容器的refreshBeanFactory()方法
         refreshBeanFactory();  
        ConfigurableListableBeanFactory beanFactory = getBeanFactory();  
        if (logger.isDebugEnabled()) {  
            logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);  
        }  
        return beanFactory;  
    }
```

AbstractApplicationContext子类的refreshBeanFactory\(\)方法：



   AbstractApplicationContext类中只抽象定义了refreshBeanFactory\(\)方法，容器真正调用的是其子类AbstractRefreshableApplicationContext实现的    refreshBeanFactory\(\)方法，方法的源码如下：



