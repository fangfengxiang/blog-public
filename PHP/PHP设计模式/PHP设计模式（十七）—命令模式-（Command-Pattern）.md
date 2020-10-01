**命令模式 （Command Pattern）**： 将一个请求封装为一个对象，从而使我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作(Action)模式或事务(Transaction)模式。

**(一)为什么需要命令模式**

1， 使用命令模式，能够让请求发送者与请求接收者消除彼此之间的耦合，让对象之间的调用关系更加灵活。

2，使用命令模式可以比较容易地设计一个命令队列和宏命令（组合命令），而且新的命令可以很容易地加入系统

**(二)命令模式 UML图**

![Command Pattern](http://upload-images.jianshu.io/upload_images/5261067-84f4f1ad0aa18c08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**(三)简单实例**

有关命令模式的一个经典的例子，就是电视机与遥控器。如果按照命令模式的UML图那么，就有这样的对应关系：电视机是请求的接收者，遥控器是请求的发送者。遥控器上有一些按钮，不同的按钮对应电视机的不同操作。这些按钮就是对应的具体命令类。抽象命令角色由一个命令接口来扮演，有三个具体的命令类实现了抽象命令接口，这三个具体命令类分别代表三种操作：打开电视机、关闭电视机和切换频道。

```
<?php
//抽象命令角色
abstract class Command{
  protected $receiver;
  function __construct(TV $receiver)
  {
      $this->receiver = $receiver;
  }
  abstract public function execute();
}
//具体命令角色 开机命令
class CommandOn extends Command
{
  public function execute()
  {
      $this->receiver->action();
  }
}
//具体命令角色 关机机命令
class CommandOff extends Command
{
  public function execute()
  {
      $this->receiver->action();
  }
}
//命令发送者   --遥控器
class Invoker
{
  protected $command;
  public function setCommand(Command $command)
  {
      $this->command = $command;
  }

  public function send()
  {
      $this->command->execute();
  }
}
//命令接收者 Receiver =》 TV
class TV
{
  public function action()
  {
      echo "接收到命令，执行成功".PHP_EOL;
  }
}

//实例化命令接收者 -------买一个电视机
$receiver = new TV();
//实例化命令发送者-------配一个遥控器
$invoker  = new Invoker();
//实例化具体命令角色 -------设置遥控器按键匹配电视机
$commandOn = new CommandOn($receiver);
$commandOff = new CommandOff($receiver);
//设置命令  ----------按下开机按钮
$invoker->setCommand($commandOn);
//发送命令
$invoker->send();
//设置命令  -----------按下关机按钮
$invoker->setCommand($commandOff);
//发送命令
$invoker->send();
```

在实际使用中，命令模式的receiver经常是一个抽象类，就是对于不同的命令，它都有对应的具体命令接收者。命令模式的本质是对命令进行封装，将发出命令的责任和执行命令的责任分割开。
