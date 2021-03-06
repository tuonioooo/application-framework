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

## 相关描述

**狭义的迪米特法则**

如果两个类不必彼此直接通信，那么这两个类就不应当发生直接的相互作用。如果其中的一个类需要调用另一个类的某一个方法的话，可以通过第三者转发这个调用。

朋友圈的确定

“朋友”条件：

1）当前对象本身（this）

2）以参量形式传入到当前对象方法中的对象

3）当前对象的[实例变量](https://baike.baidu.com/item/%E5%AE%9E%E4%BE%8B%E5%8F%98%E9%87%8F/3386159)直接引用的对象

4）当前对象的实例变量如果是一个聚集，那么聚集中的元素也都是朋友

5）当前对象所创建的对象

任何一个对象，如果满足上面的条件之一，就是当前对象的“朋友”；否则就是“陌生人”。

**狭义的迪米特法则的缺点：**

在系统里造出大量的小方法，这些方法仅仅是传递间接的调用，与系统的商务逻辑无关。

遵循类之间的迪米特法则会是一个系统的局部设计简化，因为每一个局部都不会和远距离的对象有直接的关联。但是，这也会造成系统的不同模块之间的通信效率降低，也会使系统的不同模块之间不容易协调。

[门面模式](https://baike.baidu.com/item/%E9%97%A8%E9%9D%A2%E6%A8%A1%E5%BC%8F/764642)和[调停者模式](https://baike.baidu.com/item/%E8%B0%83%E5%81%9C%E8%80%85%E6%A8%A1%E5%BC%8F/6465553)实际上就是迪米特法则的应用。

**广义的迪米特法则在类的设计上的体现：**

优先考虑将一个类设置成不变类。

尽量降低一个类的访问权限。

谨慎使用Serializable。

尽量降低成员的访问权限。

