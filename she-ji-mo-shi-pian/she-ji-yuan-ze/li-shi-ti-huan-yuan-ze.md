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

## 形象理解

我们来研究一下LSP的实质。学习OO的时候，我们知道，一个对象是一组状态和一系列行为的

[组合体](https://baike.baidu.com/item/%E7%BB%84%E5%90%88%E4%BD%93)

。状态是对象的内在特性，行为是对象的外在特性。LSP所表述的就是在同一个继承体系中的对象应该有共同的行为特征。

这一点上，表明了OO的继承与日常生活中的继承的本质区别。举一个例子：生物学的分类体系中把企鹅归属为鸟类。我们模仿这个体系，设计出这样的类和关系。

类“鸟”中有个方法fly，企鹅自然也继承了这个方法，可是企鹅不能飞阿，于是，我们在企鹅的类中覆盖了fly方法，告诉方法的调用者：企鹅是不会飞的。这完全符合常理。但是，这违反了LSP，企鹅是鸟的子类，可是企鹅却不能飞！需要注意的是，此处的“鸟”已经不再是生物学中的鸟了，它是软件中的一个类、一个抽象。

有人会说，企鹅不能飞很正常啊，而且这样编写代码也能正常编译，只要在使用这个类的客户代码中加一句判断就行了。但是，这就是问题所在！首先，客户代码和“企鹅”的代码很有可能不是同时设计的，在当今软件外包一层又一层的开发模式下，你甚至根本不知道两个模块的原产地是哪里，也就谈不上去修改客户代码了。客户程序很可能是遗留系统的一部分，很可能已经不再维护，如果因为设计出这么一个“企鹅”而导致必须修改客户代码，谁应该承担这部分责任呢？（大概是上帝吧，谁叫他让“企鹅”不能飞的。^\_^）“修改客户代码”直接违反了OCP，这就是OCP的重要性。违反LSP将使既有的设计不能封闭！

修正后的设计如下：

但是，这就是LSP的全部了么？书中给了一个经典的例子，这又是一个不符合常理的例子：正方形不是一个长方形。这个悖论的详细内容能在网上找到，我就不多废话了。

LSP并没有提供解决这个问题的方案，而只是提出了这么一个问题。

于是，工程师们开始关注如何确保对象的行为。1988年，B. Meyer提出了Design by Contract（

[契约式设计](https://baike.baidu.com/item/%E5%A5%91%E7%BA%A6%E5%BC%8F%E8%AE%BE%E8%AE%A1)

）理论。DbC从

[形式化方法](https://baike.baidu.com/item/%E5%BD%A2%E5%BC%8F%E5%8C%96%E6%96%B9%E6%B3%95)

中借鉴了一套确保对象行为和自身状态的方法，其基本概念很简单：

**Pre-condition:**

每个方法调用之前，该方法应该校验传入参数的正确性，只有正确才能执行该方法，否则认为调用方违反契约，不予执行。这称为前置条件\(Pre-condition\)。

**Post-Condition:**

一旦通过前置条件的校验，方法必须执行，并且必须确保执行结果符合契约，这称之为后置条件\(Post-condition\)。

**Invariant:**

对象本身有一套对自身状态进行校验的检查条件，以确保该对象的本质不发生改变，这称之为不变式\(Invariant\)。

以上是单个对象的约束条件。为了满足LSP，当存在继承关系时，子类中方法的前置条件必须与超类中被覆盖的方法的前置条件相同或者更宽松；而子类中方法的后置条件必须与超类中被覆盖的方法的后置条件相同或者更为严格

一些OO语言中的特性能够说明这一问题：

继承并且覆盖超类方法的时候，子类中的方法的可见性必须等于或者大于超类中的方法的可见性，子类中的方法所抛出的受检异常只能是超类中对应方法所抛出的受检异常的子类。

public class SuperClass{

public void methodA\(\) throws Exception{}

}

public class SubClassA extends SuperClass{

//this overriding is illegal.

private void methodA\(\) throws IOException{}

}

public class SubClassB extends SuperClass{

//this overriding is OK.

public void methodA\(\) throws FileNotFoundException{}

}

从Java5开始，子类中的方法的返回值也可以是对应的超类方法的返回值的子类。这叫做“

[协变](https://baike.baidu.com/item/%E5%8D%8F%E5%8F%98)

”\(Covariant\)

public class SuperClass {

public Number caculate\(\){

return null;

}

}

public class SubClass extends SuperClass{

//only compiles in Java 5 or later.

public Integer caculate\(\){

return null;

}

}

可以看出，以上这些特性都非常好地遵循了LSP。但是DbC呢？很遗憾，主流的面向对象语言（不论是动态语言还是静态语言）还没有加入对DbC的支持。但是随着AOP概念的产生，相信不久DbC也将成为OO语言的一个重要特性之一。

一些题外话：

前一阵子《敲响OO时代的丧钟》和《丧钟为谁而鸣》两篇文章引来了无数议论。其中提到了不少OO语言的不足。事实上，遵从LSP和OCP，不管是静态类型还是动态类型系统，只要是OO的设计，就应该对对象的行为有严格的约束。这个约束并不仅仅体现在方法签名上，而是这个具体行为的本身。这才是LSP和DbC的真谛。从这一点来说并不能说明“万事万物皆对象”的动态语言和“C++，Java”这种“按接口编程”语言的优劣，两类语言都有待于改进。庄兄对DJ的设想倒是开始引入DbC的概念了。这一点还是非常值得期待的。^\_^

另外，接口的语义正被OCP、LSP、DbC这样的概念不断地强化，接口表达了对象行为之间的“契约”关系。而不是简单地作为一种实现多继承的语法糖。

