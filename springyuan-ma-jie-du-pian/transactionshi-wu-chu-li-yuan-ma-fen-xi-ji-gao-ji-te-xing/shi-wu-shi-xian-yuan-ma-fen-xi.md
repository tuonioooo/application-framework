# 事务实现源码分析

## **PlatformTransactionManager**

PlatformTransactionManager是spring事务的核心接口。 结构图如下：

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



