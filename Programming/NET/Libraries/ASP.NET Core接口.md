# ASP.NET Core接口

## 接口

### 请求参数

#### 内置类型

```csharp
[HttpPost("save")]
public async Task Save([FromBody] string content)
{
}
```
如果使用了`[FromBody]`，而且类型使用`string`或`int`等，则请求内容也得是纯字符串，而且是必须带`"`的字符串，而不能是`{"content":"data"}`等`json`内容，否则会请求失败，获得诸如`JSON value could not be converted to System.String`的错误。
这个设计很逆天，是我讨厌的设计之一，因为客户端那边根本没法通过：
```csharp
var result = await Http.PostAsync("api/method", new StringContent(strData));
```
调用，这里的`string`是不带`"`的，会报错，就很反人类。。
首先`StringContent`的`MediaType`就不对，是`plain/text`的，而`FromBody`是只支持`application/json`格式的，会报`Unsupported Media Type`的错误。
然后就是`"`的问题了，具体错误忘记记录了。
具体用法要看`FromBody`的限制：
[Parameter Binding in ASP.NET Web API - ASP.NET 4.x | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/web-api/overview/formats-and-model-binding/parameter-binding-in-aspnet-web-api#using-frombody)

所以最好使用`FromBody`只用DTO request类来写，其他的限制太复杂。


#### Minimal API
需要用`async (HttpResponse response, HttpContext context)`的异步才能被`Swagger`展示出来，可能是个bug

```csharp
//获取Channel列表
endpoints.MapGet("/data-channel/channels", async (HttpResponse response, HttpContext context) =>
{
    var channels = DataChannelCentral.Channels.Select(p => p.Value).Select(p => new
    {
        p.Id,
        Middlewares = p.GetMiddlewares().Select(m => m.GetType().Name),
        Endpoints = p.GetEndpoints().Select(m => m.GetType().Name)
    });
    await context.Response.WriteAsJsonAsync(channels);
}).WithName("获取DataChannel状态列表").WithOpenApi(operation =>
{
    operation.Summary = "获取DataChannel状态列表";
    operation.Description = "获取DataChannel状态列表";
    operation.Tags = tagGroup;
    return operation;
});
```





## 问题


## 待学
[深入解析ASP.NET Core MVC应用的模块化设计[上篇]-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2394132)



## 中间件

中间件的顺序非常重要，比如 `UseRouting` 必须在 `UseEndpoints` 之前，否则会报错。

`UseRouting` 必须接一个 `UseEndpoints`，用于配置 `UseRouting` 中路由信息。而且一旦第一次用了 `UseEndpoints`，后续再次使用 `UseEndpoints` 的操作都会附加到第一次 `UseEndpoints` 上，意味着两个 `UseEndpoints` 之间的中间件 `Middleware` 都会失效。

因此应该先添加中间件，最后添加 `UserEndpoints`。

示例顺序：
```cs
 app.UseMoExceptionHandler();
 app.UseRouting();
 app.UseSharedCors(config);
 app.UseSharedDapr(config);
 app.UseSharedLogging(config);
 app.UseSharedJwt(config);
 app.UseInvocationProxy(config);
 app.UseSharedGrpcServices(config);
 app.UseSharedHttpServices(config);
 app.UseSharedDatabase(config);
 app.UseEndpointsHttpServices(config, "基础功能");
 app.UseEndpointsSharedJwt(config, "基础功能");
 app.UseEndpointsSharedDatabase(config, "基础功能");
 app.UseEndpointsSharedConfig(config, "基础功能");

```
