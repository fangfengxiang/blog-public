**模板模式 （Template Pattern）**： 定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板模式使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

**(一)为什么需要模板模式**

1，一次性实现一个算法的不变的部分，并将可变的行为留给子类来实现。

  
2， 多个子类有相同的方法，逻辑基本相同时。可将相同的逻辑代码提取到父类

3，重构时，将相同代码抽取到父类，然后通过钩子函数约束其行为

**(二)模板模式 UML图**

![Template Pattern](http://upload-images.jianshu.io/upload_images/5261067-b1447c9f169d99f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**(三)简单实例**

这里举《PHP设计模式》的一个例子：一个银行可以有许多不同类型的银行账户，但是所有账户的处理方式基本相同。假设我们现在有两类账户，一类是普通账户，一类是信用卡账户。现在进行支付，信用卡允许透支，普通账户不允许透支，即账户金额不允许小于零

```
<?php 
//抽象模板类
abstract class Template{
	protected $balance = 100;       //账户余额,为测试方便，直接赋初值100
	//结算方法
	abstract protected function adjust($num);
	//支付信息显示 
	abstract protected function display($num);
	final public function apply($num){
		$this->adjust($num);
		$this->display($num);
	}
}

//普通账户
class Account extends Template{
	protected $falg;  //用于判断支付是否成功
	protected function adjust($num){
		if($this->balance > $num){//只有余额大于所需支付金额才允许支付
			$this->balance-=$num;
			$this->falg = true;
		}else{
			$this->falg = false;
		}
	}
	protected function display($num){
		if($this->falg){
			echo '支付成功,所剩余额为'.$this->balance.PHP_EOL;
		}else{
			echo '余额不足，支付失败,所剩余额为'.$this->balance.PHP_EOL;
		}
	}
}

//信用卡用户
class Credit extends Template{
	protected function adjust($num){
		$this->balance-=$num;
	}
	protected function display($num){
		echo '感谢您使用信用支付，所剩余额为'.$this->balance.PHP_EOL;
	}
}

//普通账户使用
$account = new Account;
//普通账户使用
$account -> apply(80);
//普通账户透支
$account -> apply(30);

//信用卡账户使用
$credit = new Credit;
$credit -> apply(200);
```


模板模式的好处在于行为由父类控制，而具体的实现由子类实现。这就可以把一个操作延迟绑定到子类上。还有另一种应用是把复杂的核心代码设计为模板方法，周边的相关细节则由子类实现。
