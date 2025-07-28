> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aDf5bCTHPMV6b2IDzS3sxA)

*   • 发布接口分析
    

*   • 2.1 x-secsdk-csrf-token
    
*   • 2.2 public-key
    
*   • 2.3 bd-ticket-guard-client-data
    
*   • 2.4 x-tt-session-dtrait (jsvmp)
    

发布接口剩余的风控参数分析
-------------

上次前置接口的加密解决完 能发布之后，发现连续发布 第三条会掉号

所以继续解决剩下的几个参数

### 2.1 x-secsdk-csrf-token

head接口访问 https://creator.douyin.com/web/api/media/anchor/search

然后从响应头里提取X-Ware-Csrf-Token

```
`csrf_token_list = csrf_token.split(',')  
if len(csrf_token_list) > 1:  
    csrf_token = csrf_token_list[1]`
```

### 2.2 public-key

**可忽略：最后发现 浏览器里会存到localStorage里面**

直接搜headers的名字，可以定位到这里，很明显的控制流

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xpJOJSJpDpMoGBZFHqr9iaXyiclLsRYe2YUjjCYmRL7FGjVibreQXmJ7fg/640?wx_fmt=png&from=appmsg&randomid=rljxp5x4&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
传入一个e,r 这是一个私钥一个公钥

关于这个密钥，可以往上搜一下，以及明文js里面名字得到，大概意思是 浏览器会一开始生成一个ECDH密钥，后面的计算都用到了这个东西，把它看成浏览器环境对应账号就好了。

一个账号/一个环境对应一个公钥私钥就好了，所以把e,r都复制下来先

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xx8kWh85hOgFdURyaWeaGCGWF5EHlIu2zLEQ3j1WOfMIkL82crruCHg/640?wx_fmt=png&from=appmsg&randomid=8fjglx4t&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
getPubKeyBase64跟进去

控制流

先去掉公钥首尾的东东，再base64解码 解析DER格式 转为hex

丢给ai操作就好了

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xGOYT6EQicfhibicfsHG3uwibiaH61ZZmibA0lZL7smc0LjK5jPagibd610ypg/640?wx_fmt=png&from=appmsg&randomid=j4y9ycu2&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
再通过convertHexToBase64函数就好了

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xnApCNwRkic7feC3umJqboDR3YRsIcCtpkyJHwl41TfwagnI3JRQOG8w/640?wx_fmt=png&from=appmsg&randomid=qo2xe0i6&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xWqibkfziaUR6rHns8qQoKqFsZuNtCjibicya9GlFbPndBUTu89cpicdwkVA/640?wx_fmt=png&from=appmsg&randomid=mlwvu4z9&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

### 2.3 bd-ticket-guard-client-data

老规矩直接页面上搜索

搜出来多出，附近都下断点，第一次找出来的，是那几个东西拼接以下base64

没用，我们发布接口，要找后面生成的，可以直接发布接口 解以下base64 发现用到了ts_sign和req_sign 以及时间戳这些 ，要找这个

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xcqxtpmibC8qt4Nsia3ZTHyJ6LBIdlUS7icFHwiaFTs4cdyCUYXq3aF5Txg/640?wx_fmt=png&from=appmsg&randomid=sq1zgym6&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xAeNV9ksricoN7fGRcNtPsqiaMwzHRMfDGJUtGMOiataib5Lib36U0kVqJJA/640?wx_fmt=png&from=appmsg&randomid=b1zp6kbb&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
多次看headers的cliend_data参数 base64解一下 ，发现只有req_sign和datestamp会变，所以要搞定这两个东西(其他的ts_sign等信息 登录之后就能拿到 所以忽略)

搜 req_sign，

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xvGJzsoHMIY3v2ylCBREBEbv7Y7E7e5sBp789JnOT9LylzicetDBnJaw/640?wx_fmt=png&from=appmsg&randomid=ajzc6r51&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
这个不太好跟，

藏得太深，要有耐心，

在控制流的时候一个一个慢慢看，跟点：switch控制流 default .icall的位置 单步

直接说结论了。

首先要先跟一下这个ecdh_key(私钥和证书生成的)

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xmIGIyGvlNX0m9YA5w1a2WhrpjKkMu5alLOAyWtzKCl12F6v6xNJy2Q/640?wx_fmt=png&from=appmsg&randomid=8utj2pa4&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
生成key的js代码

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xF6dln3W3XtFyXwpmPUS1K2U892hozvg5T8OOHLEsFxibiapzribH56KeQ/640?wx_fmt=png&from=appmsg&randomid=yjn460ro&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
python的

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xyg3FibykPdcTNY7NT2FwaFq5o7qGhpLuGpn1kzVYT2DVEZoJWEueeLQ/640?wx_fmt=png&from=appmsg&randomid=qppdn6tt&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
然后组装一下 base64就得到结果了

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4x4CGicJ1nfjMjeIQSwCUAibe41WialgBNnnwApGH4jMbQOREibwlbicbThfg/640?wx_fmt=png&from=appmsg&randomid=hyos4jtw&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
核对结果

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xnRroTJ7hnHf8BfRNpPoMvRxT5Ds4ibc3iayg10jIdvaVf0AO6XqnCHIQ/640?wx_fmt=png&from=appmsg&randomid=3hgfcqfu&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

