
# 抓包检测绕过


#### SwitchyOmega无法抓包本地问题

只添加 `<-loopback>` 到SwitchyOmega的 Bypass-list 配置里面，需要去掉默认的：

```
127.0.0.1  
::1  
localhost
```


#### 证书问题

如果浏览器报不安全的连接，说明证书未被信任成功，需要导入证书并放入受信任的根证书认证机构。

#### 代理

![](../../attachments/Pasted%20image%2020230921000057.png)