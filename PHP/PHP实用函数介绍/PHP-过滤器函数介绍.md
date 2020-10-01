[PHP过滤器函数](https://secure.php.net/manual/zh/ref.filter.php)：`此扩展程序通过验证或清理数据来过滤数据。当数据源包含未知（或外部）数据（如用户提供的输入）时，这非常有用`

- 该套函数自** PHP5.2 **以后默认安装，受`php.ini`配置影响。[具体配置](https://secure.php.net/manual/zh/filter.configuration.php)
- 该套函数主要用来验证和过滤数据。
[具体过滤类型](https://secure.php.net/manual/zh/filter.filters.php)
[预定义常量](https://secure.php.net/manual/zh/filter.constants.php)

![过滤器函数列表.png](http://upload-images.jianshu.io/upload_images/5261067-c5dfc7efe794d02c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

filter_list
---

[filter_list](https://secure.php.net/manual/zh/function.filter-list.php) : 返回所支持的过滤器列表
![filter_list.png](http://upload-images.jianshu.io/upload_images/5261067-77308eb6a9219c03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

根据文档的介绍，我们可以使用该函数查看我们当前的PHP版本支持哪些过滤器。
![](http://upload-images.jianshu.io/upload_images/5261067-eec47324768afb94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如图：该数组给出了该版本所支持的过滤器名称，你可以通过该名称调用filter_id()函数获取对应的过滤器Id，当然这些过滤器Id实际上已经被内置为常量，所以实际使用并不需要知道过滤器的具体Id值。

filter_id
---

[filter_id](https://secure.php.net/manual/zh/function.filter-id.php) : 返回与某个特定名称的过滤器相关联的id
![图片.png](http://upload-images.jianshu.io/upload_images/5261067-54bef03760aa86b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
$filters = filter_list();
foreach($filters as $filter_name):
   echo $filter_name.':'.filter_id($filter_name).PHP_EOL;
endforeach;
```
![](http://upload-images.jianshu.io/upload_images/5261067-982cf93a57bd05b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如图 ： 我们看到可以通过过滤器名称获取对应的过滤器Id，例如int过滤器的id是257，但是实际上这种方法并不常用，因为对应的过滤器Id已经预定义为对应的常量了，例如int过滤器的预定义常量为 `FILTER_VALIDATE_INT`

![FILTER_VALIDATE_INT](http://upload-images.jianshu.io/upload_images/5261067-a8718170e049f02e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

filter_var
---
[filter_var](https://secure.php.net/manual/zh/function.filter-var.php) : 使用特定的过滤器过滤一个变量

![filter_var](http://upload-images.jianshu.io/upload_images/5261067-40efadb0131f539a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

示例：
```
$a = 1;
$b = '1';
$c = 'a';
$d = [];
var_dump(filter_var($a,FILTER_VALIDATE_INT));
var_dump(filter_var($b,FILTER_VALIDATE_INT));
var_dump(filter_var($c,FILTER_VALIDATE_INT));
var_dump(filter_var($d,FILTER_VALIDATE_INT));
```

![filter_val](http://upload-images.jianshu.io/upload_images/5261067-d319df32ac5518e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

filter_val_array
---

[filter_val_array](https://secure.php.net/manual/zh/function.filter-var-array.php) : 获取多个变量并且过滤它们

![filter_val_array](http://upload-images.jianshu.io/upload_images/5261067-dbafc861b9d69e2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

示例：
```
$arr =[1,'abc',true,[]];
var_dump(filter_var_array($arr,FILTER_VALIDATE_INT));
```

![](http://upload-images.jianshu.io/upload_images/5261067-558f6b8ad8577222.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


