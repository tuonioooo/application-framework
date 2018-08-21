# IOC机制从设计理念到源码解读

## 设计理念

IOC全称是Inversion of Control，即反转控制,或者说是依赖注入更为合适。选择别纠结这些全称的专业词。我们可以用别外一些方式去理解它，IOC，是一种设计模式。_**它的延生所要实现的是把藕合从代码中移出去，统一放到XML文件中，通过一个容器在需要的时候把这个依赖关系形成，即把需要的接口实现注入到需要它的类中**_，这可能就是“依赖注入”说法的来源了。

## 源码解读

主要涉及到`org.springframework.beans`和`org.springframework.context`包是Spring框架的IoC容器的基础，ApplicationContext、BeanFactory、BeanPostProcessor、BeanFactoryPostProcessor

[https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/core.html\#beans](https://docs.spring.io/spring/docs/5.0.8.RELEASE/spring-framework-reference/core.html#beans)

