# EnableAutoConfigurationImportSelector 是如何工作的 ?

原文：https://blog.csdn.net/andy\_zhang2007/article/details/78580980

### 功能 {#功能}

EnableAutoConfigurationImportSelector 是一个DeferredImportSelector，由 spring boot autoconfigure 从版本1.3开始,提供用来处理EnableAutoConfiguration自动配置。

EnableAutoConfigurationImportSelector继承自AutoConfigurationImportSelector,从 spring boot 1.5 以后，EnableAutoConfigurationImportSelector已经不再被建议使用，而是推荐使用 AutoConfigurationImportSelector。

### 何时被引入 {#何时被引入}

Spring boot应用中使用了注解@SpringBootApplication，该注解隐含地导入了EnableAutoConfigurationImportSelector,如下注解依赖链所示 :

```
// 注解链
@SpringBootApplication
    => @EnableAutoConfiguration
        => @Import(EnableAutoConfigurationImportSelector.class)
```

### 何时被执行 {#何时被执行}

Springboot应用启动过程中使用ConfigurationClassParser分析配置类时，如果发现注解中存在@Import\(ImportSelector\)的情况，就会创建一个相应的ImportSelector对象， 并调用其方法 public String\[\] selectImports\(AnnotationMetadata annotationMetadata\), 这里 EnableAutoConfigurationImportSelector的导入@Import\(EnableAutoConfigurationImportSelector.class\) 就属于这种情况,所以ConfigurationClassParser会实例化一个 EnableAutoConfigurationImportSelector 并调用它的 selectImports\(\) 方法。

```
// selectImports 的具体执行逻辑
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
        try {
            // 从配置文件中加载 AutoConfigurationMetadata
            AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
                    .loadMetadata(this.beanClassLoader);
            AnnotationAttributes attributes = getAttributes(annotationMetadata);
            // 获取所有候选配置类EnableAutoConfiguration
            // 使用了内部工具使用SpringFactoriesLoader，查找classpath上所有jar包中的
            // META-INF\spring.factories，找出其中key为
            // org.springframework.boot.autoconfigure.EnableAutoConfiguration 
            // 的属性定义的工厂类名称。
            // 虽然参数有annotationMetadata,attributes,但在 AutoConfigurationImportSelector 的
            // 实现 getCandidateConfigurations()中，这两个参数并未使用
            List<String> configurations = getCandidateConfigurations(annotationMetadata,
                    attributes);
            // 去重                   
            configurations = removeDuplicates(configurations);
            // 排序 : 先按字典序，再按order属性，再考虑注解  @AutoConfigureBefore @AutoConfigureAfter
            configurations = sort(configurations, autoConfigurationMetadata);
            // 应用 exclusion 属性
            Set<String> exclusions = getExclusions(annotationMetadata, attributes);
            checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            // 应用过滤器AutoConfigurationImportFilter，
            // 对于 spring boot autoconfigure，定义了一个需要被应用的过滤器 ：
            // org.springframework.boot.autoconfigure.condition.OnClassCondition，
            // 此过滤器检查候选配置类上的注解@ConditionalOnClass，如果要求的类在classpath
            // 中不存在，则这个候选配置类会被排除掉
            configurations = filter(configurations, autoConfigurationMetadata);
            // 现在已经找到所有需要被应用的候选配置类
            // 广播事件 AutoConfigurationImportEvent
            fireAutoConfigurationImportEvents(configurations, exclusions);
            return configurations.toArray(new String[configurations.size()]);
        }
        catch (IOException ex) {
            throw new IllegalStateException(ex);
        }
    }
    /**
     * Return the auto-configuration class names that should be considered. By default
     * this method will load candidates using SpringFactoriesLoader with
     * getSpringFactoriesLoaderFactoryClass().
     * @param metadata the source metadata
     * @param attributes the getAttributes(AnnotationMetadata) annotation
     * attributes
     * @return a list of candidate configurations
     */
    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
            AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
                getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
        Assert.notEmpty(configurations,
                "No auto configuration classes found in META-INF/spring.factories. If you "
                        + "are using a custom packaging, make sure that file is correct.");
        return configurations;
    }

    /**
     * Return the class used by SpringFactoriesLoader to load configuration
     * candidates.
     * @return the factory class
     */
    protected Class<?> getSpringFactoriesLoaderFactoryClass() {
        return EnableAutoConfiguration.class;
    }   
    /**
      * 根据autoConfigurationMetadata信息对候选配置类configurations进行过滤
      **/
    private List<String> filter(List<String> configurations,
            AutoConfigurationMetadata autoConfigurationMetadata) {
        long startTime = System.nanoTime();
        String[] candidates = configurations.toArray(new String[configurations.size()]);
        // 记录候选配置类是否需要被排除,skip为true表示需要被排除,全部初始化为false,不需要被排除
        boolean[] skip = new boolean[candidates.length];
        // 记录候选配置类中是否有任何一个候选配置类被忽略，初始化为false
        boolean skipped = false;
        // 获取AutoConfigurationImportFilter并逐个应用过滤
        for (AutoConfigurationImportFilter filter : getAutoConfigurationImportFilters()) {
            // 对过滤器注入其需要Aware的信息
            invokeAwareMethods(filter);
            // 使用此过滤器检查候选配置类跟autoConfigurationMetadata的匹配情况
            boolean[] match = filter.match(candidates, autoConfigurationMetadata);
            for (int i = 0; i < match.length; i++) {
                if (!match[i]) {
                // 如果有某个候选配置类不符合当前过滤器，将其标记为需要被排除，
                // 并且将 skipped设置为true，表示发现了某个候选配置类需要被排除
                    skip[i] = true;
                    skipped = true;
                }
            }
        }
        if (!skipped) {
        // 如果所有的候选配置类都不需要被排除，则直接返回外部参数提供的候选配置类集合
            return configurations;
        }
        // 逻辑走到这里因为skipped为true，表明上面的的过滤器应用逻辑中发现了某些候选配置类
        // 需要被排除，这里排除那些需要被排除的候选配置类，将那些不需要被排除的候选配置类组成
        // 一个新的集合返回给调用者
        List<String> result = new ArrayList<String>(candidates.length);
        for (int i = 0; i < candidates.length; i++) {
            if (!skip[i]) {
                result.add(candidates[i]);
            }
        }
        if (logger.isTraceEnabled()) {
            int numberFiltered = configurations.size() - result.size();
            logger.trace("Filtered " + numberFiltered + " auto configuration class in "
                    + TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startTime)
                    + " ms");
        }
        return new ArrayList<String>(result);
    }   
    /**
      * 使用内部工具 SpringFactoriesLoader，查找classpath上所有jar包中的
      * META-INF\spring.factories，找出其中key为
      * org.springframework.boot.autoconfigure.AutoConfigurationImportFilter 
      * 的属性定义的过滤器类并实例化。
      * AutoConfigurationImportFilter过滤器可以被注册到 spring.factories用于对自动配置类
      * 做一些限制，在这些自动配置类的字节码被读取之前做快速排除处理。
      * spring boot autoconfigure 缺省注册了一个 AutoConfigurationImportFilter :
      * org.springframework.boot.autoconfigure.condition.OnClassCondition
    **/
    protected List<AutoConfigurationImportFilter> getAutoConfigurationImportFilters() {
        return SpringFactoriesLoader.loadFactories(AutoConfigurationImportFilter.class,
                this.beanClassLoader);
    }
```



