# 事务实现源码分析

### **TransactionDefinition **

事务的定义接口，包含事务的两个重要属性：传播特性和隔离级别

```
public interface TransactionDefinition {
    int PROPAGATION_REQUIRED = 0;
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
    int ISOLATION_DEFAULT = -1;
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
    int TIMEOUT_DEFAULT = -1;

    int getPropagationBehavior();

    int getIsolationLevel();

    int getTimeout();

    boolean isReadOnly();

    String getName();
}
```

### **PlatformTransactionManager**

PlatformTransactionManager是spring事务的核心接口。 结构图如下：

![](/assets/import-pl-01.png)

![](/assets/pl2.png)

接口如下：

```
public interface PlatformTransactionManager {
    TransactionStatus getTransaction(TransactionDefinition var1) throws TransactionException;

    void commit(TransactionStatus var1) throws TransactionException;

    void rollback(TransactionStatus var1) throws TransactionException;
}
```

> 用它来对事务提交和回滚。我们的业务逻辑只要写在TransactionCallback.doInTransaction（）方法里面既可以，每次执行这个方法前，先会transactionManager.getTransaction\(this\)开启一   个事务，执行TransactionCallback.doInTransaction（）异常的话会调用transactionManager.rollback\(status\)来回滚事务，正确的话就会调用transactionManager.commit\(status\)提交事务；



