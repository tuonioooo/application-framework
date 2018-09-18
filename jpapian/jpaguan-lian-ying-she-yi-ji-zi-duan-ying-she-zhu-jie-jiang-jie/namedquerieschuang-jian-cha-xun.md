# @NamedQueries创建查询

## 概述

命名查询是 JPA 提供的一种将查询语句从方法体中独立出来，以供多个方法共用的功能。Spring Data JPA 对命名查询也提供了很好的支持。用户只需要按照 JPA 规范在 orm.xml 文件或者在代码中使用 @NamedQuery（或 @NamedNativeQuery）定义好查询语句，唯一要做的就是为该语句命名时，需要满足”DomainClass.methodName\(\)”的 命名规则。

## 接口实现

```
public interface FindUserByNamedQueryRepository extends JpaRepository<User, Integer> {
    User findUserWithName(@Param("name") String name);
}
```

```
@Entity
@NamedQueries(value={
        @NamedQuery(name="User.findUserWithName",query="select u from User u where u.name = :name")
})
// 注意：此处如果是多个方法，那么需要使用@NamedQueries，如果只有一个方法，则可以使用@NamedQuery，写法如下：@NamedQuery(name="User.findUserWithName",query="select u from User u where u.name = :name")
public class FindUserByNamedQuery {
    /**
     * 注意：此处必须要给这个实体类定义一个唯一标识，否则会报异常
     */
    @Id
    @GeneratedValue
    private Integer id;
}
```

测试类

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:applicationContext-config.xml" })
@TransactionConfiguration(defaultRollback = false)
@Transactional
public class FindUserByNamedQueryRepositoryTest {
	@Autowired
	private FindUserByNamedQueryRepository dao;
	
	@Test
	public void testFindUserByName(){
		User user = dao.findUserWithName("caican");
		System.out.println(JSON.toJSONString(user));
	}
}

```

通过解析方法名创建查询

顾名思义，就是根据方法的名字，就能创建查询，也许初听起来，感觉很不可思议，等测试后才发现，原来一切皆有可能。

