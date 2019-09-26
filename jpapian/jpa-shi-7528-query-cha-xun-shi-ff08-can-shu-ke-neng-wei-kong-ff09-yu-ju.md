# Jpa 使用@Query查询时 （参数可能为空）语句

```
@Query(value = "select * from xxx where if(?1 !='',x1=?1,1=1) and if(?2 !='',x2=?2,1=1)" +
            "and if(?3 !='',x3=?3,1=1)  ",nativeQuery = true)
     List<XXX> find(String X1,String X2,String X3);

```



