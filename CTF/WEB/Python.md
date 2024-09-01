# WAF(Web Application Firewalls)绕过

## Unicode字符

`print("\u0068")` 是python语言自己的特性

## 十六进制字符

`print("\x80")` 


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

