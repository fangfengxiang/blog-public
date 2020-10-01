**空对象模式（Null Object Pattern）**：用一个空对象取代 NULL，减少对实例的检查。这样的空对象可以在数据不可用的时候提供默认的行为

**(一)为什么需要空对象模式**


> 解决在需要一个对象时返回一个null值，使其调用函数出错的情况

**(二)空对象模式UML图**

![Null Object Pattern](http://upload-images.jianshu.io/upload_images/5261067-5bf8009eb3d0cfbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


上图是Java的空对象模式UML图，网上很多PHP设计模式的代码实现都是照着上面这个UML图

实际上PHP在空对象模式的实现上比Java更加简单些。因为PHP有美妙的语法糖，魔术方法__call方法。

我们只要实现空对象的__call方法就可以实现空对象模式，并不需要使用nullobject去继承对应的抽象object

**(三)简单实例**

假设现在我们有这么一段代码

```
<?php
//测试类
class Person{
    public function code(){
        echo 'code makes me happy'.PHP_EOL;
    }
}


//定义一个生成对象函数，只有PHPer才允许生成对象
function getPerson($name){
    if($name=='PHPer'){
        return new Person;
    }
}

$phper = getPerson('PHPer');
$phper->code();
```

是的，现在这段代码会输出`code makes me happy`。如果有时候这个函数是别人调用的，它并没传入合适的参数呢？

```
$phper = getPerson('Javaer');
$phper->code();
```

这个时候就会报错了`error : Call to a member function code() on null`。是的$phper现在是一个null值，所以调用code方法就会报错

这种情况很常见，系统并没有返回一个我们期待的对象，而是返回了一个null值。所以多数情况下，我们的代码都要这样写

```
$phper = getPerson('Javaer');
if(!is_null($phper)){
    $phper->code();
}
```

或者是

```
if(is_object($phper)){
    $phper->code();
}
```

这样让太多的if判断不可避免地存在于我们的代码中
如果我们使用NullObject模式的话，我们就可以让函数没有返回值时返回一个nullobject对象。而不是一个null值（没有return 默认null值）


```
//测试类
class Person{
    public function code(){
        echo 'code makes me happy'.PHP_EOL;
    }
}


//空对象模式
class NullObject{}

//定义一个生成对象函数，只有PHPer才允许生成对象
function getPerson($name){
    if($name=='PHPer'){
        return new Person;
    }else{
        return new NullObject;
    }
}

$phper = getPerson('PHer');
$phper->code();
```

这个时候就不会再报一个null调用函数的错误了，但是会报一个`call to undefined method的错误`，这是由于NullObject对象没有code方法。这个时候我们只需实现魔术方法`__call`，就不会报错了。

```
//空对象模式
class NullObject{
    public  function __call($method,$arg){
        echo 'this is NullObject';
    }
}
```


我们可以通过返回一个NullObject对象来取代返回null，这样就不用在调用方法时判断是否为null，而且只要你实现了__call方法，不管真正的对象它原来是调用那个方法的，NullObject都可以去调用而且不报错（`实际是隐式调用了魔术方法__call`）。当然，如果你原本的逻辑是返回对象是null的话什么都不做，那么你可以让__call()什么都不做。或者你也可以让它抛出一个异常。
