PHP基础知识
===



---

常量与数据类型
---

### 三大数据类型
- 标量
	- 整型
	- 浮点
	- 字符串
	- 布尔
- 复合
	- 数组
	- 对象
- 特殊
 	- null
	- 资源

### 标量类型
- 浮点类型不能用于直接比较大小 (IEEI754标准)
- 布尔false 七种情况 
	- 整型 0 
	- 浮点型 0.0
	- 字符串 ' ' 和 '0'
	- 布尔型 false
	- 数组 [] 
	- null


### 字符串3种定义方式
- 单引号
- 双引号
- heredoc 和 newdoc

区别：
- 单引号不能解析变量
- 单引号不能解析转义字符，只能解析单引号和反斜线本身
- 双引号可以解析变量，变量可以使用特殊字符和{}包含
- 双引号可以解析所有的转义字符
- 单引号的效率要明显高于双引号

heredoc 和 newdoc
heredoc 和 newdoc 通俗来说就是定界符,主要用来处理长字符串或大文本
heredoc 相当于双引号，newdoc相当于当引号
```
<?php
$str = <<< EOL

EOL

$str = <<< 'EOL'

EOL
```
### PHP数据类型



### 数组类型
#### 超全局数组
- $GLOBALS
- $_GET
- $_POST
- $_REQUEST
- $_SESSION
- $_COOKLE
- $_SERVER
- $_FILES
- $_ENV

#### $_SERVER
- $_SERVER['SERVER_ADDR'] 服务端IP地址
- $_SERVER['SERVER_NAME'] 服务器名称
- $_SERVER['REQUEST_TIME'] 请求时间
- $_SERVER['QUERY_STRING'] URL问号携带的字符串
- $_SERVER['REMOTE_ADDR'] 客户端IP 

### 常量的两种定义方式
- const
- define()

区别：
const 是语言结构，define() 是函数
const 比define更快
const 可用来定义类常量，define 不行

- 预定义常量

---

运算符
---

### 运算符优先级
http://php.net/manual/zh/language.operators.precedence.php

### == 和 === 的区别
- == 表示相等，只要值相等就为真
- === 表示全等，只有值和类型同时相等，才为真

### 递增递减符
- 递增递减不影响布尔类型
- 递减null 值无效果
- 递增null 值为 1
- 递增和递减在前就先运算 后返回，在后就先返回后运算
- i++ 要不 ++ i 要慢 

### 逻辑运算符
- 短路作用
- || && 比 and or 的运算级高
```
$a = false || true
$b = false or true
```

### 错误抑制运算符
> foo() 和 @foo() 的区别？
> @是错误抑止运算符，将其放置在一个表达式之前，该表达式可能产生的任何错误都会被忽略


---

流程控制
---

### 三种循环数组的方式
- 请列出PHP 三中数组循环操作的语法，并注明各种循环的区别
```
$arr = [];

// for 循环  只能遍历索引数组
for($i=)

// foreach 循环  既能遍历索引数组和关联数组
// foreach 循环 一开始的时候 就会reset数组指针
 
// while + list() + each() 既能遍历索引数组和关联数组
// while + list() + each() 组合不会reset()数组指针
// 所以 while 有可能只遍历了数组的第一部分，建议遍历前先重置一下数组指针
while(list($k,$v)=each($arr)){
}
```
### switch注意点
- switch 的case 只能是 整型 浮点型 和字符串类型
- continue 作用于 switch 相当于 breake
- switch 会生成跳转表，效率要不if else 高一些
- continue 和 breake 后可以加 数字 表示跳出几层 默认 不写就是1 （continue 2 ）

### 优化多个 if else 语句
- 表达式可能性高的放前面
- 考虑使用switch

---

函数
---

### 变量作用域

#### 全局变量
```
<?php
$a = 1;
function foo()
{
	$GLOBALS['a'] = 2;
}
echo $a;
```
#### 静态变量
写出一下程序的输出结果
```
<?php
function get_count()
{
	static $count = 0;
	return $count++;
}
$count = 5;
echo $count;
$count++;
ehco get_count();
echo get_count();
```
- 仅初始化一次
- 初始化时需要赋值(也就是static $i = 后面的数必须是常量 否则会报语法错误)
- 每次执行，该值都会保存
- static 是局部变量
- 常用来记录调用次数，结束递归

