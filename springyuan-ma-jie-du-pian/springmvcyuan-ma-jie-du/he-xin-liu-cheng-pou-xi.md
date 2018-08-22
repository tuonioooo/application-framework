# 核心流程剖析及原理分析

## DispatcherServlet 处理流程

在整个 Spring MVC 框架中，DispatcherServlet 处于核心位置，它负责协调和组织不同组件完成请求处理并返回响应工作。在看 DispatcherServlet 类之前，我们先来看一下请求处理的大致流程：

1. Tomcat 启动，对 DispatcherServlet 进行实例化，然后调用它的 init\(\) 方法进行初始化，在这个初始化过程中完成了：
2. 对 web.xml 中初始化参数的加载；建立 WebApplicationContext \(SpringMVC的IOC容器\)；进行组件的初始化；

3. 客户端发出请求，由 Tomcat 接收到这个请求，如果匹配 DispatcherServlet 在 web.xml 中配置的映射路径，Tomcat 就将请求转交给 DispatcherServlet 处理；

4. DispatcherServlet 从容器中取出所有 HandlerMapping 实例（每个实例对应一个 HandlerMapping 接口的实现类）并遍历，每个 HandlerMapping 会根据请求信息，通过自己实现类中的方式去找到处理该请求的 Handler \(执行程序，如Controller中的方法\)，并且将这个 Handler 与一堆 HandlerInterceptor \(拦截器\) 封装成一个 HandlerExecutionChain 对象，一旦有一个 HandlerMapping 可以找到 Handler 则退出循环；（详情可以看  
   [\[Java\]SpringMVC工作原理之二：HandlerMapping和HandlerAdpater](https://www.cnblogs.com/tengyunhao/p/www)  
    这篇文章）

5. DispatcherServlet 取出 HandlerAdapter 组件，根据已经找到的 Handler，再从所有 HandlerAdapter 中找到可以处理该 Handler 的 HandlerAdapter 对象；
6. 执行 HandlerExecutionChain 中所有拦截器的 preHandler\(\) 方法，然后再利用 HandlerAdapter 执行 Handler ，执行完成得到 ModelAndView，再依次调用拦截器的 postHandler\(\) 方法；
7. 利用 ViewResolver 将 ModelAndView 或是 Exception（可解析成 ModelAndView）解析成 View，然后 View 会调用 render\(\) 方法再根据 ModelAndView 中的数据渲染出页面；
8. 最后再依次调用拦截器的 afterCompletion\(\) 方法，这一次请求就结束了。

示例**一**![](/assets/import-dispatcher-01.png)

示例**二**

![](/assets/import-Dispatcher-02.png)

示例三

![](/assets/import-dispatcher-03.png)

## DispatcherServlet 源码分析

![](https://images2017.cnblogs.com/blog/1118116/201709/1118116-20170914121516563-1545541531.png)

DispatcherServlet 继承自 HttpServlet，它遵循 Servlet 里的“init-service-destroy”三个阶段，首先我们先来看一下它的 init\(\) 阶段。

1 初始化

1.1 HttpServletBean 的 init\(\) 方法

DispatcherServlet 的 init\(\) 方法在其父类 **HttpServletBean** 中实现的，它覆盖了 GenericServlet 的 init\(\) 方法，主要作用是加载 web.xml 中 

DispatcherServlet 的 &lt;init-param&gt; 配置，并调用子类的初始化。下面是 init\(\) 方法的具体代码：

```
@Override
public final void init() throws ServletException {
    try {
        // ServletConfigPropertyValues 是静态内部类，使用 ServletConfig 获取 web.xml 中配置的参数
        PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
        // 使用 BeanWrapper 来构造 DispatcherServlet
        BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
        ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
        bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
        initBeanWrapper(bw);
        bw.setPropertyValues(pvs, true);
    } catch (BeansException ex) {}
    // 让子类实现的方法，这种在父类定义在子类实现的方式叫做模版方法模式
    initServletBean();
}
```

1.2 FrameworkServlet 的 initServletBean\(\) 方法

在 HttpServletBean 的 init\(\) 方法中调用了 initServletBean\(\) 这个方法，它是在**FrameworkServlet**类中实现的，主要作用是建立 WebApplicationContext 容器（有时也称上下文），并加载 SpringMVC 配置文件中定义的 Bean 到改容器中，最后将该容器添加到 ServletContext 中。下面是 initServletBean\(\) 方法的具体代码：

```
@Override
protected final void initServletBean() throws ServletException {
    try {
        // 初始化 WebApplicationContext (即SpringMVC的IOC容器)
        this.webApplicationContext = initWebApplicationContext();
        initFrameworkServlet();
    } catch (ServletException ex) {
    } catch (RuntimeException ex) {
    }
}
```

WebApplicationContext 继承于 ApplicationContext 接口，从容器中可以获取当前应用程序环境信息，它也是 SpringMVC 的 IOC 容器。下面是 initWebApplicationContext\(\) 方法的具体代码：

```
protected WebApplicationContext initWebApplicationContext() {
    // 获取 ContextLoaderListener 初始化并注册在 ServletContext 中的根容器，即 Spring 的容器
    WebApplicationContext rootContext =
    　　　　WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;
    if (this.webApplicationContext != null) {
        // 因为 WebApplicationContext 不为空，说明该类在构造时已经将其注入
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {
                if (cwac.getParent() == null) {
                    // 将 Spring 的容器设为 SpringMVC 容器的父容器
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
    　　// 如果 WebApplicationContext 为空，则进行查找，能找到说明上下文已经在别处初始化。
    　　wac = findWebApplicationContext();
    }
    if (wac == null) {
        // 如果 WebApplicationContext 仍为空，则以 Spring 的容器为父上下文建立一个新的。
        wac = createWebApplicationContext(rootContext);
    }
    if (!this.refreshEventReceived) {
        // 模版方法，由 DispatcherServlet 实现
        onRefresh(wac);
    }
    if (this.publishContext) {
        // 发布这个 WebApplicationContext 容器到 ServletContext 中
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
    }
    return wac;
}
```

下面是查找 WebApplicationContext 的 findWebApplicationContext\(\) 方法代码：

```
protected WebApplicationContext findWebApplicationContext() {
    String attrName = getContextAttribute();
    if (attrName == null) {
        return null;
    }
    // 从 ServletContext 中查找已经发布的 WebApplicationContext 容器
    WebApplicationContext wac =
    WebApplicationContextUtils.getWebApplicationContext(getServletContext(), attrName);
    if (wac == null) {
        throw new IllegalStateException("No WebApplicationContext found: initializer not registered?");
    }
    return wac;
}
```

1.3 DispatcherServlet 的 onRefresh\(\) 方法

建立好 WebApplicationContext\(上下文\) 后，通过 onRefresh\(ApplicationContext context\) 方法回调，进入 DispatcherServlet 类中。onRefresh\(\) 方法，提供 SpringMVC 的初始化，具体代码如下：

```
    @Override
    protected void onRefresh(ApplicationContext context) {
        initStrategies(context);
    }
    protected void initStrategies(ApplicationContext context) {
        initMultipartResolver(context);
        initLocaleResolver(context);
        initThemeResolver(context);
        initHandlerMappings(context);
        initHandlerAdapters(context);
        initHandlerExceptionResolvers(context);
        initRequestToViewNameTranslator(context);
        initViewResolvers(context);
        initFlashMapManager(context);
    }
```

在 initStrategies\(\) 方法中进行了各个组件的初始化，先来看一下这些组件的初始化方法，稍后再来详细分析这些组件。



