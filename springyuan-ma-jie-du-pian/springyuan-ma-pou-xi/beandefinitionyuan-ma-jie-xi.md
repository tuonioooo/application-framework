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

* **BeanDefinition**

```
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
    String SCOPE_SINGLETON = "singleton";
    String SCOPE_PROTOTYPE = "prototype";
    int ROLE_APPLICATION = 0;
    int ROLE_SUPPORT = 1;
    int ROLE_INFRASTRUCTURE = 2;

    void setParentName(String var1);

    String getParentName();

    void setBeanClassName(String var1);

    String getBeanClassName();

    void setScope(String var1);

    String getScope();

    void setLazyInit(boolean var1);

    boolean isLazyInit();

    void setDependsOn(String... var1);

    String[] getDependsOn();

    void setAutowireCandidate(boolean var1);

    boolean isAutowireCandidate();

    void setPrimary(boolean var1);

    boolean isPrimary();

    void setFactoryBeanName(String var1);

    String getFactoryBeanName();

    void setFactoryMethodName(String var1);

    String getFactoryMethodName();

    ConstructorArgumentValues getConstructorArgumentValues();

    MutablePropertyValues getPropertyValues();

    boolean isSingleton();

    boolean isPrototype();

    boolean isAbstract();

    int getRole();

    String getDescription();

    String getResourceDescription();

    BeanDefinition getOriginatingBeanDefinition();
}
```



