**桥接模式 （Bridge Pattern）**：将抽象与实现解耦，使得两者可以独立的变化

**(一)为什么需要桥接模式**

 1，如果一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性，避免在两个层次之间建立静态的继承联系，通过桥接模式可以使它们在抽象层建立一个关联关系。

2，抽象化角色和实现化角色可以以继承的方式独立扩展而互不影响，在程序运行时可以动态将一个抽象化子类的对象和一个实现化子类的对象进行组合，即系统需要对抽象化角色和实现化角色进行动态耦合。

3，虽然在系统中使用继承是没有问题的，但是由于抽象化角色和具体化角色需要独立变化，设计要求需要独立管理这两者。对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用。

**(二)桥接模式UML图**

![Bridge Pattern](http://upload-images.jianshu.io/upload_images/5261067-547b3ef6db4db7e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 - Abstraction为抽象化角色，主要职责是定义该角色的行为，同时保存一个对实现化角色的引用。
 - Implementor为实现化角色，它是一个接口或抽象类。在PHP中一般是抽象类。

有点不知所云吧。我学的时候也是。老规矩，来个实例吧。

**(三)简单实例**

如果我现在是小米公司的雷布斯，小米公司旗下有小米mix和小米note手机，现在有一个底层的语音输出软件，由于小米mix的全面屏设计没有开孔使用了骨传导，所以和小米note有些不同。那么我们要如何来设计这个输出功能呢？

传统的做法就应该是，一个抽象手机品牌，mix和note手机品牌继承抽象手机品牌。然后有一个mix品牌的输出软件继承自mix品牌。如果这时候有一个redmi的牌子的输出软件就应继承自redmi品牌，redmi继承抽象手机品牌。

如果除了这个语音输出软件，我们还有其它的软件呢，一个手机有很多软件，那么这种继承，如果画出继承链的话，那一个品牌下得有多少子类？

如果我们使用桥接模式的话，就可以把这个软件的具体实现与抽象品牌分离出来，这样也能是我们动态增减实现化角色时更为方便。

```
<?php
//抽象化角色
abstract class MiPhone{
    protected $_audio;      //存放音频软件对象
    abstract function output();
    public function __construct(Audio $audio){
        $this->_audio = $audio;
    }
}
//具体手机
class Mix extends MiPhone{
    //语音输出功能
    public function output(){
        $this->_audio->output();
    }
}
class Note extends MiPhone{
    public function output(){
        $this->_audio->output();
    }
}
//实现化角色 功能实现者
abstract class Audio{
    abstract function output();
}
//具体音频实现者 -骨传导音频输出
class Osteophony extends Audio{
    public function output(){
        echo "骨传导输出的声音-----哈哈".PHP_EOL;
    }
}
//普通音频输出---声筒输出
class Cylinder extends Audio{
    public function output(){
        echo "声筒输出的声音-----呵呵".PHP_EOL;
    }
}

//让小米mix和小米note输出声音
$mix = new Mix(new Osteophony);
$mix->output();
$note = new Note(new Cylinder);
$note->output();
```

这样写的好处是把抽象化角色手机和实现化角色手机的具体功能音频输出 分离了出来。如果现在最新的小米note系列也要用上骨传导输出，那么我们只需实例化时传入的声筒音频类改为骨传导音频类。如果我们还有一个扬声器输出，小米mix和小米note都有这个功能。我们只需要再添加一个扬声器输出类继承Audio类，然后谁使用就保存这个实例在属性中。没有桥接模式的话，我们可是要写两个，一个是小米mix的扬声器输出，一个是小米note。

如果扬声器输出对小米mix和小米note不一样，那么我们确实需要写两个扬声器输出类。使用桥接模式，依然比原来直接继承好，就是因为有一天扬声器技术驱动更新了，我们要更新扬声器不用修改手机类代码，而只需传入一个更新的扬声器类。
