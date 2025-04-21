# 概述

模块化的基础设施库，以`ASP.NET Core`为基础，大程度上解耦基础设施、库间的依赖，可单独使用某个模块而无需引入整个繁重的框架。

## 特性

1. 统一的、直觉性的基础设施库的注册、配置方式，上手简易。
2. 自动进行中间件注册，只需要进行依赖注入配置即可。
3. 无需担忧服务、中间件重复注册，任何模块均自动仅注册一次。
4. 提供高性能的服务注册方式，对于反射的自动注册操作，在所有的`MoModule`注册过程中，仅遍历一次。
5. 及时释放注册临时对象，减少注册内存占用。如果普通方式进行注册，通常使用`static`来贯穿服务注册及中间件阶段，而一般不会释放。
6. 自动解决中间件注册顺序
7. 可视化注册过程中的依赖关系，及时提醒可能的注册失败、误操作等。


## 组成部分
MoModule作为库的核心注册机制
每个Library有自己的MoModule，该Module中，并
1. `ModuleOption{ModuleName}`: 模块Option的设置
2. `ModuleGuide{ModuleName}`: 模块配置的向导类
3. `Module{ModuleName}`: 含有依赖注入的方式以及配置ASP.NET Core中间件等具体实现
4. `ModuleBuilderExtensions{ModuleName}`: 面向用户的扩展方法

## 使用方式
开发者使用原生的方式注册Module，每个Module的方式都类似如下示例：
```cs
services.AddMoModuleAuthorization(Action<ModuleOptionAuthorization> option = null)
```


其中注册方法返回值为ModuleGuide，用于指引用户进一步配置模块相关功能
```cs
public class ModuleGuideAuthorization
{
     public ModuleGuideAuthorization AddPermissionBit<TEnum>(string claimTypeDefinition) where TEnum : struct, Enum
 {
     ConfigureExtraServices(nameof(AddPermissionBit), context =>
     {
         var checker = new PermissionBitChecker<TEnum>(claimTypeDefinition);
         PermissionBitCheckerManager.AddChecker(checker);
         context.Services.AddSingleton<IPermissionBitChecker<TEnum>, PermissionBitChecker<TEnum>>(_ => checker);
     });
     return this;
 }
}
```


### 模块配置
	
为了提高开发者设置的优先级，在开发者AddMoModule的过程中，配置Option的Action设置如果不是模块第一次注册，仍会覆盖上一次的配置设置。这是因为模块的级联注册可能在开发者使用模块之前，已经进行了模块的配置。
如若有其他配置需求，供开发者提供如下扩展方法：
```cs
services.ConfigMoModulePost<TModuleOption>(Action<TModuleOption> config);
services.ConfigMoModulePre<TModuleOption>(Action<TModuleOption> config);
```


### 模块级联注册

模块内部进行级联注册时可采用如下方法获取`Guide`类进行进一步配置。
```cs
protected TOtherModuleGuide DependsOnModule<TOtherModuleGuide>()  where TOtherModuleGuide : MoModuleGuide, new()
{
    return new TOtherModuleGuide()
    {
        GuideFrom = CurModuleEnum()
    };
}
```




# 原理

## MoDomainTypeFinder 

用于获取当前应用程序相关程序集及搜索，可设置业务程序集。
用于Core扫描相关程序集所有类型进行自动注册、项目单元发现等，提高整个框架的性能。

## ModuleBuilderExtensions
依赖注入背后的设置方式
```cs
public static ModuleGuideAuthorization AddMoModuleAuthorization<TEnum>(this IServiceCollection services, string claimTypeDefinition) where TEnum : struct, Enum
{
    return new ModuleGuideAuthorization()
        .AddDefaultPermissionBit<TEnum>(claimTypeDefinition);
}
```