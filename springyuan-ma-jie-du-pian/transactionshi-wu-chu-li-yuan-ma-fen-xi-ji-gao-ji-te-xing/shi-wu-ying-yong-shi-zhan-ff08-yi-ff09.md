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

**步骤三、在接口或类的声明处 ,写一个@Transactional.**

要是只在接口上写, 接口的实现类就会继承下来、接口的实现类的具体方法,可以覆盖类声明处的设置

@Transactional   //类级的注解、适用于类中所有的public的方法

**事务的传播行为和隔离级别**

大家在使用spring的注解式事务管理时，对事务的传播行为和隔离级别可能有点不知所措，下边就详细的介绍下以备方便查阅。

事物注解方式: @Transactional

当标于类前时, 标示类中所有方法都进行事物处理 , 例子:

```
@Transactional
public class TestServiceBean implements TestService {}
```

当类中某些方法不需要事物时:

```
@Transactional
public class TestServiceBean implements TestService {
private TestDao dao;
public void setDao(TestDao dao) {
this.dao = dao;
}
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public List<Object> getAll() {
return null;
}
}
```

**事物传播行为介绍:**

* @Transactional\(propagation=Propagation.REQUIRED\) 如果有事务, 那么加入事务, 没有的话新建一个\(默认情况下\)
* @Transactional\(propagation=Propagation.NOT\_SUPPORTED\) 容器不为这个方法开启事务
* @Transactional\(propagation=Propagation.REQUIRES\_NEW\) 不管是否存在事务,都创建一个新的事务,原来的挂起,新的执行完毕,继续执行老的事务
* @Transactional\(propagation=Propagation.MANDATORY\) 必须在一个已有的事务中执行,否则抛出异常
* @Transactional\(propagation=Propagation.NEVER\) 必须在一个没有的事务中执行,否则抛出异常\(与Propagation.MANDATORY相反\)
* @Transactional\(propagation=Propagation.SUPPORTS\) 如果其他bean调用这个方法,在其他bean中声明事务,那就用事务.如果其他bean没有声明事务,那就不用事务.

**事物超时设置:**

@Transactional\(timeout=30\) //默认是30秒

**事务隔离级别:**

@Transactional\(isolation = Isolation.READ\_UNCOMMITTED\)

读取未提交数据\(会出现脏读, 不可重复读\) 基本不使用

@Transactional\(isolation = Isolation.READ\_COMMITTED\)

读取已提交数据\(会出现不可重复读和幻读\)

@Transactional\(isolation = Isolation.REPEATABLE\_READ\)

可重复读\(会出现幻读\)

@Transactional\(isolation = Isolation.SERIALIZABLE\)

串行化

MYSQL: 默认为REPEATABLE\_READ级别

SQLSERVER: 默认为READ\_COMMITTED

脏读 : 一个事务读取到另一事务未提交的更新数据

不可重复读 : 在同一事务中, 多次读取同一数据返回的结果有所不同, 换句话说,

后续读取可以读到另一事务已提交的更新数据. 相反, "可重复读"在同一事务中多次

读取数据时, 能够保证所读数据一样, 也就是后续读取不能读到另一事务已提交的更新数据

幻读 : 一个事务读到另一个事务已提交的insert数据

**@Transactional注解中常用参数说明**

| 参 数 名 称 | 功 能 描 述 |
| :--- | :--- |
| readOnly | 该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false。例如：@Transactional\(readOnly=true\) |
| rollbackFor | 该属性用于设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。例如：指定单一异常类：@Transactional\(rollbackFor=RuntimeException.class\)指定多个异常类：@Transactional\(rollbackFor={RuntimeException.class, Exception.class}\) |

续表）

| 参 数 名 称 | 功 能 描 述 |
| :--- | :--- |
| rollbackForClassName | 该属性用于设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚。例如：指定单一异常类名称：@Transactional\(rollbackForClassName="RuntimeException"\)指定多个异常类名称：@Transactional\(rollbackForClassName={"RuntimeException","Exception"}\) |
| noRollbackFor | 该属性用于设置不需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，不进行事务回滚。例如：指定单一异常类：@Transactional\(noRollbackFor=RuntimeException.class\)指定多个异常类：@Transactional\(noRollbackFor={RuntimeException.class, Exception.class}\) |
| noRollbackForClassName | 该属性用于设置不需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，不进行事务回滚。例如：指定单一异常类名称：@Transactional\(noRollbackForClassName="RuntimeException"\)指定多个异常类名称：@Transactional\(noRollbackForClassName={"RuntimeException","Exception"}\) |
| propagation | 该属性用于设置事务的传播行为，具体取值可参考表6-7。例如：@Transactional\(propagation=Propagation.NOT\_SUPPORTED,readOnly=true\) |
| isolation | 该属性用于设置底层数据库的事务隔离级别，事务隔离级别用于处理多事务并发的情况，通常使用数据库的默认隔离级别即可，基本不需要进行设置 |
| timeout | 该属性用于设置事务的超时秒数，默认值为-1表示永不超时 |