### 2.4 x-tt-session-dtrait (jsvmp)

直接搜索，发现搜不到关键字，只能通过xhr 往上跟栈

在点发布前 拦截断点，查找最后生成的地方

最后跟到这里

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xdBOsFfSQg3iap6rIKcOo2raAIicJhw3yhtIfdiaNc7vWDE5piawKfpibmtw/640?wx_fmt=png&from=appmsg&randomid=oohuhvf8&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
省略N步。。。 最后会跟到一个vmp里面 如下图

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xNuXknW7IFD7baT7vFykKn7HZNLmNwj2WFqrX8ib5DzRXnbJBQ5apXdw/640?wx_fmt=png&from=appmsg&randomid=7v16m8j7&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
当时忘了怎么跟的

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xlf2fNibSrVJPbrQXlt3GjmpQnfufHPGLicbyEHcyxZGpfiardia0oHzumQ/640?wx_fmt=png&from=appmsg&randomid=b4ol9aeb&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
大概的最后的加密逻辑代码在这附近

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xH1vn6NzsKx6U8TQh0xaAaLkKzLQT5YRgRredRQ7nykDvVxgKgVwsuQ/640?wx_fmt=png&from=appmsg&randomid=n7yrbt01&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
直接说加密逻辑吧

结果格式为："d0_xx_xx"

d0固定 第一个xx为rsa加密，第二个xx为aes-cbc加密

在发布页面(编辑页面)时 会随机生成一个aeskey, 再通过rsa把key加密 放到第一个xx

rsa的公钥通过一个接口获取，目前看获取的固定的

然后再随机生成一个iv,通过aes把一段内容通过刚刚的key+iv加密

上述加密简单，具体要分析加密内容(vmp里面找-加密的内容是浏览器环境)

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4x3H1IaSicLEM32djvBcxJEGxLkcPJh2RlTibicEL82rmGkxHoiaHSSianl5g/640?wx_fmt=png&from=appmsg&randomid=tc1tpdsm&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
加密原文：{"dtrait":"IAAAAA......","timestamp":1753688900,"sdkVersion":"1.0.31-beta.4","path":"/web/api/media/aweme/create_v2/"}

需要分析这个dtrait生成过程

首先 到vmp里面 打日志断点(控制指令，栈，apply toString 以及结果)

这个vmp几十万的日志 直接浏览器卡死了，建议跑一段分析一段 ，懒得继续回去分析了，直接贴几张图

最后是一个数组 通过fromCharCode再base64

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xgrquqx43zKVFOBp1fNLruCerhk3ibxts6nZbnyv728kNIOfoyRibbHQQ/640?wx_fmt=png&from=appmsg&randomid=cvp0tyoq&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
所以需要分析这个数组怎么来的

通过vmp日志分析，数组又是另外一个object int运算过来的

```
`keys = sorted(data.keys())  
vmp_result = []  
for key in keys:  
    value = data[key]  
    vmp_result.append(key)  
    vmp_result.append((value >> 24) & 255)  
    vmp_result.append((value >> 16) & 255)  
    vmp_result.append((value >> 8) & 255)  
    vmp_result.append(value & 255)  
prepend = [32, 0, 0, 0, 0, 144]  
new_arr = prepend + vmp_result  
bytes_data = bytes(new_arr)  
base64_str = base64.b64encode(bytes_data).decode('ascii')  
return base64_st`
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xJbibGf1ZBnuLpxfd0MdclKDAh4SQ0DqTYPlo0IOIH8y9U3HYNBlhErg/640?wx_fmt=png&from=appmsg&randomid=zd3ttnc3&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
好多数值会变 要找一下数值来源

数值分析，比如直接搜第50个 823932446

先获取ua 再经过Oa这个函数计算，搜了一下Oa这个函数 就是mmh3 hash算法

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xyA5vEDsTvcvU6mqe1UtM8PHK35vT0DBPWlKqGYlR1dSGC6CcTcCjzg/640?wx_fmt=png&from=appmsg&randomid=nu690v8n&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

  
以此类推找到其他的值就好了，贴一张大概说明

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xyA5vEDsTvcvU6mqe1UtM8PHK35vT0DBPWlKqGYlR1dSGC6CcTcCjzg/640?wx_fmt=png&from=appmsg&randomid=2oczayns&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "null")

最后 已测试 换浏览器 换账号 没问题 能连续发3条以上

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14c9bS3ibWdT9MqrdojYtOB4xrok8ZiagT4ElUmCh5hnBW7jOQSvubA6BCQlibGmZZYuyEwoBvCczemxQ/640?wx_fmt=png&from=appmsg&randomid=stafxfsn&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)