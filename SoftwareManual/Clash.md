# Clash

## 增加新规则

### 简便方法（YAML）

如果需要对下载地址为`https://example.com/profile.yaml`的配置文件进行预处理，操作如下：

1. 进入 `Settings` 界面
2. 滚动至 `Profiles` 栏
3. 点击 `Parsers` 右边 `Edit` 打开编辑器，填入：

```yaml
parsers:
  - url: https://example.com/profile.yaml
    yaml:
      prepend-rules:
        - DOMAIN,test.com,DIRECT # rules最前面增加一个规则
      append-proxies:
        - name: test # proxies最后面增加一个服务
          type: http
          server: 123.123.123.123
          port: 456
```

4. 点击编辑器右下角保存按钮

当配置文件触发刷新（包括自动更新）时，`CFW` 会读取`yaml`字段定义的值，将对应值插入/合并到原配置文件中

#### 参数说明

|键|值类型|操作|
|---|---|---|
|`append-rules`|数组|数组合并至原配置`rules`数组**后**|
|`prepend-rules`|数组|数组合并至原配置`rules`数组**前**|
|`append-proxies`|数组|数组合并至原配置`proxies`数组**后**|
|`prepend-proxies`|数组|数组合并至原配置`proxies`数组**前**|
|`append-proxy-groups`|数组|数组合并至原配置`proxy-groups`数组**后**|
|`prepend-proxy-groups`|数组|数组合并至原配置`proxy-groups`数组**前**|
|`mix-proxy-providers`|对象|对象合并至原配置`proxy-providers`中|
|`mix-rule-providers`|对象|对象合并至原配置`rule-providers`中|
|`mix-object`|对象|对象合并至原配置最外层中|
|`commands`|数组|在上面操作完成后执行简单命令操作配置文件|

#### Commands 使用方法（beta）

`commands` 是一组简单的命令，作为上面操作的补充

例子:

```yaml
commands:
  - dns.enable=false # 命令1
  - proxy-groups.0.proxies.2+DIRECT # 命令2
```

每个命令可以被分为三个部分，分别是：**定位+操作+设定值**

命令 1 中，定位是`dns.enable`，操作是`=`，设定值是`false`

命令 2 中，定位是`proxy-groups.0.proxies.2`，操作是`+`，设定值是`DIRECT`

##### 定位

定位中每个层级以`.`分割，数组类型的定位下标由 0 开始计算，命令 2 中`proxy-groups.0`即表示定位至第 1 个策略组，`.proxies`表示访问第一个策略组的 `proxies` 属性，`.proxies.2`表示 `proxies` 属性的第 3 个位置

如果不想用序号定位，也可以使用`name`值定位，`proxies`以及`proxy-groups`中的每个项目都会有`name`属性。例如在`proxies`中已经定义了名为`debug`的节点，那么修改其`udp`参数的定位即为`proxies.(debug).udp`，名称两边的`()`是保证识别，如果节点名不包括`+`、`.`或`=`这几个符号，也可以省略不写

##### 操作

目前支持三种操作：

- `=`：覆盖
- `+`：插入
- `-`：删除

命令 1 中，`=`表示将`dns`下`enable`的值覆盖为`false`

命令 2 中，`+`表示在定位的策略组中的`proxies`数组中添加一个名为`DIRECT`的值，原本其他值被向后移动 1 位。如果此处改成`=`，则会覆盖原来第一个值

##### 设定值

设定值是用于插入或覆盖的值，如果操作是`-`，则此值可有可无，例如：

```yaml
commands:
  - proxies.0- # 命令3
```

此处可以将配置文件`proxies`的第 1 个节点删除

如果设定值为纯数字，则会被识别为整数，为`true|false`则识别为布尔类型，如果 `JSON` 编码通过则识别为对象

`v0.13.7` 版本更新后，支持 3 个内置值用于设置策略组节点，分别是`[]proxyNames`，`[]groupNames`和`[]shuffledProxyNames`，并支持使用正则过滤其中节点，例子如下：

```yaml
yaml:
  prepend-proxy-groups:
    - name: myGroup # 建立新策略组
      type: fallback
      url: "http://www.gstatic.com/generate_204"
      interval: 300
      proxies:
        - DIRECT

  commands:
    - proxy-groups.myGroup.proxies=[]proxyNames|HK # 向策略组添加所有定义的节点名，并按"HK"正则表达式过滤
```

