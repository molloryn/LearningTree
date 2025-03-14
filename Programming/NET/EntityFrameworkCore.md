
# Update

Update 方法会将传入的实体的状态设置为 Modified，但它只会处理根实体。EF Core 不会自动递归地将所有关联的子实体状态也设置为 Modified。


Update 方法会更改所有实体数据为当前状态，所以一般用于`Disconnected Entity`的设置。
`https://www.learnentityframeworkcore.com/dbcontext/modifying-data`

# 问题

## ABP仓储层

### UpdateManyAsync

如果开启跟踪，UpdateMany不论怎么传入都将将所有改变的实体进行保存。

```cs
var list = repo.GetQueryableAsync(); //.. where .. ToList(); 假设返回100个实体
list.Foreach(p=>p.Name = "XX");
repo.UpdateManyAsync(list.Take(20));
```

其中，80个实体将采用如下
```sql
- 其中80个
Update XX SET Name = "XX"

- 其中20个是完整的语句
Update Column1 ... SET Column1...

```

ABP的UpdateMany的实现是通过
```cs
dbContext.Set<TEntity>().UpdateRange(); 
```
批量设置Entity的State为`Modified`。性能较更改跟踪可能更慢。

### GetDbContextAsync
在同一个上下文获取出来的似乎是同一个DbContext
所以SaveChanges也可以有效。如上面的例子
```cs
var list = repo.GetQueryableAsync(); //.. where .. ToList(); 假设返回100个实体
list.Foreach(p=>p.Name = "XX");
var context = repo.GetDbContextAsync();
context.SaveChanges(); //可以成功保存。
```
