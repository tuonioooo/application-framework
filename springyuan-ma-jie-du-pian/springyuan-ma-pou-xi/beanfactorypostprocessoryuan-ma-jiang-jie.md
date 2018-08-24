# BeanFactoryPostProcessor源码讲解

## 前言

BeanFactoryPostProcessor和BeanPostProcessor都是spring初始化bean时对外暴露的扩展点。但它们有什么区别呢？  
由《[理解Bean生命周期](http://blog.csdn.net/soonfly/article/details/69480058)》的图可知：BeanFactoryPostProcessor是生命周期中最早被调用的，远远早于BeanPostProcessor。它在spring容器加载了bean的定义文件之后，在bean实例化之前执行的。也就是说，Spring允许BeanFactoryPostProcessor在容器创建bean之前读取bean配置元数据，并可进行修改。例如增加bean的属性和值，重新设置bean是否作为自动装配的侯选者，重设bean的依赖项等等，因此_**BeanPostProcessor和BeanFactoryPostProcessor主要区别在于：BeanFactoryPostProcessor可以操作bean配置元数据 ;**_



