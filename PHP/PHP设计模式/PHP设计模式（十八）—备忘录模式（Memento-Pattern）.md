**备忘录模式 （Memento Pattern）**： 在不破坏封闭的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。又叫做快照模式（Snapshot Pattern）或Token模式

**(一)为什么需要备忘录模式**

1，有时一些发起人对象的内部信息必须保存在发起人对象以外的地方，但是必须要由发起人对象自己读取，这时，使用备忘录模式可以把复杂的发起人内部信息对其他的对象屏蔽起来，从而可以恰当地保持封装的边界。

2，本模式简化了发起人类。发起人不再需要管理和保存其内部状态的一个个版本，客户端可以自行管理他们所要的这些状态的版本。

**(二)备忘录模式 UML图**

![Memento Pattern](http://upload-images.jianshu.io/upload_images/5261067-cf7945e0a408b568.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**(三)简单实例**

备忘录模式往简单了说，就是打副本。这里我们给出一个备忘录模式的小例子，备份一个游戏角色，也就是发起者的初始状态，并恢复。

```
<?php
//发起人，所需备份者
class Originator{
    //内部状态
    private $state;
    //设置状态
    public function setState($state){
        $this->state = $state ;
    }
    //查看状态
    public function getState(){
        echo $this->state,PHP_EOL;
    }
    //创建一个备份录
    public function createMemento(){
        return new Memento($this->state);
    }
    //恢复一个备份
    public function restoreMemento(Memento $memento){
        $this->state = $memento->getState();
    }
}
//备忘录角色
class Memento{
    private $state; //用于存放发起人备份时的状态
    public function __construct($state){
        $this->state = $state;
    }
    public function getState(){
        return $this->state;
    }
}
//备忘录管理者
class Caretaker{
    private $menento;
    //存档备忘录
    public function setMemento(Memento $memento){
        $this->memento = $memento;
    }
    //取出备忘录
    public function getMemento(){
        return $this->memento;
    }
}

//实例化发起人 假如是个游戏角色
$role = new Originator;
//设置状态 满血
$role->setState('满血');
//备份
//创建备份录管理者
$caretaker = new Caretaker;
//创建备份
$caretaker->setMemento($role->createMemento());
//状态更改
$role->setState('死亡');
$role->getState();
//恢复备份
$role->restoreMemento($caretaker->getMemento());
//重新查看状态
$role->getState();
```


可能最后那段恢复备份的代码有点绕，这是因为我们引入了备份管理者。其实如果对于只有一个备份，那么我们也可以不用备份管理者。而备份管理者存在的好处，当然是管理多个备份了。如果对于多个备份，我们可以把备份管理者的memento属性改为数组变量，就可以存放多个备份了。

其实备份在原型模式我们也提过，我们完全可以通过clone关键字来备份，但是备忘录模式相对于原型模式更精简，可能有些时候我们只想备份的就只有这一个属性呢。而且从本质上说备忘录模式恢复备份后还是原来那个对象，而原型模式就不一定了。如果原型模式恢复备份是直接使用clone出来的对象副本，那么其实它就不算原来那个对象了，虽然它和被clone的对象几乎一模一样，使用无差别，但是对于var_dump，它的object#id肯定是不一样的。
