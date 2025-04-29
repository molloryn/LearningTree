# Mapster

```csharp
TypeAdapterConfig<TSource, TDestination>
    .NewConfig()
    .Map(dest => dest.Id, src => src.Id)
    .Map(dest => dest.Name, src => src.Name)
    .IgnoreNonMapped(true) //这里的意思是除了显式配置的Map，其他都忽略
```

## TroubleShotting

### Cannot convert immutable type
映射的是不同的类型，比如`DateTime`和`DateOnly` 除非写了两种类型的映射？

### 没有映射上
映射的类型可能无法访问。比如属性没有`set`方法或不是`public`的`set`方法。
