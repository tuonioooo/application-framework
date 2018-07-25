# 开闭原则

## 参考文档：

_**24种设计模式介绍与6大设计原则.pdf**_   链接: [https://pan.baidu.com/s/1WnFwH9\_o29Z48UpQyadiPA](https://pan.baidu.com/s/1WnFwH9_o29Z48UpQyadiPA) 密码: i238



## 概述

开闭原则（OCP）是[面向对象设计](https://baike.baidu.com/item/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1)中“可复用设计”的基石，是面向对象[设计](https://baike.baidu.com/item/%E8%AE%BE%E8%AE%A1/290622)中最重要的原则之一，其它很多的设计原则都是实现开闭原则的一种手段。对于扩展是开放的，对于修改是关闭的，这意味着模块的行为是可以扩展的。当应用的需求改变时，我们可以对模块进行扩展，使其具有满足那些改变的新行为。也就是说，我们可以改变模块的功能。对模块行为进行扩展时，不必改动模块的源代码或者二进制代码。模块的二进制可执行版本，无论是可链接的库、DLL或者.EXE文件，都无需改动。

## 简介

开闭原则（OCP）是

[面向对象设计](https://baike.baidu.com/item/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1)

中“可复用设计”的基石，是面向对象设计中最重要的原则之一，其它很多的设计原则都是实现开闭原则的一种手段。

1988年，勃兰特·梅耶（Bertrand Meyer）在他的著作《面向对象软件构造（Object Oriented Software Construction）》中提出了开闭原则，它的原文是这样：“Software entities should be open for extension,but closed for modification”。翻译过来就是：“软件实体应当对扩展开放，对修改关闭”。这句话说得略微有点专业，我们把它讲得更通俗一点，也就是：软件系统中包含的各种组件，例如

[模块](https://baike.baidu.com/item/%E6%A8%A1%E5%9D%97)

（Modules）、

[类](https://baike.baidu.com/item/%E7%B1%BB)

（Classes）以及功能（Functions）等等，应该在不修改现有代码的基础上，引入新功能。开闭原则中“开”，是指对于组件功能的扩展是开放的，是允许对其进行功能扩展的；开闭原则中“闭”，是指对于原有代码的修改是封闭的，即修改原有的代码对外部的使用是透明的。

