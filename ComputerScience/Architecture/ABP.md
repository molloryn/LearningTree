
# 简介
ABP 是 ASP.NET Boilerplate的全称，意味着它是一种代码层面的最佳实践的框架：
1. 实现DDD的领域设计概念，以经典分层结构实现。
2. 模块化设计。(Modular)
3. 实现了诸多的基础设施层，提供了开箱即用的模板机制。
4. 也提供前端的解决方案。
5. 支持多租户架构（Multi-Tenancy）
6. 支持微服务架构。

[[Abp vNext 源码分析] - 文章目录 - MyZony - 博客园 (cnblogs.com)](https://www.cnblogs.com/myzony/p/10722506.html)


# 数据库

### Unit Of Work
[Getting Started AspNetCore Application | Documentation Center | ABP.IO](https://docs.abp.io/en/abp/latest/Getting-Started-AspNetCore-Application)


# 错误排查

### 数据库异常

#### A second operation was started on this context instance
同一个依赖注入的类的多个仓储共用一个DbContext（待确认），因此无法同步执行。注意异步方法的调用，是否都进行了await。注意入口方法是否是void忘记等待。

#### AbpDbConcurrencyException

ConcurrencyStamp原理是生成SQL语句时带上`ConcurrencyStamp=@old`，然后更新时更新为新的，如果失败证明数据库那边已经被其他修改了（证明版本不一致）。

##### 修改令牌
`GetAsync()`查出的实体实例被修改后，然后又重新多次查询相关实例并客户端侧修改，即使没有使用 `Update` 等方法也会导致并发异常。（这里是同事写了个递归函数）
初步判断应该是令牌修改是ABP客户端侧判断，而非交给数据库判断，然后多次查询修改时发现令牌不匹配，直接在客户端侧触发并发修改异常。

只读查询功能似乎要额外设置。具体看 `GetAsync()`设置。

##### 多线程触发
领域事件中UpdateAsync产生AbpDbConcurrencyException问题。最后发现其实就是多线程并发异常。眼光不能局限在某个服务，这次是事件多次触发，Redis拿到旧的数据导致的
`https://sourcegraph.com/github.com/abpframework/abp@4f6426add5b69bfb273f601b1ddd9f1f89099a72/-/blob/framework/src/Volo.Abp.EntityFrameworkCore/Volo/Abp/EntityFrameworkCore/AbpDbContext.cs?L347:17&popover=pinned`
`https://sourcegraph.com/github.com/abpframework/abp@4f6426add5b69bfb273f601b1ddd9f1f89099a72/-/blob/framework/src/Volo.Abp.EntityFrameworkCore/Volo/Abp/EntityFrameworkCore/AbpDbContext.cs?L520:28&popover=pinned`

[处理并发冲突 - EF Core | Microsoft Learn](https://learn.microsoft.com/zh-cn/ef/core/saving/concurrency?tabs=data-annotations)

#### Cannot access a disposed context instance. A common cause of this error is disposing a context instance that was resolved from dependency injection and then later trying to use the same context instance elsewhere in your application.'
Repository中的DbContext不可以`using`，直接交由ABP框架管理生命周期。
```cs
await using var context = await _repository.GetDbContextAsync(); //导致错误
//直接使用
var context = await _repository.GetDbContextAsync();

```

**而且Repository依赖UnitOfWork，否则无法获取DbContext**
**对于不是ApplicationService等默认开启了UnitOfWork的，需要手动增加UnitOfWork标签，或手动使用unitOfWork进行Begin**。比如在EventHandler中。注意要使用virtual才会生效。
```cs
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
只能通过Abp Module进行注册，不能简单地绕过它的module使用AddAssembly方法进行自动注册，否则会带来很多蜜汁问题。

### 注册问题
没有引用合适的Module
```
The requested service 'Volo.Abp.DependencyInjection.ObjectAccessor`1[[Microsoft.AspNetCore.Builder.IApplicationBuilder, Microsoft.AspNetCore.Http.Abstractions, Version=7.0.0.0, Culture=neutral, PublicKeyToken=adb9793829ddae60]]' has not been registered
```





# 依赖注入


```cs
public IAbpLazyServiceProvider LazyServiceProvider { get; set; } = default!;
```


属性注入必须为 public property with public setter

## 替换默认实现

[Dependency Injection | Documentation Center | ABP.IO](https://docs.abp.io/en/abp/latest/Dependency-Injection#replace-a-service)
使用 
```csharp
[Dependency(ReplaceServices = true)]
```



# Distributed Event Bus
`EventBus`实现基类为`EventBusBase`


#### DaprEventBus

`DaprDistributedEventBus.cs`中继承`EventBus`实现。
`IDaprSerializer`用于序列化，`ABP`默认实现为`Utf8JsonDaprSerializer`，实际使用`IJsonSerializer`实现。而该接口`ABP`默认实现为`AbpSystemTextJsonSerializer`，使用`AbpSystemTextJsonSerializerOptions`进行配置。

推送时有个巨坑，在`IsAbpDaprEventData`中会直接检测5个Json Node是否包含，但其中有可能null，因此如果使用了 IgnoreWhenDefault，这里检测不通过就不会进入Dapr的事件Handler

> `app.UseCloudEvents();` 用于解析data_base64等application/cloudevents+json格式的请求。必须位于abp的UseConfiguredEndpoints之前。

## 仓储层推送

[Distributed Event Bus | Documentation Center | ABP.IO](https://docs.abp.io/en/abp/latest/Distributed-Event-Bus#subscribing-to-the-events)

IEntityChangeEventHelper接口来判断是否需要Publish Entity变更Event


# Json序列化
`IJsonSerializer`接口
实现`AbpSystemTextJsonSerializer`
其中配置项是`AbpSystemTextJsonSerializerOptions`。只作用于Abp的`IJsonSerializer`、`DaprClientFactory`等，而不会作用于`Controller`(`MVC`框架)用的`JsonOptions`，需要分别配置


```cs
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
#### 丢失精度问题

雪花类型long id均使用string代替，否则会遇到json序列化丢失精度问题`https://jsoneditoronline.org/indepth/parse/why-does-json-parse-corrupt-large-numbers/`

# Time （UTC与本地时间）
