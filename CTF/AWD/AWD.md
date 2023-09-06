
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

备份到本地、以及源站上`/home/xxx`目录一份，不建议备份到`/tmp`，因为攻击者通过`/var/www/`攻击的访问权限是在`/tmp`以及`www`目录下

## 主机发现

AWD主机发现
    线下
        大家处于一个网段，根据ip划分
        nmap
    线上
        根据端口划分(web\ssh),可能有其他比赛
        scan.py

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

    区分选手和裁判：
        check
            down
            waf