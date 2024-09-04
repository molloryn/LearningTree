[gchq/CyberChef: The Cyber Swiss Army Knife - a web app for encryption, encoding, compression and data analysis (github.com)](https://github.com/gchq/CyberChef)

#### ASCII码互转
- `Encode Text  US-ASCII` 或 `From charcode radix 10`
- `To Charcode radix 10`：convert text to its ASCII charcode
#### 字符批量处理
Fork可以将其按分隔符批量处理
![](../../attachments/Pasted%20image%2020230718192015.png)



## Encoding

其中有`Decode text`和`Encode text`功能，含有各种字符集。
比如：
`IBM EBCDIC International (EBCDIC)`, `US-ASCII (ASCII)`


# 暴力破解

#### XOR暴力破解(XOR Brute Force)
当Key为1时，意味着对输入的每一个字符使用单字节异或
当Key为n(n>=2)个以上时，意味着对输入每n个字符使用n个字节异或。输入不足的部分忽略。


![](../../attachments/Pasted%20image%2020240901184252.png)

#### 凯撒密码暴力破解
![](../../attachments/Pasted%20image%2020230902211937.png)