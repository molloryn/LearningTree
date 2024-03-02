Table of Contents

[Visual Studio](#visual-studio)

[CS0104 but don't have relative ambiguous definition actually](#cs0104-but-dont-have-relative-ambiguous-definition-actually)

[Predefined type 'System.Object' is not defined or imported](#_Toc137395877)

[error msb3030 could not copy the file docker-compose](#_Toc137395878)

[**IIS**](#_Toc137395879)

[401未授权:由于凭据无效,访问被拒绝解决](#401未授权由于凭据无效访问被拒绝解决)

[EFCore](#efcore)

[发现增加条目时某个字段未被算上](#发现增加条目时某个字段未被算上)

[System.InvalidOperationException: The property 'Flight.Id' has a temporary value while attempting to change the entity's state to 'Modified'. Either set a permanent value explicitly, or ensure that the database is configured to generate values for this property.](#systeminvalidoperationexception-the-property-flightid-has-a-temporary-value-while-attempting-to-change-the-entitys-state-to-modified-either-set-a-permanent-value-explicitly-or-ensure-that-the-database-is-configured-to-generate-values-for-this-property)

[C\#](#c)

[开发](#开发)

[System.InvalidOperationException: 'Sequence contains no elements'](#systeminvalidoperationexception-sequence-contains-no-elements)

[ASP.NET Core](#aspnet-core)

[gRPC](#grpc)

[InvalidOperationException: Unable to get subchannel from HttpRequestMessage."](#invalidoperationexception-unable-to-get-subchannel-from-httprequestmessage)

[部署](#部署)

[Unhandled exception. System.IO.DirectoryNotFoundException: C:\\Users\\xxx\\.nuget\\packages\\mudblazor\\6.2.0\\staticwebassets\\](#unhandled-exception-systemiodirectorynotfoundexception-cusersxxxnugetpackagesmudblazor620staticwebassets)

[执行dotnet dll无反应，且dotnet高占用](#执行dotnet-dll无反应且dotnet高占用)

[**Docker**](#_Toc137395893)

[Can not find the container with the name starting with xxxx](#can-not-find-the-container-with-the-name-starting-with-xxxx)

[docker.errors.DockerException: Error while fetching server API version: ('Connection aborted.', FileNotFoundError(2, 'No such file or directory'))](#dockererrorsdockerexception-error-while-fetching-server-api-version-connection-aborted-filenotfounderror2-no-such-file-or-directory)

[bind: An attempt was made to access a socket in a way forbidden by its access permissions](#bind-an-attempt-was-made-to-access-a-socket-in-a-way-forbidden-by-its-access-permissions)

[win10 docker启动后检查版本报错：](#win10-docker启动后检查版本报错)

[Failed to get D-Bus connection: Operation not permitted](#failed-to-get-d-bus-connection-operation-not-permitted)

[Cannot connect to the Docker daemon at unix:/var/run/docker.sock. Is the docker daemon running?](#cannot-connect-to-the-docker-daemon-at-unixvarrundockersock-is-the-docker-daemon-running)

[Docker service start failed. Failed to start Docker Application Container Engine. Unit docker.service entered failed state.](#docker-service-start-failed-failed-to-start-docker-application-container-engine-unit-dockerservice-entered-failed-state)

[部署](#部署-1)

[构建镜像之后，运行脚本文件时，提示无法找到相应sh文件(not such file or directory)，比如/docker-entrypoint.sh](#构建镜像之后运行脚本文件时提示无法找到相应sh文件not-such-file-or-directory比如docker-entrypointsh)

[Can not find docker container with the name starting with xxx](#can-not-find-docker-container-with-the-name-starting-with-xxx)

[Unable to find the target operating system for the project](#unable-to-find-the-target-operating-system-for-the-project)

[Docker - failed to compute cache key: not found - runs fine in Visual Studio](#docker---failed-to-compute-cache-key-not-found---runs-fine-in-visual-studio)

[No packages exist with this id in source(s): nuget.org](#no-packages-exist-with-this-id-in-sources-nugetorg)

[K8S](#k8s)

[IPVS模式下externalIP导致节点故障浅析](#ipvs模式下externalip导致节点故障浅析)

[前端](#前端)

[NPM](#npm)

[**Error: getaddrinfo ENOTFOUND 127.0.0.1:41315**](#_Toc137395911)

[Dapr](#dapr)

[部署](#部署-2)

[no pubsub configured](#no-pubsub-configured)

[开发](#开发-1)

[Type 'FlightProtocol.Models.Flight' cannot be serialized. Consider marking it with the DataContractAttribute attribute, and marking all of its members you want serialized with the DataMemberAttribute attribute. Alternatively, you can ensure that the type is public and has a parameterless constructor - all public members of the type will then be serialized, and no attributes will be required.](#type-flightprotocolmodelsflight-cannot-be-serialized-consider-marking-it-with-the-datacontractattribute-attribute-and-marking-all-of-its-members-you-want-serialized-with-the-datamemberattribute-attribute-alternatively-you-can-ensure-that-the-type-is-public-and-has-a-parameterless-constructor---all-public-members-of-the-type-will-then-be-serialized-and-no-attributes-will-be-required)

[The state store is not configured to use the actor runtime. Have you set the - name: actorStateStore value: "true" in your state store component file?](#the-state-store-is-not-configured-to-use-the-actor-runtime-have-you-set-the---name-actorstatestore-value-true-in-your-state-store-component-file)

# Visual Studio

### Error CS0006 Metadata file 'xxx\\xxx.dll' could not be found

①该项目所依赖的项目build失败。

②如果该项目是采用跨解决方案的项目引用方式，则需要在那个大解决方案下build后才能读取到它所依赖的dll

### NuGet Error NU1105 If you are using Visual Studio, this may be because the project is unloaded or not part of the current solution so run a restore from the command-line. Otherwise, the project file may be invalid or missing targets required for restore.

执行dotnet restore MySolution.sln

### CS0104 but don't have relative ambiguous definition actually

产生了ambiguous definition of a symbol or identifier in the code，但是代码编译器里面却没提示。

很可能是因为当前的code的版本比这个项目引用的dll更加新，换言之就是要build一下产生错误的那个dll。

### Predefined type 'System.Object' is not defined or imported

Close the visual studio. Delete the .vs folder and restart visual studio. Everything should work fine.

Usually this happens when you change branches or merge branches.

### error msb3030 could not copy the file docker-compose

Set Project Build Order in Visual Studio

将docker-compose的依赖设定一下，让它靠后build。

![Graphical user interface, text, application Description automatically generated](../attachments/4bb0282ee3c8083f962fdfd27f529c1f.png)

**IIS**

### 401未授权:由于凭据无效,访问被拒绝解决

<http://www.winwin7.com/JC/12013.html>

一般是匿名身份认证里面设定的用户可能不存在或没有加到正确组中，或组没有访问那个文件夹的权限之类。

# EFCore

### 发现增加条目时某个字段未被算上

①property是{get;}只读字段，它不算上。可以加上private set;

### System.InvalidOperationException: The property 'Flight.Id' has a temporary value while attempting to change the entity's state to 'Modified'. Either set a permanent value explicitly, or ensure that the database is configured to generate values for this property.

可能Id是0，然后调用的是Attach、Update方法，导致无法匹配上。

# C\#

## 开发

### System.InvalidOperationException: 'Sequence contains no elements'

First()

Single()

Last()

LastAsync()

Max()

Min()

Average()

Aggregate()

如果Where后没有符合的条件，这些方法都会引起这个异常。可以使用.DefaultIfEmpty()去规避。如果有类似SingleOrDefault()的也行。注意引用类型的DefaultIfEmpty是null，所以用Min之类的要处理null的情况。

# ASP.NET Core

可以先在appsettings.json中开启日志

"Logging": {

"LogLevel": {

"Default": "Information",

"Microsoft.AspNetCore": "Information",

"Microsoft": "Debug",

"Microsoft.Hosting.Lifetime": "Information"

}

},

## gRPC

### Code: Unimplemented, Message: Service is unimplemented

dapr会调用

dapr.proto.runtime.v1.AppCallback/ListTopicSubscriptions

发现是因为如果采用了gRPC，则不能用Topic特性标签那套（那套HTTP专用）了，得自己重新实现gRPC版的回调实现。

postman等会调用

grpc.reflection.v1alpha.ServerReflection/ServerReflectionInfo

方法获取gRPC的方法信息。

这种如果使用的是protobuf-net的，需要用

builder.Services.AddCodeFirstGrpcReflection();

app.MapCodeFirstGrpcReflectionService();

这俩帮你写好的回调方法注册进去，就支持了。

还有就是自己的方法，可以先用postman测一下支持的grpc方法。

如果没发现自己定义的，就需要检查定义

（protobuf-net）

比如：

① 请求响应类是否打上了ProtoContract标签

② service接口是否打上了ServiceContract标签

③ 请求响应类里面是否有类型不支持：

截至发现目前不支持的类型：

1.DateOnly、TimeOnly

2.Computed property

所以DTO类尽量用最简单的类型来表示。

### Connection closed before server preface received

检查dapr的app-protocol配置

检查是否server是开启TLS，dapr似乎默认不是走tls？

检查server是否默认使用**HTTP2协议**

### InvalidOperationException: Unable to get subchannel from HttpRequestMessage."

Grpc.Core.RpcException: Status(StatusCode="Internal", Detail="Error starting gRPC call. HttpRequestException: Unable to get subchannel from HttpRequestMessage. (127.0.0.1:41315) InvalidOperationException: Unable to get subchannel from HttpRequestMessage.", DebugException="System.Net.Http.HttpRequestException: Unable to get subchannel from HttpRequestMessage. (127.0.0.1:41315)")

After further investigation, this appears related to proxy settings on my machine related to my company's network setup. Specifically, I have the following environment variables defined:

http_proxy https_proxy

.NET populates the HttpClient DefaultProxy from these environment variables. My company proxy appears to be interfering with HTTP/2 (unsupported?), preventing gRPC from working correctly. The workaround for local development is to manually set the default proxy for the HttpClient before making the gRPC call:

HttpClient.DefaultProxy = new WebProxy();

using var channel = GrpcChannel.ForAddress(@"http://localhost:5000");

var client = new AlgorithmRunner.AlgorithmRunnerClient(channel);

using var call = client.RunAlgorithm(new RunAlgorithmRequest());

## 部署

### Unhandled exception. System.IO.DirectoryNotFoundException: C:\\Users\\xxx\\.nuget\\packages\\mudblazor\\6.2.0\\staticwebassets\\

不能从运行过的本地环境直接拷贝到生产，就有这样一个文件：staticwebassets.runtime.json

使得运行出现问题。

生产上的这个文件要删除，然后重新运行。

### 执行dotnet dll无反应，且dotnet高占用

set COREHOST_TRACE=1 environment variable and capture the stderr output (dotnet web.dll 2\>trace.log) 查看输出日志

最终发现原因是因为程序执行报错问题？未能catch异常导致

又发现一个原因，ASP.NET Core的最后没有使用app.Run()运行！！！

**Docker**

### Can not find the container with the name starting with xxxx

容器没有创建。

去Build-\>Configuration Manager 查看, docker-compose 项目的Build是否打勾，需要打上勾。

打上勾后，直接F5运行Docker Compose，会构建容器。

重新Build不会构建容器，只会删除容器，在启动时构建。

如果使用ReSharperBuild，也会出现该问题，如果要使用DockerCompose功能，则不能使用Resharper Build。

### docker.errors.DockerException: Error while fetching server API version: ('Connection aborted.', FileNotFoundError(2, 'No such file or directory'))

原因是 docker 没有启动

开启docker: systemctl start docker

查看docker 进程 ps -ef \| grep docker

最后执行docker-compose up -d

### bind: An attempt was made to access a socket in a way forbidden by its access permissions

端口占用问题。Hyper-V 会保留部分tcp端口，开始到结束范围内的端口不可用, 使用如下命令查看保留的端口：

netsh interface ipv4 show excludedportrange protocol=tcp

可以用如下方式临时解决：

net stop winnat

net start winnat

### win10 docker启动后检查版本报错：

Docker.Core.Backend.BackendException:

Error response from daemon: open \\.\\pipe\\docker_engine_linux: The system cannot find the file specified.

在win10 命令行提示符执行：

Net stop com.docker.service

Net start com.docker.service

### Failed to get D-Bus connection: Operation not permitted

centos无法使用systemctl等，需要创建容器的时候，使用特权模式。docker run -d -it --privileged {Image ID} /usr/sbin/init （需要使用-d在后台，然后用exec -it /bin/bash进入）

### Cannot connect to the Docker daemon at unix:/var/run/docker.sock. Is the docker daemon running?

sudo su

systemctl start docker

systemctl enable docker

systemctl restart docker

### Docker service start failed. Failed to start Docker Application Container Engine. Unit docker.service entered failed state.

如果有/etc/docker/daemon.json，则删除

然后直接运行命令：dockerd

如果提示Error starting daemon:... invalid cross-device link

直接运行rm -rf /var/lib/docker/runtimes 删除这个目录，然后再运行dockerd

## 部署

### 构建镜像之后，运行脚本文件时，提示无法找到相应sh文件(not such file or directory)，比如/docker-entrypoint.sh

**是个超级大坑，必须要检查Copy到镜像的脚本文件换行回车编码格式是否是LF，而不是CRLF！不然会产生各种谜之错误。**

### Can not find docker container with the name starting with xxx

This error started happening to me when someone on the team unchecked the build checkbox for the docker-compose project. Make sure it is enabled in Build -\> Configuration Manager

![A screenshot of a computer program Description automatically generated with medium confidence](../attachments/dd10bcb360f6e2272972995ec5ebf9c3.png)

### Unable to find the target operating system for the project

重新用其他配置运行一下，然后再回到docker配置运行，就发现可以了（迷惑）

### Docker - failed to compute cache key: not found - runs fine in Visual Studio

.net core 及5以上 需要在.sln文件夹下（或在项目文件夹下使用docker build .. -f Dockerfile）使用docker build .，然后用-f指定Dockerfile位置。

.net framework是在项目目录下

### No packages exist with this id in source(s): nuget.org

<https://stackoverflow.com/questions/48821991/dockerfile-cant-see-local-file-or-private-nuget-server>

Copying the Nuget.Config to the solution or project folder will work if your 'private nuget feed' is accessible via a url. But if the private feed is a local folder used as a nuget source, this approach will still fail with an error that the folder is outside the build context or simply because the Windows path does not get resolved by the Docker build process.

(但最后仍然失败了)

# K8S

### IPVS模式下externalIP导致节点故障浅析

<https://blog.csdn.net/qq_41586875/article/details/124330823>

k8s集群一旦将svc中的externalIP设置成集群内任何一个节点IP，就会导致calico、kubelet、kube-proxy等组件无法与apiserver进行通信

# 前端

## NPM

**Error: getaddrinfo ENOTFOUND 127.0.0.1:41315**

npm ERR! ERROR: Failed to set up Chrome r113.0.5672.63! Set "PUPPETEER_SKIP_DOWNLOAD" env variable to skip download.

npm ERR! Error: getaddrinfo ENOTFOUND 127.0.0.1:41315

npm ERR! at GetAddrInfoReqWrap.onlookup [as oncomplete] (node:dns:107:26) {

npm ERR! errno: -3008,

npm ERR! code: 'ENOTFOUND',

npm ERR! syscall: 'getaddrinfo',

npm ERR! hostname: '127.0.0.1:41315'

npm ERR! }

似乎是不能设置proxy为socket5，改成http的proxy即可

# Dapr

## 部署

### no pubsub configured

配置问题。注意组件yaml文件中的namespace是指需要用此组件的项目所在命名空间。而非dapr-system所在命名空间。

## 开发

### error getting topic list from app: rpc error: code = Unavailable desc = last connection error: connection closed before server preface received

连接问题。

比如可能gRPC server服务开了必须使用TLS。

还有一个非常可能的原因是没有使用HTTP2以上协议，默认asp.net core不是使用这个。要在appsetting.json中添加：

"Kestrel": {

"EndpointDefaults": {

"Protocols": "Http2"

}

}

或

builder.WebHost.ConfigureKestrel(opt =\>

{

opt.ListenLocalhost(5003, p =\> p.Protocols = HttpProtocols.Http2); //启用Http2支持

});

### Type 'FlightProtocol.Models.Flight' cannot be serialized. Consider marking it with the DataContractAttribute attribute, and marking all of its members you want serialized with the DataMemberAttribute attribute. Alternatively, you can ensure that the type is public and has a parameterless constructor - all public members of the type will then be serialized, and no attributes will be required.

Strongly Type的是使用DataContract

可采用Weakly Type，是使用System.Text.Json格式化。

### The state store is not configured to use the actor runtime. Have you set the - name: actorStateStore value: "true" in your state store component file?

除了未配置state store之外，还可能是未配置正确dapr-placement所在端口。

![Text Description automatically generated](../attachments/f8465574550795aeae5a9aa36f80364a.png)

The placement service provides a distributed hashing, it's used by Dapr runtimes to distribute the actor instances across various pods of the user service.

# Windows

### 打开程序只显示托盘图标，不显示程序界面

shift+右键点击托盘图标，然后选择移动，可以将鼠标迅速定位到程序所在位置进行拖动。

或者某些有问题的程序，选择最大化，可以强行显示。
