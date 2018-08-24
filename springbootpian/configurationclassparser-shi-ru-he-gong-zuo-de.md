# ConfigurationClassParser 是如何工作的 ？

#### 简介 {#简介}

Spring的工具类 ConfigurationClassParser 用于分析一个 @Configuration 注解的类，产生一组ConfigurationClass对象。其分析过程主要是递归分析注解中隐含的@Import，找出其中所有的配置类，然后返回这组配置类。

**这个工具类自身的逻辑并不会注册Bean定义，但是它用到了另外一个工具ComponentScanAnnotationParser在执行时会做Bean定义注册。**

一般情况下一个@Configuration注解的类只会产生一个ConfigurationClass对象，但是因为@Configuration注解的类可能会使用另外一个注解@Import引入其他配置类，所以总的来看，ConfigurationClassParser分析一个  
@Configuration注解的类，可能产生任意多个ConfigurationClass对象。

这个工具类分离了两个不同点关注点 :

* @Configuration注解的类的结构分析
* 注册BeanDefinition对象

#### 主要功能分析 {#主要功能分析}

##### 公开方法 parse\(\) 外部调用入口 {#公开方法-parse-外部调用入口}

```
/**
     * 总体工作流程 :
     *  1. 外部提供参数 configCandidates , 是一组需要被分析的候选配置类 ;
     *  2. 调用该类的 parse() 方法， parse() 方法针对每个候选配置类，递归查找其注解@Import,
     *     以及注解的注解中的@Import，然后处理这些@Import，每个@Import可能对应一个ImportSelector,
     *     ImportBeanDefinitionRegistrar,或者是另外一个@Configuration注解的配置类，parse()会
     *     递归处理这些进一步的导入，直到所有所有的这些配置类都被发现，这些发现的每个ConfigurationClass，
     *     会被记录到 configurationClasses;
     *  3. 外部通过调用该类的方法 getConfigurationClasses() 获得分析过程中保存在 configurationClasses
     *     中的配置类；
     * @参数 configCandidates : 外部指定需要被分析的一组候选配置类
     **/
    public void parse(Set<BeanDefinitionHolder> configCandidates) {
        this.deferredImportSelectors = new LinkedList<DeferredImportSelectorHolder>();

        for (BeanDefinitionHolder holder : configCandidates) {
            BeanDefinition bd = holder.getBeanDefinition();
            try {
                // 这里根据Bean定义的不同类型走不同的分支，但是最终都会调用到方法
                //  processConfigurationClass(ConfigurationClass configClass)
                if (bd instanceof AnnotatedBeanDefinition) {
                    // bd 是一个 AnnotatedBeanDefinition
                    parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
                }
                else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
                    // bd 是一个 AbstractBeanDefinition,并且指定 beanClass 属性
                    parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
                }
                else {
                    // 其他情况
                    parse(bd.getBeanClassName(), holder.getBeanName());
                }
            }
            catch (BeanDefinitionStoreException ex) {
                throw ex;
            }
            catch (Throwable ex) {
                throw new BeanDefinitionStoreException(
                        "Failed to parse configuration class [" + bd.getBeanClassName() + "]", ex);
            }
        }

        // 执行找到的 DeferredImportSelector 
        processDeferredImportSelectors();
    }
```

##### 保护方法 processConfigurationClass\(\)分析一个配置类 {#保护方法-processconfigurationclass分析一个配置类}

```
/**
     *  用于分析一个 ConfigurationClass，分析之后将它记录到已处理配置类记录
     **/
    protected void processConfigurationClass(ConfigurationClass configClass) throws IOException {
        if (this.conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {
            return;
        }

        ConfigurationClass existingClass = this.configurationClasses.get(configClass);
        if (existingClass != null) {
            if (configClass.isImported()) {
                if (existingClass.isImported()) {
                    //如果要处理的配置类configClass在已经分析处理的配置类记录中已存在，
                    //合并二者的importedBy属性
                    existingClass.mergeImportedBy(configClass);
                }
                // Otherwise ignore new imported config class; existing non-imported class overrides it.
                return;
            }
            else {
                // Explicit bean definition found, probably replacing an import.
                // Let's remove the old one and go with the new one.
                this.configurationClasses.remove(configClass);
                for (Iterator<ConfigurationClass> it = this.knownSuperclasses.values().iterator(); it.hasNext();) {
                    if (configClass.equals(it.next())) {
                        it.remove();
                    }
                }
            }
        }

        // Recursively process the configuration class and its superclass hierarchy.
        // 从当前配置类configClass开始向上沿着类继承结构逐层执行doProcessConfigurationClass,
        // 直到遇到的父类是由Java提供的类结束循环
        SourceClass sourceClass = asSourceClass(configClass);
        do {            
            // 循环处理配置类configClass直到sourceClass变为null
            // doProcessConfigurationClass的返回值是其参数configClass的父类，
            // 如果该父类是由Java提供的类或者已经处理过，返回null
            sourceClass = doProcessConfigurationClass(configClass, sourceClass);
        }
        while (sourceClass != null);

        // 需要被处理的配置类configClass已经被分析处理，将它记录到已处理配置类记录
        this.configurationClasses.put(configClass, configClass);
    }
```

