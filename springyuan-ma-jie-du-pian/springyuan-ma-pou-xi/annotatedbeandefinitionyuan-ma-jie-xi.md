# AnnotatedBeanDefinition源码解析

```
//接口:支持注释元数据的bean定义
public interface AnnotatedBeanDefinition extends BeanDefinition {

    //获取此bean定义bean类的注释元数据（以及基本类元数据）。
    AnnotationMetadata getMetadata();

}

//实现类一：支持注释元数据的普通bean定义
public class AnnotatedGenericBeanDefinition extends GenericBeanDefinition implements AnnotatedBeanDefinition {

    //当前bean的注解元数据
    private final AnnotationMetadata metadata;

    //通过给定beanClass 构造对象
    public AnnotatedGenericBeanDefinition(Class<?> beanClass) {
        setBeanClass(beanClass);
        this.metadata = new StandardAnnotationMetadata(beanClass, true);
    }

    //给定注解元数据，构造对象
    public AnnotatedGenericBeanDefinition(AnnotationMetadata metadata) {
        Assert.notNull(metadata, "AnnotationMetadata must not be null");
        if (metadata instanceof StandardAnnotationMetadata) {
            setBeanClass(((StandardAnnotationMetadata) metadata).getIntrospectedClass());
        }
        else {
            setBeanClassName(metadata.getClassName());
        }
        this.metadata = metadata;
    }

    //获取bean的注解元数据
    public final AnnotationMetadata getMetadata() {
         return this.metadata;
    }

}

//实现类二：支持注释元数据bean定义
//通过 Annotation 配置方式定义的 Bean 属性经 Spring 框架解析后会封装成 ScannedGenericBeanDefinition 
public class ScannedGenericBeanDefinition extends GenericBeanDefinition implements AnnotatedBeanDefinition {
    //当前bean的注解元数据
    private final AnnotationMetadata metadata;

    //metadatareader:扫描指定类的注释元数据
    public ScannedGenericBeanDefinition(MetadataReader metadataReader) {
        Assert.notNull(metadataReader, "MetadataReader must not be null");
        this.metadata = metadataReader.getAnnotationMetadata();
        setBeanClassName(this.metadata.getClassName());
    }

    //获取bean的注解元数据
    public final AnnotationMetadata getMetadata() {
        return this.metadata;
    }
}
```



