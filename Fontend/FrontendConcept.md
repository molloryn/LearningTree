# CORS(Cross-Origin Resource Sharing) 跨站资源共享

## 概念

配置CORS不是限制浏览器请求，反而是放宽浏览器的跨域请求。因为浏览器有一个默认的策略：`同源策略(Same-Origin Policy)`。
所谓同源指的是：
- URI scheme, for example `http://` or `https://`
- Hostname like `www.xyz.com`
- Port number like `8000` or `80` (default HTTP port)

只要以上三点有一个不同，浏览器就会认为这是跨站请求，默认情况下是无法拿到跨站的资源的。

但我们又有跨站的需求，所以诞生了`CORS`。

为什么要有同源策略？看一个安全风险的场景：A网站（受害网站）进行登录含有的cookies，用户访问B网站（恶意网站），B网站将cookies从浏览器中读取，然后用这个cookies去恶意请求A网站的服务器，会产生安全风险。这也是所谓的`跨站请求伪造（CSRF）`
![](../attachments/Pasted%20image%2020230808185806.png)

同源策略并没有保护A网站，因为你完全可以绕开浏览器从外部请求A网站接口。这种保护性是浏览器提供的，是给**一般的用户**提供安全性，以避免这种`CSRF`攻击，以免用户的财产受到侵害。

在严苛的同源策略下，CORS开了一个口子，让浏览器能够发起跨站请求。CORS反而给某些随便设置的服务端带来了`CSRF`漏洞。

### Cross-Origin Server与Origin Server
我们需要弄清源站和跨站的定义。
![](../attachments/Pasted%20image%2020230808191824.png)

CORS是在服务端配置的，来指明该服务端**是否支持跨站请求**。如果没有配置，则浏览器法向该`Cross-Origin Server`的请求都默认无法通过（被浏览器拦截）。

### CORS流程

#### 简单请求
##### 定义

1) 请求方法是以下三种方法之一：
- HEAD
- GET
- POST
2) HTTP的头信息不超出以下几种字段：
- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：只限于三个值`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`
![](../attachments/Pasted%20image%2020230808192247.png)
##### 流程
1. 浏览器发起请求，并且自动在`header`请求头强制加上来源`Origin`给服务器检查；
2. 服务器处理请求（有配置CORS的还可以拒绝处理响应），返回数据，有配置CORS的会返回响应头：
	- Access-Control-Allow-Origin（必含）：**该字段是必须的，否则请求按失败处理**。它的值要么是请求时`Origin`字段的值，要么是一个`*`，表示接受任意域名的请求。
	- Access-Control-Allow-Credentials（可选）：该项标志着请求当中是否能够包含cookies信息，只有一个可选值：true（必为小写）。如果不允许包含cookies，请略去该项，而不是填写false。
	- Access-Control-Expose-Headers（可选）： 该字段可选。CORS请求时，`XMLHttpRequest`对象的`getResponseHeader()`方法只能拿到6个基本字段：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`。如果想拿到其他字段，就必须在`Access-Control-Expose-Headers`里面指定。上面的例子指定，`getResponseHeader('FooBar')`可以返回`FooBar`字段的值。

> CORS请求默认不发送Cookie和HTTP认证信息。如果要把Cookie发到服务器，一方面要服务器同意，指定`Access-Control-Allow-Credentials`字段。一方面，前端的AJAX请求也要设置`withCredentials`属性。

3. 浏览器检查CORS响应头，如果包含了当前的源则放行，反之拦截。（即如果这个跨站源服务端没有配置CORS，浏览器不会使用返回的数据来渲染页面等）

	这里需要注意，浏览器是拦截响应，而不是拦截请求，跨域请求是发出去的，并且服务端做了响应，只是浏览器拦截了下来。
#### 复杂请求

##### 定义 
指那些简单请求之外的请求。
![](../attachments/Pasted%20image%2020230808212302.png)

##### 流程
1. 浏览器发起预检请求`preflight`，请求方法是`OPTIONS`，请求头带上来源`origin`，不包含请求体；
2. 服务器返回检查结果，配置CORS头。未配置CORS的，浏览器视为拒绝跨域，则停止请求。
3. 对比服务器返回的CORS头，通过则浏览器发起真正请求。
	 - Access-Control-Allow-Origin（必含）：见简单请求
	 - Access-Control-Allow-Methods（必含） ：这是对预请求当中Access-Control-Request-Method的回复，这一回复将是一个以逗号分隔的列表。尽管客户端或许只请求某一方法，但服务端仍然可以返回所有允许的方法，以便客户端将其缓存。 
	 - Access-Control-Allow-Headers（当预请求中包含Access-Control-Request-Headers时必须包含） ： 这是对预请求当中Access-Control-Request-Headers的回复，和上面一样是以逗号分隔的列表，可以返回所有支持的头部。
	 - Access-Control-Allow-Credentials（可选） ：和简单请求当中作用相同 
	 - Access-Control-Max-Age（可选）：以秒为单位的缓存时间。预请求的的发送并非免费午餐，允许时应当尽可能缓存。
4. 服务端返回数据。
5. 后续浏览器关于该请求就和简单请求一样进行。
## 参考

[Complete Guide to CORS (reflectoring.io)](https://reflectoring.io/complete-guide-to-cors/)
[跨域资源共享 CORS 详解 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2016/04/cors.html)
