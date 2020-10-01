###  起步
- vim conf/nginx.conf
```
location /print_param {
       content_by_lua_file lua_script/params.lua;
}
```

### 获取GET请求参数
- vim lua_script/params.lua
```
local arg = ngx.req.get_uri_args()
for k, v in pairs(arg) do
    ngx.say('[GET ] key:', k, ' v:', v)
end
```
- `curl -X GET 'localhost/print_params?hello=world&test=now'`
![image.png](https://upload-images.jianshu.io/upload_images/5261067-3463ae4567447f8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 获取POST请求参数
- vim lua_script/params_lua
```
local arg = ngx.req.get_post_args()
for k, v in pairs(arg) do
    ngx.say('[POST] key:', k, ' v:', v)
end
```
- `curl -X POST 'localhost/print_params' -d 'hello=world'`
```
<html>
<head><title>500 Internal Server Error</title></head>
<body>
<center><h1>500 Internal Server Error</h1></center>
<hr><center>openresty/1.15.8.2</center>
</body>
</html>
```
why ？ 500 我们直接看一下服务端的错误日志。当然如果你想看客户端的报文，也可以加`-v ` 参数 ，curl就会打印出报文再发请求

- `tail   logs/error.log  -fn 10`
```
2019/10/05 17:36:47 [error] 42559#991866: *32 lua entry thread aborted: runtime error: /Users/fangfengxiang/Code/lua_test/lua_script/params.lua:7: no request body found; maybe you should turn on lua_need_request_body?
stack traceback:
coroutine 0:
	[C]: in function 'get_post_args'
	/Users/fangfengxiang/Code/lua_test/lua_script/params.lua:7: in main chunk, client: 127.0.0.1, server: localhost, request: "POST /print_params HTTP/1.1", host: "localhost"

```
可以看到日志提示我们 `maybe you should turn on lua_need_request_body`

- lua_need_request_body

![image.png](https://upload-images.jianshu.io/upload_images/5261067-56a8b144d2617eae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 文档说了 ，nginx 是不会默认读客户端的请求体，如果我们想读取到请求体，有两个方法：
1.  `vim conf/nginx.conf` 在server 中加上
```
lua_need_request_body on;
```
注意要重启openresty（因为改的是nginx配置文件，不是lua代码）
2. `vim lua_script/params.lua` 在    `ngx.req.get_post_args` 之前加上
```
ngx.req.read_body()
```
使用这种是官方推荐，并且也不用重启openresty。因为你改的是lua代码，所以能热加载。
再次执行   ` curl -X POST 'localhost/print_params' -d 'hello=world'`
![image.png](https://upload-images.jianshu.io/upload_images/5261067-590b7a43eb790e31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

补充：如果这个请求体，只在此处使用。你也可以使用`ngx.req.discard_body()` ，此函数会在获取连接中的数据后立即丢弃。

`ngx.req.get_post_args()` 返回的是一个lua table，如果你需要返回一个lua字符串，可以使用  `ngx.req.get_body_data()`
```
ngx.req.read_body() 
local arg = ngx.req.get_body_data()
ngx.say(arg)
```
### 获取http请求头
```
local headers = ngx.req.get_headers()
for k, v in pairs(headers) do
    ngx.say('[header] key:', k, ' v:', v)
end
```
```
$ curl -X POST 'localhost/print_params' -d 'hello=world'
[header] key:host v:localhost
[header] key:content-length v:11
[header] key:user-agent v:curl/7.54.0
[header] key:accept v:*/*
[header] key:content-type v:application/x-www-form-urlencoded
```
### 使用Nginx 内置绑定变量
ngx.req的get_xxx方法能获取到我们想要到数据，但是有些时候，其实并不需要那么麻烦，有些数据已经被填充进了nginx内置变量。
例如获取当前HTTP请求是POST OR GET，你当然可以使用  `ngx.req.get_method()` 。但是，更简洁地，你也可以
```
ngx.say(ngx.var.request_method)
```
`ngx.val` 可以使用nginx内置变量。具体的，可以查看下面的文档

---

### 推荐阅读
- [lua_need_request_body](https://github.com/openresty/lua-nginx-module#lua_need_request_body)
- [ngxreqread_body](https://github.com/openresty/lua-nginx-module#ngxreqread_body)
- [ngxreqget_body_data](https://github.com/openresty/lua-nginx-module#ngxreqget_body_data)
- [ngxreqget_headers](https://github.com/openresty/lua-nginx-module#ngxreqget_headers)
- [ngxvarvariable](https://github.com/openresty/lua-nginx-module#ngxvarvariable)
