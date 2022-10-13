> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/QPJAbE2kO9vUIF5F964cvg)

**郑重声明：本文只为大家学习交流，如若侵犯贵司权益请联系本人删除，严禁用于商业用途，产生严重后果与纠纷与本人无关**    

  

    距离上次发文已经隔了5个月之久，首先是因为忙，其次是觉得也没啥写的，也没时间和精力像其他大佬那样去研究，以后公众号的发文基本会发些工作中遇到的，至于需要研究的一些新技术空闲了自然会写文章蹭蹭热度。

目标网站:

aHR0cDovL3d3dy50bXNmLmNvbS8=

一、css

很简单，这里就简单带过  
进入网站后随便点一个楼盘，进入后点个一房一价，可以看到如下图  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueFr5lGbVQ1rthVBZSgjJHmaFwG1e7WXiaQcUH4rEPVfjSCJVuQribC3cFjrfj4pdosbibomyZH2HFicKQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

找到css文件

  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5WR6VQXrbzVOkRBEdVAdHLuicgVLsYEmZW4cjuxR56IXjNkKf8oEiazpg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里除了number3.css之外还有个number2.css的不一样的文件，根据background-position-x找到不同的对应映射关系即可  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5q6pV6GM6WMibqbjXtPvRrT627BmHKPKt0MMn0Z4LP8xtUwrs4kib0kaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  
  

二、浏览器的一些指纹  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5A4zhRiacL2lZqEGspsO73QFRRmYjP9CKW5m40yibU5JDmUnKmMLHF7kw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

ck里有个BSFIT_DEVICEID是需要获取的，看这个命名就知道有门道了

这个值在首页，清除缓存ck，刷新后找到如下接口

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5ElKhxsVhmD5VresA3xaQGutnbf8jSRIR3PKGIuPyw1ibQmSf8TQoyhg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

返回的dfp的值就是所需要，其实玩过12306的小伙伴看到的话是熟悉的。我们要知道请求的参数是如何生成，老规矩追栈打断点

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5zDFuBdOiapEDULmJOxIShAiavHgjkX9masKVbAzOvSZFZ9riaTurcDN5g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

首先看到的就是aIgID，也就是kh，kh如何生成的，看下图  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5663yxh8HpyJde5QoM6qutwjjhBuNlkq0usdSgrx4mLnqStgvbibDwfg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里提一点，kh是会变的，也就是说a里传入的那几个数字是会变化的，这点要注意，继续往下看  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5obFoBs2wDGDibC5jqX98g9N7GFzfH6y8AyPXPHiclPrpcfSqydtVZoOA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最终的a[Tj](f, k, r)生成的value值其实就是hashcode，那其他参数，从f的打印中就能看出来是对应的哪些参数了。  

我们接下来先看hashcode生成的js，f里的值我们过会去看。  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5lmRUSM7kv46Hg2LQzIyHhq4WJvkfuc9vIdfuFKhhw3PiasFMkUGSMqw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图我们最主要的是Gc这个函数

  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB54OlF0vr05icLE9JxhwicicwoNibQU38ZuLjGt8M63csPEWIX22icvu7HNYw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

看到这里暂时不管是否有魔改，先直接扣，主要找Hc这个函数

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5icsZeXPIzFqz0Joe7aVOTbdtbudoBic3c0ZtmD8addNoA9DMOsemNvxQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

找到了，怎么扣呢，还是整体的思想，方法置于全局  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5ic1ZUBXFLJBu3UKicLAmM3rPBGyxFfqKWh1RSEujMjUw1ogsOZPfIpZQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

js中无关的代码舍弃掉即可，js源码里函数jk就是请求参数的对照关系，可以打印出来看看，最终结果  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5ZAgfVXepqBq2sXZ5IuWRoEGJSUhwXOaQrGX8qjUYf6mEUUf8fBhDUQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

python算法还原：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5nE2vAcvgCCaEtOU3V1m3xXic4HyicDqy8N1355W9geZd3aT9DQ1VxWKw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

可以看出也就是sha256和base64，结果会有所不同，对照替换一些符号即可

  

f中的指纹我们找个例子看下  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5EicANMTjOctRoYw60z5GCpnK7C8xSAL0gMUthfWjLWYgJ29cc9SibMOQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/OUuGDHj1ueHuVmLibCNtERuGWpJUEjcB5ep0Nh46tNHpjQgoZibHwvjo0B2bpcPJoXCoCQMFquwHTlicxU8ar0vxw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

看标记的那几个值，MD5试试？后面的像什么jsFonts，将Arial这些进行同样加密，几分钟追下就行了。

  

这样一步步看下来加密难度反而觉得挺简单的，关键也就伪造后对方检测是否严格的事，这个网站检测其实还好，不严格，自己伪造下完事。

  

文章水是水了点，学习无止境，各位逆向加油。