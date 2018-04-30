ajax 基本工作原理

#### ajax基本概念
> 异步的javascript 和 xml

#### XMLHttpRequest 对象使用步骤
- 创建一个XMLHttpRequest 对象
```
var xhr = new XMLHttpRequest();
```
- 打开请求的url ，如果是post请求的话 需要设置请求头
	- open(method,URL,[asyncFlag],[userName],[password])
	- setRequestHeader(header, value)
```
xhr.setRequestHeader("Content-type","application/x-www-form-urlencoded");
xhr.open('post', 'index.php' );
```
- 发送请求
```
xhr.send(name=fox&age=18) // get 请求的参数要是在open的url后拼接了 可以不传，直接写 xhr.send()
```
- 注册事件 onreadystatechange 状态改变就会调用
onreadystatechange
```
xhr.onreadystatechange = function () { 
	// 这步为判断服务器是否正确响应
  if (xhr.readyState == 4 && xhr.status == 200) {
    console.log(xhr.responseText);//响应内容处理
  } 
};
```


### ajax不能跨域解决方案
- 1 代理
通过同域下的服务端脚本转发请求
例如要请求的是B服务器的index.php脚本，先请求A服务器的index.php  然后A服务器帮忙请求B服务器的index.php再将数据返回给ajax请求
- 2 CORS 
原理：服务器设置Access-Control-Allow-Origin HTTP响应头之后，浏览器将会允许跨域请求
> CORS是HTML5标准提出的跨域资源共享(Cross Origin Resource Share)，支持GET、POST等所有HTTP请求。CORS需要服务器端设置 Access-Control-Allow-Origin 头，否则浏览器会因为安全策略拦截返回的信息。（也叫 XHR2？）
```
//服务端设置 响应头信息
Access-Control-Allow-Origin: *    # 允许所有域名访问，或者
Access-Control-Allow-Origin: http://a.com # 只允许所有域名访问

CORS又分为简单跨域和非简单跨域请求
```
更详细参考 [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)

- 3 jsonp
JSONP是JSON with Padding的略称，jsonp 只支持GET请求
原理：所有具有 src 属性的HTML标签都是可以跨域的

- webSocket
[浏览器同源政策及其规避方法](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html) 


### jQuery中使用 ajax
- $.ajax 最底层的
- `$.load, $.get,$.post` 第二层
- `$.getScript $.getJson` 高层方法
```
load(url,[data],[callback])
get(url,[data],[callback],[type]) // type 为返回类型格式
get()的callback有两个参数 function（data,textStatus）{}
post() 与 get()几乎一致
```