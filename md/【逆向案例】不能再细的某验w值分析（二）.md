> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Qkvp9eVrW0CihdHHTuGJyQ)

  

免责声明  

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6eccyrNSOic72rU60MknlI9pEoWgY2WcDNygkNG5NtzxaWSibj18LFoScqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

文章中所有内容仅供学习交流使用，不用于其他任何目的，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业和非法用途，否则由此产生的一切后果与作者无关。若有侵权，请在公众号【爬虫逆向小林哥】联系作者

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6ecnicjTbwdy5ze4PutY5ADhJD1xKy7WpcxMqLhyw93ccIQO4QCP1T30Xg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

  

  

01

—

说明

  

_书接上回_

  

  

  

02

—  

逆向过程

  

参数o

```
`{``"lang": "zh-cn",``"userresponse": "c77c7cc67c338", // 对滑块距离和challenge进行了加密``"passtime": 631, // 轨迹滑动总时长``"imgload": 86, // 图片加载时间，随机生成``"aa": "i(!:!Oawsytsstsssssss(!!($)M718202Z020$)0$*s0DM$*i", //对轨迹和响应的c和s参数进行了加密``"ep": {``"v": "7.8.8",``"$_BIQ": False,``"me": True,``"tm": {``"a": 1667985267223,``"b": 1667985267319,``"c": 1667985267319,``"d": 0,``"e": 0,``"f": 1667985267224,``"g": 1667985267230,``"h": 1667985267232,``"i": 1667985267232,``"j": 1667985267260,``"k": 1667985267244,``"l": 1667985267260,``"m": 1667985267306,``"n": 1667985267307,``"o": 1667985267322,``"p": 1667985267490,``"q": 1667985267490,``"r": 1667985267493,``"s": 1667985267493,``"t": 1667985267493,``"u": 1667985267493` `},``"td": -1` `}, // window["performance"]["timing"] 的属性``"nk8k": "2080253906",``"rp": "ffedc769e64a7afe5eb6338ba9988973" // 对gt、challenge、passtime进行了md5加密``}`
```

这些参数好多，需要一个个分析。先从nk8k

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmojaQkCGE22bNEHIsZI9H1JvU5sAfpeeGO2w3OzQL3dAxT5icVkdIpOg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

逻辑执行了window[$_CAHJR(771)](s)之后，nk8k就出来了

进入加密函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmSyYw53pibnVI0Pho9A8N1DKJesEgglz10V6ZgfA8ra40QjliaVKkQ8tg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里逻辑很乱，暂时把这个值写死吧   

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmEPaenp9mXFhmdDCpqH9Sy0RMf3WeA8zrFWyJPCuLRgSCRrTvzOS7pA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

再来看 userresponse

参数t 滑动的距离，另一个是challenge

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYm6BAF0ygZ3VmNaLicJXocevubmiaZrEw8YhljQNqvouqa8oh6rxIoicg6w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们直接进入H

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmUufd1A2meUoNDXGtjadIdasgAYvxqgFia3UTyTicYpxr8hicAIvYGiaSEQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

直接把H复制下来到我们上面分析u的本地，运行会提示下面的错误

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmgicejRM65ics5oBp6z9ehOW3HtX1JXvzHsLQn65j0qLwmKlYpU4CuFXQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其中的$_CJEZ 我们找不到，我们向上找 H函数的开头应该会定义

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmGl7Vyr7cuXmgX2Eic0HYKhkRbnqUgPiapmqnhJ2bQDR71Z9H2bibr7Yrg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们加在本地，然后补上这个wv_ZX.$_CN，搞定了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmX6QxXJcf4vPibF2bVxuUBYicwvZFV8nOGJmU85V9fv8jUFeyZb2qW4Fg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

o的参数 已经出来两个了

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmxFj7qySadX4LDk5RAemt6jOKbanXKibFdUsbLFcgAUWiahyibjpeXgdyA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下来看passtime，也就是这里的n

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmG25J1v1YYQK1r3iaZBV6GMXcltFcjwiadfbzrVEOP4q7ZBzdkOfx1XzQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

看函数的调用栈

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmKzsEDbpet9Ryx1MasUuZJyfbyWUq20Du8yxGIE0Tvl10iay4Akhcv2A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

暂定为滑动时间，就是结束时候的时间戳-滑动开始的时间戳

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmnXuotD3iaWYB5fIvkA0wxxFrOOwRhB5nc4CJxOBboAE1MBL2XjRPnsw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

再接着是 imgload

  

发现几次都一样，网上查了下这个字段是图片加载时间，随机生成的，这里就写死吧

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYm9TIwIrELmuGbVrOU2IU2LFV9SVshfQNWWSREpLyCohnC8CYoL54XPQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmcib3k0XxU1hJDoGeI8kibXjjrb2uDtIq3NBUF8AFGKyr8TFHicq56RAaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

再接着是 aa

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmLN3PmQFDykvSzgmFt3HHcTfrGUqUTuEaS8wstSxBf6dSBeR3Ema0Rg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

aa是上一个函数传过来的，我们看栈，就是这里的l

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmTFDGFPkWoiafGPoz5P6ASBRaDd5yfzL8QlFibhtUo5oEoDjhXWlYGBbQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

```
`// 函数``n[$_CJJJ_(957)][$_DAAAl(1022)]``// 参数``n[$_DAAAl(957)][$_DAAAl(1027)](), n[$_DAAAl(90)][$_CJJJ_(1004)], n[$_CJJJ_(90)][$_CJJJ_(386)]`
```

先来看参数

n[$_DAAAl(957)][$_DAAAl(1027)]()

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmduiccpJXciaDjbo2dTRpdSEG3P3LCfMkMot7nSiavWWKS6MDicUUHfozjg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmey72frGwgDzfXh4GyRs09R0eiccUTokohWIfSS3cLkbOgx3d1QJMXRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

把这整个w抠下来

运行报错 t 没有lenth属性

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmQK1ibeyJmQDjzNwmTzNrI6FeaYgVSsRzZ80jpcLHZv645VEM0twGSuw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

我们在console查看 t，感觉看上去像轨迹

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmhLT4ib5zUZkb47EhS2PBXcWDxcTmbkloo1Rr1iaKpZjFicW2nG1mBhFMg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

这里的this就是W函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmVCvPkdhicMYFdDYlZcreiaxUlomCVibuU3Ho6mChoecSZZl2Eo7MibDxeA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

应该是个轨迹

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmSA1xtOWr5lXcQzicA9icPgCVYx1sd0vQqy01iblibrib5U3Xh2KXmuYLyqw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

这里的t = this[$_BEGJj(311)] 也就是W的一个属性，我们这里改写一下，直接将轨迹传给

"\u0024\u005f\u0047\u0046\u004a" 供 t 函数调用

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmWdTTDMATcia05f4D7sx0D1rvQ6xSTYPuibhtbWUsTEsgm6ENBLpRwGZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

然后接着报错 ct 没有定义

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmmGnpiavI7oCGmvhG1QFRfwWpS6sBsZIvQFO8F60hVaDtS8q3xBcYsTw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

找到ct复制下来，然后又出现ct的一个属性没有定义 也需要复制下来

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmibxgOHfBpBBna2CJmk3pA3w6s06kicbUIpoI6qm4JRCEZnSAIrehJAvQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmNibZpBclS8oA2SzrGDwGL1tbyPFrtYViaOy8QhibPk497nAibvI1SajYXw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

最后

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmJgMCqytHcBgLUy9Byh1CqDI7e578bQUu5waXkqDibfkc7YRIOiaVYNkA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmb1HWq4sUibRjibcW3lEENXxrzcsQjRI4v5pA0eFfX2Dricmiay4EWeo2FQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个轨迹加密终于成功了 次奥 好嘞

回到上面 这里的第一个参数分析完了

```
`n[$_DAAAl(957)][$_DAAAl(1027)](),` `n[$_DAAAl(90)][$_CJJJ_(1004)],` `n[$_CJJJ_(90)][$_CJJJ_(386)]`
```

接着看n[$_DAAAl(90)][$_CJJJ_(1004)] 和 n[$_CJJJ_(90)][$_CJJJ_(386)]，经过几次测试就是这里响应的 c s。

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmogefoMH2DP91VEu12KdsG6FM086kbqNRibtrD7DwfN0IvrvCKvd9NNg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmV0TyORrzhX48jLe5s094DxtbbQ2WGVibQyPINNgSgW0shtvJvxdcENg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所以aa参数的是三个参数都有了， 最后就是加密函数：n[$_CJJJ_(957)][$_DAAAl(1022)]

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmpTb5OibVztYxyibsHlF7c2luiahIV2KhR1KUGoDeG5QJZeHyNkicpEqANg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

加密函数也是W的属性里面的一个函数，直接复制下来就好

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmHvdCG2fBPG5Y5Lsic1fNOic5OMHoHk8RdHV06rAalQMJwtruB8Iia7cWA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

测试

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmuibsxlpggR0URXfJY4tGb1MSwJmGMXZMa5gTe3g5QmITfz025riczeVA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmuoomgETbMLNWxzJibt4ERPvHs6zOWRAnZhNvcZKCtziaM5z1OQhcuVrA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

好了   aa参数终于结束了，现在好像只有ep了  卧槽 终于快要结束了，都两天了

```
`{``"lang": "zh-cn",``"userresponse": "c77c7cc67c338", // 对滑块距离和challenge进行了加密``"passtime": 631, // 轨迹滑动总时长``"imgload": 86, // 图片加载时间，随机生成``"aa": "i(!:!Oawsytsstsssssss(!!($)M718202Z020$)0$*s0DM$*i", //对轨迹和响应的c和s参数进行了加密``"ep": {``"v": "7.8.8",``"$_BIQ": False,``"me": True,``"tm": {``"a": 1667985267223,``"b": 1667985267319,``"c": 1667985267319,``"d": 0,``"e": 0,``"f": 1667985267224,``"g": 1667985267230,``"h": 1667985267232,``"i": 1667985267232,``"j": 1667985267260,``"k": 1667985267244,``"l": 1667985267260,``"m": 1667985267306,``"n": 1667985267307,``"o": 1667985267322,``"p": 1667985267490,``"q": 1667985267490,``"r": 1667985267493,``"s": 1667985267493,``"t": 1667985267493,``"u": 1667985267493` `},``"td": -1` `}, // window["performance"]["timing"] 的属性``"nk8k": "2080253906",``"rp": "ffedc769e64a7afe5eb6338ba9988973" // 对gt、challenge、passtime进行了md5加密``}`
```

来吧  ep

```
`"ep": {``"v": "7.8.8",``"$_BIQ": False,``"me": True,``"tm": {``"a": 1667985267223,``"b": 1667985267319,``"c": 1667985267319,``"d": 0,``"e": 0,``"f": 1667985267224,``"g": 1667985267230,``"h": 1667985267232,``"i": 1667985267232,``"j": 1667985267260,``"k": 1667985267244,``"l": 1667985267260,``"m": 1667985267306,``"n": 1667985267307,``"o": 1667985267322,``"p": 1667985267490,``"q": 1667985267490,``"r": 1667985267493,``"s": 1667985267493,``"t": 1667985267493,``"u": 1667985267493` `},``"td": -1` `}, // window["performance"]["timing"] 的属性`
```

对应下面的5个属性

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYm6ico4WmNtIUpxSib18uicLvAZSvbWlCXgmdAvk0V34ytGzdGUsCzQ0eRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其中 v参数对应slider.js的版本号![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmh6pdibooSpnZ5P8fKNHl3B5McHJ4J38a4PpvcHff69C3pGEib1wj8K3w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

唯一要分析的就是这个tm

  

这个其实没啥好跟的，就是取取环境window['performance']['timing']，完全可以构造

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYm28cnQUiavia3CnWgIicNAjrVZxSprAj39mOmNcCRLoCbqHDGXmxyKowwg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

到这个 o 完全分析完了

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmBzLnP2RicFNYNZECovVJVR4BickKWojYiamfyPIF02NMYheEnKDPcTQZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

接下来接着看

```
 l = V[$_CAIAZ(339)](pt[$_CAHJR(278)](o), r[$_CAHJR(721)]())
```

参数 r[$_CAHJR(721)]() 就是之前分析的获取随机数

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmNZY7wnVA3Qv7ibFLebS8s8sYjdiaiaboMDV2l3acdkb1a10pg4ciaibCU6A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

第一个参数则是json格式化

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmRKtTYQTzAf42g1J47JPxejTrqRXK9e08J0tXXiah9R8lQpofQTC7DHw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

l这里是个魔改的aes加密，跟进去就可以看到

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmQ0xM9XBpgrZF783Ch3X8zczYxCPMZUtVT2ic6xp20IOI00Hrw60MNgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmux6ialvGHJGMm2QuSfsIlEcOoefSHs1Stlicm713zcQiaCltibyyRcy3Vw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmnshJ0NwlVreoAKHsVCtvgdvZueDGaFaRGhIqs9AnmfYia5pIicIoZklA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里就不在跟了，复现代码如下:

```
`function AES_Encrypt(word, keyy) {``var key = CryptoJS.enc.Utf8.parse(keyy);``var iv = CryptoJS.enc.Utf8.parse("0000000000000000");``var srcs = CryptoJS.enc.Utf8.parse(word);``var encrypted = CryptoJS.AES.encrypt(srcs, key, {``iv: iv,``mode: CryptoJS.mode.CBC,``padding: CryptoJS.pad.Pkcs7` `});``for (var r = encrypted, o = r.ciphertext.words, i = r.ciphertext.sigBytes, s = [], a = 0; a < i; a++) {``var c = o[a >>> 2] >>> 24 - a % 4 * 8 & 255;` `s.push(c);` `}``return s;``}`
```

接在看最后h的生成

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYm3NW5PHwo3mzzZYVte6qaYuDHQxfgryQwSNk79VZjoJlQkuemrIqkFQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

看下m ， 看到这里就感觉很熟悉了，应该是个base64

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmcichhQMWxp2O6xAoOaziaqTW1P4VjEkE45ysk6pNo2BclLjoibDKVOTBw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

但是听说jy base64也魔改过了，还是需要跟进去看看的

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmdiczzOaX6QzUwVzPrdrSxGMCB8FkH6AY6OC0XXVVqdWMkJPBibTP4k9A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)  
加密地方就在这个this[$_HEEw(233)]，我们进去

因为我们没有扣这个函数的父级，所以o=this 后面用到o的地方 我们会报错

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmYlfdnVupsKwic3ZZBmm1PrEsKtLJ9D4EtRsKbict6hAMwMTNOpV7xqqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmE2tkp9Xqsb9kMgfC5hIgibqzibYB6maZtaI27FG69INm7PKyP6W8OGtA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYm84Vcg4WafCs88ezUibWxv5JCBGxME7lf8Sebwm9tItONUHbRAPicHYxg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

注意这里返回了之后，需要合并，我们直接在返回前合并好，就不用扣第一个函数了

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYm0JZ4gLEvxatcEu8bbNCbB48j4PypRAWKuB1HqRJZn9TThzkNiaHoR3Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

h 测试，注意要把 o 和 随机数保持与console 里面一致才可以测试

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmtIzxxNGxvVWYOwYIkaviaXt9jqVYzj5HiciaR2BJ5N356KiauJksydtSTw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmUTuiaic1libfwQLpm05EMl5eKYCw6OVRz3fqDoLnShPbQQbED2ZzUYtkw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmjMZHSvpFmISLzZW3mLpBMTzKkWnjgia3nsncN08YaHcvY1P0UhcnyDA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

哇 h结束了 对了

w = h + u 终于结束了

测试的话断点滑动轨迹，拿出轨迹和challenge 等信息  本地生成w 提交测试  

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYmvgs1xadqicicBHNAOyev7GR8oXnk3Xe4EOMFGWQ7gUP2icvewx8eNxGZw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwibAYBia2kFv5uXruILD1mYm3DsN1sT4JraHOiaoyqd9CzHfgYlzmPPjfckCcGALlUfFtRUvd68t31g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

  

  

03

—  

算法还原

  

> 公众号回复： 【某验w值】 
> 
>   
> 
> 这里直接给上w值的完成生成代码，不想看逆向过程的直接领走吧，由于文章是分析w值的生成，并没有做轨迹生成验证，验证过程是复制浏览器生成的轨迹的。

  

04

—  

归纳总结

  

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6ecwtV3Ot2Y5VCuGU0DibxkkurkYJ2QzbN96L6ibFbBgOEM8TYpH4P8A8eQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6ecwyVZbsGqS6q1xRoreyqHokuq1KdtUq6A4dkuPFpqVjR0loz0QElpNQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

添加好友回复：交流群