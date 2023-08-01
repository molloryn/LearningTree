# 概念

就是将自己的程序以及运行它所需的一切环境依赖等等封装成一个包，开箱即用。

在Docker的世界里，这个包叫做Image（镜像）。它可以看作是一个虚拟机镜像，已经装好了运行程序所需的所有依赖。

通过Image实例化出一个Container（容器），启动容器就是启动了自己的程序。就像是使用了虚拟机镜像并开机。

Docker平台对容器提供了一系列功能，最终实现CI/CD。

Docker虽然像虚拟机，但差别还是很大。简单来说，它是轻量级的虚拟化，没有打包操作系统进去，而是直接使用了宿主机的操作系统。

![Traditional vs New Gen](../attachments/8318fe534711228f439b8bbdf67493b2.png)

## Docker Engine

Docker Engine is an open-source containerization technology for building and containerizing your applications. Docker Engine acts as a client-server application with:

A server with a long-running daemon process dockerd.

APIs which specify interfaces that programs can use to talk to and instruct the Docker daemon.

A command line interface (CLI) client docker.

### 概述

Docker是CS架构体系。

Docker使用的是CLI式的客户端去对运行的Docker服务（The Docker daemon or dockerd，which does the heavy lifting of building, running, and distributing your Docker containers）做一系列操作，服务本身提供了一系列的操作接口，以供客户但调用。

![](../attachments/9238d36410d9a82a5376b61f185db37b.png)

### The Docker daemon（守护进程）

The Docker daemon (dockerd) listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes. A daemon can also communicate with other daemons to manage Docker services.

### The Docker client

The Docker client (docker) is the primary way that many Docker users interact with Docker. When you use commands such as docker run, the client sends these commands to dockerd, which carries them out. The docker command uses the Docker API. The Docker client can communicate with more than one daemon.

### Docker Desktop

Docker Desktop is an easy-to-install application for your Mac or Windows environment that enables you to build and share containerized applications and microservices. Docker Desktop includes the Docker daemon (dockerd), the Docker client (docker), Docker Compose, Docker Content Trust, Kubernetes, and Credential Helper.

### Docker registries

（类比GitHub）

A Docker registry stores Docker images. Docker Hub is a public registry that anyone can use, and Docker is configured to look for images on Docker Hub by default. You can even run your own private registry.

When you use the docker pull or docker run commands, the required images are pulled from your configured registry. When you use the docker push command, your image is pushed to your configured registry.

### Docker objects

When you use Docker, you are creating and using images, containers, networks, **volumes**, plugins, and other objects. This section is a brief overview of some of those objects.

### Images

An image is a read-only template with instructions for creating a Docker container. Often, an image is based on another image, with some additional customization. For example, you may build an image which is based on the ubuntu image, but installs the Apache web server and your application, as well as the configuration details needed to make your application run.

You might create your own images, or you might only use those created by others and published in a registry. To build your own image, you create a Dockerfile with a simple syntax for defining the steps needed to create the image and run it. Each instruction in a Dockerfile creates a layer in the image. When you change the Dockerfile and rebuild the image, only those layers which have changed are rebuilt. This is part of what makes images so lightweight, small, and fast, when compared to other virtualization technologies.

简单理解，Image类比到“类”，dockerFile是类的构造函数，Container就是实例化的Image。

### Containers

A container is **a runnable instance of an image**. You can create, start, stop, move, or delete a container using the Docker API or CLI. You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state.

By default, a container is relatively well isolated from other containers and its host machine（意味着要连接宿主机需要做端口映射）. You can control how isolated a container’s network, storage, or other underlying subsystems are from other containers or from the host machine.

A container is defined by its image as well as any configuration options you provide to it when you create or start it. When a container is removed, any changes to its state that are not stored in persistent storage disappear.

### The underlying technology

Docker is written in the **Go** programming language and takes advantage of several features of the Linux kernel to deliver its functionality. Docker uses a technology called **namespaces** to provide the isolated workspace called the container. When you run a container, Docker creates a set of namespaces for that container.

These namespaces provide a layer of isolation. Each aspect of a container runs in a separate namespace and its access is limited to that namespace

# 命令

You can combine single character flags to shorten the full command.

即

docker run -t -i 可以简写为-it