### 进阶方法（JavaScript）

如果需要对下载地址为`https://example.com/profile.yaml`的配置文件进行预处理，操作如下：

1. 进入 `Settings` 界面
2. 滚动至 `Profiles` 栏
3. 点击 `Parsers` 右边 `Edit` 打开编辑器，填入：

```javascript
parsers:
  - url: https://example.com/profile.yaml
    code: |
      module.exports.parse = async (raw, { axios, yaml, notify, console }, { name, url, interval, selected }) => {
        const obj = yaml.parse(raw)
        return yaml.stringify(obj)
      }
```

4. 点击编辑器右下角保存按钮

当配置文件触发刷新（包括自动更新）时，`CFW` 会调用此方法对下载的配置文件内容进行处理，再写入本地文件中

当然，`parsers` 也支持使用路径引入代码：

```yaml
parsers:
  - url: https://example.com/profile.yaml
    file: "C:/Users/cfw/parser.js"
```

> **TIP**
> 
> 使用文件时，允许调用该文件目录下的 `node_modules` 模块

版本 `0.20.10` 开始支持从远端获取

```yaml
parsers:
  - url: https://example.com/profile.yaml
    remote:
      url: https://gist.githubusercontent.com/Fndroid/40e6117252f794aa629b875aa1ecadea/raw/d1ba6d230746c9d2ecfbef211c52fd9a567a781e/parser.js
      cache: true # 默认为false，指示是否对重复下载此预处理代码使用缓存
```

> **注意**
> 
> 使用远端配置请选择可信的代码提供者，如不能信任代码提供者，可以从远端拷贝代码然后使用`code`填入而非使用`remote`方式引入！

#### 参数说明

`CFW` 调用用户定义的`parse`方法时，会传入 3 个参数，分别是**配置文件文本内容**，**工具类对象/方法**以及**配置文件元数据**

##### 配置文件文本内容

`raw` 是一个字符串，一般需要用 `yaml` 库解析成 `JavaScript` 对象

##### 工具类对象/方法

包括：

- `axios`：网络请求框架，[GitHub](https://github.com/axios/axios)
- `yaml`：yaml 框架，[GitHub](https://github.com/eemeli/yaml)
- `notify`：发出系统通知方法，签名为`function notify(title:string, message:string, silent:bool)`
- `console`：日志输出至文件，方便调试，在 `Settings` 界面中 `Parser` 设置下方打开
- `homeDir`：Home Directory 目录

> **TIP**
> 
> 除了以上工具类，在使用`.js`文件时，也可以通过`npm`引入第三方模块

##### 配置文件元数据

元数据为 `JavaScript` 对象，包括：

- `name`：名称
- `url`：下载地址
- `interval`：更新周期
- `selected`：策略组选择缓存，数组
- `mode`：模式缓存

> **TIP**
> 
> 元数据在配置文件首次下载时只有 `url` 参数

#### 返回说明

`parse`方法需要返回一个字符串，`CFW` 会将返回的字符串存入对应的配置文件中

### 多处理器及正则匹配

#### 正则匹配

上面例子中，使用`url`匹配配置文件地址，如果一个处理器需要处理多个配置文件，也可以使用正则表达式进行匹配，使用关键字`reg`设置

```yaml
parsers:
  - reg: ^https://test\.com/.+$ # 正则匹配域名
    yaml:
      prepend-rules:
        - DOMAIN,test.com,DIRECT
```

#### 多处理器

`parser` 定义的数组支持多个处理器从上至下按顺序执行，例如：

```yaml
parsers:
  - reg: https://test.com.+$ # 第一个执行的parser
    file: "C:/Users/cfw/parser.yaml"
  - url: https://example.com/profile.yaml # 对上一个parser执行的结果进行处理
    file: "C:/Users/cfw/parser.js"
```

> **TIP**
> 
> `file` 同时支持 `yaml` 及 `js` 格式的文件

### 使用案例

#### 向本地配置文件添加订阅信息

1. 准备一个本地配置文件的副本，记下文件名和路径。为方便后续说明，以下用`myprofile.yml`指代此文件名，用`C:\...\myprofile.yml`指代此文件路径。

> **NOTICE**
> 
> 配置完成后，此副本文件将作为 `CFW` 配置文件的订阅源使用，对配置文件的**非临时性**修改需要在此副本文件中进行编辑，并在修改后执行订阅更新。移动、重命名此文件后需要修改 `CFW` 中相应的配置选项。

> **TIP**
> 
> 此处使用 `CFW` 创建的配置文件本身而非副本作为订阅源虽然可以实现相同的功能，但无法保证运行时的可靠性，如果没有特殊情况不建议这样使用。

2. 准备一个包含订阅信息`subscription-userinfo`的订阅链接。为方便后续说明，以下用`https://example.com/subscription_url`指代此订阅链接。

> **TIP**
> 
> 可使用`curl -I 'https://example.com/subscription_url'`检查订阅链接是否包含订阅信息。

3. 修改 `CFW` 中的配置文件选项，在`URL`中填写副本文件的本地映射地址`file:///C:\...\myprofile.yml`。保存后执行订阅更新并确保没有报错。

4. 添加对应的预处理脚本，并确保正常匹配到配置文件。

```javascript
parsers: # array
  - reg: "myprofile.yml"
    code: |
      module.exports.parse = async (raw, { axios, yaml, notify, console }) => {
        raw = raw.replace(/# upload=\d*; download=\d*; total=\d*; expire=\d*;*\n/gm,'')
        const url = 'https://example.com/subscription_url'
        let { headers:{"subscription-userinfo": si = ""}={}, status } = await axios.head(url)
        si = si.replace(/;*$/g,'')
        if (status === 200 && si) {
          return `# ${si};\n${raw}`
        }
        return raw
      }
