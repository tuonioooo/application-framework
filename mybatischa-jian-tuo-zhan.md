# Mybatis插件拓展

官网详细教程：[http://www.mybatis.org/mybatis-3/zh/configuration.html\#plugins](http://www.mybatis.org/mybatis-3/zh/configuration.html#plugins)

## **插件原理**

* **首先，我们从插件&lt;plugins&gt;解析开始，源码位于XMLConfigBuilder的pluginElement方法中：**

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

InterceptorChain是一个拦截器链，存储了所有定义的拦截器以及相关的几个操作的方法：

```
public class InterceptorChain {
    private final List<Interceptor> interceptors = new ArrayList();

    public InterceptorChain() {
    }

    public Object pluginAll(Object target) {
        Interceptor interceptor;
        for(Iterator var2 = this.interceptors.iterator(); var2.hasNext(); target = interceptor.plugin(target)) {
            interceptor = (Interceptor)var2.next();
        }

        return target;
    }

    public void addInterceptor(Interceptor interceptor) {
        this.interceptors.add(interceptor);
    }

    public List<Interceptor> getInterceptors() {
        return Collections.unmodifiableList(this.interceptors);
    }
}
```

分别有添加拦截器、为目标对象添加所有拦截器、获取当前所有拦截器三个方法。

* **pluginAll方法添加插件**

```
public Object pluginAll(Object target) {
        Interceptor interceptor;
        for(Iterator var2 = this.interceptors.iterator(); var2.hasNext(); target = interceptor.plugin(target)) {
            interceptor = (Interceptor)var2.next();
        }
        return target;
}
```

上面我们在InterceptorChain中看到了一个pluginAll方法，_**pluginAll方法为目标对象生成代理，之后目标对象调用方法的时候走的不是原方法而是代理方法**_

> 注意的是：
>
> 1. 形参Object target，这个是Executor、ParameterHandler、ResultSetHandler、StatementHandler接口的实现类，换句话说，plugin方法是要为Executor、ParameterHandler、ResultSetHandler、StatementHandler的实现类生成代理，从而在调用这几个类的方法的时候，其实调用的是InvocationHandler的invoke方法
> 2. 这里的target是通过for循环不断赋值的，也就是说如果有多个拦截器，那么如果我用P表示代理，生成第一次代理为P\(target\)，生成第二次代理为P\(P\(target\)\)，生成第三次代理为P\(P\(P\(target\)\)\)，不断嵌套下去，这就得到一个重要的结论：&lt;plugins&gt;...&lt;/plugins&gt;中后定义的&lt;plugin&gt;实际其拦截器方法先被执行，因为根据这段代码来看，后定义的&lt;plugin&gt;代理实际后生成，包装了先生成的代理，自然其代理方法也先执行

* plugin方法中调用MyBatis提供的现成的生成代理的方法Plugin.wrap\(Object target, Interceptor interceptor\)，接着我们看下wrap方法的源码实现。

Plugin的wrap方法实现为：

```
public static Object wrap(Object target, Interceptor interceptor) {
        Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
        Class<?> type = target.getClass();
        Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
        return interfaces.length > 0 ? Proxy.newProxyInstance(type.getClassLoader(), interfaces, new Plugin(target, interceptor, signatureMap)) : target;
    }
```

首先看一下第2行的代码，获取Interceptor上定义的所有方法签名：

```
private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
        Intercepts interceptsAnnotation = (Intercepts)interceptor.getClass().getAnnotation(Intercepts.class);
        if (interceptsAnnotation == null) {
            throw new PluginException("No @Intercepts annotation was found in interceptor " + interceptor.getClass().getName());
        } else {
            Signature[] sigs = interceptsAnnotation.value();
            Map<Class<?>, Set<Method>> signatureMap = new HashMap();
            Signature[] var4 = sigs;
            int var5 = sigs.length;

            for(int var6 = 0; var6 < var5; ++var6) {
                Signature sig = var4[var6];
                Set<Method> methods = (Set)signatureMap.get(sig.type());
                if (methods == null) {
                    methods = new HashSet();
                    signatureMap.put(sig.type(), methods);
                }

                try {
                    Method method = sig.type().getMethod(sig.method(), sig.args());
                    ((Set)methods).add(method);
                } catch (NoSuchMethodException var10) {
                    throw new PluginException("Could not find method on " + sig.type() + " named " + sig.method() + ". Cause: " + var10, var10);
                }
            }

            return signatureMap;
        }
    }
```

> 看到先拿@Intercepts注解，如果没有定义@Intercepts注解，抛出异常，这意味着**使用MyBatis的插件，必须使用注解方式**。
>
> 接着拿到@Intercepts注解下的所有@Signature注解，获取其type属性（表示具体某个接口），再根据method与args两个属性去type下找方法签名一致的方法Method（如果没有方法签名一致的就抛出异常，此签名的方法在该接口下找不到），能找到的话key=type，value=Set&lt;Method&gt;，添加到signatureMap中，构建出一个方法签名映射。举个例子来说，就是我定义的@Intercepts注解，Executor下我要拦截的所有Method、StatementHandler下我要拦截的所有Method。

回过头继续看wrap方法，在拿到方法签名映射后，调用getAllInterfaces方法，传入的是Target的Class对象以及之前获取到的方法签名映射：

```
private static Class<?>[] getAllInterfaces(Class<?> type, Map<Class<?>, Set<Method>> signatureMap) {
    Set<Class<?>> interfaces = new HashSet<Class<?>>();
    while (type != null) {
      for (Class<?> c : type.getInterfaces()) {
        if (signatureMap.containsKey(c)) {
          interfaces.add(c);
        }
      }
      type = type.getSuperclass();
    }
    return interfaces.toArray(new Class<?>[interfaces.size()]);
}
```



