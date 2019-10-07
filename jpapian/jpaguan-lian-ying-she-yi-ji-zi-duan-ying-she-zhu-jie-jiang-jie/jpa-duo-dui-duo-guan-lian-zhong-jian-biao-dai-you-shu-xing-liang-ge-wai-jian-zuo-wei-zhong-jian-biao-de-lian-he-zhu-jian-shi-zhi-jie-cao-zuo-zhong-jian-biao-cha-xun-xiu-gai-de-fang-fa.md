## JPA 多对多关联  中间表带有属性 两个外键作为中间表的联合主键时 直接操作中间表查询修改的方法

[https://blog.csdn.net/u013756305/article/details/83542758](https://blog.csdn.net/u013756305/article/details/83542758)



**1）、建立联合主键**

```
public class WorkDateTimeProductKey  implements Serializable{

       private static final long serialVersionUID = 3586335994284551414L;

       private Product product;

       private WorkDateTime workDateTime;

}
```

**2）、中间表的仓库**

```
public interface  WorkDateTimeProductRepository extends JpaRepository<WorkDateTimeProduct, Long> {

      WorkDateTimeProduct findByWorkDateTime_IdAndProduct_Id(long workDateTimeId ,long productId);

}

```

**3）、测试代码**

```
@SpringBootTest

@RunWith(SpringRunner.class)

public class WorkDateTimeProductRepositoryTest {

 

       @Autowired

       WorkDateTimeProductRepository WorkDateTimeProductRepository;

       

       @Test

       public void findByWorkDateTimeIdAndProductIdTest(){

               WorkDateTimeProduct  workDateTimeProduct =WorkDateTimeProductRepository.findByWorkDateTime_IdAndProduct_Id(12l, 13l);

               workDateTimeProduct.getAmount();

              System.out.println(workDateTimeProduct.getAmount());

       }

}
```

**WorkDateTime**

```


@Entity

//@JsonIgnoreProperties(value={"workDateTimeProducts"})

public class WorkDateTime implements Serializable {

      
       private static final long serialVersionUID = 6788933059497808914L;

       @Id

       @GeneratedValue(strategy=GenerationType.IDENTITY)

       private long id ;

       

       @JsonFormat(pattern="yyyy-MM-dd",timezone = "GMT+8")

       @DateTimeFormat(pattern = "yyyy-MM-dd")

       private Date dinnerDay;

       

       private byte dinnerTime; //  1noon 中餐，2 evening 晚餐,3 morning

       private byte state;  //订餐时间段是否有效 

       @OneToMany( mappedBy="workDateTime")

       private List<WorkDateTimeProduct> workDateTimeProducts = new ArrayList<WorkDateTimeProduct>();

       private byte printAllow;   //1为不可以打  2为可以打，

}

```

**Product **

```
@Entity

//@JsonIgnoreProperties(value = { "hibernateLazyInitializer", "handler" })

public class Product  implements Serializable{

       private static final long serialVersionUID = -3700731687896498304L;

       

       @Id

       @GeneratedValue(strategy=GenerationType.IDENTITY)

       private long id ;

       private String name;

       private String description;

       private short amount  ;

       private short price ;

       private String productImg;

       @OneToMany(mappedBy="product")

       List<WorkDateTimeProduct> workDateTimeProducts =new ArrayList<WorkDateTimeProduct> ();

}
```

**WorkDateTimeProduct**

```
@Entity

@IdClass(WorkDateTimeProductKey.class)

public class WorkDateTimeProduct implements Serializable{

       private static final long serialVersionUID = 1207408560047174539L;

 

       @Id

       @ManyToOne()

       @JoinColumn(name="product_id")

       private Product product;

       @Id

       @JsonIgnore

       @ManyToOne()

       @JoinColumn(name="workdatetime_id")

       private WorkDateTime workDateTime;

       private byte state;

     
       private short amount ;

 

}

```



