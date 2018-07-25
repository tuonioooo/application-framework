# 迪米特法则

## 参考文档：

_**24种设计模式介绍与6大设计原则.pdf**_   链接: [https://pan.baidu.com/s/1WnFwH9\_o29Z48UpQyadiPA](https://pan.baidu.com/s/1WnFwH9_o29Z48UpQyadiPA) 密码: i238

## 概述

迪米特法则（Law of Demeter）又叫作最少知识原则（Least Knowledge Principle 简写LKP），就是说一个对象应当对其他对象有尽可能少的了解,不和陌生人说话。英文简写为: LoD.

## 来源历史

1987年秋天由美国Northeastern University的Ian Holland提出，被UML的创始者之一[Booch](https://baike.baidu.com/item/Booch/6573414)等普及。后来，因为在经典著作《 The Pragmatic Programmer》而广为人知。

## 模式与意义

迪米特法则可以简单说成：**talk only to your immediate friends。**

对于OOD来说，又被解释为下面几种方式：一个软件实体应当尽可能少的与其他实体发生相互作用。每一个软件单位对其他的单位都只有最少的知识，而且局限于那些与本单位密切相关的软件单位。

迪米特法则的[初衷](https://baike.baidu.com/item/初衷)在于降低类之间的[耦合](https://baike.baidu.com/item/耦合/2821124)。由于每个类尽量减少对其他类的依赖，因此，很容易使得系统的功能模块功能独立，相互之间不存在（或很少有）依赖关系。

迪米特法则不希望类之间建立直接的联系。如果真的有需要建立联系，也希望能通过它的[友元类](https://baike.baidu.com/item/友元类/518734)来转达。因此，应用迪米特法则有可能造成的一个后果就是：系统中存在大量的中介类，这些类之所以存在完全是为了传递类之间的相互调用关系——这在一定程度上增加了系统的复杂度。

有兴趣可以研究一下设计模式的[门面模式](https://baike.baidu.com/item/门面模式/764642)（[Facade](https://baike.baidu.com/item/Facade/2954918)）和中介模式（Mediator），都是迪米特法则应用的例子。

值得一提的是，虽然Ian Holland对计算机科学的贡献也仅限于这一条法则，其他方面的建树不多，但是，这一法则却不仅仅局限于计算机领域，在其他领域也同样适用。比如，美国人就在航天系统的设计中采用这一法则。



