# 迭代器相关简介

**（一）迭代器和迭代器接口是什么**

 - **迭代器**： 

> 迭代器（iterator）有时又称游标（cursor）是程序设计的软件设计模式，可在容器（container，例如链表或阵列）上遍访的接口，设计人员无需关心容器的内容。

迭代器，你可以简单理解为一个可以foreach的对象。
 
- **迭代器接口**：

> 通过实现这个接口的类，就相当于一个迭代器。可以被用在foreach循环结构中。并且能够提供一些高级的数据访问模式。

**（二）迭代器接口的作用**

> SPL迭代器接口的作用在于帮组实现高级的迭代算法，允许为类创建精巧的数据访问方法。这些接口形成了创建迭代器的基础。可以直接实现这些接口去创建所需的迭代器。SPL同时也扩展定义了更多的内置迭代器类。

**（三）SPL提供的迭代器接口**

> SPL提供了6个关于迭代器的接口

 - *Traversable*：无法被单独实现的基本抽象接口，其他的迭代器接口都直接或间接继承自该接口
 - *Iterator*：直接继承自Traversable接口的两个基本迭代器接口之一
 - *SeekableIterator*：Iterator接口的扩展，实现该接口允许通过键值进行查找
 - *IteratorAggregate*：直接继承自Traversable接口的两个基本迭代器接口之一，允许将迭代器所需实现方法委托给一个实现Iterator接口的迭代器
 - *OuterIterator*：继承自Iterator接口，允许将多个迭代器包裹其中
 - *RecurisveIterator*：继承自Iterator接口，提供递归访问功能

