# VMWare


# 网络

## Bridged模式

这个似乎可以连接互联网，但不知为何无法SSH

## NAT模式

测试过可以SSH连接，有时候也能连互联网

## Host-only模式



# TroubleShooting

#### vmware workstation does not support nested virtualization on this host
![](../../attachments/Pasted%20image%2020230730175319.png)
	应该是勾选了下图的选项所致
	![](../../attachments/Pasted%20image%2020230730175440.png)
	取消勾选即可。