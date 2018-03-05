# 前言
**数组重载**： 数组重载是指将对象作为数组使用的过程。具有这种功能的对象也称为索引器。
>数组重载是将对象作为数组使用的一个过程。这意味着允许使用 [ ] 数组语法访问数据。


**数组重载的学习令我想起一个问题**：
为何JavaScript中的一切皆对象？或者简单的说是为何JavaScript的数组是对象，可以以```arr.lenth```的方式调属性，以```arr.append()```的方式调方法呢？

当然对于大多的JavaScript Developer，是不会有这个疑问的。因为JavaScript中没有数组这种变量类型。而大多数第一门语言是C或者是PHP的开发者，第一次接触JavaScript的数组时，不免会觉得别扭。

如果你也是一个无法理解**JavaScript的数组是对象**的PHPer，觉得很不可思议，这篇文章或许能帮助你。好了，先把问题放下。我们来学习SPL的数组重载，说不定学着学着就明白了呢。


----
# 一、ArrayAccess Interface
[**ArrayAccess Interface：**](http://php.net/manual/en/class.arrayaccess.php)PHP提供的索引器接口，属于[预定义接口](http://php.net/manual/zh/reserved.interfaces.php)之一，是数组重载的核心，它提供了挂载到**Zend引擎**所必须的功能。

**索引器接口**：该接口提供了将对象作为数组访问的功能，也称为数组式访问接口或数组重载接口。

![ArrayAccess Interface](http://upload-images.jianshu.io/upload_images/5261067-e7994706c197ef64..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
实现ArrayAccess接口的类，就是一个**索引器**。可以**使用标准的数组语法读取和操纵**对象中的内容。
**注意：**ArrayAccess接口提供访问数组一样访问对象能力的接口，但不提供迭代访问能力。所以不能迭代访问该索引器，即不能```foreach```。除非该索引器同时实现了**迭代器接口**。同理不能对该索引器使用```count```计数函数，除非其实现**计数器接口**

**ArrayAccess Interface 实现：**

```
<?php
//自定义索引器类 实现 ArrayAccess接口
class MyArray implements ArrayAccess{
	protected $_arr =array() ;	//用于存放数据
	
	//设置或替换给定偏移量上的数据
	public function offsetSet($offset,$value){
		$this->_arr[$offset] = $value;
	}
	//返回指定偏移量位置上的数据
	public function offsetGet($offset){
		return $this->_arr[$offset];
	}
	//判断指定的偏移量是否存在于数组中
	public  function offsetExists($offset){
		return array_key_exists($offset,$this->_arr);
	}
	//删除指定偏移量位置上的数据
	public function OffsetUnset($offset){
		unset($this->_arr[$offset]);
	}
}

//使用示例
$myArr = new MyArray;
//赋值
$myArr['first']=1;
$myArr['second']=2;
//使用$myArr['second']
echo $myArr['second'].PHP_EOL;
//改变$myArr['second']值
$myArr['second']='two';
//再次使用$myArr['second']
echo $myArr['second'].PHP_EOL;
//删除$myArr['second']
unset($myArr['second']);
//再次使用$myArr['second']
echo $myArr['second'].PHP_EOL;

```

**运行结果：**


![image](http://upload-images.jianshu.io/upload_images/5261067-5934b2959f73a65d..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里我们可以看到实现ArrayAccess接口后，我们可以把一个对象当做数组一样**赋值**，**使用**，**删除**某个键值。
为什么实现该接口，就有这种功能呢？我们来一探究竟。与往常的套路一样我们在每个方法里，加上怎么一句

```
echo __METHOD__,PHP_EOL;
```

```
<?php
//自定义索引器类 实现 ArrayAccess接口
class MyArray implements ArrayAccess{
	protected $_arr =array() ;	//用于存放数据
	
	//设置或替换给定偏移量上的数据
	public function offsetSet($offset,$value){
		echo __METHOD__,PHP_EOL;
		$this->_arr[$offset] = $value;
	}
	//返回指定偏移量位置上的数据
	public function offsetGet($offset){
		echo __METHOD__,PHP_EOL;
		return $this->_arr[$offset];
	}
	//判断指定的偏移量是否存在于数组中
	public  function offsetExists($offset){
		echo __METHOD__,PHP_EOL;
		return array_key_exists($offset,$this->_arr);
	}
	//删除指定偏移量位置上的数据
	public function OffsetUnset($offset){
		echo __METHOD__,PHP_EOL;
		unset($this->_arr[$offset]);
	}
}

//使用示例
$myArr = new MyArray;
//赋值
$myArr['first']=1;
$myArr['second']=2;
//使用$myArr['second']
echo $myArr['second'].PHP_EOL;
//改变$myArr['second']值
$myArr['second']='two';
//再次使用$myArr['second']
echo $myArr['second'].PHP_EOL;
//删除$myArr['second']
unset($myArr['second']);
//再次使用$myArr['second']
echo $myArr['second'].PHP_EOL;
```
**再次运行：**


![image](http://upload-images.jianshu.io/upload_images/5261067-6284e9a9fdac3000..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过分析，我们可以知道：当我们实现```ArrayAccess```接口后，我们以数组的方式给索引器对象赋值时，PHP自动帮我们调用了```offsetSet```方法。```echo``` 这个索引器对象某个键值时，则调用了```offsetGet```方法。```isset```判断该键值时，则自动调用```offsetExists```方法。删除该键值时，则自动调用```offsetUnset```方法。

通过上面的代码，我们可以看到实现ArrayAccess接口的一个简单形式。**它为我们演示了数组机制是如何运作的**。

>提供ArrayAccess接口的主要原因是并非所有的集合都是基于真实的数组。使用ArrayAccess接口的可能会将请求代理到面向服务的架构（SOA）的后端程序，或者其他形式的离线程序。这允许推迟底层数组的获取时间，直到它被实际访问时才去获取这些数据。

然而，对于大多数情况来说，可能会使用数组作为底层数据的表达形式。然后，你将会向这个类添加处理这一数据的方法。为实现这一目的，SPL提供了内置的**ArrayObject**类。



------
# 二、Countable Interface

[**Countable Interface：**](http://php.net/manual/zh/class.countable.php)SPL提供的计数接口，实现该接口的类可被用于**count（）**函数计数，并返回数据长度。

![Countable Interface](http://upload-images.jianshu.io/upload_images/5261067-7e6332f1bca25cb0..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们知道，当一个类实现了ArrayAccess接口，就可以把它当成一个数组一样操作。但是并不能使用count函数进行计数。如果我们想让这个对象更像一个数组，就必须实现计数接口，为它提供计数能力。

实现Countable接口非常简单，只需要实现它的抽象方法count方法。由手册我们知道，count方法必须返回一个int类型的值，实际上这个值就是Array对象的有效元素数目。

**代码示例：**

```
//自定义数组对象实现Countable接口
class MyArray implements Countable{
	protected $_arr = array(1,2,3); //为演示方便直接赋值
	public function count(){
		return count($this->_arr);
	}
}
//实例化自定义数组对象
$myArr = new MyArray;
//计数
echo count($myArr);
```

**运行结果：**
![image](http://upload-images.jianshu.io/upload_images/5261067-8adf302f10300d1c..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

同样的，我们加上一句
```
echo __METHOD__,PHP_EOL;
```

```
<?php
//自定义数组对象实现Countable接口
class MyArray implements Countable{
	protected $_arr = array(1,2,3); //为演示方便直接赋值
	public function count(){
		echo __METHOD__,PHP_EOL;
		return count($this->_arr);
	}
}
//实例化自定义数组对象
$myArr = new MyArray;
//计数
echo count($myArr);
```
**再次运行：**

![image](http://upload-images.jianshu.io/upload_images/5261067-73ea25d85d790494..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如我们意料中的一样，实现Countable的接口后，当我们把这个对象传递给Count函数时，PHP会帮我们自动调用对象里的Count方法。

当然，这个有一个问题：
为什么SPL内置的一个实现Countable接口的类ArrayObject，实现count方法时，方法体里无实现代码，也没有返回值，但是依然能够计数。而我们如果没有return，则默认是null。使用Count函数计数时，就会返回零。为什么ArrayObject不用返回呢？

![image](http://upload-images.jianshu.io/upload_images/5261067-245025731b491177..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这个很纳闷，如果你知道答案，请你告诉我。谢谢！
具体怎么回事，这里给出[ArrayObject源码地址](https://github.com/php/php-src/blob/master/ext/spl/spl.php)

好了，虽然存在一点疑问，但是不影响。Countable接口的实现就是怎么简单。

如果说我们有一个类同时实现了```ArrayAccess```，```Countable```，甚至还有```Iterator```接口，那么这个类的操作几乎和数组无差异了。实现```ArrayAccess```接口拥有数组访问，实现```Countable```接口拥有计数的能力，实现```Iterator```接口拥有迭代访问的能力。

----
# 三、ArrayObject class
[**ArrayObject :**](http://php.net/manual/zh/class.arrayobject.php)  ArrayObject是SPL内置的一个数组对象类，它同时实现了```ArrayAccess```，```Countable```，```IteratorAggregate```接口。提供了迭代，排序和处理数据非常有用的方法，与数组的使用几乎是无差别的。其中实现```IteratorAggregate```接口委托的迭代器是在构造方法传递参数中默认了```ArrayIterator```迭代器。


![image](http://upload-images.jianshu.io/upload_images/5261067-ee90d44ac91d31c7..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**ArrayObject简单使用：**

```
$myArr = new ArrayObject ;
//赋值
$myArr['first']=1;
$myArr['second']=7;
$myArr['third']=5;
$myArr['fourth']=9;
//打印
print_r($myArr);
//删除
unset($myArr['fourth']);
//遍历
foreach($myArr as $key => $val){
	echo $key.'=>'.$val.PHP_EOL;
}

```
**运行结果：**
![image](http://upload-images.jianshu.io/upload_images/5261067-2834ff900b403d78..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>ArrayObject内部是基于Array实现的，所以它的限制在于只能处理真实的已经完全填充好的数据。但是它可以为应用程序提供有价值的基类。

也就是说，如果你要自定义一个实现ArrayAccess接口的类，实际上很多时候我们直接继承ArrayObject就可以了。

>ArrayObject的好处在于，你自定义的类可以继承它可以得到与数组使用无差异的功能。

好了，到这里。你想想实例化一个SPL的ArrayObject对象，得到了什么？
>一个与数组使用无差异的对象

记住上面这句话，它意味着什么呢？我们再想一下为什么**JavaScript中数组也是对象**呢？SPL提供的```ArrayObject```不就和JavaScript的数组**很相似**了吗？打破思维的藩篱，我们换一个视角来看这个问题。

假如PHP一开始就和JavaScript一样，没有提供数组这种变量类型，只给我们提供了```ArrayObject```类。甚至```array()```函数返回的不是一个数组，而是一个```ArrayObject```的实例。我们不是也可以通过```ArrayObject```创造出一个与传统数组概念上使用无差异的**‘新数组’**吗。而这种新数组的使用与传统数组无差别，且这种新数组还能调用方法，调用属性。这不正是JavaScript中数组吗？那么这个时候，这种概念的数组，不就是所谓的**数组也是对象**吗？

所以说，JavaScript的数组从这个角度上看，就和PHP中SPL提供的```ArrayObject```的实例化数组对象一样。JavaScript没有像PHP这种*纯粹的*数组，它定义的数组就已经是一个对象了。是的，PHP的数组相对于JavaScript是底层了，PHP完全可以**封装**出JavaScript那种数组对象。

当然，相较于JavaScript，只有PHP的数组不是对象，它就算是纯粹的数组吗？未必吧。PHP的数组与C语言比较呢？C语言的数组是**无法动态扩展**的，而PHP是可以的。从这个角度上来看，PHP中的数组是否相较与C语言也是一种封装呢？

---
# 四、使用ArrayObject内置方法进行排序

实现```ArrayAccess```接口的索引器,包括```ArrayObject```的实例虽与数组使用几乎无差异，但却无法使用PHP提供的```sort```，```asort```,```ksort```等函数进行排序。

```
$myArr = new ArrayObject ;
//赋值
$myArr['first']=1;
$myArr['second']=7;
$myArr['third']=5;
$myArr['fourth']=9;
//打印
asort($myArr);
```
**运行结果：**

![image](http://upload-images.jianshu.io/upload_images/5261067-ddd6013cdae474fb..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**但是如果你这样写：**
```
$arr = new ArrayObject;
$arr = [1,2,4,5,8,3];
print_r($arr);
sort($arr);     //排序
print_r($arr);
```
**运行结果：**
![image](http://upload-images.jianshu.io/upload_images/5261067-aef360c670adb26d..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
貌似是可以使用的，但是实际上是不行的。
因为你写```$arr = [1,2,4,5,8,3];```时，已经为```$arr```**重新赋值**一个数组了，```$arr```的变量类型已经不是```ArrayObject```对象，而是数组类型了。数组当然能调用```asort```函数咯。

所以，实现```ArrayAccess```接口的数组对象无法使用PHP自带的排序函数进行排序，这点你要注意。
不过，方便的是SPL中```ArrayObject```类已经拥有```asort```,```ksort```等方法它让我们可以实现类似于JavaScript那种操作，调用```$arr->asort();```给这个数组对象排序。

**代码示例：**
```
$myArr = new ArrayObject ;
//赋值
$myArr['first']=1;
$myArr['second']=7;
$myArr['third']=5;
$myArr['fourth']=9;
//排序
$myArr->asort();
//打印
print_r($myArr);
```
**运行结果：**

![image](http://upload-images.jianshu.io/upload_images/5261067-d9aed3bb89b6e07f..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----

# 五、SPL购物车
>ArrayObject的最常见应用是在web购物车中，令购物车类继承自ArrayObject，这样你实例化的购物车，不仅是一个对象而且还是一个数组。

**代码示例：**
```
<?php
//商品类
class Product{
	public $_name;	//商品名称
	public $_price;	//商品价格
	public function __construct($name,$price){
		$this->_name = $name;
		$this->_price = $price;
	}
}
//购物车类
class Cart extends ArrayObject{
	//计算购物车商品总价格
	public function sum(){
		$num = 0;
		foreach($this as $product){
			$num+=$product->_price ;
		}
		return PHP_EOL.'购物车总价 : '.$num.PHP_EOL;
	}
}

//实例化商品
$book = new Product('<补刀心法>',57);
$pen = new Product('破铁牌钢笔',2);
//实例化购物车
$cart = new Cart;
$cart[] = $book;
$cart[] = $pen;
//查看打印
print_r($cart);
//查看购物车商品总数
echo count($cart);
//计算购物车商品总价格
echo $cart->sum();
```
**运行结果：**

![image](http://upload-images.jianshu.io/upload_images/5261067-6b24fae48fffb91f..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从上面代码可以看到，通过让购物车类继承ArrayObject，我们可以以数组的形式添加商品，计算商品个数，购物车总金额。使用这种方法可以让购物车对象的操作变得更加简单方便。

当然这里我们要强调一点，```storage```属性是```ArrayObject```的私有属性，```Cart```作为```ArrayObject```的子类，是无法直接访问和使用的。但是由于```ArrayObject```的特性，我们可以在```Cart```中使用```$this```来访问父类的```storage```属性，同理可以使用```$this[0]```来访问第一个元素。

 ```ArrayObject``` 实现了```ArrayAccess```接口，所有继承它的类实例出来的对象都可以当做数组一般使用。其实，```ArrayObject```的构造方法还允许传入一个数组，它可以让我们把一个数组当成对象一般使用。
 ```
$arr = [1,8,3,9,5];
$arr = new ArrayObject($arr);
$arr->append(2);  //添加一个元素
echo $arr->offsetGet($arr->count()-1);   //获取最后一个数据 
 ```
**运行结果：**


![image](http://upload-images.jianshu.io/upload_images/5261067-c93be8ea86111685..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
----
# 六、使用对象作为数组键值
有些时候，我们或许需要将对象的作为数组键值。在PHP中如果直接这么做，明显是不行的，会得到一个警告。
```
<?php
class MyObject{}
$a = new MyObject;
$arr = array($a=>'test');
```
**运行结果：**

![image](http://upload-images.jianshu.io/upload_images/5261067-d74f813f7b1695a5..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

幸运地是，SPL为我们提供一个获取对象散列值的函数**spl_object_hash()**
![image](http://upload-images.jianshu.io/upload_images/5261067-739934c3fd4f1d89..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这个函数可以为所有的对象实例创建一个唯一的标识符。即使两个对象属于同一类，它的标识符依然是唯一的。
```
<?php
class MyObject{}
$a = new MyObject;
$b = new MyObject;

echo spl_object_hash($a);
echo PHP_EOL;
echo spl_object_hash($b);
```
![image](http://upload-images.jianshu.io/upload_images/5261067-bbd6860a93a7eeab..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
于是我们可以通过这个函数间接实现把一个对象当作数组的键值
```
<?php
class MyObject{}
$a = new MyObject;
//使用对象作为数组键值
$arr = array(spl_object_hash($a)=>'test');
//输出
echo $arr[spl_object_hash($a)];
```
**运行结果：**

![image](http://upload-images.jianshu.io/upload_images/5261067-728b5bfd541b8b2c..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用对象作为键值的好处是可以在一个数组中避免存放同个对象的多个引用。
```
<?php
class MyObject{}
$a = new MyObject;
$b = $a;

//赋值
$arr[spl_object_hash($a)] = $a;    
$arr[spl_object_hash($b)] = $b;
//打印
print_r($arr);
```
**运行结果:**
![image](http://upload-images.jianshu.io/upload_images/5261067-c090fe64a3ce9f6a..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)由于```$a```与```$b```是同一个对象实例，```spl_object_hash()```返回值相同，所以key值相同。```$arr[spl_object_hash($b)] = $b;```并没有为数组创建新元素。保证了数组中相同对象实例只有一个。


这里我们还要强调一点：虽然在一个脚本中对一个对象获取多次散列值，得到的结果是一样的。但是每次运行这个脚本获取这个对象的散列值是不同的。
```
<?php
class MyObject{}
$a = new MyObject;
$b = $a;


echo spl_object_hash($a);    //第一次获取
echo PHP_EOL;
echo spl_object_hash($b);   //第二次获取
```

**运行结果：**
![image](http://upload-images.jianshu.io/upload_images/5261067-3e7ecf9c833ccb48..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----
**上一篇：**[**SPL迭代器接口介绍**](https://juejin.im/entry/59151dc8a22b9d0058fcd4f8)

感谢阅读，由于笔者能力有限，文章不可避免地有失偏颇
后续更新SPL的其他内容，欢迎大家评论指正

----------
我最近的学习总结：
 - [PHP设计模式](http://www.jianshu.com/c/ecfd249dcea2)
 - [SPL](http://www.jianshu.com/c/825af8dcb549)
----------
![欢迎大家关注我的微信公众号 火风鼎](http://upload-images.jianshu.io/upload_images/5261067-25aa166d07588633?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)