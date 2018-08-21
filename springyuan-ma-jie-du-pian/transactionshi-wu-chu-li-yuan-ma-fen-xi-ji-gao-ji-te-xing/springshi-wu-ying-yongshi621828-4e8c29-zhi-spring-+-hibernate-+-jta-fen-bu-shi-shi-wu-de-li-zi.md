# Spring事务应用实战（二）之spring+hibernate+JTA 分布式事务的例子

## **前言**

对于横跨多个Hibernate SessionFacotry的分布式事务，只需简单地将 JtaTransactionManager 同多个 LocalSessionFactoryBean 的定义结合起来作为事务策略。你的每一个DAO通过bean属性得到各自的 SessionFactory 引用。如果所有的底层JDBC数据源都是支持事务的容器，那么只要业务对象使用了 JtaTransactionManager 作为事务策略，它就可以横跨多个DAO和多个session factories来划分事务，而不需要做任何特殊处理。

## **示例配置一：**

```
 <beans>


  <bean id="myDataSource1" class="org.springframework.jndi.JndiObjectFactoryBean">

    <property name="jndiName" value="java:comp/env/jdbc/myds1"/>

  </bean>


  <bean id="myDataSource2" class="org.springframework.jndi.JndiObjectFactoryBean">

    <property name="jndiName" value="java:comp/env/jdbc/myds2"/>

  </bean>


  <bean id="mySessionFactory1" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">

    <property name="dataSource" ref="myDataSource1"/>

    <property name="mappingResources">

      <list>

        <value>product.hbm.xml</value>

      </list>

    </property>

    <property name="hibernateProperties">

      <value>

        hibernate.dialect=org.hibernate.dialect.MySQLDialect

        hibernate.show_sql=true

      </value>

    </property>

  </bean>


  <bean id="mySessionFactory2" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">

    <property name="dataSource" ref="myDataSource2"/>

    <property name="mappingResources">

      <list>

        <value>inventory.hbm.xml</value>

      </list>

    </property>

    <property name="hibernateProperties">

      <value>

        hibernate.dialect=org.hibernate.dialect.OracleDialect

      </value>

    </property>

  </bean>


  <bean id="myTxManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>


  <bean id="myProductDao" class="product.ProductDaoImpl">

    <property name="sessionFactory" ref="mySessionFactory1"/>

  </bean>


  <bean id="myInventoryDao" class="product.InventoryDaoImpl">

    <property name="sessionFactory" ref="mySessionFactory2"/>

  </bean>


  <!-- this shows the Spring 1.x style of declarative transaction configuration -->

  <!-- it is totally supported, 100% legal in Spring 2.x, but see also above for the sleeker, Spring 2.0 style -->

  <bean id="myProductService"

      class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">

    <property name="transactionManager" ref="myTxManager"/>

    <property name="target">

      <bean class="product.ProductServiceImpl">

        <property name="productDao" ref="myProductDao"/>

        <property name="inventoryDao" ref="myInventoryDao"/>

      </bean>

    </property>

    <property name="transactionAttributes">

      <props>

        <prop key="increasePrice*">PROPAGATION_REQUIRED</prop>

        <prop key="someOtherBusinessMethod">PROPAGATION_REQUIRES_NEW</prop>

        <prop key="*">PROPAGATION_SUPPORTS,readOnly</prop>

      </props>

    </property>

  </bean>


</beans>  

```

## **示例配置二：**

```
<bean id="myDataSource1" class="org.springframework.jndi.JndiObjectFactoryBean">    
   <property name="jndiName" value="java:comp/env/jdbc/myds1"/>    
 </bean>    
   
 <bean id="myDataSource2" class="org.springframework.jndi.JndiObjectFactoryBean">    
   <property name="jndiName" value="java:comp/env/jdbc/myds2"/>    
 </bean>    
   
 <bean id="mySessionFactory1" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">    
   <property name="dataSource" ref="myDataSource1"/>    
   <property name="mappingResources">    
     <list>    
       <value>product.hbm.xml</value>    
     </list>    
   </property>    
   <property name="hibernateProperties">    
     <value>    
       hibernate.dialect=org.hibernate.dialect.MySQLDialect     
       hibernate.show_sql=true    
     </value>    
   </property>    
 </bean>    
   
 <bean id="mySessionFactory2" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">    
   <property name="dataSource" ref="myDataSource2"/>    
   <property name="mappingResources">    
     <list>    
       <value>inventory.hbm.xml</value>    
     </list>    
   </property>    
   <property name="hibernateProperties">    
     <value>    
       hibernate.dialect=org.hibernate.dialect.OracleDialect     
     </value>    
   </property>    
 </bean>    
   
 <bean id="myTxManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>    
   
 <bean id="myProductDao" class="product.ProductDaoImpl">    
   <property name="sessionFactory" ref="mySessionFactory1"/>    
 </bean>    
   
 <bean id="myInventoryDao" class="product.InventoryDaoImpl">    
   <property name="sessionFactory" ref="mySessionFactory2"/>    
 </bean>    
   
  <bean id="myProductService" class="product.service.myProductServiceImpl">    
   <property name="productDao" ref="myProductDao"/>    
   <property name="inventoryDao" ref="myInventoryDao"/>    
 </bean>    
  
 <aop:config>  
    <aop:pointcut id="managerTx"  
        expression="execution(* product.service..*Service.*(..))" />  
    <aop:advisor advice-ref="txAdvice" pointcut-ref="managerTx" />  
</aop:config>  
  
<tx:advice id="txAdvice" transaction-manager="myTxManager">  
    <tx:attributes>  
        <tx:method name="get*" read-only="true" />  
        <tx:method name="find*" read-only="true" />  
        <tx:method name="save*" propagation="REQUIRED" />  
        <tx:method name="remove*" propagation="REQUIRED" />  
        <tx:method name="*" />  
    </tx:attributes>  
</tx:advice>
```



