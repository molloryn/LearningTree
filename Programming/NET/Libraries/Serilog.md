# Serilog


## 基本概念

上半部分是 Microsoft Logging系统，通过配置 `builder.Host.UseSerilog(Log.Logger);` 可以设置为 `Serilog Logging Provider`。设置后，原来的日志配置将失效，转而使用 `Serilog` 的日志配置。其中支持的日志等级与内置等级稍微不同。
```json
 "Logging": {
   "LogLevel": {
     "Default": "Information",
     "Microsoft.AspNetCore": "Information",
     "Microsoft": "Debug",
     "Microsoft.Hosting.Lifetime": "Information"
   }
 },
 "Serilog": {
   "MinimumLevel": {
     "Default": "Information",
     "Override": {
       "System": "Information",
       "Microsoft.AspNetCore": "Information",
       "Microsoft.EntityFrameworkCore": "Information",
       "Microsoft.Hosting.Lifetime": "Information",
       "Microsoft.EntityFrameworkCore.Database.Transaction": "Debug"
     }
   }
 }

```



## 问题排查

### Serilog死锁 写大日志会出现该问题
[Console logging sometimes causes application to hang · Issue #84 · serilog/serilog-sinks-console](https://github.com/serilog/serilog-sinks-console/issues/84)

### 二次构建写不出日志

两次建立需要手动关闭Serilog的Sink，否则如文件形式的将会被占用导致无日志输出。

```cs
    internal static void RegisterSharedLog(WebApplicationBuilder builder, UnifiedServiceConfig config, bool isFirstCreate = false)
    {
        var logBuilder = new LoggerConfiguration()
            .ReadFrom.Configuration(builder.Configuration);
        if (!isFirstCreate)
        {
            Log.CloseAndFlush(); //巨坑：两次建立需要手动关闭Serilog的Sink，否则如文件形式的将会被占用导致无日志输出。
        }

#if DEBUG
        Log.Logger = CreateLogger(config, logBuilder, true, isFirstCreate);
#else
            Log.Logger = CreateLogger(config, logBuilder, false, isFirstCreate);
#endif
    }
```




## 其他问题

### 禁用日志

Serilog 不支持 Microsoft 的 Logging 中的 None 的日志等级，暂时没法禁用日志，只能调整日志等级。 [Support for LogEventLevel.HigherThanFatal · Issue #1049 · serilog/serilog](https://github.com/serilog/serilog/issues/1049)

默认情况下，`UseExceptionHandler`中间件包含了一个默认的异常处理中间件，会打印异常信息到控制台。如若要禁用此功能，需要在日志配置中设置

```json
"Microsoft.AspNetCore.Diagnostics.ExceptionHandlerMiddleware": "None"
```

