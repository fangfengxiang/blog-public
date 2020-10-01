**适配器模式（Adapter Pattern）**：将某个对象的接口适配为另一个对象所期望的接口。属于结构型设计模式。

**（一）为什么需要适配器模式**

1，某个操作数据库的有两套不同的数据库操作方法，我们通过适配器统一成一个接口。例如，我们待会把mysql和mysqli统一成一个接口。

2，我们有多套数据库对应了多种数据库操作，例如MySQL，SqlServer,Oralce,Redis都有对应的操作函数，或操作类。PDO把这些都统一成一个接口。

3，系统的增加一些新功能，创建了一个新的接口，但是老的接口并不想废弃。可以使用适配器模式，对用户隐藏这两个接口，提供用户所希望的接口。

**（二）适配器UML图**

![Adapter Pattern](http://upload-images.jianshu.io/upload_images/5261067-e5df0e531a0b70cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**(三)设计实例**

把MySQL和mysqli统一成一个接口，用户可以调用同样的方法使用MySQL和mysqli操作数据库。

```
<?php
//MySQL待操作适配类
class MySQLAdaptee implements Target
{
    protected $conn;    //用于存放数据库连接句柄
    //实现连接方法
    public function connect($host, $user, $passwd, $dbname)
    {
        $conn = mysql_connect($host, $user, $passwd);
        mysql_select_db($dbname, $conn);
        $this->conn = $conn;
    }
    //查询方法
    public function query($sql)
    {
        $res = mysql_query($sql, $this->conn);
        return $res;
    }
    //关闭方法
    public function close()
    {
        mysql_close($this->conn);
    }
}
//MySQLi操作待适配类
class MySQLiAdaptee 
{
    protected $conn;
    public function connect($host, $user, $passwd, $dbname)
    {
        $conn = mysqli_connect($host, $user, $passwd, $dbname);
        $this->conn = $conn;
    }
    public function query($sql)
    {
        return mysqli_query($this->conn, $sql);
    }
    public function close()
    {
        mysqli_close($this->conn);
    }
}
//用户所期待的接口
Interface Target{
    public function connect($host, $user, $passwd, $dbname);
    public function query($sql);
    public function close();
}
//用户期待适配类
Class DataBase implements Target {
    protected $db ;     //存放MySQLiAdapter对象或MySQLAdapter对象
    public function  __construct($type){
        $type = $type."Adapter" ;
        $this->db = new $type ;
    }
    public function connect($host, $user, $passwd, $dbname){
        $this->db->connect($host, $user, $passwd, $dbname);
    }
    public function query($sql){
        return $this->db->query($sql);
    }
    public function close(){
        $this->db->close();
    }
}
//用户调用同一个接口，使用MySQL和mysqli这两套不同示例。
$db1 = new DataBase('MySQL');
$db1->connect('127.0.0.1','root','1234','myDB');die;
$db1->query('select * from test');
$db1->close();

$db2 = new DataBase('MySQLi');
$db2->connect('127.0.0.1','root','1234','myDB');
$db2->query('select * from test');
$db2->close();
```

上面的代码只是一个示例，如果你运行以上的代码报了mysql函数不存在或是被废弃的错误。这是正常的，因为MySQL这套函数在PHP5.5以上的版本已经被废弃了。感兴趣的还可以去了解一下PDO的实现。
通过上面的代码，我们可以看到，使用适配器可以把不同的操作接口封装起来，对外显示成用户所期望的接口。
> 这就好比你家墙上有一个电源三相插孔，但是插孔的孔距之间太小。你的电器三相插头插脚距太大的插不进去，或许你还有个两相的插头，或许你还有条USB线和type-C线，这些都没法插到三相接口里。于是你买了个插脚适合插到你墙上的排插，然后这个排插是这些年新出的，USB也能插。于是你把你的三相插头，两相插头，USB线，type-c线都插到排插上。实际上就是间接地连在了你墙壁上的三相插孔上。

没错，适配器要做的就是这么回事。

> 有些书也把适配器模式分为：类的适配器模式，对象的适配器模式，接口的适配器模式



后续更新，欢迎大家评论指正
欢迎大家关注我的微信公众号 火风鼎

![火风鼎](http://upload-images.jianshu.io/upload_images/5261067-e41dd2eb70ab0fe7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
