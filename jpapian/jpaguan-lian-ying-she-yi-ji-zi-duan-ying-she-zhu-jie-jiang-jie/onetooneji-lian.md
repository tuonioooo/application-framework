# OneToOne级联

## 注解讲解

@OneToOne\(fetch=FetchType,cascade=CascadeType\)

可选

@OneToOne描述一个一对一的关联

fetch:表示抓取策略，默认为FetchType.LAZY

cascade:表示级联操作策略

## 单向级联

@OneToOne 定义：一对一关系。

  生活中的一对一关系，举例：人（man） 和 宠物（pet）。前提（一人只养一个宠物）



  为什么这个一对一关系是单向的？如果，人养了宠物，那么我们通过“人”就能得到他所拥有的“宠物”的实体。但是，是不是通过“宠物”就能得到“人”的实体呢？！恐怕未必吧～因为在实际生活中，有很多走失的宠物，我们无法通过它们找到它们的主人。



 类似于上述这种情况，或者业务关系。实体间的关系是一对一，并且，我们只需要通过一个实体得到其对应的实体，而且并不需要反向执行这个操作的时候。我们就需要使用单向一对一关系。

例子：

```
@Entity
@Table(name = "people")
public class People (){

     @Id  //JPA注释： 主键
     @GeneratedValue(strategy = GenerationType.AUTO)   //设置 id 为自增长
     private Long id;

     private String name;

     //由于，people 是这个一对一的关系的主控方，所以，在people表中添加了一个 pet 的外键。
     //通过这个外键来维护 people和pet的一对一关系，而不是用第三张码表。这个是通过@JoinColumn注释实现的。
     @OneToOne //JPA注释： 一对一 关系
     @JoinColumn(name="pet_fk" )// 在pepole中，添加一个外键 "pet_fk"
     private Pet pet;

     //省略 get / set  方法...
}



@Entity
@Table(name = "pet")
public class Pet (){

     @Id  
     @GeneratedValue(strategy = GenerationType.AUTO)  
     private Long id;

     private String name;

     //省略 get / set  方法...
     //因为这是一个单向的一对一关系，并且，是从 people 到 pet 的一对一关系。
     //所以，在 pet 中没有与 people 管理的 属性。也就是说，无法通过 pet 找到 people
}

参考：http://blog.sina.com.cn/s/blog_625d79410101d8st.html
```

参考：

[http://blog.sina.com.cn/s/blog\_625d79410101d8st.html](http://blog.sina.com.cn/s/blog_625d79410101d8st.html)

## 双向关联

* **主Pojo**

```
@Entity
@Table(name = "T_ONEA")
public class OneA implements Serializable {

private static final long serialVersionUID = 1L;

@Id
@Column(name = "ONEA_ID", nullable = false)
private String oneaId;

@Column(name = "DESCRIPTION")
private String description;

@OneToOne(cascade = CascadeType.ALL, mappedBy = "oneA")//主Pojo这方的设置比较简单，只要设置好级联和映射到从Pojo的外键就可以了。
private OneB oneB;
```

* **从Pojo**

```
@Entity
@Table(name = "T_ONEB")
public class OneB implements Serializable {
private static final long serialVersionUID = 1L;

@Id
@Column(name = "ONEB_ID", nullable = false)
private String onebId;

@Column(name = "DESCRIPTION")
private String description;


//设置从方指向主方的关联外键，这个ONEA_ID其实是表T_ONEA的主键
@JoinColumn(name = "ONEA_ID", unique=true, referencedColumnName = "ONEA_ID", insertable=false)
@OneToOne
private OneA oneA;
```

* **持久化关联关系设置**

```
OneA oneA = new OneA();
OneB oneB = new OneB();
oneA.setOneB(oneB);
oneB.setOneA(oneA);
jpaRepository.save(oneA);//即可级联保存oneB
```



