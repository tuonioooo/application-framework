# Bean的生命周期

### **图1**![](/assets/import-springbean-01.png)

### **图2与上图类似，bean的生命周期流程图**：

![](/assets/import-springbean-02.png)

### **图3稍微复杂一点**

在spring中，singleton属性默认是true，只有设定为false，则每次指定别名取得的Bean时都会产生一个新的实例

一个Bean从创建到销毁，如果是用BeanFactory来生成,管理Bean的话，会经历几个执行阶段\(如图3\)：

图3

![](/assets/import-springbean-03.png)

1：Bean的建立：

容器寻找Bean的定义信息并将其实例化。

2：属性注入：

使用依赖注入，Spring按照Bean定义信息配置Bean所有属性

3：BeanNameAware的setBeanName\(\)：

如果Bean类有实现org.springframework.beans.BeanNameAware接口，工厂调用Bean的setBeanName\(\)方法传递Bean的ID。

4：BeanFactoryAware的setBeanFactory\(\)：

如果Bean类有实现org.springframework.beans.factory.BeanFactoryAware接口，工厂调用setBeanFactory\(\)方法传入工厂自身。

5：BeanPostProcessors的ProcessBeforeInitialization\(\)

如果有org.springframework.beans.factory.config.BeanPostProcessors和Bean关联，那么其postProcessBeforeInitialization\(\)方法将被将被调用。

6：initializingBean的afterPropertiesSet\(\)：

如果Bean类已实现org.springframework.beans.factory.InitializingBean接口，则执行他的afterProPertiesSet\(\)方法

7：Bean定义文件中定义init-method：

可以在Bean定义文件中使用"init-method"属性设定方法名称例如：

```
<bean id="life_singleton" class="com.bean.LifeBean" scope="singleton" 
            init-method="init" destroy-method="destory" lazy-init="true"/>
```

如果有以上设置的话，则执行到这个阶段，就会执行initBean\(\)方法

8：BeanPostProcessors的ProcessaAfterInitialization\(\)

如果有任何的BeanPostProcessors实例与Bean实例关联，则执行BeanPostProcessors实例的ProcessaAfterInitialization\(\)方法

此时，Bean已经可以被应用系统使用，并且将保留在BeanFactory中知道它不在被使用。有两种方法可以将其从BeanFactory中删除掉\(如图4\):

图4

![](/assets/import-springbean-04.png)

## **总结**

找工作的时候有些人会被问道Spring中Bean的生命周期，其实也就是考察一下对Spring是否熟悉，工作中很少用到其中的内容，那我们简单看一下。

```
在说明前可以思考一下Servlet的生命周期：实例化，初始init，接收请求service，销毁destroy；

Spring上下文中的Bean也类似，如下

1、实例化一个Bean－－也就是我们常说的new；

2、按照Spring上下文对实例化的Bean进行配置－－也就是IOC注入；

3、如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName\(String\)方法，此处传递的就是Spring配置文件中Bean的id值

4、如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory\(setBeanFactory\(BeanFactory\)传递的是Spring工厂自身（可以用这个方式来获取其它Bean，只需在Spring配置文件中配置一个普通的Bean就可以）；
```

5、如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext\(ApplicationContext\)方法，传入Spring上下文（同样这个方式也可以实现步骤4的内容，但比4更好，因为ApplicationContext是BeanFactory的子接口，有更多的实现方法）；

```
6、如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitialization\(Object obj, String s\)方法，BeanPostProcessor经常被用作是Bean内容的更改，并且由于这个是在Bean初始化结束时调用那个的方法，也可以被应用于内存或缓存技术；

7、如果Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法。

8、如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessAfterInitialization\(Object obj, String s\)方法、；

注：以上工作完成以后就可以应用这个Bean了，那这个Bean是一个Singleton的，所以一般情况下我们调用同一个id的Bean会是在内容地址相同的实例，当然在Spring配置文件中也可以配置非Singleton，这里我们不做赘述。

9、当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用那个其实现的destroy\(\)方法；

10、最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。
```



