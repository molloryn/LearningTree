# KubeSphere

## 概念

| 名称     |作用| 备注   |
|:-------- |:---------------------------------------------------------------------------------------- |:------ |
| KubeKey  | 用于便捷安装K8S和KubeSphere的工具                                                        | 简称kk |
| VIP      | virtual IP，它是一群节点共用的一个虚拟IP                                                 |        |
| Ceph     | 分布式存储方案，实现K8S的存储卷方面的能力                                                |        |
| Ceph-CSI | Ceph Container Storage Interface. Ceph 容器存储接口是一个用于 RBD 和 CephFS 的驱动程序。 |        |
| HAproxy  |负载均衡器，用于创建高可用K8S集群。provides high availability, load balancing, and proxying for TCP and HTTP-based applications. |  |
|Keepalived |provides high availability for Linux systems by allowing multiple servers to share a virtual IP address. |  |
## HAproxy

负载均衡器及反向代理服务器。
load balancer and proxy server software that is used to distribute network traffic across multiple servers to improve performance, reliability, and scalability of web applications. It uses a round-robin algorithm to distribute incoming requests to the servers in a balanced way.
## Keepalived

Keepalived通过VRRP协议实现服务或网络层高可用，即实现节点健康状态监测、剔除集群中故障节点。
它一般与负载均衡器一起实现高可用，所以可以看到有`nginx+keepalived`，以及这里的`HAproxy+keepalived`的搭配。
### VRRP

`FHRP(First Hop Redundancy Protocol)`协议，第一跳冗余协议，解决路由器单点故障问题，从协议上进行解决，让多个路由器使用一个虚拟IP。FHRP stands for First Hop Redundancy Protocol, which is used to provide redundancy for the first hop of a network. This is typically done by having multiple routers share the same virtual IP address, so that if one router fails, another can take over seamlessly. 
FHRP协议有很多种具体协议实现，私有的如思科的`HSRP`、`GLBP`等，公有的则是著名的`VRRP`。`VRRP (Virtual Router Redundancy Protocol)` is a public protocol that provides the same functionality as HSRP, allowing multiple routers to share a virtual IP address to provide redundancy. VRRP is not tied to any specific vendor or platform, making it more widely used than HSRP.

`VRRP`协议的使用就是架构图中常见的`VIP`
![](../attachments/Pasted%20image%2020230809141442.png)

## Ceph

### 三种存储类型

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

# Kuboard

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

# Minikube

1 Node K8s Cluster，即Master node、Work node都在一个Node上。

一般用于本地测试、学习。
