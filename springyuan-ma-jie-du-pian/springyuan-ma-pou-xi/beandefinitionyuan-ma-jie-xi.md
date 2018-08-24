# BeanDefinition源码解析

SpringIOC容器管理了我们定义的各种Bean对象及其相互的关系，Bean对象在Spring实现中是以BeanDefinition来描述的，其继承体系如下：

![](/assets/import-beande-01.png)

Bean 的解析过程非常复杂，功能被分的很细，因为这里需要被扩展的地方很多，必须保证有足够的灵活性，以应对可能的变化。Bean 的解析主要就是对 Spring 配置文件的解析。这个解析过程主要通过下图中的类完成：

![](/assets/import-beande-02.png)

* **AttributeAccessor**

```
//元数据操作接口
 public interface AttributeAccessor { 
  //设置元数据 
  void setAttribute(String name, Object value); 
  //获取元数据 
   Object getAttribute(String name); 
  //删除元数据 
   Object removeAttribute(String name); 
   //是否含有元数据
   boolean hasAttribute(String name);  
   //获取元数据的name数组
   String[] attributeNames();  
}
```

* **BeanMetadataElement**

```
//接口：用于承载bean对象
public interface BeanMetadataElement {
    //获取当前元素的配置源bean对象
    Object getSource();
}
```

* **BeanDefinition**

```
//用于描述一个具体bean实例
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
     //scope值，单例
    String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;

    //scope值，非单例
    String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;


    //Bean角色：
    //用户
    int ROLE_APPLICATION = 0;
    //某些复杂的配置
    int ROLE_SUPPORT = 1;
    //完全内部使用
    int ROLE_INFRASTRUCTURE = 2;


    //返回此bean定义的父bean定义的名称，如果有的话 <bean parent="">
    String getParentName();
    void setParentName(String parentName);

    //获取bean对象className <bean class="">
    String getBeanClassName();
    void setBeanClassName(String beanClassName);

    //定义创建该Bean对象的工厂l类  <bean factory-bean="">
    String getFactoryBeanName();
    void setFactoryBeanName(String factoryBeanName);


    //定义创建该Bean对象的工厂方法 <bean factory-method="">
    String getFactoryMethodName();
    void setFactoryMethodName(String factoryMethodName);


    //<bean scope="singleton/prototype">
    String getScope();
    void setScope(String scope);


    //懒加载 <bean lazy-init="true/false">
    boolean isLazyInit();
    void setLazyInit(boolean lazyInit);

    //依赖对象  <bean depends-on="">
    String[] getDependsOn();
    void setDependsOn(String[] dependsOn);


    //是否为被自动装配 <bean autowire-candidate="true/false">
    boolean isAutowireCandidate();
    void setAutowireCandidate(boolean autowireCandidate);

    //是否为主候选bean    使用注解：@Primary
    boolean isPrimary();
    void setPrimary(boolean primary);


    //返回此bean的构造函数参数值。
    ConstructorArgumentValues getConstructorArgumentValues();

    //获取普通属性集合
    MutablePropertyValues getPropertyValues();
    //是否为单例
    boolean isSingleton();
    //是否为原型
    boolean isPrototype();
    //是否为抽象类
    boolean isAbstract();

    //获取这个bean的应用
    int getRole();

    //返回对bean定义的可读描述。
    String getDescription();

    //返回该bean定义来自的资源的描述（用于在出现错误时显示上下文）
    String getResourceDescription();

    BeanDefinition getOriginatingBeanDefinition();
}
```

* AbstractBeanDefinition

