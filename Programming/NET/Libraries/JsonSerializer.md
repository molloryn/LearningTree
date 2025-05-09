# System.Text.Json


## JsonConverter

不管是`JsonConverterFactory`还是`JsonConverter`，引擎按顺序遍历，每个类型只会选取第一次`CanConvert`返回`true`的`Converter`进行转换，所以注意顺序。


## TroubleShooting

### 未成功反序列化
`System.Text.Json`默认行为是只反序列化到`public`属性（或`init`？），对于非`public`的属性的需要加`JsonInclude`标签才会设置。
[c# - Use System.Text.Json to deserialize properties with private setters - Stack Overflow](https://stackoverflow.com/questions/62270834/use-system-text-json-to-deserialize-properties-with-private-setters/67206063#67206063)
默认是大小写敏感的，需要设置option insensitive
.NET 7后新增`JsonRequired`

#### 元组无法序列化

直至`.NET 8.0`，仍对元组序列化支持不够，目前仅有设置`IncludeFields=true`才会序列化成功，且序列化为`Items = xx`的形式，若有命名，反序列化时也为`(null, null)`


### 序列化值为{}
默认情况下是序列化当前类型字段，如果用`this`关键字要小心。

```csharp
/// <summary>
/// 将当前状态提取为可追踪链信息
/// </summary>
/// <returns></returns>
public string GetCurTracingData()
{
    return JsonSerializer.Serialize(this, TracingDataJsonOptions); //应改为(object)this
}
```

### 序列化失败

默认情况下不会序列化`Field`，只会序列化`Properties`。而`Tuple`是两个public field，所以也不会序列化`Tuple`

#### 类型不一致

比如`labelId:100000` 就无法序列化到 `string labelId`上

### JsonConstructor 特性

Constructor上的参数必须与属性或字段一样，否则会报错：`Each parameter in the deserialization constructor on type xxx must bind to an object property`
似乎多了或少了都不行？

```csharp
public class Person
{
    public string  Name { get; init; }
    public string  Age { get; init; }
    public string TracingData { get; set; }
    public List<object> ChangingList { get; set; }
    [JsonConstructor]
    public Person(string name, string age, string tracingData, List<object> changingList)
    {
    }
}

var json2 = """
           {
           "tracingData": "123",
           "changingList": [
            "ddd"
           ],
           "name" : "",
           "age": ""
           }
           """;
```
