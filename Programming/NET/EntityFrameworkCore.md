
# 问题
## Update

Update 方法会将传入的实体的状态设置为 Modified，但它只会处理根实体。EF Core 不会自动递归地将所有关联的子实体状态也设置为 Modified。


Update 方法会更改所有实体数据为当前状态，所以一般用于`Disconnected Entity`的设置。
`https://www.learnentityframeworkcore.com/dbcontext/modifying-data`


### DbUpdateConcurrencyException

`The database operation was expected to affect 1 row(s), but actually affected 0 row(s).`

问题可通过EFCore生成的SQL语句进行排查，**可能实际上使用的SQL语句不一定就是真正执行的SQL**（详见Sqlite的问题）。
1. 一般是因为Update操作时，此数据不存在。此数据可能已经被删除或已经被Update而无法匹配上。
也有可能是需要Add的操作，错误的使用了Update方法。
2. 还有就是SQL语句生成了并发检查相关的问题

其实最根本的原因就是生成的Update SQL语句Where条件不匹配，找不到要更新的数据，然后判断EffectRow不一致。

### AbpDbConcurrencyException

ConcurrencyStamp原理是生成SQL语句时带上`ConcurrencyStamp=@old`，然后更新时更新为新的，如果失败证明数据库那边已经被其他修改了（证明版本不一致）。

其他可能：
1. 因为令牌在AbpContext SaveChanges时进行修改，若这次进行保存数据库失败，下次再进行修改，则也会抛出该异常。
2. 一次请求，A微服务需要修改，A调用B，B恰好也去修改状态，这时候A再进行修改则会取出旧令牌匹配。（逻辑上是串行，其实没有问题）
##### 修改令牌
`GetAsync()`查出的实体实例被修改后，然后又重新多次查询相关实例并客户端侧修改，即使没有使用 `Update` 等方法也会导致并发异常。（这里是同事写了个递归函数）
初步判断应该是令牌修改是ABP客户端侧判断，而非交给数据库判断，然后多次查询修改时发现令牌不匹配，直接在客户端侧触发并发修改异常。

只读查询功能似乎要额外设置。具体看 `GetAsync()`设置。

##### 多线程触发
领域事件中UpdateAsync产生AbpDbConcurrencyException问题。最后发现其实就是多线程并发异常。眼光不能局限在某个服务，这次是事件多次触发，Redis拿到旧的数据导致的
`https://sourcegraph.com/github.com/abpframework/abp@4f6426add5b69bfb273f601b1ddd9f1f89099a72/-/blob/framework/src/Volo.Abp.EntityFrameworkCore/Volo/Abp/EntityFrameworkCore/AbpDbContext.cs?L347:17&popover=pinned`
`https://sourcegraph.com/github.com/abpframework/abp@4f6426add5b69bfb273f601b1ddd9f1f89099a72/-/blob/framework/src/Volo.Abp.EntityFrameworkCore/Volo/Abp/EntityFrameworkCore/AbpDbContext.cs?L520:28&popover=pinned`

