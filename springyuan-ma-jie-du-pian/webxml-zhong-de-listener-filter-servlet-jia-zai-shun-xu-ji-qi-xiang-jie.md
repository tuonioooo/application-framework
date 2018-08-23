# web.xml 中的listener、 filter、servlet 加载顺序及其详解

在项目中总会遇到一些关于加载的优先级问题，刚刚就遇到了一个问题，由于项目中使用了quartz任务调度，quartz在web.xml中是使用listener进行监听的,使得在tomcat启动的时候能马上检查数据库查看那些任务未被按时执行，而数据库的配置信息在是在web.xml中使用servlet配置的，导致tomcat启动后在执行quartz任务时报空指针，原因就是servlet中的数据库连接信息未被加载。网上查询了下web.xml中配置的加载优先级：

首先可以肯定的是，加载顺序与它们在 web.xml 文件中的先后顺序无关。即不会因为 filter 写在 listener 的前面而会先加载 filter。最终得出的结论是：\_\_listener -&gt; filter -&gt; servlet\_\_

同时还存在着这样一种配置节：context-param，它用于向 ServletContext 提供键值对，即应用程序上下文信息。我们的 listener, filter 等在初始化时会用到这些上下文中的信息，那么 context-param 配置节是不是应该写在 listener 配置节前呢？实际上 context-param 配置节可写在任意位置，因此\_\_真正的加载顺序为：context-param -&gt; listener -&gt; filter -&gt; servlet\_\_

对于某类配置节而言，与它们出现的顺序是有关的。以 filter 为例，web.xml 中当然可以定义多个 filter，与 filter 相关的一个配置节是 filter-mapping，这里一定要注意，对于拥有相同 filter-name 的 filter 和 filter-mapping 配置节而言，filter-mapping 必须出现在 filter 之后，否则当解析到 filter-mapping 时，它所对应的 filter-name 还未定义。web 容器启动时初始化每个 filter 时，是按照 filter 配置节出现的顺序来初始化的，当请求资源匹配多个 filter-mapping 时，\_\_filter 拦截资源是按照 filter-mapping 配置节出现的顺序来依次调用\_\_ doFilter\\(\\) 方法的。

**servlet 同 filter 类似** ，此处不再赘述。

   由此，可以看出，web.xml 的加载顺序是：\*\*context-param -&gt; listener -&gt; filter -&gt; servlet\*\* ，而同个类型之间的实际程序调用的时候的顺序是根据对应的 mapping 的顺序进行调用的。

web.xml文件详解

