# 请求映射机制

HandlerMapping

作用是根据当前请求的找到对应的 Handler，并将 Handler（执行程序）与一堆 HandlerInterceptor（拦截器）封装到 HandlerExecutionChain 对象中。在 HandlerMapping 接口的内部只有一个方法，如下：

* HandlerExecutionChain getHandler\(HttpServletRequest request\);

HandlerMapping 是由 DispatcherServlet 调用，DispatcherServlet 会从容器中取出所有 HandlerMapping 实例并遍历，让 HandlerMapping 实例根据自己实现类的方式去尝试查找 Handler，而 HandlerMapping 具体有哪些实现类下面就会详细分析。

