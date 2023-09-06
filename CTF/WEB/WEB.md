# 工具


### 查看网页源码

除了F12的功能，还可以通过`view-source:http://foobar.com`方式查看源码，这种方式代码已经全部展开，粗略浏览所有代码效率高。

不过F12也有此功能，但对大量数据处理较卡顿。
**Expand / collapse node and all its children**
`Ctrl + Alt + Click` on arrow icon
To expand all elements, use the `<html>` element's arrow icon in the shortcut.

# 绕过方式
## 双写绕过

比如过滤给定字符串内关键字：`CTF`，只过滤一次，但想要最终结果还保留有该关键字，就可以采用双写绕过：
`CCTFTF`

## 双重编码绕过


## 响应绕过

### ROT13

比如对响应包含`flag`的关键字做了过滤，我们直接对其做ROT13等类似算法，即能绕过响应过滤。