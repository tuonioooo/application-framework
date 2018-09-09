# Hibernate 不同数据库的连接及SQL方言

如果出现如下错误，则可能是Hibernate SQL方言 \(hibernate.dialect\)设置不正确。

Caused by: java.sql.SQLException: \[Microsoft\]\[SQLServer 2000 Driver for JDBC\]\[SQLServer\]'last\_insert\_id' 不是可以识别的 函数名。



```
  <!--MySql 驱动程序 eg. mysql-connector-java-5.0.4-bin.jar-->  
  <property name="dialect">org.hibernate.dialect.MySQLDialect</property> 
  <property name="connection.driver_class">com.mysql.jdbc.Driver</property>

  <!-- JDBC URL -->
  <property name="connection.url">jdbc:mysql://localhost/dbname?characterEncoding=gb2312</property>

  <!-- 数据库用户名-->
  <property name="connection.username">root</property>

  <!-- 数据库密码-->
  <property name="connection.password">root</property>
  
  
  <!--Sql Server 驱动程序 eg. jtds-1.2.jar-->
  <property name="dialect">org.hibernate.dialect.SQLServerDialect</property>
  <property name="connection.driver_class">net.sourceforge.jtds.jdbc.Driver</property>

  <!-- JDBC URL -->
  <property name="connection.url">jdbc:jtds:sqlserver://localhost:1433;DatabaseName=dbname</property>

  <!-- 数据库用户名-->
  <property name="connection.username">sa</property>

  <!-- 数据库密码-->
  <property name="connection.password"></property>
```

**Hibernate 不同数据库的连接及SQL方言**  


| RDBMS | 方言 |
| :--- | :--- |
| DB2 | org.hibernate.dialect.DB2Dialect |
| DB2 AS/400 | org.hibernate.dialect.DB2400Dialect |
| DB2 OS390 | org.hibernate.dialect.DB2390Dialect |
| PostgreSQL | org.hibernate.dialect.PostgreSQLDialect |
| MySQL | org.hibernate.dialect.MySQLDialect |
| MySQL with InnoDB | org.hibernate.dialect.MySQLInnoDBDialect |
| MySQL with MyISAM | org.hibernate.dialect.MySQLMyISAMDialect |
| Oracle \(any version\) | org.hibernate.dialect.OracleDialect |
| Oracle 9i/10g | org.hibernate.dialect.Oracle9Dialect |
| Sybase | org.hibernate.dialect.SybaseDialect |
| Sybase Anywhere | org.hibernate.dialect.SybaseAnywhereDialect |
| Microsoft SQL Server | org.hibernate.dialect.SQLServerDialect |
| SAP DB | org.hibernate.dialect.SAPDBDialect |
| Informix | org.hibernate.dialect.InformixDialect |
| HypersonicSQL | org.hibernate.dialect.HSQLDialect |
| Ingres | org.hibernate.dialect.IngresDialect |
| Progress | org.hibernate.dialect.ProgressDialect |
| Mckoi SQL | org.hibernate.dialect.MckoiDialect |
| Interbase | org.hibernate.dialect.InterbaseDialect |
| Pointbase | org.hibernate.dialect.PointbaseDialect |
| FrontBase | org.hibernate.dialect.FrontbaseDialect |
| Firebird | org.hibernate.dialect.FirebirdDialect |



