# Docker错误排查

## COPY找不到文件

首先要弄清楚 `context` 原理：
dockerfile的context是 `docker -f xxxx.dockerfile -t tagname <context>` 中 `context` 参数。
docker-compose中的context设置是
```yaml
flight-service-api:
  image: local-test/flight-service-api
  build:
    context: ../
    dockerfile: ./configs/dockerfiles/FlightServiceAPI.dockerfile
```
docker-compose以及dockerfile支持`.dockerignore`，但注意它只能在`context`所在目录下，因此要确定是否COPY的文件目录被忽略了，导致找不到文件。
而后COPY的目录都是相当于设定的上下文开始的，也是最外层，再往context的上一层就不行了。

> volumn中挂载的是**按照实际目录相对路径**来的，而不是Context的目录。

## docker.errors.DockerException: Error while fetching server API version: ('Connection aborted.', FileNotFoundError(2, 'No such file or directory'))

原因是 docker 没有启动

开启docker: `systemctl start docker`

查看docker 进程 `ps -ef | grep docker`

最后执行`docker-compose up -d`

## win10 docker启动后检查版本报错

```log
Docker.Core.Backend.BackendException:

Error response from daemon: open \.\pipe\docker_engine_linux: The system cannot find the file specified.
```

在win10 命令行提示符执行：

```sh
net stop com.docker.service
net start com.docker.service
```

## Failed to get D-Bus connection: Operation not permitted
centos无法使用systemctl等，需要创建容器的时候，使用特权模式。

`Docker`的设计理念是在容器里面不运行后台服务，容器本身就是宿主机上的一个独立的主进程，也可以间接的理解为就是容器里运行服务的应用进程。一个容器的生命周期是围绕这个主进程存在的，所以正确的使用容器方法是将里面的服务运行在前台。

再说到`systemd`，这个套件已经成为主流`Linux`发行版（比如`CentOS7`、`Ubuntu14+`）默认的服务管理，取代了传统的`SystemV`风格服务管理。`systemd`维护系统服务程序，它需要特权去会访问`Linux`内核。而容器并不是一个完整的操作系统，只有一个文件系统，而且默认启动只是普通用户这样的权限访问`Linux`内核，也就是没有特权，所以自然就用不了！

因此，请遵守容器设计原则，一个容器里运行一个前台服务！

我就想这样运行，难道解决不了吗？

答：可以，以特权模式创建镜像。

```sh
# 创建容器：
docker run -d -it --name xxxx --privileged=true {Image ID} /sbin/init

# 还有说是/user/sbin/init，不知道是哪个

# 进入容器：
docker exec -it centos7 /bin/bash
```

这样可以使用systemctl启动服务了。

## Docker-in-Docker问题，daemon时不时挂掉
搭建jenkins时直接使用jenkins容器中的docker，导致进行docker build等推送时常出现`ERROR: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`的错误
分析日志后发现可能是由于存储驱动问题，回退到`vfs`这种不稳定的驱动，会导致镜像构建和容器运行时性能低下、磁盘占用暴增，最终可能因资源不足引发连锁问题。
```
time="2025-04-01T09:09:02.013354925+08:00" level=error msg="failed to mount overlay: invalid argument" storage-driver=overlay2
time="2025-04-01T09:09:02.013431016+08:00" level=error msg="exec: \"fuse-overlayfs\": executable file not found in $PATH" storage-driver=fuse-overlayfs
time="2025-04-01T09:09:02.223953127+08:00" level=info msg="Docker daemon" storage-driver=vfs
```

## 部署到Docker

### 使用Publish到文件夹

先使用vs的publish功能到指定文件夹，然后在那个文件夹，创建`Dockerfile`，写命令，构建镜像。

