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

缺点：

**(五)使用Trait关键字实现类似于继承单例类的功能**

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

//注意若父类方法为public，则子类只能为pubic，若父类为private，子类为public ，protected，private都可以。
```

补充，大多数书籍介绍单例模式，都会讲三私一公，公优化静态方法作为提供对象的接口，私有属性用于存放唯一一个单例对象。私有化构造方法，私有化克隆方法保证只存在一个单例。
 
但实际上，虽然我们无法通过new 关键字和clone出一个新的对象，但我们若想得到一个新对象。还是有办法的，那就是通过序列化和反序列化得到一个对象。私有化__sleep()和__wakeup()方法依然无法阻止通过这种方法得到一个新对象。或许真得要阻止，你只能去__wakeup添加删除一个实例的代码，保证反序列化增加一个对象，你就删除一个。不过这样貌似有点怪异。

后续更新，欢迎大家评论指正
欢迎大家关注我的微信公众号 火风鼎

![火风鼎](http://upload-images.jianshu.io/upload_images/5261067-e41dd2eb70ab0fe7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
