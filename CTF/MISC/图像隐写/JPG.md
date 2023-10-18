# 工具

#### stegdetect
用于检测JPG中常见的工具隐写。**但检测不一定准确**，因此如果输出检测失败也不能放弃对那些工具的嫌疑。 
支持的隐写工具有：`jsteg`, `outguess`, `jphide`, `invisible secrets`
```sh
stegdetect -s 100 # 设置容错，尝试调整可以获得不一样的结果
```

####  jphs(jphide)
使用stegdetect出现的提示的工具，用于有密码（也可能没密码）的jpg隐写文件，目前网上能下载到的带界面的为`jphswin`


#### silenteye 
可能**无需密码** 可以尝试看看有没有什么信息，一般会有提示，如“眼睛”。


# 结构

|标记|十六进制|名称|说明|注释|
|---|---|---|---|---|
|SOI|FFD8|Start Of Image|图像开始|  |
|**SOF0**|FFC0|Start Of Frame (Baseline DCT)|帧开始（基线DCT）|包含图像的宽度，高度，颜色分量数和采样因子|
|SOF2|FFC2|Start Of Frame (Progressive DCT) |帧开始（渐进DCT）|同上，但使用渐进编码|
|DHT|FFC4|Define Huffman Table(s)|定义霍夫曼表|包含霍夫曼编码的符号和长度|
|DQT|FFC5|Define Quantization Table(s)|定义量化表|包含量化系数的矩阵|
|DRI|FFDD|Define Restart Interval|定义重启间隔|包含重启间隔的MCU数|
|SOS|FFDA|Start Of Scan|扫描开始|包含扫描的颜色分量数，霍夫曼表选择和谱选择参数|
|RSTn|FFD0 to FFD7|Restart|重启 |无有效载荷，用于同步解码器|
|APPn|FFE0 to FFEF|Application-specific|应用程序特定|包含不同应用程序的元数据，如EXIF，JFIF，XMP等|
|COM|FFEE|Comment|注释|包含任意文本|
|EOI|FFD9|End Of Image|图像结束 |  |


### SOFx

SOFx is a generic term for **Start Of Frame** markers in JPEG files. There are different types of SOF markers, such as SOF0, SOF1, SOF2, etc., that indicate different encoding methods and parameters for the image data.

这里含有图片宽高信息，因此宽高隐写要找到这个块所在位置。

![](../../../attachments/Pasted%20image%2020230803213143.png)