\-d -p 80:80 可以简写为 -dp 80:80

| 命令                                | 含义                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | 备注                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| build -t \<name\> .                 | 建造Image，通过指定目录（这里.指示当前目录）下的Docker file -t tag 给建造的Image起个名                                                                                                                                                                                                                                                                                                                                                                                          | --build-arg http_proxy=http://proxy.exaple.com --build-arg https_proxy=http://proxy.exaple.com **可以指定代理**  -f 可以指定Dockerfile的位置。一般情况是在Dockerfile所在文件夹中运行build。但VS生成的Dockerfile，则需要在sln文件夹下，然后用-f指定Dockerfile位置。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| exec -it \<container\> /bin/sh      | 启动并进入容器内部                                                                                                                                                                                                                                                                                                                                                                                                                                                              | 与run的区别是exec是启动或连接到已经存在/运行的容器中去。 而run是对于还未创建的新容器而言，即run是需要传image                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| cp \<from\> \<to\>                  | 文件导入导出                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | 容器内部路径的写法8592134a642d:/folder                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| run -i -t ubuntu /bin/bash          | 从指定Image（这里是ubuntu）中实例化出一个新的容器并运行。 -d run as detached. 如需要Long-running 的服务，放在后台运行。 -p 8080:80 将本机8080端口映射到容器的80端口 （如果要暴露多个，可以多写几个-p映射） --name 给实例化的新容器起个名，不然docker会随机起名 --privileged=true 以特权模式运行，可以执行systemctl等命令(需要以/sbin/init启动然后再用exec命令进入容器)  "--rm" flag tells Docker to automatically remove the container when it exits  -e XXX=’xxx’ 设置环境变量 | ①If you do not have the ubuntu image locally, Docker pulls it from your configured registry, as though you had run docker pull ubuntu manually.  ②Docker creates a new container, as though you had run a docker container create command manually.  ③Docker allocates a read-write filesystem to the container, as its final layer. This allows a running container to create or modify files and directories in its local filesystem.  ④Docker creates a network interface to connect the container to the default network, since you did not specify any networking options. This includes assigning an IP address to the container. By default, containers can connect to external networks using the host machine’s network connection.  ⑤Docker starts the container and executes /bin/bash. Because the container is running interactively and attached to your terminal (due to the -i and -t flags), you can provide input using your keyboard while the output is logged to your terminal.  ⑥When you type exit to terminate the /bin/bash command, the container stops but is not removed. You can start it again or remove it.  对于一些CLI程序的容器而言，可以这样去使用他们： docker run -it --rm remnux/ciphey --help  蓝色部分就是指示的它是进入了容器后执行的 |
| images                              | 列出当前已有的images                                                                                                                                                                                                                                                                                                                                                                                                                                                            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| commit \<container\> [image]        | 将指定容器更改保存到新镜像 Create a new image from a container’s changes                                                                                                                                                                                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ps                                  | processes 当前运行的容器 -a 显示所有包括未运行的.                                                                                                                                                                                                                                                                                                                                                                                                                               | --size 显示当前容器占用空间大小                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| logs \<container\>                  | 查看容器日志 -f follow，追踪当前容器的输出                                                                                                                                                                                                                                                                                                                                                                                                                                      | 比如使用了-detached参数启动在后台运行的容器，要看它的输出可以使用该命令                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| rm \<container\>                    | 移除容器 -f force 强制移除容器，即使它在运行                                                                                                                                                                                                                                                                                                                                                                                                                                    | Once the container has stopped, it can be removed.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| rmi \<image\>                       | 移除镜像                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| stop \<container\>                  | 停止运行一个容器                                                                                                                                                                                                                                                                                                                                                                                                                                                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| push \<docker hub repository name\> | 推送image到docker hub                                                                                                                                                                                                                                                                                                                                                                                                                                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| pull \<docker hub repository name\> | 拉取image到本地                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| update                              | 更新配置？                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | --restart=no \<CONTAINER ID\> 对某一个容器关闭自启动 --restart=no \$(docker ps -q) 取消所有容器自启动                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| export                              | 导出容器实例                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | 主要用来制作基础镜像，比如我们从一个 ubuntu 镜像启动一个容器，然后安装一些软件和进行一些设置后，使用 docker export 保存为一个基础镜像。然后，把这个镜像分发给其他人使用，比如作为基础的开发环境。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| import                              | 导入容器实例                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| save                                | 保存镜像                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | -o 导出的路径 建议.tar结尾 可以同时将多个镜像打包到一个文件中                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| load                                | 加载镜像                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | -i 输入文件路径                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |

