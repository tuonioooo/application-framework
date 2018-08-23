# Spring源码中的那些设计模式

应该说设计模式是我们在写代码时候的一种被承认的较好的模式。好的设计模式就像是给代码造了一个很好的骨架，在这个骨架里，你可以知道心在哪里，肺在哪里，因为大多数人都认识这样的骨架，就有了很好的传播性。这是从易读和易传播来感知设计模式的好处。当然设计模式本身更重要的是设计原则的一种实现，比如开闭原则，依赖倒置原则，这些是在代码的修改和扩展上说事。说到底就是人类和代码发生关系的四种场合：阅读，修改，增加，删除。让每一种场合都比较舒服的话，就需要用设计模式。

下面来简单列举Spring中的设计模式：

**1. 简单工厂**

又叫做静态工厂方法（StaticFactory Method）模式，但不属于23种GOF设计模式之一。

简单工厂模式的实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。

Spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得Bean对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。

**2. 工厂方法（Factory Method）**

定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method使一个类的实例化延迟到其子类。

Spring中的FactoryBean就是典型的工厂方法模式。如下图：

![](/assets/import-factorymethod-01.png)

**3. 单例（Singleton）**

保证一个类仅有一个实例，并提供一个访问它的全局访问点。

Spring中的单例模式完成了后半句话，即提供了全局的访问点BeanFactory。但没有从构造器级别去控制单例，这是因为Spring管理的是是任意的Java对象。

**4. 适配器（Adapter）**

将一个类的接口转换成客户希望的另外一个接口。Adapter模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

Spring中在对于AOP的处理中有Adapter模式的例子，见如下图：

![](/assets/import-adapter-11.png)

由于Advisor链需要的是MethodInterceptor对象，所以每一个Advisor中的Advice都要适配成对应的MethodInterceptor对象。

**5.包装器（Decorator）**

动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator模式相比生成子类更为灵活。

![](/assets/import-decorator-01.png)

Spring中用到的包装器模式在类名上有两种表现：一种是类名中含有Wrapper，另一种是类名中含有Decorator。基本上都是动态地给一个对象添加一些额外的职责。

**6. 代理（Proxy）**

为其他对象提供一种代理以控制对这个对象的访问。

从结构上来看和Decorator模式类似，但Proxy是控制，更像是一种对功能的限制，而Decorator是增加职责。

![](/assets/import-proxy-11.png)

Spring的Proxy模式在aop中有体现，比如JdkDynamicAopProxy和Cglib2AopProxy。

**7.观察者（Observer）**

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

![](/assets/import-observer-01.png)

Spring中Observer模式常用的地方是listener的实现。如ApplicationListener。

**8. 策略（Strategy）**

定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化。

Spring中在实例化对象的时候用到Strategy模式，见如下图：

![](/assets/import-strategy-01.png)

在SimpleInstantiationStrategy中有如下代码说明了策略模式的使用情况：

![](/assets/import-strategy-02.png)

还有，第一个地方，加载资源文件的方式，使用了不同的方法，比如：ClassPathResourece，FileSystemResource，ServletContextResource，UrlResource但他们都有共同的借口Resource；

第二个地方就是在Aop的实现中，采用了两种不同的方式，JDK动态代理和CGLIB代理；  


第三个地方就是Spring的事务管理，PlatformTransactionManager代表事务管理接口，但是它不知道底层如何管理事务，它只要求事务管理

提供开始事务\(getTransaction\(\),commit\(\),rollback\(\)三个方法，但是如何实现则交给具体实现类来完成--不同的实现类代表不同的事务管理策略。

一般来说，spring事务管理下面主要针对

1\) JDBC\(org.springframework.jdbc.datasource.DataSourceTransactionManager\), 

2\) Hibernate \(org.springframework.orm.hibernate3.HibernateTransactionManager\)，

3\) JTA \(org.springframework.transaction.jta.JtaTransactionManager\)和

4\) JPA\(org.springframework.orm.jpa.JpaTransactionManager\)

四种具体的底层事务控制来包装的。

**9.模板方法（Template Method）**

定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。Template Method使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。Template Method模式一般是需要继承的。这里想要探讨另一种对Template Method的理解。Spring中的JdbcTemplate，在用这个类时并不想去继承这个类，因为这个类的方法太多，但是我们还是想用到JdbcTemplate已有的稳定的、公用的数据库连接，那么我们怎么办呢？我们可以把变化的东西抽出来作为一个参数传入JdbcTemplate的方法中。但是变化的东西是一段代码，而且这段代码会用到JdbcTemplate中的变量。怎么办？那我们就用回调对象吧。在这个回调对象中定义一个操纵JdbcTemplate中变量的方法，我们去实现这个方法，就把变化的东西集中到这里了。然后我们再传入这个回调对象到JdbcTemplate，从而完成了调用。这可能是Template Method不需要继承的另一种实现方式吧。

以下是一个具体的例子：

JdbcTemplate中的execute方法：

![](http://www.uml.org.cn/j2ee/images/2013010748.png)

JdbcTemplate执行execute方法：

![](http://www.uml.org.cn/j2ee/images/2013010749.png)

