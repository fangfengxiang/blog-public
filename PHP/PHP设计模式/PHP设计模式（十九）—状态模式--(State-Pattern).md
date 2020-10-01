**状态模式  (State Pattern)** ：允许一个对象在其内部状态改变时改变它的行为，让不同状态的对象看起来似乎修改了它的类，或者说是看起来不是来自同一个类。

**(一)为什么需要状态模式**

1，将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为。

2，本模式简化了发起人类。发起人不再需要管理和保存其内部状态的一个个版本，客户端可以自行管理他们所要的这些状态的版本。

**(二)状态模式 UML图**

![State Pattern](http://upload-images.jianshu.io/upload_images/5261067-407873c0f8d0a341.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**(三)简单实例**

状态模式一个最妙的应用就是通过变化状态拥有不同的能力。比如我们以水为例，水如果是固态，那么它就能融化成液态，如果是液态那么它就能蒸发成气态，而气态也能凝华成固态。现在就让我们用程序来模拟这个过程。

```
<?php
//抽象状态类
abstract class State{
  abstract function handle();
}
//固态
class Solid extends State{
    public function handle(){
        echo '固态 =>融化 =>液态转化中'.PHP_EOL;
    }
}
class Liquid extends State{
    public function handle(){
        echo '液态 =>蒸发 =>气态转化中'.PHP_EOL;
    }
}
class Gas extends State{
    public function handle(){
        echo '气态 =>凝华 =>固态转化中'.PHP_EOL;
    }
}
//context环境类 -----water
class Water{
  protected $states = array();
  protected $current=0;
  public function __construct()
  {
      $this->states[]=new Solid;
      $this->states[]=new Liquid;
      $this->states[]=new Gas;
  }
  //水的变化
  public function change(){
    //告知当前状态
    echo '当前所处状态'.get_Class($this->states[$this->current]).PHP_EOL;
    //当前状态能力
    $this->states[$this->current]->handle();
    //状态变化
    $this->changeState();
  }
  //状态变化
  public function changeState()
  {
      $this->current++ == 2 && $this->current = 0;
  }
}



//实例化具体环境角色-----水
$water = new Water;
//水的能力变化   ---与它的状态相关
$water->change();
$water->change();
$water->change();
$water->change();
```


当然我们这里只是一个简单的示例，你完全可以让一个状态有多个能力，或者通过给water给一个对外的接口，通过传参使其转化为你指定的状态。
