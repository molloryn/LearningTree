

# Base家族
## [base32786](https://github.com/qntm/base32768/blob/main/src/index.js)

`Binary-to-text encoding highly optimised for UTF-16`

**特征**:  中文字符,见提示

**测试数据:** 

```
媒腻㐤┖ꈳ埳
```



## [base100](http://www.atoolbox.net/Tool.php?Id=936)

又叫Emoji表情符号编码/解码

**特征**:   表情符号

**测试数据:** 

```
🏜🎳🍷🏜🎆🎈🏜🎮🎜🏜🍼🎮🏠🎒🍽🏜🎇🍿🐗👙👰🐗👣👜👦👥🐫🐧🐭🐷🐬🐩👧👦👡👠👜🐥👚👥
```

## [base16](https://cryptowikis.com/EncodeDecode/base16/)

使用16个ASCII可打印字符（数字0-9和字母A-F）对任意字节数据进行编码, 同hex

**特征**:  0-9A-F组成

**测试数据:** 

```
E5BC80E58F91E5B7A5E585B7E99B86E59088206279206C656F6E343036403532706F6A69652E636E
```

## [base2048](https://github.com/qntm/base2048/blob/main/src/index.js)

`Binary encoding optimised for Twitter`

**特征**:  见提示

**测试数据:** 

```
څƏൻڕ
```



## [base32](https://cryptowikis.com/EncodeDecode/base32/)

使用了ASCII编码中可打印的32个字符(大写字母A~Z和数字2~7)对任意字节数据进行编码.

**特征**: A-Z  2-7  =

**测试数据**:

```
4W6IBZMPSHS3PJPFQW36TG4G4WIIQIDCPEQGYZLPNY2DANSAGUZHA33KNFSS4Y3O
```

## [base36](https://en.wikipedia.org/wiki/Base36)

**特征**: 0-9 A-Z

**测试数据**:

```
MAHJV1X5YMIHRRDJ0HQLTZ0WNFLYDP0W01ME2E8MTAT3QNDXRXGNH7HJYAYY5Q
```

## [base45](https://github.com/patrikhson/base45)

**特征**:  0-9 A-Z ` `$%*+-./:

**测试数据**:

```
.1TCDGX6I$1TY/KV+G/NTI2HACIA443EFTVD-3E+Q6S%6SW6S9EGKDRZCTPC
```

## [base58](https://zh.m.wikipedia.org/zh-hans/Base58)

相比Base64，Base58不使用数字"0"，字母大写"O"，字母大写"I"，和字母小写"l"，以及"+"和"/"符号

**特征**:  1-9 A-H J-Z a-k m-z

**测试数据**: 

```
CR58UvatBfMNr917q5LwvMbAtrpuA5s3iCQe5eDivFqEz8LN1Ytu6aH
```

## [base58Check](https://en.bitcoin.it/wiki/Base58Check_encoding)

相比Base64，Base58不使用数字"0"，字母大写"O"，字母大写"I"，和字母小写"l"，以及"+"和"/"符号

**特征**: 1-9 A-H J-Z a-k m-z

**测试数据**:

```
2HhMuaDzQFGwDdVBD7S8MJRYAspzUi9zUGCLeQ1hsAdBGXBnq7FnKXsTc2iFp
```

## [base62](https://en.wikipedia.org/wiki/Base62)

相比base64 不使用"+"和"/"

**特征**:   0-9 A-Z a-Z

**测试数据**:

```
JJLamodrHXspZr5qUcfZYO3u0Gdw3fhzQqxO834pCgRbqcvOn3Vkju
```

## [base64](https://zh.m.wikipedia.org/wiki/Base64)

基于64个可打印字符来表示二进制数据的表示方法,3个字节可由4个可打印字符来表示

**特征**: 0-9 A-Z a-Z +/ =    长度为4的倍数

**测试数据**:

```
5byA5Y+R5bel5YW36ZuG5ZCIIGJ5IGxlb240MDZANTJwb2ppZS5jbg==
```

## base64Url

在*BASE64*的基础上编码形成新的加密方式,解决网页中 `+`解码成空格的问题,原base64 `+` 换成`-`,`/`换成 `_`,去除 `=`

**特征**: 0-9 A-Z a-Z -_ 

**测试数据**:

```
5byA5Y-R5bel5YW36ZuG5ZCIIGJ5IGxlb240MDZANTJwb2ppZS5jbg==
```

