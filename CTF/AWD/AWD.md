
# 环境

## 靶机环境

如果没有安装python，可以尝试安装到有权限写入的目录：
```sh
./configure --prefix=/home/ctf/python32  
make  
make install
```



# 命令
通常用于自动化脚本编写。
#### scp
基于ssh

```sh
# 将本地文件上传到 远程ssh server 端口为12345的目录上 
scp -P 12345 /home/1.zip root@127.0.0.1:/home
```


# 防御

## 源码备份

```sh
# zip
zip -r xxx.zip ./*
unzip xxx.zip -d

# tar
tar -cvf xxx.tar ./* -p # -p 保持文件权限一致
tar -xvf xxx.tar -C /home/ctf/xxx

```

备份到本地、以及源站上`/home/xxx`目录一份，不建议备份到`/tmp`，因为攻击者通过`/var/www/`攻击的访问权限是在`/tmp`以及`/tmp/www/html`目录下
## 信息收集


```sh
id # 获取用户所在组。根据权限的不同，进行网站恢复、不死马的查杀。
ls -l /var/www/html # 获取权限 所属者 所属组
# 权限格式：所有者权限 组权限 其他用户权限


```


### 主机发现

AWD主机发现
    线下
        大家处于一个网段，根据ip划分
        nmap
    线上
        根据端口划分(web\ssh),可能有其他比赛
        scan.py

区分选手和裁判：
        check
        down
        waf
```python
import requests

ip = 'http://61.147.171.104:'

for i in range(51200,51230):
    url = ip + str(i)
    try:
        req = requests.get(url, timeout=1).text
        if '网络探测demo' in req:
            print(url + '    success')
        else:
            print(url + '    fail')
    except:
        print(url + '    error')

```

### 目录发现

```sh
# 2是标准错误输出，输出到一个不存在的路径，可以随便填，优雅点就填下面的
find / -name 'xxx.php' 2> /dev/null  # 注意2>必须写在一起

```

## 主机恢复


```sh
# 以www-data用户执行 （比如使用蚁剑，或自建相关木马脚本然后通过网站执行）
kill -9 -1  # 终止你拥有的全部进程
rm -rf *
# 恢复
```


## 攻击

### 上传

利用内置后门
`file_put_contents('xx', base64_decode(base64_encode('TrojansContent')))`
### 防利用

#### 后门加密码

- 写死密码
- 动态密码
- MD5
- RSA密码
#### 流量混淆

难以让别人通过流量监控知道如何攻击

### 防检测防删除
#### 混淆
- 文件名混淆、或假名
#### 困难删除
- 命名：使用`     .php`或`-- -.php`等不太好手动删除的文件（但可以直接通过FTP工具删除）
- 

#### 时间变化
 有三种时间：
- atime (access time)
- ctime (change time)
- mtime (modified time)

```sh
stat <file> # 可以查看文件状态信息，包括三种时间。
touch -r <oldFile> <newFile> # 可以将`oldFile`和`newFile`时间戳同步，但changeTime似乎会修改失败。
```

#### 不死马
首先需要www-data用户对目录有写权限去上传文件。
免杀
##### 内存马
自身执行一次后，把自己删除，自身已经驻留在内存之中，可以不断执行一些复活等操作，来进行权限维持。

**防御方法**
- 使用内存马的相同的用户，运行`kill -9 -1 && rm -rf *`
- 使用竞争，写相同的复活脚本，同名文件，对内容进行篡改，然后复活时间小于等于内存马，可以使其失效。

### 反弹Shell
正向连接是指由客户端向服务端发起连接请求，而反向连接则是相反。在反弹shell (reverse shell) 中，由客户端监听指定的 TCP/UDP 端口，由服务端发起连接请求到该端口，建立起shell连接。与常见的 Telnet、ssh等shell相比，不同的是由服务端主动发起。
#### 适用情况
- 由于防火墙等的控制访问策略，客户端无法访问服务端，但服务端可以向外网发起请求
- IP会动态变化，攻击机无法直接连接
- 向局域网中的主机散播木马病毒等，网络环境未知的情况下
- 服务端的AV对正向连接的流量检测非常严格，但是对出网流量校验不足
#### 过程
不同的操作系统可能不太一样

```sh
# 客户端监听端口
nc -l 9999

# Netcat 是 Linux 系统下常用的网络工具，当然也有Windows版本，在 Linux 下使用命令 `nc`，前面的端口监听就是使用 Netcat
```
> 在不同的反弹shell方式中，都需要客户端监听，下文将不再赘述，仅讲述不同的反弹方式



# Linux易忽略点

#### 反引号命令执行 \`\` 
在Linux中，反引号（\`）用于将其中的命令执行结果以参数形式（非字符串）展开（或者说是将结果替换到命令），因此它是最先执行的，展开后才轮到bash去处理整段命令。
```sh
echo `echo 123` # 执行完里面的内容，直接以参数展开，然后变为了echo 123，这时才轮到babsh执行这段命令
echo $(ls) # 除了反引号，也可以使用$()来实现相同的效果
```


#### 重定向符号

- `>`：将echo命令的输出`覆盖写入`到指定文件，如果文件不存在则创建文件。
- `>>`：将echo命令的输出`追加写入`到指定文件，如果文件不存在则创建文件。
- `2>`：将echo命令的错误输出覆盖写入到指定文件。
- `2>>`：将echo命令的错误输出追加写入到指定文件。
- `&>` 或 `&>>`：将echo命令的标准输出和错误输出合并后写入到指定文件。
```sh
echo "How are you?" >> output.txt # 命令的输出追加到文件末尾
echo "Error occurred." 2> error.txt # 错误输出写入文件
```

