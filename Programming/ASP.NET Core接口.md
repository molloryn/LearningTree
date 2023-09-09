
# 接口

## 请求参数

### 内置类型

```csharp
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

具体用法要看`FromBody`的限制：
[Parameter Binding in ASP.NET Web API - ASP.NET 4.x | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/web-api/overview/formats-and-model-binding/parameter-binding-in-aspnet-web-api#using-frombody)