## [base65536](https://base65536.penclub.club/)

使用unicode(32比特)进行编码

**特征**: 中文生僻字

**测试数据**:

```
𤋥𦮀𡞏𣷥𦮥𣶅𢇩𦮆𠺐霠啹马𐙯攴甶朵𒁰鹪捥𐙣
```

## [base69](https://pshihn.github.io/base69/)

在*BASE64*的基础上编码形成新的加密方式

**特征**: 数字字母 `+/-*<>|`   结尾AA数字=

**测试数据**:

```
tBqBQAOAsA+AjAgBWBkB8ATBtAaBOBbA>A5AyAIA*ABA|A0BQAbAMARB2B4AjBwAbAQAGAOBTA*AZBlB0AUBlAhBbA4AAA2=
```



## [base85](https://en.wikipedia.org/wiki/Ascii85)

又称**Ascii85**, 字典为ASCII码序列

**特征**: 数字+ 大小写字母 +符号(`.-:+=^!/*?&<>()[]{}@%$#`) 

**测试数据**:

```
jh--*O-/P5V<*E?l'mFhOGG#gGp$p7Df.Bc2F',TE,TK*AM.J1
```

## base85 IPv6

base85变种, 编码IPv6 地址(RFC 1924)

**特征**:  数字+ 大小写字母 +符号(`!#$%&()*+-;<=>?@^_  {|}~) 

**测试数据**:

```
<-CC9kCElKrR9aU>6?b-kcc2+c_3_MZ*DX&Hb6BpaBpg9WiDfG
```

## base91

91个字符来表示ASCII编码的二进制数据。从94个可打印ASCII字符（0x21-0x7E）中排除三个字符 `-` `\` `'`

**特征**:  可见字符排除三个字符 `-` `\` `'`

**测试数据**:

```
a[:hQLeff={07_Q]1SQUCG}LfVG!U^;m1t*EplJB2TX6},?iTB
```

## base92

用92个可见字符表示

**特征**: 数字+ 大小写字母 +符号

**测试数据**:

```
sjT_Vni^B1<]D9f:XapY99'b/v8l*vMG4B$E!<Ws$JmoAFJMHa
```


# 进制表示
## binary

二进制表示

**特征**:  0 1 组成

**测试数据**:

```
11100101101111001000000011100101100011111001000111100101101101111010010111100101100001011011011111101001100110111000011011100101100100001000100000100000011000100111100100100000011011000110010101101111011011100011010000110000001101100100000000110101001100100111000001101111011010100110100101100101001011100110001101101110
```

## decimal

字符十进制表示

**特征**: 数字+ 分隔符

**测试数据**:

```
24320 21457 24037 20855 38598 21512 32 98 121 32 108 101 111 110 52 48 54 64 53 50 112 111 106 105 101 46 99 110
```


## radix

10进制数字与任意进制互转

**特征**: 见提示

**测试数据**:

自定义字典 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ_

```
JTWZO
```

解码十进制 37123123

## radix32

32进制数字

**特征**:  大整数,32进制表示

**测试数据**: 

```
smu81pcfi7irf9f5gmruj6s6sm88g832f4g6opbfdoq30di06kp70rrad5iisore
```

## radix64

64进制数字

**特征**: 大整数,64进制表示

**测试数据**:

```
3Zw.3W8P3Zcj3WU14XsE3XAGGEH3GEvjZ02yKBX.LRHuZ0nnXQ3hZe
```

## radix8

8进制数字

**特征**: 大整数,8进制表示

**测试数据**:

```
34557100345437107455572274541333751467033454410404030474440330625573343206015440065144701573246454513461556
```

## radix9

9进制数字

**特征**: 大整数,9进制表示,见提示

**测试数据**:

```
256686600358460680312315208536503
```
## radix10

10进制数字

**特征**: 大整数 0-9

**测试数据**:

```
1916850967896681345870693862611931789614436539962913401095350402234325684881885811965102220272494
```

## radixN

n进制数字,需要自定义字典

**特征**: 大整数,n进制表示,见提示

**测试数据**:

字典 ABCDEFGHIJLKML

```
CMHMEBHBDDMLBKG
```



# 其他
## [ecoj](https://github.com/keith-turner/ecoji)

> Ecoji encodes data using 1024 [emojis](https://unicode.org/emoji/). T
>

**特征**: 表情符号,见提示

**测试数据**:

```
👦🔉🦐🔼🍉🖍🎐🌮💦😫☕☕
```

## escape

转义非ASCII和特殊字符

**特征**: %u

**测试数据**: 

```
%u5F00%u53D1%u5DE5%u5177%u96C6%u5408%20by%20leon406@52pojie.cn
```

## escapeAll

escape所有的字符

**特征**: %u

**测试数据**:

```
%u5f00%u53d1%u5de5%u5177%u96c6%u5408%20%62%79%20%6c%65%6f%6e%34%30%36%40%35%32%70%6f%6a%69%65%2e%63%6e
```
## octal

字符8进制表示

**特征**: 数字 + 分隔符 只有0-7

**测试数据**:

```
57400 51721 56745 50567 113306 52010 40 142 171 40 154 145 157 156 64 60 66 100 65 62 160 157 152 151 145 56 143 156
```

## hex

16进制表示, 同base16

**特征**: 0-9A-F

**测试数据**:

```
e5bc80e58f91e5b7a5e585b7e99b86e59088206279206c656f6e343036403532706f6a69652e636e
```

## hexOctBin

16进制8进制二进制混合编码

**特征**: 包含 `0b` `0x` `0o` , 0-9a-z

**测试数据**:

```
0b1101000b10000010x350b1101010o650o710b1101000x350o640x440x350b1100100o640b10000100b1101000o1050b1101000x420b1100110o650b1101000o640b1101000o700b1101000b1101010b1101010b10000010b1101000b1100100x350x330b1101000x380b1101000x360x340o1030o650o660o650x330x350b1101100b1101000b1100110o640x420x350b1100000x340b10000010o640o1030o640x350o640x390b1101010b1101110x350o620x350x350x340o1020b1101000b1101100x350b1101100x340x360x340o630b1100110b1101100b1101000o630x350o640x340b1101110x340o1010x350x350x340x360o640x330x350o660b1101000o640x340o1060o640o1010x340x410b1100110x340x340b1101110x340o650b1101000x440b1100110x330b1101000b10001000x340b10000110b1101000x360b1101000b10000110o640b1101010x340o1060o650o650x350o640x340o1020o640b10000100x340o650b1101010b10000010b1101000x340b1101000o1020x330b1100110o650o640o640x340x340o1020b1101010b1101100b1101010b1101010b1101000o670o630b1101100o650b1101000x330x320o650b1101100o640x430b1101000x410o640b1100100x340x350x330b1101000b1101010x330o640b1100110x340b1100110x340o710b1101000b10001010b1101010o620b1101010o670x330x320o640x460b1101000o1010o650x350x340o1040o650x320o640o630o640b1101110b1101010b1100010o650o640x350o640b1101000b1100100x340x410b1101000b1100100b1101000b1110010o650b1101000o640o1020x330x440o630o1040o630x44
```

## hexReverse

hex高低位交换

**特征**: 0-9A-F

**测试数据**:

```
5e cb 08 5e f8 19 5e 7b 5a 5e 58 7b 9e b9 68 5e 09 88 02 26 97 02 c6 56 f6 e6 43 03 63 04 53 23 07 f6 a6 96 56 e2 36 e6
```

## [htmlEntity](https://wikipedia.org/wiki/List_of_XML_and_HTML_character_entity_references)

**特征**: &#数字;

**测试数据**:

```
&#24320;&#21457;&#24037;&#20855;&#38598;&#21512;&#32;&#98;&#121;&#32;&#108;&#101;&#111;&#110;&#52;&#48;&#54;&#64;&#53;&#50;&#112;&#111;&#106;&#105;&#101;&#46;&#99;&#110;
```

## jsHex

javascript 16进制表示, shell code

**特征**: \x 两位16进制

**测试数据**:

```
\xe5\xbc\x80\xe5\x8f\x91\xe5\xb7\xa5\xe5\x85\xb7\xe9\x9b\x86\xe5\x90\x88\x20\x62\x79\x20\x6c\x65\x6f\x6e\x34\x30\x36\x40\x35\x32\x70\x6f\x6a\x69\x65\x2e\x63\x6e
```

## jsOctal

javascript 8进制表示

**特征**:  \数字

**测试数据**:

```
\345\274\200\345\217\221\345\267\245\345\205\267\351\233\206\345\220\210\40\142\171\40\154\145\157\156\64\60\66\100\65\62\160\157\152\151\145\56\143\156
```


## [punnyCode](https://en.wikipedia.org/wiki/Punycode)

国际化域名编码

**特征**: xn--

**测试数据**:

```
xn-- by leon406@52pojie-9n25ag54av8ai52m9sfx650b.cn
```

## [quotedPrintable](https://wikipedia.org/wiki/Quoted-printable)

可打印字符引用编码,电子邮件原始信息编码

**特征**: = 2位16进制

**测试数据**:

```
=e5=bc=80=e5=8f=91=e5=b7=a5=e5=85=b7=e9=9b=86=e5=90=88=20by=20leon406@52poj=
ie.cn
```






## [urlEncode](http://cryptowikis.com/EncodeDecode/URL/)

如果 URL 中出现了`拉丁字母、阿拉伯数字、.-_~` 外的符号，则必须用使用`%`编码，所以 URL 编码也称为百分号编码。 如空格字符，ascii 码是 32，对应 16 进制数是 20，那么 url 编码结果是 `%20`
又叫百分号编码，是统一资源定位(URL)编码方式。

**特征**:  % + 2位16进制

**测试数据**:

```
%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7%E9%9B%86%E5%90%88%20by%20leon406%4052pojie.cn
```


## [uuEncode](http://cryptowikis.com/EncodeDecode/UUencode/)

最早在unix 邮件系统中使用，全称：Unix-to-Unix encoding

与base64及其相似，但是UUencode不用写映射表，是通过加32转换到可打印字符范围中。比起base64来，更为简单！

字典为64位

**特征**:  数字+大写字母 + 符号

**测试数据**:

```
HY;R`Y8^1Y;>EY86WZ9N&Y9"((&)Y(&QE;VXT,#9`-3)P;VII92YC;@``
```

## [xxEncode](http://cryptowikis.com/EncodeDecode/XXencode/)

字典为64位

**特征**: 数字+大小写字母 + 符号(+-)

**测试数据**:

```
ctPm+tMyFtPSZtMKruNi4tN06647t64lZPqsoA1N+BH7kPqddNGtXPU++
```

## Z85

base85变种, 没有 `" ` `\`

**特征**:  数字+ 大小写字母 +符号(`.-:+=^!/*?&<>()[]{}@%$#`) 

**测试数据**:

```
<?cc9KceLkRr9Au(6)B?KCC2*C{3{mz/dx=hB6bPAbPG9wIdFg
```



# 字符编码

## 字符集（character set）

为每一个「字符」分配一个唯一的 ID（学名为码位 / 码点 / Code Point）

**代码点（码位 / 码点）**

字符集中的每个字符都被分配到一个“代码点”。每个代码点都有一个特定的唯一数值，称为标值。该标量值通常用十六进制表示。（就是ID的总值吧）

**Unicode与UCS（Universal Character Set 通用字符集）**

其实两者都是为了统一的单一标准字符集，区别在于是历史上两种组织（ISO 国际化标准组织和软件制造商组成的Unicode统一码联盟），前者是ISO/IEC 10646项目，后者是Unicode项目，最终两个组织发现目的相同，开始合并双方成果，后面标准码基本趋于相同（即某个代码点代表的字符相同且有相同的名字）所以UCS字符集目前已经和Unicode兼容。

Unicode 、ASCII(American Standard Code for Information Interchange)是「**字符集**」，字符集只规定了符号的二进制代码，却没有规定这个二进制代码应该如何存储。出于节省空间的目的，对字符集编码的实现方式有所不同。Unicode这个标准不仅同时定义了字符集，还定义了一系列的编码规则

UTF-8(8-bit Unicode Transformation Format Unicode转换格式 之 8位) 是「**编码规则**」

### 编码规则

将「码位」转换为字节序列的规则（编码/解码 可以理解为 加密/解密 的过程）

广义的 Unicode 是一个标准，定义了一个字符集以及一系列的编码规则，即 Unicode 字符集和 UTF-8、UTF-16、UTF-32 等等编码……。

**代码单元**

在每种编码形式中，代码点被映射到一个或多个代码单元。“代码单元”是各个编码方式中的单个单元。代码单元的大小等效于特定编码方式的位数，也就是最小编码单位（就是一个字符用多少位来表示，比如8位的代码单元就是，一个字符最少要用8位二进制来表示）utf-8就是最少使用8位表示一个字符，又因为是变长，如果表示不了就用多个8位。utf-16就是最少使用16位，utf-32类似。

UCS字符集（已经与Unicode兼容），其所对应的编码方式为UCS-2，UCS-4。其中UCS-2是16位来表示一个字符，但这个与utf-16不同的是定长的，固定16位，所以不支持某些扩展字符（超过16位的），UCS-4也是定长，用32位来表示一个字符。所以UCS是落伍的编码方式。

### Unicode与UTF8的关系

UTF-8 顾名思义，是一套以 8 位为一个编码单位的可变长编码。会将一个码位编码为 1 到 4 个字节

例如「知」的Unicode码位是 30693，记作 U+77E5（30693 的十六进制为 0x77E5）

Unicode UTF-8

U+ 0000 \~ U+ 007F: 0XXXXXXX

U+ 0080 \~ U+ 07FF: 110XXXXX 10XXXXXX

U+ 0800 \~ U+ FFFF: 1110XXXX 10XXXXXX 10XXXXXX

U+10000 \~ U+10FFFF: 11110XXX 10XXXXXX 10XXXXXX 10XXXXXX

根据上表中的编码规则，之前的「知」字的码位 U+77E5 属于第三行的范围：

```
       7    7    E    5   
    0111 0111 1110 0101    二进制的 77E5
--------------------------
    0111   011111   100101 二进制的 77E5
1110XXXX 10XXXXXX 10XXXXXX 模版（上表第三行）
11100111 10011111 10100101 代入模版
   E   7    9   F    A   5
```

这就是将 U+77E5 按照 UTF-8 编码为字节序列 E79FA5 的过程。反之亦然。

#### UTF-8 BOM（Byte Order Mark 字节序标记）

在UCS的规范中有一个叫做*zero-width non-breaking*的字符，它的码位是U+FEFF。

UCS规范建议我们在传输字节流前，先传输 字符”ZERO WIDTH NO-BREAK SPACE“。

BOM它能表明信息的一些编码方式：

The byte order, or endianness, of the text stream in the cases of 16-bit and 32-bit encodings;

The fact that the text stream's encoding is Unicode, to a high level of confidence;

Which Unicode character encoding is used.

UTF-8因为它的编码特性，是字节序无关的，所以BOM其实是可选的（Unicode没有建议使用BOM）

BOM在设计中意思是“不可见字符”，在编辑器中不可见，但在UNIX设计规范中，就是文档中存在的数据必须可见。所以linux中对于bom的存在争议颇大，引起很多不兼容的问题。

windows坚持用bom，比如自带的文本编辑器保存的utf8就是with bom的。

## ebcdic (Extended Binary Coded Decimal Interchange Code)
EBCDIC (Extended Binary Coded Decimal Interchange Code) 为国际商用机器公司(IBM)于1963年-64年间推出的字符编码表，根据早期打孔机式的二进制编码的十进制数(BCD, Binary Coded Decimal)排列而成。

**特征** ：无，见提示

**测试数据**：

```
97 81 A2 A2 A6 96 99 84 7A C5 C2 C3 C4 C9 C3
```

使用010以十六进制创建文件，然后修改编码格式为EBCDIC编辑。

## utf7

[RFC 2152标准](https://www.rfc-editor.org/rfc/inline-errata/rfc2152.html)

**特征**:  编码字符为 +开头  -结尾

**测试数据**:

```
+AHgAJwA7AHgAcwBzADoAZQB4AHAAcgBlAHMAcwBpAG8AbgAoAGEAbABlAHIAdAAoADEAKQApADsAZgBvAG4AdAAtAGYAYQBtAGkAbAB5ADoAJw-
```

## utf7-all

所有字符都进行utf7编码

**特征**:  编码字符为  +开头  -结尾

**测试数据**:

```
+XwBT0V3lUXeWxlQIAGIAeQBsAGUAbwBuADQAMAA2AEAANQAyAHAAbwBqAGkAZQAuAGMAbg-
```

## utf7(imap)

[RFC 3501标准](https://www.rfc-editor.org/rfc/inline-errata/rfc3501.html)

**特征**:  编码字符为 &开头  -结尾

**测试数据**:

```
&XwBT0V3lUXeWxlQI-byleon406@52pojie.cn
```

## [unicode](http://cryptowikis.com/EncodeDecode/Unicode/)

统一编码

**特征**:  \u+16进制

**测试数据**:

```
\u5f00\u53d1\u5de5\u5177\u96c6\u5408\u20\u62\u79\u20\u6c\u65\u6f\u6e\u34\u30\u36\u40\u35\u32\u70\u6f\u6a\u69\u65\u2e\u63\u6e
```
