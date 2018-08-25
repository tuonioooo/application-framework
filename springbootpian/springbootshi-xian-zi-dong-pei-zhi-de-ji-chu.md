# SpringBoot实现自动配置的基础

## 引子 {#引子}

### 问题是什么 {#问题是什么}

Spring中纷繁复杂的配置文件一直让广大开发人员颇有微词，但是随着业务的复杂度越来越高，这也是在所难免的事情。

哪怕是要创建一个简单的Web应用，需要配置的东西也是一坨坨的，这个过程虽然不复杂但是很繁琐，而且非常容易出错。所以聪明的开发人员想出了很多办法来解决这个问题，比如Maven的Archetype创建，又或者各个公司内部的脚手架小工具。但是这些方案总是有这样那样的问题，比如维护不方便，不好定制等等。

在云计算，弹性计算以及微服务越来越普及的今天，急需要一种自动配置的方式，从而方便地按需部署。所以Spring Boot应用而生，而自动配置作为Spring Boot的闪光点之一，自然非常受人关注。

Spring Boot的自动配置功能，其实从本质上说并没有引入什么新功能，它只是将Spring现存的能力做了一次组合和封装。那么在深入了解Spring Boot的自动配置原理之前，可以先了解一下Spring的这些已知能力，打下良好地基础。

## 和配置相关的Spring已有能力 {#和配置相关的spring已有能力}

### @Profile {#profile}

很多时候，我们开发的Spring应用，需要根据所在环境的不同注册相应的Bean到上下文中。

比如本地开发环境中，数据库连接对象往往指向的是开发数据库；在测试环境中，又会指向测试数据库；而到了线上，指向的自然是生产数据库。

为了满足这个常见需求，Spring 3.1中引入了Profile的概念。比如在下面的代码中，配置类会根据所在环境\(Profile\)的不同，向上下文中注入对应的Bean实例。

```
@Configuration
public class DataSourceConfiguration
{
    @Bean
    @Profile("DEV")
    public DataSource devDataSource() {
        // ...
    }

    @Bean
    @Profile("TEST")
    public DataSource testDataSource() {
        // ...
    }

    @Bean
    @Profile("PROD")
    public DataSource prodDataSource() {
        // ...
    }
}
```

那么，如何声明应用所处的Profile呢？还是有几种选择：

1. 配置文件，比如在application.properties中声明
   `spring.profiles.active=DEV`
2. 启动参数，比如
   `-Dspring.profiles.active=DEV`

这个Profile的概念很直观，但是由于它仅仅是依赖一个字符串的值作出决策，所以不够灵活和强大。因此就有了下面@Conditional注解和Condition接口的诞生。

### @Conditional以及Condition接口 {#conditional以及condition接口}

它们是Spring 4中引入的新功能。

Condition接口和@Conditional接口通常会一起配合使用。

Condition接口的定义如下：

```
public interface Condition {

    /**
     * 决定Condition是否被满足
     * @param context 上下文信息
     * @param metadata 类或方法上的元数据注解信息
     */
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);

}
```

下面是一个该接口的实现类：

```
public class CustomCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 通过一系列计算决定Condition是否被满足，如果满足返回true，反之返回false
        return true;
    }
}
```

