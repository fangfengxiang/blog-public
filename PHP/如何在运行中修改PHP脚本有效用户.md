### 引言
前几天，有个朋友(别问,问就是无中生友) 在群里发了个问题 ：
1. crontab是以root权限启动PHP定时脚本，生成的文件所属用户都是root用户。
2. root用户的文件不利于fpm进程(www)用户权限的访问。
3. 如何才能让定时脚本生成的文件，文件所属用户是www


### 思路一
首先肯定是让crontab以www用户来运行php脚本，但是试了一下crontab写上
```
0 * * * *  -u www php xxx.php
```
没有成功。那么就手动打包一下指令： 
```
php xxx.php
```
改为 ：
```
su www && php xxx.php
```
但是，这位同学的ubuntu系统，root用户切其他用户竟然要输入密码。所以就妥妥 失败了。

### 思路二
然后 我就想呀。哪就干脆在PHP运行时切换用户角色吧。《unix环境编程》中有介绍过，程序如何在运行中修改有效用户。也就是
```
setuid()
setgid()
```
php 应该也有类似的函数才对。于是我就搜了一下： 
https://www.php.net/manual/zh/function.posix-setuid.php
https://www.php.net/manual/zh/function.posix-setgid.php

然后直接就上代码了：
```
<?php

$info = posix_getpwnam("www");

posix_setgid($info['gid']);
posix_setuid($info['uid']);

$intTime = time();
$name = "/home/www/test{$intTime}.txt";
$ret = file_put_contents($name,"$intTime");

var_dump($ret,$name);
```
效果：
![image.png](https://upload-images.jianshu.io/upload_images/5261067-18d81dae1f2f6838.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这里值得一提的是：
你必须先setgid，再setuid, 如果顺序相反会导致setgid失败。
因为setuid后你已经是www用户了，就再无权限setgid了。


### 思路三
当然不要忘记其实 还有这个
https://www.php.net/manual/zh/function.chown.php
https://www.php.net/manual/zh/function.chgrp.php
所以  没事还是要多翻翻手册。惭愧惭愧