### Failed to get D-Bus connection: Operation not permitted

Docker的设计理念是在容器里面不运行后台服务，容器本身就是宿主机上的一个独立的主进程，也可以间接的理解为就是容器里运行服务的应用进程。一个容器的生命周期是围绕这个主进程存在的，所以正确的使用容器方法是将里面的服务运行在前台。

再说到systemd，这个套件已经成为主流Linux发行版（比如CentOS7、Ubuntu14+）默认的服务管理，取代了传统的SystemV风格服务管理。systemd维护系统服务程序，它需要特权去会访问Linux内核。而容器并不是一个完整的操作系统，只有一个文件系统，而且默认启动只是普通用户这样的权限访问Linux内核，也就是没有特权，所以自然就用不了！

因此，请遵守容器设计原则，一个容器里运行一个前台服务！

我就想这样运行，难道解决不了吗？

答：可以，以特权模式运行容器。

创建容器：

\# docker run -tid --name centos_1 --privileged=true centos:latest /sbin/init

进入容器：

\# docker exec -it centos7 /bin/bash

这样可以使用systemctl启动服务了。

# 扩展

## Docker File

## Docker Network

### Drivers

也是容器可用的网络种类

#### bridge

![图示 描述已自动生成](../attachments/46c8772bd55a774ac297151c029a2f23.png)

创建的容器默认都是这种模式(default bridge driver)，和主机通过docker生成的虚拟网桥连接起来。容器之间和主机之间可以互相通过ip访问。

如果是用户定义的bridge(user-defined bridge)，则默认可以通过容器名互相访问。

#### 

If you are using Docker on MacOS or Windows 18.03+, you can connect to the magic hostname host.docker.internal.

如果是WSL2，host会有相应的虚拟网卡，通过ipconfig可以看到虚拟ip。容器也可以通过该ip访问到宿主机。(但有时候好像又不行)

注意host.docker.internal和wsl虚拟的ip不一致但都能访问到宿主机。

#### host

直接使用宿主机网络，即就像是运行在宿主机上的服务。

### 命令

| 命令                                              | 含义                     | 备注                                                                                                |
|---------------------------------------------------|--------------------------|-----------------------------------------------------------------------------------------------------|
| network ls                                        | 列出所有创建的网络       |                                                                                                     |
| network inspect \<network-id\>                    | 查看网络信息             | 可以看到当前网络有哪些容器加入。 想要看某个容器加入了什么网络，使用docker inspect \<container-id\>  |
| network create \<network-name\>                   | 创建一个user-defined网络 | 默认是创建user-defined bridge网络                                                                   |
| network connect \<network-name\> \<container\>    | 连接一个现有的容器到网络 |                                                                                                     |
| network disconnect \<network-name\> \<container\> |                          |                                                                                                     |

## Docker Storage

If you want to persist data in Docker, the recommended way is to use Docker Volumes. Without the use of Docker Volumes, the writeable layer of your Docker Container is removed from your Host as soon as you remove the Docker Container. But a Docker Volume is a separate Docker Object, which is not removed when you remove a Docker Container.

Volumes可以看作是独立出来的一个writeable layer，它在容器管理之外（有自己的管理方法），它可以在容器创建的时候设置映射，将容器内的某个目录映射到指定的volume中去。

![types of mounts and where they live on the Docker host](../attachments/cd09b2d732da72b743dc5c5d68f7e0f6.png)

| Type         | Explanation                                                                                                                                                                                                                                 |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Volumes      | Volumes are stored in a part of the host filesystem which is managed by Docker (/var/lib/docker/volumes/ on Linux). Non-Docker processes should not modify this part of the filesystem. Volumes are the best way to persist data in Docker. |
| Bind mounts  | Bind mounts may be stored anywhere on the host system. They may even be important system files or directories. Non-Docker processes on the Docker host or a Docker container can modify them at any time.                                   |
| tmpfs mounts | tmpfs mounts are stored in the host system’s memory only, and are never written to the host system’s filesystem.                                                                                                                            |

