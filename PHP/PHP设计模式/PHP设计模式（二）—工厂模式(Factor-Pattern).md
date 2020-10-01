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


后续更新，欢迎大家评论指正
欢迎大家关注我的微信公众号 火风鼎

![火风鼎](http://upload-images.jianshu.io/upload_images/5261067-e41dd2eb70ab0fe7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
