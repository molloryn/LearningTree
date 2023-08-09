# 路由配置

### 正则转发

匹配正则表达式：`^/dapr/(.+?-service-api.*?)/(.*)`
转发路径模板：`/v1.0/invoke/$1/method/$2`

# 问题

## 幻觉问题

存在之前配置的路由，即使已经下线或删除，仍然保留其作用，影响后续的请求。触发条件未知，可能是路由为`/*`所致。

## 跨域问题

可以对网关整体进行跨域配置：
[Annotations | Apache APISIX® -- Cloud-Native API Gateway](https://apisix.apache.org/docs/ingress-controller/concepts/annotations/#cors)
也可以针对某一路由进行跨域配置，在路由编辑第三步插件中选择`CORS`插件进行配置即可