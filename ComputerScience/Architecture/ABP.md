# ABP

## 简介

`ABP` 是 `ASP.NET Boilerplate`的全称，意味着它是一种代码层面的最佳实践的框架：

1. 实现`DDD`的领域设计概念，以经典分层结构实现。
2. 模块化设计。(`Modular`)
3. 实现了诸多的基础设施层，提供了开箱即用的模板机制。
4. 也提供前端的解决方案。
5. 支持多租户架构（`Multi-Tenancy`）
6. 支持微服务架构。

[Abp vNext 源码分析 - 文章目录 - MyZony - 博客园](https://www.cnblogs.com/myzony/p/10722506.html)

![](../../attachments/Pasted%20image%2020240620151358.png)

## 模块化

### 启动

#### OnApplicationInitializationAsync

> **`OnApplicationInitializationAsync` 和 `OnApplicationInitialization` 在官方的 module 中一般只用Async，而不会执行到同步的方法**

### UI

管理员密码：`admin/1q2w3E*`

## 数据库

### Unit Of Work

[Getting Started AspNetCore Application | Documentation Center | ABP.IO](https://docs.abp.io/en/abp/latest/Getting-Started-AspNetCore-Application)

## 错误排查

### 数据库异常

#### A second operation was started on this context instance

同一个依赖注入的类的多个仓储共用一个`DbContext`（待确认），因此无法同步执行。注意异步方法的调用，是否都进行了`await`。注意入口方法是否是`void`忘记等待。

#### DbUpdateConcurrencyException

`The database operation was expected to affect 1 row(s), but actually affected 0 row(s).`

是因为`Update`操作时，此数据不存在。此数据可能已经被删除或已经被`Update`而无法匹配上。
也有可能是需要`Add`的操作，错误的使用了`Update`方法。

#### AbpDbConcurrencyException

`ConcurrencyStamp`原理是生成`SQL`语句时带上`ConcurrencyStamp=@old`，然后更新时更新为新的，如果失败证明数据库那边已经被其他修改了（证明版本不一致）。

其他可能：

1. 因为令牌在`AbpContext SaveChanges`时进行修改，若这次进行保存数据库失败，下次再进行修改，则也会抛出该异常。
2. 一次请求，A微服务需要修改，A调用B，B恰好也去修改状态，这时候A再进行修改则会取出旧令牌匹配。（逻辑上是串行，其实没有问题）

##### 修改令牌

`.GetAsync()`查出的实体实例被修改后，然后又重新多次查询相关实例并客户端侧修改，即使没有使用 `Update` 等方法也会导致并发异常。（这里是同事写了个递归函数）
初步判断应该是令牌修改是`ABP`客户端侧判断，而非交给数据库判断，然后多次查询修改时发现令牌不匹配，直接在客户端侧触发并发修改异常。

只读查询功能似乎要额外设置。具体看 `.GetAsync()`设置。

##### 多线程触发

领域事件中`UpdateAsync`产生`AbpDbConcurrencyException`问题。最后发现其实就是多线程并发异常。眼光不能局限在某个服务，这次是事件多次触发，`Redis`拿到旧的数据导致的

[处理并发冲突 - EF Core | Microsoft Learn](https://learn.microsoft.com/zh-cn/ef/core/saving/concurrency?tabs=data-annotations)

#### Cannot access a disposed context instance

`Cannot access a disposed context instance. A common cause of this error is disposing a context instance that was resolved from dependency injection and then later trying to use the same context instance elsewhere in your application.`

`Repository`中的`DbContext`不可以`using`，直接交由`ABP`框架管理生命周期。

```csharp
await using var context = await _repository.GetDbContextAsync(); //导致错误
//直接使用
var context = await _repository.GetDbContextAsync();
```

**而且`Repository`依赖`UnitOfWork`，否则无法获取`DbContext`，特别是`ABP`封装的方法会触发，如`Any`、`FirstOrDefault`等，而`GetList`等却不会触发**

**对于不是`ApplicationService`等默认开启了`UnitOfWork`的，需要手动增加`UnitOfWork`标签，或手动使用`unitOfWork`进行`Begin`**。比如在`EventHandler`中。注意要使用`virtual`才会生效。

```csharp
[UnitOfWork]
public virtual async Task HandleEventAsync(MyEto eventData)

//还有手动的方式
using (var uow = _unitOfWorkManager.Begin()
{
    var client = await clientCredentialRepository.SingleOrDefaultAsync(x => x.ClientId == client_id);
    await uow.CompleteAsync();
}
```

### Repository中的基类属性为null没有注册

只能通过`Abp Module`进行注册，不能简单地绕过它的`module`使用`AddAssembly`方法进行自动注册，否则会带来很多蜜汁问题。

### 注册问题

没有引用合适的`Module`

```
The requested service 'Volo.Abp.DependencyInjection.ObjectAccessor`1[[Microsoft.AspNetCore.Builder.IApplicationBuilder, Microsoft.AspNetCore.Http.Abstractions, Version=7.0.0.0, Culture=neutral, PublicKeyToken=adb9793829ddae60]]' has not been registered
```

### Instances cannot be resolved and nested lifetimes cannot be created

`Instances cannot be resolved and nested lifetimes cannot be created from this LifetimeScope as it has already been disposed.`

[C# Async Antipatterns (markheath.net)](https://markheath.net/post/async-antipatterns)

```csharp
public void Main()
{
	IRepoXXX a;
	Method(a);//未等待导致多线程异常。
}

public async void Method(IRepoXXX a)
{
	await a.GetQueryable();
}
```

### 发现没有保存

可能要到`UnitOfWork`结束后才会一起保存，或使用`autoSave=true`

## 依赖注入

```csharp
public IAbpLazyServiceProvider LazyServiceProvider { get; set; } = default!;
```

属性注入必须为 `public property with public setter`

### 替换默认实现

[Dependency Injection | Documentation Center | ABP.IO](https://docs.abp.io/en/abp/latest/Dependency-Injection#replace-a-service)

使用 

```csharp
[Dependency(ReplaceServices = true)]
```

## Distributed Event Bus

`EventBus`实现基类为`EventBusBase`

### DaprEventBus

`DaprDistributedEventBus.cs`中继承`EventBus`实现。
`IDaprSerializer`用于序列化，`ABP`默认实现为`Utf8JsonDaprSerializer`，实际使用`IJsonSerializer`实现。而该接口`ABP`默认实现为`AbpSystemTextJsonSerializer`，使用`AbpSystemTextJsonSerializerOptions`进行配置。

推送时有个巨坑，在`IsAbpDaprEventData`中会直接检测5个`Json Node(pubSubName, topic, messageId, jsonData, correlationId)`是否包含，但其中的`CorrelationId`可能是`null`，因此如果使用了`IgnoreWhenDefault`，这里检测不通过就不会进入`Dapr`的事件`Handler`
可以通过使用`app.UseCorrelationId()`解决。

> `app.UseCloudEvents();` 用于解析`data_base64`等`application/cloudevents+json`格式的请求。必须位于`abp`的`UseConfiguredEndpoints`之前。

### 仓储层推送

[Distributed Event Bus | Documentation Center | ABP.IO](https://docs.abp.io/en/abp/latest/Distributed-Event-Bus#subscribing-to-the-events)

`IEntityChangeEventHelper`接口来判断是否需要`Publish Entity`变更`Event`

### 错误排查

#### Value cannot be null

在`Dapr GetEventType`时`Topic`为`null`报错
因为换用了自己的`JsonSerialize`，但是由于`ABP`拼接`topic`，`data`，`pubsubname`等都是用

## Json序列化

`IJsonSerializer`接口
实现`AbpSystemTextJsonSerializer`
其中配置项是`AbpSystemTextJsonSerializerOptions`。只作用于`Abp`的`IJsonSerializer`、`DaprClientFactory`等，而不会作用于`Controller`(`MVC`框架)用的`JsonOptions`，需要分别配置

```csharp
// 巨坑：这个Abp的AbpAspNetCoreMvcModule默认帮我们实现了DateTime转换，自己写的Converter将不会进入。而且它的转换失败不会报错，只会为null。
Configure<AbpJsonOptions>(options =>
{
	options.InputDateTimeFormats.Add("yyyy-MM-dd HH:mm:ss");
	options.InputDateTimeFormats.Add("yyyy-MM-dd");
	options.InputDateTimeFormats.Add("yyyy-MM-ddTHH:mm:ss");
	options.InputDateTimeFormats.Add("yyyy-MM-dd HH:mm");

	options.OutputDateTimeFormat = "yyyy-MM-dd HH:mm:ss";
});

// 巨坑：Abp的DateTime转换位于AbpSystemTextJsonSerializerModifiersOptions，以及AbpDateTimeConverterModifier
Configure<AbpSystemTextJsonSerializerModifiersOptions>(options =>
{
    options.Modifiers.Clear();
});
```

### 丢失精度问题

雪花类型`long id`均使用`string`代替，否则会遇到`json`序列化丢失精度问题

[Why does JSON.parse corrupt large numbers? - JSON Editor Online](https://jsoneditoronline.org/indepth/parse/why-does-json-parse-corrupt-large-numbers/)

`C# System.Text.Json` 可以通过设置 `JsonSerializerOptions.NumberHandling` 来设定是否转为字符串类型。

## Time （UTC与本地时间）
