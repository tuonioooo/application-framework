# java代理实现与原理详细分析

## 前言

关于Java中的代理，我们首先需要了解的是一种常用的设计模式--代理模式，而对于代理，根据创建代理类的时间点，又可以分为静态代理和动态代理。

## 代理模式

代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。简单的说就是，我们在访问实际对象时，是通过代理对象来访问的，代理模式就是在访问实际对象时引入一定程度的间接性，因为这种间接性，可以附加多种用途。在后面我会解释这种间接性带来的好处。代理模式结构图（图片来自《大话设计模式》）：

## ![](/assets/import-proxy-01.png)

## 静态代理

**1、静态代理**

静态代理：由程序员创建或特定工具自动生成源代码，也就是在编译时就已经将接口，被代理类，代理类等确定下来。在程序运行之前，代理类的.class文件就已经生成。

**2、静态代理简单实现**

根据上面代理模式的类图，来写一个简单的静态代理的例子。

比如西门庆找潘金莲，那潘金莲不好意思答复呀，咋办，找那个王婆做代理，表现在程序上时这样的：

先定义一种类型的女人：

```
package com.master.proxy;


/**
 *  @Author tuonioooo
 *  @Date 2018-8-22 10:35
 *  @Info 定义一种类型的女人，王婆和潘金莲都属于这个类型的女人
 *  @Blog https://blog.csdn.net/tuoni123
 */
public interface KindWomen {

    //这种类型的女人能做什么事情呢？
    public void makeEyesWithMan(); //抛媚眼
    public void happyWithMan(); //happy what? You know that!
    public void dragonEnteredShuangFeng();//龙入双峰

}
```

一种类型嘛，那肯定是接口，然后定义潘金莲：

```
package com.master.proxy;

/**
*  @Author tuonioooo
*  @Date 2018-8-22 10:37
*  @Info 潘金莲
*  @Blog https://blog.csdn.net/tuoni123
*/
public class PanJinLian implements KindWomen {

    @Override
    public void makeEyesWithMan() {
        System.out.println("makeEyesWithMan：makeEyesWithMan");
    }

    @Override
    public void happyWithMan() {
        System.out.println("happyWithMan：papapa");
    }

    @Override
    public void dragonEnteredShuangFeng() {
        System.out.println("dragonEnteredShuangFeng：hahaha");
    }
}
```

再定一个丑陋的王婆：

```
package com.master.proxy;

/**
*  @Author tuonioooo
*  @Date 2018-8-22 10:40
*  @Info 王婆
*  @Blog https://blog.csdn.net/tuoni123
*/
public class WangPo implements KindWomen {

    private KindWomen kindWomen;

    public WangPo() { //默认的话，是潘金莲的代理
        this.kindWomen = new PanJinLian();
    }

    //她可以是KindWomen的任何一个女人的代理，只要你是这一类型
    public WangPo(KindWomen kindWomen) {
        this.kindWomen = kindWomen;
    }

    public void happyWithMan() {
        this.kindWomen.happyWithMan(); //自己老了，干不了，可以让年轻的代替
    }

    @Override
    public void dragonEnteredShuangFeng() {
        this.kindWomen.dragonEnteredShuangFeng(); //下垂了，让胸大的来干吧
    }

    public void makeEyesWithMan() {
        this.kindWomen.makeEyesWithMan(); //王婆这么大年龄了，谁看她抛媚眼？！
    }
}
```

两个女主角都上场了，男主角也该出现了：

