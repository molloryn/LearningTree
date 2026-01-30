# Publish & subscribe

如果应用程序有配置，则程序本身需要暴露 `/dapr/subscribe` 接口，供边车获取程序所监听的主题。

边车启动初始化时，首先会去调用应用的 `/dapr/subscribe` 方法获取应用是否有监听分布式事件，如有则注册到边车之中。

> `Visual Studio DockerCompose`容器开发中，容器边车需要重新启动才会重新读取，并监听新的主题。



## 消息消费模式

目前消息消费模式是按照消费者组，对应 `RabbitMQ` 中的 `Queue` 的概念，一个主题是一个 `Exchanges`，比如主题 `ProtocolPlatform.PublishedLanguages.DomainFlight.Events.EventAlterFlight`，对于不同服务的消费者组是类似如下的 `Queue`：
```
dev-service-adaptor-msg-preprocessor-for-atc-ProtocolPlatform.PublishedLanguages.DomainFlight.Events.EventAlterFlight
dev-service-alarm-api-ProtocolPlatform.PublishedLanguages.DomainFlight.Events.EventAlterFlight
dev-service-atc-flight-adaptor-ProtocolPlatform.PublishedLanguages.DomainFlight.Events.EventAlterFlight
```
相同消费组中消息进行竞争。

> 注意 Dapr Bindings默认则只是Queue，不同服务也是同一个Queue （没有 Exchanges) 


### 同消费者组非竞争模式

[Allow PubSub to opt-out from competing consumer pattern to send events to every instance · Issue #3176 · dapr/dapr](https://github.com/dapr/dapr/issues/3176)
[PubSub: notify all instances of an application (rather than \"competing consumers\") · Issue #3332 · dapr/dapr](https://github.com/dapr/dapr/issues/3332)

当前解决方案是，定义一个不同行为的`component`，其中提供`consumerID`为`{uuid}`，这样经过该`component`的所有创建及消费主题将会变成指定模式。

## 批量推送

`Dapr`提供的批量推送，在消息队列入队时是自动将消息拆分成了单个，比如一次推送1000条一个包，那么入消息队列的消息是1000条消息，保持了队列的数据结构一致性。


## 发布订阅

边车启动初始化时，首先会去调用应用的 `/dapr/subscribe` 方法获取应用是否有监听分布式事件，如有则注册到边车之中。


## 问题排查

### Kafka

#### 问题一

`1195725856 is GET[space] encoded as a big-endian, four-byte integer (see here for more information on how that works). This indicates that HTTP traffic is being sent to Kafka port 9092, but Kafka doesn't accept HTTP traffic, it only accepts its own protocol (which takes the first four bytes as the receive size, hence the error).`

#### 问题二 dapr显示无法连接到kafka（is your cluster reachable?）

需要配置好地址。

项目`Rebuild`后可能是`Dapr`开的比较快，而`Kafka`还没起来，导致以为连不上。第二次再启动就行了。

#### 问题三 Message was too large, server rejected it to avoid allocation error when using Headers

```yaml
- name: maxMessageBytes
  value: 10485760000
```

配置弄大点？

注意`Dapr`这些`component`的`yaml`配置文件需要`Rebuild`才会生效

#### 问题四 无法从外部连接kafka，dapr访问地址将自动变为内部集群ip

`Kafka`需配置外部访问监听地址：[Kafka Listeners - Explained](https://rmoff.net/2018/08/02/kafka-listeners-explained/)



#### 问题五 (rabbitmq) operation basic.consume caused a connection exception not_allowed: "attempt to reuse consumer tag 

这里dapr将ConsumerID和ConsumerTag合并了，RabbitMQ不允许有相同的ConsumerID被重新注册多次，ConsumerID在RabbitMQ是指的一个连接对象，不是消费者组。 RabbitMQ的消费者组概念是相同的TopicQueue。 
我遇到这个问题是因为使用了 `streaming subcription api`，然后边车对于相同的主题（不同Handler）注册了多次，没有处理这个情况。 

#### 一直启动时失败

1. **正常启动阶段**：日志前半部分是 Kafka 启动时的常规操作 —— 加载各个分区的日志（LogLoader），包括业务主题（如 UserService 相关）和 Kafka 内部主题`__consumer_offsets`（消费者偏移量存储），此时 Kafka 还在加载 93 个分区的日志（进度到 59/93），尚未完成启动。
2. **关键触发点**：`Terminating process due to signal SIGTERM` 表明 Kafka 进程收到了 SIGTERM 终止信号（通常由 kill 命令、容器停止、系统超时机制触发）。
你的配置中`initialDelaySeconds: 10`（仅等待 10 秒就开始检测），但从日志能看到 Kafka 加载 93 个分区的日志需要数十秒甚至数分钟，**即使 client 端口（默认 9092）提前监听，Kafka 内部仍未完成启动**，此时若触发健康检查失败或外部终止逻辑，就会导致启动中断。
```yaml
livenessProbe:
  tcpSocket:
    port: client  # 对应Kafka的客户端端口（默认9092）
  initialDelaySeconds: 300  # 延长至5分钟（核心修改），给足日志加载时间
  timeoutSeconds: 5         # 检测超时时间保持5秒即可
  periodSeconds: 10         # 每10秒检测一次
  successThreshold: 1       # 1次成功即判定存活
  failureThreshold: 6       # 失败6次（总计60秒）才重启，避免偶发网络波动误判
```