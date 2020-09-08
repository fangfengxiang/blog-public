## proxy_pass使用内置变量报错

---



### 缘起

前两天有同学提到个问题：有一个域名`a.fangle.com` 下uri携带指定参数的路由转发到`b.fangle.com` 这个域名的app模块下。请求要达到的效果大概是这样的：

```
http://a.fangle.com/test/index.php?r=tools/Translator&volume=50
=> 
http://b.fangle.com/app/test/index.php?r=tools/Translator&volume=50
```
nginx 配置大概如下：
- nginx.conf
```
events {
    worker_connections 1024;
}

http {
    include a.conf;
    include b.conf;
}
```
-  a.conf
```
server {
    listen 80;
    server_name a.fangle.com;
 
    location ~\.php$ {
        if ($args ~ r=tools\/Translator) {  
           proxy_pass http://b.fangle.com/app?$request_uri;
           break;
        }
    }

    access_log  logs/a.access.log;
    error_log  logs/a.error.log;
}
```
- b.conf
```
server {

    server_name b.fangle.com;
    listen 80;

    location / {
        echo "b.com";
        echo $uri;
        echo $request_uri;
        echo "b.end";
    }

    access_log  logs/b.access.log;
    error_log  logs/b.error.log;
}
```
but 一直502 ，于是我就过来玩了一把。
![image.png](https://upload-images.jianshu.io/upload_images/5261067-7fdb0d587425e2e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### proxy_pass 
我们找到error_log
```
2020/02/22 23:19:10 [error] 20274#149076: *57 no resolver defined to resolve b.fangle.com, client: 127.0.0.1, server: a.fangle.com, request: "GET /test/index.php?r=tools/Translator&volume=50 HTTP/1.1", host: "a.fangle.com"
```
这是因为proxypass使用了nginx变量，会自动做域名反向解析。
解决方法:
- a.conf  的location 中添加
```
 resolver 127.0.0.1;
```
重启发现报了一个新错
```
2020/02/22 23:22:05 [error] 20342#150105: *59 b.fangle.com could not be resolved (3: Host not found), client: 127.0.0.1, server: a.fangle.com, request: "GET /test/index.php?r=tools/Translator&volume=50 HTTP/1.1", host: "a.fangle.com"
```
开启本机的dns服务后：
 ![image.png](https://upload-images.jianshu.io/upload_images/5261067-012169802196cf91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 重写uri
当然你可能会说，我能不能不要resolve。我看你就是想为难我胖虎。
其实也是可以的，proxy_pass 如果不指定uri的时候，nginx会自动带上当前的uri
- a.conf
```
if ($args ~ r=tools\/Translator) {   
        proxy_pass http://b.fangle.com/app$request_uri;
 }
```
![image.png](https://upload-images.jianshu.io/upload_images/5261067-b8d30895f5356554.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
但是这个地址reqest_uri不是我们想要的，so我们还是要重写以下uri才能使用 
放弃这种想法

```
if ($args ~ r=tools\/Translator) {   
       set $request_uri "/app${request_uri}"; 
        proxy_pass http://b.fangle.com;
 }
```
nginx内置变量不允许修改

### 利用rewrite重写

```
set $proxy_pass_b false;
location ~\.php$ { 
        if ($args ~ r=tools\/Translator) {  
            set $proxy_pass_b true; 
            rewrite ^(.*)$ /app$request_uri? last;
        }
 }

location /app {
    #echo $request_uri;
    #echo $args;
        
    if ($proxy_pass_b = true) {
          proxy_pass http://b.fangle.com;
    }
}
```
![image.png](https://upload-images.jianshu.io/upload_images/5261067-676cf7d77169823f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 利用openresty 重写
待测试

如果允许域名发生变化，也可以考虑使用return or rewirte
### return
return 是nginx最简单的转发方式
- a.conf
```
 if ($args ~ r=tools\/Translator) {  
       return 301 http://b.fangle.com/app?$request_uri;
 }
```
效果：
![image.png](https://upload-images.jianshu.io/upload_images/5261067-9c0aea87137064b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### rewrite
- a.conf
```
if ($args ~ r=tools\/Translator) {   
            rewrite ^(.*)$ http://b.fangle.com/app$uri last;
 }
```
效果同return
![image.png](https://upload-images.jianshu.io/upload_images/5261067-9c0aea87137064b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
rewrite 和 return 没有本质上的区别，当然这种情况常用return，因为语义更清晰。rewirte 常用于url重写，就是server内部跳转，效率更高



---

### 参考阅读
- [Nginx中文文档](https://www.nginx.cn/doc/)
- [Nginx echo 模块的使用](https://www.jianshu.com/p/a636ca5fa8fa)
- [DNSmasq for mac 搭建局域网DNS服务器](https://www.jianshu.com/p/c1765b927aea)
- [一篇文章说透Nginx的rewrite模块](https://www.cnblogs.com/minirice/p/8872093.html)