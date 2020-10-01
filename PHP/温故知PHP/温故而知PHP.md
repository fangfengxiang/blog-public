#### SPAI专题

###### [命令行](http://php.net/manual/en/features.commandline.php)
php --rf  => 列出函数 php --rc
cli 命令增加扩展
- [readline](http://php.net/manual/en/book.readline.php)
- [Newt](http://php.net/manual/en/book.newt.php) 
- [Ncurses](http://php.net/manual/en/book.ncurses.php) 

#### cli交互开发 => stdin  stdout 流
cli_set_process_title

declare 实现异步 (http://php.net/manual/zh/function.unregister-tick-function.php)
--- 
#### 序列化专题
- [json](http://php.net/manual/zh/book.json.php)
- 踩坑 ： json 只能序列化文本类型数据，MIME类型的 使用searize序列化，或先使用base64编码做转化。而且PHP的json只支持utf8编码。
- 扩展知识点 ： 实现 **JsonSerializable** 接口的类可以 在 [json_encode()](http://php.net/manual/zh/function.json-encode.php) 时定制他们的 JSON 表示法

#### stream 
流
- 踩坑 ： file_get_conntent 发post请求 要 把参数 http_build_query 。否则出现post没有接收到的情况。
https://blog.csdn.net/Ni9htMar3/article/details/69812306?locationNum=2&fps=1
https://blog.csdn.net/qq_33904831/article/details/78814567


自定义协议：
http://php.net/manual/zh/function.stream-wrapper-register.php
支持的协议和封装的协议
http://php.net/manual/zh/wrappers.php
通知回调
http://php.net/manual/zh/function.stream-notification-callback.php
get 请求url上的参数 无法通过php://input 获取
