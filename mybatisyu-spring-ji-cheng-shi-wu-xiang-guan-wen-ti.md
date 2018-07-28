# Mybatis与Spring集成事务相关问题

一般使用Mybaits也就这么几种情况。

（1）单独使用Mybatis不与Spring集成，那么应该这么使用，最原始的方式。

Java代码

```
     //这里要传false 手动事务

    SqlSession session = sqlSessionFactory.openSession(false);

     try {

        //插入A表

        //修改B表

        session.commit();

    } catch (Exception e) {

        session.rollback();

    } finally {

        session.close();

    }
```

（2）与Spring集成使用，但是没有将事务托管给Spring,一般都是使用SqlSessionTemplate这个类，这种情况在Mybatis执行增删改查以后Mybatis会自动提交事务，关闭session。

Java代码

```
private class SqlSessionInterceptor implements InvocationHandler {  

        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

            final SqlSession sqlSession = getSqlSession(

                    SqlSessionTemplate.this.sqlSessionFactory,

                    SqlSessionTemplate.this.executorType,

                    SqlSessionTemplate.this.exceptionTranslator);

            try {

                Object result = method.invoke(sqlSession, args);

                if (!isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {

                    // force commit even on non-dirty sessions because some databases require  

                    // a commit/rollback before calling close()  

                    sqlSession.commit(true);

                }

                return result;

            } catch (Throwable t) {

                Throwable unwrapped = unwrapThrowable(t);

                if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {

                    Throwable translated = SqlSessionTemplate.this.exceptionTranslator.translateExceptionIfPossible((PersistenceException) unwrapped);

                    if (translated != null) {

                        unwrapped = translated;

                    }

                }

                throw unwrapped;

            } finally {

                closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);

            }

        }  

    }
```

（3）与Spring集成使用，并且使用了Spring的事物上下文，那么事物会由Spring管理，这与hibernate与Spring集成的事物没有区别，Spring会管理事物。只需要配置好Spring事务管理，在相关接口是加注解@Transactional 即可

