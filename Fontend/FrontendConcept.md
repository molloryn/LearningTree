# CORS(Cross-Origin Resource Sharing) 跨站资源共享

配置CORS不是限制浏览器请求，反而是放宽浏览器的跨域请求。因为浏览器有一个默认的策略：`同源策略(Same-Origin Policy)`。
所谓同源指的是：
- URI scheme, for example `http://` or `https://`
- Hostname like `www.xyz.com`
- Port number like `8000` or `80` (default HTTP port)

只要以上三点有一个不同，浏览器就会认为这是跨站请求，默认情况下是无法拿到跨站的资源的。

但我们又有跨站的需求，所以诞生了`CORS`。

为什么要有同源策略？看一个安全风险的场景：A网站进行登录含有的cookies，用户访问B网站，B网站将cookies从浏览器中读取，然后用这个cookies去恶意请求A网站的服务器，会产生安全风险。这也是所谓的`跨站请求伪造（CSRF）`
![](../attachments/Pasted%20image%2020230808185806.png)

CORS并没有保护A网站，因为你完全可以绕开浏览器从外部请求A网站接口。这种保护性是浏览器提供的，是给一般的用户提供安全性，以避免这种`CSRF`攻击。