##### 私有方法 processDeferredImportSelectors\(\) 处理需要延迟处理的ImportSelector {#私有方法-processdeferredimportselectors-处理需要延迟处理的importselector}

```
/**
      * 对属性deferredImportSelectors中记录的DeferredImportSelector进行处理
      **/
    private void processDeferredImportSelectors() {
        List<DeferredImportSelectorHolder> deferredImports = this.deferredImportSelectors;
        this.deferredImportSelectors = null;
        Collections.sort(deferredImports, DEFERRED_IMPORT_COMPARATOR);

        // 循环处理每个DeferredImportSelector
        for (DeferredImportSelectorHolder deferredImport : deferredImports) {
            ConfigurationClass configClass = deferredImport.getConfigurationClass();
            try {
                //调用DeferredImportSelector的方法selectImports,获取需要被导入的类的名称
                String[] imports = deferredImport.getImportSelector().selectImports(configClass.getMetadata());
                // 处理任何一个 @Import 注解
                processImports(configClass, asSourceClass(configClass), asSourceClasses(imports), false);
            }
            catch (BeanDefinitionStoreException ex) {
                throw ex;
            }
            catch (Throwable ex) {
                throw new BeanDefinitionStoreException(
                        "Failed to process import candidates for configuration class [" +
                        configClass.getMetadata().getClassName() + "]", ex);
            }
        }
    }
```

##### 私有方法 doProcessConfigurationClass\(\)对一个配置类执行处理和构建的主逻辑 {#私有方法-doprocessconfigurationclass对一个配置类执行处理和构建的主逻辑}

```
/**
     * Apply processing and build a complete ConfigurationClass by reading the
     * annotations, members and methods from the source class. This method can be called
     * multiple times as relevant sources are discovered.
     * @param configClass the configuration class being build
     * @param sourceClass a source class
     * @return the superclass, or null if none found or previously processed
     */
    protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
            throws IOException {

        // Recursively process any member (nested) classes first，首先递归处理嵌套类
        processMemberClasses(configClass, sourceClass);

        // Process any @PropertySource annotations，处理每个@PropertySource注解
        for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
                sourceClass.getMetadata(), PropertySources.class,
                org.springframework.context.annotation.PropertySource.class)) {
            if (this.environment instanceof ConfigurableEnvironment) {
                processPropertySource(propertySource);
            }
            else {
                logger.warn("Ignoring @PropertySource annotation on [" + sourceClass.getMetadata().getClassName() +
                        "]. Reason: Environment must implement ConfigurableEnvironment");
            }
        }

        // Process any @ComponentScan annotations，处理每个@ComponentScan注解
        Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
                sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
        if (!componentScans.isEmpty() &&
                !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
            for (AnnotationAttributes componentScan : componentScans) {
                // 该配置类上注解了@ComponentScan,现在执行扫描，获取其中的Bean定义
                Set<BeanDefinitionHolder> scannedBeanDefinitions =
                        this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
                // 对Component scan得到的Bean定义做检查，看看里面是否有需要处理的配置类，
                // 有的话对其做分析处理
                for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
                    if (ConfigurationClassUtils.checkConfigurationClassCandidate(
                            holder.getBeanDefinition(), this.metadataReaderFactory)) {
                        // 如果该Bean定义是一个配置类，它进行分析
                        parse(holder.getBeanDefinition().getBeanClassName(), holder.getBeanName());
                    }
                }
            }
        }

        // Process any @Import annotations，处理每个@Import注解        
        // 注意这里调用到了getImports()方法，它会搜集sourceClass上所有的@Import注解的value值，
        // 具体搜集的方式是访问sourceClass直接注解的@Import以及递归访问它的注解中隐含的所有的@Import    
        processImports(configClass, sourceClass, getImports(sourceClass), true);

        // Process any @ImportResource annotations，处理每个@ImportResource注解
        if (sourceClass.getMetadata().isAnnotated(ImportResource.class.getName())) {
            AnnotationAttributes importResource =
                    AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
            String[] resources = importResource.getStringArray("locations");
            Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
            for (String resource : resources) {
                String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
                configClass.addImportedResource(resolvedResource, readerClass);
            }
        }

        // Process individual @Bean methods,处理配置类中每个带有@Bean注解的方法
        Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
        for (MethodMetadata methodMetadata : beanMethods) {
            configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
        }

        // Process default methods on interfaces，处理接口上的缺省方法
        processInterfaces(configClass, sourceClass);

        // Process superclass, if any
        // 如果父类superclass存在，并且不是java语言提供的类，也没有处理过，
        // 才返回它以便外层循环继续，否则直接返回null
        if (sourceClass.getMetadata().hasSuperClass()) {
            String superclass = sourceClass.getMetadata().getSuperClassName();
            if (!superclass.startsWith("java") && !this.knownSuperclasses.containsKey(superclass)) {
                this.knownSuperclasses.put(superclass, configClass);
                // Superclass found, return its annotation metadata and recurse
                return sourceClass.getSuperClass();
            }
        }

        // No superclass -> processing is complete，没找到需要处理的父类，处理结果
        return null;// 用返回null告诉外层循环结束
    }
```

