# JpaSpecificationExecutor接口

## 概述

封装 JPA Criteria 查询条件。通常使用匿名内部类的方式来创建该接口的对象

## 接口使用方式——Repository

```
public interface TaskPlanRepository extends JpaRepository<TaskPlan, String>, JpaSpecificationExecutor<TaskPlan> {

}
```

官方接口API：[https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaSpecificationExecutor.html](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaSpecificationExecutor.html)



## 复杂查询场景