```
package com.master.proxy;

/**
 * @Author tuonioooo
 * @Date 2018-8-22 10:43
 * @Info 西门庆（这人色中饿鬼）
 * @Blog https://blog.csdn.net/tuoni123
 */
public class XiMenQing {
    /*
     * 水浒里是这样写的：西门庆被潘金莲用竹竿敲了一下难道，痴迷了，
     * 被王婆看到了, 就开始撮合两人好事，王婆作为潘金莲的代理人
     * 收了不少好处费，那我们假设一下：
     * 如果没有王婆在中间牵线，这两个不要脸的能成吗？难说的很！
     */
    public static void main(String[] args) {
        //把王婆叫出来
        WangPo wangPo = new WangPo();
        //然后西门庆就说，我要和潘金莲happy，然后王婆就安排了西门庆丢筷子的那出戏:
        wangPo.makeEyesWithMan(); //看到没，虽然表面上时王婆在做，实际上爽的是潘金莲
        wangPo.happyWithMan();
    }
}
```

那这就是活生生的一个例子，通过代理人实现了某种目的，如果真去掉王婆这个中间环节，直接是西  
门庆和潘金莲勾搭，估计很难成就武松杀嫂事件。

那我们再考虑一下，水浒里还有没有这类型的女人？有，卢俊义的老婆贾氏（就是和那个固管家苟合

的那个），这名字起的：“假使”，那我们也让王婆做她的代理：

```
package com.master.proxy;

/**
 * @Author tuonioooo
 * @Date 2018-8-22 10:48
 * @Info 贾氏
 * @Blog https://blog.csdn.net/tuoni123
 */
public class JiaShi implements KindWomen {

    public void happyWithMan() {
        System.out.println("贾氏正在Happy中......");

    }

    @Override
    public void dragonEnteredShuangFeng() {
        System.out.println("dragonEnteredShuangFeng：hahaha");
    }

    public void makeEyesWithMan() {
        System.out.println("贾氏抛媚眼");
    }
}
```

西门庆勾贾氏：

```
package com.master.proxy;

/**
 * @Author tuonioooo
 * @Date 2018-8-22 10:43
 * @Info 西门庆（这人色中饿鬼）
 * @Blog https://blog.csdn.net/tuoni123
 */
public class XiMenQing {
    /*
     * 水浒里是这样写的：西门庆被潘金莲用竹竿敲了一下难道，痴迷了，
     * 被王婆看到了, 就开始撮合两人好事，王婆作为潘金莲的代理人
     * 收了不少好处费，那我们假设一下：
     * 如果没有王婆在中间牵线，这两个不要脸的能成吗？难说的很！
     */
    public static void main(String[] args) {
        //把王婆叫出来
        WangPo wangPo = new WangPo();
        //然后西门庆就说，我要和潘金莲happy，然后王婆就安排了西门庆丢筷子的那出戏:
        wangPo.makeEyesWithMan(); //看到没，虽然表面上时王婆在做，实际上爽的是潘金莲
        wangPo.happyWithMan();


        //改编一下历史，贾氏被西门庆勾走：
        JiaShi jiaShi = new JiaShi();
        WangPo wangPo1 = new WangPo(jiaShi); //让王婆作为贾氏的代理人
        wangPo.makeEyesWithMan();
        wangPo.happyWithMan();
    }
}
```

说完这个故事，那额总结一下，代理模式主要使用了 Java 的多态，干活的是被代理类，代理类主要是接活，你让我干活，好，我交给幕后的类去干，你满意就成，那怎么知道被代理类能不能干呢？同根就成，大家知根知底，你能做啥，我能做啥都清楚的很，同一个接口呗。

## **动态代理**

** 1.动态代理**

代理类在程序运行时创建的代理方式被成为动态代理。 我们上面静态代理的例子中，代理类\(studentProxy\)是自己定义好的，在程序运行之前就已经编译完成。然而动态代理，代理类并不是在Java代码中定义的，而是在运行时根据我们在Java代码中的“指示”动态生成的。相比于静态代理， 动态代理的优势在于可以很方便的对代理类的函数进行统一的处理，而不用修改每个代理类中的方法。

> 实现原理：在程序运行时，运用反射机制动态创建而成。

比如，想要在每个代理的方法前都加上一个处理方法：

```
public void happyWithMan() {
    giveMoney(10);
    System.out.println("贾氏正在Happy中......");

}
```

