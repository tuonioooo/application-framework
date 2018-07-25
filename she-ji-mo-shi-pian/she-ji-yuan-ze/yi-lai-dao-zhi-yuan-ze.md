# 依赖倒置原则

## 参考文档：

_**24种设计模式介绍与6大设计原则.pdf**_   链接: [https://pan.baidu.com/s/1WnFwH9\_o29Z48UpQyadiPA](https://pan.baidu.com/s/1WnFwH9_o29Z48UpQyadiPA) 密码: i238

## 定义

A.高层次的模块不应该依赖于低层次的模块，他们都应该依赖于抽象。

B.抽象不应该依赖于具体实现，具体实现应该依赖于抽象。

## 概述

依赖倒置原则（Dependence Inversion Principle）是程序要依赖于抽象接口，不要依赖于具体实现。简单的说就是要求对抽象进行编程，不要对实现进行编程，这样就降低了客户与实现模块间的耦合。

## 意图

[面向过程](https://baike.baidu.com/item/%E9%9D%A2%E5%90%91%E8%BF%87%E7%A8%8B)的开发，上层调用下层，上层依赖于下层，当下层剧烈变动时上层也要跟着变动，这就会导致模块的复用性降低而且大大提高了开发的成本。

[面向对象](https://baike.baidu.com/item/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1)的开发很好的解决了这个问题，一般情况下抽象的变化概率很小，让

[用户程序](https://baike.baidu.com/item/%E7%94%A8%E6%88%B7%E7%A8%8B%E5%BA%8F)依赖于抽象，实现的细节也依赖于抽象。即使实现细节不断变动，只要抽象不变，客户程序就不需要变化。这大大降低了客户程序与实现细节的[耦合度](https://baike.baidu.com/item/%E8%80%A6%E5%90%88%E5%BA%A6)。

面向过程思想的结构图：

图一

背景1：公司是[福特](https://baike.baidu.com/item/%E7%A6%8F%E7%89%B9)和[本田](https://baike.baidu.com/item/%E6%9C%AC%E7%94%B0/444932)

公司的金牌合作伙伴，现要求开发一套自动驾驶系统，只要汽车上安装该系统就可以实现无人驾驶，该系统可以在福特和本田车上使用，只要这两个品牌的汽车使用该系统就能实现自动驾驶。于是有人做出了分析如图一。

对于图一分析：我们定义了一个AutoSystem类，一个FordCar类，一个HondaCar类。FordCar类和HondaCar类中各有三个方法:Run\(启动Car\)、Turn\(转弯Car\)、Stop\(停止Car\)，当然了一个汽车肯定不止这些功能，这里只要能说明问题即可。AutoSystem类是一个自动驾驶系统，自动操纵这两辆车。



