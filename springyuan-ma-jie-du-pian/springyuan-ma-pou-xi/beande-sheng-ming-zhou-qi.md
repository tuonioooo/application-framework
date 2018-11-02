# Bean的生命周期

**图1**![](/assets/import-springbean-01.png)**图2与上图类似，bean的生命周期流程图**：

![](/assets/import-springbean-02.png)



**图3稍微复杂一点**

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

