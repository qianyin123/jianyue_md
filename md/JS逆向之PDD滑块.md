> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KqRSdX7YHwAy24hzpn3o9A)

  

 ![](http://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaT2WIQxAIBU1e1AwvCLDnxPX5G0GmMm0OvISKGKIv9vHeQsYUsWOiaLfOoNoZs1nicv6NGZFTdr5I9g/0?wx_fmt=png) ** python爬虫和阿J的逆向 ** python爬虫和JS逆向的学习交流 9篇原创内容   公众号

目标网站：aHR0cHM6Ly9tbXMucGluZHVvZHVvLmNvbS9sb2dpbi8=

一、触发滑块
======

多次点击登入后，就会出现这个滑块了

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaRicyrRlVuP32Yj5htITXU83pEicI7SRqT210eKtqTVibAqtjQ5bSFGZ1yia8UTFIl08kbuRJTibw3ZRRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

  

二、参数分析
======

先来看看出验证码的这个接口

![图片](https://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaRicyrRlVuP32Yj5htITXU834ntgW4hTDr3u50ShEicxSiczjiaX6ClY2sfn1tAB2DlibF15l0q8GYgsAA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

参数还是蛮多的，其中crawlerInfo这个就是pdd系列的，也就是请求头中的anti-content，这篇文章主讲滑块，这个参数就不聊了，主要是补环境，网上也有开源的

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaRicyrRlVuP32Yj5htITXU83ySyiccXmribGjwWcCiaNjXM8GX4NW1xhdmYlhuQ7cxwqbzOF2VHR52ib7w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaRicyrRlVuP32Yj5htITXU83umib6cwpHgicsce4oerf5NoqqL5z9Gvuae778ksldEicyIictHJlkTSyXw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

fingerprint就是浏览器指纹了，password是密码的rsa加密

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaRicyrRlVuP32Yj5htITXU83nzoBEJKf0Na2sSzFMdbIVsbialMWvkje4J22mrsfagSm3c94TafZ6Ew/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

riskSign是账号密码+时间戳的组合md5

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

然后请求下这个接口，拿到了verifyAuthToken

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

之后就是获取验证码图片的接口，这里出了个新的参数captcha_collect，是鼠标轨迹的加密，这个接口可以写空

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

然后就拿到了图片的信息，然而返回的并不是图片的base64编码，是经过加密的

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

图片解密在这个位置，接出来后就是base64编码的信息

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

之后就是搞下缺口位置，方法很多，网上自行copy，然后就是看验证滑块的接口了

这里captcha_collect是需要的轨迹加密信息

verify_code这个东西很神秘，是换算后的缺口位置

![图片](https://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaRicyrRlVuP32Yj5htITXU83MH7OqhlIc7vfgzxowBDpCbhd6WziatqjlVYrNcMfEiaXnxnJhsd6PK6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

然后轨迹加密的位置在这，`h(p(JSON[u("0x1a")](r)))`，这里p函数是gzip加密，然后h是AES加密

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaRicyrRlVuP32Yj5htITXU83KlcCBE4RQx0Hf5atD1TicL3eBcicaKFqEJUdsfuvLfomzjs4CRjDib3mg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

入口参数r是这么些东西，包含轨迹信息，浏览器指纹等

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

captcha_collect基本上就能搞定了，之后就是这个verify_code，在这里生成的

![图片](https://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaRicyrRlVuP32Yj5htITXU83icg16MEr9TS44UDeAOBqIvlKU19WPrOpLYVzSKhVvEzYwX2icUibZYXRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

参数都能搞定了，最后发下请求看看，基本上都是可以过得

![图片](https://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaRicyrRlVuP32Yj5htITXU839rfP6iahllNW6qh2byibm5FYgO3n7hEuqNGibLKH7nn84NQurLF0oKicicA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/wNWExdcfqaRicyrRlVuP32Yj5htITXU833rr9hPwe3XOns4E2TSWJ1nGqLhyfOC9V7hic6Te2pAOMicOlz0ibJOhtA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

三、总结
====

最后总结下，这个轨迹是不校验的，主要是缺口位置，直接opencv模板匹配出来的是不对的，需要经过一定换算，然后这个verify_code，跟轨迹里的最终位置也是不同的，也需要一点的换算。