# BeanPostProcessor源码讲解

## BeanPostProcessor API：

```
public interface BeanPostProcessor {  

    /** 
     * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean 
     * initialization callbacks (like InitializingBean's {@code afterPropertiesSet} 
     * or a custom init-method). The bean will already be populated with property values.    
     */  
    //实例化、依赖注入完毕，在调用显示的初始化之前完成一些定制的初始化任务  
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;  


    /** 
     * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean 
     * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}   
     * or a custom init-method). The bean will already be populated with property values.       
     */  
    //实例化、依赖注入、初始化完毕时执行  
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;  

}
```

## **BeanPostProcessor接口作用：**

如果我们想在Spring容器中完成bean实例化、配置以及其他初始化方法前后要添加一些自己逻辑处理。我们需要定义一个或多个BeanPostProcessor接口实现类，然后注册到Spring IoC容器中。

```
package com.test.spring;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
/**
 * bean后置处理器
 * @author zss
 *
 */
public class PostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean,
            String beanName) throws BeansException {
        if ("narCodeService".equals(beanName)) {//过滤掉bean实例ID为narCodeService
            return bean;
        }
        System.out.println("后置处理器处理bean=【"+beanName+"】开始");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean,
            String beanName) throws BeansException {
        if ("narCodeService".equals(beanName)) {
            return bean;
        }
        System.out.println("后置处理器处理bean=【"+beanName+"】完毕!");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return bean;
    }

}
```

> 注意:
>
> 接口中两个方法不能返回null，如果返回null那么在后续初始化方法将报空指针异常或者通过getBean\(\)方法获取不到bena实例对象
>
> 因为后置处理器从Spring IoC容器中取出bean实例对象没有再次放回IoC容器中

将Spring的后置处理器PostProcessor配置到Spring配置文件中

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 定义一个bean -->
     <bean id="narCodeService" class="com.test.service.impl.NarCodeServiceImpl">
     </bean>
    <bean id="beanLifecycle" class="com.test.spring.BeanLifecycle" init-method="init" destroy-method="close">
        <property name="name" value="张三"></property>
        <property name="sex" value="男"></property>
    </bean>

    <!-- Spring后置处理器 -->
    <bean id="postProcessor" class="com.test.spring.PostProcessor"/>
</beans>
```

```
由API可以看出:
```

  


1：后置处理器的postProcessorBeforeInitailization方法是在bean实例化，依赖注入之后及自定义初始化方法\(例如：配置文件中bean标签添加init-method属性指定Java类中初始化方法、

  


@PostConstruct注解指定初始化方法，Java类实现InitailztingBean接口\)之前调用

  


2：后置处理器的postProcessorAfterInitailization方法是在bean实例化、依赖注入及自定义初始化方法之后调用

  


  


注意：

  


   1.BeanFactory和ApplicationContext两个容器对待bean的后置处理器稍微有些不同。ApplicationContext容器会自动检测Spring配置文件中那些bean所对应的Java类实现了BeanPostProcessor

  


接口，并自动把它们注册为后置处理器。在创建bean过程中调用它们，所以部署一个后置处理器跟普通的bean没有什么太大区别。

  


      2.BeanFactory容器注册bean后置处理器时必须通过代码显示的注册，在IoC容器继承体系中的ConfigurableBeanFactory接口中定义了注册方法

参考

[https://docs.spring.io/spring/docs/5.0.8.RELEASE/javadoc-api/](https://docs.spring.io/spring/docs/5.0.8.RELEASE/javadoc-api/)