@Conditional注解的定义如下：

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

    /**
     * 该注解接受的默认参数类型是一个Class数组，这些Class都应该实现Condition接口，并且当其中的matches都返回true的时候，对应的Bean才会被注入
     */
    Class<? extends Condition>[] value();

}
```

然后一般在Java Config类中将它们结合在一起使用：

```
@Configuration
public class CustomConfiguration {
    // 当CustomCondition中的matches返回true的时候才会注入CustomClass这个类的实例Bean。
    @Bean
    @Conditional(CustomCondition.class)
    public CustomClass customClass() {
        return new CustomClass();
    }
}
```

因此通过将@Conditional注解和Condition接口的能力进行结合，就可以实现根据外界条件来决定一个Bean是不是应该被加载到Spring上下文中。对比一下@Profile注解，可以发现这种方式提供了更高的灵活度。

在@Profile注解中，唯一决定是否加载一个Bean的是一个字符串，即所在环境的标识符例如DEV，TEST，PRODUCTION这类。但是在@Conditional的方案中，决定是否加载的逻辑被抽象到了Condition接口中，这样能够做的判断就非常多了。

### @ConfigurationProperties以及@EnableConfigurationProperties {#configurationproperties以及enableconfigurationproperties}

要实现自动配置，还有一个比较关键的环节。我们都知道Bean之所以有意义是因为每个Bean中都有自己独特的字段，而很多情况下，Bean中的字段都是可配置的。因此，要实现自动配置，我们首先需要对一些关键字段给予一个默认值，当然也需要提供一个机制让用户能够对这些字段进行覆盖。在Spring Boot中，提供了一个新的机制叫做Configuration Properties\(可配置的属性\)，它有几个特点：

1. 类型安全的配置方式\(基于Java类\)，基于Spring Conversion API完成类型转换
2. 可以通过配置文件，启动参数等多种方式完成覆盖
3. 和JavaConfig类无缝集成

这个机制有两个关键的注解类型，即@ConfigurationProperties和@EnableConfigurationProperties。下面我们就来看看它们的定义以及如何应用：

#### 定义方式 {#定义方式}

**@ConfigurationProperties**

```
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ConfigurationProperties {

    @AliasFor("prefix")
    String value() default "";

    @AliasFor("value")
    String prefix() default "";

    // ...
}
```

只捡主要的属性介绍，这个注解中比较关键的就是value/prefix属性。这两个属性是一个意思，互相设置了@AliasFor。

**@EnableConfigurationProperties**

```
/**
 * 用于支持@ConfigurationProperties标注的Beans。
 * @ConfigurationProperties注解的Bean也可以通过标准方式(比如使用@Bean方法)，但是这个注解提供了一个更方便的方法。
 *
 * @author Dave Syer
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(EnableConfigurationPropertiesImportSelector.class)
public @interface EnableConfigurationProperties {

    Class<?>[] value() default {};

}
```

#### 应用方式 {#应用方式}

以Spring Web应用中需要的HttpEncodingFilter这个Filter的自动配置为例，看看如何应用这两注解：

```
@ConfigurationProperties(prefix = "spring.http.encoding")
public class HttpEncodingProperties {

    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

    private Charset charset = DEFAULT_CHARSET;

    private Boolean force;

    // ...

}
```

这是一个配置类，它设置了前缀spring.http.encoding。因此，这个配置类中的字段都会对应某一个具体的配置KEY，比如charset字段对应的配置KEY为spring.http.encoding.charse; force字段对应的是spring.http.encoding.force。同时可以给每个字段设置一个默认值，比如上述的charset字段就有一个默认值DEFAULT\_CHARSET，对应的是UTF-8的字符集。

如果我们不想用默认的UTF-8编码方式，想换成GBK，那么在配置文件\(如application.properties中\)，可以进行如下覆盖：

```
spring.http.encoding.charset=GBK
```

这里有个地方值得一提，在配置文件中的GBK明明是字符串类型的值，而在对应的配置属性类这个属性是Charset类型的，那么肯定是有一个步骤完成了从字符串到Charset类型的转换工作。完成这个步骤的就是Spring框架中的Conversion API。这里只贴一张相关的图，不进行深入分析，感兴趣的同学可以在配置属性类的setCharset方法上打个断点然后深挖一下。

![](/assets/import-condition-01.png)

从这张图可以看出，Spring Conversion API中维护了一系列Converters，它实际上是一个从源类型到目标类型的Map对象。从String类型到Charset类型的Converter也被注册到了这个Map对象中。因此，当发现配置文件中的值类型和配置Java类中的字段类型不匹配时，就会去尝试从这个Map中找到相应的Converter，然后进行转换。

### 自定义注解实现自动配置 {#自定义注解实现自动配置}

下面我们尝试来创建一个自定义的注解，实现Bean的按需加载。假设我们要创建的注解名为@DatabaseType。

具体的需求是：当启动参数中的dbType=MYSQL的时候，创建一个数据源为MySQL的UserDAO对象；当启动参数中的dbType=ORACLE的时候，创建一个数据源为Oracle的UserDAO对象。

最终配置类的代码可以是这样的：

```
@Configuration
public class DatabaseConfiguration
{
  @Bean
  @DatabaseType("MYSQL")
  public UserDAO mysqlUserDAO() {
      return new MySQLUserDAO();
  }

  @Bean
  @DatabaseType("ORACLE")
  public UserDAO oracleUserDAO() {
      return new OracleUserDAO();
  }
}
```

可以知道，@DatabaseType这个注解能够接受一个字符串作为参数。然后将该参数和启动参数中的dbType值进行比对，如果相等的话，就会进行对应Bean的注入工作。@DatabaseType的实现如下：

```
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Conditional(DatabaseTypeCondition.class)
public @interface DatabaseType {
    String value();
}
```

其中比较重要的是`@Conditional(DatabaseTypeCondition.class)`，表明它的觉得实际上也是委托给了DatabaseTypeCondition这个类：

```
public class DatabaseTypeCondition implements Condition {
    // 这里也展示了AnnotatedTypeMetadata这个参数的用法
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata metadata) {
        Map<String, Object> attributes = metadata.getAnnotationAttributes(DatabaseType.class.getName());
        String type = (String) attributes.get("value");
        String enabledDBType = System.getProperty("dbType", "MYSQL");
        return (enabledDBType != null && type != null && enabledDBType.equalsIgnoreCase(type));
    }
}
```

通过matches方法的第二个参数AnnotatedTypeMetadata，可以得到指定注解类型的所有属性，本例中就是@DatabaseType这个注解。然后将注解中的value值和启动参数dbType\(不存在时使用MYSQL作为默认值\)进行比对，然后返回相应的值来决定是否需要注入Bean。

所以，通过这个例子我们也可以发现。通常而言为了程序的可读性，可以将@Conditional和Condition接口的实现类给封装到一个业务含义更加明确的注解类型中，比如上面的@DatabaseType类型。它的意义就很明确，当类型是MySQL的该如何如何，当类型为Oracle的时候又当如何如何。

在后面深究Spring Boot的自动配置机制时，可以发现它也在@Conditional和Condition接口的基础上，定义了很多相关类型，用于更好地定义自动配置行为。

##  总结

本文介绍了Spring中和自动配置相关的两个概念：

1. @Profile
2. @Conditional以及Condition接口

然后列举了一个使用自定义注解来完成自动配置的例子。

有了这些知识作为基础，在下一篇文章中，就可以开始深入Spring Boot实现自动配置的原理了。

## 参考

[https://blog.csdn.net/dm\_vincent/article/details/77435515](https://blog.csdn.net/dm_vincent/article/details/77435515)

