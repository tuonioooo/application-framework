# EnableAutoConfigurationImportSelector 是如何工作的 ?

### 功能 {#功能}

EnableAutoConfigurationImportSelector 是一个DeferredImportSelector，由 spring boot autoconfigure 从版本1.3开始,提供用来处理EnableAutoConfiguration自动配置。

EnableAutoConfigurationImportSelector继承自AutoConfigurationImportSelector,从 spring boot 1.5 以后，EnableAutoConfigurationImportSelector已经不再被建议使用，而是推荐使用 AutoConfigurationImportSelector。



