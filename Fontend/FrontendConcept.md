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

在严苛的同源策略下，CORS开了一个口子，让浏览器能够发起跨站请求。
同源策略并没有保护A网站，因为你完全可以绕开浏览器从外部请求A网站接口。这种保护性是浏览器提供的，是给**一般的用户**提供安全性，以避免这种`CSRF`攻击，以免用户的财产受到侵害。

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
1. 浏览器发起请求，并且自动加上请求的来源`origin`给服务器检查；
2. 服务器返回数据，并返回检查结果，配置CORS响应头；
3. 浏览器检查CORS响应头，如果包含了当前的源则放行，反之拦截；

这里需要注意，浏览器是拦截响应，而不是拦截请求，跨域请求是发出去的，并且服务端做了响应，只是浏览器拦截了下来。
#### 复杂请求