### Docker Volume

**Created and managed** by Docker.

Volumes provide the ability to connect specific filesystem paths of the container back to the host machine. If a directory in the container is mounted, changes in that directory are also seen on the host machine.

它可以映射（挂载mounting）容器目录到宿主机的特定目录

它的路径由docker自己决定，在创建容器时，-v的使用方式是my-volume:/usr/local/data

### Bind mounts

The file or directory does not need to exist on the Docker host already. It is created on demand if it does not yet exist（但如果使用—mount，则不会自动创建，而是error）

路径由自己控制，-v /path/to/data:/usr/local/data，可以使用relative path.

windows下，也可以使用Unix-style path

C:\\Users\\user\\work -\> /c/Users/user/work

### 命令

| 命令                                                                                                                                                                     | 含义                                                                                                                 | 备注                                                                                                                                                       |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| volume create \<volume name\>                                                                                                                                            | 创建一个volume对象                                                                                                   |                                                                                                                                                            |
| volume ls                                                                                                                                                                | 列出拥有的volume对象                                                                                                 |                                                                                                                                                            |
| volume inspect \<volume name\>                                                                                                                                           | 查看volume对象信息                                                                                                   | Mountpoint就是实际映射到的url，具体与driver有关。 当driver是local，即挂载到宿主机（windows的是在docker VM上）。                                            |
| volume prune                                                                                                                                                             | 移除未使用的Volume                                                                                                   | As long as volumes are associated with a container (either running or not), they cannot be removed.                                                        |
| docker run -e ‘ACCEPT_EULA=Y’ -e ‘SA_PASSWORD=passw0rd1!’ -p 1433:1433 –name sql2019 -v sql_volume:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2019-GA-ubuntu-16.04 | 其中-v是设置容器中/var/opt/mssql目录挂载到sql_volume，即容器对于该目录任何read-write操作将会直接作用在sql_volume上。 | -v 名字(docker volumn)或目录（bind mounts）：容器目录 比较容易使用 官方建议使用--mount  --mount type=bind,source=/tmp,target=/usr 最大区别在于显式与隐式。 |

d

## Docker Compose

使用.yml文件，整合多个创建镜像的dockerfile文件，并配置镜像，如端口映射、容器名称等

### Visual studio 集成

![Graphical user interface, text, application Description automatically generated](../attachments/81252500c55a7899b75ad4b5b7bda8ce.png)

.env 文件里面可以定义全局的环境变量VARIABLE_NAME=XXX

然后在docker-compose.yml等文件中可以\${VARIABLE_NAME}这样使用。

还有dapr中namespace环境变量的作用：

\# If the NAMESPACE env var is set, Dapr does not load any

\# component that does not specify the same namespace.

NAMESPACE=fips

docker-compose.yml中定义的是镜像及网络

![Text Description automatically generated](../attachments/28be9916e14084384c26ca651f95740b.png)

docker-compose.override.yml中定义的是配置。

注意使用 服务名:端口 中的端口是容器内部实际端口（即ports的右边那个），而不是对外暴露的映射主机的端口(还未确定)

![Text Description automatically generated](../attachments/6144cea34177aa54866c21c45176ba03.png)

### docker-compose.yml文件

其中服务使用Volume时候，可以有四种写法：

1.Relative Path

2.Absolute Path

3.Docker Volume Default Path

4.Docker Volume with Absolute Path

<https://stackoverflow.com/a/46910980/18731746>

其中Relative Path在windows中，使用./xxx开头，实际上是相对于指定的docker-compose.yaml文件所在文件夹中。

### 命令

| 命令 | 含义 | 备注 |
|------|------|------|
|      |      |      |
|      |      |      |
|      |      |      |
|      |      |      |
|      |      |      |
|      |      |      |

# 错误排查

docker.errors.DockerException: Error while fetching server API version: ('Connection aborted.', FileNotFoundError(2, 'No such file or directory'))

原因是 docker 没有启动

开启docker: systemctl start docker

查看docker 进程 ps -ef \| grep docker

最后执行docker-compose up -d

##### win10 docker启动后检查版本报错：

