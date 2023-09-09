
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
如果使用了`[FromBody]`，而且类型使用`string`或`int`等，则请求内容也得是纯字符串，而不能是`{"content":"data"}`等`json`内容，否则会请求失败，获得诸如`JSON value could not be converted to System.String`的错误。
