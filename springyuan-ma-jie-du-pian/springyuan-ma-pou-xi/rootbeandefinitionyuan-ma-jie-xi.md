# RootBeanDefinition源码解析

```
public class RootBeanDefinition extends AbstractBeanDefinition {
    //默认允许缓存
    boolean allowCaching = true;
    //bean定义的持有者(修饰bean定义)
    private BeanDefinitionHolder decoratedDefinition;
    //bean指定的目标类型定义
    private volatile Class<?> targetType;
    //是否已经指定引用非重载方法的工厂方法名。
    boolean isFactoryMethodUnique = false;
    //用于构造函数注入属性的锁对象
    final Object constructorArgumentLock = new Object();

    /** Package-visible field for caching the resolved constructor or factory method */
    //缓存已解析的构造函数或工厂方法
    Object resolvedConstructorOrFactoryMethod;

    /** Package-visible field that marks the constructor arguments as resolved */
    //将构造函数参数标记为已解析。
    boolean constructorArgumentsResolved = false;

    /** Package-visible field for caching fully resolved constructor arguments */
    //缓存完全解析的构造函数参数
    Object[] resolvedConstructorArguments;

    /** Package-visible field for caching partly prepared constructor arguments */
    //缓存部分准备好的构造函数参数
    Object[] preparedConstructorArguments;
    //后处理锁对象
    final Object postProcessingLock = new Object();

    /** Package-visible field that indicates MergedBeanDefinitionPostProcessor having been applied */
    //表明已应用mergedbeandefinitionpostprocessor
    boolean postProcessed = false;

    /** Package-visible field that indicates a before-instantiation post-processor having kicked in */
    //指示已启动的一个实例化之前的后置处理器。
    volatile Boolean beforeInstantiationResolved;

    //1.
    private Set<Member> externallyManagedConfigMembers;
    //2.
    private Set<String> externallyManagedInitMethods;
    //3.
    private Set<String> externallyManagedDestroyMethods;


    //构造方法
    public RootBeanDefinition() {
        super();
    }

    //构造方法
    public RootBeanDefinition(Class<?> beanClass) {
        super();
        setBeanClass(beanClass);
    }

    //构造方法
    @Deprecated
    public RootBeanDefinition(Class beanClass, boolean singleton) {
        super();
        setBeanClass(beanClass);
        setSingleton(singleton);
    }

    //构造方法
    @Deprecated
    public RootBeanDefinition(Class beanClass, int autowireMode) {
        super();
        setBeanClass(beanClass);
        setAutowireMode(autowireMode);
    }

    //构造方法
    public RootBeanDefinition(Class<?> beanClass, int autowireMode, boolean dependencyCheck) {
        super();
        setBeanClass(beanClass);
        setAutowireMode(autowireMode);
        if (dependencyCheck && getResolvedAutowireMode() != AUTOWIRE_CONSTRUCTOR) {
            setDependencyCheck(RootBeanDefinition.DEPENDENCY_CHECK_OBJECTS);
        }
    }

    //构造方法
    @Deprecated
    public RootBeanDefinition(Class beanClass, MutablePropertyValues pvs) {
        super(null, pvs);
        setBeanClass(beanClass);
    }

    //构造方法
    @Deprecated
    public RootBeanDefinition(Class beanClass, MutablePropertyValues pvs, boolean singleton) {
        super(null, pvs);
        setBeanClass(beanClass);
        setSingleton(singleton);
    }

    //构造方法
    public RootBeanDefinition(Class<?> beanClass, ConstructorArgumentValues cargs, MutablePropertyValues pvs) {
        super(cargs, pvs);
        setBeanClass(beanClass);
    }

    //构造方法
    public RootBeanDefinition(String beanClassName) {
        setBeanClassName(beanClassName);
    }

    //构造方法
    public RootBeanDefinition(String beanClassName, ConstructorArgumentValues cargs, MutablePropertyValues pvs) {
        super(cargs, pvs);
        setBeanClassName(beanClassName);
    }

    //构造方法
    public RootBeanDefinition(RootBeanDefinition original) {
        super((BeanDefinition) original);
        this.allowCaching = original.allowCaching;
        this.decoratedDefinition = original.decoratedDefinition;
        this.targetType = original.targetType;
        this.isFactoryMethodUnique = original.isFactoryMethodUnique;
    }

    //构造方法
    RootBeanDefinition(BeanDefinition original) {
        super(original);
    }

    //不含有父bean定义
    public String getParentName() {
        return null;
    }
    //不含有父bean定义，否则抛错
    public void setParentName(String parentName) {
        if (parentName != null) {
            throw new IllegalArgumentException("Root bean cannot be changed into a child bean with parent reference");
        }
    }

    //设置bean定义的修饰者（对bean定义进行一层修饰，持有bean定义）
    public void setDecoratedDefinition(BeanDefinitionHolder decoratedDefinition) {
        this.decoratedDefinition = decoratedDefinition;
    }
    public BeanDefinitionHolder getDecoratedDefinition() {
        return this.decoratedDefinition;
    }

    //设置这个bean指定的目标类型定义，如果已知的提前。
    public void setTargetType(Class<?> targetType) {
        this.targetType = targetType;
    }
    public Class<?> getTargetType() {
        return this.targetType;
    }


    //指定引用非重载方法的工厂方法名。
    public void setUniqueFactoryMethodName(String name) {
        Assert.hasText(name, "Factory method name must not be empty");
        setFactoryMethodName(name);
        this.isFactoryMethodUnique = true;
    }

    /**
     * Check whether the given candidate qualifies as a factory method.
     * 检查给定的候选方法是否为工厂方法
     */
    public boolean isFactoryMethod(Method candidate) {
        return (candidate != null && candidate.getName().equals(getFactoryMethodName()));
    }

     //返回解析后的工厂方法作为Java对象方法,如果可用。
    public Method getResolvedFactoryMethod() {
        synchronized (this.constructorArgumentLock) {
            Object candidate = this.resolvedConstructorOrFactoryMethod;
            return (candidate instanceof Method ? (Method) candidate : null);
        }
    }

    public void registerExternallyManagedConfigMember(Member configMember) {
        synchronized (this.postProcessingLock) {
            if (this.externallyManagedConfigMembers == null) {
                this.externallyManagedConfigMembers = new HashSet<Member>(1);
            }
            this.externallyManagedConfigMembers.add(configMember);
        }
    }

    public boolean isExternallyManagedConfigMember(Member configMember) {
        synchronized (this.postProcessingLock) {
            return (this.externallyManagedConfigMembers != null &&
                    this.externallyManagedConfigMembers.contains(configMember));
        }
    }

    public void registerExternallyManagedInitMethod(String initMethod) {
        synchronized (this.postProcessingLock) {
            if (this.externallyManagedInitMethods == null) {
                this.externallyManagedInitMethods = new HashSet<String>(1);
            }
            this.externallyManagedInitMethods.add(initMethod);
        }
    }

    public boolean isExternallyManagedInitMethod(String initMethod) {
        synchronized (this.postProcessingLock) {
            return (this.externallyManagedInitMethods != null &&
                    this.externallyManagedInitMethods.contains(initMethod));
        }
    }

    public void registerExternallyManagedDestroyMethod(String destroyMethod) {
        synchronized (this.postProcessingLock) {
            if (this.externallyManagedDestroyMethods == null) {
                this.externallyManagedDestroyMethods = new HashSet<String>(1);
            }
            this.externallyManagedDestroyMethods.add(destroyMethod);
        }
    }

    public boolean isExternallyManagedDestroyMethod(String destroyMethod) {
        synchronized (this.postProcessingLock) {
            return (this.externallyManagedDestroyMethods != null &&
                    this.externallyManagedDestroyMethods.contains(destroyMethod));
        }
    }


    @Override
    public RootBeanDefinition cloneBeanDefinition() {
        return new RootBeanDefinition(this);
    }

    @Override
    public boolean equals(Object other) {
        return (this == other || (other instanceof RootBeanDefinition && super.equals(other)));
    }

    @Override
    public String toString() {
        return "Root bean: " + super.toString();
    }

}
```



