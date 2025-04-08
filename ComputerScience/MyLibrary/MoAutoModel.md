# 目标
1. 面向接口，支持依赖注入
2. 开箱即用
3. 无需耦合实体代码

# 设计


#### 自定义查询方法

#### 字段与值关系

#### 字段与值表达式生成

#### 自定义字段Query组装器


```cs
CustomFor(nameof(Entity.FieldName), (input, querable))
```
#### 前端请求类

可以以传统方式编写请求类
```cs


```


#### 值转换器

`Name in "AB%|%CD%|EF"`
`CreateTime in "[2023-1-1,2024-1-1]"`
`CreateTime >= "2023-1-1" && CreateTime <= "2024-1-1"`
`Name = "ABC"`

高阶：
`CreateTime in "[Today,Tomorrow]"`
`UpdateTime in "[EOBT,EOBT+30min]"`

