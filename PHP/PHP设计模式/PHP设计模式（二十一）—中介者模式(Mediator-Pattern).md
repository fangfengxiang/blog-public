**中介者模式(Mediator Pattern)**： 用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。中介者模式又称为调停者模式，它是一种对象行为型模式。

**(一)为什么需要中介者模式**

1，中介者模式可以使对象之间的关系数量急剧减少。

2，中转作用（结构性）：通过中介者提供的中转作用，各个同事对象就不再需要显式引用其他同事，当需要和其他同事进行通信时，通过中介者即可。该中转作用属于中介者在结构上的支持。

3，协调作用（行为性）：中介者可以更进一步的对同事之间的关系进行封装，同事可以一致地和中介者进行交互，而不需要指明中介者需要具体怎么做，中介者根据封装在自身内部的协调逻辑，对同事的请求进行进一步处理，将同事成员之间的关系行为进行分离和封装。该协调作用属于中介者在行为上的支持。

**(二)中介者模式 UML图**
![Mediator Pattern](http://upload-images.jianshu.io/upload_images/5261067-f328aca70dfdb772.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**(三)简单实例**

中介者模式的思想在现实生活中也很常见，比如说交换机。没有交换机存在的时代，每个电话之间都需要电话线连接才能进行通话。如果一个台电话要和其它100台电话通话，那么它就必须要有100条电话线与其它100个电话连接。

后来为了解决这种麻烦，交换机出现了。每个电话只需连入交换机，通话时。只需构建一条电话-交换机-电话的链路，就可以进行通话。所以现在我们的电话理论上可以同世界上任何一台电话通话，但是只需一条电话线。当然现在用电话的人少了，但是手机呀，计算机网络的实现也是在传统通信网的设计上进行演进的。

其实交换机对应的就是中介者模式的中介者，而电话机就是中介者中的同事。下面，就让我们用代码来实现这个思想。

```
<?php
//抽象同事类 --------电话机
abstract class Colleague{
    protected $mediator;    //用于存放中介者
    abstract public function sendMsg($num,$msg);
    abstract public function receiveMsg($msg);
    //设置中介者
    final public function setMediator(Mediator $mediator){
      $this->mediator = $mediator;
    }
}
//具体同事类 ---------座机
class Phone extends Colleague
{
    public function sendMsg($num,$msg)
    {
      echo __class__.'--发送声音：'.$msg.PHP_EOL;
      $this->mediator->opreation($num,$msg);
    }

    public function receiveMsg($msg)
    {
      echo __class__.'--接收声音：'.$msg.PHP_EOL;
    }
}
//具体同事类----------手机
class Telephone extends Colleague
{
    public function sendMsg($num,$msg)
    {
        echo __class__.'--发送声音：'.$msg.PHP_EOL;
        $this->mediator->opreation($num,$msg);
    }
    //手机接收信息前 会智能响铃
    public function receiveMsg($msg)
    {   
        echo '响铃-------'.PHP_EOL;
        echo __class__.'--接收声音：'.$msg.PHP_EOL;
    }
}
//抽象中介者类
abstract class Mediator{
  abstract public function opreation($id,$message);
  abstract public function register($id,Colleague $colleague);
}
//具体中介者类------交换机
class switches extends Mediator
{
    protected  $colleagues = array();
    //交换机业务处理
    public function opreation($num,$message)
    {
        if (!array_key_exists($num,$this->colleagues)) {
            echo __class__.'--交换机内没有此号码信息，无法通话'.PHP_EOL;
        }else{
            $this->colleagues[$num]->receiveMsg($message);
        }
    }
    //注册号码
    public function register($num,Colleague $colleague)
    {
      if (!in_array($colleague, $this->colleagues)) {
          $this->colleagues[$num] = $colleague;
      }
      $colleague->setMediator($this);
    }
}
//实例化固话
$phone = new Phone;
//实例化手机
$telephone = new Telephone;
//实例化交换机
$switches = new Switches;
//注册号码  ---放号
$switches->register(6686668,$phone);
$switches->register(18813290000,$telephone);
//通话
$phone->sendMsg(18813290000,'hello world');
$telephone->sendMsg(6686668,'请说普通话');
$telephone->sendMsg(6686660,'你好');
```
