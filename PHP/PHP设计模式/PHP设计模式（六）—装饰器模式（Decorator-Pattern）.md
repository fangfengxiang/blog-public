**装饰器模式（Decorator Pattern）：** 允许向一个已有的对象添加新的功能或部分内容，同时又不改变其结构。属于结构型模式，它是作为现有的类的一个包装。

**（一）为什么需要装饰器模式：**

1，我们要对一个已有的对象添加新功能，又不想修改它原来的结构。

2，使用子类继承的方法去实现添加新功能，会不可避免地出现子类过多，继承链很长的情况。而且不少书籍都规劝我们竭力保持一个对象的父与子关系不超过3个。

3，装饰器模式，可以提供对对象内容快速非侵入式地修改。

**（二）装饰器模式UML图**

![Decorator Pattern](http://upload-images.jianshu.io/upload_images/5261067-1d50d0685ce057d3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**（三）简单实例**

如果有一个游戏角色，他原来就是默认穿一件长衫。现在游戏改进了，觉得这个角色，除了穿一件长衫前，还可以在里面穿一件袍子，或是一件球衣。在外面穿一套盔甲或是宇航服。

```
<?php
/*游戏原来的角色类
class Person{
    public function clothes(){
        echo "长衫".PHP_EOL;
    }
}
*/

//装饰器接口
interface Decorator
{
   public function beforeDraw();
   public function afterDraw();
}
//具体装饰器1-宇航员装饰
class AstronautDecorator implements Decorator
{
    public function beforeDraw()
    {
        echo "穿上T恤".PHP_EOL;
    }
    function afterDraw()
    {
        echo "穿上宇航服".PHP_EOL;
        echo "穿戴完毕".PHP_EOL;
    }
}
//具体装饰器2-警察装饰
class PoliceDecorator implements Decorator{
    public function beforeDraw()
    {
        echo "穿上警服".PHP_EOL;
    }
    function afterDraw()
    {
        echo "穿上防弹衣".PHP_EOL;
        echo "穿戴完毕".PHP_EOL;
    }
}
//被装饰者
class Person{
    protected $decorators = array(); //存放装饰器
    //添加装饰器
    public function addDecorator(Decorator $decorator)
    {
        $this->decorators[] = $decorator;
    }
    //所有装饰器的穿长衫前方法调用
    public function beforeDraw()
    {
        foreach($this->decorators as $decorator)
        {
            $decorator->beforeDraw();
        }
    }
    //所有装饰器的穿长衫后方法调用
    public function afterDraw()
    {
        $decorators = array_reverse($this->decorators);
        foreach($decorators as $decorator)
        {
            $decorator->afterDraw();
        }
    }
    //装饰方法
    public function clothes(){
        $this->beforeDraw();
        echo "穿上长衫".PHP_EOL;
        $this->afterDraw();
    }
}
//警察装饰
$police = new Person;
$police->addDecorator(new PoliceDecorator);
$police->clothes();
//宇航员装饰
$astronaut = new Person;
$astronaut->addDecorator(new AstronautDecorator);
$astronaut->clothes();
//混搭风
$madman = new Person;
$madman->addDecorator(new PoliceDecorator);
$madman->addDecorator(new AstronautDecorator);
$madman->clothes();
```


当然，上面的代码没有严格地按照UML图，这是因为当被装饰者只有一个时，那 Component也就是ConcreteComponent。同理，如果，只有一个装饰器，那也没必要实现一个implment接口。

如果我们有两个不同的被装饰者，那当然就应该抽象出一个Component，让这两个被装饰者去继承它。也许，那继承不是又来了吗。既然外面都用到继承，直接把装饰器的方法放到继承里面不就行了。

可是你想，如果，直接通过继承的话，那装饰过的被装饰者就应该继承自被装饰者，而且被装饰者因为装饰的不同还要有很多不同类型的子类。而使用装饰者模式的话，继承链缩短了，而且不同的装饰类型还可以动态增加。
