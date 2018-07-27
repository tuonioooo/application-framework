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

如果开启MyBatis事务管理，则需要手动进行事务提交，否则事务会回滚到原状态;

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

> 如果在具体操作执行完后不通过sqlSession.commit\(\)方法提交事务，事务在sqlSession关闭时会自动回滚到原状态；只有执行了commit\(\)事务提交方法才会真正完成操作；
>
> 如果不执行sqlSession.commit\(\)操作，直接执行sqlSession.close\(\)，则会在close\(\)中进行事务回滚；
>
> 如果不执行sqlSession.commit\(\)操作也不手动关闭sqlSession，在程序结束时关闭数据库连接时会进行事务回滚；

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
session.commit();
session.close();
```

## 事务管理入口 {#1-事务管理入口}

在XML配置文件中定义事务工厂类型，JDBC或者MANAGED分别对应JdbcTransactionFactory.class和ManagedTransactionFactory.class;

> 如果type=”JDBC”则使用JdbcTransactionFactory事务工厂则MyBatis独立管理事务，直接使用JDK提供的JDBC来管理事务的各个环节：提交、回滚、关闭等操作；
>
> 如果type=”MANAGED”则使用ManagedTransactionFactory事务工厂则MyBatis不在ORM层管理事务而是将事务管理托付给其他框架，如Spring

## 事务对UPDATE操作的影响 {#2-事务对update操作的影响}

### 事务的提交 {#事务的提交}

在sqlSession中执行了UPDATE操作，需要执行sqlSession.commit\(\)方法提交事务，不然在连接关闭时候会自动回滚；

```
@Override
public void commit() {
  commit(false);
}