```

5. 执行订阅更新，配置文件模块中出现订阅信息即说明配置成功。

> **TIP**
> 
> 配置完成后可自行设置自动更新选项。

## 支持的规则类型

- `DOMAIN-SUFFIX`：域名后缀匹配
- `DOMAIN`：域名匹配
- `DOMAIN-KEYWORD`：域名关键字匹配
- `IP-CIDR`：IP段匹配，例如 `IP-CIDR,188.1.0.0/16,DIRECT`
- `SRC-IP-CIDR`：源IP段匹配
- `GEOIP`：GEOIP数据库（国家代码）匹配
- `DST-PORT`：目标端口匹配
- `SRC-PORT`：源端口匹配
- `PROCESS-NAME`：源进程名匹配
- `RULE-SET`：Rule Provider规则匹配
- `MATCH`：全匹配

## 示例

### Bing及ChatGPT

```yaml
parsers:
  - reg: ^http
    yaml:
      preprend-rules:
        - DOMAIN-SUFFIX,location.microsoft.com,🔰 选择节点
        - DOMAIN-SUFFIX,bing,🔰 选择节点
        - DOMAIN-SUFFIX,bing.com,🔰 选择节点
        - DOMAIN-SUFFIX,bing.net,🔰 选择节点
        - DOMAIN-SUFFIX,bing123.com,🔰 选择节点
        - DOMAIN-SUFFIX,bingads.com,🔰 选择节点
        - DOMAIN-SUFFIX,bingagencyawards.com,🔰 选择节点
        - DOMAIN-SUFFIX,bingapis.com,🔰 选择节点
        - DOMAIN-SUFFIX,bingapistatistics.com,🔰 选择节点
        - DOMAIN-SUFFIX,bingparachute.com,🔰 选择节点
        - DOMAIN-SUFFIX,bingsandbox.com,🔰 选择节点
        - DOMAIN-SUFFIX,bingvisualsearch.com,🔰 选择节点
        - DOMAIN-SUFFIX,bingworld.com,🔰 选择节点
        - DOMAIN-SUFFIX,bluehatnights.com,🔰 选择节点
        - DOMAIN-SUFFIX,dictate.ms,🔰 选择节点
        - DOMAIN-SUFFIX,flipwithsurface.com,🔰 选择节点
        - DOMAIN-SUFFIX,masalladeloslimites.com,🔰 选择节点
        - DOMAIN-SUFFIX,microsoft-give.com,🔰 选择节点
        - DOMAIN-SUFFIX,microsoftcloudsummit.com,🔰 选择节点
        - DOMAIN-SUFFIX,microsoftdiplomados.com,🔰 选择节点
        - DOMAIN-SUFFIX,microsoftlatamholiday.com,🔰 选择节点
        - DOMAIN-SUFFIX,microsoftmxfilantropia.com,🔰 选择节点
        - DOMAIN-SUFFIX,microsoftpartnersolutions.com,🔰 选择节点
        - DOMAIN-SUFFIX,msunlimitedcloudsummit.com,🔰 选择节点
        - DOMAIN-SUFFIX,office365love.com,🔰 选择节点
        - DOMAIN-SUFFIX,office365tw.com,🔰 选择节点
        - DOMAIN-SUFFIX,renovacionoffice.com,🔰 选择节点
        - DOMAIN-SUFFIX,sprinklesapp.com,🔰 选择节点
        - DOMAIN-SUFFIX,bing.com.cn,🔰 选择节点
        - DOMAIN-SUFFIX,cn.bing.com,🔰 选择节点
        - DOMAIN-SUFFIX,cn.bing.net,🔰 选择节点
        - DOMAIN-SUFFIX,cn.mm.bing.net,🔰 选择节点
        - DOMAIN-SUFFIX,ditu.live.com,🔰 选择节点
        - DOMAIN-SUFFIX,bj1.api.bing.com,🔰 选择节点
        - DOMAIN-SUFFIX,emoi-cncdn.bing.com,🔰 选择节点
        - DOMAIN-SUFFIX,openai.com,🔰 选择节点
        - DOMAIN-SUFFIX,pay.openai.com,🔰 选择节点
        - DOMAIN-SUFFIX,chat.openai.com,🔰 选择节点
        - DOMAIN-SUFFIX,challenges.cloudflare.com,🔰 选择节点
        - DOMAIN-SUFFIX,auth0.com,🔰 选择节点
        - DOMAIN-SUFFIX,auth0.openai.com,🔰 选择节点
        - DOMAIN-SUFFIX,platform.openai.com,🔰 选择节点
        - DOMAIN-SUFFIX,chatgpt.com,🔰 选择节点
        - DOMAIN-SUFFIX,hcaptcha.com,🔰 选择节点
        - DOMAIN-SUFFIX,recaptcha.net,🔰 选择节点
        - DOMAIN-SUFFIX,sfx.ms,🔰 选择节点
        - DOMAIN-SUFFIX,microsoft.com,🔰 选择节点
        - DOMAIN-SUFFIX,oaistatic.com,🔰 选择节点
        - DOMAIN-SUFFIX,oaiusercontent.com,🔰 选择节点
        - DOMAIN-SUFFIX,ai.com,🔰 选择节点
        - DOMAIN-SUFFIX,invoice.stripe.com,🔰 选择节点
        - DOMAIN-SUFFIX,stripe.com,🔰 选择节点
        - DOMAIN,bard.google.com,🔰 选择节点
        - DOMAIN-SUFFIX,bing.com,🔰 选择节点
        - DOMAIN-SUFFIX,sentry.io,🔰 选择节点
        - DOMAIN-SUFFIX,identrust.com,🔰 选择节点
        - DOMAIN,openaiapi-site.azureedge.net,🔰 选择节点
        - DOMAIN-SUFFIX,poe.com,🔰 选择节点
        - DOMAIN,servd-anthropic-website.b-cdn.net,🔰 选择节点
        - DOMAIN-SUFFIX,anthropic.com,🔰 选择节点
        - DOMAIN-SUFFIX,claude.ai,🔰 选择节点
```


## 绕过系统代理

- 进入 Settings 页面
- 点击 System proxy Bypass 右边 Edit 小字打开编辑界面
- 若要增加绕过example.com域名，只需在修改编辑界面内容为：

```yaml
bypass:  
- "example.com" 
- 127.0.0.1  
- 192.1.*
```

点击编辑器右下角保存

bypass 类型为数组，item 为需要绕过的域名或节点，支持通配符*



## 遇到诸如Microsoft TODO无法访问的问题
`General` 中有一个 `UWP Loopback` 功能，勾上对应的程序保存。