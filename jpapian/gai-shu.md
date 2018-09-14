# 概述

JPA是Java Persistence API的简称，中文名Java持久层API，是JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体[对象持久化](https://baike.baidu.com/item/对象持久化/7316192)到数据库中。

Sun引入新的JPA ORM规范出于两个原因：其一，简化现有Java EE和Java SE应用开发工作；其二，Sun希望整合ORM技术，实现天下归一。

# 优势

* ### 标准化

JPA 是 JCP 组织发布的 Java EE 标准之一，因此任何声称符合 JPA 标准的框架都遵循同样的架构，提供相同的访问[API](https://baike.baidu.com/item/API)，这保证了基于JPA开发的企业应用能够经过少量的修改就能够在不同的JPA框架下运行。

* **容器级特性的支持**

JPA框架中支持大数据集、[事务](https://baike.baidu.com/item/事务)、并发等容器级事务，这使得 JPA 超越了简单持久化框架的局限，在企业应用发挥更大的作用。

* ### 简单方便

JPA的主要目标之一就是提供更加简单的编程模型：在JPA框架下创建实体和创建Java 类一样简单，没有任何的约束和限制，只需要使用 javax.persistence.Entity进行注释，JPA的框架和接口也都非常简单，没有太多特别的规则和设计模式的要求，开发者可以很容易地掌握。JPA基于非侵入式原则设计，因此可以很容易地和其它框架或者容器集成。

* ### 查询能力

JPA的查询语言是[面向对象](https://baike.baidu.com/item/面向对象)而非面向数据库的，它以面向对象的自然语法构造查询语句，可以看成是Hibernate HQL的等价物。JPA定义了独特的[JPQL](https://baike.baidu.com/item/JPQL)（Java Persistence Query Language），JPQL是EJB QL的一种扩展，它是针对实体的一种查询语言，操作对象是实体，而不是关系数据库的表，而且能够支持批量更新和修改、JOIN、GROUP BY、HAVING 等通常只有 SQL 才能够提供的高级查询特性，甚至还能够支持[子查询](https://baike.baidu.com/item/子查询)。

* ### 高级特性

JPA 中能够支持[面向对象](https://baike.baidu.com/item/面向对象)的高级特性，如类之间的继承、[多态](https://baike.baidu.com/item/多态)和类之间的复杂关系，这样的支持能够让开发者最大限度的使用面向对象的模型设计企业应用，而不需要自行处理这些特性在关系数据库的持久化。

## 供应商

JPA 的目标之一是制定一个可以由很多供应商实现的API，并且开发人员可以编码来实现该API，而不是使用私有供应商特有的API。因此开发人员只需使用供应商特有的API来获得JPA规范没有解决但应用程序中需要的功能。尽可能地使用JPA API，但是当需要供应商公开但是规范中没有提供的功能时，则使用供应商特有的API。

* ### Hibernate

JPA是需要Provider来实现其功能的，Hibernate就是JPA Provider中很强的一个，应该说无人能出其右。从功能上来说，JPA就是Hibernate功能的一个子集。Hibernate 从3.2开始，就开始兼容JPA。Hibernate3.2获得了Sun TCK的JPA\(Java Persistence API\) 兼容认证。

只要熟悉Hibernate或者其他ORM框架，在使用JPA时会发现其实非常容易上手。例如实体对象的状态，在Hibernate有自由、持久、游离三种，JPA里有new，managed，detached，removed，明眼人一看就知道，这些状态都是一一对应的。再如flush方法，都是对应的，而其他的再如说Query query = manager.createQuery\(sql\)，它在Hibernate里写法上是session，而在JPA中变成了manager，所以从Hibernate到JPA的代价应该是非常小的同样，JDO，也开始兼容JPA。在ORM的领域中，看来JPA已经是[王道](https://baike.baidu.com/item/%E7%8E%8B%E9%81%93)，规范就是规范。在各大厂商的支持下，JPA的使用开始变得广泛。

* ### Spring

Spring + Hibernate 常常被称为 Java Web 应用人气最旺的框架组合。而在 JCP 通过的 Web Beans JSR ，却欲将JSF + EJB + JPA 、来自 JBoss Seam（Spring 除外）的一些组件和EJB 3（能够提供有基本拦截和依赖注入功能的简化 Session Bean框架）的一个 Web 组合进行标准化。Spring 2.0 为 JPA 提供了完整的 EJB容器契约，允许 JPA在任何环境内可以在 Spring 管理的服务层使用（包括 Spring 的所有DI 和 AOP增强）。同时，关于下一个Web应用组合会是 EJB、Spring + Hibernate 还是 Spring + JPA 的论战，早已充斥于耳。在Spring 2.0.1中，正式提供对JPA的支持，这也促成了JPA的发展，要知道JPA的好处在于可以分离于容器运行，变得更加的简洁。

* ### OpenJPA

[OpenJPA](https://baike.baidu.com/item/OpenJPA)是 Apache 组织提供的开源项目，它实现了 EJB 3.0 中的 JPA 标准，为开发者提供功能强大、使用简单的持久化数据管理框架。OpenJPA 封装了和关系型数据库交互的操作，让开发者把注意力集中在编写业务逻辑上。OpenJPA 可以作为独立的

[持久层](https://baike.baidu.com/item/%E6%8C%81%E4%B9%85%E5%B1%82)框架发挥作用，也可以轻松的与其它 Java EE 应用框架或者符合 EJB 3.0 标准的容器集成。支持的实现包括Toplink、Hibernate Entitymanager等。

[TopLink](https://baike.baidu.com/item/TopLink)以前需要收费，如今开源了。OpenJPA虽然免费，但功能、性能、普及性等方面更加需要加大力度。

对于EJB来说，实体Bean一直是被批评的对象，由于其太复杂和庞大。JPA的出现，很大程度的分离了复杂性。这让EJB的推广也变得容易。

总而言之，JPA规范主要关注的仅是API的行为方面，而由各种实现完成大多数性能有关的调优。尽管如此，所有可靠的实现都应该拥有某种[数据缓存](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E7%BC%93%E5%AD%98)，以作为选择。但愿不久的将来，JPA能成为真正的标准。

# 小结

EJB 3.0和JPA 毫无疑问将是Java EE 5的主要卖点。在某些领域中，它们给Java社区带来了竞争优势，并使Java 在其他领域与竞争对手不分伯仲（因为，不可否认，某些领域尚不存在基于标准的方法）。

过去数年来，Spring Framework一直是EJB在企业领域的主要竞争对手。EJB 3.0规范解决了很多促进Spring兴起的问题。随着它的出现，EJB3.0毫无疑问比Spring提供了更好的开发体验——最引人注目的优势是它不需要[配置文件](https://baike.baidu.com/item/%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)。

JPA提供一种标准的OR映射解决方案，该解决方案完全集成到EJB3.0兼容的容器中。JPA的前辈将会继续稳定发展，但是业务应用程序中的 raw 使用将可能会减少。实现 JPA 兼容的实体管理器似乎很可能是此类技术的发展方向。

Java EE系列规范的较大问题与JPA没有任何关系。Java EE 系列规范的问题涉及到 Web和EJB容器之间的集成。Spring在此领域仍然具有主要竞争优势。JBoss的Seam项目尝试使用自定义的方法来解决这一问题。Caucho Resin[应用服务器](https://baike.baidu.com/item/%E5%BA%94%E7%94%A8%E6%9C%8D%E5%8A%A1%E5%99%A8)

试图扩展容器边界并支持在Web容器中使用@EJB注释。我们希望Java EE 5.1将解决层集成的问题，为我们提供一个全面而标准的

[依赖性](https://baike.baidu.com/item/%E4%BE%9D%E8%B5%96%E6%80%A7)注入方法。

在不久的将来，Oracle可能会将JPA作为一个单独的JSR对待，同时JPA还可能作为Java SE的一部分。不过这些都不太重要，重要的是，我们已经可以在脱离容器的情况下、在Java SE应用中使用JPA了。

JPA已经作为一项[对象持久化](https://baike.baidu.com/item/%E5%AF%B9%E8%B1%A1%E6%8C%81%E4%B9%85%E5%8C%96)的标准，不但可以获得Java EE应用服务器的支持，还可以直接在Java SE中使用。开发者将无需在现有多种ORM框架中艰难地选择，按照Sun的预想，现有ORM框架头顶的光环将渐渐暗淡，不再具有以往的吸引力。



