# 代理模式

## 参考文档：

_**23种java设计模式.pdf   **_链接: [https://pan.baidu.com/s/1bDaSG8YbFagcPl6Fll1WQA](https://pan.baidu.com/s/1bDaSG8YbFagcPl6Fll1WQA) 密码: 9kh8

_**24种设计模式介绍与6大设计原则.pdf**_   链接: [https://pan.baidu.com/s/1WnFwH9\_o29Z48UpQyadiPA](https://pan.baidu.com/s/1WnFwH9_o29Z48UpQyadiPA) 密码: i238

## 简介

即Proxy Pattern，23种常用的面向对象软件的设计模式之一。（设计模式的说法源自《设计模式》一书，原名《Design Patterns: Elements of Reusable Object-Oriented Software》。1995年出版，出版社：Addison Wesly Longman.Inc。该书提出了23种基本设计模式，第一次将设计模式提升到理论高度，并将之规范化。）

代理模式的定义：为其他对象提供一种[代理](https://baike.baidu.com/item/代理)以控制对这个对象的访问。在某些情况下，一个对象不适合或者不能直接引用另一个对象，而代理对象可以在客户端和目标对象之间起到中介的作用。

## 优点

\(1\).职责清晰

真实的角色就是实现实际的[业务逻辑](https://baike.baidu.com/item/业务逻辑)，不用关心其他非本职责的事务，通过后期的代理完成一件完成事务，附带的结果就是编程简洁清晰。

\(2\).代理对象可以在客户端和目标对象之间起到中介的作用，这样起到了中介的作用和保护了目标对象的作用。

\(3\).高扩展性

## 模式结构

一个是真正的你要访问的对象\(目标类\)，一个是代理对象,真正对象与代理对象实现同一个接口,先访问代理类再访问真正要访问的对象。

代理模式分为静态代理、动态代理。

静态代理是由程序员创建或工具生成代理类的源码，再编译代理类。所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。

动态代理是在实现阶段不用关心代理类，而在运行阶段才指定哪一个对象。