Docker.Core.Backend.BackendException:

Error response from daemon: open \\.\\pipe\\docker_engine_linux: The system cannot find the file specified.

在win10 命令行提示符执行：

Net stop com.docker.service

Net start com.docker.service

#### Failed to get D-Bus connection: Operation not permitted

centos无法使用systemctl等，需要创建容器的时候，使用特权模式。docker run -d -it --privileged {Image ID} /usr/sbin/init （需要使用-d在后台，然后用exec -it /bin/bash进入）

# Visual Studio 集成

visual studio 中自动Build生成的镜像无法离开VS而生效。

<https://docs.microsoft.com/en-us/visualstudio/containers/edit-and-refresh?view=vs-2022>

是不是因为没有dot net .dll启动 entry point导致直接退出

## 部署到Docker

### 使用Publish到文件夹

先使用vs的publish功能到指定文件夹，然后在那个文件夹，创建Dockerfile，写命令，构建镜像。

参考asp.net 6的Dockerfile

FROM mcr.microsoft.com/dotnet/aspnet:6.0

WORKDIR /app

COPY . .

EXPOSE 80

EXPOSE 443

ENTRYPOINT ["dotnet", "QuestionnaireReport.dll"]

### 使用Publish到镜像仓库

## 错误排查

#### Cannot use file stream for [\*.deps.json]: No such file or directory

删除publish文件夹再次publish

#### Resharper Build时，Docker compose总不成功

截至Resharper 2022.2.3 似乎不太支持？因为不会重新构建容器。

换用VSbuild即可以Docker Compose运行。

#### Unable to find the target operating system for the project

重新用其他配置运行一下，然后再回到docker配置运行，就发现可以了（迷惑）

#### Docker - failed to compute cache key: not found - runs fine in Visual Studio

.net core 及5以上 需要在.sln文件夹下（或在项目文件夹下使用docker build .. -f Dockerfile）使用docker build .，然后用-f指定Dockerfile位置。

.net framework是在项目目录下

#### No packages exist with this id in source(s): nuget.org

<https://stackoverflow.com/questions/48821991/dockerfile-cant-see-local-file-or-private-nuget-server>

Copying the Nuget.Config to the solution or project folder will work if your 'private nuget feed' is accessible via a url. But if the private feed is a local folder used as a nuget source, this approach will still fail with an error that the folder is outside the build context or simply because the Windows path does not get resolved by the Docker build process.

(但最后仍然失败了，用publish然后自己写dockerfile那个方法替代)

Nuget.Config 在 Users\\\${username}\\AppData\\Roaming\\NuGet下

#### 执行dockercompose后报UTF8问题

docker compose文件里面不能有中文

## 增加

You can add Docker support to an existing project by selecting Add \> Docker Support in Solution Explorer. The Add \> Docker Support and Add \> Container Orchestrator Support commands are located on the right-click menu (or context menu) of the project node for an ASP.NET Core project in Solution Explorer, as shown in the following screenshot:

![](../attachments/3e5ee20a6a84cbaefb79b73951bdb637.png)

# Kubernetes (Orchestration)

用于容器编排

部署一个Kubernetes，实际上就是部署了一个Cluster。

![](../attachments/18c6f89fd0d7c2bb3af7226ae657baae.png)

Kubernetes每个实例是以Cluster为单位的。每个Work Node（VM或物理机）中有Kubelet，即一个agent，由Control Plane进行中心化调度管理（由Master Node组成）。

![](../attachments/f30159ce56ad05756bd53552d9d7817d.png)

用户可以通过Kubectl（Kubernetes Client），通过它调用Kubernetes的API进行管理。

一个k8s集群，可以通过namespace区分区域。

## 网络

在K8S中，同一个命名空间（namespace）下的服务之间调用，之间通过服务名（service name）调用即可。不过在更多时候，我们可能会将一些服务单独隔离在一个命名空间中（比如我们将中间件服务统一放在 middleware 命名空间中，将业务服务放在 business 命名空间中）。

遇到这种情况，我们就需要跨命名空间访问，K8S 对service 提供了四种不同的类型，针对这个问题我们选用 ExternalName 类型的 service 即可。

注意service端口是否开对

## 命令

