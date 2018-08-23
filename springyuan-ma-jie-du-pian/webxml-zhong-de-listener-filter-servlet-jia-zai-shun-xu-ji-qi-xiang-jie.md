# web.xml 中的listener、 filter、servlet 加载顺序及其详解

## 概述

在项目中总会遇到一些关于加载的优先级问题，刚刚就遇到了一个问题，由于项目中使用了quartz任务调度，quartz在web.xml中是使用listener进行监听的,使得在tomcat启动的时候能马上检查数据库查看那些任务未被按时执行，而数据库的配置信息在是在web.xml中使用servlet配置的，导致tomcat启动后在执行quartz任务时报空指针，原因就是servlet中的数据库连接信息未被加载。网上查询了下web.xml中配置的加载优先级：

首先可以肯定的是，加载顺序与它们在 web.xml 文件中的先后顺序无关。即不会因为 filter 写在 listener 的前面而会先加载 filter。最终得出的结论是：**listener -&gt; filter -&gt; servlet**

同时还存在着这样一种配置节：context-param，它用于向 ServletContext 提供键值对，即应用程序上下文信息。我们的 listener, filter 等在初始化时会用到这些上下文中的信息，那么 context-param 配置节是不是应该写在 listener 配置节前呢？实际上 context-param 配置节可写在任意位置，因此\_\_真正的加载顺序为：context-param -&gt; listener -&gt; filter -&gt; servlet

对于某类配置节而言，与它们出现的顺序是有关的。以 filter 为例，web.xml 中当然可以定义多个 filter，与 filter 相关的一个配置节是 filter-mapping，这里一定要注意，对于拥有相同 filter-name 的 filter 和 filter-mapping 配置节而言，filter-mapping 必须出现在 filter 之后，否则当解析到 filter-mapping 时，它所对应的 filter-name 还未定义。web 容器启动时初始化每个 filter 时，是按照 filter 配置节出现的顺序来初始化的，当请求资源匹配多个 filter-mapping 时，filter 拦截资源是按照 filter-mapping 配置节出现的顺序来依次调用doFilter\(\) 方法的。

**servlet 同 filter 类似** ，此处不再赘述。

由此，可以看出，web.xml 的加载顺序是：\*\*context-param -&gt; listener -&gt; filter -&gt; servlet ，而同个类型之间的实际程序调用的时候的顺序是根据对应的 mapping 的顺序进行调用的。

## **web.xml文件详解**

* web.xml常用元素    

```
<web-app>    
<display-name></display-name>定义了WEB应用的名字    
<description></description> 声明WEB应用的描述信息    

<context-param></context-param> context-param元素声明应用范围内的初始化参数。    
<filter></filter> 过滤器元素将一个名字与一个实现javax.servlet.Filter接口的类相关联。    
<filter-mapping></filter-mapping> 一旦命名了一个过滤器，就要利用filter-mapping元素把它与一个或多个servlet或JSP页面相关联。    
<listener></listener>servlet API的版本2.3增加了对事件监听程序的支持，事件监听程序在建立、修改和删除会话或servlet环境时得到通知。    
                     Listener元素指出事件监听程序类。    
<servlet></servlet> 在向servlet或JSP页面制定初始化参数或定制URL时，必须首先命名servlet或JSP页面。Servlet元素就是用来完成此项任务的。    
<servlet-mapping></servlet-mapping> 服务器一般为servlet提供一个缺省的URL：http://host/webAppPrefix/servlet/ServletName。    
              但是，常常会更改这个URL，以便servlet可以访问初始化参数或更容易地处理相对URL。在更改缺省URL时，使用servlet-mapping元素。    

<session-config></session-config> 如果某个会话在一定时间内未被访问，服务器可以抛弃它以节省内存。    
          可通过使用HttpSession的setMaxInactiveInterval方法明确设置单个会话对象的超时值，或者可利用session-config元素制定缺省超时值。    

<mime-mapping></mime-mapping>如果Web应用具有想到特殊的文件，希望能保证给他们分配特定的MIME类型，则mime-mapping元素提供这种保证。    
<welcome-file-list></welcome-file-list> 指示服务器在收到引用一个目录名而不是文件名的URL时，使用哪个文件。    
<error-page></error-page> 在返回特定HTTP状态代码时，或者特定类型的异常被抛出时，能够制定将要显示的页面。    
<taglib></taglib> 对标记库描述符文件（Tag Libraryu Descriptor file）指定别名。此功能使你能够更改TLD文件的位置，    
                  而不用编辑使用这些文件的JSP页面。    
<resource-env-ref></resource-env-ref>声明与资源相关的一个管理对象。    
<resource-ref></resource-ref> 声明一个资源工厂使用的外部资源。    
<security-constraint></security-constraint> 制定应该保护的URL。它与login-config元素联合使用    
<login-config></login-config> 指定服务器应该怎样给试图访问受保护页面的用户授权。它与sercurity-constraint元素联合使用。    
<security-role></security-role>给出安全角色的一个列表，这些角色将出现在servlet元素内的security-role-ref元素    
                   的role-name子元素中。分别地声明角色可使高级IDE处理安全信息更为容易。    
<env-entry></env-entry>声明Web应用的环境项。    
<ejb-ref></ejb-ref>声明一个EJB的主目录的引用。    
< ejb-local-ref></ ejb-local-ref>声明一个EJB的本地主目录的应用。    
</web-app>
```