参考`asp.net 6`的`Dockerfile`

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0
WORKDIR /app
COPY . .
EXPOSE 80
EXPOSE 443
ENTRYPOINT ["dotnet", "QuestionnaireReport.dll"]
```

### 使用Publish到镜像仓库

## Failed to start Docker Application Container Engine.

查看日志
```bash
sudo journalctl -u docker.service
```
发现是配置问题：
```log
Apr 26 14:29:15 lavm-xi6x7kk6zz systemd[1]: docker.service: Consumed 27min 196ms CPU time. -- Boot 5cd1554dd89e43c582bcbfe75e1facfa -- Apr 26 14:30:14 lavm-xi6x7kk6zz systemd[1]: Starting Docker Application Container Engine... Apr 26 14:30:14 lavm-xi6x7kk6zz dockerd[820]: unable to configure the Docker daemon with file /etc>
```

少了逗号
```json
cat /etc/docker/daemon.json
```

## The "Configuration" parameter is not supported by the "WaitForWarmupCompletion" task loaded from assembly: Microsoft.VisualStudio.Containers.Tools.Tasks

似乎是bug，如果多个项目引用了不同的 `Container.Tools.Targets` 包，就有可能导致该问题。
`Version=17.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a from the path: xxxxxxxxxxxxx\microsoft.visualstudio.azure.containers.tools.targets\1.19.4\tools\Microsoft.VisualStudio.Containers.Tools.Tasks.dll. Verify that the parameter exists on the task, the <UsingTask> points to the correct assembly, and it is a settable public instance property.`
出现后需要拉齐包版本，去提示的目录下删除那个版本的库，然后重新启动vs。

## Cannot use file stream for [*.deps.json]: No such file or directory

删除publish文件夹再次publish

## Resharper Build时，Docker compose总不成功

截至`Resharper 2022.2.3` 似乎不太支持？因为不会重新构建容器。

换用VSbuild即可以`Docker Compose`运行。

## Unable to find the target operating system for the project

重新用其他配置运行一下，然后再回到docker配置运行，就发现可以了（迷惑）

## Docker - failed to compute cache key: not found - runs fine in Visual Studio

`.net core` 及5以上 需要在`.sln`文件夹下（或在项目文件夹下使用`docker build .. -f Dockerfile`）使用`docker build .`，然后用`-f`指定`Dockerfile`位置。

`.net framework`是在项目目录下

## No packages exist with this id in source(s): nuget.org

[Stack Overflow - Dockerfile can't see local file or private nuget server](https://stackoverflow.com/questions/48821991/dockerfile-cant-see-local-file-or-private-nuget-server)

Copying the `Nuget.Config` to the solution or project folder will work if your 'private nuget feed' is accessible via a url. But if the private feed is a local folder used as a nuget source, this approach will still fail with an error that the folder is outside the build context or simply because the Windows path does not get resolved by the Docker build process.

(但最后仍然失败了，用publish然后自己写dockerfile那个方法替代)

`Nuget.Config` 在 `Users\${username}\AppData\Roaming\NuGet`下

## 执行dockercompose后报UTF8问题

docker compose文件里面不能有中文

## 执行后无限等待，CANCELED [xxx internal] load build context

docker compose中镜像构建的 context 文件找不到，可能是路径写错或者没创建相应的`Dockerfile`。

## The path must be absolute.
[Regression: "The path must be absolute." on Linux · Issue #38175 · dotnet/aspnetcore (github.com)](https://github.com/dotnet/aspnetcore/issues/38175)
将Development的容器环境变量换成Production即可。似乎是Bug。

## Failed to create CoreCLR
`.NET 8` 在另外的机器上跑同个镜像失败。原因是docker engine版本过低，20需升级到23以上。

## Docker compose起来后边车无法连接RabbitMQ
`rabbitMQ`还没起来，边车就去访问了，需要用depends on解决先后顺序。

## Can not find the container with the name starting with XXX
检查是否是以 `Release` 模式运行，而又禁用了 `Release` 的容器运行

## 增加

You can add Docker support to an existing project by selecting Add > Docker Support in Solution Explorer. The Add > Docker Support and Add > Container Orchestrator Support commands are located on the right-click menu (or context menu) of the project node for an ASP.NET Core project in Solution Explorer, as shown in the following screenshot:

![](../../attachments/3e5ee20a6a84cbaefb79b73951bdb637.png)