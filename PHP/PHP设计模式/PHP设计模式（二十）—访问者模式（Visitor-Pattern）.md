**访问者模式（Visitor Pattern）** ： 表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素类的前提下定义作用于这些元素的新操作。

**(一)为什么需要访问者模式**

> 访问者模式能够在不更改对象的情况下向该对象添加新的功能性

**(二)访问者模式 UML图**

![Visitor Pattern](http://upload-images.jianshu.io/upload_images/5261067-1ad8cb14a91a3caa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

访问者模式UML图通常是比较复杂，如果对于只有一个元素和一种访问者，我们其实也可以不用抽象元素和抽象访问者，不要objectStruct。
下图给出《PHP设计模式》中的访问者模式UML图。

![Visitor Pattern](http://upload-images.jianshu.io/upload_images/5261067-8fca108926c357e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**(三)简单实例**

由于访问者模式的复杂，可能一开始大家看了好久也不知其所然。我也是看了好久都不知道访问者模式的意思。所以这里我只用《PHP设计模式》一书中的UML图，实现如何在不更改对象的情况下向该对象添加新的功能性。

```
<?php
//具体元素
class Superman{
    public $name;
    public function doSomething(){
        echo '我是超人，我会飞'.PHP_EOL;
    }
    public function accept(Visitor $visitor){
        $visitor->doSomething($this);
    }
}
//具体访问者
class Visitor{
    public function doSomething($object){
        echo '我可以返老还童到'.$object->age = 18;
    }
}
//实例化具体对象
$man = new Superman;
//使用自己的能力
$man->doSomething();
//通过添加访问者，把访问者能能力扩展成自己的
$man->accept(new Visitor);
```


我们可以看到，通过调用accept方法接收一个访问者，具体对象可以把访问者的doSomething能力也扩展为自己能力。当然如果你需要多个扩展能力，你可以有多个访问者。而accept方法调用访问者的dosomething方法时，传入$this是为了能使用Superman的属性和方法，使其感觉扩展完就是Superman的真正一部分。
