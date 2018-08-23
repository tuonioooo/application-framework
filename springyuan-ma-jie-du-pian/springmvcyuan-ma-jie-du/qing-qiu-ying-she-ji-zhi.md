# 请求映射机制

## HandlerMapping

作用是根据当前请求的找到对应的 Handler，并将 Handler（执行程序）与一堆 HandlerInterceptor（拦截器）封装到 HandlerExecutionChain 对象中。在 HandlerMapping 接口的内部只有一个方法，如下：

* HandlerExecutionChain getHandler\(HttpServletRequest request\);

HandlerMapping 是由 DispatcherServlet 调用，DispatcherServlet 会从容器中取出所有 HandlerMapping 实例并遍历，让 HandlerMapping 实例根据自己实现类的方式去尝试查找 Handler，而 HandlerMapping 具体有哪些实现类下面就会详细分析。

```
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    // 这些 HandlerMapping 在容器初始化时创建，在 initHandlerMappings 时放入集合中
    for (HandlerMapping hm : this.handlerMappings) {
        HandlerExecutionChain handler = hm.getHandler(request);
        if (handler != null) {
            return handler;
        }
    }
    return null;
}
```

另外上面说到的 Handler 有可能是一个 HandlerMethod（封装了 Controller 中的方法）对象，也有可能是一个 Controller 对象、 HttpRequestHandler 对象或 Servlet 对象，而这个 Handler 具体是什么对象，也是与所使用的 HandlerMapping 实现类有关。如下图所示，可以看到 HandlerMapping 实现类有两个分支，分别继承自 AbstractHandlerMethodMapping（得到 HandlerMethod）和 AbstractUrlHandlerMapping（得到 HttpRequestHandler、Controller 或 Servlet），它们又统一继承于 AbstractHandlerMapping。

![](/assets/import-handlerMapping-01.png)

先来看一下 AbstractHandlerMapping，它实现了 HandlerMapping 接口中的 getHandler\(\) 方法，源码如下所示

```
@Override
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    // 根据请求获取执行程序，具体的获取方式由子类决定，getHandlerInternal() 是抽象方法
    Object handler = getHandlerInternal(request);
    if (handler == null) {
        handler = getDefaultHandler();
    }
    if (handler == null) {
        return null;
    }
    // Bean name or resolved handler?
    if (handler instanceof String) {
        String handlerName = (String) handler;
        handler = getApplicationContext().getBean(handlerName);
    }
    // 将 Handler 与一堆拦截器包装到 HandlerExecutionChain 对象中
    return getHandlerExecutionChain(handler, request);
}
```

以看到在这个方法中又调用了 getHandlerInternal\(\) 方法获取到了 Handler 对象，而 Handler 对象具体内容是由它的子类去定义的。下面就来一看下 AbstractHandlerMapping 的两个分支子类

**1 AbstractUrlHandlerMapping**

AbstractUrlHandlerMapping 这个分支获取的 Handler 的类型实际就是一个 Controller 类，所以一个 Controller 只能对应一个请求（或者像 Struts2 那样定位到方法，使同一个业务的方法放在同一个类里），源码如下所示

```
protected Object getHandlerInternal(HttpServletRequest request) throws Exception {
    // 根据当前请求获取“查找路径”
    String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
    // 根据路径获取 Handler（即Controller），先尝试直接匹配，再尝试模式匹配
    Object handler = lookupHandler(lookupPath, request);
    if (handler == null) {
       // We need to care for the default handler directly, since we need to
        // expose the PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE for it as well.
        Object rawHandler = null;
        if ("/".equals(lookupPath)) {
            rawHandler = getRootHandler();
        }
        if (rawHandler == null) {
            rawHandler = getDefaultHandler();
        }
        if (rawHandler != null) {
            // Bean name or resolved handler?
            if (rawHandler instanceof String) {
                String handlerName = (String) rawHandler;
                rawHandler = getApplicationContext().getBean(handlerName);
            }
            validateHandler(rawHandler, request);
            handler = buildPathExposingHandler(rawHandler, lookupPath, lookupPath, null);
        }
    }
    return handler;
}
```

**1.1 AbstractUrlHandlerMapping 实现类及使用**

![](/assets/import-urlmapping-01.png)

* ** ControllerClassNameHandlerMapping：根据类名访问 Controller。**

```
<!-- 注册 HandlerMapping -->
<bean class="org.springframework.web.servlet.handler.ControllerClassNameHandlerMapping" />
<!-- 注册 Handler -->
<bean class="com.controller.TestController" />
```

* **ControllerBeanNameHandlerMapping：根据 Bean 名访问 Controller，与 BeanNameUrlHandlerMapping 类似，但是bean名称不用遵循URL公约**。

```
<!-- 注册 HandlerMapping -->
<bean class="org.springframework.web.servlet.handler.ControllerBeanNameHandlerMapping" />
<!-- 注册 Handler -->
<bean id="test" class="com.controller.TestController" />
```

* **BeanNameUrlHandlerMapping：利用 BeanName 来作为 URL 使用。**

```
<!-- 注册 HandlerMapping -->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
<!-- 注册 Handler -->
<bean id="/test.do" class="com.controller.TestController" />
```



