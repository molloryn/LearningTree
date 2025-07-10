# AsyncLocal



## AsyncLocal 用法简介

> 文章来源：[.NET AsyncLocal 避坑指南 - 黑洞视界 - 博客园](https://www.cnblogs.com/eventhorizon/p/17170301.html)

通过 AsyncLocal 我们可以在一个逻辑上下文中维护一份私有数据，该上下文后续代码中都可以访问和修改这份数据，但另一个无关的上下文是无法访问的。

无论是在新创建的 Task 中还是 await 关键词之后，我们都能够访问前面设置的 AsyncLocal 的数据。

```C#
class Program
{
    private static AsyncLocal<string> _asyncLocal = new AsyncLocal<string>();
    
    static async Task Main(string[] args)
    {
        _asyncLocal.Value = "Hello World!";
        Task.Run(() => Console.WriteLine($"AsyncLocal in task: {_asyncLocal.Value}"));

        await FooAsync();
        Console.WriteLine($"AsyncLocal after await FooAsync: {_asyncLocal.Value}");
    }

    private static async Task FooAsync()
    {
        await Task.Delay(100);
        Console.WriteLine($"AsyncLocal after await in FooAsync: {_asyncLocal.Value}");
    }
}
```

输出结果：

> ```Log
> AsyncLocal in task: Hello World!
> AsyncLocal after await in FooAsync: Hello World!
> AsyncLocal after await FooAsync: Hello World!
> ```

# AsyncLocal 实现原理

在我之前的博客 [揭秘 .NET 中的 AsyncLocal](https://mp.weixin.qq.com/s?__biz=MzkyOTQyMzg2OA==&mid=2247483796&idx=1&sn=7bed5f6da13431610f2ec4c7e63485f9&chksm=c2088907f57f00116cd084da5c08b9761c8911861e77df9912d8172472612a05cf3e40ba3498&token=1405093735&lang=zh_CN#rd) 中深入介绍了 AsyncLocal 的实现原理，这里只做简单的回顾。

AsyncLocal 的实际数据存储在 ExecutionContext 中，而 ExecutionContext 作为线程的私有字段与线程绑定，在线程会发生切换的地方，runtime 会将切换前的 ExecutionContext 保存起来，切换后再恢复到新线程上。

这个保存和恢复的过程是由 runtime 自动完成的，例如会发生在以下几个地方：

- new Thread(ThreadStart start).Start()
- Task.Run(Action action)
- ThreadPool.QueueUserWorkItem(WaitCallback callBack)
- await 之后

以 await 为例，当我们在一个方法中使用了 await 关键词，编译器会将这个方法编译成一个状态机，这个状态机会在 await 之前和之后分别保存和恢复 ExecutionContext。

```C#
class Program
{
    private static AsyncLocal<string> _asyncLocal = new AsyncLocal<string>();
    
    static async Task Main(string[] args)
    {
        _asyncLocal.Value = "Hello World!";
        await FooAsync();
        Console.WriteLine($"AsyncLocal after await FooAsync: {_asyncLocal.Value}");
    }

    private static async Task FooAsync()
    {
        await Task.Delay(100);
    }
}
```

输出结果：

> ```Log
> AsyncLocal after await FooAsync: Hello World!
> ```

![](../../../attachments/Pasted%20image%2020250708105102.png)

# AsyncLocal 的坑

有时候我们会在 FooAsync 方法中去修改 AsyncLocal 的值，并希望在 Main 方法在 await FooAsync 之后能够获取到修改后的值，但是实际上这是不可能的。

```C#
class Program
{
    private static AsyncLocal<string> _asyncLocal = new AsyncLocal<string>();
    
    static async Task Main(string[] args)
    {
        _asyncLocal.Value = "A";
        Console.WriteLine($"AsyncLocal before FooAsync: {_asyncLocal.Value}");
        await FooAsync();
        Console.WriteLine($"AsyncLocal after await FooAsync: {_asyncLocal.Value}");
    }

    private static async Task FooAsync()
    {
        _asyncLocal.Value = "B";
        Console.WriteLine($"AsyncLocal before await in FooAsync: {_asyncLocal.Value}");
        await Task.Delay(100);
        Console.WriteLine($"AsyncLocal after await in FooAsync: {_asyncLocal.Value}");
    }
}
```

输出结果：

> ```Log
> AsyncLocal before FooAsync: A
> AsyncLocal before await in FooAsync: B
> AsyncLocal after await in FooAsync: B
> AsyncLocal after await FooAsync: A
> ```

为什么我们在 FooAsync 方法中修改了 AsyncLocal 的值，但是在 await FooAsync 之后，AsyncLocal 的值却没有被修改呢？

原因是 ExecutionContext 被设计成了一个不可变的对象，当我们在 FooAsync 方法中修改了 AsyncLocal 的值，实际上是创建了一个新的 ExecutionContext，原来其他的 AsyncLocal 的值被值拷贝到了新的 ExecutionContext 中，新的 AsyncLocal 的值只会写入到新的 ExecutionContext 中，而原来的 ExecutionContext 及其关联的 AsyncLocal 仍然保持不变。

这样的设计是为了保证线程的安全性，因为在多线程环境下，如果 ExecutionContext 是可变的，那么在切换线程的时候，可能会出现数据不一致的情况。

我们通常把这种设计称为 Copy On Write（简称COW），即在修改数据的时候，会先拷贝一份数据，然后在拷贝的数据上进行修改，这样就不会影响到原来的数据。

**ExecutionContext 中可能不止一个 AsyncLocal 的数据，修改任意一个 AsyncLocal 都会导致 ExecutionContext 的 COW。**

所以上面代码的执行过程如下：  
![](../../../attachments/Pasted%20image%2020250708105117.png)


# AsyncLocal 的避坑指南

那么我们如何在 FooAsync 方法中修改 AsyncLocal 的值，并且在 Main 方法中获取到修改后的值呢？

我们需要借助一个中介者，让中介者来保存 AsyncLocal 的值，然后在 FooAsync 方法中修改中介者的属性值，这样就可以在 Main 方法中获取到修改后的值了。

下面我们设计一个 ValueHolder 来保存 AsyncLocal 的值，修改 Value 并不会修改 AsyncLocal 的值，而是修改 ValueHolder 的属性值，这样就不会触发 ExecutionContext 的 COW。

我们还需要设计一个 ValueAccessor 来封装 ValueHolder 对值的访问和修改，这样可以保证 ValueHolder 的值只能在 ValueAccessor 中被修改。

```C#
class ValueAccessor<T> : IValueAccessor<T>
{
    private static AsyncLocal<ValueHolder<T>> _asyncLocal = new AsyncLocal<ValueHolder<T>>();

    public T Value
    {
        get => _asyncLocal.Value != null ? _asyncLocal.Value.Value : default;
        set
        {
            _asyncLocal.Value ??= new ValueHolder<T>();

            _asyncLocal.Value.Value = value;
        }
    }
}

class ValueHolder<T>
{
    public T Value { get; set; }
}

class Program
{
    private static IValueAccessor<string> _valueAccessor = new ValueAccessor<string>();

    static async Task Main(string[] args)
    {
        _valueAccessor.Value = "A";
        Console.WriteLine($"ValueAccessor before await FooAsync in Main: {_valueAccessor.Value}");
        await FooAsync();
        Console.WriteLine($"ValueAccessor after await FooAsync in Main: {_valueAccessor.Value}");
    }

    private static async Task FooAsync()
    {
        _valueAccessor.Value = "B";
        Console.WriteLine($"ValueAccessor before await in FooAsync: {_valueAccessor.Value}");
        await Task.Delay(100);
        Console.WriteLine($"ValueAccessor after await in FooAsync: {_valueAccessor.Value}");
    }
}
```

输出结果：

> ```Log
> ValueAccessor before await FooAsync in Main: A
> ValueAccessor before await in FooAsync: B
> ValueAccessor after await in FooAsync: B
> ValueAccessor after await FooAsync in Main: B
> ```

# HttpContextAccessor 的实现原理

我们常用的 `HttpContextAccessor` 通过`HttpContextHolder` 来间接地在 `AsyncLocal` 中存储 `HttpContext。`

如果要更新 HttpContext，只需要在 HttpContextHolder 中更新即可。因为 AsyncLocal 的值不会被修改，更新 HttpContext 时 ExecutionContext 也不会出现 COW 的情况。

不过 HttpContextAccessor 中的逻辑有点特殊，它的 HttpContextHolder 是为保证清除 HttpContext 时，这个 HttpContext 能在所有引用它的 ExecutionContext 中被清除（可能因为修改 HttpContextHolder 之外的 AsyncLocal 数据导致 ExecutionContext 已经 COW 很多次了）。

下面是 HttpContextAccessor 的实现，英文注释是原文，中文注释是我自己的理解。

```C#
/// </summary>
public class HttpContextAccessor : IHttpContextAccessor
{
    private static readonly AsyncLocal<HttpContextHolder> _httpContextCurrent = new AsyncLocal<HttpContextHolder>();

    /// <inheritdoc/>
    public HttpContext? HttpContext
    {
        get
        {
            return _httpContextCurrent.Value?.Context;
        }
        set
        {
            var holder = _httpContextCurrent.Value;
            if (holder != null)
            {
                // Clear current HttpContext trapped in the AsyncLocals, as its done.
                // 这边的逻辑是为了保证清除 HttpContext 时，这个 HttpContext 能在所有引用它的 ExecutionContext 中被清除
                holder.Context = null;
            }

            if (value != null)
            {
                // Use an object indirection to hold the HttpContext in the AsyncLocal,
                // so it can be cleared in all ExecutionContexts when its cleared.
                // 这边直接修改了 AsyncLocal 的值，所以会导致 ExecutionContext 的 COW。新的 HttpContext 不会被传递到原先的 ExecutionContext 中。
                _httpContextCurrent.Value = new HttpContextHolder { Context = value };
            }
        }
    }

    private sealed class HttpContextHolder
    {
        public HttpContext? Context;
    }
}
```

但 HttpContextAccessor 的实现并不允许将新赋值的非 null 的 HttpContext 传递到外层的 ExecutionContext 中，可以参考上面的源码及注释理解。

```C#
class Program
{
    private static IHttpContextAccessor _httpContextAccessor = new HttpContextAccessor();
    
    static async Task Main(string[] args)
    {
        var httpContext = new DefaultHttpContext
        {
            Items = new Dictionary<object, object>
            {
                { "Name", "A"}
            }
        };
        _httpContextAccessor.HttpContext = httpContext;
        Console.WriteLine($"HttpContext before await FooAsync in Main: {_httpContextAccessor.HttpContext.Items["Name"]}");
        await FooAsync();
        // HttpContext 被清空了，下面这行输出 null
        Console.WriteLine($"HttpContext after await FooAsync in Main: {_httpContextAccessor.HttpContext?.Items["Name"]}");
    }

    private static async Task FooAsync()
    {
        _httpContextAccessor.HttpContext = new DefaultHttpContext
        {
            Items = new Dictionary<object, object>
            {
                { "Name", "B"}
            }
        };
        Console.WriteLine($"HttpContext before await in FooAsync: {_httpContextAccessor.HttpContext.Items["Name"]}");
        await Task.Delay(1000);
        Console.WriteLine($"HttpContext after await in FooAsync: {_httpContextAccessor.HttpContext.Items["Name"]}");
    }
}
```

输出结果：

> ```Log
> HttpContext before await FooAsync in Main: A
> HttpContext before await in FooAsync: B
> HttpContext after await in FooAsync: B
> HttpContext after await FooAsync in Main: 
> ```



## 阻止上下文流动

在一般情况下，当前上下文无论是自己创建Task、Thread，都会使得AsyncLocal捕获到开启这些线程的上下文，如果特殊情况想要在并发操作时规避掉获取上下文（如ABP UOW），可以有如下方案：

### ExecutionContext.SuppressFlow

- 在创建新线程前调用，阻止当前线程的上下文（包括 `AsyncLocal`）流向新线程。
- 操作后通过 `new Thread()` 创建的新线程会获得干净的上下文。


```cs
 static void StartNewThread()
    {
        ExecutionContext.SuppressFlow(); // 🔥 阻止上下文流动
        var thread = new Thread(() =>
        {
            Console.WriteLine($"[Thread] ID: {Thread.CurrentThread.ManagedThreadId}, Value: {asyncLocal.Value ?? "null"}");
        });
        
        thread.Start();
        ExecutionContext.RestoreFlow(); // 恢复上下文流动
    }
```
