
# 接口

## 请求参数

### 内置类型

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

所以最好使用`Frombody`只用DTO request类来写，其他的限制太复杂。


### Minimal API
需要用`async (HttpResponse response, HttpContext context)`的异步才能被Swagger展示出来，可能是个bug

```cs
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


# 依赖注入

#### 使用依赖注入的方式构造实例，且支持部分参数由用户传递（顺序不限）
`object ActivatorUtilities.Createlnstance(IServiceProvider provider, Type instanceType, params object[] parameters)`

#### 部分注入
```cs
 var client = DaprClient.CreateInvokeHttpClient(appId);
//方式一
services.TryAddScoped<ICommandFlight>(provider => ActivatorUtilities.CreateInstance<CommandFlightHttpApi>(provider, client));
//方式二 （HttpClient适用，因为被封装）未测试
services.TryAddScoped<ICommandFlight, CommandFlightHttpApi>();
services.AddHttpClient<CommandFlightHttpApi>(_ => client.CreateGrpcService<ICommandFlight>());
```


# 问题

## 