```
//抽象类：基础bean，基石
@SuppressWarnings("serial")
public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor
        implements BeanDefinition, Cloneable {


    //第一部分：一些常量

    //默认scope值，bean的作用范围
    public static final String SCOPE_DEFAULT = "";

    //自动装配方式常量
    //不自动装配，需手动注入
    public static final int AUTOWIRE_NO = AutowireCapableBeanFactory.AUTOWIRE_NO;
    public static final int AUTOWIRE_BY_NAME = AutowireCapableBeanFactory.AUTOWIRE_BY_NAME;
    public static final int AUTOWIRE_BY_TYPE = AutowireCapableBeanFactory.AUTOWIRE_BY_TYPE;
    //按构造器参数类型自动装配（constructor跟byType十分相似.）
    public static final int AUTOWIRE_CONSTRUCTOR = AutowireCapableBeanFactory.AUTOWIRE_CONSTRUCTOR;
    //首先尝试使用constructor进行自动装配。如果失败，再尝试使用byType进行自动装配
    @Deprecated
    public static final int AUTOWIRE_AUTODETECT = AutowireCapableBeanFactory.AUTOWIRE_AUTODETECT;


    //依赖检查类型常量
    //依赖检查：无依赖
    public static final int DEPENDENCY_CHECK_NONE = 0;
    //依赖检查：对象间的引用
    public static final int DEPENDENCY_CHECK_OBJECTS = 1;
    ///依赖检查：会核对所有的原始类型和String类型的属性。
    public static final int DEPENDENCY_CHECK_SIMPLE = 2;
    // 依赖检查：所有属性。（对象引用及原始类型和String类型的属性）。
    public static final int DEPENDENCY_CHECK_ALL = 3;

    public static final String INFER_METHOD = "(inferred)";

    //第二部分，bean的属性描述

    //存放bean的class对象
    private volatile Object beanClass;
    //bean的作用范围
    private String scope = SCOPE_DEFAULT;
    //是否单例
    private boolean singleton = true;
    //是否原型
    private boolean prototype = false;
    //是否抽象
    private boolean abstractFlag = false;
    //是否延迟加载
    private boolean lazyInit = false;
    //默认不自动装配
    private int autowireMode = AUTOWIRE_NO;
    //默认依赖检查：无依赖
    private int dependencyCheck = DEPENDENCY_CHECK_NONE;
    //依赖列表
    private String[] dependsOn;
    //当前bean时候做为自动装配的候选者
    private boolean autowireCandidate = true;
    //默认为非主候选
    private boolean primary = false;
    //用于纪录qualifier，对应子元素Qualifier
    // @Qualifier 注释和 @Autowired 注释通过指定哪一个真正的 bean 将会被装配来消除混乱
    private final Map<String, AutowireCandidateQualifier> qualifiers =
            new LinkedHashMap<String, AutowireCandidateQualifier>(0);
    //允许访问非公开的构造器方法
    private boolean nonPublicAccessAllowed = true;
    /**
    * 是否以一种宽松的模式解析构造函数，默认为 true
    * 如果为false，则下面情况报错
    * interface ITest{}
    * class ITestImpl implements ITest{}
    * class Main{
    *   Main(ITest){}
    *   Main(ITestImpl){}
    * }
    */
    private boolean lenientConstructorResolution = true;

    /**
    * 记录构造函数注入属性，如：
    *<bean id="student" class="com.rc.sp.Student"> 
    *  <constructor-arg name="id" value="1"/>
    *  <constructor-arg name="score">
    *  <map>
    *      <entry key="math" value="90"/>
    *        <entry key="english" value="85"/>
    *    </map>
    *  </constructor-arg>
    */</bean>
    private ConstructorArgumentValues constructorArgumentValues;

    /**
    * 普通属性的集合，如：
    * <bean id="student" class="com.rc.sp.Student"> 
    *   <property name="id" value="1"/>
    *   <property name="dao" ref="dao" />
    *   <property name="map">
    *     <map>
    *       <entry key="math" value="90"/>
    *       <entry key="english" value="85"/>
    *     </map>
    *   </property>
    * </bean>
    */
    private MutablePropertyValues propertyValues;
    //方法重写的持有者，记录 lookup-method，replaced-method元素
    private MethodOverrides methodOverrides = new MethodOverrides();
    //构造当前实例工厂类名称
    private String factoryBeanName;
    //构造当前实例工厂类中方法名称
    private String factoryMethodName;
    //初始化方法
    private String initMethodName;
    //bean被销毁时，调用的方法
    private String destroyMethodName;
    //是否执行initMethod，程序设置
    private boolean enforceInitMethod = true;
    //是否执行destroyMethod，程序设置
    private boolean enforceDestroyMethod = true;

    private boolean synthetic = false;
    //Bean角色
    private int role = BeanDefinition.ROLE_APPLICATION;
    //bean的描述信息
    private String description;
    //该bean定义来自的资源
    private Resource resource;


    //空构造方法
    protected AbstractBeanDefinition() {
        this(null, null);
    }

    //指定的构造函数的参数值和属性值
    protected AbstractBeanDefinition(ConstructorArgumentValues cargs, MutablePropertyValues pvs) {
        setConstructorArgumentValues(cargs);
        setPropertyValues(pvs);
    }

    //深拷贝构造
    @Deprecated
    protected AbstractBeanDefinition(AbstractBeanDefinition original) {
        this((BeanDefinition) original);
    }

    //深拷贝构造
    protected AbstractBeanDefinition(BeanDefinition original) {
        setParentName(original.getParentName());
        setBeanClassName(original.getBeanClassName());
        setFactoryBeanName(original.getFactoryBeanName());
        setFactoryMethodName(original.getFactoryMethodName());
        setScope(original.getScope());
        setAbstract(original.isAbstract());
        setLazyInit(original.isLazyInit());
        setRole(original.getRole());
        setConstructorArgumentValues(new ConstructorArgumentValues(original.getConstructorArgumentValues()));
        setPropertyValues(new MutablePropertyValues(original.getPropertyValues()));
        setSource(original.getSource());
        copyAttributesFrom(original);

        if (original instanceof AbstractBeanDefinition) {
            AbstractBeanDefinition originalAbd = (AbstractBeanDefinition) original;
            if (originalAbd.hasBeanClass()) {
                setBeanClass(originalAbd.getBeanClass());
            }
            setAutowireMode(originalAbd.getAutowireMode());
            setDependencyCheck(originalAbd.getDependencyCheck());
            setDependsOn(originalAbd.getDependsOn());
            setAutowireCandidate(originalAbd.isAutowireCandidate());
            copyQualifiersFrom(originalAbd);
            setPrimary(originalAbd.isPrimary());
            setNonPublicAccessAllowed(originalAbd.isNonPublicAccessAllowed());
            setLenientConstructorResolution(originalAbd.isLenientConstructorResolution());
            setInitMethodName(originalAbd.getInitMethodName());
            setEnforceInitMethod(originalAbd.isEnforceInitMethod());
            setDestroyMethodName(originalAbd.getDestroyMethodName());
            setEnforceDestroyMethod(originalAbd.isEnforceDestroyMethod());
            setMethodOverrides(new MethodOverrides(originalAbd.getMethodOverrides()));
            setSynthetic(originalAbd.isSynthetic());
            setResource(originalAbd.getResource());
        }
        else {
            setResourceDescription(original.getResourceDescription());
        }
    }


    // 从给定的bean定义（大概是子）中覆盖此bean定义（可能是来自父子继承关系的复制父）的设置。
    @Deprecated
    public void overrideFrom(AbstractBeanDefinition other) {
        overrideFrom((BeanDefinition) other);
    }

    // 从给定的bean定义（大概是子）中覆盖此bean定义（可能是来自父子继承关系的复制父）的设置。
    public void overrideFrom(BeanDefinition other) {
        //不为空时
        if (StringUtils.hasLength(other.getBeanClassName())) {
            setBeanClassName(other.getBeanClassName());
        }
        //不为空时
        if (StringUtils.hasLength(other.getFactoryBeanName())) {
            setFactoryBeanName(other.getFactoryBeanName());
        }
        //不为空时
        if (StringUtils.hasLength(other.getFactoryMethodName())) {
            setFactoryMethodName(other.getFactoryMethodName());
        }
        //不为空时
        if (StringUtils.hasLength(other.getScope())) {
            setScope(other.getScope());
        }
        setAbstract(other.isAbstract());
        setLazyInit(other.isLazyInit());
        setRole(other.getRole());
        //更新已存在的，新增未拥有的
        getConstructorArgumentValues().addArgumentValues(other.getConstructorArgumentValues());
        getPropertyValues().addPropertyValues(other.getPropertyValues());
        setSource(other.getSource());
        copyAttributesFrom(other);
        //更新AbstractBeanDefinition中特有的属性
        if (other instanceof AbstractBeanDefinition) {
            AbstractBeanDefinition otherAbd = (AbstractBeanDefinition) other;
            if (otherAbd.hasBeanClass()) {
                setBeanClass(otherAbd.getBeanClass());
            }
            setAutowireCandidate(otherAbd.isAutowireCandidate());
            setAutowireMode(otherAbd.getAutowireMode());
            copyQualifiersFrom(otherAbd);
            setPrimary(otherAbd.isPrimary());
            setDependencyCheck(otherAbd.getDependencyCheck());
            setDependsOn(otherAbd.getDependsOn());
            setNonPublicAccessAllowed(otherAbd.isNonPublicAccessAllowed());
            setLenientConstructorResolution(otherAbd.isLenientConstructorResolution());
            if (StringUtils.hasLength(otherAbd.getInitMethodName())) {
                setInitMethodName(otherAbd.getInitMethodName());
                setEnforceInitMethod(otherAbd.isEnforceInitMethod());
            }
            if (StringUtils.hasLength(otherAbd.getDestroyMethodName())) {
                setDestroyMethodName(otherAbd.getDestroyMethodName());
                setEnforceDestroyMethod(otherAbd.isEnforceDestroyMethod());
            }
            getMethodOverrides().addOverrides(otherAbd.getMethodOverrides());
            setSynthetic(otherAbd.isSynthetic());
            setResource(otherAbd.getResource());
        }
        else {
            //更新资源描述
            setResourceDescription(other.getResourceDescription());
        }
    }


    //将所提供的默认值应用于此bean。
    public void applyDefaults(BeanDefinitionDefaults defaults) {
        setLazyInit(defaults.isLazyInit());
        setAutowireMode(defaults.getAutowireMode());
        setDependencyCheck(defaults.getDependencyCheck());
        setInitMethodName(defaults.getInitMethodName());
        setEnforceInitMethod(false);
        setDestroyMethodName(defaults.getDestroyMethodName());
        setEnforceDestroyMethod(false);
    }


    //操作beanclass对象
    //是否已经设置
    public boolean hasBeanClass() {
        return (this.beanClass instanceof Class);
    }
    //设置class对象
    public void setBeanClass(Class<?> beanClass) {
        this.beanClass = beanClass;
    }
    //若已经解析就返回这个包装类。
    public Class<?> getBeanClass() throws IllegalStateException {
        Object beanClassObject = this.beanClass;
        if (beanClassObject == null) {
            throw new IllegalStateException("No bean class specified on bean definition");
        }
        if (!(beanClassObject instanceof Class)) {
            throw new IllegalStateException(
                    "Bean class name [" + beanClassObject + "] has not been resolved into an actual Class");
        }
        return (Class<?>) beanClassObject;
    }
    //设置className
    public void setBeanClassName(String beanClassName) {
        this.beanClass = beanClassName;
    }
    //获取设置className
    public String getBeanClassName() {
        Object beanClassObject = this.beanClass;
        if (beanClassObject instanceof Class) {
            return ((Class<?>) beanClassObject).getName();
        }
        else {
            return (String) beanClassObject;
        }
    }

    //解析类
    public Class<?> resolveBeanClass(ClassLoader classLoader) throws ClassNotFoundException {
        String className = getBeanClassName();
        if (className == null) {
            return null;
        }
        Class<?> resolvedClass = ClassUtils.forName(className, classLoader);
        this.beanClass = resolvedClass;
        return resolvedClass;
    }

    //scope操作
    public void setScope(String scope) {
        this.scope = scope;
        this.singleton = SCOPE_SINGLETON.equals(scope) || SCOPE_DEFAULT.equals(scope);
        this.prototype = SCOPE_PROTOTYPE.equals(scope);
    }
    public String getScope() {
        return this.scope;
    }


    //设置是否单例
    @Deprecated
    public void setSingleton(boolean singleton) {
        this.scope = (singleton ? SCOPE_SINGLETON : SCOPE_PROTOTYPE);
        this.singleton = singleton;
        this.prototype = !singleton;
    }
    public boolean isSingleton() {
        return this.singleton;
    }

    //是否为原型
    public boolean isPrototype() {
        return this.prototype;
    }


    //设置是否抽象
    public void setAbstract(boolean abstractFlag) {
        this.abstractFlag = abstractFlag;
    }
    public boolean isAbstract() {
        return this.abstractFlag;
    }


    //设置是否懒加载
    public void setLazyInit(boolean lazyInit) {
        this.lazyInit = lazyInit;
    }
    public boolean isLazyInit() {
        return this.lazyInit;
    }


    //设置自动装配方式
    public void setAutowireMode(int autowireMode) {
        this.autowireMode = autowireMode;
    }
    public int getAutowireMode() {
        return this.autowireMode;
    }

    /**
     * Return the resolved autowire code,
     * (resolving AUTOWIRE_AUTODETECT to AUTOWIRE_CONSTRUCTOR or AUTOWIRE_BY_TYPE).
     * @see #AUTOWIRE_AUTODETECT
     * @see #AUTOWIRE_CONSTRUCTOR
     * @see #AUTOWIRE_BY_TYPE
     */
    public int getResolvedAutowireMode() {
        if (this.autowireMode == AUTOWIRE_AUTODETECT) {
            // Work out whether to apply setter autowiring or constructor autowiring.
            // If it has a no-arg constructor it's deemed to be setter autowiring,
            // otherwise we'll try constructor autowiring.
            Constructor<?>[] constructors = getBeanClass().getConstructors();
            for (Constructor<?> constructor : constructors) {
                if (constructor.getParameterTypes().length == 0) {
                    return AUTOWIRE_BY_TYPE;
                }
            }
            return AUTOWIRE_CONSTRUCTOR;
        }
        else {
            return this.autowireMode;
        }
    }

    //设置依赖检查
    public void setDependencyCheck(int dependencyCheck) {
        this.dependencyCheck = dependencyCheck;
    }
    public int getDependencyCheck() {
        return this.dependencyCheck;
    }

    //设置依赖
    public void setDependsOn(String[] dependsOn) {
        this.dependsOn = dependsOn;
    }
    public String[] getDependsOn() {
        return this.dependsOn;
    }


    //设置是否做为其他bean自动装配的候选者
    public void setAutowireCandidate(boolean autowireCandidate) {
        this.autowireCandidate = autowireCandidate;
    }
    public boolean isAutowireCandidate() {
        return this.autowireCandidate;
    }

    //设置是否做为主候选
    public void setPrimary(boolean primary) {
        this.primary = primary;
    }
    public boolean isPrimary() {
        return this.primary;
    }

    //设置Qualifier
    public void addQualifier(AutowireCandidateQualifier qualifier) {
        this.qualifiers.put(qualifier.getTypeName(), qualifier);
    }
    public boolean hasQualifier(String typeName) {
        return this.qualifiers.keySet().contains(typeName);
    }
    public AutowireCandidateQualifier getQualifier(String typeName) {
        return this.qualifiers.get(typeName);
    }
    /**
     * Return all registered qualifiers.
     * @return the Set of {@link AutowireCandidateQualifier} objects.
     */
    public Set<AutowireCandidateQualifier> getQualifiers() {
        return new LinkedHashSet<AutowireCandidateQualifier>(this.qualifiers.values());
    }

    //复制给定source的qualifiers到当前bean（覆盖已有的）
    public void copyQualifiersFrom(AbstractBeanDefinition source) {
        Assert.notNull(source, "Source must not be null");
        this.qualifiers.putAll(source.qualifiers);
    }


    //设置是否允许访问非公开的构造方法
    public void setNonPublicAccessAllowed(boolean nonPublicAccessAllowed) {
        this.nonPublicAccessAllowed = nonPublicAccessAllowed;
    }
    public boolean isNonPublicAccessAllowed() {
        return this.nonPublicAccessAllowed;
    }

    //设置是否以一种宽松的方式解析构造参数
    public void setLenientConstructorResolution(boolean lenientConstructorResolution) {
        this.lenientConstructorResolution = lenientConstructorResolution;
    }
    public boolean isLenientConstructorResolution() {
        return this.lenientConstructorResolution;
    }


    //记录构造函数的注入属性
    public void setConstructorArgumentValues(ConstructorArgumentValues constructorArgumentValues) {
        this.constructorArgumentValues =
                (constructorArgumentValues != null ? constructorArgumentValues : new ConstructorArgumentValues());
    }
    public ConstructorArgumentValues getConstructorArgumentValues() {
        return this.constructorArgumentValues;
    }
    public boolean hasConstructorArgumentValues() {
        return !this.constructorArgumentValues.isEmpty();
    }


    //记录普通的属性集合
    public void setPropertyValues(MutablePropertyValues propertyValues) {
        this.propertyValues = (propertyValues != null ? propertyValues : new MutablePropertyValues());
    }
    public MutablePropertyValues getPropertyValues() {
        return this.propertyValues;
    }
    public void setMethodOverrides(MethodOverrides methodOverrides) {
        this.methodOverrides = (methodOverrides != null ? methodOverrides : new MethodOverrides());
    }
    public MethodOverrides getMethodOverrides() {
        return this.methodOverrides;
    }

    //工厂类操作
    public void setFactoryBeanName(String factoryBeanName) {
        this.factoryBeanName = factoryBeanName;
    }
    public String getFactoryBeanName() {
        return this.factoryBeanName;
    }
    public void setFactoryMethodName(String factoryMethodName) {
        this.factoryMethodName = factoryMethodName;
    }
    public String getFactoryMethodName() {
        return this.factoryMethodName;
    }

    //初始化方法和销毁方法操作
    public void setInitMethodName(String initMethodName) {
        this.initMethodName = initMethodName;
    }
    public String getInitMethodName() {
        return this.initMethodName;
    }
    public void setEnforceInitMethod(boolean enforceInitMethod) {
        this.enforceInitMethod = enforceInitMethod;
    }
    public boolean isEnforceInitMethod() {
        return this.enforceInitMethod;
    }
    public void setDestroyMethodName(String destroyMethodName) {
        this.destroyMethodName = destroyMethodName;
    }
    public String getDestroyMethodName() {
        return this.destroyMethodName;
    }
    public void setEnforceDestroyMethod(boolean enforceDestroyMethod) {
        this.enforceDestroyMethod = enforceDestroyMethod;
    }
    public boolean isEnforceDestroyMethod() {
        return this.enforceDestroyMethod;
    }

    /**
    * 设置是否这个bean的定义是“合成”，不是由应用程序定义的
     * Set whether this bean definition is 'synthetic', that is, not defined
     * by the application itself (for example, an infrastructure bean such
     * as a helper for auto-proxying, created through {@code &ltaop:config&gt;}).
     */
    public void setSynthetic(boolean synthetic) {
        this.synthetic = synthetic;
    }
    public boolean isSynthetic() {
        return this.synthetic;
    }

    //设置role
    public void setRole(int role) {
        this.role = role;
    }
    //获取role
    public int getRole() {
        return this.role;
    }


    //为这个bean定义一个可读的描述。
    public void setDescription(String description) {
        this.description = description;
    }
    //获取bean的描述
    public String getDescription() {
        return this.description;
    }

    //设置此bean定义的资源（为了在出错时显示上下文）。
    public void setResource(Resource resource) {
        this.resource = resource;
    }
    //获取此bean定义的资源（为了在出错时显示上下文）。
    public Resource getResource() {
        return this.resource;
    }

    //设置该bean定义的资源的描述（用于显示错误情况下的上下文）。
    public void setResourceDescription(String resourceDescription) {
        this.resource = new DescriptiveResource(resourceDescription);
    }
    public String getResourceDescription() {
        return (this.resource != null ? this.resource.getDescription() : null);
    }


    //设置源(e.g. decorated）BeanDefinition，如果任何。
    public void setOriginatingBeanDefinition(BeanDefinition originatingBd) {
        this.resource = new BeanDefinitionResource(originatingBd);
    }

    public BeanDefinition getOriginatingBeanDefinition() {
        return (this.resource instanceof BeanDefinitionResource ?
                ((BeanDefinitionResource) this.resource).getBeanDefinition() : null);
    }

    //验证当前bean的定义
    public void validate() throws BeanDefinitionValidationException {
        //无法将静态工厂方法与方法重写组合在一起
        if (!getMethodOverrides().isEmpty() && getFactoryMethodName() != null) {
            throw new BeanDefinitionValidationException(
                    "Cannot combine static factory method with method overrides: " +
                    "the static factory method must create the instance");
        }
        //已经设置beanClass属性，校验
        if (hasBeanClass()) {
            prepareMethodOverrides();
        }
    }


    //开始准备验证这个bean定义的重写方法，检查指定名称的方法是否存在。
    public void prepareMethodOverrides() throws BeanDefinitionValidationException {
        //检测当前lookup methods是否存在
        MethodOverrides methodOverrides = getMethodOverrides();
        if (!methodOverrides.isEmpty()) {
            for (MethodOverride mo : methodOverrides.getOverrides()) {
                prepareMethodOverride(mo);
            }
        }
    }

    //准备验证给定的重写方法
    //检查指定名称的方法是否存在，如果没有找到，则标记为未重载。
    protected void prepareMethodOverride(MethodOverride mo) throws BeanDefinitionValidationException {
        int count = ClassUtils.getMethodCountForName(getBeanClass(), mo.getMethodName());
        //class对象中未找到当前方法，报错
        if (count == 0) {
            throw new BeanDefinitionValidationException(
                    "Invalid method override: no method with name '" + mo.getMethodName() +
                    "' on class [" + getBeanClassName() + "]");
        }
        else if (count == 1) {
            //标记不是重载，以避免类型检查的开销。
            mo.setOverloaded(false);
        }
    }


    /**
     * Public declaration of Object's {@code clone()} method.
     * Delegates to {@link #cloneBeanDefinition()}.
     * @see Object#clone()
     */
    @Override
    public Object clone() {
        return cloneBeanDefinition();
    }

    /**
     * Clone this bean definition.
     * To be implemented by concrete subclasses.
     * @return the cloned bean definition object
     */
    public abstract AbstractBeanDefinition cloneBeanDefinition();


    @Override
    public boolean equals(Object other) {
        if (this == other) {
            return true;
        }
        if (!(other instanceof AbstractBeanDefinition)) {
            return false;
        }

        AbstractBeanDefinition that = (AbstractBeanDefinition) other;

        if (!ObjectUtils.nullSafeEquals(getBeanClassName(), that.getBeanClassName())) return false;
        if (!ObjectUtils.nullSafeEquals(this.scope, that.scope)) return false;
        if (this.abstractFlag != that.abstractFlag) return false;
        if (this.lazyInit != that.lazyInit) return false;

        if (this.autowireMode != that.autowireMode) return false;
        if (this.dependencyCheck != that.dependencyCheck) return false;
        if (!Arrays.equals(this.dependsOn, that.dependsOn)) return false;
        if (this.autowireCandidate != that.autowireCandidate) return false;
        if (!ObjectUtils.nullSafeEquals(this.qualifiers, that.qualifiers)) return false;
        if (this.primary != that.primary) return false;

        if (this.nonPublicAccessAllowed != that.nonPublicAccessAllowed) return false;
        if (this.lenientConstructorResolution != that.lenientConstructorResolution) return false;
        if (!ObjectUtils.nullSafeEquals(this.constructorArgumentValues, that.constructorArgumentValues)) return false;
        if (!ObjectUtils.nullSafeEquals(this.propertyValues, that.propertyValues)) return false;
        if (!ObjectUtils.nullSafeEquals(this.methodOverrides, that.methodOverrides)) return false;

        if (!ObjectUtils.nullSafeEquals(this.factoryBeanName, that.factoryBeanName)) return false;
        if (!ObjectUtils.nullSafeEquals(this.factoryMethodName, that.factoryMethodName)) return false;
        if (!ObjectUtils.nullSafeEquals(this.initMethodName, that.initMethodName)) return false;
        if (this.enforceInitMethod != that.enforceInitMethod) return false;
        if (!ObjectUtils.nullSafeEquals(this.destroyMethodName, that.destroyMethodName)) return false;
        if (this.enforceDestroyMethod != that.enforceDestroyMethod) return false;

        if (this.synthetic != that.synthetic) return false;
        if (this.role != that.role) return false;

        return super.equals(other);
    }

    @Override
    public int hashCode() {
        int hashCode = ObjectUtils.nullSafeHashCode(getBeanClassName());
        hashCode = 29 * hashCode + ObjectUtils.nullSafeHashCode(this.scope);
        hashCode = 29 * hashCode + ObjectUtils.nullSafeHashCode(this.constructorArgumentValues);
        hashCode = 29 * hashCode + ObjectUtils.nullSafeHashCode(this.propertyValues);
        hashCode = 29 * hashCode + ObjectUtils.nullSafeHashCode(this.factoryBeanName);
        hashCode = 29 * hashCode + ObjectUtils.nullSafeHashCode(this.factoryMethodName);
        hashCode = 29 * hashCode + super.hashCode();
        return hashCode;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("class [");
        sb.append(getBeanClassName()).append("]");
        sb.append("; scope=").append(this.scope);
        sb.append("; abstract=").append(this.abstractFlag);
        sb.append("; lazyInit=").append(this.lazyInit);
        sb.append("; autowireMode=").append(this.autowireMode);
        sb.append("; dependencyCheck=").append(this.dependencyCheck);
        sb.append("; autowireCandidate=").append(this.autowireCandidate);
        sb.append("; primary=").append(this.primary);
        sb.append("; factoryBeanName=").append(this.factoryBeanName);
        sb.append("; factoryMethodName=").append(this.factoryMethodName);
        sb.append("; initMethodName=").append(this.initMethodName);
        sb.append("; destroyMethodName=").append(this.destroyMethodName);
        if (this.resource != null) {
            sb.append("; defined in ").append(this.resource.getDescription());
        }
        return sb.toString();
    }
}
```

