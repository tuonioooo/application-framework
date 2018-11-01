# BeanFactory

Spring Bean的创建是典型的工厂模式，这一系列的Bean工厂，也即IOC容器为开发者管理对象间的依赖关系提供了很多便利和基础服务，在Spring中有许多的IOC容器的实现供用户选择和使用，其相互关系如下：

![](/assets/import-beanfactory-02.png)

其中BeanFactory作为最顶层的一个接口类，它定义了IOC容器的基本功能规范，BeanFactory 有三个子类：ListableBeanFactory、HierarchicalBeanFactory 和AutowireCapableBeanFactory。但是从上图中我们可以发现最终的默认实现类是 DefaultListableBeanFactory，他实现了所有的接口。那为何要定义这么多层次的接口呢？查阅这些接口的源码和说明发现，每个接口都有他使用的场合，它主要是为了区分在 Spring 内部在操作过程中对象的传递和转化过程中，对对象的数据访问所做的限制。例如 ListableBeanFactory 接口表示这些 Bean 是可列表的，而 HierarchicalBeanFactory 表示的是这些 Bean 是有继承关系的，也就是每个Bean 有可能有父 Bean。AutowireCapableBeanFactory 接口定义 Bean 的自动装配规则。这四个接口共同定义了 Bean 的集合、Bean 之间的关系、以及 Bean 行为.

最基本的IOC容器接口BeanFactory

```
public interface BeanFactory {
    //对FactoryBean的转义定义，因为如果使用bean的名字检索FactoryBean得到的对象是工厂生成的对象，
    //如果需要得到工厂本身，需要转义 
    String FACTORY_BEAN_PREFIX = "&";
    //根据bean的名字，获取在IOC容器中得到bean实例    
    Object getBean(String var1) throws BeansException;
    //根据bean的名字和Class类型来得到bean实例，增加了类型安全验证机制。  
    <T> T getBean(String var1, Class<T> var2) throws BeansException;

    Object getBean(String var1, Object... var2) throws BeansException;

    <T> T getBean(Class<T> var1) throws BeansException;

    <T> T getBean(Class<T> var1, Object... var2) throws BeansException;

    //提供对bean的检索，看看是否在IOC容器有这个名字的bean    
    boolean containsBean(String var1);
    //根据bean名字得到bean实例，并同时判断这个bean是不是单例 
    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;

    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, ResolvableType var2) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, Class<?> var2) throws NoSuchBeanDefinitionException;
    //得到bean实例的Class类型    
    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;
    //得到bean的别名，如果根据别名检索，那么其原名也会被检索出来   
    String[] getAliases(String var1);
}
```

ListableBeanFactory接口

