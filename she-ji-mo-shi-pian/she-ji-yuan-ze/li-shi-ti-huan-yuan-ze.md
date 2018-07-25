# 里氏替换原则

## 参考文档：

_**24种设计模式介绍与6大设计原则.pdf**_   链接: [https://pan.baidu.com/s/1WnFwH9\_o29Z48UpQyadiPA](https://pan.baidu.com/s/1WnFwH9_o29Z48UpQyadiPA) 密码: i238

## 前言

里氏替换原则，OCP作为OO的高层原则，主张使用“抽象\(Abstraction\)”和“多态\(Polymorphism\)”将设计中的静态结构改为动态结构，维持设计的封闭性。“抽象”是语言提供的功能。“多态”由继承语义实现。



## 简介

里氏替换原则\(Liskov Substitution Principle LSP\)面向对象设计的基本原则之一。 里氏替换原则中说，任何基类可以出现的地方，子类一定可以出现。 LSP是继承复用的基石，只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。

如此，问题产生了：“我们如何去度量继承关系的质量？”

Liskov于1987年提出了一个关于继承的原则“Inheritance should ensure that any property proved about supertype objects also holds for subtype objects.”——“继承必须确保超类所拥有的性质在子类中仍然成立。”也就是说，当一个子类的实例应该能够替换任何其超类的实例时，它们之间才具有is-A关系。

该原则称为Liskov Substitution Principle——里氏替换原则。林先生在上课时风趣地称之为“老鼠的儿子会打洞”。^\_^

