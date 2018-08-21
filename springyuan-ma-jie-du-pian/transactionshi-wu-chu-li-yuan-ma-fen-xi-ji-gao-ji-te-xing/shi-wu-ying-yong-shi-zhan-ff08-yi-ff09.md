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



