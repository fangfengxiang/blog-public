注册树模式（Registry Pattern ）：注册树模式为应用中经常使用的对象创建一个中央存储器来存放这些对象 —— 通常通过一个只包含静态方法的抽象类来实现（或者通过单例模式）。也叫做注册器模式

**(一)为什么需要注册树模式**

> 解决常用对象的存放问题，实现类似于全局变量的功能。

**(二)注册树模式UML图**

暂无，跪求提供

**(三)简单实例**

```
<?php
//User类用于测试
class User{}

//注册树类
class Registry
{
    protected static $objects;  //用于存放实例
    //存入实例方法
    static public function set($key, $object)
    {
        self::$objects[$key] = $object;
    }
    //获取实例方法
    static public function get($key)
    {
        if (!isset(self::$objects[$key]))
        {
            return false;
        }
        return self::$objects[$key];
    }
    //删除实例方法
    static public function _unset($key)
    {
        unset(self::$objects[$key]);
    }
}


$user = new User;
//存入实例
Registry::set('User',$user);
//查看实例
var_dump(Registry::get('User'));
//删除实例
Registry::_unset('User');
//再次查看实例
var_dump(Registry::get('User'));
```


注册树经常与单例模式一起使用，先查看注册树上是否有该实例，有就直接使用，没有就生成一个实例，并挂到树上。有些时候我们还可以这样做，让get方法如果get不到实例的时候就自动new一个存放起来，这样我们使用时就不用管有没有存放过这个实例，反正没有的话get方法也会帮我们存放。
   

```
  //获取实例方法
    static public function get($key)
    {
        if (!isset(self::$objects[$key]))
        {
            self::$objects[$key] = new $key;
        }
        return self::$objects[$key];
    }
```

当然使用这种方式的话，查看实例是否存在，就不能使用get方法了。因为调用get方法以后，不存在也会生成一个实例。
