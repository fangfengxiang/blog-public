**观察者模式 （observer Pattern）**： 定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。也叫发布-订阅模式

**(一)为什么需要观察者模式**

1，一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作

2，完美的将观察者和被观察的对象分离开,使得每个类将重点放在某一个功能上，一个对象只做一件事情。

3，观察者模式在模块之间划定了清晰的界限，提高了应用程序的可维护性和重用性。

**(二)观察者模式 UML图**
![observer Pattern](http://upload-images.jianshu.io/upload_images/5261067-1f6fabd98763d154.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**(三)简单实例**

观察者模式也叫发布订阅模式，如果说我们现在在做一个系统。我们让所有客户端订阅我们的服务端，那么当我们的服务端有更新信息的时候，就通知客户端去更新。这里的服务端就是被观察者，客户端就是观察者。

```
<?php
//抽象被观察者
abstract class Subject{
    //定义一个观察者数组
    private $observers = array();
    //增加观察者方法
    public function addObserver(Observer $observer){
        $this->observers[] = $observer;
        echo "添加观察者成功".PHP_EOL;
    }
    //删除观察者方法
    public function delObserver(Observer $observer){
        $key = array_search($observer,$this->observers); //判断是否有该观察者存在
        if($observer===$this->observers[$key]) { //值虽然相同 但有可能不是同一个对象 ，所以使用全等判断
            unset($this->observers[$key]);
            echo '删除观察者成功'.PHP_EOL;
        } else{
            echo '观察者不存在，无需删除'.PHP_EOL;
        }
    }
    //通知所有观察者
    public function notifyObservers(){
        foreach($this->observers as $observer){
            $observer->update();
        }
    }
}
//具体被观察者 服务端
class Server extends Subject{
    //具体被观察者业务 发布一条信息，并通知所有客户端
    public function publish(){
        echo '今天天气很好，我发布了更新包'.PHP_EOL;
        $this->notifyObservers();
    }
}
//抽象观察者接口
Interface Observer{
    public function update();
}
//具体观察者类
//微信端
class Wechat implements Observer{
    public function update(){
        echo '通知已接收，微信更新完毕'.PHP_EOL;
    }
}
//web端
class Web implements Observer{
    public function update(){
        echo '通知已接收，web端系统更新中'.PHP_EOL;
    }
}
//app端
class App implements Observer{
    public function update(){
        echo '通知已接收，APP端稍后更新'.PHP_EOL;
    }
}

//实例化被观察者
$server = new Server ;
//实例化观察者
$wechat = new Wechat ;
$web = new Web ;
$app = new App;
//添加被观察者
$server->addObserver($wechat);
$server->addObserver($web);
$server->addObserver($app);
//被观察者 发布信息
$server->publish();

//删除观察者
$server->delObserver($wechat);
//再次发布信息
$server->publish();

//尝试删除一个未添加成观察者的对象
$server->delObserver(new Web);
//再次发布信息
$server->publish();
```


观察者模式的一个关键词就是触发，被观察者的动作触发观察者的做出对应的响应。观察者模式的另一个常用领域在于插件系统。

在PHP中观察者的另一种实现方式，是通过实现SplSubject接口和SplObserver,这种实现方法涉及到spl（standard php library）的内容，我将会在SPL相关文章中进行介绍。欢迎大家到时候阅读指正
