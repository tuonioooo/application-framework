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

通过解析方法名创建查询。顾名思义，就是根据方法的名字，就能创建查询，也许初听起来，感觉很不可思议，等测试后才发现，原来一切皆有可能。

接口实现

```
public interface SimpleConditionQueryRepository extends JpaRepository<User, Integer> {
	/**
	 * 说明：按照Spring data 定义的规则，查询方法以find|read|get开头
     * 涉及条件查询时，条件的属性用条件关键字连接，要注意的是：条件属性首字母需大写
	 */
	
	
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.name = :name and u.email = :email
	 * 参数名大写，条件名首字母大写，并且接口名中参数出现的顺序必须和参数列表中的参数顺序一致
	 */
	User findByNameAndEmail(String name, String email);
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.name = ?1 or u.password = ?2
	 */
	List<User> findByNameOrPassword(String name, String password);
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.id between ?1 and ?2
	 */
	List<User> findByIdBetween(Integer start, Integer end);
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.id < ?1
	 */
	List<User> findByIdLessThan(Integer end);
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.id > ?1
	 */
	List<User> findByIdGreaterThan(Integer start);
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.name is null
	 */
	List<User> findByNameIsNull();
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.name is not null
	 */
	List<User> findByNameIsNotNull();
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.name like ?1
	 */
	List<User> findByNameLike(String name);
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.name not like ?1
	 */
	List<User> findByNameNotLike(String name);
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.password = ?1 order by u.id desc
	 */
	List<User> findByPasswordOrderByIdDesc(String password);
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.name <> ?1
	 */
	List<User> findByNameNot(String name);
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.id in ?1
	 */
	List<User> findByIdIn(List<Integer> ids);
	
	/**
	 * 注：此处这个接口相当于发送了一条SQL:select u from User u where u.id not in ?1
	 */
	List<User> findByIdNotIn(List<Integer> ids);
}

```