[处理并发冲突 - EF Core | Microsoft Learn](https://learn.microsoft.com/zh-cn/ef/core/saving/concurrency?tabs=data-annotations)


#### SQLlite相关问题

遇到一个更新用户数据失败问题，随便修改某个字段都会报错。而从EFCore的SQL语句也没法看出问题：
![](../../attachments/d5c19e6587b245290ad303ae6af8e09.png)

SQLite对于GUID字段的存储是TEXT，是大小写敏感的，但是C# GUID对象是大小写不敏感的，日志默认ToString是小写的。又因为EFCore对于的GUID类型生成的SQL是使用大写生成的，所以匹配不上导致更新失败。

> 不要被程序生成的参数列表误导了，这里的参数日志是格式化的程序guid，不是真正的sql参数

相关issues:
[SQLite: Lower-case Guid strings don't work in queries · Issue #19651 · dotnet/efcore](https://github.com/dotnet/efcore/issues/19651)
[Issue with uppercase/lowercase GUID · Issue #25043 · dotnet/efcore](https://github.com/dotnet/efcore/issues/25043)

SQLite解决方案：

```cs
builder.Property(p=>p.Id).HasConversion(new GuidToStringConverter());

```



### A second operation was started on this context instance
同一个依赖注入的类的多个仓储共用一个DbContext（待确认），因此无法同步执行。**注意异步方法的调用，是否都进行了await**。注意入口方法是否是void忘记等待。


#### Cannot access a disposed context instance. A common cause of this error is disposing a context instance that was resolved from dependency injection and then later trying to use the same context instance elsewhere in your application.'
Repository中的DbContext不可以`using`，直接交由ABP框架管理生命周期。
```cs
await using var context = await _repository.GetDbContextAsync(); //导致错误
//直接使用
var context = await _repository.GetDbContextAsync();

```


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

## 外键问题

### 自动生成了Shadow state property

在配置一对多关系的时候，误写成了如下配置：
```cs
  builder.HasOne<Role>().WithMany().HasForeignKey(p => p.GroupId);
```
导致会自动生成`RoleId`列。
应写为：
```cs
  builder.HasOne(p=>p.Role).WithMany().HasForeignKey(p => p.GroupId);
```

## 继承关系
在EF Core中，当实体类之间存在继承关系并使用TPH（Table-Per-Hierarchy）映射策略时，会自动生成Discriminator列。该列用于区分同一表中不同类型的实体，该列的值表示每一行对应的具体实体类型（如基类名或子类名）。
继承关系有多种映射策略，如`Table-Per-Hierarchy`，`Table-Per-Type`等。

如果发现自动生成了Discriminator列，一般是因为将基类和子类添加到了当前DbContext，如`DbSet<BaseEntity>`，或通过`IEntityTypeConfiguration`自动注册进来的实体。



## 更新

### ChangeTracker

ChangeTracker判断更新的原理是在调用ChangeTracker.Entries()（内部调用了ChangeTracker.DetectChanges）时会与Originally值进行对比，如果值不一致才会刷新状态是Modified，否则将还是UnChanged。
只有开启了跟踪才会变为`Unchanged`状态，也就是正在跟踪，此时的状态进行修改属性会记录下`Original`值。否则是为`Detached`状态，不会进行变化。但有其他方式将`Detached`状态转为其他跟踪状态（待补充），如Remove、Update等操作。


在实现CDC时发现删除操作未能成功执行，ChangeTracker发现最后因为软删除置为Unchanged后SaveChanges时会调用一次ChangeTracker.Entries()计算值是否变化， 计算结果为Unchanged。

```cs
   public override async Task DeleteManyAsync(IEnumerable<TEntity> entities, bool autoSave = false, CancellationToken cancellationToken = default)
   {
       var entityArray = entities.ToArray();
       if (entityArray.IsNullOrEmptySet())
       {
           return;
       }
       
       var dbContext = await GetDbContextAsync();

       dbContext.RemoveRange(entityArray.Select(x => x));

       if (autoSave)
       {
           await dbContext.SaveChangesAsync(cancellationToken);
       }
   }
 protected virtual void ApplyConceptsForDeletedEntity(EntityEntry entry)
 {
     if (entry.Entity is not IHasSoftDelete entity)
     {
         return;
     }

     entry.State = EntityState.Unchanged;
     entity.IsDeleted = true;

     //ObjectHelper.TrySetProperty(entry.Entity.As<IHasSoftDelete>(), x => x.IsDeleted, () => true);
     SetDeletionAuditProperties(entry);
 }


```


还有`Attach()`方法可以标记实体为`Unchanged`状态，即认为当前实体已经在数据库存在（`Originally`标记当前值），然后后续修改都可以被跟踪为`Modified`，就仅更新已更新的字段。



# 基础知识

ORM（Object Relational Mapping）框架

注意MySQL数据库不能用MyISAM，需要用InnoDB，不然不支持外键和事务等，发挥不了EF的效果

## 依赖注入

#### DbContext依赖注入

[https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-6.0/whatsnew\#dbcontext-factory-improvements](https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-6.0/whatsnew#dbcontext-factory-improvements)

IDbContextFactory\<SomeDbContext\> contextFactory

这种注入的适合Actor等，需要用 using var context1 = \_contextFactory.CreateDbContext();

注册需要这样：

builder.Services

.AddDbContextFactory\<FlightContext\>(options =\> options.UseMySql(connectionString, version))

.BuildServiceProvider();

简单的可以直接用

builder.Services.AddDbContext\<FlightContext\>(

options =\> options.UseMySql(connectionString, version));

这种在constructor内就直接用FlightContext即可（适用于Controller）

## 配置

### 加载关联数据

#### Lazy loading延迟加载

### 字段配置

By convention, all public properties with a getter and a setter will be included in the model.

默认只会映射含有get、set字段的public属性。

### 模型配置

配置有两种配置方式，一种是使用fluentAPI配置，另一种是对模型使用Attribute。

#### fluentAPI

可在派生上下文中覆写 OnModelCreating 方法，并使用 ModelBuilder API 来配置模型。 此配置方法最为有效，并可在不修改实体类的情况下指定配置。 Fluent API 配置具有最高优先级，并将替代约定和数据注释。

![](../../attachments/10ba1df7b1aa6044b4f0cd0c53941792.png)

#### 数据注释（特性）

也可将特性（称为数据注释）应用于类和属性。 数据注释会替代约定，但会被 Fluent API 配置替代。

![](../../attachments/1626f51f453de92cbe2e3099808270c1.png)

以上两图两者等价，择一配置。

| Fluent API                                                                                              | 数据注释                                                  | 说明                                                                                                                                                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IsRequired()                                                                                            | [Required]                                            |                                                                                                                                                                                                                                                                                       |
| .HasKey(c =\> c.xxxxx)                                                                                  | [Key]                                                 | 此键映射到关系数据库中主键的概念                                                                                                                                                                                                                                                                      |
| .HasKey(c =\> new { c.xxx1, c.xxx2 }                                                                    | 无                                                     | 组合键，只能用fluent api配置。                                                                                                                                                                                                                                                                  |
| .HasPrincipalKey(b =\> b.xxx)                                                                           |                                                       | Principal key：The properties that uniquely identify the principal entity. This may be the primary key or an alternate key.                                                                                                                                                            |
| .HasAlternateKey(c =\> c.xxx);                                                                          |                                                       | 除了主键外，备用键还可用作每个实体实例的替代唯一标识符;它可用作关系的目标。 使用关系数据库时，这将映射到备用键列上的唯一索引/约束和引用列的一个或多个外键约束的概念。支持组合键。                                                                                                                                                                                            |
| HasXXXKey(xxxxx).HasName("xxxx")                                                                        |                                                       | 配置xxxx约束的名称。                                                                                                                                                                                                                                                                          |
| .IsConcurrencyToken()                                                                                   | [ConcurrencyCheck]                                    | 并发标记                                                                                                                                                                                                                                                                                  |
| .HasOne(p =\> p.Blog).WithMany(b =\> b.Posts);                                                          | [InverseProperty(“映射到引用的实体类型的反向导航属性的nameof”)]         | 显式声明反向导航属性（因为有多个同一个类型的导航属性的时候，映射到哪个反向导航属性是二义性的，需要进行显式配置） HasOne、HasMany是指明配置自己实体类上的引用的导航属性（即对方），WithOne、WithMany是指明引用的实体类型的反向导航属性（即自身） 逻辑即：我（Post）有一个对应的Blog（可导航过去），这个Blog有很多Post（通过这个Blog找到Post，即自己，叫反向导航，是相对于这个模型而非某个字段而言的）                                                         |
| .HasMany(b =\> b.Posts).WithOne();                                                                      |                                                       | 只有导航属性，没有反向导航属性；即导航属性那个类没有自身的引用                                                                                                                                                                                                                                                       |
| .HasForeignKey(p=\>p.BlogForeignKey); .HasForeignKey(s =\> new { s.CarState, s.CarLicensePlate });（组合键） | [ForeignKey("BlogForeignKey")]（仅支持简单键）                | 指明当前多对多/一对多/多对一/一对一的关系的导航属性的外键是根据(依赖)哪个(或多个)属性 注意在one-to-one的关系中需要显式配置外键 如果前面使用了只有导航属性没有反向导航属性的，那么HasForeignKey无法自动推断，需要使用HasForeignKey\<T\>的泛型形式                                                                                                                                     |
| HasPrincipalKey(p =\> p.Id);                                                                            |                                                       | 指明当前多对多/一对多/多对一/一对一的关系的主键，与外键相应，即这边的外键映射到那边的主键，本来外键默认对应是那边的Primary Key，但是可以换成这设定的Principal Key。 PS：当该关系所映射的主键与本身表主键设置不一致时才需要指明。                                                                                                                                                       |
| .OnDelete(DeleteBehavior.Cascade);                                                                      |                                                       | 配置级联删除                                                                                                                                                                                                                                                                                |
| .Ignore(b=\>b.LoadedFromDatabase)                                                                       | [NotMapped]                                           | 排除一个属性                                                                                                                                                                                                                                                                                |
| .Property(b=\>b.BlogId).HasColumnName("blog_id");                                                       | [Column("blog_id")]                                   | 默认情况下是映射与字段名一致的属性，不一致要指明列名                                                                                                                                                                                                                                                            |
| .HasColumnType("decimal(5, 2)")                                                                         | [Column(TypeName = "decimal(5, 2)")]                  | 按照数据库的类型方式标注类型                                                                                                                                                                                                                                                                        |
| .ToTable("blogs")                                                                                       | [Table("blogs")]                                      | 指定映射的数据库表名                                                                                                                                                                                                                                                                            |
| .ToTable("blogs", schema: "blogging")                                                                   | [Table("blogs", Schema = "blogging")]                 | 指定映射的数据库表的视图名                                                                                                                                                                                                                                                                         |
| .HasDefaultValue(x)                                                                                     |                                                       | 指定某个属性有默认值                                                                                                                                                                                                                                                                            |
| .HasDefaultValueSql("getdate()")                                                                        |                                                       | 指定某个属性有默认值（用的sql里面的默认值）                                                                                                                                                                                                                                                               |
| .ValueGeneratedOnAddOrUpdate()                                                                          | [DatabaseGenerated(DatabaseGeneratedOption.Computed)] | This just lets EF know that values are generated for added or updated entities, it does not guarantee that EF will setup the actual mechanism to generate values.                                                                                                                     |
| .HasComputedColumnSql("[LastName] + ', ' + [FirstName]")                                                |                                                       | Computed columns In some cases, the column's value is computed every time it is fetched (sometimes called virtual columns), and in others it is computed on every update of the row and stored (sometimes called stored or persisted columns). This varies across database providers. |

术语：

Post.Blog is a reference navigation property（引用导航属性，是一个）

Blog.Posts is a collection navigation property（集合导航属性，是多个）

Post.Blog is the inverse navigation property （反向导航属性）of Blog.Posts (and vice versa 反之亦然) 是两者之间的关系，能相互导航过去

显式指明导航属性，有冲突的情况：

![](../../attachments/e771f44954f4ec97925196d0480dd9f7.png)

![](../../attachments/2fa253323ef8af111b635da5434a0508.png)

### EFCore跟踪修改

//如果直接使用user则会报错，似乎是因为user也是从KouContext中取出来的，ef认为被修改了，没有取消跟踪。

如果一个Model中的外键对象是用的之前从context中取出的模型而被修改，则会报错，可能是因为不能同时修改外键对象又增加Model。只能从context中取出最新的外键对象然后加到Model中，才可绑定。

## 日志排查

optionsBuilder.LogTo(Console.WriteLine);

options.EnableSensitiveDataLogging();

## 注意事项

### 在Context内就SaveChanges

比如blog和posts的关系，首先需要blog.Incloude(p=\>p.posts)

在两个context下进行修改时，会出现问题。

比如在第一个context中

将Blog.Posts = new List\<Post\>();了

然后将这个Blog对象传到另一个Context中，

context.Blog.Update(blog);

这里将不会自动trace Blog原有的post，都会当作新的Post加入到表中。

### Equal重写

重写后的Equal在EFCore的linq to sql中似乎没有用（是使用的设定的候选键对比而不是override之后的），必须转换为Client Evaluation才有效。

EFCore的Single、Update、Delete等等都是通过设定的候选键，而非重写后的Equal。

### Parameterized constructor

Navigator property不会被认为是entity的property，可以使用private constructor，做个无参constructor.

另外如果其他的property无法映射，需要build设置 property 指明它是entity的property.

### Backing Field

Starting with EF Core 3.0, if the backing field for a property is known, then EF Core will always read and write that property using the backing field. This could cause an application break if the application is relying on additional behavior coded into the getter or setter methods.

即，如果有Name这个property且有_name，会自动的找到它的_name这个Backing Field（需要满足条件才可以自动找到，<https://docs.microsoft.com/en-us/ef/core/modeling/backing-field>），然后读写它而不是通过property的get或者set property。所以如果需要property的读写逻辑，则需要

modelBuilder.UsePropertyAccessMode(PropertyAccessMode.PreferFieldDuringConstruction);

By default, EF will always read and write to the backing field - assuming one has been properly configured - and will never use the property. However, EF also supports other access patterns. For example, the following sample instructs EF to write to the backing field only while materializing, and to use the property in all other cases:

即，只在初始化阶段使用field。

## 反向工程

自动根据已设计好的数据库信息生成model类以及dbContext

Scaffold-DbContext -Connection "Server=127.0.0.1;User Id=root;Password=root;Database=kou;" -Provider MySql.Data.EntityFrameworkCore -OutputDir Models/EFTemp -DataAnnotations -Project Koubot.SDK -force -Verbose -Tables system_global_setting,system_alias_list,system_plugin_enable_setting

如果build failed是因为整个解决方案无法build，说明有错误，要解决这些错误然后重新生成解决方案才可以进行反向工程

DataAnnotations 这个是生成自动生成model字段的attribute的

force是覆盖已经存在的文件的

outputDir是生成的文件路径

Provider mysql那个是provider

Verbose 显示详细

Tables 是指定表名（一般更新的时候用）

连接字段是connection

详细见：<https://docs.microsoft.com/zh-cn/ef/core/miscellaneous/cli/powershell>

框架默认具有公共getter和setter的属性会被包括在模型中，可以用NotMapped排除

public class Blog  
{  
 public int BlogId { get; set; }  
 public string Url { get; set; }  
​  
 [NotMapped]  
 public DateTime LoadedFromDatabase { get; set; }  
}

## Migration

| Terminal                                                                                                                           | 操作                                                                                                                     | 解释                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| dotnet ef migrations add \<Alter Operation Name\>                                                                                  | Add-Migration \<Alter Operation Name\>                                                                                   | 每次操作变动需要Add Migration，相当于git中的commit。 -Project Project.Name 来指定Target project，当然可以直接在Default Project中选择。Target project实际上是Migration所在Assembly，默认是在Context所在Assembly下，可以通过DbContextOptionsBuilder中设置MigrationsAssembly，分离Migration到其他项目（Migration所在项目必须是Class Library） 似乎第一次Migration无法识别，需要先在Context项目上生成一次，然后直接复制Migration文件到Migration项目。然后也可以随意更改生成的namespace，下次migrate会自动识别。 默认需要一个启动项目，先获取到Context对象，然后进行模型对比映射，得出变更，进而生成Migration文件。 启动项目是Console或asp.net core等项目，可以通过自定义一个启动入口类，管理启动项目获取到的Context对象是如何构造的：IDesignTimeDbContextFactory\<FlightContext\>  至于terminal中，需要先移动到启动的项目文件夹下，然后使用--project来指定migration项目 |
| dotnet ef database update                                                                                                          | Update-Database                                                                                                          | 操作变动后需要同步到数据库，相当于git中的push                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|                                                                                                                                    | Update-Database [ToSpecificState]                                                                                        | 可以将数据库回滚到特定的Migration状态                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| dotnet ef migrations remove                                                                                                        | Remove-Migration                                                                                                         | 移除最新一次的Add Migration操作                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| dotnet ef migrations script                                                                                                        | Script-Migration                                                                                                         | 需要到生产环境时，使用该命令进行同步修改                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| dotnet ef migrations script AddNewTables AddAuditTable                                                                             | Script-Migration [AddNewTables] [AddAuditTable]                                                                          | 生成从指定migration状态到指定migration状态的修改SQL语句                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| dotnet ef migrations list                                                                                                          | Get-Migration                                                                                                            | list all existing migrations                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| dotnet ef dbcontext scaffold "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=Chinook" Microsoft.EntityFrameworkCore.SqlServer | Scaffold-DbContext 'Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=Chinook' Microsoft.EntityFrameworkCore.SqlServer | Reverse Engineering 反向工程 DB First  -Tables Artist, Album可以指定仅反向给定表名  -Force 需要重新进行反向工程  -Context 指定Context                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
