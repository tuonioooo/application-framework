# 事物应用实战（一）

**步骤一、在spring配置文件中引入&lt;tx:&gt;命名空间**

```
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns:tx="http://www.springframework.org/schema/tx"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
 http://www.springframework.org/schema/tx
 http://www.springframework.org/schema/tx/spring-tx-2.0.xsd">
```

**步骤二、具有@Transactional 注解的bean自动配置为声明式事务支持 **

```
<!-- 事务管理器配置, Hibernate单数据源事务 -->
<bean id="defaultTransactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
<property name="sessionFactory" ref="sessionFactory" />
</bean>
<!-- 使用annotation定义事务 -->
<tx:annotation-driven transaction-manager="defaultTransactionManager" proxy-target-class="true" />
```



