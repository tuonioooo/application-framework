# 注解关联时报错总结

1）

org.hibernate.MappingException: Could not determine type for: java.util.List, at table: task\_plan, for columns: \[org.hibernate.mapping.Column\(details\)\]

检出双向关联的注解\(@OneToMany、@ManyToOne、@OneToOne\)要么全都在属性上，要么都在 getter方法上！！，否则会报上面的错误

2）

```
//@Transient  //这个属性千万不要放到关联映射注解上，如果放到上面，你就悲剧了
@OneToMany(mappedBy = "taskPlanCascade", fetch = FetchType.EAGER)
private List<WorkManage> details = new ArrayList<WorkManage>();
```

> @Transient代表该方法不会被JPA映射管理，认为是普通方法



