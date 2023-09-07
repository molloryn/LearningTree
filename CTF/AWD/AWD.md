
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
tar -cvf xxx.tar ./*
tar -xvf xxx.tar -C /home/ctf/xxx

```

备份到本地、以及源站上`/home/xxx`目录一份，不建议备份到`/tmp`，因为攻击者通过`/var/www/`攻击的访问权限是在`/tmp`以及`/tmp/www/html`目录下
## 信息收集


```sh
id # 获取用户所在组。根据权限的不同，进行网站恢复、不死马的查杀。
ls -l /var/www/html # 获取权限 所属组 所属者 

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
find / -name 'xxx.php' 2 > /dev/null
```

## 主机恢复


```sh
# 以www-data用户执行 （比如使用蚁剑，或自建相关木马脚本然后通过网站执行）
kill -9 -1  # 终止你拥有的全部进程
rm -rf *
# 恢复
```


## 隐匿后门

### 上传

利用内置后门
`file_put_contents('xx', base64_decode(base64_encode('TrojansContent')))`
### 被利用

#### 加密码

- 写死密码
- 动态密码
- MD5
- RSA密码

### 被检测被删除
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
免杀
##### 内存马
自身执行一次后，把自己删除，自身已经驻留在内存之中，可以不断执行一些复活等操作，来进行权限维持。
