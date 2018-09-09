# 自定义一个方言类——Hibernate Dialect

该类需要继承与我们使用的数据库相应的方言类。比如：如果我们用的是MySql（版本为5.x.x），我们需要继承“org.hibernate.dialect.MySQL5Dialect”；如果我们使用的是DB2，那么我们应该继承“org.hibernate.dialect.DB2Dialect”；我用的是SqlServer2008，所以我要继承“org.hibernate.dialect.SQLServerDialect”

```
import java.sql.Types;
import org.hibernate.Hibernate;
import org.hibernate.dialect.MySQL5Dialect;

/**
 * 重写MySQL5Dialect类，注册Types
 * @author daizhao
 *
 */

public class MyDialect extends MySQL5Dialect {

 public MyDialect(){

  super();

  registerHibernateType(Types.DECIMAL, Hibernate.BIG_DECIMAL.getName());

  registerHibernateType(Types.LONGVARCHAR,Hibernate.STRING.getName());

  registerHibernateType(Types.BINARY,Hibernate.STRING.getName());

  registerHibernateType(-1, Hibernate.STRING.getName());

 }

}
```

### **说明:**

如果你的数据库是mysql,而又用了decimal类型,报错应该是 No Dialect mapping for JDBC type: 3

注意这个3, 它说明hibernate不能将这种数据类型映射到你的java类中. 就需要在自定义的方言中用到:

**registerHibernateType\(Types.DECIMAL, Hibernate.BIG\_DECIMAL.getName\(\)\);**

如果你用了text数据类型,hibernate根本就不认识这种数据类型,所以会返回No Dialect mapping for JDBC type: -1

这样的话,就需要在方言中加入:

**registerHibernateType\(-1,Hibernate.STRING.getName\(\)\);**

**或**

**registerHibernateType\(Types.LONGVARCHAR, Hibernate.TEXT.getName\(\)\); **



```
import java.sql.Types;

import org.hibernate.Hibernate;
import org.hibernate.dialect.MySQL5InnoDBDialect;

//此处继承的是Hibernate MySql方言类，视不同开发情况而定
public class MySQLDialect extends MySQL5InnoDBDialect {

public MySQLDialect() { 
        super();
        //调用父类的registerHibernateType方法，注册Text类型
        registerHibernateType(Types.LONGVARCHAR, Hibernate.TEXT.getName()); 
    }
}
```

**映射配置文件**

```
<property name="hibernate.dialect">
    com.yourcompany.MyDialect
</property>
```

**Springboot属性配置例子**

```
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
spring.jpa.properties.hibernate.dialect=com.yourcompany.MyDialect
```



