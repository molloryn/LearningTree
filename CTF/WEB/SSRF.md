

[了解SSRF,这一篇就足够了 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/2115)
[SSRF漏洞用到的其他协议（dict协议，file协议） - My_Dreams - 博客园 (cnblogs.com)](https://www.cnblogs.com/zzjdbk/p/12970919.html)



# 协议

## Gopher

gopher协议支持发出GET、POST请求：可以先截获get请求包和post请求包，在构成符合gopher协议的请求。gopher协议是ssrf利用中最强大的协议

## File
基本形式：`file://host/path`
