# System.Text.Json


## TroubleShooting

### 未成功序列化
`SystemText`.`Json`默认行为是只反序列化到`public`属性，对于非`public`的属性的需要加`JsonInclude`标签才会设置。


### 序列化值为{}
默认情况下是序列化当前类型字段，如果用`this`关键字要小心。

```cs

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
#### 类型不一致

比如`labelId:100000` 就无法序列化到 `string labelId`上
### JsonConstructor 特性

Constructor上的参数必须与属性或字段一样，否则会报错：`Each parameter in the deserialization constructor on type xxx must bind to an object property`
似乎多了或少了都不行？

```cs

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
