# BeanFactoryPostProcessor源码讲解

## 前言

BeanFactoryPostProcessor和BeanPostProcessor都是spring初始化bean时对外暴露的扩展点。但它们有什么区别呢？  
由《[理解Bean生命周期](http://blog.csdn.net/soonfly/article/details/69480058)》的图可知：BeanFactoryPostProcessor是生命周期中最早被调用的，远远早于BeanPostProcessor。它在spring容器加载了bean的定义文件之后，在bean实例化之前执行的。也就是说，Spring允许BeanFactoryPostProcessor在容器创建bean之前读取bean配置元数据，并可进行修改。例如增加bean的属性和值，重新设置bean是否作为自动装配的侯选者，重设bean的依赖项等等，因此_**BeanPostProcessor和BeanFactoryPostProcessor主要区别在于：BeanFactoryPostProcessor可以操作bean配置元数据 ;**_

## 接口定义

在srping配置文件中可以同时配置多个BeanFactoryPostProcessor，并通过在xml中注册时设置’order’属性来控制各个BeanFactoryPostProcessor的执行次序。

**BeanFactoryPostProcessor接口定义如下：**

```
public interface BeanFactoryPostProcessor {
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

> 接口详细说明：
>
> 在应用程序上下文的标准初始化之后修改其内部bean工厂。所有bean定义都已加载，但尚未实例化bean。这允许覆盖或添加属性，即使是对急切初始化bean。

接口只有一个方法postProcessBeanFactory。该方法的参数是ConfigurableListableBeanFactory类型，实际开发中，我们常使用它的getBeanDefinition\(\)方法获取某个bean的元数据定义，请参考 [BeanDefinition源码解析](/springyuan-ma-jie-du-pian/springyuan-ma-pou-xi/beandefinitionyuan-ma-jie-xi.md)

## **示例**

**xml配置**

```
<bean id="messi" class="twm.spring.LifecycleTest.footballPlayer">
    <property name="name" value="Messi"></property>
    <property name="team" value="Barcelona"></property>
</bean>
```

**创建类beanFactoryPostProcessorImpl，实现接口BeanFactoryPostProcessor：**

```
public class beanFactoryPostProcessorImpl implements BeanFactoryPostProcessor{

    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("beanFactoryPostProcessorImpl");
        BeanDefinition bdefine=beanFactory.getBeanDefinition("messi");
        System.out.println(bdefine.getPropertyValues().toString());
        MutablePropertyValues pv =  bdefine.getPropertyValues();  
            if (pv.contains("team")) {
                PropertyValue ppv= pv.getPropertyValue("name");
                TypedStringValue obj=(TypedStringValue)ppv.getValue();
                if(obj.getValue().equals("Messi")){
                    pv.addPropertyValue("team", "阿根延");  
                }
        }  
            bdefine.setScope(BeanDefinition.SCOPE_PROTOTYPE);
    }
}
```

**调用类：**

```
public static void main(String[] args) throws Exception {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
    footballPlayer obj = ctx.getBean("messi",footballPlayer.class);
    System.out.println(obj.getTeam());
}
```

输出：

```
PropertyValues: length=2; bean property ‘name’; bean property ‘team’ 
阿根延
```



