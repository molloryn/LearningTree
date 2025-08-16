# Service invocation

`daprd sidecar `有`app-port`，是指宿主微服务的`API`监听地址，由`sidecar`去调用的

而`dapr-http-port`是设置`sidecar`本身的`HTTP`的监听地址，是由`dapr-placement`解析出边车地址然后两个边车间沟通的`HTTP`监听地址。`gRPC`的监听地址也可以设置

`sidecar`的默认的`HTTP`监听是`3500`，`gRPC`的是`50001`。一般不需要修改（修改后暂时不知道怎么调通，因为`dapr-placement`默认是映射到默认端口？）。

## 常见问题

### initial http2 frame from server is not a settings frame:http2.GoAwayFrame

边车配置的 `app-protocol` 是`gRPC`，然后应用中却通过`Dapr HttpClient`去调用`Http`服务，造成问题。

### ERR_DIRECT_INVOKE fail to invoke

有可能是命名空间的问题，边车与边车之间不在同一个命名空间，则默认无法互相通信。比如在`K8S`中挂载的边车，默认是采用了`K8S`自己项目的命名空间。

可以通过改写调用`api`进行，如：`https://localhost:3500/v1.0/invoke/myappid.<namespace>/method/ping`
[How to: Service invocation across namespaces | Dapr Docs](https://docs.dapr.io/developing-applications/building-blocks/service-invocation/service-invocation-namespaces/)

#### 如果发现dapr解析后请求的地址与容器组地址对应不上

1. 有可能是修改了`K8S DNS`的缓存刷新时间或策略，导致重启微服务后未能立刻刷新缓存导致边车错误路由到旧的容器组地址。排查`K8S DNS`问题：[Kubernetes(K8S)](Kubernetes(K8S).md#DNS)
2. 如果测试发现仅有一个节点的容器如`worker1`，其上的服务的边车均有解析问题，解析`DNS`不对，其他节点正常，使用`busybox`测试`coreDNS`解析正常，则该问题暂无解决方案(`Dapr 1.14.4`)

### error invoke  50002 Unavailable  (dapr 1.14.4)

调用接口时突然中断提示不可用，边车自动重启。
排查后期间怀疑是设置了内存限制带来的问题。
最后根据`daprd`边车重启堆栈发现问题指向`StateStore`。重新部署`Redis`后正常？最后根据堆栈指向的`BulkGet`操作发现可能是`Parallelism`未进行限制的问题，增加限制后正常。

```log
Conn has unread data happened
panic: interface conversion: interface {} is string, not map[interface {}]interface {} goroutine 2745 [running]: github.com/dapr/components-contrib/state/redis.(*StateStore).getDefault(0xc0013dd2d0, {0x75512f0, 0xc000997350}, 0xc0008dc180) /home/runner/go/pkg/mod/github.com/dapr/components-contrib@v1.14.4/state/redis/redis.go:250 +0x406 github.com/dapr/components-contrib/state/redis.(*StateStore).Get(0xc0013dd2d0, {0x75512f0, 0xc000997350}, 0xc0008dc180) /home/runner/go/pkg/mod/github.com/dapr/components-contrib@v1.14.4/state/redis/redis.go:315 +0x566 github.com/dapr/components-contrib/state.DoBulkGet.func1(0x166) /home/runner/go/pkg/mod/github.com/dapr/components-contrib@v1.14.4/state/bulk.go:90 +0x186 created by github.com/dapr/components-contrib/state.DoBulkGet in goroutine 2370 /home/runner/go/pkg/mod/github.com/dapr/components-contrib@v1.14.4/state/bulk.go:82 +0x85 panic: interface conversion: interface {} is string, not map[interface {}]interface {}
```