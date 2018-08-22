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

那这就是活生生的一个例子，通过代理人实现了某种目的，如果真去掉王婆这个中间环节，直接是西门庆和潘金莲勾搭，估计很难成就武松杀嫂事件。

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



