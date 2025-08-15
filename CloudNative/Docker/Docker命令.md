# Docker命令

You can combine single character flags to shorten the full command.

即

`docker run -t -i` 可以简写为`-it`

`-d -p 80:80` 可以简写为 `-dp 80:80`

| 命令 | 含义 | 备注 |
| :--- | :--- | :--- |
| `build -t <name> .` | 建造Image，通过指定目录（这里`.`指示当前目录）下的`Docker file` <br />`-t tag` 给建造的Image起个名 | `--build-arg http_proxy=http://proxy.exaple.com` <br />`--build-arg https_proxy=http://proxy.exaple.com`<br /> **可以指定代理**<br /> `-f` 可以指定`Dockerfile`的位置。一般情况是在`Dockerfile`所在文件夹中运行build。但VS生成的`Dockerfile`，则需要在sln文件夹下，然后用`-f`指定`Dockerfile`位置。 |
| `exec -it <container> /bin/sh` | 进入容器内部执行命令 | 与`run`的区别是`exec`是启动或连接到已经存在/运行的容器中去。 而`run`是对于还未创建的新容器而言，即`run`是需要传image。<br />exit可以退出容器 |
| `cp <from> <to>` | 文件导入导出 | 容器内部路径的写法`8592134a642d:/folder` |
| `run -i -t ubuntu /bin/bash` | 从指定Image（这里是ubuntu）中实例化出一个新的容器并运行。<br /> `-d run as detached` 如需要Long-running 的服务，放在后台运行。 <br />`-p 8080:80` 将本机8080端口映射到容器的80端口 （如果要暴露多个，可以多写几个-p映射） <br />`--name` 给实例化的新容器起个名，不然docker会随机起名 <br />`--privileged=true` 以特权模式运行，可以执行`systemctl`等命令(需要以`/sbin/init`启动然后再用`exec`命令进入容器) <br />`--rm` flag tells Docker to automatically remove the container when it exits <br />`-e XXX='xxx'` 设置环境变量 | 1. If you do not have the `ubuntu` image locally, Docker pulls it from your configured registry, as though you had run docker pull ubuntu manually.<br />2. Docker creates a new container, as though you had run a docker container create command manually.<br/>3. Docker allocates a read-write filesystem to the container, as its final layer. This allows a running container to create or modify files and directories in its local filesystem.<br />4. Docker creates a network interface to connect the container to the default network, since you did not specify any networking options. This includes assigning an IP address to the container. By default, containers can connect to external networks using the host machine's network connection.<br />5. Docker starts the container and executes `/bin/bash`. Because the container is running **interactively** and attached to your terminal (due to the `-i` and `-t` flags), you can provide input using your keyboard while the output is logged to your terminal.<br />6. When you type exit to terminate the `/bin/bash` command, the container stops but is not removed. You can start it again or remove it. |
| `images` | 列出当前已有的`images` | |
| `commit <container> [image]` | 将指定容器更改保存到新镜像 <br />Create a new image from a container's changes | |
| `ps` | processes <br />当前运行的容器 <br />`-a` 显示所有包括未运行的. | `--size` 显示当前容器占用空间大小 |
| `logs <container>` | 查看容器日志 `-f follow`，追踪当前容器的输出 | 比如使用了`-detached`参数启动在后台运行的容器，要看它的输出可以使用该命令 |
| `rm <container>` | 移除容器 <br />`-f force` 强制移除容器，即使它在运行 | Once the container has stopped, it can be removed. |
| `rmi <image>` | 移除镜像 | |
| `stop <container>` | 停止运行一个容器 | |
| `push <docker hub repository name>` | 推送image到docker hub | 首先需要 `docker tag local-image:tagname new-repo:tagname`<br>没有权限需要`docker login` |
| `pull <docker hub repository name>` | 拉取image到本地 | |
| `update` | 更新配置？ | `--restart=no <CONTAINER ID>` 对某一个容器关闭自启动<br />`--restart=no $(docker ps -q)` 取消所有容器自启动 |
| `export` | 导出容器实例 | 主要用来制作基础镜像，比如我们从一个 ubuntu 镜像启动一个容器，然后安装一些软件和进行一些设置后，使用 `docker export` 保存为一个基础镜像。然后，把这个镜像分发给其他人使用，比如作为基础的开发环境。 |
| `import` | 导入容器实例 | |
| `save` | 保存镜像 | `-o` 导出的路径 建议.tar结尾 可以同时将多个镜像打包到一个文件中 |
| `load` | 加载镜像 | `-i` 输入文件路径 |
| `container prune` | 移除停止的容器 | `image prune` 移除悬挂镜像。`-a`则是所有未使用的镜像。 |
| `stats` | 监控容器资源占用情况 | |
| `update --restart=no <CONTAINER ID>` | 取消容器自启动 | |

## 运行轻量级程序

对于一些CLI程序的容器而言，可以这样去使用他们： `docker run -it --rm remnux/ciphey --help` ，其中的 `--help` 就是指示的它是进入了容器后执行的。