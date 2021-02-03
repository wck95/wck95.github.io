---
title: HTTP-跨域问题
date: 2021-02-02 23:26:46
image: /images/title_images/500.jpg  #设置本地图片
keywords: cors, 跨域
tags:
---

## 什么是跨域问题？

在浏览器端进行 Ajax 请求时会出现跨域问题，那么什么是跨域?   如何解决跨域呢？先看浏览器端出现跨域问题的现象，如下图所示：

<img src="WebHttp-cors/cors.jpg" alt="img" style="zoom:200%;" />

## [#](#什么是跨域问题？)什么是跨域问题？

跨域，指的是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器对 JavaScript 施加的安全限制。同源策略（Sameoriginpolicy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。同源策略会阻止一个域的javascript脚本和另外一个域的内容进行交互。 

## [#](#什么是同源？)什么是同源？

所谓同源是指:  协议，域名，端口均相同 ! 一个请求url的**协议、域名、端口**三者之间任意一个与当前页面url不同即为跨域;

### 非同源的一些限制

【1】无法读取非同源网页的 Cookie、LocalStorage 和 IndexedDB

【2】无法接触非同源网页的 DOM

【3】无法向非同源地址发送 AJAX 请求

| **当前页面url**           | **被请求页面url**               | **是否跨域** | **原因**                       |
| ------------------------- | ------------------------------- | ------------ | ------------------------------ |
| http://www.test.com/      | http://www.test.com/index.html  | 否           | 同源（协议、域名、端口号相同） |
| http://www.test.com/      | https://www.test.com/index.html | 跨域         | 协议不同（http/https）         |
| http://www.test.com/      | http://www.baidu.com/           | 跨域         | 主域名不同（test/baidu）       |
| http://www.test.com/      | http://blog.test.com/           | 跨域         | 子域名不同（www/blog）         |
| http://www.test.com:8080/ | http://www.test.com:7001/       | 跨域         | 端口号不同（8080/7001）        |

## [#](#如何解决跨域问题？)如何解决跨域问题？

### [#](#使用-cors（跨资源共享）解决跨域问题)使用 CORS（跨资源共享）解决跨域问题

CORS 是一个 W3C 标准，全称是"跨域资源共享"（Cross-origin resource sharing）。它允许浏览器向跨源服务器，发出 XMLHttpRequest 请求，从而克服了 AJAX 只能同源使用的限制。

CORS 需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE 浏览器不能低于 IE10。整个 CORS 通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS 通信与同源的 AJAX 通信没有差别，代码完全一样。浏览器一旦发现 AJAX 请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。因此，实现 CORS 通信的关键是服务器。只要服务器实现了 CORS 接口，就可以跨源通信（在 `header` 中设置：`Access-Control-Allow-Origin`）

1】**设置document.domain解决无法读取非同源网页的 Cookie问题**

因为浏览器是通过document.domain属性来检查两个页面是否同源，因此只要通过设置相同的document.domain，两个页面就可以共享Cookie（此方案仅限主域相同，子域不同的跨域应用场景。）

```javascript
// 两个页面都设置
document.domain = 'test.com';
```

【2】**跨文档通信 API：window.postMessage()**

调用postMessage方法实现父窗口http://test1.com向子窗口http://test2.com发消息（子窗口同样可以通过该方法发送消息给父窗口）

它可用于解决以下方面的问题：

- 页面和其打开的新窗口的数据传递
- 多窗口之间消息传递
- 页面与嵌套的iframe消息传递
- 上面三个场景的跨域数据传递

```javascript
// 父窗口打开一个子窗口
var openWindow = window.open('http://test2.com', 'title');
// 父窗口向子窗口发消息(第一个参数代表发送的内容，第二个参数代表接收消息窗口的url)
openWindow.postMessage('Nice to meet you!', 'http://test2.com');
```

调用message事件，监听对方发送的消息

```javascript
// 监听 message 消息
window.addEventListener('message', function (e) {
  console.log(e.source); // e.source 发送消息的窗口
  console.log(e.origin); // e.origin 消息发向的网址
  console.log(e.data);   // e.data   发送的消息
},false);
```

【3】**使用 JSONP 解决跨域问题**

JSONP （JSON with Padding）是服务器与客户端跨源通信的常用方法。最大特点就是简单适用，兼容性好（兼容低版本IE），缺点是只支持get请求，不支持post请求。JSONP是 JSON 的一种“使用模式”，可用于解决主流浏览器的跨域数据访问的问题。由于同源策略，一般来说位于 `server1.example.com` 的网页无法与 `server2.example.com` 的服务器沟通，而 HTML 的 `<script>` 元素是一个例外。利用 `<script>` 元素的这个开放策略，网页可以得到从其他来源动态产生的 JSON 资料，而这种使用模式就是所谓的 JSONP。用 JSONP 抓到的资料并不是 JSON，而是任意的 JavaScript，用 JavaScript 直译器执行而不是用 JSON 解析器解析（需要目标服务器配合一个 `callback` 函数）。

①原生实现：

```html
<script src="http://test.com/data.php?callback=dosomething"></script>
// 向服务器test.com发出请求，该请求的查询字符串有一个callback参数，用来指定回调函数的名字
// 处理服务器返回回调函数的数据
<script type="text/javascript">
    function dosomething(res){
        // 处理获得的数据
        console.log(res.data)
    }
</script>
```

② jQuery ajax：

```javascript
$.ajax({
    url: 'http://www.test.com:8080/login',
    type: 'get',
    dataType: 'jsonp',  // 请求方式为jsonp
    jsonpCallback: "handleCallback",    // 自定义回调函数名
    data: {}
});
```

