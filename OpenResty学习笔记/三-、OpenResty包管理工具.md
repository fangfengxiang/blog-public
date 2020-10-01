### Opm
官方说OpenResty 1.11版本后默认安装，但是我是1.15版本，还是没有安装。所以我们先检查一下。
```
ll /usr/local/openresty/bin/
```
![](https://upload-images.jianshu.io/upload_images/5261067-086ed394b2246c2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

没有的话 就执行 
- ` yum install openresty-opm` 安装


#### 使用
- 搜索包
` opm search http`
- 网址
https://opm.openresty.org/

### LUAROCKS
https://luarocks.org/

 是 OpenResty 世界的另一个包管理器，诞生在 OPM 之前。不同于 OPM 里只包含 OpenResty 相关的包，LuaRocks 里面还包含 Lua 世界的库。举个例子，LuaRocks 里面的 LuaSQL-MySQL，就是 Lua 世界中连接 MySQL 的包，并不能用在 OpenResty 中。

### awesome
https://github.com/bungle/awesome-resty


#### Opm 项目 介绍

目录 结构介绍
