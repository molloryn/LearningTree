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

### Kubectl

| Command  |Function| Remark                                                                               |
|----------|-----------------|--------------------------------------------------------------------------------------|
| get pods | 获取k8s集群pods | --namespace 指定获取某个名称空间的k8s集群pods  不指定默认是获取default命名空间的pods |
|          |                 |                                                                                      |
|          |                 |                                                                                      |
|          |                 |  |
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

Pod是对容器化的Application的抽象，不仅仅是指代docker，可以替换为任何一种容器技术。只需要遵循`Container Runtime Interface (CRI)`实现和`OCI`规范。

OCI stands for the [Open Container Initiative](https://opencontainers.org/about/overview/), which standardized many of the interfaces between container tools and technologies. They maintain a standard specification for packaging container images (OCI image-spec) and running containers (OCI runtime-spec). They also maintain an actual implementation of the runtime-spec in the form of [runc](https://github.com/opencontainers/runc), which is the underlying default runtime for both [containerd](https://containerd.io/) and [CRI-O](https://cri-o.io/). The `CRI` builds on these low-level specifications to provide an end-to-end standard for managing containers.
CRI is an interface between container runtimes and container orchestration platforms that allows for seamless integration between the two. Docker is a popular containerization platform that has been impacted by these standards.

![图形用户界面 低可信度描述已自动生成](../attachments/ec859af1d041089310cf91dfdb84b9dc.png)

一个Pod一般指一个容器实例，如图中rocketchat，db

但一个Pod也可以有多个容器实例，在Docker中的这种情况，如teamgram-server，称为容器组。

容器组中的容器共享生命周期、资源、本地网络和存储卷，在K8S中也称为 Pod。**容器组有公共的IP地址，两个容器监听不同的端口。**

dapr中的边车，就是很好的例子，边车与应用程序位于同一个Pod内。

### Service

一个Pod有一个自己的IP Address，一个Pod搭配一个Service，Service 管理Pod的IP，Pod挂掉IP也不会变。Service分Internal service和external service，即可被外部访问以及不可被外部访问的。

Pod之间通过Service进行通信。

Node中有多个Pod，多个Service，可以用一个Ingress用于外部路由Service。

可以对Pod进行Replicate，实现HA（是对无状态的Pod）。即使用一个新的Node，运行一样的Pod，然后这两个Pod之间使用同一个Service，这时候，Service也有Load balance的功能。这种功能可以通过“Deployment”组件实现。

而对于DB这类有状态的Pod进行Replicate，需要使用“StatefulSet”组件实现。但是非常复杂，所以一般来说DB是部署在K8S Cluster之外的。

### Configuration

对于Pod的外部配置，可以使用ConfigMap或Secrets。

# Harbor镜像仓库

## 使用方式

### 推送到镜像仓库

```shell
# 获取源镜像文件
docker save <镜像ID> -o <输出目录>.tar

docker load -i <源镜像名> # 这里load出来的是none，可以用镜像ID进行控制
cat /etc/hosts # 查看仓库地址（一般设置了hosts）
docker tag 镜像ID <仓库名>/<项目名>/<镜像名>:[版本号] 
docker push <仓库名>/<项目名>/<镜像名>:[版本号] # push只能通过tag而不能通过ID
# 同一个镜像名不同tag版本号可以在镜像仓库里面选择版本。
```


[K8S集群管理](K8S集群管理.md)