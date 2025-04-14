# 概念

模块化的基础设施库，以`ASP.NET Core`为基础，尽量解耦基础设施间的依赖，无需引入繁重的框架

# 设计

实现MoDomainTypeFinder用于获取当前应用程序相关程序集及搜索，可设置业务程序集。
用于Core扫描相关程序集所有类型进行自动注册、项目单元发现等，提高整个框架的性能。

整个Services注册仅扫描一次业务程序集

```cs
AddService()
{
    Module.Register<TModule>(Action(option=>option));
}
```