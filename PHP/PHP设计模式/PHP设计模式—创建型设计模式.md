** 创建型设计模式 **：
* 单例模式（Singleton Pattern）
* 工厂方法模式(Factor Pattern)
* 抽象工厂模式( Abstract Factor Pattern)
* 建造者模式(Builder Pattern)
* 原型模式（Prototype Pattern）

抽象工厂模式就好比工厂方法模式的升级版，所以本文把工厂方法模式和抽象工厂模式混在一起讲了

------------

#一.单例模式（Singleton Pattern）

------------
**单例模式(Singleton Pattern)**：顾名思义，就是只有一个实例。作为对象的创建模式，单例模式确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。

**（一）为什么要使用PHP单例模式**

1，php的应用主要在于数据库应用, 一个应用中会存在大量的数据库操作, 在使用面向对象的方式开发时, 如果使用单例模式,
则可以避免大量的new 操作消耗的资源,还可以减少数据库连接这样就不容易出现 too many connections情况。

2，如果系统中需要有一个类来全局控制某些配置信息, 那么使用单例模式可以很方便的实现. 这个可以参看zend Framework的FrontController部分。

3，在一次页面请求中, 便于进行调试, 因为所有的代码(例如数据库操作类db)都集中在一个类中, 我们可以在类中设置钩子, 输出日志，从而避免到处var_dump, echo

**（二）单例模式结构图**

