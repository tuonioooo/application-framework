# @MappedSuperclass、@Embedded、@OrderBy、@Lob、@Data

**@MappedSuperclass**

可选

@MappedSuperclass可以将超类的JPA注解传递给子类,使子类能够继承超类的JPA注解

示例:

```
@MappedSuperclass
public class Employee() {
    ….
}
@Entity
public class Engineer extends Employee {
    …..
}
@Entity
public class Manager extends Employee {
    …..
}
```

**@Embedded**

可选

@Embedded将几个字段组合成一个类,并作为整个Entity的一个属性.

例如User包括id,name,city,street,zip属性.

我们希望city,street,zip属性映射为Address对象.这样,User对象将具有id,name和address这三个属性.

Address对象必须定义为@Embededable

示例:

```
@Embeddable
public class Address {city,street,zip}
@Entity
public class User {
    @Embedded
    public Address getAddress() {
        ……….
    }
}
```

> 总结：单独使用`@Embedded`或者只使用`@Embeddable`都会产生作用，那么这两个都使用效果也一定是一样的

详细示例：[https://blog.csdn.net/lmy86263/article/details/52108130](https://blog.csdn.net/lmy86263/article/details/52108130) 

**@OrderBy**

可选

在加载数据的时候可以为其指定顺序

示例：

```
@Table(name = "USERS")
public class User {

@OrderBy(name = "group_name ASC, name DESC")
private List books = new ArrayList();

}
```

**@Lob 大字段**

@Lob //对应Blob字段类型

@Column\(name = "PHOTO"\)

private Serializable photo;

@Lob //对应Blob字段类型

@Column\(name = "DESCRIPTION"\)

private String description;

