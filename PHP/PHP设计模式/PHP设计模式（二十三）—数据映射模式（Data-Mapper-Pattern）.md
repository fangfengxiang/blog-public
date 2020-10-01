**数据映射模式（Data Mapper Pattern ）**：描述如何创建提供透明访问任何数据源的对象。数据映射模式，也叫数据访问对象模式，或数据对象映射模式。

**(一)为什么需要数据映射模式**

> 数据映射模式的目的是让持久化数据存储层、驻于内存的数据表现层、以及数据映射本身三者相互独立、互不依赖。这个数据访问层由一个或多个映射器（或者数据访问对象）组成，用于实现数据传输。通用的数据访问层可以处理不同的实体类型，而专用的则处理一个或几个。


**(二)数据映射模式UML图**

![Data Mapper Pattern ](http://upload-images.jianshu.io/upload_images/5261067-97e8bf21415b5d2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**(三)简单实例**

通过数据对象映射模式，我们可以实现一个对象对应一条数据库记录，对象的属性对应记录的字段。但对象的属性改变时，自动更新数据库记录。

例如我们有一个用户类与数据库的用户表对应

```
<?php
//数据模式映射类
class User
{
    protected $id;
    protected $data;
    protected $db;
    protected $change = false;

    public function __construct($id)
    {   
        $this->id = $id;
        //实例化数据库对象，这里使用了工厂方法
        $this->db = Factory::getDatabase();
        //从数据库查询数据，存放到data属性中
        $this->data  = $this->db->query("select * from user where id = $id limit 1");

    }

    public function __get($key)
    {
        if (isset($this->data[$key]))
        {
            return $this->data[$key];
        }
    }

    public function __set($key, $value)
    {
        $this->data[$key] = $value;
        $this->change = true;
    }
    //析构方法
    public function __destruct()
    {
        //如果对象属性改变过，则change属性为true 则调更新方法更新数据库
       $this->change && $this->update();
    }
    //更新记录方法
    public function update(){
         foreach ($this->data as $k => $v)
            {
                $fields[] = "$k = '{$v}'";
            }
            $this->db->query("update user set " . implode(', ', $fields) . "where
            id = {$this->id} limit 1");
    }
}
//实例化对象
$user = new User(1);
//改变名字
$user->name = 'admin';
```


如果我们要实现实时更新，也可以不要change属性，直接在__set方法中调用update方法，不用等到对象销毁前再统一更新。当然实时更新时更新方法可以精简地不需要foreach，只写更新一个字段指令就OK，但是这样也带来频繁操作数据库的问题。
