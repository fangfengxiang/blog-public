### Linux软件源码安装常用步骤
1. wget下载压缩包
2. 解压并进入源码目录
3. ./configure  运行配置 添加配置常数(./configure --help)
4. make 编译
5. make install 安装


### 源码编译安装PHP

#### 下载安装包
`wget  http://am1.php.net/get/php-7.2.4.tar.gz`

wget  [参数] [URL]
wget 指令介绍 

#### 解压安装包并进入源码目录
```
tar -zxvf php-7.2.4.tar.gz
cd php-7.2.4
```
tar 指令介绍
#### configure 配置
- 可查看configure 配置参数
`./configure --help`
- 指定安装目录
`--prefix=/usr/bin/php`
- 指定php.ini存放目录
`--with-config-file-path=/usr/bin/php/etc`
不设置默认是lib，配置参数指定的是cli的ini。fpm的ini要在fpm.conf中去设置

此步骤常见错误是缺少安装依赖软件如gcc,autoconfig,libxml2等，可根据提示安装对应的依赖软件`sudo apt install xxx`

#### 编译
`make`

此步骤常见的错误是报内存不够用(virtual memory exhausted: Cannot allocate memory)，可试着重新设置configure参数，不要安装fileinfo扩展，或者设置交换分区

 #### 安装
 `make install`

#### 测试是否安装成功
`/usr/bin/php/bin/php -m`

### 将PHP添加进环境变量
```
sudo ~/.bashrc
alias php7=/usr/bin/php
source ~/.bashrc
```
这种方法只为当前用户将php添加到环境变量，并没有为所有的用户。如果要为所有用户都添加环境变量，则需要修改`
/etc/profile`。

### 添加php.ini 文件
```
cp php.ini-development /usr/bin/php/etc
mv /usr/bin/php/etc/php.ini-development /usr/bin/php/etc/php.ini 
```
当然上面的存放php.ini的目录不一定正确，php.ini存放的目录依赖于configure时配置的参数，所以，合理的是执行
`php -i | grep php.ini` 来获取应该存放的目录

### 源码编译安装php遇到的坑

#### 缺少依赖的软件
- 没有gcc 
`sudo apt install gcc`
- autoconfig
 `sudo apt install autoconfig`
- 没有libxml2
 `sudo apt install libxml2-dev`

事实上，如果报错缺少libxml2这个软件，`apt install libxml2`不一定能安装上，因为在你的apt源里，libxml2的软件包可能叫libxml2-dev,所以你可以根据错误提示搜索一下这个软件包相关的有哪些。

#### 配置php.ini后不生效
这种情况，通常是因为php.ini存放的位置不正确引起的。如果configure的时候没有设置`--with-config-file-path=Path`,那么默认是在安装目录的lib目录下，有设置则在对应的目录。

可以通过`php -i | grep php.ini`获得php.ini应该存放在哪个目录下，对应的将php.ini引入就能生效

#### 编译过程中内存不够
常见于内存小于2G的服务器( virtual memory exhausted: Cannot allocate memory)
有两种解决方案：
- 不安装fileinfo扩展
因为5.3以后会默认安装fileinfo扩展，如果你的服务器内存过小就会在编译的过程中报内存不够的错误。可通过重新设置./configure的参数`--disable-fileinfo` 再重新编译解决。当然这种方法治标不治本。建议还是通过设置虚拟内存来解决

- 配置虚拟内存(交换分区)
使用free可查看内存信息`free `
使用`sudo swapon -s` 查看交换分区信息
设置虚拟内存步骤：
```
创 建 swap 文 件
cd /var
sudo dd if=/dev/zero of=swapfile bs=1024 count=2048000
sudo mkswap swapfile
激 活
sudo chmod 600 swapfile
sudo swapon swapfile
开 机 自动 创建
sudo vim /etc/fstab
/swapfile   none    swap    sw    0   0
```

### 源码编译安装PHP扩展
下载安装包
`wget`
进入源码目录
`cd`
执行phpize(PHP目录下的bin目录下)生成configure文件
`/usr/bin/php/bin/phpize`
执行configure设置参数
```
./configure  --with-php-config=/usr/bin/php/bin/php-config 
``` 
编译安装
`make && make install`

如果是php自带的扩展,还可以直接进入php源码的/ext目录对应的扩展目录
`cd php-7.2.5/ext/`执行`phpize`编译扩展，不需要下载扩展源码包，因为/ext里已经有该扩展的源码了。

而且自带的扩展除了用phpize编译某个扩展，也可以重新编译整个PHP，在编译时用参数添加扩展，例如在源码根目录运行
```
./configure --enable-pcntl --enable-posix
make && make install
```

其他一键式安装方式：
- apt install
- pecl install (/usr/bin/php/bin/pecl)
