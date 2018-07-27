# Sqlsession原理

## 概述

每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。也绝不能将 SqlSession 实例的引用放在任何类型的管理作用域中，比如 Servlet 架构中的 HttpSession。如果你现在正在使用一种 Web 框架，要考虑 SqlSession 放在一个和 HTTP 请求对象相似的作用域中。换句话说，每次收到的 HTTP 请求，就可以打开一个 SqlSession，返回一个响应，就关闭它。这个关闭操作是很重要的，你应该把这个关闭操作放到 finally 块中以确保每次都能执行关闭。下面的示例就是一个确保 SqlSession 关闭的标准模式：

```
SqlSession session = sqlSessionFactory.openSession();
try {
  // do work
} finally {
  session.close();
}
```

在你的所有的代码中一致性地使用这种模式来保证所有数据库资源都能被正确地关闭。

## 工作原理

在Mybatis中向DAO层提供的这个能够与数据库交互并执行SQL语句的对象叫做SqlSession。这个是Mybatis最核心的一个对象。SqlSession完全包含了面向数据库执行SQL命令所需的全部方法。

那么如何获得SqlSession这个对象呢？

```
package com.master;

import com.master.bean.Account;
import com.master.mapper.AccountMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-7-26
 * Time: 17:19
 * info: 用XML构建SqlSessionFactory获取SqlSession，进行持久化操作
 */
public class CreateSqlSessionByXML {

    public static void main(String[] args) throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        xmlMapper(sqlSessionFactory);

    }

    public static void xmlMapper(SqlSessionFactory sqlSessionFactory){
        SqlSession session = sqlSessionFactory.openSession();
        try {
            Account account = (Account) session.selectOne("mapper.AccountMapper.findAccount", 1);
        } finally {
            session.close();
        }
    }

    public static void interfaceMapper(SqlSessionFactory sqlSessionFactory){
        SqlSession session = sqlSessionFactory.openSession();
        try {
            AccountMapper mapper = session.getMapper(AccountMapper.class);
            Account account = mapper.findAccount(1);
        } finally {
            session.close();
        }
    }
}

```



