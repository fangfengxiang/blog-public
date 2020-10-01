**责任链模式（ Chain of Responsibility Pattern）**： 为请求创建了一个接收者对象的链，并沿这条链传递该请求，直到有对象处理它为止。这种模式能够给予请求的类型，对请求的发送者和接收者进行解耦。

**(一)为什么需要责任链模式**

1，将请求的发送者和请求的处理者解耦了。责任链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可，无须关心请求的处理细节和请求的传递。

2， 发出这个请求的客户端并不知道链上的哪一个对象最终处理这个请求，这使得系统可以在不影响客户端的情况下动态地重新组织和分配责任

**(二)责任链模式 UML图**

![Chain of Responsibility Pattern](http://upload-images.jianshu.io/upload_images/5261067-c6820b131115da9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**(三)简单实例**

如果我们现在做一个员工系统，如果说公司有规定说员工有请示，要根据请示的级别由不同人批示。比如请假由组长批示，休假由主管批示，辞职由总经理批示。好了，如果我们使用代码如何实现呢？员工类，组长类，主管类，总经理类。那么员工对象的请示直接交给总经理吗，显然是不好。如果我们按照现实世界的逻辑来实现，那应该是怎样的呢？（我很赞同一句话：`任何代码的逻辑都在现实世界能找到与之对应的场景，如果没有，就是你的逻辑有问题`）

```
<?php
//抽象处理者
abstract class Handler{
    private $next_handler ;//存放下一处理对象
    private $lever = 0;             //处理级别默认0
    abstract protected function response();  //处理者回应
    //设置处理级别
    public function setHandlerLevel($lever){
        $this->lever = $lever ;
    }
    //设置下一个处理者是谁
    public function setNext(Handler $handler){
        $this->next_handler = $handler;
        $this->next_handler->setHandlerLevel($this->lever+1);
    }
    //责任链实现主要方法，判断责任是否属于该对象，不属于则转发给下一级。使用final不允许重写
    final public function handlerMessage(Request $request){
        if($this->lever == $request->getLever()){
            $this->response();
        }else{
            if($this->next_handler != null){
                $this->next_handler->handlerMessage($request);
            }else{
                echo '洗洗睡吧，没人理你'.PHP_EOL;
            }
        }
    }
}
//具体处理者
// headman 组长 
class HeadMan extends Handler{
    protected function response(){
        echo '组长回复你：同意你的请求'.PHP_EOL;
    }
}
//主管director
class Director extends Handler{
    protected function response(){
        echo '主管回复你：知道了，退下'.PHP_EOL;
    }
}
//经理manager
class Manager extends Handler{
    protected function response(){
        echo '经理回复你：容朕思虑，再议'.PHP_EOL;
    }
}
//请求类
class Request{
    protected $level = array('请假'=>0,'休假'=>1,'辞职'=>2);//测试方便，默认设置好请示级别对应关系
    protected $request;
    public function __construct($request){
        $this->request = $request;
    }
    public function getLever(){
        //获取请求对应的级别，如果该请求没有对应级别 则返回-1
        return array_key_exists($this->request,$this->level)?$this->level[$this->request]:-1;
    }
}

//实例化处理者
$headman = new HeadMan;
$director = new Director;
$manager = new Manager;
//责任链组装
$headman->setNext($director);
$director->setNext($manager);
//传递请求
$headman->handlerMessage(new Request('请假'));
$headman->handlerMessage(new Request('休假'));
$headman->handlerMessage(new Request('辞职'));
$headman->handlerMessage(new Request('加薪')); 
```

具体的处理者即对应UML图中的ConcreteHandler,请求类对应UML图的client。使用责任链，可以使对应的处理者有对应的单一的response方法。请求只需交给最低级的处理者，由属性$lever判断，一层层传递到与请求级别相同的处理者中做出对应的回应。请求无需知道，要交给对应的那个处理者。
当然，当责任链过长时也会引起性能问题。对此我们应避免使用过长的责任链。
