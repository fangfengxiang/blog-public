**策略模式(Strategy Pattern)**：定义一系列算法，将每一个算法封装起来，并让它们可以相互替换。策略模式让算法独立于使用它的客户而变化，也称为政策模式(Policy)

**(一)为什么需要策略模式**

1，在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护。

2，利用面向对象的继承和多态机制，将多个算法解耦。避免类中出现太多的if-else语句

**(二)策略模式 UML图**

![Strategy Pattern](http://upload-images.jianshu.io/upload_images/5261067-af48a1e91eacc0f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> **Context(应用场景)**:
> 1、需要使用ConcreteStrategy提供的算法。
> 2、 内部维护一个Strategy的实例。
> 3、 负责动态设置运行时Strategy具体的实现算法。
> 4、负责跟Strategy之间的交互和数据传递。
> **Strategy(抽象策略类)**：
>  定义了一个公共接口，各种不同的算法以不同的方式实现这个接口，Context使用这个接口调用不同的算法，一般使用接口或抽象类实现。
> **ConcreteStrategy(具体策略类)**：
>  实现了Strategy定义的接口，提供具体的算法实现。

**(三)简单实例**

如果我们现在在做一个商城，用户来了，我们想向其推荐对应的商品。比如说一个女性用户登录的话，我们就问她需要衣服，男性用户就推荐螺丝刀，不确定性别就推荐《葵花宝典》

```
<?php
//抽象策略接口
abstract class Strategy{
  abstract function peddle();
}

//具体封装的策略方法   女性用户策略
class ConcreteStrategyA extends Strategy
{
    public function peddle(){
        echo '美女穿这件衣服肯定很好看'.PHP_EOL;
    }
}

//男性用户策略
class ConcreteStrategyB extends Strategy
{
    public function peddle(){
        echo '每一个男人都需要一个工具箱，充实工具箱，从购买各种螺丝刀开始'.PHP_EOL;
    }

}

//未知性别用户策略
class ConcreteStrategyC extends Strategy
{
  public function peddle(){
        echo '骨骼清奇，这本《葵花宝典》最适合你'.PHP_EOL;
    }
}

//环境类
class Context{
    protected $strategy;

    public function __construct(Strategy $strategy)
    {
      $this->strategy = $strategy;
    }

    public function request()
    {
      $this->strategy->peddle($this);
    }
}

//若处于女性用户环境
$female_context = new Context(new ConcreteStrategyA);
$female_context->request();
//若处于男性用户环境
$male_context = new Context(new ConcreteStrategyB);
$male_context->request();
//若处于未知性别用户环境
$unknow_context = new Context(new ConcreteStrategyC);
$unknow_context->request();
```

这里我们可以看到，我们可以通过不同的环境选择不同的策略，用户性别的判断 如果对于一个商城系统来说，完全可以在用户登陆的时候通过数据库的用户信息得到，然后做出判断，选择适合的策略。

而且策略模式最显而易见的优势，就是如果我想增加一个新的策略怎么办？比如说，如果用户是小孩，就给他推荐《五年中考三年模拟》。是的，相信你们心里已经有答案了。那就是，再增加一个具体策略类。
