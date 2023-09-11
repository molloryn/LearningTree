# 弱类型

![](../../attachments/Pasted%20image%2020230911233713.png)
> 1. 注意 `php8.0` 之后，字符串不再自动转为数字与数字比较。
> 2. 里面`"php"`指的是字符串
> 3. 下面都指的是`php7及以下`

弱类型中，最容易忽视的是：
```php
var_dump("admin"==0); //true
var_dump("1admin"==1); //true 与数字比较时，自动转化前面的字符串数字为数字，碰到其他字符结束
var_dump("admin1"==1); //false
var_dump("admin1"==0); //true
var_dump("0e123124"=="0e44912"); //true 科学计数法时(0e开头) 视为相等，php8也适用
```

# 哈希绕过

`md5(array())`将等于 `null`，因为函数接收了非有效参数输入。