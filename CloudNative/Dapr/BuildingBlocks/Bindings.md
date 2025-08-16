# Bindings

## 输出绑定

[How-To: Use output bindings to interface with external resources | Dapr Docs](https://docs.dapr.io/developing-applications/building-blocks/bindings/howto-bindings/)

定义外部系统资源，封装系统调用，由`Dapr`解决调用方式。
按大类进行抽象如`MQ`、数据库、`SMTP`、`HTTP`，将消息传递通过`Dapr`实现的调用方式，给具体的外部服务（具体`SMTP`服务）或组件（具体如`Kafka`、`MySQL`等）。

### 输出绑定组件

定义外部绑定组件资源，以供边车调用。
定义连接种类、资源名、连接方式、鉴权等基础信息

### 操作

类似`RESTful`对资源定义，对资源的操作似乎只有增删改查几种`operation`？具体看`Dapr`支持的`Component`的文档，弄清支持的`operation`。

## 输入绑定

[How-To: Trigger your application with input bindings | Dapr Docs](https://docs.dapr.io/developing-applications/building-blocks/bindings/howto-triggers/)

内部服务接收来自外部的事件，可以看作是一个`trigger`

### 输入绑定会批量通知所有dapr边车

`Dapr`有一个设计，`component`可以有`scopes`，限定`binding component`到特定微服务。
[How-To: Scope components to one or more applications | Dapr Docs](https://docs.dapr.io/operations/components/component-scopes/)

### 绑定配置正确但未生效

日志显示成功初始化 `binding` 但未能接收事件。发现是因为 `pubsub` 的 `component` 配置有异常导致 `pubsub`模块初始化不成功，其中`pubsub`的队列地址与 `binding` 的消息队列地址一致。发现将`pubsub component`配置正确后 `binding`问题解决。