# Pod

`Pod` 是对容器化的 `Application` 的抽象，不仅仅是指代 `docker`，可以替换为任何一种容器技术。只需要遵循 `Container Runtime Interface (CRI)` 实现和 `OCI` 规范。

`OCI` stands for the [Open Container Initiative](https://opencontainers.org/about/overview/), which standardized many of the interfaces between container tools and technologies. They maintain a standard specification for packaging container images (`OCI image-spec`) and running containers (`OCI runtime-spec`). They also maintain an actual implementation of the `runtime-spec` in the form of [runc](https://github.com/opencontainers/runc), which is the underlying default runtime for both [containerd](https://containerd.io/) and [CRI-O](https://cri-o.io/). The `CRI` builds on these low-level specifications to provide an end-to-end standard for managing containers.
`CRI` is an interface between container runtimes and container orchestration platforms that allows for seamless integration between the two. `Docker` is a popular containerization platform that has been impacted by these standards.

![图形用户界面 低可信度描述已自动生成](../../attachments/ec859af1d041089310cf91dfdb84b9dc.png)

一个 `Pod` 一般指一个容器实例，如图中 `rocketchat`，`db`

但一个 `Pod` 也可以有多个容器实例，在 `Docker` 中的这种情况，如 `teamgram-server`，称为容器组。

容器组中的容器共享生命周期、资源、本地网络和存储卷，在 `K8S` 中也称为 `Pod`。**容器组有公共的 `IP` 地址，两个容器监听不同的端口。**

`dapr` 中的边车，就是很好的例子，边车与应用程序位于同一个 `Pod` 内。
[kubernetes里的各种port解惑 - 周国通 - 博客园](https://www.cnblogs.com/tylerzhou/p/11023157.html)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tomcat-server
spec:
  type: NodePort
  ports:
    - targetPort: 8080  # Pod的外部访问端口，也就是容器设定需暴露的container port。port和nodePort的数据通过这个端口进入到Pod内部，Pod里面的containers的端口映射到这个端口，提供服务
      port: 11111  # [集群内部]service暴露在cluster ip上的端口，其他容器通过<cluster ip>:port访问服务,通过此端口集群内的服务可以相互访问
      nodePort: 30001 # [集群外部]Node节点的端口，<nodeIP>:nodePort 是提供给集群外部客户访问service的入口
  selector:
    tier: frontend
```

## Service

一个 `Pod` 有一个自己的 `IP Address`，一个 `Pod` 搭配一个 `Service`，`Service` 管理 `Pod` 的 `IP`，`Pod` 挂掉 `IP` 也不会变。`Service` 分 `Internal service` 和 `external service`，即可被外部访问以及不可被外部访问的。

`Pod` 之间通过 `Service` 进行通信。

`Node` 中有多个 `Pod`，多个 `Service`，可以用一个 `Ingress` 用于外部路由 `Service`。

可以对 `Pod` 进行 `Replicate`，实现 `HA`（是对无状态的 `Pod`）。即使用一个新的 `Node`，运行一样的 `Pod`，然后这两个 `Pod` 之间使用同一个 `Service`，这时候，`Service` 也有 `Load balance` 的功能。这种功能可以通过 `Deployment` 组件实现。

而对于 `DB` 这类有状态的 `Pod` 进行 `Replicate`，需要使用 `StatefulSet` 组件实现。但是非常复杂，所以一般来说 `DB` 是部署在 `K8S Cluster` 之外的。

## Configuration

对于 `Pod` 的外部配置，可以使用 `ConfigMap` 或 `Secrets`。

## 容器权限
特权模式还有一个用户和用户组的配置。效果不一。

曾遇到 `opentelemetry collector` 无法收集 `pod` 的 `log` 问题，虽然启用了特权模式但仍无效，但以特定 `uid` 及 `gid` 启动则可以：
```yaml
securityContext:
  runAsUser: 0
  runAsGroup: 0
```
[find files with '/var/log/pods/*/*/*.log' pattern: open .: permission denied · Issue #33083 · open-telemetry/opentelemetry-collector-contrib](https://github.com/open-telemetry/opentelemetry-collector-contrib/issues/33083)

## 挂载模式

### 访问模式

- **ReadWriteOnce:**
    - 读写权限，只能被单个节点挂载
    - 也允许运行在同一节点上的多个 `Pod` 访问卷
- **ReadOnlyMany**
    - 只读权限，可以被多个节点挂载
- **ReadWriteMany**
    - 读写权限，可以被多个节点挂载
- **ReadWriteOncePod**
    - 读写权限，只能被单个 `pod` 挂载
    - 如果你想确保整个集群中只有一个 `Pod` 可以读取或写入该 `PVC`， 请使用 `ReadWriteOncePod` 访问模式。
    - 这只支持 `CSI` 卷以及需要 `Kubernetes 1.22` 以上版本。

### 挂载单个文件
默认挂载 `configMap` 时，`kubernetes` 会覆盖掉挂载的整个目录，哪怕使用 `items` 也会导致整个目录被覆盖，那么如何实现挂载单个文件而不覆盖整个目录呢。下面说一下 `kubernetes` 中如何挂载单个文件而不是整个目录。
实际上 `kubernetes` 本身提供了 `volumeMounts.subPath` 属性用于挂载单个文件而不是整个目录。
#### 问题
- 如果使用 `ConfigMap` 的 `subPath` 挂载为 `Container` 的 `Volume`，`Kubernetes` 不会做自动热更新
- `ConfigMap` 哪怕不使用 `subPath` 的挂载方式 `C#` 文件修改监听功能也没有触发，`Golang` 的正常，怀疑可能和自己的实现有关，因为还有其他事情没有继续跟进，下周准备继续跟进一下。