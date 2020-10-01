OpenResty 颠覆了 高性能服务端的开发模式

### 搭建高性能服务端的关键点有两个
- 缓存
- 异步非阻塞

### 缓存速度
- 内存 > SSD > 本地磁盘
- 本机内存 > 网络
- 进程内 > 进程外

### 异步非阻塞
- 事件驱动

nginx module c是性能之王
lua 是嵌入式 脚本语言
openResty 使用 lua jit

### 工作原理
![image.png](https://upload-images.jianshu.io/upload_images/5261067-ba9d29f5cc62d570.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把luaJIT的虚拟机嵌在nginx的worker中，性能接近于Cmodule, 更容易测试。

- 同node.js 对比
node.js 是第一门把异步非阻塞加入语言的。
比较麻烦的是，使用回调来实现异步非阻塞，不符合开发思维。我们希望使用同步代码的方式来实现异步功能。
- Golang
可能需要在线热调试。Golang不支持。（systemTap 开始支持了golang）


### nginx Lua API 介绍
- 40多个指令，120个API 


### openResty 两种缓存方式
- shared_dict  字典缓存
预设缓存大小，有锁的开销。在多个worker中间共享
- lua_resty_lrucache
预设缓存Key个数，每个worker独有，减少锁开销。内存占用太多。
只有GET SET Delete 3个API


### 高性能服务端 需要考虑的点
- 缓存失效风暴。
利用lua_resty_lock 加锁来解决，缓存到期时，对于多个请求同时查库的，只请求一个，另外的等该请求重设缓存后，从缓存中获取


###  OpenResty 的执行阶段 
是对nginx 执行流程的简化
![image.png](https://upload-images.jianshu.io/upload_images/5261067-ac453296770bc788.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- set_by_lua
对流程分支的判断
- rewirte_by_lua 
转发操作
- access_by_lua
接口权限，外部应用防火墙功能
- content_by_lua
业务代码执行
- header_filter_by_lua
过滤应答头
- body_filter_by_lua
应答包过滤处理
- log_by_lua
异步完成日志

不同阶段，有不同阶段的处理行为。这个是OpenResty的一大特色
![image.png](https://upload-images.jianshu.io/upload_images/5261067-0b0bf57dbaa3e54e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如何自学

### OpenResty 

OpenResty 不等于 lua_nginx_module
它包含lua_nginx_module,测试集，resty库，工具集




