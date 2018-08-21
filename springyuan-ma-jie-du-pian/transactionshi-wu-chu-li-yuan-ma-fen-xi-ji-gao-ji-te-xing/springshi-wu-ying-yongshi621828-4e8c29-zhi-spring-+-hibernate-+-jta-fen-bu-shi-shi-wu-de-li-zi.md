# Spring事务应用实战（二）之spring+hibernate+JTA 分布式事务的例子

## **前言**

对于横跨多个Hibernate SessionFacotry的分布式事务，只需简单地将 JtaTransactionManager 同多个 LocalSessionFactoryBean 的定义结合起来作为事务策略。你的每一个DAO通过bean属性得到各自的 SessionFactory 引用。如果所有的底层JDBC数据源都是支持事务的容器，那么只要业务对象使用了 JtaTransactionManager 作为事务策略，它就可以横跨多个DAO和多个session factories来划分事务，而不需要做任何特殊处理。

## **示例配置：**



