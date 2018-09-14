# @Basic、@Column、@Transient

**@Basic**

可选

@Basic表示一个简单的属性到数据库表的字段的映射,对于没有任何标注的getXxxx\(\)方法,默认即为@Basic

fetch: 表示该属性的读取策略,有EAGER和LAZY两种,分别表示主支抓取和延迟加载,默认为EAGER.

optional:表示该属性是否允许为null,默认为true

示例:

```
@Basic(optional=false)
public String getAddress() {
        return address;
}
```

**@Column**

可选

@Column描述了数据库表中该字段的详细定义,这对于根据JPA注解生成数据库表结构的工具非常有作用.

name:表示数据库表中该字段的名称,默认情形属性名称一致

nullable:表示该字段是否允许为null,默认为true

unique:表示该字段是否是唯一标识,默认为false

length:表示该字段的大小,仅对String类型的字段有效

insertable:表示在ORM框架执行插入操作时,该字段是否应出现INSETRT语句中,默认为true

updateable:表示在ORM框架执行更新操作时,该字段是否应该出现在UPDATE语句中,默认为true.对于一经创建就不可以更改的字段,该属性非常有用,如对于birthday字段.

columnDefinition: 表示该字段在数据库中的实际类型.通常ORM框架可以根据属性类型自动判断数据库中字段的类型,但是对于Date类型仍无法确定数据库中字段类型究竟是 DATE,TIME还是TIMESTAMP.此外,String的默认映射类型为VARCHAR,如果要将String类型映射到特定数据库的BLOB或 TEXT字段类型,该属性非常有用.

示例:

```
@Column(name="BIRTH", nullable="false", columnDefinition="DATE")
public String getBithday() {
        return birthday;
}
```

**@Transient**

可选

@Transient表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性.

如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic

示例:

```
//根据birth计算出age属性
@Transient
public int getAge() {
   return getYear(new Date()) – getYear(birth);
}
```



