# Spring5 Framework体系结构

## 概述

体系结构有如下构成:

| [Core](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/core.html#spring-core)（[核心](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/core.html#spring-core)） | IoC container, Events, Resources, i18n, Validation, Data Binding, Type Conversion, SpEL, AOP. |
| :--- | :--- |
| [Testing](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testing)（测试） | Mock objects, TestContext framework, Spring MVC Test, WebTestClient. |
| [Data Access](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/data-access.html#spring-data-tier)（数据处理） | Transactions, DAO support, JDBC, ORM, Marshalling XML. |
| [Web Servlet](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#spring-web)（Spring MVC） | Spring MVC, WebSocket, SockJS, STOMP messaging. |
| [Web Reactive](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web-reactive.html#spring-webflux)（WEB 响应式编程） | Spring WebFlux, WebClient, WebSocket. |
| [Integration](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#spring-integration)（集成） | Remoting, JMS, JCA, JMX, Email, Tasks, Scheduling, Cache. |
| [Languages](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/languages.html#languages)（语言） | Kotlin, Groovy, Dynamic languages. |

结构细化如下：

| spring-aop | aop 原理及源码剖析 |
| :--- | :--- |
| spring-aspects |  |
| spring-beans | cores的核心bean组件 |
| spring-context-indexer | cores的核心context组件 |
| spring-context-support | cores的核心context组件 |
| spring-context | cores的核心context组件 |
| spring-core | cores的核心组件 |
| spring-expression | cores的核心表达式组件 |
| spring-framework-bom |  |
| spring-instrument | Springframework结构设计组件 |
| spring-jcl | 日志框架组件 |
| spring-jdbc | data access的数据持久层组件 |
| spring-jms | data access的消息服务组件 |
| spring-messaging |  |
| spring-orm | data access对象关系映射组件 |



