# WAF(Web Application Firewalls)绕过

## Unicode字符

`print("\u0068")` 是python语言自己的特性

## 十六进制字符

`print("\x80")` 

```python
str = b'\xe6\x82\xa8\xe7\x9a\x84\xe4\xbf\xa1\xe6\x81\xa
# Decode the hex data assuming
str.decode('utf-8')
```


# 反序列化

[从零开始python反序列化攻击：pickle原理解析 & 不用reduce的RCE姿势 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/89132768)


# RCE (Remote Code Execution)


# 常用内置函数

- `type()` 获取类型
- `chr()` function can be used to convert an ASCII **(and Unicode)** ordinal number to a character (the `ord()` function does the opposite).
- `ord()` can get the unicode character value in base 10.
- `bytes.fromhex(str)` convert hex to bytes. `b'xx'.hex()`on byte strings to get the hex representation.
- `bin()`将数字转成二进制字符串表示，以`0b`开头
- `int(number, 2)` 将二进制字符串数字表示从二进制转换为整数(10进制)，支持`0b`开头的字符串。
- `' '.join()` 以空格分割字符串数组

### 常用库函数

#### import base64
- `base64.b64encode()` 

#### from pwn 

##### from pwn import xor



#### from Crypto.Util.number import *
> pip install PyCryptodome

密码学中，常将数据由bytes转为长整型数据。

```python
message: HELLO  
ascii bytes: [72, 69, 76, 76, 79]  
hex bytes: [0x48, 0x45, 0x4c, 0x4c, 0x4f]  
base-16: 0x48454c4c4f   # 16进制数据，是由十六进制数组拼接得到
base-10: 310400273487   # 10进制数据，是由十六进制数字转为十进制得到
```

以上过程在`PyCryptodome`库中函数：`bytes_to_long()` and `long_to_bytes()`已经实现。

# 数字

## 二进制


```python
# 可用0b开头代表二进制数字

# AND 操作
a = 0b101 
b = 0b111 
c = a & b 
# 0b101 AND 0b111 = 0b101

# OR 操作
a = 0b101 
b = 0b110 
c = a | b 
# 0b101 OR 0b110 = 0b111

# XOR 操作
a = 0b101 
b = 0b110 
c = a ^ b 
# 0b101 XOR 0b110 = 0b011

a = 0b101 
b = ~a 
# NOT 0b101 = 0b...111111111111111111101010
print(bin(b)) # 输出结果为：-0b110
# 在Python中，NOT运算符执行按位取反操作，由于整数在计算机内部使用补码表示，因此NOT 运算符将一个正整数转换为负整数，并在二进制表示中添加前导1。

```


# 字符串

## **`r' '`: 原始字符串**

`r`前缀表示原始字符串（raw string），它会取消字符串中的转义字符（如`\n`、`\t`）的特殊含义。原始字符串适用于需要保留转义字符原始形式的情况，如正则表达式、文件路径等。

## **`b' '`: 字节字符串**

`b`前缀表示字节字符串（bytes string），它用于处理二进制数据，而不是文本数据。字节字符串是不可变的，通常用于处理图像、音频、网络协议等二进制数据。

字节字符串来表示原始数据，经过特定编码转换为普通字符串

```python
# 字符串与字节字符串的区别
text = 'Hello'
binary_data = b'Hello'

print(type(text))  # 输出 <class 'str'>
print(type(binary_data))  # 输出 <class 'bytes'>
```

## **`u' '`: Unicode字符串**

> 在Python 3中，无需使用`u`前缀，普通字符串即为Unicode字符串。

`u`前缀表示Unicode字符串，它用于处理Unicode编码的文本数据。在Python 3中，所有的字符串都是Unicode字符串，因此很少需要使用`u`前缀。在Python 2中，`u`前缀用于表示Unicode字符串。

```python
# Unicode字符串与普通字符串的区别（Python 2示例）
text = 'Hello'
unicode_text = u'Hello'

print(type(text))  # 输出 <type 'str'>
print(type(unicode_text))  # 输出 <type 'unicode'>
```

在Python 2中，Unicode字符串与普通字符串是不同的数据类型，用于区分文本编码。

## **`f' '`: 格式化字符串**

`f`前缀表示格式化字符串（formatted string），它用于在字符串中嵌入表达式的值。在格式化字符串中，可以使用大括号`{}`来引用变量或表达式，并将其值插入字符串中。

```python
# 使用f前缀创建格式化字符串
name = 'Alice'
age = 30
greeting = f'Hello, my name is {name} and I am {age} years old.'
print(greeting)
```

在上述示例中，`f`前缀表示格式化字符串，大括号`{}`内的表达式会被计算并插入到字符串中。




# 自制脚本收集

#### 已知XOR后部分明文恢复部分密钥

因为异或性质是异或相同后抵消，那可以直接恢复出已知明文的密钥。

```python
data = '0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104'
decoded_data = bytes.fromhex(data)
plain_text = 'crypto{'
key = ''.join((chr(decoded_data[i] ^ ord(plain_text[i]))) for i in range(7))
print(key) # 结果是myXORke，所以可以直接猜测最后XOR的密文是myXORkey
```