③ Vue.js

```javascript
this.$http.jsonp('http://www.domain2.com:8080/login', {
    params: {},
    jsonp: 'handleCallback'
}).then((res) => {
   console.log(res); 
})
```

【4】**使用 CORS 解决跨域问题**

CORS 是跨域资源分享（Cross-Origin Resource Sharing）的缩写。它是 W3C 标准，属于跨源 AJAX 请求的根本解决方法。

**1、普通跨域请求：只需服务器端设置Access-Control-Allow-Origin**

**2、带cookie跨域请求：前后端都需要进行设置**

**【前端设置】**根据xhr.withCredentials字段判断是否带有cookie

①原生ajax

```javascript
var xhr = new XMLHttpRequest(); // IE8/9需用window.XDomainRequest兼容

// 前端设置是否带cookie
xhr.withCredentials = true;

xhr.open('post', 'http://www.domain2.com:8080/login', true);

xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');

xhr.send('user=admin');

xhr.onreadystatechange = function() {
 	if (xhr.readyState == 4 && xhr.status == 200) {
		alert(xhr.responseText);
    }
};
```

② jQuery ajax 

```javascript
$.ajax({
   url: 'http://www.test.com:8080/login',
   type: 'get',
   data: {},
   xhrFields: {
       withCredentials: true    // 前端设置是否带cookie
   },
	crossDomain: true,   // 会让请求头中包含跨域的额外信息，但不会含cookie
}); 
```

③vue-resource

```javascript
Vue.http.options.credentials = true
```

④ axios 

```javascript
axios.defaults.withCredentials = true
```

### [#](#cors-与-jsonp-的比较)CORS 与 JSONP 的比较

【1】CORS 与 JSONP 的使用目的相同，但是比 JSONP 更强大。

【2】JSONP 只支持 GET 请求，CORS 支持所有类型的 HTTP 请求。JSONP 的优势在于支持老式浏览器，以及可以向不支持 CORS 的网站请求数据。

【3】核心思想：网页通过添加一个`<script>元素`，向服务器请求 JSON 数据，服务器收到请求后，将数据放在一个指定名字的回调函数的参数位置传回来。



**【服务端设置】**

服务器端对于CORS的支持，主要是通过设置Access-Control-Allow-Origin来进行的。如果浏览器检测到相应的设置，就可以允许Ajax进行跨域的访问。

① Java后台

```java
/*
 * 导入包：import javax.servlet.http.HttpServletResponse;
 * 接口参数中定义：HttpServletResponse response
 */

// 允许跨域访问的域名：若有端口需写全（协议+域名+端口），若没有端口末尾不用加'/'
response.setHeader("Access-Control-Allow-Origin", "http://www.domain1.com"); 

// 允许前端带认证cookie：启用此项后，上面的域名不能为'*'，必须指定具体的域名，否则浏览器会提示
response.setHeader("Access-Control-Allow-Credentials", "true"); 

// 提示OPTIONS预检时，后端需要设置的两个常用自定义头
response.setHeader("Access-Control-Allow-Headers", "Content-Type,X-Requested-With");
```

② Nodejs后台

```javascript
var http = require('http');

var server = http.createServer();

var qs = require('querystring');

server.on('request', function(req, res) {
    var postData = '';
    
    // 数据块接收中
    req.addListener('data', function(chunk) {
        postData += chunk;
    });

    // 数据接收完毕
	req.addListener('end', function() {

        postData = qs.parse(postData);
		// 跨域后台设置
        res.writeHead(200, {
            'Access-Control-Allow-Credentials': 'true',     // 后端允许发送Cookie
			'Access-Control-Allow-Origin': 'http://www.domain1.com', // 允许访问的域（协议+域名+端口）
           /* 
            * 此处设置的cookie还是domain2的而非domain1，因为后端也不能跨域写cookie(nginx反向代理可以实现)，但只要
            * domain2中写入一次cookie认证，后面的跨域接口都能从domain2中获取cookie，从而实现所有的接口都能跨域访问
            */
            'Set-Cookie': 'l=a123456;Path=/;Domain=www.domain2.com;HttpOnly'//HttpOnly的作用是让js无法读取cookie  
        });
	    res.write(JSON.stringify(postData));
        res.end();
    });
});
server.listen('8080');
console.log('Server is running at port 8080...');
```



### [#](#使用-nginx-反向代理解决跨域问题)使用 Nginx 反向代理解决跨域问题

以上跨域问题解决方案都需要服务器支持，当服务器无法设置 `header` 或提供 `callback` 时我们就可以采用 Nginx 反向代理的方式解决跨域问题。

以下为文件上传的跨域配置方案：

```bash
user  nginx;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen 80;
        server_name upload.myshop.com;
        add_header 'Access-Control-Allow-Origin'  '*';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range';
        location / {
            proxy_pass  http://192.168.0.104:8888;
            if ($request_method = 'OPTIONS') {
                add_header Access-Control-Allow-Origin  *;
                add_header Access-Control-Allow-Headers X-Requested-With;
                add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,PATCH,OPTIONS;
                # 解决假请求问题，如果是简单请求则没有这个问题，但这里是上传文件，首次请求为 OPTIONS 方式，实际请求为 POST 方式
                # Provisional headers are shown.
                # Request header field Cache-Control is not allowed by Access-Control-Allow-Headers in preflight response.
                add_header Access-Control-Allow-Headers DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range;
                return 200;
            }
        }
    }
}
```