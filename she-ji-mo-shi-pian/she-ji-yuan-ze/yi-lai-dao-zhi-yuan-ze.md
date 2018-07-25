# 依赖倒置原则

## 参考文档：

_**24种设计模式介绍与6大设计原则.pdf**_   链接: [https://pan.baidu.com/s/1WnFwH9\_o29Z48UpQyadiPA](https://pan.baidu.com/s/1WnFwH9_o29Z48UpQyadiPA) 密码: i238

## 定义

A.高层次的模块不应该依赖于低层次的模块，他们都应该依赖于抽象。

B.抽象不应该依赖于具体实现，具体实现应该依赖于抽象。

## 概述

依赖倒置原则（Dependence Inversion Principle）是程序要依赖于抽象接口，不要依赖于具体实现。简单的说就是要求对抽象进行编程，不要对实现进行编程，这样就降低了客户与实现模块间的耦合。

## 意图

[面向过程](https://baike.baidu.com/item/面向过程)的开发，上层调用下层，上层依赖于下层，当下层剧烈变动时上层也要跟着变动，这就会导致模块的复用性降低而且大大提高了开发的成本。

[面向对象](https://baike.baidu.com/item/面向对象)的开发很好的解决了这个问题，一般情况下抽象的变化概率很小，让

[用户程序](https://baike.baidu.com/item/用户程序)依赖于抽象，实现的细节也依赖于抽象。即使实现细节不断变动，只要抽象不变，客户程序就不需要变化。这大大降低了客户程序与实现细节的[耦合度](https://baike.baidu.com/item/耦合度)。

面向过程思想的结构图：

图一

![](file:///C:\Users\tony\AppData\Roaming\Tencent\Users\596807862\QQ\WinTemp\RichOle\IKZCW%28~Q%28YI~%28S{7}%28[S%29NS.png)

背景1：公司是[福特](https://baike.baidu.com/item/福特)和[本田](https://baike.baidu.com/item/本田/444932)

公司的金牌合作伙伴，现要求开发一套自动驾驶系统，只要汽车上安装该系统就可以实现无人驾驶，该系统可以在福特和本田车上使用，只要这两个品牌的汽车使用该系统就能实现自动驾驶。于是有人做出了分析如图一。

对于图一分析：我们定义了一个AutoSystem类，一个FordCar类，一个HondaCar类。FordCar类和HondaCar类中各有三个方法:Run\(启动Car\)、Turn\(转弯Car\)、Stop\(停止Car\)，当然了一个汽车肯定不止这些功能，这里只要能说明问题即可。AutoSystem类是一个自动驾驶系统，自动操纵这两辆车。

## 代码实现

```
public class HondaCar{
    public void Run(){
        Console.WriteLine("本田开始启动了");
    }
    public void Turn(){
        Console.WriteLine("本田开始转弯了");
    }
    public void Stop(){
        Console.WriteLine("本田开始停车了");
    }
}
public class FordCar{
    publicvoidRun(){
        Console.WriteLine("福特开始启动了");
    }
    publicvoidTurn(){
        Console.WriteLine("福特开始转弯了");
    }
    publicvoidStop(){
        Console.WriteLine("福特开始停车了");
    }
}
public class AutoSystem{
    public enum CarType{
        Ford,Honda
    };
    private HondaCar hcar=new HondaCar();
    private FordCar fcar=new FordCar();
    private CarType type;
    public AutoSystem(CarType type){
        this.type=type;
    }
    private void RunCar(){
        if(type==CarType.Ford){
            fcar.Run();
        } else {
            hcar.Run();
        }
    }
    private void TurnCar(){
        if(type==CarType.Ford){
            fcar.Turn();
        } else { 
            hcar.Turn();
        }
    }
    private void StopCar(){
        if(type==CarType.Ford){
            fcar.Stop();
            } else {
                hcar.Stop();
            }
    }
}
```

代码分析：上面的程序确实能够实现针对Ford和Honda车的无人驾驶，但是软件是在不断变化的，软件的需求也在不断的变化。

背景2：公司的业务做大了，同时成为了通用、三菱、大众的金牌合作伙伴，于是公司要求该自动驾驶系统也能够安装在这3种公司生产的汽车上。于是我们不得不变动AutoSystem：

```
public class AutoSystem{
public enum CarType{
Ford,Honda,Bmw
};
HondaCar hcar=new HondaCar();
FordCarf car=new FordCar();
BmwCar bcar=new BmwCar();
private CarType type;
public AutoSystem(CarTypetype){
this.type=type;
}
private void RunCar(){
if(type==CarType.Ford){
fcar.Run();
}
else if(type==CarType.Honda){
hcar.Run();
}
else if(type==CarType.Bmw){
bcar.Run();
}
}
private void TurnCar(){
if(type==CarType.Ford){
fcar.Turn();
}
else if(type==CarType.Honda){
hcar.Turn();
}
else if(type==CarType.Bmw){
bcar.Turn();
}
}
private void StopCar(){
if(type==CarType.Ford){
fcar.Stop();
}
else if(type==CarType.Honda){
hcar.Stop();
}
else if(type==CarType.Bmw){
bcar.Stop();
}
}
}
```

分析：这会给系统增加新的相互依赖。随着时间的推移，越来越多的车种必须加入到AutoSystem中，这个“AutoSystem”模块将会被if/else语句弄得很乱，而且依赖于很多的低层模块，只要低层模块发生变动，AutoSystem就必须跟着变动，它最终将变得僵化、脆弱。

导致上面所述问题的一个原因是，含有高层策略的模块，如AutoSystem模块，依赖于它所控制的低层的具体细节的模块（如HondaCar\(\)和FordCar\(\)）。如果我们能够找到一种方法使AutoSystem[模块独立](https://baike.baidu.com/item/模块独立)于它所控制的具体细节，那么我们就可以自由地复用它了。我们就可以用这个模块来生成其它的程序，使得系统能够用在需要的汽车上。OOD给我们提供了一种机制来实现这种“依赖倒置”。

## 结构图

图二

看图 2中这个简单的类图。这儿有一个“AutoSystem”类，它包含一个“ICar”接口。这个“AutoSystem”类根本不依赖于“FordCar”和“HondaCar”。所以，依赖关系被“倒置”了：“AutoSystem”模块依赖于抽象，那些具体的汽车操作也依赖于相同的抽象。

于是可以添加ICar：

```
public interface ICar
{
void Run();
void Turn();
void Stop();
}
public class BmwCar:ICar
{
public void Run()
{
Console.WriteLine("宝马开始启动了");
}
public void Turn()
{
Console.WriteLine("宝马开始转弯了");
}
public void Stop()
{
Console.WriteLine("宝马开始停车了");
}
}
public class FordCar:ICar
{
publicvoidRun()
{
Console.WriteLine("福特开始启动了");
}
public void Turn()
{
Console.WriteLine("福特开始转弯了");
}
public void Stop()
{
Console.WriteLine("福特开始停车了");
}
}
public class HondaCar:ICar
{
publicvoidRun()
{
Console.WriteLine("本田开始启动了");
}
public void Turn()
{
Console.WriteLine("本田开始转弯了");
}
public void Stop()
{
Console.WriteLine("本田开始停车了");
}
}
public class AutoSystem
{
private ICar icar;
public AutoSystem(ICar icar)
{
this.icar=icar;
}
private void RunCar()
{
icar.Run();
}
private void TurnCar()
{
icar.Turn();
}
private void StopCar()
{
icar.Stop();
}
}
```

现在AutoSystem系统依赖于ICar 这个抽象，而与具体的实现细节HondaCar、FordCar、BmwCar无关，所以实现细节的变化不会影响AutoSystem。对于实现细节只要实现ICar 即可，即实现细节依赖于ICar 抽象。

综上：

一个应用中的重要策略决定及业务模型正是在这些高层的模块中。也正是这些模型包含着应用的特性。但是，当这些模块依赖于低层模块时，低层模块的修改将会直接影响到它们，迫使它们也去改变。这种境况是荒谬的。应该是处于高层的模块去迫使那些低层的模块发生改变。应该是处于高层的模块优先于低层的模块。无论如何高层的模块也不应依赖于低层的模块。而且，我们想能够复用的是高层的模块。通过[子程序](https://baike.baidu.com/item/子程序)库的形式，我们已经可以很好地复用低层的模块了。当高层的模块依赖于低层的模块时，这些高层模块就很难在不同的环境中复用。但是，当那些高层

[模块独立](https://baike.baidu.com/item/模块独立)于低层模块时，它们就能很简单地被复用了。这正是位于框架设计的最核心之处的原则。

总结：依赖倒置原则

A.高层次的模块不应该依赖于低层次的模块，他们都应该依赖于抽象。

B.抽象不应该依赖于具体，具体应该依赖于抽象。

