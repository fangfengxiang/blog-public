**享元模式 （Flyweight Pattern）:** 池技术的重要实现方式， 运用共享技术有效的支持大量的细粒度对象，用于减少创建对象的数量，以减少内存占用和提高性能。

**(一)为什么需要享元模式**

1，在有大量对象时，有可能会造成内存溢出，我们把其中共同的部分抽象出来，如果有相同的业务请求，直接返回在内存中已有的对象，避免重新创建。

2，系统有大量相似对象。

3，需要缓冲池的场景。

**(二)享元模式UML图**

![Flyweight Pattern](http://upload-images.jianshu.io/upload_images/5261067-32a06fa5694bd4b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**(三)简单实例**

这里引用《设计模式之禅》的例子，就是如果有一个报考信息系统，每个考生进来就实例化一个对象，传入他要报考的考点。由于考生分为30个考点，所以其实我们没有必要不停的实例化对象并释放，我们完全可以做一个缓冲池，若缓冲池中没有与这个考生相同考点的对象，则实例化一个，使用完不需要释放。下一个相同考点的考生进来后，只需复用这个对象。

```
<?php
//抽象享元对象
abstract class Flyweight{
    //考点
    public $address;
    //享元角色必修设置考点
    public function __construct($address){
        $this->address = $address;
    }
}
//具体享元角色  考生类
class ConcreteFlyweight extends Flyweight{
    //报考动作
    public function register(){
        echo "我的报考点是:{$this->address}".PHP_EOL;
    }
    //退出
    public function quit(){
        unset($this);
    }
}
//享元工厂 缓冲池
class FlyweightFactor{
    static private $students = array();
    static public function getStudent($address){
        $students =self::$students;
        //判断该键值是否存在
        if(array_key_exists($address,$students)){
            echo "缓冲池有考点为{$address}，从池中直接取".PHP_EOL;   
        }else{
            echo "缓冲池没有，创建了考点为{$address}的对象并放到池中".PHP_EOL;
            self::$students[$address] = new ConcreteFlyweight($address);
        }
        return self::$students[$address];
    }
}

//实例化学生对象
$student_1 = FlyweightFactor::getStudent('广州');
//报考
$student_1 ->register();
// 退出
$student_1->quit();
//第二个学生进来
$student_2 = FlyweightFactor::getStudent('东莞');
//报考
$student_2 ->register();
// 退出
$student_2->quit();
//第三个学生进来
$student_3 = FlyweightFactor::getStudent('广州');
//报考
$student_3 ->register();
// 退出
$student_3->quit();
```


这里我们可以看到，当第三个学生进来时，由于和第一个学生的报考地点一致，所以只需从缓冲池取出。

`ps:虽然ConcreteFlyweight的quit方法unset了$this，但是由于在FlyweightFactor中的students还存放着这个对象，所以unset只释放了变量$student_1,并没有完全删除这个对象，这就是student_3进来还能从缓冲池取得对象的原因`

享元模式在PHP中可能比较少遇，但在Java中常常有这种情况。就是代码产生了大量的对象，虽然使用完有释放，但是由于垃圾回收需要一定时间，导致内存被耗尽。PHP经常应用web编程，脚本执行时间很短（30秒）。所以很少遇见这种情况，甚至我们使用完变量，连unset()函数都不用调用，就等着脚步执行结束后，自动释放。但是如果你试过在cli模式下运行PHP脚本，做一些socket通信，发送邮件等长耗时或是多连接任务时，就难免会遇到这种情况。
