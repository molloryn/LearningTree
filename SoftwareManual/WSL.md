# WSL

可以通过windows terminal的配置，将启动目录改到与运行目录相同，方便直接使用linux相关工具

## 文件管理
可以直接在windows按下面的方式访问：
`\\wsl$`
`\\wsl.localhost\kali-linux\root`

## 命令

<https://docs.microsoft.com/zh-cn/windows/wsl/basic-commands>

|                                                                                                           |                                                                                                                                                                                                                                                                       |
|-----------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| wsl --list --online                                                                                       | 列出可用的linux发行版                                                                                                                                                                                                                                                 |
| wsl --install -d kali-linux                                                                               | 下载kali-linux并安装启动                                                                                                                                                                                                                                              |
| wsl --shutdown                                                                                            | 终止所有子系统                                                                                                                                                                                                                                                        |
| wsl --terminate docker-desktop-data                                                                       | 终止指定的子系统, 如 docker-desktop-data                                                                                                                                                                                                                              |
| wsl --export docker-desktop-data F:/WSL/docker-desktop-data/docker-desktop.tar                            | 将子系统导出为tar包                                                                                                                                                                                                                                                   |
| wsl --unregister docker-desktop-data                                                                      | 使用wsl命令注销并删除子系统                                                                                                                                                                                                                                           |
| wsl --import docker-desktop-data F:/WSL/docker-desktop-data F:/WSL/docker-desktop-data/docker-desktop.tar | 重新导入子系统到指定目录，然后tar包可以删除了                                                                                                                                                                                                                         |
| wsl -l -v                                                                                                 | 列出当前安装的wsl列表，以及版本信息                                                                                                                                                                                                                                   |
| cat /etc/resolv.conf \| grep nameserver                                                                   | WSL 每次启动的时候都会有不同的 IP 地址，所以并不能直接用静态的方式来设置代理。WSL2 会把 IP 写在 /etc/resolv.conf 中                                                                                                                                                   |
| wsl -d(--distribution) \<Distribution Name\> --user \<User Name\>                                         | 若要通过特定用户运行特定 Linux 发行版，请将 \<Distribution Name\> 替换为你首选的 Linux 发行版的名称（例如 Debian），将 \<User Name\> 替换为现有用户的名称（例如 root）。 如果 WSL 发行版中不存在该用户，你将会收到一个错误。 若要输出当前用户名，请使用 whoami 命令。 |
| wsl --set-default \<Distribution Name\>                                                                   | 设定默认打开的Linux发行版                                                                                                                                                                                                                                             |

### WSL连接宿主机代理

<https://zinglix.xyz/2020/04/18/wsl2-proxy/>

