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

[集群管理](集群管理.md)