# 策略模式

## 参考文档：

_**23种java设计模式.pdf   **_链接: [https://pan.baidu.com/s/1bDaSG8YbFagcPl6Fll1WQA](https://pan.baidu.com/s/1bDaSG8YbFagcPl6Fll1WQA) 密码: 9kh8

_**24种设计模式介绍与6大设计原则.pdf**_   链接: [https://pan.baidu.com/s/1WnFwH9\_o29Z48UpQyadiPA](https://pan.baidu.com/s/1WnFwH9_o29Z48UpQyadiPA) 密码: i238

## 前言

策略模式是指有一定行动内容的相对稳定的策略名称。策略模式在古代中又称“计策”，简称“计”，如《汉书·高帝纪上》：“汉王从其计”。这里的“计”指的就是计谋、策略。策略模式具有相对稳定的形式，如“避实就虚”、“出奇制胜”等。一定的策略模式，既可应用于战略决策，也可应用于战术决策；既可实施于大系统的全局性行动，也可实施于大系统的局部性行动。

## 组成

—抽象策略角色： 策略类，通常由一个接口或者抽象类实现。

—具体策略角色：包装了相关的算法和行为。

—环境角色：持有一个策略类的引用，最终给客户端调用。

## 概念

原文：The Strategy Pattern defines a family of algorithms,encapsulates each one,and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.）

Context\(应用场景\):

1、需要使用ConcreteStrategy提供的算法。

2、 内部维护一个Strategy的实例。

3、 负责动态设置运行时Strategy具体的实现算法。

4、负责跟Strategy之间的交互和数据传递。

Strategy\(抽象策略类\)：

1、 定义了一个公共接口，各种不同的算法以不同的方式实现这个接口，Context使用这个接口调用不同的算法，一般使用接口或抽象类实现。

ConcreteStrategy\(具体策略类\)：

2、 实现了Strategy定义的接口，提供具体的算法实现



