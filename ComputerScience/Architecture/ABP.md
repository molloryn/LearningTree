
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

#### AbpDbConcurrencyException

领域事件中UpdateAsync产生AbpDbConcurrencyException问题。最后发现其实就是并发异常。眼光不能局限在某个服务，这次是事件多次触发，Redis拿到旧的数据导致的
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

而且Repository依赖UnitOfWork：
对于不是ApplicationService等默认开启了UnitOfWork的，需要手动增加UnitOfWork标签。比如在EventHandler中。注意要使用virtual才会生效。
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