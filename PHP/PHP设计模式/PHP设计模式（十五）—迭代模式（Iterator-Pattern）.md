**迭代模式（Iterator Pattern）**：迭代器模式可帮组构造特定的对象，那些对象能够提供单一的标准接口循环或迭代任何类型的可计数数据。

**(一)为什么需要迭代模式**

1，我们想要向遍历数组那样，遍历对象，或是遍历一个容器。

2，迭代器模式可以隐藏遍历元素所需的操作-

**(二)迭代模式 UML图**

![Iterator Pattern](http://upload-images.jianshu.io/upload_images/5261067-02170da8119ab419.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图是截取了《PHP设计模式》中迭代器的UML图，仅供参考。在PHP中我们对迭代模式的使用，通常只需实现Iterator接口即可。

下图给出实现 Interator接口 所需实现的方法

![Iterator Interface](http://upload-images.jianshu.io/upload_images/5261067-dff20ec6d9e18ed3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**(三)简单实例**

现在我们把一个类当做一个容器，让其实现Iterator接口，使其可以被遍历。

```
<?php
class ArrayContainer implements Iterator
{
    protected $data = array();
    protected $index ;

    public function __construct($data)
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

//初始化数组容器
$arr = array(0=>'唐朝',1=>'宋朝',2=>'元朝');
$container = new ArrayContainer($arr);

//遍历数组容器
foreach($container as $a => $dynasty){
    echo '如果有时光机，我想去'.$dynasty.PHP_EOL;
}
```

迭代器其实也类似于数据库的游标，可以在容器内上下翻滚，遍历它所需要查看的元素。

通过实现Iterator接口，我们就可以把一个对象里的数据当一个数组一样遍历。也许你会说我把一个数组直接遍历不就行了吗，为什么你还要把数组扔给容器对象，再遍历容器对象呢？是这样的，通过容器对象，我们可以隐藏我们foreach的操作。比如说，我想遍历时，一个元素输出，一个元素不输出怎么办呢？利用迭代器模式，你只需把容器对象中的next方法中的index++语句改为index+=2即可。这点，你可以试试。

为何实现一个Iterator接口就必须实现current那些方法呢？其实foreach容器对象的时候，PHP是自动帮我们依次调用了，valid，next这些方法。具体的调用顺序，还有Iterator接口的实现，其实是属于SPL（Standard PHP Library）的内容。对于迭代器接口的更多内容，我将会在SPL相关的文章中，进行介绍。欢迎大家到时指正交流。