| Command  | Function        | Remark                                                                               |
|----------|-----------------|--------------------------------------------------------------------------------------|
| kubectl  |                 |                                                                                      |
| get pods | 获取k8s集群pods | --namespace 指定获取某个名称空间的k8s集群pods  不指定默认是获取default命名空间的pods |
|          |                 |                                                                                      |
|          |                 |                                                                                      |
|          |                 |                                                                                      |
|          |                 |                                                                                      |
|          |                 |                                                                                      |

## Node

一个Node即一个VM或物理机。

分为Work Node和Master Node

### Work Node

一个Work Node至少需要运行：

#### Container Runtime

如Docker，用以运行容器实例，即Pod。

#### Kubelet

调度Pod，以及提供接口。

#### Kube Proxy

用于代理，转发来自各个Service的Request到各个对应的Pod中去。

### Master Node

实际上就是Control Panel。

一个Master Node至少要运行：

#### Api server

用于用户通过Kubernetes Client（如K8S dashboard或Kubectl）进行Cluster的管理，其实就是Cluster的API Gateway。

#### Scheduler

资源调度中心，可以根据资源情况、环境，决定新的Node、Pod该如何安排。

#### Controller manager

监测Pod的状态，提醒Scheduler进行恢复等。

#### etcd

是一个Key-value store 服务，用于存储Cluster的状态，是整个k8s Cluster的大脑。

## Pod

Pod是对容器化的Application的抽象，不仅仅是指代docker，可以替换为任何一种容器技术。

![图形用户界面 低可信度描述已自动生成](../attachments/ec859af1d041089310cf91dfdb84b9dc.png)

一个Pod一般指一个容器实例，如图中rocketcha，db

但也可以指一群容器实例，如teamgram-server，即需要其他一些中间件以提供一个完整的Application。

### Service

一个Pod有一个自己的IP Address，一个Pod搭配一个Service，Service 管理Pod的IP，Pod挂掉IP也不会变。Service分Internal service和external service，即可被外部访问以及不可被外部访问的。

Pod之间通过Service进行通信。

Node中有多个Pod，多个Service，可以用一个Ingress用于外部路由Service。

可以对Pod进行Replicate，实现HA（是对无状态的Pod）。即使用一个新的Node，运行一样的Pod，然后这两个Pod之间使用同一个Service，这时候，Service也有Load balance的功能。这种功能可以通过“Deployment”组件实现。

而对于DB这类有状态的Pod进行Replicate，需要使用“StatefulSet”组件实现。但是非常复杂，所以一般来说DB是部署在K8S Cluster之外的。

### Configuration

对于Pod的外部配置，可以使用ConfigMap或Secrets。

## Other

### Kuboard

国产K8s集群管理Dashboard

#### Pod添加

可以通过UI手动创建工作负载，也可以从Yaml文件导入

一个Pod（工作负载）可以有多个容器，生命周期是一起的？而且这几个容器不能使用同一个端口。

Pod下的容器端口，实际上基本只要填本机端口（表示容器开了这个端口），一般不填宿主机端口。

#### 同命名空间下服务互调

需要在Pod编辑中，服务/应用路由中开通服务，暴露端口。

选择NodePort：服务对外暴露的端口、服务本身目标端口（一般一致，然后就在本命名空间下的服务可以通过服务名+端口访问了），中间是集群外部公布的端口？可以通过外部IP：端口供给集群外部访问。

#### 跨命名空间访问

可以通过ClusterIP

#### 配置中心

ConfigMap既可以映射环境变量，也可以将Key-Value变成文件名，文件内容的形式进行文件映射。

文件映射需要在Pod中设置，存储挂载中，添加数据卷

选择配置字典，KeyToPath，这里就可以选择文件并给定映射到的路径

数据卷挂载到容器，上面是容器目录，下面是映射到的数据卷目录

#### Pod容器日志

现仅支持查看追踪日志，容器日志需要Kuborad另外配置？

#### 连接容器

容器下面bash、sh可以用，还可以通过文件浏览器直接管理容器的文件。

快捷访问端口：开通的端口可以简单通过端口右边绿色的按钮通过代理访问。

### Minikube

1 Node K8s Cluster，即Master node、Work node都在一个Node上。

一般用于本地测试、学习。
