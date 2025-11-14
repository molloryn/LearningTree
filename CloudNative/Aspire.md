# Aspire

## 问题排查

### 启动出现Requesting HTTP version 2.0 

一般是由于电脑使用了环境变量 `HTTP_PROXY` 以及 `HTTPS_PROXY` 导致的，`.NET HTTP Client`会使用此环境变量作为代理。Alternatively, set NO_PROXY="localhost,127.0.0.1" in environment. Restart the app.

> 清除环境变量后需要重启IDE生效

```log
fail: Aspire.Hosting.Dashboard.ServiceClient.DashboardClient[0]
      Error #6 watching resources.
      Grpc.Core.RpcException: Status(StatusCode="Internal", Detail="Error starting gRPC call. HttpRequestException: Requesting HTTP version 2.0 with version policy RequestVersionExact while unable to establish HTTP/2 connection.", DebugException="System.Net.Http.HttpRequestException: Requesting HTTP version 2.0 with version policy RequestVersionExact while unable to establish HTTP/2 connection.")
       ---> System.Net.Http.HttpRequestException: Requesting HTTP version 2.0 with version policy RequestVersionExact while unable to establish HTTP/2 connection.
         at System.Net.Http.HttpConnectionPool.ThrowGetVersionException(HttpRequestMessage request, Int32 desiredVersion, Exception inner)
         at System.Net.Http.HttpConnectionPool.SendWithVersionDetectionAndRetryAsync(HttpRequestMessage request, Boolean async, Boolean doRequestAuth, CancellationToken cancellationToken)
         at System.Net.Http.DiagnosticsHandler.SendAsyncCore(HttpRequestMessage request, Boolean async, CancellationToken cancellationToken)
         at System.Net.Http.RedirectHandler.SendAsync(HttpRequestMessage request, Boolean async, CancellationToken cancellationToken)
         at Grpc.Net.Client.Balancer.Internal.BalancerHttpHandler.SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
         at Grpc.Net.Client.Internal.GrpcCall`2.RunCall(HttpRequestMessage request, Nullable`1 timeout)
         --- End of inner exception stack trace ---
         at Grpc.Net.Client.Internal.GrpcCall`2.GetResponseHeadersCoreAsync()
         at Grpc.Net.Client.Internal.Retry.RetryCallBase`2.GetResponseHeadersCoreAsync()
         at Aspire.Dashboard.ServiceClient.DashboardClient.IsUnimplemented(AsyncDuplexStreamingCall`2 call) in /_/src/Aspire.Dashboard/ServiceClient/DashboardClient.cs:line 511
         at Aspire.Dashboard.ServiceClient.DashboardClient.WatchInteractionsAsync(RetryContext retryContext, CancellationToken cancellationToken) in /_/src/Aspire.Dashboard/ServiceClient/DashboardClient.cs:line 437
         at Aspire.Dashboard.ServiceClient.DashboardClient.WatchWithRecoveryAsync(Func`3 action, CancellationToken cancellationToken) in /_/src/Aspire.Dashboard/ServiceClient/DashboardClient.cs:line 321
```