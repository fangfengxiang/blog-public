**代理模式（Proxy Pattern）**：构建了透明置于两个不同对象之内的一个对象，从而能够截取或代理这两个对象间的通信或访问。

**（一）为什么需要代理模式**

1，`远程代理`，也就是为了一个对象在不同地址空间提供局部代表。隐藏一个对象存在于不同地址空间的事实。

2，`虚拟代理`，根据需要来创建开销很大的对象，通过它来存放实例化需要很长时间的真实对象。

3，`安全代理`，用来控制真实对象的访问对象。

4，`智能指引`，当调用真实对象的时候，代理处理一些事情。

**（二）代理模式UML图**


![Proxy Pattern](http://upload-images.jianshu.io/upload_images/5261067-b44ee03078c7ed9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**（三）简单实例**

`案例一`：你想买一张学友哥的新唱片，以前你都是在县城CD店里买的。现在CD行业不景气，没得卖了。你只能找人去香港帮你代购一张。

```
<?php
//代理抽象接口
interface shop{
    public function buy($title);
}
//原来的CD商店，被代理对象
class CDshop implements shop{
    public function buy($title){
        echo "购买成功，这是你的《{$title}》唱片".PHP_EOL;
    }
}
//CD代理
class Proxy implements shop{
    public function buy($title){
        $this->go();
        $CDshop = new CDshop;
        $CDshop->buy($title);
    }
    public function go(){
        echo "跑去香港代购".PHP_EOL;
    }
}

//你93年买了张 吻别
$CDshop = new CDshop;
$CDshop->buy("吻别");
//14年你想买张 醒着做梦 找不到CD商店了，和做梦似的，不得不找了个代理去香港帮你代购。
$proxy = new Proxy;
$proxy->buy("醒着做梦");
```

`案例二`：通过代理实现MySQL的读写分离，如果是读操作，就连接127.0.0.1的数据库，写操作就读取127.0.0.2的数据库

```
<?php
class Proxy
{   
    protected $reader;
    protected $wirter;
    public function __construct(){
        $this->reader = new PDO('mysql:host=127.0.0.1;port=3306;dbname=CD;','root','password');
        $this->writer = new PDO('mysql:host=127.0.0.2;port=3306;dbname=CD;','root','password');
    }
    public function query($sql)
    {
        if (substr($sql, 0, 6) == 'select')
        {
            echo "读操作: ".PHP_EOL;
            return $this->reader->query($sql);
        }
        else
        {
            echo "写操作：".PHP_EOL;
            return  $this->writer->query($sql);
        }
    }
}
//数据库代理
$proxy = new Proxy;
//读操作
$proxy->query("select * from table");
//写操作
$proxy->query("INSERT INTO table SET title = 'hello' where id = 1");


//当然对于数据库来说，这里应该使用单例模式的方法来存放$reader和$writer,但我只是举个例子，不想把单例加进来把代码搞复杂。
//但是如果你要实现这样的一个数据库代理，我觉得还是有必要用上单例模式的知识
```


一句话来说，代理模式，就是在访问对象时通过一个代理对象去访问你想访问的对象。而在代理对象中，我们可以实现对访问对象的截断或权限控制等操作。