当然，现在PHP手册已经将```Traversable```,```Iterator```,```IteratorAggregate```归结到预定义接口，所以你在SPL接口中没有看到它们很正常，请移步[预定义接口](http://php.net/manual/zh/reserved.interfaces.php)

----------

# SPL迭代器接口（一）—Traversable Interface

[**Traversable Interface：**](http://php.net/manual/zh/class.traversable.php)无法被单独实现的基本抽象接口，其他的迭代器接口都直接或间接继承自该接口。

![Traversable Interface](http://upload-images.jianshu.io/upload_images/5261067-037264c52cad1376.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> Traversable接口其实不是一个接口，更像是一个特性。因为只有用C语言编写的内部类才可以直接实现Traversable接口。任何需要实现Traversable接口的自定义类都通过实现从Traversable派生出来的的用户自定义接口才能做到。

简单点来说，就是如果你是写PHP代码的，那么Traversable跟你关系不大，因为你写的类无法直接实现Traversable接口（直接实现会报错）。

Traversable接口是给用C语言写PHP扩展的人准备的，只有C写的类才能直接实现Traversable接口。

所以对于PHPer来说，我们更应该关注的是SPL两个派生自Traversable接口的基础级别接口，Iterator接口和IteratorAggregate接口。这两个接口才是你写的类可以直接实现的。

** Traversable接口直接实现会报错：**

```
class MyTraversable implements Traversable{}
```

![Traversable error](http://upload-images.jianshu.io/upload_images/5261067-ce0345e5f05e8087.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----------

# SPL迭代器接口（二）—Iterator Interface


[**Iterator Interface：**](http://php.net/manual/zh/class.iterator.php)直接继承自Traversable接口的两个基本迭代器接口之一。


![Iterator Interface](http://upload-images.jianshu.io/upload_images/5261067-5ca058a545d552a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> Iterator接口允许一个类实现一个基本的迭代功能，从而使它可以被循环访问，根据键值访问和回滚。

** 示例代码：**

```
<?php
//自定义数组容器实现迭代器接口
class MyIterator implements Iterator
{
    protected $data = array('唐朝','宋朝','元朝');  //存放数据
    protected $index ;         //存放当前指针

    public function current()
    {   
        return $this->data[$this->index];
    }
    //指针+1
    public function next()
    {   
        $this->index ++;
    }
    //验证指针是否越界
    public function valid()
    {
        return $this->index < count($this->data);
    }
    //重置指针
    public function rewind()
    {
        $this->index = 0;
    }
    //返回当前指针
    public function key()
    {   
        return $this->index;
    }
}

//实例化一个迭代器
$container = new MyIterator();

//遍历数组容器
foreach($container as $key =>$dynasty){
    echo '如果有时光机，我想去'.$dynasty.PHP_EOL;
}
```

**运行结果**：


![](http://upload-images.jianshu.io/upload_images/5261067-1e570016e797e2a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


为什么实现Iterator接口就可以把一个`MyIterator`对象当成数组一样进行遍历呢？我们在代码中的每个方法中加入一句。

```
     echo __METHOD__,PHP_EOL;
```




```
<?php
//自定义数组容器实现迭代器接口
class MyIterator implements Iterator
{
    protected $data = array('唐朝','宋朝','元朝');  //存放数据
    protected $index ;         //存放当前指针

    //返回当前指针指向数据
    public function current()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->data[$this->index];
    }
    //指针+1
    public function next()
    {   
        echo __METHOD__,PHP_EOL;
        $this->index ++;
    }
    //验证指针是否越界
    public function valid()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->index < count($this->data);
    }
    //重置指针
    public function rewind()
    {   
        echo __METHOD__,PHP_EOL;
        $this->index = 0;
    }
    //返回当前指针
    public function key()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->index;
    }
}

//实例化一个迭代器
$container = new MyIterator();

//遍历数组容器
foreach($container as $key => $dynasty){
    echo '如果有时光机，我想去'.$dynasty.PHP_EOL;
}
```

** 再次运行 **：

![](http://upload-images.jianshu.io/upload_images/5261067-1ecdb68d9a08c59a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这个时候我们可以发现`foreach`语句的本质，实际上是PHP帮我们自动顺序调用了迭代器对象的`rewind->valid->currrent->key->next->valid->..........` 从而实现把类中的属性数组迭代出来。

** 具体流程图 **：

（key方法和current方法顺序倒了，以我们的测试为主，应该是current方法先调用）

![流程图](http://upload-images.jianshu.io/upload_images/5261067-94c43b7a4738cc18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


** 如果我们使用两次foreach呢？**

```
//遍历数组容器
foreach($container as $key =>$dynasty){
    echo '如果有时光机，我想去'.$dynasty.PHP_EOL;
}

echo '------第二次使用迭代器-----'.PHP_EOL;

foreach($container as $key =>$dynasty){
    echo '如果有时光机，我想去'.$dynasty.PHP_EOL;
}
```

** 再次运行 **：

![](http://upload-images.jianshu.io/upload_images/5261067-9b9394289aceae87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意foreach中不能使用break；


```
//遍历数组容器
foreach($container as $key =>$dynasty){
    $key==1 && break;
    echo '如果有时光机，我想去'.$dynasty.PHP_EOL;
}
```

![Parse error](http://upload-images.jianshu.io/upload_images/5261067-d9d0ad9e1cb30bda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----------

# SPL迭代器接口（三）—SeekableIterator Interface

[**SeekableIterator Interface：**](http://php.net/manual/zh/class.seekableiterator.php)    Iterator接口的扩展，实现该接口允许通过键值进行查找。



![SeekableIterator Interface](http://upload-images.jianshu.io/upload_images/5261067-69aee4f44a729909.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


SeekableIterator接口继承自Iterator接口。实现SeekableIterator接口，除了需实现Iterator的5个方法以外，还要求实现seek()方法，参数是元素的下标。

SeekableIterator接口通过要求实现seek方法，通过实现seek方法提供了按下标索引元素的能力。

** 代码示例：**

```
<?php
//自定义数组容器实现迭代器接口
class MySeekableIterator implements SeekableIterator
{
    protected $data = array('唐朝','宋朝','元朝');  //存放数据
    protected $index ;         //存放当前指针
    //按下标返回元素
    public function seek($key){
        return $this->data[$key];
    }
    //返回当前指针指向数据
    public function current()
    {   
        return $this->data[$this->index];
    }
    //指针+1
    public function next()
    {   
        $this->index ++;
    }
    //验证指针是否越界
    public function valid()
    {   
        return $this->index < count($this->data);
    }
    //重置指针
    public function rewind()
    {   
        $this->index = 0;
    }
    //返回当前指针
    public function key()
    {   
        return $this->index;
    }
}

//实例化一个迭代器
$container = new MySeekableIterator();
//遍历数组容器
foreach($container as $key =>$dynasty){
    echo '如果有时光机，我想去'.$dynasty.PHP_EOL;
}
//按下标返回元素
echo '我最想去的是'.$container->seek(1);
```

** 运行结果：**

![](http://upload-images.jianshu.io/upload_images/5261067-8c0fa2eca60d3e80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----------

# SPL迭代器接口（四）—IteratorAggregate Interface

[**IteratorAggregate Interface：**](http://php.net/manual/zh/class.iteratoraggregate.php)    直接继承自Traversable接口的两个基本迭代器接口之一，允许将迭代器所需实现方法委托给一个实现Iterator接口的迭代器

![IteratorAggregate Interface](http://upload-images.jianshu.io/upload_images/5261067-51b5a6b047a5d5ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

IteratorAggregate接口与Iterator接口一样继承自Traversable接口。

> IteratorAggregate接口是用来将Iterator接口要求实现的5个迭代器方法委托给其他类的。它可以让你在类的外部实现迭代功能，并允许重新使用常用的迭代器方法，而不是在编写的每个可迭代类中重复这些方法。

简单来说，实现IteratorAggregate接口的类和实现Iterator接口的类一样也是一个迭代器，不过它不需要实现Iterator接口的5个要求实现的方法，它只需实现`getIterator`方法把Iterator接口要求实现的方法委托给一个已经实现的迭代器。

实现一个迭代器前，必需先实现另一个迭代器？有点绕是吧。我们看代码就明白了。


** 代码示例：**

```
<?php
//自定义迭代器实现IteratorAggregate接口
class MyIteratorAggregate implements IteratorAggregate{
    protected $arr = array(1,2,3);
    //IteratorAggregate接口要求实现的方法
    public function getIterator(){
        //返回一个实现Iterator接口类的实例
        return new MyIterator($this->arr);
    }
}

//MyIterator实现Iterator接口，Iterator接口那节讲过。
//这里我们做点修改，让它能接收外部数据，实际上就是实现__construct方法
class MyIterator implements Iterator
{
    protected $data = array();  //存放数据
    protected $index ;         //存放当前指针
    //构造方法接收外部传入数据
    public function __construct(Array $data)
    {   
        $this->data = $data;
    }
    //返回当前指针指向数据
    public function current()
    {   
        return $this->data[$this->index];
    }
    //指针+1
    public function next()
    {   
        $this->index ++;
    }
    //验证指针是否越界
    public function valid()
    {   
        return $this->index < count($this->data);
    }
    //重置指针
    public function rewind()
    {   
        $this->index = 0;
    }
    //返回当前指针
    public function key()
    {   
        return $this->index;
    }
}
//实例化一个迭代器
$container = new MyIteratorAggregate;
//使用MyIteratorAggregate迭代器
foreach($container as $key => $val){
    echo $val.PHP_EOL;
}

```

** 运行结果：**

![](http://upload-images.jianshu.io/upload_images/5261067-9543be9078b7415c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


也许你会说，这功能不是和实现Iterator接口的迭代器一样么？是的，我们想一下如果有十个要实现迭代器类，我们如果都实现Iterator接口的话，那么这十个类都要实现Iterator接口要求实现的5个方法。这些类中都重复了这5个方法，代码不是很冗余吗？

所以我们只需先写一个实现Iterator接口的类，再让这十个迭代器类实现IteratorAggregate接口，代码不就简洁多了吗?

而且美妙地是，SPL内置的迭代器类中，已经有一个实现Iterator接口`ArrayIterator`类，它仿佛就是专门为了给实现IteratorAggregate接口的getIterator方法而存在的。也就是说我们实现getIterator方法其实不需要直接写一个实现Iterator接口的类，直接使用SPL内置的ArrayIterator就可以了。

```
public function getIterator(){
        //返回一个实现Iterator接口类的实例
        return new ArrayIterator($this->arr);
    }
```


当然，为了探究实现IteratorAggregate接口的迭代器是如何工作的，我们还是暂时还是使用我们自己实现的MyIterator类。

那么，foreach的时候，实现IteratorAggregate接口的迭代器是怎么工作的呢？它为什么能只实现`getIterator`方法就可以实现迭代呢？
老规矩 ，我们在MyIteratorAggregate和MyIterator方法中，加上这一句

```
       echo __METHOD__,PHP_EOL;
```


```
<?php
//自定义迭代器实现IteratorAggregate接口
class MyIteratorAggregate implements IteratorAggregate{
    protected $arr = array(1,2,3);
    //IteratorAggregate接口要求实现的方法
    public function getIterator(){
        echo __METHOD__,PHP_EOL;
        //返回一个实现Iterator接口类的实例
        return new MyIterator($this->arr);
    }
}

//MyIterator实现Iterator接口，Iterator接口那节讲过。
//这里我们做点修改，让它能接收外部数据，实际上就是实现__construct方法
class MyIterator implements Iterator
{
    protected $data = array();  //存放数据
    protected $index ;         //存放当前指针
    //构造方法接收外部传入数据
    public function __construct(Array $data)
    {   
        echo __METHOD__,PHP_EOL;
        $this->data = $data;
    }
    //返回当前指针指向数据
    public function current()
    {   
        return $this->data[$this->index];
    }
    //指针+1
    public function next()
    {   
        echo __METHOD__,PHP_EOL;
        $this->index ++;
    }
    //验证指针是否越界
    public function valid()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->index < count($this->data);
    }
    //重置指针
    public function rewind()
    {   
        echo __METHOD__,PHP_EOL;
        $this->index = 0;
    }
    //返回当前指针
    public function key()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->index;
    }
}
//实例化一个迭代器
$container = new MyIteratorAggregate;
//使用MyIteratorAggregate迭代器
foreach($container as $key => $val){
    echo $val.PHP_EOL;
}
```

** 运行结果 **：

![](http://upload-images.jianshu.io/upload_images/5261067-818c3e21aaed054b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们结果就跟我们意料中的一样foreach的时候，PHP自动调用了MyIteratorAggregate的`getIterator`方法，然后实例化一个MyIterator对象，调用它的构造方法并把MyIteratorAggregate的数据$arr交给它。接下去就把迭代的事情委托给MyIterator对象了，所以我们看到后面又是顺序调用`rewind->valid->current->key->next...........`直到把MyIteratorAggregate交给它的数据全部遍历出来。

----------

# SPL迭代器接口（五）—OuterIterator Interface

[** OuterIterator Interface：** ](http://php.net/manual/zh/class.outeriterator.php)  继承自Iterator接口，允许将多个迭代器包裹其中。

![OuterIterator Interface](http://upload-images.jianshu.io/upload_images/5261067-ec6cd284ee549586.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> OuterIterator接口继承自Iterator接口。实现OuterIterator接口的迭代器类，允许将一个或多个迭代器包裹其中，并按顺序访问这些迭代器。

OuterIterator比Iterator多了一个getInnerIterator方法，该方法根据定义的返回值类型，可以知道应该返回当前正在访问的迭代器。

> OuterIterator与IteratorAggregate不一样，foreach的时候PHP是不会自动调用getInnerIterator方法的，我刚开始实现这个接口的时候就是以为foreach会自动调用。以为直接返回当前指向的迭代器，就能想IteratorAggregate那样，把迭代器遍历出来。

但是，实际上并不行。网上这方面的资料比较少。我参考了PHP源码里的内置的[IteratorIterator](https://github.com/php/php-src/blob/master/ext/spl/internal/iteratoriterator.inc)迭代器和[AppendIterator迭代器](https://github.com/php/php-src/blob/master/ext/spl/internal/appenditerator.inc)，实现了OuterIterator接口。代码如下，欢迎大家交流指正。

** 代码示例：**

```
<?php
//自定义MyOuterIterator实现OuterIterator接口
class MyOuterIterator implements OuterIterator{
    protected $iterators = array();  //存放迭代器
    protected $index;         //存放当前迭代器指针
    //构造方法，接收外部传入的迭代器
    public function __construct(Array $iterators){
        $this->iterators = $iterators;
    }
    //返回当前正在访问的迭代器
    public function getInnerIterator(){
        return $this->iterators[$this->index];
    }
    //返回当前指针指向数据
    public function current()
    {   
        echo __METHOD__,PHP_EOL;
        //返回当前指针指向MyIterator迭代器的指针，指向数据
        return $this->getInnerIterator()->current();
    }
    //数据指针+1
    public function next()
    {   
        echo __METHOD__,PHP_EOL;
        //当前指针指向MyIterator迭代器的指针，即数据指针+1
        $this->getInnerIterator()->next();
        //判断该数据指针是否越界，如果越界，则把迭代器指针下一个MyIterator迭代器。
        if(!$this->getInnerIterator()->valid()){
            //不判断是否小于count($this->iterators)的话，自增若越界会引起MyIterator初始化指针报错
            ++$this->index < count($this->iterators)  && $this->getInnerIterator()->rewind();
        }
    }
    //验证指针是否越界
    public function valid()
    {   
        echo __METHOD__,PHP_EOL;
        return  $this->index < count($this->iterators);
    }
    //重置指针
    public function rewind()
    {   
        $this->index = 0;
        //初始化MyIterator迭代器指针
        $this->getInnerIterator()->rewind();
    }
    //返回当前指针
    public function key()
    {   
        echo __METHOD__,PHP_EOL;
        //获取MyIterator指针
        $key = $this->getInnerIterator()->key();
        //为避免不同MyIterator迭代器实例有相同的key值，拼上前缀(迭代器存放位置下标)
        return $this->index.'-'.$key;
    }
}

//自定义的MyIterator类
class MyIterator implements Iterator
{
    protected $data = array();  //存放数据
    protected $index=0 ;         //存放当前指针
    //构造方法接收外部传入数据
    public function __construct(Array $data)
    {   
        $this->data = $data;
    }
    //返回当前指针指向数据
    public function current()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->data[$this->index];
    }
    //指针+1
    public function next()
    {   
        echo __METHOD__,PHP_EOL;
        $this->index ++;
    }
    //验证指针是否越界
    public function valid()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->index < count($this->data);
    }
    //重置指针
    public function rewind()
    {   
        echo __METHOD__,PHP_EOL;
        $this->index = 0;
    }
    //返回当前指针
    public function key()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->index;
    }
}

$arr1=array(1,2);
$arr2=array(3,4);
$iterators = array(new MyIterator($arr1),new MyIterator($arr2));
//实例化一个MyOuterIterator迭代器
$container = new MyOuterIterator($iterators);
//使用MyOuterIterator迭代器
foreach($container as $key => $val){
    echo $key.'=>'.$val.PHP_EOL;
}
```

** 运行结果：**

![](http://upload-images.jianshu.io/upload_images/5261067-cf0b7ef3baa29713.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/5261067-28fde3899ed7957b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


遗憾地是，上面的代码可以顺序访问不同类型迭代器，但局限于这些迭代器都是实现Iterator接口的迭代器。不能访问实现IteratorAggregate接口的迭代器。

  下面是实现方式可以顺序访问实现IteratorAggregate接口的不同类型迭代器，同样遗憾地是，它不支持实现Iterator接口的迭代器。

** 代码示例：**

```
//自定义MyOuterIterator实现OuterIterator接口
class MyOuterIterator implements OuterIterator{
    protected $iterators = array();  //存放迭代器
    protected $index;         //存放当前迭代器指针
    protected $iterator;        //存放当前MyIteratorAggregate生成的MyIterator迭代器
    //构造方法，接收外部传入的迭代器
    public function __construct(Array $iterators){
        $this->iterators = $iterators;
    }
    //返回当前正在访问的迭代器
    public function getInnerIterator(){
        return $this->iterator;
    }
    //返回当前指针指向数据
    public function current()
    {   
        echo __METHOD__,PHP_EOL;
        //返回当前指针指向MyIterator迭代器的指针，指向数据
        return $this->getInnerIterator()->current();
    }
    //数据指针+1
    public function next()
    {   
        echo __METHOD__,PHP_EOL;
        //当前指针指向MyIterator迭代器的指针，即数据指针+1
        $this->getInnerIterator()->next();
        //判断该数据指针是否越界，如果越界，则把迭代器指针下一个MyIteratorAggregate迭代器。
        if(!$this->getInnerIterator()->valid()){
            //不判断是否小于count($this->iterators)的话，自增若越界会引起MyIterator初始化指针报错
            if(++$this->index < count($this->iterators)){
                $this->iterator = $this->iterators[$this->index]->getIterator();
                $this->iterator->rewind();
            }
        }
    }
    //验证指针是否越界
    public function valid()
    {   
        echo __METHOD__,PHP_EOL;
        return  $this->index < count($this->iterators);
    }
    //重置指针
    public function rewind()
    {   
        $this->index = 0;
        $this->iterator = $this->iterators[$this->index]->getIterator();
        //初始化MyIterator迭代器指针
        $this->getInnerIterator()->rewind();
    }
    //返回当前指针
    public function key()
    {   
        echo __METHOD__,PHP_EOL;
        //获取MyIterator指针
        $key = $this->getInnerIterator()->key();
        //为避免不同MyIterator迭代器实例有相同的key值，拼上前缀(迭代器存放位置下标)
        return $this->index.'-'.$key;
    }


}

//自定义的MyIterator类
class MyIterator implements Iterator
{
    protected $data = array();  //存放数据
    protected $index=0 ;         //存放当前指针
    //构造方法接收外部传入数据
    public function __construct(Array $data)
    {   
        $this->data = $data;
    }
    //返回当前指针指向数据
    public function current()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->data[$this->index];
    }
    //指针+1
    public function next()
    {   
        echo __METHOD__,PHP_EOL;
        $this->index ++;
    }
    //验证指针是否越界
    public function valid()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->index < count($this->data);
    }
    //重置指针
    public function rewind()
    {   
        echo __METHOD__,PHP_EOL;
        $this->index = 0;
    }
    //返回当前指针
    public function key()
    {   
        echo __METHOD__,PHP_EOL;
        return $this->index;
    }
}
//自定义迭代器实现IteratorAggregate接口
class MyIteratorAggregate implements IteratorAggregate{
    protected $arr = array();
    public function __construct($arr){
        $this->arr = $arr;
    }
    //IteratorAggregate接口要求实现的方法
    public function getIterator(){
        echo __METHOD__,PHP_EOL;
        //返回一个实现Iterator接口类的实例
        return new MyIterator($this->arr);
    }
}

$arr1=array(1,2);
$arr2=array(3,4);
$iterators = array(new MyIteratorAggregate($arr1),new MyIteratorAggregate($arr2));
//实例化一个MyOuterIterator迭代器
$container = new MyOuterIterator($iterators);
//使用MyOuterIterator迭代器
foreach($container as $key => $val){
    echo $key.'=>'.$val.PHP_EOL;
}
```

** 运行结果: **

![](http://upload-images.jianshu.io/upload_images/5261067-62789e36c5548031.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](http://upload-images.jianshu.io/upload_images/5261067-7931251f7e0416d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

感觉有点复杂，可能我打开的方式不对，大家如果有可以同时支持Iterator和IteratorAggregate的实现方法，麻烦联系我(`fxlxzf10@163.com`)，让我也能学习。谢谢。

庆幸地是，PHP中已经有一个内置的实现OuterIterator接口，可以顺序访问不同类型迭代器的迭代器。它就是[AppendIterator](http://www.php.net/manual/en/class.appenditerator.php)迭代器，继承自[IteratorIterator](http://www.php.net/manual/zh/class.iteratoriterator.php)迭代器。

AppendIterator并没有直接实现OuterIterator，但它的父类IteratorIterator实现了OuterIterator。关于AppendIterator和IteratorIterator，我将会在SPL内置迭代器中介绍。欢迎大家到时候交流指正

> OuterIterator接口通常很少直接实现，经常是通过继承IteratorIterator。

----------

# SPL迭代器接口（六）—RecursiveIterator Interface
[** RecursiveIterator Interface：** ](http://php.net/manual/zh/class.recursiveiterator.php)   继承自Iterator接口，提供递归访问功能。


![RecursiveIterator Interface](http://upload-images.jianshu.io/upload_images/5261067-d9cceaa240fa00b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


RecursiveIterator与OuterIterator接口一样，继承自Iterator接口。

> RecursiveIterator接口的作用在于提供递归迭代访问功能。这种类型的迭代器接口可以表达为一个树形的数据结构，其中包含了节点元素和叶子元素。目录就是一个典型递归结构。

** RecursiveIterator接口实现代码：**

```
<?php
//自定义数组迭代器实现递归迭代器接口
class MyRecursiveIterator implements RecursiveIterator 
{ 
    protected $arr;      //存放数据
    protected $index;  //存放当前指针

    public function __construct(array $arr) 
    { 
        $this->arr = $arr; 
    } 
    //判断是否有子元素 
    public function hasChildren() 
    {     
        return is_array($this->arr[$this->index]); 
    }
    //返回子元素的迭代器实例
    public function getChildren() 
    { 
        return new self($this->arr[$this->index]);
    }
    //验证指针是否越界
    public function valid() 
    { 
        return isset($this->arr[$this->index]); 
    }
    //返回当前指针指向数据
    public function current() 
    { 
        return $this->arr[$this->index]; 
    }
    //指针指向下一个元素
    public function next() 
    { 
        $this->index++; 
    } 
    //重置指针
    public function rewind() 
    {     
        $this->index = 0; 
    } 
    //返回当前key值
    public function key() 
    { 
        return $this->index; 
    } 
} 
//递归函数----递归遍历所有元素
function recursive(MyRecursiveIterator $container){
    foreach ($container as $c => $v) {
        if ($container->hasChildren()) { //判断是否有子元素
            echo "第{$c}个元素含有子元素:".PHP_EOL; 
            recursive($container->getChildren()); //递归调用
        } else { 
            echo "第{$c}个元素：$v".PHP_EOL; 
        }
    }
}
//测试数组
$arr = array( 
   0,1,2,array(1,2),array(1,array(1,2,3))
);
//实例化MyRecursiveIterator迭代器
$container = new MyRecursiveIterator($arr);
//调用函数
recursive($container);
```

** 运行结果：**

![](http://upload-images.jianshu.io/upload_images/5261067-fc473d83ac84894d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


有点难看，我们让层级之间缩进一下

```
//递归函数----递归遍历所有元素
function recursive(MyRecursiveIterator $container,$i=0){
    foreach ($container as $c => $v) {
        echo str_repeat("\t",$i);//缩进
        if ($container->hasChildren()) { //判断是否有子元素
            echo "第{$c}个元素含有子元素:".PHP_EOL; 
            recursive($container->getChildren(),$i+1); //递归调用
        } else { 
            echo "第{$c}个元素：$v".PHP_EOL; 
        }
    }
}
```

** 再次运行：**


![](http://upload-images.jianshu.io/upload_images/5261067-3f86e736f0338e22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----------

后续更新SPL迭代器介绍，欢迎大家评论指正

----------
我最近的学习总结：
 - [PHP设计模式](http://www.jianshu.com/c/ecfd249dcea2)
 - [SPL](http://www.jianshu.com/c/825af8dcb549)
----------
![欢迎大家关注我的微信公众号 火风鼎](http://upload-images.jianshu.io/upload_images/5261067-e41dd2eb70ab0fe7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)