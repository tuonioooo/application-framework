# 更新的同时删除多的一方的旧数据

```
public class ApplyObject {
    @OneToMany(cascade=CascadeType.ALL,mappedBy="applyObject",orphanRemoval = true)
    private List<ApplyObjectList> applyObjectObject;//物品列表
}
```

```
public class ApplyObjectList {
    @JoinColumn(name="aoo_ao_id",nullable=false)
    @ManyToOne(cascade=CascadeType.PERSIST,fetch=FetchType.LAZY)
    private ApplyObject applyObject;//物品申请表
}
```

> 关键的就是在OneToMany的一方加上`orphanRemoval = true`，这样在更新的时候这个集合原来的旧数据才会被删除再重新添加新数据。



