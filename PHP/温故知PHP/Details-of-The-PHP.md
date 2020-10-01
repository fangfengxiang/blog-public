### 预定义变量

#### $_POST
- [http://php.net/manual/zh/reserved.variables.post.php](http://php.net/manual/zh/reserved.variables.post.php)
- php 的 $_POST 变量只能接收部分POST请求报文的数据，并非全部。
- 当 POST 请求的` Content-Type 是 application/x-www-form-urlencoded 或 multipart/form-data` 时，会将变量以关联数组形式传入当前脚本。
-  当不是$_POST 能解析的报文时，PHP将原始的POST数据填充到
 `GLOBALS[ HTTP_RAW_POST_DATA] 即HTTP_RAW_POST_DATA ` 中，所以PHP5.6以下很多XML报文都使用了这个变量。但是PHP7已经移除这个变量，应使用 `file_get_contents("php://input") ` 来替代
- POST变量填充的配置，详见[http://php.net/manual/en/ini.core.php#ini.always-populate-raw-post-data](http://php.net/manual/en/ini.core.php#ini.always-populate-raw-post-data)

#### 面向对象基础
- [PHP empty 函数判断结果为空，但实际值却为非空](https://learnku.com/articles/12530/the-shock-php-empty-function-is-empty-but-the-actual-value-is-nonempty)


####  curl 调试篇
curl  设置代理 。curl允许查看报文

