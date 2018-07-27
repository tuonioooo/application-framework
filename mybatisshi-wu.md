# Mybatis事务

## 概述

### **事务管理器（transactionManager）**

在 MyBatis 中有两种类型的事务管理器（也就是 type=”\[JDBC\|MANAGED\]”）：

* JDBC – 这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域。
* MANAGED – 这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。例如:
  ```
  <transactionManager type="MANAGED">
    <property name="closeConnection" value="false"/>
  </transactionManager>
  ```

> 提示如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器， 因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

这两种事务管理器类型都不需要任何属性。它们不过是类型别名，换句话说，你可以使用 TransactionFactory 接口的实现类的完全限定名或类型别名代替它们。

```
public interface TransactionFactory {
  void setProperties(Properties props);  
  Transaction newTransaction(Connection conn);
  Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit);  
}
```

任何在 XML 中配置的属性在实例化之后将会被传递给 setProperties\(\) 方法。你也需要创建一个 Transaction 接口的实现类，这个接口也很简单：

```
public interface Transaction {
  Connection getConnection() throws SQLException;
  void commit() throws SQLException;
  void rollback() throws SQLException;
  void close() throws SQLException;
  Integer getTimeout() throws SQLException;
}
```

使用这两个接口，你可以完全自定义 MyBatis 对事务的处理。



## MyBatis事务遇到的问题 {#0-mybatis事务遇到的问题}

* 如果开启MyBatis事务管理，则需要手动进行事务提交，否则事务会回滚到原状态;

```
String resource = "mybatis/config.xml";
InputStream is = Main.class.getClassLoader().getResourceAsStream(resource);
SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
SqlSession session = sessionFactory.openSession();

User user = new User();
user.setName("liuliu");
user.setPassword("123123");
user.setScore("88");
String statement = "mybatis.mapping.UserMapper.insertUser";
session.insert(statement,user);
session.close();

```

* 如果在具体操作执行完后不通过sqlSession.commit\(\)方法提交事务，事务在sqlSession关闭时会自动回滚到原状态；只有执行了commit\(\)事务提交方法才会真正完成操作；
* 如果不执行sqlSession.commit\(\)操作，直接执行sqlSession.close\(\)，则会在close\(\)中进行事务回滚；
* 如果不执行sqlSession.commit\(\)操作也不手动关闭sqlSession，在程序结束时关闭数据库连接时会进行事务回滚；





