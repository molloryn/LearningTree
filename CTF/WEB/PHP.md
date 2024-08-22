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



# 安全配置

#### allow_url_include
The term "allow_url_include" refers to a PHP setting that controls whether or not remote file inclusion is allowed. When set to "On", it allows PHP scripts to include files from remote locations using a URL. This can be useful for accessing resources or libraries hosted on other servers, but it can also pose a security risk if not used carefully.

#### magic_quotes_gpc
"magic_quotes_gpc" is a security feature in PHP that automatically adds escape characters to certain characters in user input data, such as quotes and backslashes, to prevent SQL injection attacks. However, this feature is now deprecated and should not be relied upon as the sole means of preventing such attacks. Therefore, it is recommended to turn it off and use other security measures, such as prepared statements and input validation.


# 伪协议

[文件包含&PHP伪协议利用_file_get_contents()支持的协议_红云谈安全的博客-CSDN博客](https://blog.csdn.net/qq_51524329/article/details/121439731)

[phpfilter的妙用_拓海AE的博客-CSDN博客](https://blog.csdn.net/weixin_44576725/article/details/124177555)


#### php://filter

可用于 `include`等
```
page=php://filter/convert.base64-encode/resource=index.php
源码泄露？

```


[关于 CTF 中 php 考点与绕过那些事的总结_ctf php-CSDN博客](https://blog.csdn.net/Myon5/article/details/136455078)
# 绕过

## 函数绕过

### array_search
array_search(value,array,strict) 函数用于在数组中搜索指定的值，如果找到则返回其键（默认键是数组下标，但是如果是字典则返回它的key），否则返回 false。
如果第三个参数不传，默认为false，其实就是使用`==`的方式去遍历数组，这时候就可以使用弱类型的绕过了。