### 返回引用
- 省略return ，返回值为null
- 存在static时，返回引用。
```
<?php
function &foo()
{
	static $i = 'foo';
	return $i;
}

$a = foo();
echo $a;
echo PHP_EOL;

$a = &foo();
echo $a;
echo PHP_EOL;

$a = 'hello world';
echo foo();
```
或参数为引用变量也可以
```
<?php
function &foo(&$i)
{
	$i.= ' foo';
	return $i;
}

$first = 'hello';
$a = &foo($first);
echo $a;
echo PHP_EOL;
echo $first;
echo PHP_EOL;

$a = 'bye world';
echo $first;
```
上面这两种很少使用，返回引用类型的一个稍微实际点的用处，可能就是用来重置static 变量的值吧。

### 自带函数

#### 时间函数
- date()
- strtotime()
- mktime() 取得一个日期的 Unix 时间戳
- time()
- microtime()
- date_default_timezone_set() 设定用于一个脚本中所有日期时间函数的默认时区(有对应的get 方法)

#### 打印处理函数
- print(string)
print实际上不是函数（而是语言结构），所以可以不用圆括号包围参数列表。
和 echo 最主要的区别： print 仅支持一个参数，并总是返回1
- printf()
printf => print+format => 格式化输出
返回输出字符串的长度
- sprintf()
与printf一样格式化，区别在于printf是直接输出，sprintf()不输出 返回这个格式化后的数组
- print_r()
打印关于变量的易于理解的信息
- echo
- var_dump
打印变量的相关信息
- var_export()
输出或返回一个变量的字符串表示，它和 var_dump() 类似，不同的是其返回的表示是合法的 PHP 代码。

#### 序列化函数和反序列化函数
- serialize 产生一个可存储的值的表示
- unserialize 从已存储的表示中创建 PHP 的值

#### 字符串处理函数
- trim
- explode  字符串切成数组
- join  数组切成字符串  =>  implode 函数的别名
- strrev  反转字符串
- strstr  查找字符串的首次出现
- number_format 以千位分隔符方式格式化一个数字，传入float类型 返回string 类型

#### 数组函数
- array_keys
- array_values
- array_diff 返回数组中不同的键值对
- array_intersect 计算数组的交集

### 正则表达式
写出 11 位手机号码的正则表达式
> 正则表达式的作用：分割，查找，匹配，替换字符串

- 分隔符
/ , #, ~  业界常用 /
- 通用原子
\d => (0-9) , \D => (\d 取反) , \s => (空白符) , \S , \w , \W
- 元字符
. * ? ^ $ + {n} {n,} {n,m} [] () [^] | [-]
- 模式修正符
i m e(php7取消) s U x A D u
- 后向引用
- 贪婪模式
- 常用函数
preg_match()  preg_match_all() preg_replace() preg_split()
- 中文匹配
utf8 汉字编码范围0x4e00-0x9fa5 
gb2312编码范围 0xb0-0xf7 0xa1-0xfe
utf8要使用U修正符使模式字符串被当成utf8 
gb2312要使用chr将ASCII码转为字符
- 写正则的技巧
先写出要匹配的字符串
自左向右的顺序使用正则表达式的原子和元字符进行拼接
最终加入模式修正符

---

文件与目录处理
---

### 文件读取基本函数
- fopen()模式
r 只读
r+ 读写
w 只写
w+ 读写
a 追加
a+ 追加+读
b 打开二进制文件
- file_get_contents
- file_put_contents


### 读取远程文件
php.ini 需开启 allow_url_fopen,http协议只能是只读模式，ftp可以是只读和只写模式。

### 目录相关函数
名称相关 ： basename dirname pathinfo
目录读取 ： opendir  readdir closedir rewinddir
删除目录 ： rmdir  (该目录下必须为空才能删除)
创建目录  ： mkdir 
磁盘大小 disk_free_space disk_total_space
文件类型 filetype
重名 rename 同时可以移动路径
文件锁 flock

### 面试题 
通过PHP函数对目录进行遍历
```
function loopDir($dir)
{
	$handle = opendir($dir);
	
	while(false!==($file = readdir($handle))){
		if($file != '.' && $file != '..'){
			echo $file.PHP_EOL;
			if(filetype($dir.'/'.$file) == 'dir'){
				loopDir($dir.'/'.$file);
			}
		}
	}
}
```
