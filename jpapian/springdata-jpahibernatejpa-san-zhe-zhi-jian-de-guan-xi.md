# SpringData Jpa、Hibernate、Jpa 三者之间的关系

### JPA规范与ORM框架之间的关系是怎样的呢？

JPA规范本质上就是一种ORM规范，注意不是ORM框架——因为JPA并未提供ORM实现，它只是制订了一些规范，提供了一些编程的API接口，但具体实现则由服务厂商来提供实现，JBoss应用服务器底层就以Hibernate作为JPA的实现。

既然JPA作为一种规范——也就说JPA规范中提供的只是一些接口，显然接口不能直接拿来使用。虽然应用程序可以面向接口编程，但JPA底层一定需要某种JPA实现，否则JPA依然无法使用。

从笔者的视角来看，Sun之所以提出JPA规范，其目的是以官方的身份来统一各种ORM框架的规范，包括著名的Hibernate、TopLink等。不过JPA规范给开发者带来了福音：开发者面向JPA规范的接口，但底层的JPA实现可以任意切换：觉得Hibernate好的，可以选择Hibernate JPA实现；觉得TopLink好的，可以选择TopLink JPA实现……这样开发者可以避免为使用Hibernate学习一套ORM框架，为使用TopLink又要再学习一套ORM框架。

下图是JPA和Hibernate、TopLink等ORM框架之间的关系：

![](/assets/import-jpa-01.png)



## Hibernate与JPA关系

JPA和Hibernate的关系就像JDBC和JDBC驱动的关系，JPA是规范，Hibernate除了作为ORM框架之外，它也是一种JPA实现。JPA怎么取代Hibernate呢？JDBC可以驱动JDBC驱动吗？

具体请参考知乎解答：https://www.zhihu.com/question/30691648

