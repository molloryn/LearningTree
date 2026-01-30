# 依赖注入

## 关系矩阵
在 ASP.NET Core 的依赖注入（Dependency Injection, DI）系统中，理解生命周期的**兼容性规则**至关重要。如果配置不当，会引发“捕获依赖（Capturing Dependencies）”问题，导致内存泄漏或逻辑错误。

以下是针对 **Singleton（单例）**、**Scoped（范围内）** 和 **Transient（瞬时）** 生命周期的深度梳理。

---

## 1. 三种生命周期的基础定义

|**生命周期**|**英文名**|**行为模式**|
|---|---|---|
|**单例**|**Singleton**|根容器创建后**只创建一个实例**，直到应用程序关闭。|
|**范围内**|**Scoped**|在**每个请求（Scope）**内创建一个实例。同一次请求内共享。|
|**瞬时**|**Transient**|**每次注入/获取**时都会创建一个全新的实例。|

---

## 2. 注入规则与兼容性（核心重点）

核心原则是：**长生命周期的服务不能直接注入短生命周期的服务。**

### 注入兼容性矩阵

|**宿主（注入到谁）**|**可注入 Singleton?**|**可注入 Scoped?**|**可注入 Transient?**|
|---|---|---|---|
|**Singleton**|✅ 允许|❌ **禁止**|✅ 允许（但有风险）|
|**Scoped**|✅ 允许|✅ 允许|✅ 允许|
|**Transient**|✅ 允许|✅ 允许|✅ 允许|

---

## 3. 为什么 Singleton 不能注入 Scoped？

这是面试和开发中最常遇到的陷阱。

- **现象：** 如果你在一个 Singleton 服务中注入了一个 Scoped 服务，Scoped 服务就会被 Singleton “捕获”。
    
- **后果：** 这个 Scoped 服务本该在请求结束时销毁，但因为被 Singleton 引用，它会一直存活到程序关闭。这会导致数据库连接无法释放、用户上下文信息错乱等严重问题。
    
- **框架保护：** 在 `Development` 环境下，ASP.NET Core 默认会开启 `ValidateScopes` 检查。如果检测到 Singleton 引用 Scoped，程序启动时会直接抛出 `InvalidOperationException`。
    

---

## 4. 各种组合的副作用梳理

### 1. Singleton 注入 Transient

- **结果：** 这个 Transient 实例会变成“事实上的单例”。
    
- **注意：** 因为 Singleton 只初始化一次，它构造函数里的 Transient 实例也就固定下来了。虽然不报错，但失去了 Transient “每次新建”的特性。
    

### 2. Scoped 注入 Transient

- **结果：** Transient 实例在当前请求范围内是固定的。
    
- **注意：** 只有在当前 Scoped 服务被创建时，Transient 才会生成一次。
    

### 3. 如何在 Singleton 中使用 Scoped 服务？

如果你必须在单例（如 `IHostedService` 后台任务）中使用 Scoped 服务（如 `DbContext`），不能通过构造函数注入，而必须手动创建 Scope：



```cs
public class MySingletonService(IServiceProvider serviceProvider) 
{
    public void DoWork()
    {
        using (var scope = serviceProvider.CreateScope())
        {
            var scopedService = scope.ServiceProvider.GetRequiredService<IMyScopedService>();
            // 业务逻辑...
        } // scope 释放，scopedService 也随之销毁
    }
}
```

---

## 5. 常见组件的默认生命周期

在开发中，请务必留意框架自带组件的生命周期：

- **DbContext (EF Core):** 默认为 **Scoped**。
    
- **IConfiguration:** **Singleton**。
    
- **ILogger\<T\>:** **Singleton**。
    
- **HttpClientFactory:** 内部管理，通常作为 **Transient** 或 **Singleton** 注入使用。
    

---

## 总结图示

- **向下兼容：** 短周期可以依赖长周期。
    
- **向上孤立：** 长周期不可依赖短周期（除非手动开启局部作用域）。


## 设置

```cs
  //可用于严格检验注册的依赖注入依赖是否成功；另外还能够检测是否存在生命周期不一致注入的问题  
builder.Host.UseDefaultServiceProvider(options =>  
{  
#if DEBUG  
	options.ValidateOnBuild = true;  
	options.ValidateScopes = true;  //会严格检测生命周期，必须满足要求，有些是运行时才会发现，比如通过工厂方法注册  
	//导致注入这种错误：Cannot resolve scoped service 'Microsoft.Extensions.Options.IOptionsSnapshot`1[Monica.StateStore.StackExchange.Modules.ModuleRedisStateStoreOption]' from root provider.  
#endif  
});
```

## 注意事项

#### 使用依赖注入的方式构造实例，且支持部分参数由用户传递（顺序不限）

```csharp
object ActivatorUtilities.CreateInstance(IServiceProvider provider, Type instanceType, params object[] parameters)
```

#### Scoped和Transient的须知
`IOptionSnapshot<T>`是`Scoped`的生命周期，如果用`IApplicationBuilder`中的根容器直接`GetService()`其实只算做一次请求，下一次还是同一个实例。需要创建`Scoped`才能真正读取到不同的实例。


#### 部分注入
```csharp
var client = DaprClient.CreateInvokeHttpClient(appId);
//方式一
services.TryAddScoped<ICommandFlight>(provider => ActivatorUtilities.CreateInstance<CommandFlightHttpApi>(provider, client));
//方式二 （HttpClient适用，因为被封装）未测试
services.TryAddScoped<ICommandFlight, CommandFlightHttpApi>();
services.AddHttpClient<CommandFlightHttpApi>(_ => client.CreateGrpcService<ICommandFlight>());
```

#### 性能比较
性能不会差太远
[c# - ASP.NET Core Singleton instance vs Transient instance performance - Stack Overflow](https://stackoverflow.com/questions/54790460/asp-net-core-singleton-instance-vs-transient-instance-performance)

### 差异
`TryAdd{lifetime}()` ... for example `TryAddSingleton()` ... peeps into the DI container and looks for whether **ANY** implementation type (concrete class) has been registered for the given service type (the interface). If yes then it does not register the implementation type (given in the call) for the service type (given in the call). If no , then it does.

`TryAddEnumerable`(ServiceDescriptor) on the other hand peeps into the DI container , looks for whether the **SAME** implementation type (concrete class) as the implementation type given in the call has already been registered for the given service type. If yes, then it does not register the implementation type (given in the call) for the service type (given in the call). If no, then it does. Thats why there is the `Enumerable` suffix in there. _The suffix indicates that it CAN register more than one implementation types for the same service type!_

多次注册构造函数获取实例只会获取最后一次注册，需要获取所有实现可以注入`IEnumerable<IMyInterface>` （待验证）