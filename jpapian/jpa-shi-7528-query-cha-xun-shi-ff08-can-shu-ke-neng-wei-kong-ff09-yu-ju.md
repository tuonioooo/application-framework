# Jpa 使用@Query查询时 （参数可能为空）语句

```
@Query(value = "select * from xxx where if(?1 !='',x1=?1,1=1) and if(?2 !='',x2=?2,1=1)" +
            "and if(?3 !='',x3=?3,1=1)  ",nativeQuery = true)
     List<XXX> find(String X1,String X2,String X3);
```

工作的时候需求有搜索功能，有三个参数，但是网上找了很多关于jpa多条件查询的代码要么在调dao的时候用了大量if判断，那么需要写很多查询方法，要么说的不知所云，我结合jpa和mysql原语句研究了半天才弄出了这个方法。



xxx是数据库表名，x1、x2、x3为查询的字段名。



下面的大写的XXX是实体类的名，X1X2X3为查询的参数。



if\(?1 !='',x1=?1,1=1\) 代表传入的参数X1如果不为""（Spring类型空是""而不是null）将参数传入x1，如果为空时显示1=1 代表参数为真，对查询结果不产生作用。