@Override
public void commit(boolean force) {
  try {
    executor.commit(isCommitOrRollbackRequired(force));
    dirty = false;
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error committing transaction.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

exector.commit\(\)事务提交方法归根到底是调用了transaction.commit\(\)事务的提交方法；这里的transaction就是根据配置对应的JdbcTransaction或者ManagedTransaction；

```
public void commit(boolean required) throws SQLException {
  if (closed) {
    throw new ExecutorException("Cannot commit, transaction is already closed");
  }
  clearLocalCache();
  flushStatements();
  if (required) {
    transaction.commit();
  }
}
```

如果是JdbcTransaction的commit\(\)方法，通过调用connection.commit\(\)方法通过数据库连接实现事务提交；

```
public void commit() throws SQLException {
  if (connection != null && !connection.getAutoCommit()) {
    if (log.isDebugEnabled()) {
      log.debug("Committing JDBC Connection [" + connection + "]");
    }
    connection.commit();
  }
}
```

如果是ManagedTransaction的commit\(\)方法，则为空方法不进行任何操作；

```
@Override
public void commit() throws SQLException {
  // Does nothing
}

@Override
public void rollback() throws SQLException {
  // Does nothing
}
```

### 事务的回滚 {#事务的回滚}

sqlSession执行close\(\)关闭操作时，如果close\(\)操作之前进行了UPDATE操作未进行commit\(\)事务提交则会进行事务回滚然后再关闭会话；如果update后执行了commit则直接关闭会话；

在DefaultSqlSession类中如果执行了UPDATE操作则会将标志位dirty赋值为true

```
@Override
public int update(String statement, Object parameter) {
  try {
    dirty = true;
    MappedStatement ms = configuration.getMappedStatement(statement);
    return executor.update(ms, wrapCollection(parameter));
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error updating database.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

在事务提交时会将dirty赋值为false;

```
@Override
public void commit(boolean force) {
  try {
    executor.commit(isCommitOrRollbackRequired(force));
    dirty = false;
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error committing transaction.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

在关闭会话时会判断dirty是否为true，如果为true则需要进行事务回滚操作，否则直接关闭会话

```
@Override
public void close() {
  try {
    executor.close(isCommitOrRollbackRequired(false));
    closeCursors();
    dirty = false;
  } finally {
    ErrorContext.instance().reset();
  }
}
```

判断dirty标志是否为true

```
private boolean isCommitOrRollbackRequired(boolean force) {
  return (!autoCommit && dirty) || force;
}
```

如果dirty为true则判定forceRollback为true，执行回滚操作；

```
@Override
public void close(boolean forceRollback) {
  try {
    try {
      rollback(forceRollback);
    } finally {
      if (transaction != null) {
        transaction.close();
      }
    }
  } catch (SQLException e) {
    // Ignore.  There's nothing that can be done at this point.
    log.warn("Unexpected exception on closing transaction.  Cause: " + e);
  } finally {
    transaction = null;
    deferredLoads = null;
    localCache = null;
    localOutputParameterCache = null;
    closed = true;
  }
}
```

## 事务对SELECT的影响 {#3-事务对select的影响}

事务对select操作的影响主要体现在对缓存的影响上，主要包括一级缓存和二级缓存

### 一级缓存 {#一级缓存}

因为一级缓存是Session级别的，事务的提交回滚对MyBatis的一级缓存没有影响；一级缓存放在BaseExector中的PerpectualCache类型的localCache中；

```
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
  if (closed) {
    throw new ExecutorException("Executor was closed.");
  }
  if (queryStack == 0 && ms.isFlushCacheRequired()) {
    clearLocalCache();
  }
  List<E> list;
  try {
    queryStack++;
    list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
    if (list != null) {
      handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
    } else {
      list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
    }
  } finally {
    queryStack--;
  }
  if (queryStack == 0) {
    for (DeferredLoad deferredLoad : deferredLoads) {
      deferredLoad.load();
    }
    // issue #601
    deferredLoads.clear();
    if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
      // issue #482
      clearLocalCache();
    }
  }
  return list;
}
```

### 二级缓存 {#二级缓存}

在Mapper.xml文件解析时会根据文件中的标签或者创建Cache实例，并将该实例放入每一个MappedStatement中，在MappedStatement执行select操作时候会获取该cache;

```
@Override
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
    throws SQLException {
  Cache cache = ms.getCache();
  if (cache != null) {
    flushCacheIfRequired(ms);
    if (ms.isUseCache() && resultHandler == null) {
      ensureNoOutParams(ms, parameterObject, boundSql);
      @SuppressWarnings("unchecked")
      List<E> list = (List<E>) tcm.getObject(cache, key);
      if (list == null) {
        list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
        tcm.putObject(cache, key, list); // issue #578 and #116
      }
      return list;
    }
  }
  return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

#### 查询二级缓存 {#查询二级缓存}

根据Statement所在的Mapper的cache缓存对象和根据statement生成的cacheKey，从正式缓存中取缓存数据

```
List<E> list = (List<E>) tcm.getObject(cache, key);
```

根据Statement所在的Mapper的cache缓存对象在TransactionManager中定位对应的TransactionCache，TransactionCache中保存这正式缓存delegate和临时未提交缓存entiryToAddOnCommit；

```
public Object getObject(Cache cache, CacheKey key) {
  return getTransactionalCache(cache).getObject(key);
}
```

获取缓存时直接从正式缓存delegate中查询

```
@Override
public Object getObject(Object key) {
  // issue #116
  Object object = delegate.getObject(key);
  if (object == null) {
    entriesMissedInCache.add(key);
  }
  // issue #146
  if (clearOnCommit) {
    return null;
  } else {
    return object;
  }
}
```

#### 存储二级缓存 {#存储二级缓存}

如果从二级缓存中未命中缓存，则需要从数据库中查取，再将查询结果放入二级缓存中；查询结果首先放入二级缓存临时缓存中，只有执行了commit\(\)事务提交才正式转移到正式缓存中；也就是说只有执行了commit\(\)方法的缓存才被下次查询使用，不然仍会执行数据库查询任务并覆盖上次的临时缓存；

根据Statement所在的Mapper的cache缓存对象在TransactionManager中定位对应的TransactionCache

```
public void putObject(Cache cache, CacheKey key, Object value) {
  getTransactionalCache(cache).putObject(key, value);
}
```

将数据放入TransactionCache中的临时缓存entiryToAddOnCommit中

```
public void putObject(Object key, Object object) {
  entriesToAddOnCommit.put(key, object);
}
```

#### 提交二级缓存 {#提交二级缓存}

执行事务提交时候会调用TransactionCacheManager内的commit\(\)方法提交缓存，每一个cachingExecutor对应一个TransactionCacheManager，也就是一个SqlSeesion对应一个TransactionCacheManager，因此sqlSession.commit\(\)是提交了当前会话的所以二级缓存的临时缓存；每个sqlSession对每一个Mapper的cache都有一个临时缓存，多个sqlSession共享一个Mapper的cache的正式缓存；

```
public void commit() {
  for (TransactionalCache txCache : transactionalCaches.values()) {
    txCache.commit();
  }
}
```

```
public void commit() {
  if (clearOnCommit) {
    delegate.clear();
  }
  flushPendingEntries();
  reset();
}
```

将临时缓存entiryToAddOnCommit中的数据转移到正式缓存delegate中

```
private void flushPendingEntries() {
  for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
    delegate.putObject(entry.getKey(), entry.getValue());
  }
  for (Object entry : entriesMissedInCache) {
    if (!entriesToAddOnCommit.containsKey(entry)) {
      delegate.putObject(entry, null);
    }
  }
}
```

#### 回滚二级缓存 {#回滚二级缓存}

* 如果之前未执行UPDATE操作，dirty标志为false，在关闭会话之前没有进行事务提交则进行数据库回滚和缓存提交；

* 如果执行了UPDATE操作，dirty标志为true，在关闭会话之前没有进行事务提交则进行数据库回滚和缓存回滚；

* 可以调用session.close\(true\)关闭会话时强行回滚缓存；

```
@Override
public void close(boolean forceRollback) {
  try {
    if (forceRollback) { 
      tcm.rollback();
    } else {
      tcm.commit();
    }
  } finally {
    delegate.close(forceRollback);
  }
}
```



