# Configurations

## Configure 方法

该方法似乎可以不同地方执行然后叠加生效。

```csharp
services.Configure<MvcOptions>(p =>
{
    p.Filters.AddService(typeof(ProxyMvcFilter));
});
```

## TroubleShooting
### 对于ICollection的行为是添加
添加的行为对于`Option`类带默认值的，以及有多个来源的都是一样。
源码在`Microsoft.Extensions.Confgiuration.ConfigurationBinder #line 530` 中
