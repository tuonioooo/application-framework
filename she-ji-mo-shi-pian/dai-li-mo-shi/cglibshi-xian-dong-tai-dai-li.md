# cglib实现动态代理

JDK的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现JDK的动态代理，cglib是针对类来实现代理的，他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对final修饰的类进行代理。

按照上面的之前的例子，我们在在加入一个风流人物，陈圆圆，示例如下：

```
package com.master.proxy;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-8-22
 * Time: 11:40
 * info: 陈圆圆
 */
public class ChenYuanYuan {

    //一看陈圆圆不简单呀，不需要实现接口，自己内藏属性，天生就是做这个事情的人
    public void makeEyesWithMan() {
        System.out.println("makeEyesWithMan：makeEyesWithMan");
    }


    public void happyWithMan() {
        System.out.println("happyWithMan：papapa");
    }


    public void dragonEnteredShuangFeng() {
        System.out.println("dragonEnteredShuangFeng：hahaha");
    }
}

```

再定义一个田弘这样一个人物：

```
package com.master.proxy;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-8-22
 * Time: 11:46
 * info: 定义一个田弘这样一个cglib的人（专门给别人松美女歌姬）
 */
public class TianHongCglib implements MethodInterceptor {

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
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(this.target.getClass());
        // 回调方法
        enhancer.setCallback(this);
        // 创建代理对象
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("intercept：" + o.getClass().getName());
        return proxy.invokeSuper(o, args);
    }
}

```



