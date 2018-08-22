# 事务实现源码分析

### **TransactionDefinition **

* 事务的定义接口，包含事务的两个重要属性：传播特性和隔离级别

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

* **结构图**

![](blob:file:///094d12d2-743f-4343-8287-8388252fa641)

* **TransactionDefinition 的一个实现类：DefaultTransactionDefinition**

就是对上述属性设置一些默认值，默认的传播特性为PROPAGATION\_REQUIRED ，隔离级别为ISOLATION\_DEFAULT 

```
public class DefaultTransactionDefinition implements TransactionDefinition, Serializable {
    public static final String PREFIX_PROPAGATION = "PROPAGATION_";
    public static final String PREFIX_ISOLATION = "ISOLATION_";
    public static final String PREFIX_TIMEOUT = "timeout_";
    public static final String READ_ONLY_MARKER = "readOnly";
    static final Constants constants = new Constants(TransactionDefinition.class);
    private int propagationBehavior = 0;//默认值
    private int isolationLevel = -1;
    private int timeout = -1;
    private boolean readOnly = false;
    private String name;

    public DefaultTransactionDefinition() {
    }

    public DefaultTransactionDefinition(TransactionDefinition other) {
        this.propagationBehavior = other.getPropagationBehavior();
        this.isolationLevel = other.getIsolationLevel();
        this.timeout = other.getTimeout();
        this.readOnly = other.isReadOnly();
        this.name = other.getName();
    }

    public DefaultTransactionDefinition(int propagationBehavior) {
        this.propagationBehavior = propagationBehavior;
    }

    public final void setPropagationBehaviorName(String constantName) throws IllegalArgumentException {
        if(constantName != null && constantName.startsWith("PROPAGATION_")) {
            this.setPropagationBehavior(constants.asNumber(constantName).intValue());
        } else {
            throw new IllegalArgumentException("Only propagation constants allowed");
        }
    }

    public final void setPropagationBehavior(int propagationBehavior) {
        if(!constants.getValues("PROPAGATION_").contains(Integer.valueOf(propagationBehavior))) {
            throw new IllegalArgumentException("Only values of propagation constants allowed");
        } else {
            this.propagationBehavior = propagationBehavior;
        }
    }

    public final int getPropagationBehavior() {
        return this.propagationBehavior;
    }

    public final void setIsolationLevelName(String constantName) throws IllegalArgumentException {
        if(constantName != null && constantName.startsWith("ISOLATION_")) {
            this.setIsolationLevel(constants.asNumber(constantName).intValue());
        } else {
            throw new IllegalArgumentException("Only isolation constants allowed");
        }
    }

    public final void setIsolationLevel(int isolationLevel) {
        if(!constants.getValues("ISOLATION_").contains(Integer.valueOf(isolationLevel))) {
            throw new IllegalArgumentException("Only values of isolation constants allowed");
        } else {
            this.isolationLevel = isolationLevel;
        }
    }

    public final int getIsolationLevel() {
        return this.isolationLevel;
    }

    public final void setTimeout(int timeout) {
        if(timeout < -1) {
            throw new IllegalArgumentException("Timeout must be a positive integer or TIMEOUT_DEFAULT");
        } else {
            this.timeout = timeout;
        }
    }

    public final int getTimeout() {
        return this.timeout;
    }

    public final void setReadOnly(boolean readOnly) {
        this.readOnly = readOnly;
    }

    public final boolean isReadOnly() {
        return this.readOnly;
    }

    public final void setName(String name) {
        this.name = name;
    }

    public final String getName() {
        return this.name;
    }

    public boolean equals(Object other) {
        return other instanceof TransactionDefinition && this.toString().equals(other.toString());
    }

    public int hashCode() {
        return this.toString().hashCode();
    }

    public String toString() {
        return this.getDefinitionDescription().toString();
    }

    protected final StringBuilder getDefinitionDescription() {
        StringBuilder result = new StringBuilder();
        result.append(constants.toCode(Integer.valueOf(this.propagationBehavior), "PROPAGATION_"));
        result.append(',');
        result.append(constants.toCode(Integer.valueOf(this.isolationLevel), "ISOLATION_"));
        if(this.timeout != -1) {
            result.append(',');
            result.append("timeout_").append(this.timeout);
        }

        if(this.readOnly) {
            result.append(',');
            result.append("readOnly");
        }

        return result;
    }
}

```

* **TransactionAttribute接口**

```
public interface TransactionAttribute extends TransactionDefinition {
    String getQualifier();

    boolean rollbackOn(Throwable var1);
}
```

> 定义对什么类型的异常进行回滚

* ** TransactionAttribute的实现类：DefaultTransactionAttribute**

```
public class DefaultTransactionAttribute extends DefaultTransactionDefinition implements TransactionAttribute {
    private String qualifier;

    public DefaultTransactionAttribute() {
    }

    public DefaultTransactionAttribute(TransactionAttribute other) {
        super(other);
    }

    public DefaultTransactionAttribute(int propagationBehavior) {
        super(propagationBehavior);
    }

    public void setQualifier(String qualifier) {
        this.qualifier = qualifier;
    }

    public String getQualifier() {
        return this.qualifier;
    }

    public boolean rollbackOn(Throwable ex) {//指明了对RuntimeException 和Error进行回滚
        return ex instanceof RuntimeException || ex instanceof Error;
    }

    protected final StringBuilder getAttributeDescription() {
        StringBuilder result = this.getDefinitionDescription();
        if(this.qualifier != null) {
            result.append("; '").append(this.qualifier).append("'");
        }

        return result;
    }
}
```

* **事务模板类TransactionTemplate**

```
public class TransactionTemplate extends DefaultTransactionDefinition implements TransactionOperations, InitializingBean {
    protected final Log logger = LogFactory.getLog(this.getClass());
    private PlatformTransactionManager transactionManager;

    public TransactionTemplate() {
    }

    public TransactionTemplate(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    public TransactionTemplate(PlatformTransactionManager transactionManager, TransactionDefinition transactionDefinition) {
        super(transactionDefinition);
        this.transactionManager = transactionManager;
    }

    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    public PlatformTransactionManager getTransactionManager() {
        return this.transactionManager;
    }

    public void afterPropertiesSet() {
        if (this.transactionManager == null) {
            throw new IllegalArgumentException("Property 'transactionManager' is required");
        }
    }

    public <T> T execute(TransactionCallback<T> action) throws TransactionException {
        if (this.transactionManager instanceof CallbackPreferringPlatformTransactionManager) {
            return ((CallbackPreferringPlatformTransactionManager)this.transactionManager).execute(this, action);
        } else {
            TransactionStatus status = this.transactionManager.getTransaction(this);

            Object result;
            try {
                result = action.doInTransaction(status);
            } catch (RuntimeException var5) {
                this.rollbackOnException(status, var5);
                throw var5;
            } catch (Error var6) {
                this.rollbackOnException(status, var6);
                throw var6;
            } catch (Throwable var7) {
                this.rollbackOnException(status, var7);
                throw new UndeclaredThrowableException(var7, "TransactionCallback threw undeclared checked exception");
            }

            this.transactionManager.commit(status);
            return result;
        }
    }

    private void rollbackOnException(TransactionStatus status, Throwable ex) throws TransactionException {
        this.logger.debug("Initiating transaction rollback on application exception", ex);

        try {
            this.transactionManager.rollback(status);
        } catch (TransactionSystemException var4) {
            this.logger.error("Application exception overridden by rollback exception", ex);
            var4.initApplicationException(ex);
            throw var4;
        } catch (RuntimeException var5) {
            this.logger.error("Application exception overridden by rollback exception", ex);
            throw var5;
        } catch (Error var6) {
            this.logger.error("Application exception overridden by rollback error", ex);
            throw var6;
        }
    }
}
```

> 他的核心是里面有PlatformTransactionManager 这个事务管理类，用它来对事务提交和回滚。我们的业务逻辑只要写在TransactionCallback.doInTransaction（）方法里面既可以，每次执行这个方法前，先会transactionManager.getTransaction\(this\)开启一   个事务，执行TransactionCallback.doInTransaction（）异常的话会调用transactionManager.rollback\(status\)来回滚事务，正确的话就会调用transactionManager.commit\(status\)提交事务；

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



