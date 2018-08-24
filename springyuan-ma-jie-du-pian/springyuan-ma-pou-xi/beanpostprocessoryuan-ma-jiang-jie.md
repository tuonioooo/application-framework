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

> 注意：
>
> 1.BeanFactory和ApplicationContext两个容器对待bean的后置处理器稍微有些不同。ApplicationContext容器会自动检测Spring配置文件中那些bean所对应的Java类实现了BeanPostProcessor
>
> 接口，并自动把它们注册为后置处理器。在创建bean过程中调用它们，所以部署一个后置处理器跟普通的bean没有什么太大区别。
>
> 2.BeanFactory容器注册bean后置处理器时必须通过代码显示的注册，在IoC容器继承体系中的ConfigurableBeanFactory接口中定义了注册方法

##  Spring如何调用多个BeanPostProcessor实现类：

我们可以在Spring配置文件中添加多个BeanPostProcessor\(后置处理器\)接口实现类，在默认情况下Spring容器会根据后置处理器的定义顺序来依次调用。

**Spring配置文件：**

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- bean定义 -->    
    <bean id="narCodeService" class="com.test.service.impl.NarCodeServiceImpl">
    </bean>
    <bean id="postProcessor" class="com.test.spring.PostProcessor"/>
    <bean id="postProcessorB" class="com.test.spring.PostProcessorB"/>
</beans>
```

**BeanPostProcessor实现类：**

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
        System.out.println("后置处理器处理bean=【"+beanName+"】完毕!");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return bean;
    }
}
----------------------------------------------------------------------------------------------------------------------------------------
package com.test.spring;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class PostProcessorB implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean,
            String beanName) throws BeansException {
        System.out.println("后置处理器开始调用了");
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
        System.out.println("后置处理器调用结束了");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return bean;
    }
}
```

**测试**

```
package com.test.spring;

import org.junit.Before;
import org.junit.Test;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class T {
    AbstractApplicationContext applicationcontext=null;
    @Before
    public void before() {
        System.out.println("》》》Spring ApplicationContext容器开始初始化了......");
        applicationcontext= new ClassPathXmlApplicationContext(new String[]{"test1-service.xml"});
        System.out.println("》》》Spring ApplicationContext容器初始化完毕了......");
    }
    @Test
    public void  test() {
        applicationcontext.registerShutdownHook();   
    }
}
```

**测试结果**

```
》》》Spring ApplicationContext容器开始初始化了......
2017-03-19 10:50:29  INFO:ClassPathXmlApplicationContext-Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@18c92ff9: startup date [Sun Mar 19 10:50:29 CST 2017]; root of context hierarchy
2017-03-19 10:50:29  INFO:XmlBeanDefinitionReader-Loading XML bean definitions from class path resource [test1-service.xml]
后置处理器处理bean=【narCodeService】开始
后置处理器开始调用了
后置处理器处理bean=【narCodeService】完毕!
后置处理器调用结束了
》》》Spring ApplicationContext容器初始化完毕了......
2017-03-19 10:50:34  INFO:ClassPathXmlApplicationContext-Closing org.springframework.context.support.ClassPathXmlApplicationContext@18c92ff9: startup date [Sun Mar 19 10:50:29 CST 2017]; root of context hierarchy
```

   在Spring机制中可以指定后置处理器调用顺序，通过让BeanPostProcessor接口实现类实现Ordered接口getOrder方法，该方法返回一整数，默认值为 0，优先级最高，值越大优先级越低

例如：

```
package com.test.spring;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.core.Ordered;
/**
 * bean后置处理器
 * @author zss
 *
 */
public class PostProcessor implements BeanPostProcessor,Ordered{

    @Override
    public Object postProcessBeforeInitialization(Object bean,
            String beanName) throws BeansException {
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
        System.out.println("后置处理器处理bean=【"+beanName+"】完毕!");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return bean;
    }

    @Override
    public int getOrder() {
        return 1;
    }
}
----------------------------------------------------------------------------
package com.test.spring;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.core.Ordered;

public class PostProcessorB implements BeanPostProcessor,Ordered {

    @Override
    public Object postProcessBeforeInitialization(Object bean,
            String beanName) throws BeansException {
        System.out.println("后置处理器开始调用了");
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
        System.out.println("后置处理器调用结束了");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return bean;
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

```
测试结果：

》》》Spring ApplicationContext容器开始初始化了......
2017-03-19 11:04:10  INFO:ClassPathXmlApplicationContext-Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@18c92ff9: startup date [Sun Mar 19 11:04:10 CST 2017]; root of context hierarchy
2017-03-19 11:04:10  INFO:XmlBeanDefinitionReader-Loading XML bean definitions from class path resource [test1-service.xml]
后置处理器开始调用了
后置处理器处理bean=【narCodeService】开始
后置处理器调用结束了
后置处理器处理bean=【narCodeService】完毕!
》》》Spring ApplicationContext容器初始化完毕了......
2017-03-19 11:04:14  INFO:ClassPathXmlApplicationContext-Closing org.springframework.context.support.ClassPathXmlApplicationContext@18c92ff9: startup date [Sun Mar 19 11:04:10 CST 2017]; root of context hierarchy
```

## 参考

[https://docs.spring.io/spring/docs/5.0.8.RELEASE/javadoc-api/](https://docs.spring.io/spring/docs/5.0.8.RELEASE/javadoc-api/)

