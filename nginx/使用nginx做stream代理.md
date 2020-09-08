### 使用nginx做stream代理

---

### 缘起

我在云服务器上，搭了一些服务。为了方便，为通常是不设密码的。但是远程连的时候，不设置密码，就有安全性问题。

为了方便，我就想到用nginx做一个代理，这样服务既不需要对外网开放，也不需要设置密码。下面以redis为例。



### 实践

- 在nginx.conf 新增stream配置端

```
{

    worker_processes auto;
    error_log /var/log/nginx/error.log;
    pid /run/nginx.pid;

    # Load dynamic modules. See /usr/share/nginx/README.dynamic.
    include /usr/share/nginx/modules/*.conf;

    events {
        worker_connections 1024;
    }
    
    http {
    
    }
    
    stream {

        log_format stream_log '$remote_addr [$time_local] [id:$connection] '
                     '[$protocol $upstream_addr $status] [$bytes_sent $bytes_received] '
                     '$session_time';

        include /etc/nginx/stream.d/*conf;
    }
}
```

- 在 `  /etc/nginx/stream.d/ ` 目录下 新建 redis.conf 文件

```
upstream redis {
     server 127.0.0.1:6379;
}

server {
        listen  6000 ;
        #so_keepalive = 30m::10;
        tcp_nodelay on;
        preread_buffer_size 32k;


        access_log  /etc/nginx/logs/redis.access.log stream_log buffer=32k;

        # ip 白名单限制
        include /etc/nginx/ip_allow.conf;

        proxy_pass   redis;
        #proxy_read_timeout 120;
        proxy_connect_timeout 5s;
        proxy_timeout 5s;


}
```

- 白名单 `/etc/nginx/ip_allow.conf` 设置

```

allow 127.0.0.1;

#允许特定IP访问
#allow ip;
allow 14.145.20.203;
allow 120.197.197.124;

#禁止其他IP访问，防止攻击
deny all;
```

- 查看连接日志 `tail /etc/nginx/logs/redis.access.log -fn 20`

![image-20200908230016011](https://tva1.sinaimg.cn/large/007S8ZIlgy1gijn078nrej31kw0cadka.jpg)

403标示被拒绝的ip，200表示请求成功



有了这个模板，后面如果 有安装其他服务，只需要将nginx配置copy一份，稍做调整，就可以直接连上内网的服务了。so easy



---

### 参考阅读

- [ngx_stream_core_module](http://nginx.org/en/docs/stream/ngx_stream_core_module.html)
- [ngx_stream_upstream_module](http://nginx.org/en/docs/stream/ngx_stream_upstream_module.html)

- [ngx_stream_log_module](http://nginx.org/en/docs/stream/ngx_stream_log_module.html)

- [Accepting the PROXY Protocol](https://docs.nginx.com/nginx/admin-guide/load-balancer/using-proxy-protocol/)

  

# 