##### 私有方法 processImports\(\)处理配置类上搜集到的@Import注解 {#私有方法-processimports处理配置类上搜集到的import注解}

```
/**
      * 处理配置类上搜集到的@Import注解
      * 参数 configuClass 配置类
      * 参数 currentSourceClass 当前源码类
      * 参数 importCandidates, 所有的@Import注解的value
      * 参数 checkForCircularImports, 是否检查循环导入
      **/
    private void processImports(ConfigurationClass configClass, SourceClass currentSourceClass,
            Collection<SourceClass> importCandidates, boolean checkForCircularImports) throws IOException {

        if (importCandidates.isEmpty()) {
        // 如果配置类上没有任何候选@Import，说明没有需要处理的导入，则什么都不用做，直接返回
            return;
        }

        if (checkForCircularImports && isChainedImportOnStack(configClass)) {
        // 如果要求做循环导入检查，并且检查到了循环依赖，报告这个问题
            this.problemReporter.error(new CircularImportProblem(configClass, this.importStack));
        }
        else {
            // 开始处理配置类configClass上所有的@Import importCandidates
            this.importStack.push(configClass);
            try {
                // 循环处理每一个@Import,每个@Import可能导入三种类型的类 :
                // 1. ImportSelector
                // 2. ImportBeanDefinitionRegistrar
                // 3. @Configuration注解的类
                // 下面的for循环中对这三种情况执行了不同的处理逻辑
                for (SourceClass candidate : importCandidates) {
                    if (candidate.isAssignable(ImportSelector.class)) {
                        // Candidate class is an ImportSelector -> delegate to it to determine imports
                        Class<?> candidateClass = candidate.loadClass();
                        ImportSelector selector = BeanUtils.instantiateClass(candidateClass, ImportSelector.class);
                        ParserStrategyUtils.invokeAwareMethods(
                                selector, this.environment, this.resourceLoader, this.registry);
                        if (this.deferredImportSelectors != null && selector instanceof DeferredImportSelector) {
                            this.deferredImportSelectors.add(
                                    new DeferredImportSelectorHolder(configClass, (DeferredImportSelector) selector));
                        }
                        else {
                            String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
                            Collection<SourceClass> importSourceClasses = asSourceClasses(importClassNames);
                            processImports(configClass, currentSourceClass, importSourceClasses, false);
                        }
                    }
                    else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {
                        // Candidate class is an ImportBeanDefinitionRegistrar ->
                        // delegate to it to register additional bean definitions
                        Class<?> candidateClass = candidate.loadClass();
                        ImportBeanDefinitionRegistrar registrar =
                                BeanUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class);
                        ParserStrategyUtils.invokeAwareMethods(
                                registrar, this.environment, this.resourceLoader, this.registry);
                        configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
                    }
                    else {
                        // Candidate class not an ImportSelector or ImportBeanDefinitionRegistrar ->
                        // process it as an @Configuration class
                        this.importStack.registerImport(
                                currentSourceClass.getMetadata(), candidate.getMetadata().getClassName());
                        processConfigurationClass(candidate.asConfigClass(configClass));
                    }
                }
            }
            catch (BeanDefinitionStoreException ex) {
                throw ex;
            }
            catch (Throwable ex) {
                throw new BeanDefinitionStoreException(
                        "Failed to process import candidates for configuration class [" +
                        configClass.getMetadata().getClassName() + "]", ex);
            }
            finally {
                this.importStack.pop();
            }
        }
    }
```



