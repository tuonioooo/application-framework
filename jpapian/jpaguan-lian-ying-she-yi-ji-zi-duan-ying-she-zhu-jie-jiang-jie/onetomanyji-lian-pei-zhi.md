# @OneToMany、@ManyToOne级联配置

**@OneToMany\(fetch=FetchType,cascade=CascadeType\)**

可选

@OneToMany描述一个一对多的关联,该属性应该为集体类型,在数据库中并没有实际字段.

fetch:表示抓取策略,默认为FetchType.LAZY,因为关联的多个对象通常不必从数据库预先读取到内存

cascade:表示级联操作策略,对于OneToMany类型的关联非常重要,通常该实体更新或删除时,其关联的实体也应当被更新或删除

例如:实体User和Order是OneToMany的关系,则实体User被删除时,其关联的实体Order也应该被全部删除

示例：

```
@OneTyMany(cascade=ALL)
public List getOrders() {
        return orders;
}
```

**@ManyToOne\(fetch=FetchType,cascade=CascadeType\)**

可选

@ManyToOne表示一个多对一的映射,该注解标注的属性通常是数据库表的外键

optional:是否允许该字段为null,该属性应该根据数据库表的外键约束来确定,默认为true

fetch:表示抓取策略,默认为FetchType.EAGER

cascade:表示默认的级联操作策略,可以指定为ALL、PERSIST、MERGE、REFRESH和REMOVE中的若干组合,默认为无级联操作

targetEntity:表示该属性关联的实体类型.该属性通常不必指定,ORM框架根据属性类型自动判断targetEntity.

示例：

```
//订单Order和用户User是一个ManyToOne的关系
//在Order类中定义
@ManyToOne
@JoinColumn(name=”USER”)
public User getUser() {
   return user;
}
```

**@JoinColumn**

可选

@JoinColumn和@Column类似,介量描述的不是一个简单字段,而一一个关联字段,例如.描述一个@ManyToOne的字段.

name:该字段的名称.由于@JoinColumn描述的是一个关联字段,如ManyToOne,则默认的名称由其关联的实体决定.

例如,实体Order有一个user属性来关联实体User,则Order的user属性为一个外键,

其默认的名称为实体User的名称+下划线+实体User的主键名称

示例：见@ManyToOne



