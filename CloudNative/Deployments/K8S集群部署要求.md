# 节点要求

1. 同步时钟服务器时间
2. 永久关闭 Selinux
3. 永久关闭虚拟内存交换
4. 永久关闭防火墙
5. 创建`/etc/resolv.conf` 文件，并配置 DNS 服务器 IP 地址。
6. 执行“vi /etc/security/limits.conf”修改系统最大句柄数限制,在文件中添加“* soft nofile 65535”和“* hard nofile 65535”。再重启服务器。
![](../../attachments/Pasted%20image%2020230814163237.png)
7.  将五个服务器中“/etc/yum.repos.d/openEuler.repo”重命名备份，然 后将“配 置文 件 \openEuler.repo ” 拷贝 到5个 服务器 的“/etc/yum.repos.d”路径下。
这是配置yum仓库地址
注意这个目录下不要有多个`.repo`文件，它似乎只会识别一个去处理。
验证是否配置成功可以尝试`yum list`

8. 在服务器分别执行`dnf clean all`，`dnf makecache`命令
The commands `dnf clean all` and `dnf makecache` are related to the DNF package manager, which is used to install, update, and remove software packages on RPM-based Linux distributions. Here is a brief explanation of what they do:

- `dnf clean all` removes all cached files generated from the repository metadata. This can help to solve package installation problems that arise from corrupt or outdated metadata. It also frees up some disk space by deleting unnecessary files1
- `dnf makecache` downloads and caches metadata for enabled repositories. This can speed up the package installation process by avoiding unnecessary downloads. It also ensures that the metadata is up to date and consistent with the remote repositories2

9. 执行 `yum install -y conntrack socat tar`

`Conntrack` is a Linux kernel module that enables tracking of network connections. It allows the kernel to keep track of all currently active network connections and provides a way to manipulate them through a user-space interface.
`Socat` is a command-line utility that establishes two bidirectional byte streams and transfers data between them. It can be used for a variety of purposes, such as debugging, testing, and network exploration. It is often used as a replacement for the `netcat` utility in Linux systems.

10. 重启群集每个节点的服务器。


# 防火墙配置

> OpenEuler

- 运行 `systemctl stop firewalld.service` 命令来停止防火墙服务。
- 运行 `systemctl disable firewalld.service` 命令来禁用防火墙服务的自动启动。
- 运行 `systemctl status firewalld.service` 命令来查看防火墙服务的状态，确认已经关闭。

# 时钟配置
多服务器之间通信要保持时钟一致，特别是内网无法连接外部时间时。
## 服务端配置

1. vi /etc/chrony.conf

2. 在配置文件里添加以下配置

   server xxx.xx.xx.xx(服务端 IP) iburst (本机配置,⾃⼰既是服务端⼜是客⼾端) 允许所有连接
   allow
![](../../attachments/Pasted%20image%2020230814155326.png)
3. 按顺序执行以下语句

修改时区与同步设置

`timedatectl set-timezone 'Asia/Shanghai' timedatectl set-ntp 1`

重启服务

`systemctl enable chronyd systemctl restart chronyd 查看状态`

`systemctl status chronyd`

![](../../attachments/Pasted%20image%2020230814155551.png)

## 客户端配置

> **客户端同步时钟前必须关闭防火墙及 selinux**

1. vi /etc/chrony.conf 修改server
    
    server 188.2.93.120 iburst （IP 为时钟服务器 ip）
    

![](../../attachments/Pasted%20image%2020230814155526.png)

1. systemctl enable chronyd (3)systemctl restart chronyd 查看状态
    
2. systemctl status chronyd
    
3. 最后输入“timedatectl”命令，看到如下图所示则时钟同步成功。

![](../../attachments/Pasted%20image%2020230814155516.png)



# DNS服务器配置

配置好后可以使用`dig www.xxx.com`命令，随便输入一个网站，它会读取`/etc/resolv.conf`下`nameserver`的配置然后尝试发送请求解析，如果有回应，说明DNS配置正确。
`resolve.conf`文件内容如下：

```sh
nameserver 188.xxx.xxx.xxx
```