```
package org.springframework.beans.factory;

import java.lang.annotation.Annotation;
import java.util.Map;

import org.springframework.beans.BeansException;
import org.springframework.core.ResolvableType;

/**
 * Extension of the {@link BeanFactory} interface to be implemented by bean factories
 * that can enumerate all their bean instances, rather than attempting bean lookup
 * by name one by one as requested by clients. BeanFactory implementations that
 * preload all their bean definitions (such as XML-based factories) may implement
 * this interface.
 * 
 * 是BeanFactory接口的扩展，被能够一次性列举所有它们bean实例，而不是试图根据客户端请求一个一个的通过名字查找的
 * 的工厂容器实现。那些需要预先加载所有bean定义的工厂需要实现这个接口。
 *
 * <p>If this is a {@link HierarchicalBeanFactory}, the return values will <i>not</i>
 * take any BeanFactory hierarchy into account, but will relate only to the beans
 * defined in the current factory. Use the {@link BeanFactoryUtils} helper class
 * to consider beans in ancestor factories too.
 * 
 * 如果这是一个层次继承的接口，则只会考虑当前层次这个工厂内的bean定义，而不会考虑其他任何层次。
 * 也可以使用BeanFactoryUtils这个帮助类来针对其祖先工厂也考虑的情况。
 *
 * <p>The methods in this interface will just respect bean definitions of this factory.
 * They will ignore any singleton beans that have been registered by other means like
 * {@link org.springframework.beans.factory.config.ConfigurableBeanFactory}'s
 * {@code registerSingleton} method, with the exception of
 * {@code getBeanNamesOfType} and {@code getBeansOfType} which will check
 * such manually registered singletons too. Of course, BeanFactory's {@code getBean}
 * does allow transparent access to such special beans as well. However, in typical
 * scenarios, all beans will be defined by external bean definitions anyway, so most
 * applications don't need to worry about this differentiation.
 * 
 * 这个接口中的方法仅仅关注此工厂内部的bean定义。
 * 它们会忽略任何被像ConfigurableBeanFactory的registerSingleton方法已注册的单例的bean，
 * getBeanNamesOfType和getBeansOfType例处，但也会手动的检查已被注册的单例bean。
 * 当然，BeanFactory的getBean方法也可以透明的访问这些特殊的bean（已被注册的单例bean）。
 * 然而，在经典的场合，无论如何，所有bean都会被处部定义 定义，所以许多程序不需要这方面不同。
 *
 * <p><b>NOTE:</b> With the exception of {@code getBeanDefinitionCount}
 * and {@code containsBeanDefinition}, the methods in this interface
 * are not designed for frequent invocation. Implementations may be slow.
 * 
 * 注意：除了getBeanDefinitionCount和containsBeanDefinition，这个接口中的方法没有被当作
 * 频烦调用的方法设计，实现可以会慢。
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @since 16 April 2001
 * @see HierarchicalBeanFactory
 * @see BeanFactoryUtils
 */
public interface ListableBeanFactory extends BeanFactory {

    /**
     * Check if this bean factory contains a bean definition with the given name.
     * <p>Does not consider any hierarchy this factory may participate in,
     * and ignores any singleton beans that have been registered by
     * other means than bean definitions.
     * @param beanName the name of the bean to look for
     * @return if this bean factory contains a bean definition with the given name
     * @see #containsBean
     */
    /**
     * 检查这个BeanFactory是否包含给定名字的的bean定义。
     * 不考虑这个工厂参与的任何层级关系，
     * 忽略不是bean定义的所有通过其他方式注册的单例bean
     * @param beanName
     * @return
     */
    boolean containsBeanDefinition(String beanName);

    /**
     * Return the number of beans defined in the factory.
     * <p>Does not consider any hierarchy this factory may participate in,
     * and ignores any singleton beans that have been registered by
     * other means than bean definitions.
     * @return the number of beans defined in the factory
     */
    /**
     * 返回在此工厂中定义的bean数量。
     * 不考虑这个工厂参与的任何层级关系，
     * 忽略不是bean定义的所有通过其他方式注册的单例bean
     * @return
     */
    int getBeanDefinitionCount();

    /**
     * Return the names of all beans defined in this factory.
     * <p>Does not consider any hierarchy this factory may participate in,
     * and ignores any singleton beans that have been registered by
     * other means than bean definitions.
     * @return the names of all beans defined in this factory,
     * or an empty array if none defined
     */
    /**
     * 返回此工厂中定义的所有bean的名字。
     * 不考虑此工厂参与的所有层次关系，
     * 忽略不是bean定义的所有通过其他方式注册的单例bean
     * @return
     * 如果此工厂没有bean定义，返回一个空的数组。
     */
    String[] getBeanDefinitionNames();

    /**
     * Return the names of beans matching the given type (including subclasses),
     * judging from either bean definitions or the value of {@code getObjectType}
     * in the case of FactoryBeans.
     * 
     * 返回匹配给定类型（包括子类）的所有bean的名字，如果是普通bean，则是bean定义的名字，如果是
     * FactoryBean，则是其getObjectType方法返回的对象的名字
     * 
     * <p><b>NOTE: This method introspects top-level beans only.</b> It does <i>not</i>
     * check nested beans which might match the specified type as well.
     * 
     * 这个方法只考虑最顶层的bean，内部嵌套的bean即便可能匹配指定的类型也不考虑。
     * 
     * <p>Does consider objects created by FactoryBeans, which means that FactoryBeans
     * will get initialized. If the object created by the FactoryBean doesn't match,
     * the raw FactoryBean itself will be matched against the type.
     * 
     * 如果考虑FactoryBean创建的对象，意味着FactoryBean已经初始化。
     * 如果FactoryBean创建的对象不匹配，FactoryBean自身将与对象进行匹配。
     * 
     * <p>Does not consider any hierarchy this factory may participate in.
     * Use BeanFactoryUtils' {@code beanNamesForTypeIncludingAncestors}
     * to include beans in ancestor factories too.
     * 
     * 不考虑任何此工厂参与的层级关系。
     * 也可以使用BeanFactoryUtils的beanNamesForTypeIncludingAncestors方法处理有关继承方面的问题。
     * 
     * <p>Note: Does <i>not</i> ignore singleton beans that have been registered
     * by other means than bean definitions.
     * 
     * 不忽略不是bean定义的通过其他方式注册的单例bean.
     * 
     * <p>This version of {@code getBeanNamesForType} matches all kinds of beans,
     * be it singletons, prototypes, or FactoryBeans. In most implementations, the
     * result will be the same as for {@code getBeanNamesForType(type, true, true)}.
     * 
     * getBeansOfType方法的这个版本匹配所有种类的bean，可是是单例的，原型的，FactoryBean.
     * 在许多实现中，此方法返回的结果与调用getBeansOfType(type, true, true)一样。
     * 
     * <p>Bean names returned by this method should always return bean names <i>in the
     * order of definition</i> in the backend configuration, as far as possible.
     *  
     * 这个方法返回的bean名称应该尽可能与后台配置的bean定义顺序一样。
     * 
     * @param type the class or interface to match, or {@code null} for all bean names
     *  要匹配的类或者是接口，如果是null，返回所有bean的名字
     * @return the names of beans (or objects created by FactoryBeans) matching
     * the given object type (including subclasses), or an empty array if none
     *  如果是不存指定类型的bean，则返回一个空的数组
     * @since 4.2
     * @see #isTypeMatch(String, ResolvableType)
     * @see FactoryBean#getObjectType
     * @see BeanFactoryUtils#beanNamesForTypeIncludingAncestors(ListableBeanFactory, ResolvableType)
     */
    String[] getBeanNamesForType(ResolvableType type);

    /**
     * Return the names of beans matching the given type (including subclasses),
     * judging from either bean definitions or the value of {@code getObjectType}
     * in the case of FactoryBeans.
     * <p><b>NOTE: This method introspects top-level beans only.</b> It does <i>not</i>
     * check nested beans which might match the specified type as well.
     * <p>Does consider objects created by FactoryBeans, which means that FactoryBeans
     * will get initialized. If the object created by the FactoryBean doesn't match,
     * the raw FactoryBean itself will be matched against the type.
     * <p>Does not consider any hierarchy this factory may participate in.
     * Use BeanFactoryUtils' {@code beanNamesForTypeIncludingAncestors}
     * to include beans in ancestor factories too.
     * <p>Note: Does <i>not</i> ignore singleton beans that have been registered
     * by other means than bean definitions.
     * <p>This version of {@code getBeanNamesForType} matches all kinds of beans,
     * be it singletons, prototypes, or FactoryBeans. In most implementations, the
     * result will be the same as for {@code getBeanNamesForType(type, true, true)}.
     * <p>Bean names returned by this method should always return bean names <i>in the
     * order of definition</i> in the backend configuration, as far as possible.
     * @param type the class or interface to match, or {@code null} for all bean names
     * @return the names of beans (or objects created by FactoryBeans) matching
     * the given object type (including subclasses), or an empty array if none
     * @see FactoryBean#getObjectType
     * @see BeanFactoryUtils#beanNamesForTypeIncludingAncestors(ListableBeanFactory, Class)
     */
    /**
     * 此处翻译同上
     * @param type
     * @return
     */
    String[] getBeanNamesForType(Class<?> type);

    /**
     * Return the names of beans matching the given type (including subclasses),
     * judging from either bean definitions or the value of {@code getObjectType}
     * in the case of FactoryBeans.
     * <p><b>NOTE: This method introspects top-level beans only.</b> It does <i>not</i>
     * check nested beans which might match the specified type as well.
     * 
     * 此处以上同上。
     * 
     * <p>Does consider objects created by FactoryBeans if the "allowEagerInit" flag is set,
     * which means that FactoryBeans will get initialized. If the object created by the
     * FactoryBean doesn't match, the raw FactoryBean itself will be matched against the
     * type. If "allowEagerInit" is not set, only raw FactoryBeans will be checked
     * (which doesn't require initialization of each FactoryBean).
     * 
     * 考虑对象为FactoryBean创建的情况，如果allowEagerInit设置，FactoryBean对象会初始化，如果被FactoryBean创建
     * 的对象与指定类型不匹配，那么将与此FactoryBean本身匹配;
     * 如果allowEagerInit没有设置，仅仅FactoryBean本身被检查。
     * 
     * <p>Does not consider any hierarchy this factory may participate in.
     * Use BeanFactoryUtils' {@code beanNamesForTypeIncludingAncestors}
     * to include beans in ancestor factories too.
     * <p>Note: Does <i>not</i> ignore singleton beans that have been registered
     * by other means than bean definitions.
     * <p>Bean names returned by this method should always return bean names <i>in the
     * order of definition</i> in the backend configuration, as far as possible.
     * 
     * 此段同上
     * 
     * @param type the class or interface to match, or {@code null} for all bean names
     * @param includeNonSingletons whether to include prototype or scoped beans too
     * or just singletons (also applies to FactoryBeans)
     * 
     * 是否也包含多实例型bean或者scoped bean，或者仅仅只是单例bean（也适用于FactoryBean）
     * 
     * @param allowEagerInit whether to initialize <i>lazy-init singletons</i> and
     * <i>objects created by FactoryBeans</i> (or by factory methods with a
     * "factory-bean" reference) for the type check. Note that FactoryBeans need to be
     * eagerly initialized to determine their type: So be aware that passing in "true"
     * for this flag will initialize FactoryBeans and "factory-bean" references.
     * 
     * 是否允许马上加载 ，如果是factoryBean创建的对象，此处应是true
     * 
     * @return the names of beans (or objects created by FactoryBeans) matching
     * the given object type (including subclasses), or an empty array if none
     * @see FactoryBean#getObjectType
     * @see BeanFactoryUtils#beanNamesForTypeIncludingAncestors(ListableBeanFactory, Class, boolean, boolean)
     */
    String[] getBeanNamesForType(Class<?> type, boolean includeNonSingletons, boolean allowEagerInit);

    /**
     * Return the bean instances that match the given object type (including
     * subclasses), judging from either bean definitions or the value of
     * {@code getObjectType} in the case of FactoryBeans.
     * 
     * 返回匹配给定类型（包含子类）的实例，可能是通过bean定义创建，也可以是FactoryBean时其getObjectType返回
     * 
     * <p><b>NOTE: This method introspects top-level beans only.</b> It does <i>not</i>
     * check nested beans which might match the specified type as well.
     * 
     * 注意：此方法仅考虑最顶层bean，不含其内部嵌套的bean，即使内部嵌套的bean匹配给定类型
     * 
     * <p>Does consider objects created by FactoryBeans, which means that FactoryBeans
     * will get initialized. If the object created by the FactoryBean doesn't match,
     * the raw FactoryBean itself will be matched against the type.
     * 
     * 如果考虑FactoryBean创建的对象，需要先初始化对应的FactoryBean。
     * 如果FactoryBean创建的对象与指定类型不匹配，则需要匹配FactoryBean对象本身
     * 
     * <p>Does not consider any hierarchy this factory may participate in.
     * Use BeanFactoryUtils' {@code beansOfTypeIncludingAncestors}
     * to include beans in ancestor factories too.
     * 
     * 不考虑此工厂所参与的任何层次，也可以使用BeanFactoryUtils的beansOfTypeIncludingAncestors
     * 的方法处理考虑分层处理的情况。
     * 
     * <p>Note: Does <i>not</i> ignore singleton beans that have been registered
     * by other means than bean definitions.
     * 
     * 注：不忽略那些不是bean定义的已经通过其他方式创建的单例bean
     * 
     * <p>This version of getBeansOfType matches all kinds of beans, be it
     * singletons, prototypes, or FactoryBeans. In most implementations, the
     * result will be the same as for {@code getBeansOfType(type, true, true)}.
     * 
     * getBeansOfType方法的这个版本匹配所有种类的bean，可是是单例的，原型的，FactoryBean.
     * 在许多实现中，此方法返回的结果与调用getBeansOfType(type, true, true)一样。
     * 
     * <p>The Map returned by this method should always return bean names and
     * corresponding bean instances <i>in the order of definition</i> in the
     * backend configuration, as far as possible.
     * 
     * 这个方法返回的map应该是匹配指定类型的bean的名字与其名字对应的实例的键值对，
     * 并且顺序要尽最大可能的与配置时一样。
     * 
     * @param type the class or interface to match, or {@code null} for all concrete beans
     *          指定的类或者是接口，如空是空，则匹配所有现有bean
     * @return a Map with the matching beans, containing the bean names as
     * keys and the corresponding bean instances as values
     * @throws BeansException if a bean could not be created  
     *          如果bean不能被创建 ，则抛出此异常
     * @since 1.1.2
     * @see FactoryBean#getObjectType
     * @see BeanFactoryUtils#beansOfTypeIncludingAncestors(ListableBeanFactory, Class)
     */
    <T> Map<String, T> getBeansOfType(Class<T> type) throws BeansException;

    /**
     * Return the bean instances that match the given object type (including
     * subclasses), judging from either bean definitions or the value of
     * {@code getObjectType} in the case of FactoryBeans.
     * <p><b>NOTE: This method introspects top-level beans only.</b> It does <i>not</i>
     * check nested beans which might match the specified type as well.
     * 
     * 此处以上参考上面
     * 
     * <p>Does consider objects created by FactoryBeans if the "allowEagerInit" flag is set,
     * which means that FactoryBeans will get initialized. If the object created by the
     * FactoryBean doesn't match, the raw FactoryBean itself will be matched against the
     * type. If "allowEagerInit" is not set, only raw FactoryBeans will be checked
     * (which doesn't require initialization of each FactoryBean).
     * 
     * 考虑FactoryBean创建的对象时：如果设置allowEagerInit，也就是默认初始化FactoryBean.
     * 这时如果FactoryBean创建的对象与指定的类型不匹配，将匹配该FactoryBean实例本身。
     * 
     * <p>Does not consider any hierarchy this factory may participate in.
     * Use BeanFactoryUtils' {@code beansOfTypeIncludingAncestors}
     * to include beans in ancestor factories too.
     * <p>Note: Does <i>not</i> ignore singleton beans that have been registered
     * by other means than bean definitions.
     * <p>The Map returned by this method should always return bean names and
     * corresponding bean instances <i>in the order of definition</i> in the
     * backend configuration, as far as possible.
     * 
     * 此处同样参考上面
     * 
     * @param type the class or interface to match, or {@code null} for all concrete beans
     * @param includeNonSingletons whether to include prototype or scoped beans too
     * or just singletons (also applies to FactoryBeans)
     * @param allowEagerInit whether to initialize <i>lazy-init singletons</i> and
     * <i>objects created by FactoryBeans</i> (or by factory methods with a
     * "factory-bean" reference) for the type check. Note that FactoryBeans need to be
     * eagerly initialized to determine their type: So be aware that passing in "true"
     * for this flag will initialize FactoryBeans and "factory-bean" references.
     * @return a Map with the matching beans, containing the bean names as
     * keys and the corresponding bean instances as values
     * @throws BeansException if a bean could not be created
     * @see FactoryBean#getObjectType
     * @see BeanFactoryUtils#beansOfTypeIncludingAncestors(ListableBeanFactory, Class, boolean, boolean)
     */
    <T> Map<String, T> getBeansOfType(Class<T> type, boolean includeNonSingletons, boolean allowEagerInit)
            throws BeansException;

    /**
     * Find all names of beans whose {@code Class} has the supplied {@link Annotation}
     * type, without creating any bean instances yet.
     * @param annotationType the type of annotation to look for
     * @return the names of all matching beans
     * @since 4.0
     */
    /**
     * 通过指定的注解类型，获取所有那些还没有创建bean实例的名字。
     * @param annotationType  指定的注解类型
     * @return  匹配的所有bean的名字
     */
    String[] getBeanNamesForAnnotation(Class<? extends Annotation> annotationType);

    /**
     * Find all beans whose {@code Class} has the supplied {@link Annotation} type,
     * returning a Map of bean names with corresponding bean instances.
     * @param annotationType the type of annotation to look for
     * @return a Map with the matching beans, containing the bean names as
     * keys and the corresponding bean instances as values
     * @throws BeansException if a bean could not be created
     * @since 3.0
     */
    /**
     * 查找所有注解为指定类型的bean，返回一个bean名字与其对应实例的映射表。
     * @param annotationType    要查找的注解类型
     * @return  返回一map，以匹配的bean名字作为键，以bean名字对应的bean实例作为值。
     * @throws BeansException   如果bean实例不能被创建，抛出
     */
    Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> annotationType) throws BeansException;

    /**
     * Find an {@link Annotation} of {@code annotationType} on the specified
     * bean, traversing its interfaces and super classes if no annotation can be
     * found on the given class itself.
     * @param beanName the name of the bean to look for annotations on
     * @param annotationType the annotation class to look for
     * @return the annotation of the given type if found, or {@code null}
     * @throws NoSuchBeanDefinitionException if there is no bean with the given name
     * @since 3.0
     */

    /**
     * 查找指定bean的注解。
     * 如果在指定bean自身上面没有找到，则遍历它实现的接口和他的超类。
     * @param beanName  指定的bean 的名称
     * @param annotationType   指定的注解类型
     * @return 返回找到的注解，或者是null
     * @throws NoSuchBeanDefinitionException  如果找不到给定名字的bean定义，抛出此异常
     */
    <A extends Annotation> A findAnnotationOnBean(String beanName, Class<A> annotationType)
            throws NoSuchBeanDefinitionException;

}
```