这里有几个代理方法，就需要写几次giveMoney的方法，如果有多个静态代理，需要写很多次，非常麻烦。然而动态代理就是为了解决这个问题而生的。

**2、动态代理简单实现**

在java的java.lang.reflect包下提供了一个Proxy类和一个InvocationHandler接口，通过这个类和这个接口可以生成JDK动态代理类和动态代理对象。

* **InvocationHandler接口： **

```
public interface InvocationHandler { 
    public Object invoke(Object proxy,Method method,Object[] args) throws Throwable; 
}
```

> 参数说明：
>
> Object proxy：指被代理的对象/目标对象/委托对象。
>
> Method method：要调用的方法
>
> Object\[\] args：方法调用时所需要的参数
>
> 可以将InvocationHandler接口的子类想象成一个代理的最终操作类，替换掉ProxySubject。

* **Proxy类： **

Proxy类是专门完成代理的操作类，可以通过此类为一个或多个接口动态地生成实现类，此类提供了如下的操作方法：

```
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, 
InvocationHandler h) 
throws IllegalArgumentException
```

> **参数说明：**
>
> ClassLoader loader：类加载器
>
> Class&lt;?&gt;\[\] interfaces：得到全部的接口
>
> InvocationHandler h：得到InvocationHandler接口的子类实例
>
> **类加载器说明：**
>
> 在Proxy类中的newProxyInstance（）方法中需要一个ClassLoader类的实例，ClassLoader实际上对应的是类加载器，在Java中主要有一下三种类加载器;
>
> Booststrap ClassLoader：此加载器采用C++编写，一般开发中是看不到的；
>
> Extendsion ClassLoader：用来进行扩展类的加载，一般对应的是jre\lib\ext目录中的类;
>
> AppClassLoader：\(默认\)加载classpath指定的类，是最常使用的是一种加载器。

还是以王婆、潘金莲、西门庆为示例：

先定义一个王婆动态代理类，他可以代理所有的实现KindWomen这种类型的人（JDK动态代理类）

```
package com.master.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-8-22
 * Time: 11:09
 * info: 定义一个王婆动态代理类，他可以代理所有的实现KindWomen这种类型的人（JDK动态代理类）
 */
public class WangPoInvocationHandler implements InvocationHandler {

    private Object target;// 目标对象/委托对象/被代理对象

    /**
     *  @Author daizhao
     *  @Date 2018-8-22 11:13
     *  @Params [target]
     *  @Return java.lang.Object
     *  @Info   绑定委托对象并返回一个代理类
     */
    public Object bind(Object target) {
        this.target = target;
        //取得代理对象
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(), this);   //要绑定接口(这是一个缺陷，cglib弥补了这一缺陷)
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("WangPoInvocationHandler：invoke");
        //执行方法
        return method.invoke(target, args);


    }
}

```

潘金莲的实现步骤，参考上面的

定义一个西门庆的示例：

```
package com.master.proxy;

/**
 * @Author tuonioooo
 * @Date 2018-8-22 10:43
 * @Info 西门庆（这人色中饿鬼）
 * @Blog https://blog.csdn.net/tuoni123
 */
public class XiMenQing {
    /*
     * 水浒里是这样写的：西门庆被潘金莲用竹竿敲了一下难道，痴迷了，
     * 被王婆看到了, 就开始撮合两人好事，王婆作为潘金莲的代理人
     * 收了不少好处费，那我们假设一下：
     * 如果没有王婆在中间牵线，这两个不要脸的能成吗？难说的很！
     */
    public static void main(String[] args) {
       
        //JDK动态代理实现
        WangPoInvocationHandler wangPoInvocationHandler = new WangPoInvocationHandler();
        KindWomen kindWomen = (KindWomen) wangPoInvocationHandler.bind(new PanJinLian());
        kindWomen.happyWithMan();


    }
}
```



