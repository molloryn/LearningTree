# Node

一个 `Node` 即一个 `VM` 或物理机。

分为 `Work Node` 和 `Master Node`

## Work Node

一个 `Work Node` 至少需要运行：

### Container Runtime

如 `Docker`，用以运行容器实例，即 `Pod`。

### Kubelet

调度 `Pod`，以及提供接口。

### Kube Proxy

用于代理，转发来自各个 `Service` 的 `Request` 到各个对应的 `Pod` 中去。

## Master Node

实际上就是 `Control Panel`。

一个 `Master Node` 至少要运行：

### Api server

用于用户通过 `Kubernetes Client`（如 `K8S dashboard` 或 `Kubectl`）进行 `Cluster` 的管理，其实就是 `Cluster` 的 `API Gateway`。

### Scheduler

资源调度中心，可以根据资源情况、环境，决定新的 `Node`、`Pod` 该如何安排。

### Controller manager

监测 `Pod` 的状态，提醒 `Scheduler` 进行恢复等。

### etcd

是一个 `Key-value store` 服务，用于存储 `Cluster` 的状态，是整个 `k8s Cluster` 的大脑。