新版本WSL遇到问题的：
[Accessing network applications with WSL | Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/networking#auto-proxy)
#### 脚本


```shell
#!/bin/sh
hostip=$(cat /etc/resolv.conf | grep nameserver | awk '{ print $2 }')
wslip=$(hostname -I | awk '{print $1}')
port=<PORT> # 需要自行更改为proxy端口

PROXY_HTTP="http://${hostip}:${port}"
set_proxy(){
    export http_proxy="${PROXY_HTTP}"
    export HTTP_PROXY="${PROXY_HTTP}"
    export https_proxy="${PROXY_HTTP}"
	export HTTPS_PROXY="${PROXY_HTTP}"
git config --global http.proxy "${PROXY_HTTP}"
git config --global https.proxy "${PROXY_HTTP}"
}
# python如果无效的话，就在python执行的命令后加 --proxy=http://xxxxx吧
unset_proxy(){
    unset http_proxy
    unset HTTP_PROXY
    unset https_proxy
	unset HTTPS_PROXY
git config --global --unset http.proxy
git config --global --unset https.proxy
}



test_setting(){
    echo "Host ip:" ${hostip}
    echo "WSL ip:" ${wslip}
    echo "Current proxy:" $https_proxy
}

if [ "$1" = "set" ]
then
    set_proxy
elif [ "$1" = "unset" ]
then
    unset_proxy
elif [ "$1" = "test" ]
then
    test_setting
else
    echo "Unsupported arguments."
fi

alias proxy="source /xxx/proxy.sh"
```

另外可以在 `~/.bashrc` 中选择性的加上下面两句话，记得将里面的路径修改成你放这个脚本的路径。

```shell
alias proxy="source /xxx/proxy.sh" # 可以为这个脚本设置别名 proxy，这样在任何路径下都可以通过 proxy 命令使用这个脚本了，之后在任何路径下，都可以随时都可以通过输入 proxy unset 来暂时取消代理。

/xxx/proxy.sh set # 在每次 shell 启动的时候运行该脚本实现自动设置代理，这样以后不用额外操作就默认设置好代理啦~
```

注意，这代理不适用于某些不关注系统环境变量的程序，比如apt，firefox等。

默认情况下，WSL2是无法ping通HOST的，但能ping通宿主机，需要设置相应的防火墙规则使其支持ping通HOST。

如若遇到vEthernet 无法连接互联网的情况，可以通过联通主机的代理进行外网访问。

解决方案是直接重启电脑。猜测是hype-v的端口随机占用有概率导致无法连接问题

## 问题

### zsh语法高亮非常慢

输入第一块命令的时候，WSL2的zsh语法高亮特别慢，通过排查 `~/.zshrc`可以发现是`zsh-syntax-highlighting.zsh`的问题，遂上Github发现问题：
[syntax highlighting is super slow in WSL2 · Issue #790 · zsh-users/zsh-syntax-highlighting (github.com)](https://github.com/zsh-users/zsh-syntax-highlighting/issues/790)

其中有一个临时解决方案，禁用掉某个wsl2的功能（似乎是可以将windows的环境变量运用到wsl中，这也导致docker之类的用不了了）。

I solved this by excluding windows directories from `$PATH` by adding following in `/etc/wsl.conf`. Create the file if it doesn't exist

```
[interop]
appendWindowsPath = false
```

Then restart wsl with

```sh
wsl --shutdown
```
#### 添加需要的windows程序到环境变量

注意大小写
```zshrc
path+=( 
/mnt/c/Users/lesmo/AppData/Local/Microsoft/WindowsApps /mnt/c/Users/lesmo/AppData/Local/Programs/Microsoft VS Code/bin /mnt/c/Program Files/Docker/Docker/resources/bin /mnt/c/ProgramData/DockerDesktop/version-bin /mnt/c/WINDOWS 
)
```


# Kali

## 安装

<https://blog.csdn.net/niuiic/article/details/121908096>

<https://blog.csdn.net/tabactivity/article/details/125875242> 桌面版

`locale` 可以查看当前配置的语言环境

dpkg-reconfigure locales

下载语言包，默认Debian的语言包不是UTF-8格式的，所以中文会显示乱码。

可以下载英文UTF-8语言包，也能显示中文

默认JAVA安装在/usr/lib/jvm/目录下
### python环境

用Kex的包安装全套kali后，已经带有python3与python2，
默认`python`指向python3，而python2并不带`pip`

因此还要进行pip2的安装：
`wget https://bootstrap.pypa.io/pip/2.7/get-pip.py`

当分不清pip时，可以采用该命令：
`python2 -m pip ` 进行调用pip

### 问题

#### libcrypt.so.1: cannot open shared object file when upgrading from Stretch to Sid

<https://www.mail-archive.com/debian-bugs-dist@lists.debian.org/msg1818037.html>

```shell
$ cd /tmp
$ apt -y download libcrypt1
$ dpkg-deb -x libcrypt1_1%3a4.4.25-2_amd64.deb  .
$ cp -av lib/x86_64-linux-gnu/* /lib/x86_64-linux-gnu/
$ apt -y --fix-broken install
```

#### linux vim 上下左右只会出现ABCD

``` sh
$sudo apt-get remove vim-common
$sudo apt-get install vim
```

#### System has not been booted with systemd as init system
原因是你想用systemd命令来管理Linux上的服务，但你的系统并没有使用systemd，（很可能）使用的是经典的SysV init（sysvinit）系统。可以使用等效的命令：

![表格 描述已自动生成](../attachments/f9d570952d9f8596c790a2b67dfed94a.jpeg)

## Kex

基于Windows WSL的一个kali linux GUI界面。

To switch to Windows when using Win-KeX in window mode, you can press the **F8** key to open the client's context menu, which allows you to manage the client sessions, such as closing the client, switching between full screen and window, etc. You can disconnect from active sessions by pressing **F8** -> **Exit viewer**, this will close the client but leave the session running in the background. You can re-connect to a session by typing `kex --win --start-client`¹.

I hope that helps!

Source: Conversation with Bing, 6/30/2023

> (1) Win-KeX Window Mode | Kali Linux Documentation. https://www.kali.org/docs/wsl/win-kex-win/.
> (2) Win-KeX | Kali Linux Documentation. https://www.kali.org/docs/wsl/win-kex/.
### 问题

#### root似乎不是真正的root，某些仍需要sudo

使用su之后，发生了微小的变化。

通过echo \$PATH可以证明两者不是同一个账户（？

这就导致了很多蜜汁问题。

比如如果不使用sudo kex去启动kex，是无法打开火狐浏览器的

不同的环境变量。

用的不同的bash和zsh造成的问题。

#### VNC问题

VNC是用来连接虚拟桌面的，Linux端用vncserver，然后windows端用vnc client去连接。

出现了一个黑屏问题...
发现用esm模式连不上，是黑屏。
可以去wsl里用`sudo kex`连上（这是windows模式），很奇怪。

记录下vnc的几个命令：

```sh
# 如果不用，删除该目录可能出现 Device or resource busy 的问题
umount /tmp/.X11-unix

# 删除vnc临时文件
sudo rm -rf /tmp/.X11-unix

# 尝试启动一个新 vnc 会话
vncserver

# 查看当前所有会话
vncserver -list

# 移除第几个会话
vncserver -kill :1

# 重新设置vnc密码
vncpasswd 
```


# MyLinux

## 基于Docker

把docker镜像当作linux虚拟机来使用

```dockerfile
FROM centos:7
RUN yum install -y vim bash-com* openssh-clients openssh-server iproute cronie net-tools wget
RUN yum group install -y "Development Tools"
RUN yum clean all
RUN localedef -c -f UTF-8 -i zh_CN zh_CN.UTF-8 && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ENV LANG=zh_CN.UTF-8
docker build . -t mylinux
docker run -it -d -p 6666:22 --hostname mylinux1 --name mylinux1 --privileged=true mylinux /usr/sbin/init
# 以特权模式进入可以使用systemctl命令（特权模式必须运行/sbin/init，用于启动dbus-daemon）
docker exec -it mylinux1 /bin/bash # 进入容器
passwd root # 输入两次强制设置弱密码
```

## 安装常用包

wget //从Web下载文件

net-tools

sudo

## 安装自编译软件


```sh
# Git拉取，并编译需要的软件，比如bkcrack (源码其实可以拉去/usr/local/src大概)
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=install
cmake --build build --config Release
cmake --build build --config Release --target install

# 在工作目录下install文件夹内有二进制可执行文件
# 一般来说自编译的软件放在/usr/local/bin 目录下
# 一定要使用绝对路径进行软链接，不然无法识别
ln -s /root/xxx/bkcrack/install/bkcrack /usr/local/bin

# 此时已经生效了，如果没生效检查一下环境变量作用范围是不是有那个/usr/local/bin
echo $PATH


```

## Linux命令手册
[jaywcjlove/linux-command: Linux命令大全搜索工具，内容包含Linux命令手册、详解、学习、搜集。https://git.io/linux (github.com)](https://github.com/jaywcjlove/linux-command)
轻松通过 `docker` 部署 `linux-command` 网站。

```shell
docker pull wcjiang/linux-command
```

```shell
docker run --name linux-command --rm -d -p 9665:3000 wcjiang/linux-command:latest
```


## \~\/.zshrc

```sh
alias proxy="source /root/Scripts/proxy.sh"
alias ares="docker run -it --rm ares"
alias linuxman="docker run --name linux-command --rm -d -p 3002:3000 wcjiang/linux-command:latest | xargs echo http://localhost:3002"
alias ciphey="docker run -it --rm remnux/ciphey"
alias ctfwiki="docker run --name ctf-wiki --rm -d -p 3003:80 ctfwiki/ctf-wiki | xargs echo http://localhost:3003"
alias phpman="docker run --rm --name php8-manual -dp 3001:80 koubot/php8-manual-zh:v8.2;echo http://localhost:3001/indexes.functions.html"
alias cyberchef="docker run --name cyberchef -it --rm -dp 3004:8000 mpepping/cyberchef;echo http://localhost:3004"
alias phpdocker="docker run --rm -d -p 8080:80 -p 2222:22 --name my-apache-php-app -v "$PWD":/var/www/html php72-ctf"
eval $(thefuck --alias)

function volatility() {
  docker run --rm --user=$(id -u):$(id -g) -v "$(pwd)":/dumps:ro,Z -ti phocean/volatility $@
}
```


> 注意，自己在使用反引号、`$()`等操作时，在sh脚本中的执行效果和预期的的问题，它先执行那一部分作为结果替换到脚本中，因此自己在配置 `.zshrc`等文件时，注意执行顺序。`$PWD`等环境变量也一样，脚本第一次运行时候已经决定了结果。除非使用转义符。

Best of all, use a function instead of an alias. A function lets you write the command exactly as you would normally without _any_ extra quotes or escaping.