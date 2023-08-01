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

Pod是对容器化的Application的抽象，不仅仅是指代docker，可以替换为任何一种容器技术。

![图形用户界面 低可信度描述已自动生成](../attachments/ec859af1d041089310cf91dfdb84b9dc.png)

一个Pod一般指一个容器实例，如图中rocketcha，db

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

# K8S集群管理

## KubeSphere

### 概念

|名称 |作用|备注|
|:------ |:---- |:---- |
|KubeKey|用于便捷安装K8S和KubeSphere的工具|  |
|VIP|virtual IP，它是一群节点共用的一个虚拟IP|  |
|Ceph |分布式存储方案，实现K8S的存储卷方面的能力|  |

#### Ceph

##### 三种存储类型

**块存储（ RADOS Block Device, RBD )**
其中RADOS全称为Reliable Autonomic Distributed Object Store, 可靠的、自动化的、分布式对象存储系统。
- 优点：
    - 通过Raid与LVM等手段，对数据提供了保护；
    - 多块廉价的硬盘组合起来，提高容量；
    - 多块磁盘组合出来的逻辑盘，提升读写效率；
- 缺点：
    - 采用SAN架构组网时，光纤交换机，造价成本高；
    - 主机之间无法共享数据；
- 使用场景
    - docker容器、虚拟机磁盘存储分配；
    - 日志存储；
    - 文件存储；
**文件存储（ CephFS )**
- 优点：
    - 造价低，随便一台机器就可以了；
    - 方便文件共享；
- 缺点：
    - 读写速率低；
    - 传输速率慢；
- 使用场景
    - 日志存储；
    - FTP、NFS；
    - 其它有目录结构的文件存储
**对象存储（ Object )**
- 优点：
    - 具备块存储的读写高速；
    - 具备文件存储的共享等特性；
- 缺点：
    - 操作系统不能直接访问，只能通过应用程序级别的 API 访问
- 使用场景
    - 图片存储；
    - 视频存储；

## Kuboard

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
