# Docker环境安装

## Centos (yum)

在Linux系统上安装最新版Docker和Docker Compose，通常需要先卸载旧版本，然后通过官方仓库安装最新版本。以下是完整的安装步骤：

### 具体命令

```bash
# 1. 卸载旧版本Docker（如果已安装）
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 2. 安装必要的依赖包
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

# 3. 设置Docker稳定版仓库
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 4. 安装最新版Docker引擎、CLI和Containerd
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 5. 启动Docker服务并设置开机自启
sudo systemctl start docker
sudo systemctl enable docker

# 6. 验证Docker安装是否成功
sudo docker run hello-world

# 7. 安装最新版Docker Compose（如果需要）
# 注意：上面的步骤已通过插件方式安装了Docker Compose
# 如需独立安装最新版，可使用以下命令
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 8. 验证Docker Compose安装
docker compose version  # 对于插件方式
docker-compose version  # 对于独立安装
```

### 注意事项

1. **用户权限**：默认情况下，Docker命令需要root权限运行。你可以创建一个docker用户组并将当前用户添加进去，避免每次都使用sudo：

```bash
sudo groupadd docker  # 创建docker用户组
sudo usermod -aG docker $USER  # 将当前用户添加到docker组
newgrp docker  # 刷新用户组
```

2. **国内镜像加速**：为了提高下载速度，可以配置Docker使用国内镜像源：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

3. **更新Docker**：要更新Docker，只需再次运行安装命令：

```bash
sudo yum update docker-ce docker-ce-cli containerd.io
```




## 设置仓库地址

在 CentOS 系统上设置 Docker 仓库地址，主要涉及配置 Docker 的镜像源和私有仓库。下面我将介绍这两种配置方法。

### 配置 Docker 镜像源

Docker 默认从 Docker Hub 拉取镜像，但国内访问可能较慢，建议配置国内镜像源：

1. 创建或编辑 Docker 配置文件
2. 添加国内镜像源地址
3. 重启 Docker 服务

以下是配置步骤：

```bash
# 1. 创建或编辑 Docker 配置文件
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
          "https://xqy377lu.mirror.aliyuncs.com",
          "https://docker.zhai.cm",
    "https://a.ussh.net",
    "https://hub.littlediary.cn",
    "https://hub.rat.dev",
    "https://atomhub.openatom.cn",
    "https://docker.m.daocloud.io",
    "https://docker.1ms.run",
    "https://dytt.online",
    "https://func.ink",
    "https://lispy.org",
    "https://docker.xiaogenban1993.com",
    "https://docker.mybacc.com",
    "https://docker.yomansunter.com",
    "https://dockerhub.websoft9.com",
    "https://mirror.ccs.tencentyun.com"
  ]    
}
EOF

# 2. 重启 Docker 服务
sudo systemctl daemon-reload
sudo systemctl restart docker

# 3. 验证配置是否生效
docker info | grep -i mirror
```

### 配置私有 Docker 仓库

如果需要使用私有 Docker 仓库，需要添加仓库地址：

```bash
# 编辑 Docker 配置文件
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn"
  ],
  "insecure-registries": [
    "your-private-registry:5000"  # 替换为你的私有仓库地址
  ]
}
EOF

# 重启 Docker 服务
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 注意事项

1. **镜像源选择**：上面列出了几个常用的国内镜像源，你可以根据自己的网络情况选择合适的镜像源。

2. **私有仓库安全**：如果私有仓库没有使用 SSL 证书，需要在 `insecure-registries` 中添加仓库地址。在生产环境中，建议使用带有 SSL 证书的私有仓库。

3. **配置文件格式**：`daemon.json` 文件必须是有效的 JSON 格式，否则 Docker 将无法启动。

完成上述配置后，Docker 将从指定的镜像源拉取镜像，或者可以访问配置的私有仓库。