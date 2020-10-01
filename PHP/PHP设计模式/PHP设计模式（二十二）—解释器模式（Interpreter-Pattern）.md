**解释器模式（Interpreter Pattern）**： 提供了评估语言的语法或表达式的方式，它属于行为型模式。这种模式实现了一个表达式接口，该接口解释一个特定的上下文。这种模式被用在 SQL 解析、符号处理引擎等
**(一)为什么需要解释器模式**

> 可以将一个需要解释执行的语言中的句子表示为一个抽象语法树

**(二)解释器模式UML图**


![Interpreter Pattern](http://upload-images.jianshu.io/upload_images/5261067-2a6ed103a378dc60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**(三)简单实例**

解释器模式是开发中最少使用的，因为我们亲自编写语法解析的时候总是非常非常少。而且我能想到的解释器模式例子，几乎都是代码繁多到我自己怕。所以这里我只给出解释器模式的UML图通用代码，感兴趣的话可以自己去实现

```
<?php
//抽象表达式 
abstract class Expression{
    //任何表达式子类都应该有一种解析任务
    abstract public function interpreter($context);
}
//抽象表达式是生成语法集合（语法树）的关键，每个语法集合完成指定语法解析任务
//抽象表达式通过递归调用的方法，最终由最小语法单元进行解析完成

//终结符表达式    通常指运算变量
class TerminalExpression extends Expression{
    //终结符表达式通常只有一个
    public function interpreter($context){
        return null; //视具体业务实现
    }
}
//非终结符表达式   通常指运算的符号
class NonterminalExpression extends Expression{
    public function interpreter($context){
        return null;
    }
}
```
