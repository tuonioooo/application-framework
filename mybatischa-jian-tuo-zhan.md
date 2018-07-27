# Mybatis插件拓展

官网详细教程：[http://www.mybatis.org/mybatis-3/zh/configuration.html\#plugins](http://www.mybatis.org/mybatis-3/zh/configuration.html#plugins)



## **插件原理**

* 首先，我们从插件&lt;plugins&gt;解析开始，源码位于XMLConfigBuilder的pluginElement方法中：

```
private void pluginElement(XNode parent) throws Exception {
        if (parent != null) {
            Iterator var2 = parent.getChildren().iterator();

            while(var2.hasNext()) {
                XNode child = (XNode)var2.next();
                String interceptor = child.getStringAttribute("interceptor");
                Properties properties = child.getChildrenAsProperties();
                Interceptor interceptorInstance = (Interceptor)this.resolveClass(interceptor).newInstance();
                interceptorInstance.setProperties(properties);
                this.configuration.addInterceptor(interceptorInstance);
            }
        }

    }
```

这里拿&lt;plugin&gt;标签中的interceptor属性，这是自定义的拦截器的全路径，第6行的代码通过反射生成拦截器实例。

再拿&lt;plugin&gt;标签下的所有&lt;property&gt;标签，解析name和value属性成为一个Properties，将Properties设置到拦截器中。

最后，通过第8行的代码将拦截器设置到Configuration中，源码实现为：

```
public void addInterceptor(Interceptor interceptor) {
        this.interceptorChain.addInterceptor(interceptor);
    }
```