![Singleton Pattern](http://upload-images.jianshu.io/upload_images/5261067-d237feca85784587.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**（三）单例模式的实现**

1，私有化一个属性用于存放唯一的一个实例

2，私有化构造方法，私有化克隆方法，用来创建并只允许创建一个实例

3，公有化静态方法，用于向系统提供这个实例

**（四）代码实现**

```
class Singleton{
        //存放实例
        private static $_instance = null;

        //私有化构造方法、
        private function __construct(){
            echo "单例模式的实例被构造了";
        }
        //私有化克隆方法
        private function __clone(){

        }

        //公有化获取实例方法
        public static function getInstance(){
            if (!(self::$_instance instanceof Singleton)){
                self::$_instance = new Singleton();
            }
            return self::$_instance;
        }
    }

    $singleton=Singleton::getInstance();
```

优点：因为静态方法可以在全局范围内被访问，当我们需要一个单例模式的对象时，只需调用getInstance方法，获取先前实例化的对象，无需重新实例化。

**(五)使用Trait关键字实现类似于继承单例类的功能**

``` Trait ```是PHP5.4后加入到特性，有些书说是为了实现类似C++多重继承的功能。Trait定义的代码结构和类很相似。又有点像实现逻辑代码的接口。当使用```use```的时候，Trait的代码就好像拷贝到use所在的位置取代use。从这个角度来看，更像C的```宏定义```。
```
Trait Singleton{
        //存放实例
        private static $_instance = null;
        //私有化克隆方法
        private function __clone(){

        }

        //公有化获取实例方法
        public static function getInstance(){
            $class = __CLASS__;
            if (!(self::$_instance instanceof $class)){
                self::$_instance = new $class();
            }
            return self::$_instance;
        }
    }

class DB {
    private function __construct(){
        echo __CLASS__.PHP_EOL;
    }
}

class DBhandle extends DB {
    use Singleton;
    private function __construct(){
        echo "单例模式的实例被构造了";
    }
}
$handle=DBhandle::getInstance();

//注意若父类方法为public，则子类只能为pubic，
//若父类为private，子类为public ，protected，private都可以。
```

补充，大多数书籍介绍单例模式，都会讲三私一公，公优化静态方法作为提供对象的接口，私有属性用于存放唯一一个单例对象。私有化构造方法，私有化克隆方法保证只存在一个单例。

但实际上，虽然我们无法通过```new``` 关键字和```clone```出一个新的对象，但我们若想得到一个新对象。还是有办法的，那就是通过```序列化和反序列化```得到一个对象。私有化```__sleep()```和```__wakeup()```方法依然无法阻止通过这种方法得到一个新对象。或许真得要阻止，你只能去__wakeup添加删除一个实例的代码，保证反序列化增加一个对象，你就删除一个。不过这样貌似有点怪异。

单例模式也细分为```懒汉模式```和```饿汉模式```，感兴趣的朋友可以去了解一下。上面的代码实现是懒汉模式

--------

#二、工厂模式(Factor Pattern)与抽象工厂模式( Abstract Factor Pattern)

---------
**工厂模式(Factor Pattern)**，就是负责生成其他对象的类或方法，也叫工厂方法模式

**抽象工厂模式( Abstract Factor Pattern)**，可简单理解为工厂模式的升级版

**(一)为什么需要工厂模式**

1，工厂模式可以将对象的生产从直接new 一个对象，改成通过调用一个工厂方法生产。这样的封装，代码若需修改new的对象时，不需修改多处new语句，只需更改生产对象方法。

2，若所需实例化的对象可选择来自不同的类，可省略if-else多层判断，给工厂方法传入对应的参数，利用多态性，实例化对应的类。

**（二）工厂模式结构图**

1，工厂方法模式

![Factor Pattern](http://upload-images.jianshu.io/upload_images/5261067-158e6d7f276a3316.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


2，抽象工厂模式

![Abstract Factor Pattern](http://upload-images.jianshu.io/upload_images/5261067-ab7b6a84219683e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**（三）简单实现代码**

```
//工厂类
class Factor{   
    //生成对象方法
    static function createDB(){
        echo '我生产了一个DB实例';
        return new DB;
    }
}

//数据类
class DB{
    public function __construct(){
        echo __CLASS__.PHP_EOL;
    }
}

$db=Factor::createDB();
```


**（四）实现一个运算器**

```
//抽象运算类
abstract class Operation{
    abstract public function getVal($i,$j);//抽象方法不能包含方法体
}
//加法类
class OperationAdd extends Operation{
    public function getVal($i,$j){
        return $i+$j;
    }
}
//减法类
class OperationSub extends Operation{
    public function getVal($i,$j){
        return $i-$j;
    }
}

//计数器工厂
class CounterFactor {
    private static $operation;
    //工厂生产特定类对象方法
    static function createOperation(string $operation){
        switch($operation){
            case '+' : self::$operation = new OperationAdd;
                break;
            case '-' : self::$operation = new OperationSub;
                break;
        }
        return self::$operation;
    }
}

$counter = CounterFactor::createOperation('+');
echo $counter->getVal(1,2);
```

缺点：若是再增加一个乘法运算，除了增加一个乘法运算类之外，还得去工厂生产方法里面添加对应的case代码，违反了开放-封闭原则。

**解决方法（1）：通过传入指定类名**

```
//计算器工厂
class CounterFactor {
    //工厂生产特定类对象方法
    static function createOperation(string $operation){
        return new $operation;
    }
}
class OperationMul extends Operation{
    public function getVal($i,$j){
        return $i*$j;
    }
}
$counter = CounterFactor::createOperation('OperationMul');
```

**解决方法（2）：通过抽象工厂模式**

这里顺带提一个问题：如果我系统还有个生产一个文本输入器工厂，那么那个工厂和这个计数器工厂又有什么关系呢。

> 抽象高于实现

其实我们完全可以抽象出一个抽象工厂，然后将对应的对象生产交给子工厂实现。代码如下

```
//抽象运算类
abstract class Operation{
    abstract public function getVal($i,$j);//抽象方法不能包含方法体
}
//加法类
class OperationAdd extends Operation{
    public function getVal($i,$j){
        return $i+$j;
    }
}
//乘法类
class OperationMul extends Operation{
    public function getVal($i,$j){
        return $i*$j;
    }
}
//抽象工厂类
abstract class Factor{
    abstract static function getInstance();
}
//加法器生产工厂
class AddFactor extends Factor {
    //工厂生产特定类对象方法
    static function getInstance(){
        return new OperationAdd;
    }
}
//减法器生产工厂
class MulFactor extends Factor {
    static function getInstance(){
        return new OperationMul;
    }
}
//文本输入器生产工厂
class TextFactor extends Factor{
    static function getInstance(){}
}
$mul = MulFactor::getInstance();
echo $mul->getVal(1,2);
```
-------

#三、建造者模式(Builder Pattern)

------
**建造者模式(Builder Pattern)**：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

>建造者模式是一步一步创建一个复杂的对象，它允许用户只通过指定复杂对象的类型和内容就可以构建它们，用户不需要知道内部的具体构建细节。建造者模式属于对象创建型模式。根据中文翻译的不同，建造者模式又可以称为生成器模式。

**(一)为什么需要建造者模式**
1，对象的生产需要复杂的初始化，比如给一大堆类成员属性赋初值，设置一下其他的系统环境变量。使用建造者模式可以将这些初始化工作封装起来。
2，对象的生成时可根据初始化的顺序或数据不同，而生成不同角色。

**(二)建造者模式结构图**

![Builder Pattern](http://upload-images.jianshu.io/upload_images/5261067-984c9e515a875a66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**(三)模式应用**
在很多游戏软件中，地图包括天空、地面、背景等组成部分，人物角色包括人体、服装、装备等组成部分，可以使用建造者模式对其进行设计，通过不同的具体建造者创建不同类型的地图或人物

**(四)设计实例**
如果我们想创造出有一个person类，我们通过实例化时设置的属性不同，让他们两人一个是速度快的小孩，一个是知识深的长者

```
class person {
    public $age;
    public $speed;
    public $knowledge;
}
//抽象建造者类
abstract class Builder{
    public $_person;
    public abstract function setAge();
    public abstract function setSpeed();
    public abstract function setKnowledge();
    public function __construct(Person $person){
        $this->_person=$person;
    }
    public function getPerson(){
        return $this->_person;
    }
}
//长者建造者
class OlderBuider extends Builder{
    public function setAge(){
        $this->_person->age=70;
    }
    public function setSpeed(){
        $this->_person->speed="low";
    }
    public function setKnowledge(){
        $this->_person->knowledge='more';
    }
}
//小孩建造者
class ChildBuider extends Builder{
    public function setAge(){
        $this->_person->age=10;
    }
    public function setSpeed(){
        $this->_person->speed="fast";
    }
    public function setKnowledge(){
        $this->_person->knowledge='litte';
    }
}
//建造指挥者
class Director{
    private $_builder;
    public function __construct(Builder $builder){
        $this->_builder = $builder;
    }
    public function built(){
        $this->_builder->setAge();
        $this->_builder->setSpeed();
        $this->_builder->setKnowledge();
    }
}
//实例化一个长者建造者
$oldB = new OlderBuider(new Person);
//实例化一个建造指挥者
$director = new Director($oldB);
//指挥建造
$director->built();
//得到长者
$older = $oldB->getPerson();

var_dump($older);
```

**(五)总结** 

使用建造者模式时，我们把创建一个person实例的过程分为了两步.

一步是先交给对应角色的建造者，如长者建造者。这样的好处就把角色的属性设置封装了起来，我们不用在new一个person时，因为要得到一个older角色的实例，而在外面写了一堆$older->age=70。


另一步是交给了一个建造指挥者，调了一个built方法，通过先设置age，再设置Speed的顺序，初始化这个角色。当然在这个例子中，初始化的顺序，是无所谓的。但是如果对于一个建造汉堡，或是地图，初始化的顺序不同，可能就会得到不同的结果。

也许，你会说，我直接设置也很方便呀。是的，对于某些情况是这样的。但是如果你考虑，我现在想增加一个青年人角色呢？如果我现在想让建造有初始化有三种不同的顺序呢？

如果你使用了建造者模式，这两个问题就简单了，增加一个青年人角色，那就增加一个青年年建造者类。初始化三种不同的顺序，那么就在指挥建造者中增加两种建造方法。

------
#原型模式（Prototype Pattern）
------
**原型模式（Prototype Pattern）**：与工厂模式类似，都是用来创建对象的。利用克隆来生成一个大对象，减少创建时的初始化等操作占用开销

**(一)为什么需要原型模式**

1，有些时候，我们需要创建多个类似的大对象。如果直接通过new对象，开销很大，而且new完还得进行重复的初始化工作。可能把初始化工作封装起来的，但是对于系统来说，你封不封装，初始化工作还是要执行。,

2，原型模式则不同，原型模式是先创建好一个原型对象，然后通过clone这个原型对象来创建新的对象，这样就免去了重复的初始化工作，系统仅需内存拷贝即可。

**(二)原型模式结构图**

![Prototype Pattern](http://upload-images.jianshu.io/upload_images/5261067-44abf9fd930c2ad5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**(三)简单实例**

如果说，我们现在正开发一个游戏，有不同的地图，地图大小都是一样的，并且都有海洋，但是不同的地图温度不一样。

```
<?php
//抽象原型类
Abstract class Prototype{
    abstract function __clone();
}
//具体原型类
class Map extends Prototype{
    public $width;
    public $height;
    public $sea;
    public function setAttribute(array $attributes){
        foreach($attributes as $key => $val){
            $this->$key = $val;
        }
    }
    public function __clone(){}
}
//海洋类.这里就不具体实现了。
class Sea{}

//使用原型模式创建对象方法如下
//先创建一个原型对象
$map_prototype = new Map;
$attributes = array('width'=>40,'height'=>60,'sea'=>(new Sea));
$map_prototype->setAttribute($attributes);
//现在已经创建好原型对象了。如果我们要创建一个新的map对象只需要克隆一下
$new_map = clone $map_prototype;

var_dump($map_prototype);
var_dump($new_map);
```

通过上面的代码，我们可以发现利用原型模式，只需要实例化并初始化一个地图原型对象。以后生产一个地图对象，都可以直接通过clone原型对象产生。省去了重新初始化的过程。

但是上面的代码还是存在一些问题。那就是它只是一个```浅拷贝```，什么意思呢？map原型对象有一个属性sea存放了一个sea对象，在调用setAttribute的时候，对象的赋值方式默认是引用。而当我们克隆map对象时，直接克隆了map的sea属性，这就使得克隆出来的对象与原型对象的sea属性对指向了，同一个sea对象的内存空间。如果这个时候，我们改变了克隆对象的sea属性，那么原型对象的sea属性也跟着改变。

这显然是不合理的，我们想要的结果应该是```深拷贝```，也就是改变克隆对象的所有属性，包括用来存放sea这种其他对象的属性时，也不影响原型对象。
当然，讲到这里你可以当我在胡说。但我还是建议你打印下原型对象和克隆对象，看一下他们的sea属性吧，然后去好好了解一下什么叫```深拷贝```和```浅拷贝```。

**(三)深拷贝的实现**

深拷贝的实现，其实也简单，我们只要实现Map类的克隆方法就行了。这就是我们为什么要定义一个抽象原型类的原因。我们利用抽象类，强制所有继承的具体原型类都必须来实现这个克隆方法。改进如下：

```
//具体原型类
class Map extends Prototype{
    public $width;
    public $height;
    public $sea;
    public function setAttribute(array $attributes){
        foreach($attributes as $key => $val){
            $this->$key = $val;
        }
    }
     //实现克隆方法，用来实现深拷贝
    public function __clone(){
        $this->sea = clone $this->sea;
    }
}
```

到这里原型模式就算实现了，但是我觉还可以进一步进行封装，利用工厂模式或建造者模式的思想。

**(四)延伸**

举个例子，如果我们在克隆这个地图对象的同时我们还需要进行一下系统设置，或是说我们想给原型对象的clone_id属性赋值当前已经拷贝了多少个对象的总数量？

我们可以把clone这个动作封装到一个类似的工厂方法里面去，简单地实现一下，虽然不咋严谨。

```
<?php
//抽象原型类
Abstract class Prototype{
    abstract function __clone();
}
//具体原型类
class Map extends Prototype{
    public $clone_id=0;
    public $width;
    public $height;
    public $sea;
    public function setAttribute(array $attributes){
        foreach($attributes as $key => $val){
            $this->$key = $val;
        }
    }
    //实现克隆方法，用来实现深拷贝
    public function __clone(){
        $this->sea = clone $this->sea;
    }
}
//海洋类.这里就不具体实现了。
class Sea{}
//克隆机器
class CloneTool{
    static function clone($instance,$id){
        $instance->clone_id ++;
        system_write(get_class($instance));
        return clone $instance;
    }
}
//系统通知函数
function system_write($class){
    echo "有人使用克隆机器克隆了一个{$class}对象".PHP_EOL;
}

//使用原型模式创建对象方法如下
//先创建一个原型对象
$map_prototype = new Map;
$attributes = array('width'=>40,'height'=>60,'sea'=>(new Sea));
$map_prototype->setAttribute($attributes);
//现在已经创建好原型对象了。如果我们要创建一个新的map对象只需要克隆一下
$new_map = CloneTool::clone($map_prototype,1);

var_dump($map_prototype);
var_dump($new_map);
```

**(五)模型应用**

多用于创建大对象，或初始化繁琐的对象。如游戏中的背景，地图。web中的画布等等

-------
#五、创建型设计模式杂谈
------

1. 单例模式，工厂模式，建造者模式，原型模式都属于```创建型模式```。使用创建型模式的目的，就是为了创建一个对象。

2. 创建型模式的优点，在于如何把复杂的创建过程封装起来，如何降低系统的内销。

3. 我认为创建型模式的一个总要的思想其实就是```封装```，利用封装，把直接获得一个对象改为通过一个接口获得一个对象。这样最明显的优点，在于我们可以把一些复杂的操作也封装到接口里去，我们使用时直接调这个接口就可以了。具体的实现，我们在主线程序中就不再考虑。这样使得我们的代码看上去更少，更简洁。

4. ```单例模式```，我们把对象的生成从new改为通过一个静态方法，通过静态方法的控制，使得我们总是返回同一个实例给调用者，确保了系统只有一个实例

5. ```工厂模式```，也是一样，生成对象改为接口，还可以通过传参实例化不同的类。如果我们通过直接new的话，那么我们在主线代码中少不了要写if condition new 一个加法类，else new一个减法类。封装了之后，我们通过接口传参，还能利用多态的特性去替代if else语句。
而且我们遵循了单一原则，让类的功能单一。我们如果需要一个新功能，只需添加一个类，不用修改其他类的功能。这样使得代码的扩展性更好了。

6. ```建造者模式```，我们把初始化的工作和顺序，封装给了一个建造者和指挥者。如果，我们下次要建造的类属性，或是顺序不同。我们只需新建对应的建造者类或添加对应的指挥者方法，不必再去修改原代码。而且我们也省去了，这new对象后，还要写$attribut=array();这种一大串数组，然后调好几个方法去初始化的工作。

7. ```原型模式```，通过先创建一个原型对象，然后直接克隆，省去了new大对象带来的开销浪费。当然我们同样可以通过，封装clone这个动作。使得我们在clone的同时还可以做一些其他的准备工作。



----------
感谢阅读，由于笔者也是初学设计模式，能力有限，文章不可避免地有失偏颇
后续更新** PHP设计模式-结构型设计模式 **介绍，欢迎大家评论指正

----------
我最近的学习总结：
 - [PHP设计模式](http://www.jianshu.com/c/ecfd249dcea2)
 - [SPL](http://www.jianshu.com/c/825af8dcb549)


