常见的nginx中间件架构
===


### 常见的nginx中间件架构
- 静态资源服务
- 代理服务
- 七层负载均衡调度器(SLB)
- 动态缓存

#### 静态资源web服务
- 静态资源web服务相关配置
```
sendfile on   #开启sendfile机制从内核优化
```
新版本还可以在编译时增加参数`--with-file-aio`开启异步文件读取。这个现在应用认可度还不是很高，具体了解可以查看nginx的aio模块

- tcp-nopush
把多个包一次性整合之后再发送给客户端，而不是一次次发，在sendfile开启的情况下，提高网络包的传输效率，可在http、server和location中配置，默认是关闭(off)状态

- tcp-nodelay
数据包实时发送，不用等待，主要应用在实时性要求比较高的场景。在keepalive连接下，提高网络包的传输实时性。

- 压缩
`gzip on` 压缩传输，减少消耗带宽资源，提高网络传输效率
`gzip_comp_level level` 压缩级别
`gzip_http_version 1.0|1.1` 压缩版本

- 扩展nginx压缩模块
http_gzip_static_module 预读gzip模块
开启该功能，如果访问1.jpg文件会在目录下现在预压缩的1.jpg.gzip文件。预先压缩好，减少压缩时间
http_gunzip_module 应用支持gunzip压缩方式，为了支持某些浏览器不支持gzip的压缩包，应用比较少。

- 浏览器缓存设置-expires
添加Cache-Control,Expries头
`expires 24h` 设置缓存时间24h

- 跨域访问设置
header头设置-add_header name value
`add_header Access-Control-Allow-Origin www.baidu.com` 开启允许从这个域名跨域过来访问
`add_header Access-Control-Allow-Methods GET,POST,PUT `设置允许跨域的请求方法

- 防盗链
防盗链的解决思路：区分哪些是非正常用户的请求
基于http_refer防盗链配置模块`valid_referers none blocked`

#### 代理服务
Http代理 -> Http Server
ICMP\POP\IMAP代理 -> Mail Server
Https代理 ->Http Server 
RTMP代理 -> Media Server

- 正向代理
- 反向代理
proxy_pass URL;
- 正向代理和反向代理的区别在于代理的对象不一样，正向代理的对象是客户端，方向代理的对象是服务端
- `proxy_buffering on`	响应缓冲区，减少IO消耗

#### 负载均衡
四层负载均衡 -> 作用于OSI网络模型第四层TCP层控制,只需要进行包转发，效率高
七层负载均衡 -> 作用于应用层，nginx就是典型的七层负载均衡器

- nginx的负载均衡服务主要借助于proxy_pass和upstream
- upstream 必须配置在server层以外
- 负载均衡调度算法有多种，其中一种就是IPhash，主要是为了解决不同请求调度到不同的服务器，然后session失效的问题。使用iphash调度算法，能使同一个ip的请求调度到同一台服务器上

#### 动态缓存
- proxy_cache
- 大文件分片请求 `slice size`