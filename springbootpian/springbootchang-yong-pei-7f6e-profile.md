# SpringBoot常用配置-Profile

# 分析 {#分析}

对于Profile先做一个简单的介绍:  
单讲profile就是一组配置，不同profile提供不同组合的配置，程序运行时可以选择使用哪些profile来适应环境。

也就是Profile为在不同环境下使用不同的配置提供了支持\(开发环境下的配置和生产环境下的配置肯定是不同的，例如:数据库的配置\)

Spring 为我们提供了大量的激活 profile 的方法，可以通过代码来激活，也可以通过系统环境变量、JVM参数、servlet上下文参数来定义 spring.profiles.active 参数激活 profile,下面说下3种方法:  
1、通过设定Environment的ActiveProfiles来设定当前context需要使用的配置环境。在开发中使用@profile注解类或者方法，达到在不同情况下选择实例化不同的Bean。  
2、通过设定jvm的spring.profile.active参数来设置配置环境。  
3、Web项目设置在Service的context parameter中。

进行本示例的演示，需要先配置好Maven和Spring哦、  
见:

