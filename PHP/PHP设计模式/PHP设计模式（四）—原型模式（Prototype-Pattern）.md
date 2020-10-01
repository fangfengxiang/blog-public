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

但是上面的代码还是存在一些问题。那就是它只是一个浅拷贝，什么意思呢？map原型对象有一个属性sea存放了一个sea对象，在调用setAttribute的时候，对象的赋值方式默认是引用。而当我们克隆map对象时，直接克隆了map的sea属性，这就使得克隆出来的对象与原型对象的sea属性对指向了，同一个sea对象的内存空间。如果这个时候，我们改变了克隆对象的sea属性，那么原型对象的sea属性也跟着改变。

这显然是不合理的，我们想要的结果应该是深拷贝，也就是改变克隆对象的所有属性，包括用来存放sea这种其他对象的属性时，也不影响原型对象。
当然，讲到这里你可以当我在胡说。但我还是建议你打印下原型对象和克隆对象，看一下他们的sea属性吧，然后去好好了解一下什么叫深拷贝和浅拷贝。

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

后续更新，欢迎大家评论指正
欢迎大家关注我的微信公众号 火风鼎

![火风鼎](http://upload-images.jianshu.io/upload_images/5261067-e41dd2eb70ab0fe7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