可以运行`systemctl restart NetworkManager`命令重启使其强制生效。
# 系统资源限制配置

The default value of max count of system handle defined in Linux depends on the type of handle you are referring to. There are different kinds of handles, such as file descriptors, processes, sockets, memory mappings, etc. Each of them has a different limit and a different way to change it.

For example, the maximum number of file descriptors per process can be checked by using the command `ulimit -n`1. The default value is usually 1024, but it can be changed by editing the `/etc/security/limits.conf` file1. The maximum number of file descriptors per system can be checked by using the command `cat /proc/sys/fs/file-max`2. The default value depends on the amount of available memory, but it can be changed by using the command `sysctl fs.file-max=number`2.

Another example is the maximum number of processes per user, which can be checked by using the command `ulimit -u`3. The default value is usually 4096, but it can be changed by editing the `/etc/security/limits.conf` file3. The maximum number of processes per system can be checked by using the command `cat /proc/sys/kernel/pid_max`4. The default value is usually 32768, but it can be changed by using the command `sysctl kernel.pid_max=number`4.

A third example is the maximum number of TCP/IP connections per system, which can be checked by using the command `cat /proc/sys/net/ipv4/ip_local_port_range`. The default value is usually 32768 to 61000, but it can be changed by using the command `sysctl net.ipv4.ip_local_port_range=\"min max\"`. There are also other parameters that affect the TCP/IP connections, such as `net.ipv4.tcp_fin_timeout`, `net.ipv4.tcp_tw_recycle`, and `net.ipv4.tcp_tw_reuse`.

A fourth example is the maximum number of memory mappings per process, which can be checked by using the command `cat /proc/sys/vm/max_map_count`. The default value is usually 65530, but it can be changed by using the command `sysctl vm.max_map_count=number`.

These are just some examples of system handles and their limits in Linux. There may be other types of handles that have different limits and ways to change them. You can find more information about them by searching on the web or reading the Linux documentation. I hope this helps you understand the concept of system handles and their limits in Linux. 😊

# openeuler 软件源

## 可参考（[搭建 repo 服务器 (openeuler.org)](https://docs.openeuler.org/zh/docs/22.03_LTS/docs/Administration/%E6%90%AD%E5%BB%BArepo%E6%9C%8D%E5%8A%A1%E5%99%A8.html)）

1、将“openEuler-22.03-LTS-everything-x86_64-dvd.iso”镜像拷贝到服务器的root 目录下。

2、按顺序执行以下命令：

（1）mkdir -p /mnt/iso

（ 2    ） mount      openEuler-22.03-LTS-everything-x86_64-dvd.iso

/mnt/iso/

（3）mkdir /opt/openeuler_repo/

（3） cp -r /mnt/iso/* /opt/openeuler_repo/

（4）   im /etc/yum.repos.d/openEuler.repo

3、在 openEuler.repo 文件里写入以下内容


```config
[base] 
name=base
baseurl=file:///opt/openeuler_repo enabled=1
gpgcheck=1
gpgkey=file:///opt/openeuler_repo/RPM-GPG-KEY-openEuler
```


3、执行“yum -y install nginx”安装 nginx，将/etc/nginx/nginx.conf 文件重命名备份，然后将“nginx.conf”拷贝到/etc/nginx 路径下。
`nginx.conf`文件内容修改如下：

```nginx
user  nginx;
worker_processes  auto;                          # 建议设置为core-1
error_log  /var/log/nginx/error.log  warn;       # log存放位置
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;                 # 服务器名（url）
        client_max_body_size 4G;
        root         /usr/share/nginx/repo;                 # 服务默认目录

        location / {
            autoindex            on;            # 开启访问目录下层文件，这里一定要记得开，不然会有403Forbidden问题
            autoindex_exact_size on;
            autoindex_localtime  on; 
        }

    }
}
```

可以用软链接方式将repo文件夹链接到nginx目录下
`ln -s /opt/openeuler_repo /usr/share/nginx/repo`


4、依次执行“systemctl enable nginx”，“systemctl start nginx”， “systemctl status nginx”，看到下图则nginx 启动成功。

5、打开浏览器访问本机 IP 地址，出现下图则部署成功。
