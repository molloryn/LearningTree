# 依赖注入



## 设置

```cs
  //可用于严格检验注册的依赖注入依赖是否成功；另外还能够检测是否存在生命周期不一致注入的问题  
builder.Host.UseDefaultServiceProvider(options =>  
{  
#if DEBUG  
	options.ValidateOnBuild = true;  
	options.ValidateScopes = true;  //会严格检测生命周期，必须满足要求，有些是运行时才会发现，比如通过工厂方法注册  
	//导致注入这种错误：Cannot resolve scoped service 'Microsoft.Extensions.Options.IOptionsSnapshot`1[MoLibrary.StateStore.StackExchange.Modules.ModuleRedisStateStoreOption]' from root provider.  
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