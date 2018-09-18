# @Query 创建查询

## 概述

@Query 注解的使用非常简单，只需在声明的方法上面标注该注解，同时提供一个 JP QL 查询语句即可。很多开发者在创建 JP QL 时喜欢使用命名参数来代替位置编号，@Query 也对此提供了支持。JP QL 语句中通过": 变量"的格式来指定参数，同时在方法的参数前面使用 @Param 将方法参数与 JP QL 中的命名参数对应。此外，开发者也可以通过使用 @Query 来执行一个更新操作，为此，我们需要在使用 @Query 的同时，用 @Modifying 来将该操作标识为修改查询，这样框架最终会生成一个更新的操作，而非查询操作。

编写接口，如下：

```
/**
 * 描述：自定义查询，当Spring Data JPA无法提供时，需要自定义接口，此时可以使用这种方式
 */
public interface UserDefineBySelf extends JpaRepository<User, Integer> {
	/**
	 * 命名参数
	 * 描述：推荐使用这种方法，可以不用管参数的位置
	 */
	@Query("select u from User u where u.name = :name")
	User findUserByName(@Param("name") String name);
	
	/**
	 * 索引参数
	 * 描述：使用?占位符
	 */
	@Query("select u from User u where u.email = ?1")// 1表示第一个参数
	User findUserByEmail(String email);
	
	/**
	 * 描述：可以通过@Modifying和@Query来实现更新
	 * 注意：Modifying queries的返回值只能为void或者是int/Integer
	 */
	@Modifying
	@Query("update User u set u.name = :name where u.id = :id")
	int updateUserById(@Param("name") String name, @Param("id") int id);
}

```

> 注：@Modifying注解里面有一个配置clearAutomatically

> 它说的是可以清除底层持久化上下文，就是entityManager这个类，我们知道jpa底层实现会有二级缓存，也就是在更新完数据库后，如果后面去用这个对象，你再去查这个对象，这个对象是在二级缓存，但是并没有跟数据库同步，这个时候用clearAutomatically=true,就会刷新hibernate的二级缓存了， 不然你在同一接口中，更新一个对象，接着查询这个对象，那么你查出来的这个对象还是之前的没有更新之前的状态

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:applicationContext-config.xml" })
@TransactionConfiguration(defaultRollback = false)
@Transactional
public class UserDefineBySelfTest {
	@Autowired
	private UserDefineBySelf dao;
	
	@Test
	public void testFindUserByName(){
		User user = dao.findUserByName("chhliu");
		Assert.assertEquals("chhliu", user.getName());
		System.out.println(user.getName());
	}
	
	@Test
	public void testFindUserByEmail(){
		User user = dao.findUserByEmail("chhliu@.com");
		Assert.assertEquals("chhliu", user.getName());
		System.out.println(user.getName());
	}
	
	@Test
	public void testUpdateUserById(){
		dao.updateUserById("tanjie", 4);
	}
}

```



