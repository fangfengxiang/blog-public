OpenResty 的两个基石：NGINX 和 LuaJIT

在 OpenResty 的开发中，我们需要注意下面几点：
- 要尽可能少地配置 nginx.conf；
- 避免使用 if、set 、rewrite 等多个指令的配合；
- 能通过 Lua 代码解决的，就别用 NGINX 的配置、变量和模块来解决。

NGINX 在进程启动的时候读取配置，并加载到内存中。
如果修改了配置文件，需要你重启或者重载 NGINX，再次读取后才能生效。

nginx  结构图
![image.png](https://upload-images.jianshu.io/upload_images/5261067-23c45c412df7cab3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


OpenResty 执行阶段
![image.png](https://upload-images.jianshu.io/upload_images/5261067-25b10611484966c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