* **ChildBeanDefinition**

在配置文件中可以定义父和子，父用RootBeanDefinition表示，而子用ChildBeanDefiniton表示，而没有父的就使用RootBeanDefinition表示

```
public class ChildBeanDefinition extends AbstractBeanDefinition {
    //父bean名称
    private String parentName;

    //构造方法
    public ChildBeanDefinition(String parentName) {
        super();
        this.parentName = parentName;
    }

    //构造方法
    public ChildBeanDefinition(String parentName, MutablePropertyValues pvs) {
        super(null, pvs);
        this.parentName = parentName;
    }

    //构造方法
    public ChildBeanDefinition(
            String parentName, ConstructorArgumentValues cargs, MutablePropertyValues pvs) {

        super(cargs, pvs);
        this.parentName = parentName;
    }

    //构造方法
    public ChildBeanDefinition(
            String parentName, Class<?> beanClass, ConstructorArgumentValues cargs, MutablePropertyValues pvs) {

        super(cargs, pvs);
        this.parentName = parentName;
        setBeanClass(beanClass);
    }
    //构造方法
    public ChildBeanDefinition(
            String parentName, String beanClassName, ConstructorArgumentValues cargs, MutablePropertyValues pvs) {

        super(cargs, pvs);
        this.parentName = parentName;
        setBeanClassName(beanClassName);
    }

    //构造方法
    public ChildBeanDefinition(ChildBeanDefinition original) {
        super((BeanDefinition) original);
    }

    //设置parentName
    public void setParentName(String parentName) {
        this.parentName = parentName;
    }
    public String getParentName() {
        return this.parentName;
    }
    //验证当前bean定义
    @Override
    public void validate() throws BeanDefinitionValidationException {
        super.validate();
        if (this.parentName == null) {
            throw new BeanDefinitionValidationException("'parentName' must be set in ChildBeanDefinition");
        }
    }

    //深度拷贝bean定义
    @Override
    public AbstractBeanDefinition cloneBeanDefinition() {
        return new ChildBeanDefinition(this);
    }

    @Override
    public boolean equals(Object other) {
        if (this == other) {
            return true;
        }
        if (!(other instanceof ChildBeanDefinition)) {
            return false;
        }
        ChildBeanDefinition that = (ChildBeanDefinition) other;
        return (ObjectUtils.nullSafeEquals(this.parentName, that.parentName) && super.equals(other));
    }

    @Override
    public int hashCode() {
        return ObjectUtils.nullSafeHashCode(this.parentName) * 29 + super.hashCode();
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("Child bean with parent '");
        sb.append(this.parentName).append("': ").append(super.toString());
        return sb.toString();
    }

}
```

* **GenericBeanDefinition**

GenericBeanDefinition是自2.5以后新加入的bean文件配置属性定义类，是一站式服务类，用于替代ChildBeanDefinition类

```
public class GenericBeanDefinition extends AbstractBeanDefinition {
    //父bean名称
    private String parentName;
    //构造方法
    public GenericBeanDefinition() {
        super();
    }
    //构造方法
    public GenericBeanDefinition(BeanDefinition original) {
        super(original);
    }
    //设置parentName
    public void setParentName(String parentName) {
        this.parentName = parentName;
    }
    public String getParentName() {
        return this.parentName;
    }
    //深度拷贝自身
    @Override
    public AbstractBeanDefinition cloneBeanDefinition() {
        return new GenericBeanDefinition(this);
    }

    @Override
    public boolean equals(Object other) {
        return (this == other || (other instanceof GenericBeanDefinition && super.equals(other)));
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("Generic bean");
        if (this.parentName != null) {
            sb.append(" with parent '").append(this.parentName).append("'");
        }
        sb.append(": ").append(super.toString());
        return sb.toString();
    }

}
```



