> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/CNyJz9SqK02SB6HN_wi7ng)

前言
--

虽然没赶在中秋前发文，并且也过了中秋，但还是祝大家中秋快乐，这次采用 `视频 ＋ 图文` 的方式

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWkynMHEUvKd0MCyvPjbQK8zYRUh6XGoia8wz119Nu02sw8H0icKTS5ZDA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

 已关注  关注  重播  分享   赞    关闭**观看更多**更多

_退出全屏_[](javascript:;)_切换到竖屏全屏__退出全屏_进制珊行已关注[](javascript:;)分享视频，时长01:13:32

0/0

00:00/01:13:32 切换到横屏模式 继续播放进度条，百分之0[播放](javascript:;)00:00/01:13:3201:13:32[倍速](javascript:;)_全屏_

倍速播放中 [0.5倍](javascript:;) [0.75倍](javascript:;) [1.0倍](javascript:;) [1.5倍](javascript:;) [2.0倍](javascript:;) [超清](javascript:;) [流畅](javascript:;) <video src="https://mpvideo.qpic.cn/0bc3lqaayaaagqal7d7jwvtfaxgdbroaadaa.f10002.mp4?dis_k=36dccd44863fa3564821ef69688ab68f&dis_t=1763457020&play_scene=10120&auth_info=Jev/zatXElU3nd+bnnBHBmNOAxUlKT9YZwNGZkByNCQfTiFfND54PENhYVdsUFU=&auth_key=ae006f6cf112623e3983d0a82bd6211e&vid=wxv_3640648777184100354&format_id=10002&support_redirect=0&mmversion=false" control></video>

继续观看

【JS逆向】极验3滑块

观看更多原创,【JS逆向】极验3滑块进制珊行已关注分享点赞在看已同步到看一看[写下你的评论](javascript:;)

[视频详情](javascript:;)

一、抓包分析  

---------

1.1 验证流程
--------

### 1.1.1 register-slide

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocKubSnzSAgsSKKe8E7e7SgO2nkLTBvvJCichWhzoeickstseL4cuyEHsw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocpnKicsPbmFSaNkMhejy1y5HQLZl8L1R1UvrOhZ3IBFcj0jGNuXI5vxA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

*   url : `https://www.geetest.com/demo/gt/register-slide?t=1725377045900`
    
*   参数 t : 时间戳
    
*   响应 :
    
*   `challenge` 和 `gt`，后面会广泛用到
    

### 1.1.2 gettype.php

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoccVuAcUdibKukkQ56hFnA5YicJPcAASxUByBCpyictVLEsQ6uOLJSBSAhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocs66KDic34oJXiabyAdxjXD7jphBUugFMSR7qlwFiaa0vqtOwFGRbUcMJA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

*   url : `https://apiv6.geetest.com/gettype.php?gt=019924a82c70bb123aae90d483087f94&callback=geetest_1725377047936`
    
*   参数 :
    
*   gt : 由上一个请求 `register-slide` 返回的参数
    
*   callback : `geetest_ + 时间戳`
    
*   该请求的目的是获取极验安全校验的核心js文件`fullpage.js`和极验提供的各类型验证码对应的js文件。该请求流程是必须要有的
    

### 1.1.3 js/fullpage.9.1.9-dbjg5z.js（核心加密文件）

*   该js文件就是极验验证码的核心文件，相关的验证算法、指纹校验模拟等都被包含在该核心文件中。后期逆向时，主要针对就是该文件中相关的js代码。
    

### 1.1.4 get.php 第一个 w

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoc69klXLcfbJpib38ITu2MWgNP6LdGDEMTxRyWcq3HjtG5qEsu6RJjEvw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKochzN90RhPxa0CibXhAf27WDRWt3cC2ASEKTSMoKk8ysraN5TfG0Fp2zg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

*   获取之前注册/申请好的验证码的数据包。其中，这里的请求参数 `w` 不需要进行逆向，可以置空，也可以固定，第一个 `w`
    
*   响应中返回重要数据 `c` 和 `s`
    

1.2 点击验证后数据包
------------

### 1.2.1 ajax.php 第二个w

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocYKqczzQibUwhiawXS2x10wF41jfnQJFuPiaOEoia2pd6SYZqOVrlaDdaBw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocXXPsMegoYt0wxoagK1V9TA0trfnHQUf5IPU2dJaKkn5gJ2s4zQTic0g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

*   ajax请求，用于向极验请求指定类型的验证码，会发现响应数据中有 `result:slider` 就表示请求滑块类型验证码，然后 `status:success` 表示请求成功。
    
*   其中请求参数 `w` 是需要逆向的。callback参数是geetest结合时间戳的形式。第二个 `w`
    

### 1.2.2 xxx.webp 和 xxx.png

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocqBGB6NoacRxiaOzyazCb3nw6LRtjsbShkAW6aXECQ6bep4JN8ibUDTdg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocicTK5mmFDHg4xkDibyPPic6HfFCG1JraITGWC8a2HGNSahI4OyslOLbWQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

*   发现是被打乱顺序的验证码图片, 可以用python中 `PIL` 解决
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocIze6KgoGbhXI573Ps5c6I1ZHCCBvnhlFm2sfjbfdH3ibCGicRRicD25FA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11)

*   滑块图片，不需要逆向
    
*   但是需要去除空白部分得到滑块的边框
    

### 1.2.3 第二个get.php

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoc8uHbrosNMvNfUoiatStwbAaNnbz5BOEibqna1oFics0vtBQHuX95opopA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12)

*   用于加载还原顺序后的滑动验证码的数据包。其中并没有需要逆向的请求请求参数，不过其中的callback参数是geetest结合时间戳的形式。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocQ5zIiaFtYYhASdQauYatLsuRII0mpmb92AtQ1xlDLF7jrYooxgFlU6A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocorib5QHjnOhszMnkp8dXbpdP8yRj2f0TsLJXCOjdwNaWvPyCwibaco5g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14)

*   返回了 1.2.2 中乱序的背景图片
    

并且返回了新的 `challange、gt、c、s` ，注意 `challange` 比原来的值多了两位数

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoczTTPxkJBKfSPia2cXl6ia5oEgJLoAmN8hlau5DN0ZKqicyPibCQT4WMfhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=15)

1.3 滑动滑块后数据包捕获
--------------

### 1.3.1 ajax.php 第三个需要逆向的 w

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocjFPLgadW7Oibrm1Q0KiaWUQ2OkQYIgTjLQgn9u5jOpT4ibaBNuJcq2YXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=16)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocFHibbJLuDmMbqEqEZ9aMriboaD2eZqEribXacy5iaS1BIV8HsukOYxAgNQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=17)

*   滑动状态验证的数据包，验证滑动行为是否合法是否滑动成功。滑动成功会出现一个validate的键值对，后续带上validate请求其他数据即可
    
*   请求参数中有一个w需要逆向，该 `w` 就表示用户滑动的轨迹加密后的内容
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocoopn9sS6fJljB7oVx6zCRUDAeuxg0Kopljbdxm1pd1fbLXkCEGaYUQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=18)

这里的challenge已经发生变化，多了两位，就是上一个 `get.php` 中返回的新的 `challagne`

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoc7o8O8F6a11QbGzOGSTsBc7dE7xufMAnCEZ7DDiau5zeLEqia8aic7UBzw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=19)

最后返回我们需要的 `validate`

二、逆向 W
------

2.1 register-slide
------------------

```
import requests  
import time  
  
timestamp = int(time.time() * 1000)  
session = requests.session()  
  
headers = {  
    "priority": "u=1, i",  
    "referer": "https://www.geetest.com/demo/slide-float.html",  
    "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 Edg/128.0.0.0"  
}  
  
def get_gt_challenge():  
    url = "https://www.geetest.com/demo/gt/register-slide"  
    params = {  
        "t": timestamp  
    }  
    response = session.get(url, headers=headers, params=params).json()  
    challenge = response['challenge']  
    gt = response['gt']  
  
    return challenge, gt  

```

*   直接请求
    
*   返回：`challenge, gt`
    

2.2 gettype
-----------

需要携带的参数：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocFUicvH04bAjTYUBqVJUA5FBCVdCmEUcL48GEC2f8P7PDSPXezobvMvA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=20)

```
def req_gettype(gt):  
    url = "https://apiv6.geetest.com/gettype.php"  
    params = {  
        "gt": gt,  
        "callback": "geetest_" + timestamp  
    }  
    response = requests.get(url, headers=headers, params=params)  

```

*   直接请求
    
*   返回：`js文件路径`
    

2.3 js/fullpage
---------------

`js/fullpage.9.1.9-dbjg5z.js` 这个是核心的js文件，不需要进行请求

2.4 get.php 第一个 w，可以置空或者固定
--------------------------

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocJ9p3zvf3ghT4bLR7W0D277Hzo9kc5kWkDS1KRZIA8rzPClQu2Rm2zA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=21)

*   challenge, gt : 从第一个请求 `register-slide` 响应中获取
    
*   callback : `geetest_ + 时间戳`
    
*   w : 置空或者固定
    

2.5 ajax.php 第二个 w，也可以置空或者固定
----------------------------

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoc0OVGJYdibA2SR2ibf2hGibyHzMIyDYNFfW4TfaRISKHkibYmoSB8qics6QQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=22)

*   challenge, gt : 从第一个请求 `register-slide` 响应中获取
    
*   callback : `geetest_ + 时间戳`
    
*   w : 置空或者固定
    

2.6 第二个get.php
--------------

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKochDpPJmHwV6weicsC7CK6dSGXo1Br4StlUI0Qa7tEsDNjW8fT0eM7iaXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=23)

*   challenge, gt : 从第一个请求 `register-slide` 响应中获取
    
*   callback : `geetest_ + 时间戳`
    
*   其他参数固定
    

```
def get_new_challenge(gt, challenge):  
    url = "https://apiv6.geetest.com/get.php"  
    params = {  
        "is_next": "true",  
        "type": "slide3",  
        "gt": gt,  
        "challenge": challenge,  
        "lang": "zh-cn",  
        "https": "true",  
        "protocol": "https://",  
        "offline": "false",  
        "product": "embed",  
        "api_server": "apiv6.geetest.com",  
        "isPC": "true",  
        "autoReset": "true",  
        "width": "100%",  
        "callback": "geetest_" + timestamp  
    }  
    response = requests.get(url, headers=headers, params=params).text  
    jsn = re.findall(r"geetest_\d+\((.*?)\)", response)[0]  
    datas = json.loads(jsn)  
  
    return {  
        "bg": datas["bg"],  
        "slice": datas["slice"],  
        "gt": datas["gt"],  
        "challenge": datas["challenge"],  
        "c": datas["c"],  
        "s": datas["s"]  
    }  

```

2.7 ajax 第三个 w（需要逆向的 w）
-----------------------

### 2.7.1 w的加密位置

进入堆栈

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocCmzJMiauN2UhpahVoz2xzBAO6s0UQNcltrtvfzpyxIQaZkibrEPgqxxg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=24)

搜索 `"\u0077":` 在搜索到的地方都打上断点

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocdcC5A1p0iaINs0lRHpl6CxsiaicF8xCK0s65icZDGojVIzkxgsChelgKUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=25)

刷新后来到如下断点，并滑动滑块

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoctXDmRoAFR0Fe4UFvQv3VALicmmGTJpyIoCUgAfaytZmSU8zLgYKBlEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=26)

*   我们需要的 `w` 就在这里， `w = h + u`
    

往上看可以看见 `h 和 u`

```
var u = r[$_CAHJS(737)]()  
var l = V[$_CAHJS(392)](gt[$_CAIAK(254)](o), r[$_CAIAK(744)]())  
var h = m[$_CAIAK(792)](l)  

```

### 2.7.2 解混淆

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoclnYQvBGfGsufl76Gag6q24icKglYriaiaKI8oOt75K4glgciaT6beVvvEQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=27)

```
var u = r["$_CCDm"]()  
var l = V["encrypt"](gt["stringify"](o), r["$_CCEc"]())  
var h = m["$_FEX"](l)  

```

### 2.7.3 r 参数

跟进函数里

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocMODhPaklEwFlfaKjEZQNnVcEH08vJLcPbR8gATlzfflOAanpDKZqeA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=28)

函数返回值为 `t`，在两处下断点并运行，直接跳过了 `while` 里的 `t`

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoczCo3uyAZywSJDMBic4A29tTMPUic7nic6Brj7Rh2iccNU6AHGhH5VtsaCA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=29)

解混淆后：

```
var e = new U()["encrypt"](this["$_CCEc"](t));  

```

需要解决：

*   `this["$_CCEc"](t)`
    
*   `new U()["encrypt"]`
    

#### 2.7.3.1 this[“$_CCEc”](t)

先解决后面的 `this["$_CCEc"](t)`，跟进 `this["$_CCEc"]`函数里，传进来的参数 `e = undefined`

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocyTKHVUmzVIibGhTYXKjE7eB5bVQiaCtyGBzObgzObiajZefxscwibfRkJg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=30)

注意 `Ot = rt()`，`Ot` 就是我们要的值，所以跟进 `rt()` 函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocsR9TxK8sTut32ibDiaGg3xR3AuNCDBF2G1WNclfVoo27HYet7ZJZQpaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=31)

下断点运行后发现返回的是由四个 `t()` 函数拼接的字符串，长度为16

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocEIVTBibRmMmenc3docGxvRoUfurp6ezUySSLx7UZp2ysLhL8TnGRJtA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=32)

正好 `this["$_CCEc"](t)` 返回的参数就是长度为16的字符串，所以跟进 `t()` 函数看看

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocJJFyUqyDVqtlOVPnMHhPlhygPpjibjS1UIDCalzmAARDnNatlppIxUw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=33)

t() 函数主要逻辑：

```
(65536 * (1 + Math[$_BFBFC(21)]()) | 0)[$_BFBFC(213)](16)[$_BFBFC(415)](1);  

```

解混淆后：

```
(65536 * (1 + Math["random"]()) | 0)["toString"](16)["substring"](1);  

```

循环四次就能得到16为长度的字符串：

```
function randomStr() {  
    let str = "";  
    for (let i = 0; i < 4; i++) {  
        str += (65536 * (1 + Math["random"]()) | 0)["toString"](16)["substring"](1);  
    }  
      
    return str;  
}  

```

#### 2.7.3.2 new U()[“encrypt”]

跟进函数 `new U()["encrypt"]`

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoc8mEVkw70xf3RYqnl3tnicEuSVic7APNjouXicDib58woEkXTQjbwZLMNjw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=34)

发现此函数是由 `E` 对象里得到的，所以去找 `E` 生成的地方

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocyW54Ot0RITOODwEAOdDQFMFoZFs9dsYECLnnAWezY9TKcgwlRqyPEA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=35)

将所有代码拷贝到 `notepad++` 里，折叠所有，搜索 `E[$_JJDZ(62)][$_JJDZ(40)]` ，向上找可以发现最后 `return` 的也是 `E`

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocDoIbCIW2Y3bJibGkZa4NP7ZpcK2TLhqfdX9KGYabPcPQCgyg6ATte4g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=36)

往上找可以找到所属的函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoc4NrRUg7BCnrnZicTyxSdJicltUXCZJaUJkVzibVOXbZIUlP6sVcfTib6LA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=37)

将函数 U 全部拷贝到本地后：

```
function U() {...};  
  
function randomStr() {  
    let str = "";  
    for (let i = 0; i < 4; i++) {  
        str += (65536 * (1 + Math["random"]()) | 0)["toString"](16)["substring"](1);  
    }  
  
    return str;  
}  
  
var t = new U()["encrypt"](randomStr());  
  
var r = t['$_CCGb']()  
console.log(r)  

```

运行后报错

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoccAnjnLTYfZtoFhaiaZJuq3MAl0AQiboIqu12LRjG9hickwRv38CfcibSpg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=38)

将 `mwbxQ` 复制到控制台输出并跟进会发现是一个空函数，仔细看会发现上面有对 `mwbxQ` 操作，都拷贝到本地

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocNcPLNAgciaqBpGV9gkMBhN9rGY62BGpVlmQJqbxQ3MxU5uPmQQF5Licg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=39)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocPh1Zc1ibFQKHE78m0ph6nU35B6SDw5otH6tibtuydGPuMBEPyHeVYn5Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=40)

再次运行报错：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocMywGnJFw8YG5POnU3XeszrYWZibz3uUmvL6IxxrhdzdWQ00NuiaSLRcA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=41)

补一个：`window = global` 即可

再次报错：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKoc7a89J1bIfz6KDaJhD8icHGWhibYNX8XCXJPWtfpich9vMUCcjlPicm4HQQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=42)

到控制台输出 `ht` 会发现是 `Navigator` ，先将 `ht` 定义为空对象，后面再缺啥补啥：`ht = {}`

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocQ40Kl0E3eBzjWd7n5POvJbiaKic8oIj00iaVcE7ox0fpNAp0lpSukOc1Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=43)

再次运行，发现 `ht` 报错 `appName` 未定义，去浏览器搜索并下断点看看

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocJPQpVfd7rOTHsoaE3DPNWAar8t8eQa8qr4HfouNQnNtDlicuibdRXAHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=44)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocw8jMOOqk1dOG01gqxva1Wt5iaaIGW12xjyHr2VgGIvv1G7QibZ1mPk5Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=45)

点击验证断下后，发现 `ht["appName"]` 的值为 `"Netscape"`，拷贝到本地

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocMGY7IeY2Ww7MtyiaJsOjyhQwLyiaiahASWHTOiblPlicesn8UPjxy4NJibYA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=46)

```
let ht = {  
    appName: "Netscape"  
};  

```

再次运行，已经到 `r`变量部分报错了

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocIKyebo6H3985nWLwXVk6lVlOux076ic4Dsx0ib6ic0hVqqy5FIVfk0r3A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=47)

先试着输出 `t` 看看

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVjjNxDKYjPqUDyTwqicVtKocepXibPPHYTwyU4qUClFyPMibnkkDV3PCpAh5xWQFTpJ20MibpkgIdfrjA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=48)

已经可以得到数据，所以可以直接将 `u = t`，这里我直接按报错补了个函数

```
let t = {  
    $_CCGb: function () {  
        var t = new X()["encrypt"](randomStr());  
        return t;  
    }  
}  
// 或者 u = t  

```

至此 `w` 中的一个组成部分 `u` 已得到，以下为 `u` 部分的代码：

```
let window = globalThis;  
let ht = {  
    appName: "Netscape"  
};  
  
mwbxQ.$_Au = function() {...};  
mwbxQ.$_BR = function() {...};  
mwbxQ.$_Cg = function() {...};  
mwbxQ.$_DW = function() {...};  
function mwbxQ() {};  
  
var U = function() {...};  
  
function randomStr() {  
    let str = "";  
    for (let i = 0; i < 4; i++) {  
        str += (65536 * (1 + Math["random"]()) | 0)["toString"](16)["substring"](1);  
    }  
  
    return str;  
}  
  
var t = new U()["encrypt"](randomStr());  
console.log(t);  
  
var r = t['$_CCGb']()  
console.log(r)  

```

### 2.7.4 l 参数

可以发现 `h` 的参数是 `l`，而 `l = V[$_CAHJS(392)](gt[$_CAIAK(254)](o), r[$_CAIAK(744)]())`，所以先对 `l` 进行逆向

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWKZSZwCvnlsOuzkqZ9xRcGJ7u5WDqLWlqPxD5WIloFUw36zfJsojiacg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=49)

#### 2.7.4.1 参数 o

解混淆后：

```
l = V["encrypt"](gt["stringify"](o), r["$_CCEc"]())  

```

先看看后面的 `r["$_CCEc"]()`，会发现长度为16，有点熟悉

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWA2CVJKAXSqDP1G2uHK9EtsuXibrCTic5IchWZlyLgW17BToP2ibOpFSiag/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=50)

跟进去后，发现就是我们的 `randomStr`

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWQEVsicyMkPxxaZIb9MEHwNK02lwYGaFEtNiaaPFoufibiaHkNzX3iaJ3vGg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=51)

再看看 `gt["stringify"](o)`，可以发现，参数 `o` 原本为对象，经过 `gt["stringify"]` 处理后变成了字符串，而且 `$_CEEIS(457)` 的值为 `stringify`，所以 `ge` 就是 `JSON`，改为 `JSON.stringify(o)` 即可

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWibv7BxBWibpia3sHbiaia4pBxw8EnNQ6M9y6HgUpFBibIc7lF7I8dWpaNP9Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=52)

`o` 对象里有一些可疑的值，拿到本地看看

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWafUkTCmd02ZNl82EUpI4Xx0kvnpe6jCgyVv7fhrdciaAYyKkiaRwqkCw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=53)

经过分析：

*   userresponse : 需要逆向
    
*   rp: 需要逆向
    
*   aa : 需要逆向
    
*   passtime : 取轨迹最后一个时间
    
*   imgload : 可以固定
    
*   ep : 可以固定
    

##### rp :

先来看看rp，可以发现就在上面

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW5g0jJBBZCU1iba4r1ZYN86LQNJPZNl6az3DmiaqkHBRHeyB6oGDxKkibg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=54)

```
 o["rp"] = X(i["gt"] + i["challenge"]["slice"](0, 32) + o["passtime"]);
```

通过控制台调试后：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW8dHWtpCjibNwcRkbianyYqMqoeYiaH2NzRXCtUJLbMj2W9eVpFicHaicWicQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=55)

这些值都可以直接得到

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWklPBhpRib3iaPLEAQJtwBk8OrrMABYBs8VJKqNykU7vmWIOwib0tgrRew/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=56)

*   gt、challenge : get.php里
    
*   pasttime : 可以固定
    

再来看看 `X` 函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWDe38RqzmseZD6iaTy6z7wLbffZs7GUiasJJAv6FIqUHFzMOpnuYWPXdA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=57)

往下滑动可以发现 `MD5` 的特征码：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWJHE07ZiclZTwPUOvxrLmJdadErQR8VWTzrX7A4SebMGiawOLgJ5yrCBw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=58)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWfjzg0Dhp9m6PLcjYLPy4TrVrF2KGDlOGzsw6c5LIlvKAph0yC2c4AQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=59)

所以可以直接用 `JSEncrypt` 来实现

```
let JSEncrypt = require("crypto-js");  
let X = str => JSEncrypt.MD5(str).toString().toLowerCase();  

```

这里我直接拷贝到本地，运行后会有报错，只需要将报错部分放到控制台输出值再替换即可

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWH5OYvx0Jqbta7ZaWWvXD401NYW4icGebH3ibjicWtloMvpJPj9qajX9cQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=60)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW2uNAswvMNdUyzclQrFcdkoeztRLN9zzrxzZicRXm6N1uUyq0NiboqQGw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=61)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWHcTcxIMdR0t7YlfXXfv4O9WU6MUwYpQvbK0EcoDRxST1f8x1wY2oaQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=62)

rp值得到：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWqokpk6S2ia1GbrMbklNicTRWPFwl3yZQibjAqTAxc352INxagASlDH5KA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=63)

##### userresponse :

在 `w` 定义的地方，往上面翻可以发现 `o` 赋值的地方

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWQk6Gy0oictuZakQYqYjG5EUSvRboc6kg1ibOTXU3sSzt3VaxtIMpGfWQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=64)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWUkaKLjvbnF3MOOHib8M2pIBJFJNqlh9buaM2iczU53MsDXZr5UDqPtzQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=65)

可以发现 `userresponse` 的值为：

```
H(t, i["challenge"])  

```

*   t ：滑块滑动的距离，需要用ddddocr来识别
    
*   i[“challenge”] ：就是第二个 get.php 返回的 challange
    

进入 `H` 函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWI2nicnPgprDXovH8IOic3R8wKupjMYY2rLueDJG4ocspFbguqdqy92jQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=66)

直接将 `H` 方法拷贝到本地

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWUZ9uyY4GZ6u5RkgUSVZRfKSYZSictYGSiczwIZKjEViaa3vlZaDsStITw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=67)

运行后报错，还是输出到控制台进行替换

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW99y3lOEic59ok0a3c6wglBmicfYic1IEYh5bZqlqJDKvmxXcZecicVhpZg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=68)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWqNO3eh8qXuTG6CSCmoGYuQycVibibJJd9dVsxRgaocELK6InxeTtflwQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=69)

ps: 这里使用 `ast` 会快一点

最后得到结果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWy2FD8rFw3FH9DEfjBByfGXwr8oQVLajbC7RiaHkbFNXrmXIrBSIU9Yg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=70)

##### aa :

发现 `aa` 的值是由参数传进来的

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWS26dAZr3JXPlUSCuLicBDfwNIEmFEvgwfHmqPCJXEibPFoUFqp0r0wZg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=71)

断点到 `aa` 的下面，然后查看上一个栈

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWVPyNT9VvkGCz0nWdJvYhI8wV6KXqgVibb36K5EMKCLaVmFbUcQWDRAQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=72)

可以发现 `l` 就是我们的 `aa`

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWjcd82YtayJibZ8F2KVYXSaNVrdoDjia6SE2iaCjibZIt9q6vkRSNttXzSg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=73)

```
l = n['$_CICO']['$_BBED'](n[$_CJJJU(985)]['$_CICO'](), n['$_CJk']['c'], n['$_CJk']['s'])  

```

*   n[$_CJJJU(985)][‘$_CICO’] : 需要逆向
    
*   n[‘$_CJk’][‘c’] : 在第二个 `get.php` 中返回的数据 `c`
    
*   n[‘$_CJk’][‘s’] : 在第二个 `get.php` 中返回的数据 `s`
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWIjwk8IibSajiaCL9anPXibriadPQ02EpM1kB6lxChvzINBNlQ6w68BGosA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=74)

进入 `n[$_CJJJU(985)]['$_CICO']`

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWlCHib7vNZw6rbiarNS9XkABY4KX64ZgXOzE1IfKsYANXUEiaZjxHOo9hw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=75)

往下找发现最后返回 `r[$_BEGJO(405)]($_BEHAO(2)) + $_BEHAO(457) + i[$_BEGJO(405)]($_BEGJO(2)) + $_BEGJO(457) + o[$_BEGJO(405)]($_BEGJO(2))`，下断点并运行到此处，输出的也是我们要的数据

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWfyeXVGicd2Ls49H2KmbeUVG1WvjCnYXRrFEaJhmCjXGpIUQNQvboavA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=76)

将整个文件拷贝到本地，全部折叠后，搜索关键代码

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWibDtatY2ibOTSMrbZojoc2Vf0E726bEYnFSQT74d6qqspmugVGia0QdWQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=77)

这是一个由 `W` 构成的方法，去找找 `W`

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWficCh7YP4sT4HHMel0fU2C6PYJukUHrqh9nOh3vKtytR5UbibxQoYIWQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=78)

先将 `W` 和 `W[$_CJFt(236)]` 拷贝下来

再来看看 `t` 函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWkbfNxJDVaU3EThthoTt3YygF7VDic7kXrkFvtaKOLcSgxNKInEvla9A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=79)

经过分析，`this[$_BEGJO(359)]` 原轨迹传入函数后，变成了新的轨迹，并赋值给 `t`

后面需要我们自己生成轨迹传入函数，轨迹的生成可去 Github 上找

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWjIclclORvOp2rBEicmiaATUcTMlnelAo9I3E5nSaZZARGP4oN2lJIQBw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=80)

将 `t` 进行一个简单的改写：

```
"\u0024\u005f\u0046\u0044\u005f": function(track) { // 传入轨迹  
    var $_BEGJO = mwbxQ.$_Cg  
      , $_BEGI_ = ['$_BEHCp'].concat($_BEGJO)  
      , $_BEHAO = $_BEGI_[1];  
    $_BEGI_.shift();  
    var $_BEHBX = $_BEGI_[0];  
    function n(t) {  
        var $_DBEAu = mwbxQ.$_DW()[6][13];  
        for (; $_DBEAu !== mwbxQ.$_DW()[9][12]; ) {  
            switch ($_DBEAu) {  
            case mwbxQ.$_DW()[6][13]:  
                var e = $_BEGJO(497)  
                  , n = e[$_BEGJO(192)]  
                  , r = $_BEHAO(2)  
                  , i = Math[$_BEGJO(316)](t)  
                  , o = parseInt(i / n);  
                n <= o && (o = n - 1),  
                o && (r = e[$_BEHAO(125)](o));  
                var s = $_BEGJO(2);  
                return t < 0 && (s += $_BEGJO(418)),  
                r && (s += $_BEHAO(435)),  
                s + r + e[$_BEHAO(125)](i %= n);  
                break;  
            }  
        }  
    }  
    var t = function(t) {  
        var $_BEHEp = mwbxQ.$_Cg  
          , $_BEHDn = ['$_BEHHY'].concat($_BEHEp)  
          , $_BEHFW = $_BEHDn[1];  
        $_BEHDn.shift();  
        var $_BEHGj = $_BEHDn[0];  
        for (var e, n, r, i = [], o = 0, s = 0, a = t[$_BEHFW(192)] - 1; s < a; s++)  
            e = Math[$_BEHFW(158)](t[s + 1][0] - t[s][0]),  
            n = Math[$_BEHFW(158)](t[s + 1][1] - t[s][1]),  
            r = Math[$_BEHFW(158)](t[s + 1][2] - t[s][2]),  
            0 == e && 0 == n && 0 == r || (0 == e && 0 == n ? o += r : (i[$_BEHFW(137)]([e, n, r + o]),  
            o = 0));  
        return 0 !== o && i[$_BEHFW(137)]([e, n, o]),  
        i;  
    }(track) // 将轨迹作为参数传入  
      , r = []  
      , i = []  
      , o = [];  
    return new ct(t)[$_BEHAO(59)](function(t) {  
        var $_BEHJU = mwbxQ.$_Cg  
          , $_BEHIM = ['$_BEICs'].concat($_BEHJU)  
          , $_BEIAi = $_BEHIM[1];  
        $_BEHIM.shift();  
        var $_BEIBp = $_BEHIM[0];  
        var e = function(t) {  
            var $_BEIE_ = mwbxQ.$_Cg  
              , $_BEIDR = ['$_BEIHQ'].concat($_BEIE_)  
              , $_BEIFM = $_BEIDR[1];  
            $_BEIDR.shift();  
            var $_BEIGI = $_BEIDR[0];  
            for (var e = [[1, 0], [2, 0], [1, -1], [1, 1], [0, 1], [0, -1], [3, 0], [2, -1], [2, 1]], n = 0, r = e[$_BEIFM(192)]; n < r; n++)  
                if (t[0] == e[n][0] && t[1] == e[n][1])  
                    return $_BEIE_(441)[n];  
            return 0;  
        }(t);  
        e ? i[$_BEHJU(137)](e) : (r[$_BEIAi(137)](n(t[0])),  
        i[$_BEHJU(137)](n(t[1]))),  
        o[$_BEIAi(137)](n(t[2]));  
    }),  
    r[$_BEGJO(405)]($_BEHAO(2)) + $_BEHAO(457) + i[$_BEGJO(405)]($_BEGJO(2)) + $_BEGJO(457) + o[$_BEGJO(405)]($_BEGJO(2));  
}  

```

试着调用一下：

```
console.log(W[$_CJFt(236)]["\u0024\u005f\u0046\u0044\u005f"]);  

```

报错：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWQyCGcIkPiczI6J2hmkIA7HOgX8wHVsI2gBGRYobg0b5C2BK1sFX3D1Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=81)

进行替换即可

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWt4YT5SZS5xWXCGxSC2D85xCfBt5KgpBMnwYIgU1KJbCOPJC8cqMNrg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=82)

试着将轨迹传入：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW7yxWCf2CK7eXm2d60YSPrtTv53yiboenj2AggbmLJGuqXfKfrKHU9Hg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=83)

```
console.log(W['prototype']["\u0024\u005f\u0046\u0044\u005f"]([[-29,-28,0],[0,0,0],[1,0,31],[2,0,41],[4,0,48],[6,0,55],[8,0,62],[9,0,68],[11,-1,73],[13,-1,79],[15,-1,84],[16,-1,90],[18,-1,94],[20,-2,98],[22,-2,103],[23,-2,110],[23,-4,113],[25,-4,115],[27,-4,119],[29,-4,123],[30,-4,129],[30,-6,132],[32,-6,136],[34,-6,142],[36,-6,148],[37,-6,156],[37,-8,161],[39,-8,166],[39,-8,228]]));  

```

运行后报错 `ct` 未定义：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWGFTZ2sqeSPppp3QdTKmHpx7wO9Xav9djvB3gtfToSMvE9DibX3piaLVg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=84)

搜索后发现 `ct` 还有其他地方的属性，都拷贝到本地

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWDkg0sBtjPEzVbTYBRKDlIIeK6WGO65cerMvEz2G0vuapVGcchdzxkg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=85)

运行后报错：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWaAiazibCt2bzYuQWJJKprVXEiblemm6oXOiah5zRxhwka9rR4QTyYXKmVA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=86)

还是进行替换即可

最后成功输出值：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW847wLkIAfnS36nd2ynWM69thmJdcFicEm9x4toHZuXM0tRibiaghgqUew/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=87)

`n['$_CICO']['$_BBED']`的第一个参数 `(n[$_CJJJU(985)]['$_CICO']()`解决完毕，接下来分析 `n['$_CICO']['$_BBED']`

进入函数：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWA5GBic30rSTN1mVxu9zESVR5B5KBaQqPQruXXCG3ueWHN1cHibAicGdtQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=88)

向上翻一翻，会发现很熟悉，该方法也是在 `W` 里

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWPo0yJQSBYJTtGNpNpAgpPmj7lxLbd51GqPYhcC4JV9cOVB5Ete9iaEA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=89)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWx1vUnKGyoKicKXumsHBWCUrgxdwYCteoo9bQWPqrQnQibxpO9c9Y7p8Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=90)

所以可以直接进行改写，先将 `l` 拿下来，也就是 `aa`：

```
l = n[$_DAAAU(985)][$_CJJJU(1075)](n[$_CJJJU(985)][$_CJJJU(1073)](), n[$_CJJJU(67)][$_CJJJU(1033)], n[$_DAAAU(67)][$_CJJJU(345)])  

```

改写后：

```
l = w['prototype']["\u0024\u005f\u0042\u0042\u0045\u0044"](n[$_CJJJU(985)][$_CJJJU(1073)](), n[$_CJJJU(67)][$_CJJJU(1033)], n[$_DAAAU(67)][$_CJJJU(345)])  

```

将里面的值固定验证一下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWjI4njV69iav7bdQb2oicPmKWXgJtuibUNRIjXwkNXTLkA66bxUV8sicErg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=91)

```
l = w['prototype']["\u0024\u005f\u0042\u0042\u0045\u0044"]('J((((!!Isstttszttstzts!*ttts!*ttts!*t(!!(L3000/././--.0,*--/,-//1..p', [12,58,98,36,43,95,62,15,12], '7764656c')
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW7nTXJlibGQUibdk9MSLia3xfTIQKoFDCwqPD9zNiaPXvxMV3pGxuz91k0A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=92)

至此，`o` 参数里的值都解决了

```
function ct(t) {...};  
ct['prototype'] = {...};  
ct['$_BBJN'] = function(t) {...};  
function W(t) {...};  
W['prototype'] = {...};  
function H(t, e) {...};  
  
function get_o() {  
    let o = {  
        "lang": "zh-cn",    // 固定  
        "userresponse": "", // 需要逆向  
        "passtime": 228,    // 滑块滑动时间，取轨迹最后时间  
        "imgload": 77,      // 图片加载时间 随机值  
        "aa": "",           // 需要逆向  
        "ep": {...},  		// 可以固定，也可以随机  
        "h9s9": "1816378497",       // 固定值  
        "rp": ""            // 需要逆向  
    }	  
    parama_one = W['prototype']["\u0024\u005f\u0046\u0044\u005f"]([[-29,-28,0],[0,0,0],[1,0,31],[2,0,41],[4,0,48],[6,0,55],[8,0,62],[9,0,68],[11,-1,73],[13,-1,79],[15,-1,84],[16,-1,90],[18,-1,94],[20,-2,98],[22,-2,103],[23,-2,110],[23,-4,113],[25,-4,115],[27,-4,119],[29,-4,123],[30,-4,129],[30,-6,132],[32,-6,136],[34,-6,142],[36,-6,148],[37,-6,156],[37,-8,161],[39,-8,166],[39,-8,228]]);  
    o["rp"] = X("019924a82c70bb123aae90d483087f94" + "c8adadebff0d331137fd730423f4c637" + 53857)  
    o["userresponse"] = H(19, "c8adadebff0d331137fd730423f4c637jr");  
    o["aa"] = W['prototype']["\u0024\u005f\u0042\u0042\u0045\u0044"](parama_one, [12,58,98,36,43,95,62,15,12], '7764656c');  
  
    return o  
};  

```

现在回到 `w` 赋值的地方

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWvhrq7fPe2IbRBdIHpW6lIqKCKQzQpzjKEqPSWCD6NZ2TpbqZdpsKBw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=93)

```
l = V[$_CAHJS(392)](gt[$_CAIAK(254)](o), r[$_CAIAK(744)]())  

```

再重温一遍：

*   `gt[$_CAIAK(254)]`：JSON.stringfy()
    
*   `o`：对象，里面的 rp、userresponse、aa 已经逆向
    
*   `r[$_CAIAK(744)]()`：我们自己构造的 `randomStr` 函数返回的16位随机值
    

接下来就是 `V[$_CAHJS(392)]`

跟进函数：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWVOMhebhTj48q6JjicfkeyCvl4sHmW2bwKHyIooSs3AEzmSxKfPzZyVA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=94)

拷贝到本地，折叠，搜索关键代码

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWy8hMM2ByjtDLn9qlwVy5hl1vpria6xjLwjxIlCC5OSCrHkZqKlEm5ag/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=95)

可以发现是在 `V` 里面，将 `V` 整个拷贝下来并进行改写：

```
l = V['encrypt'](JSON['stringify'](o), randomStr());  

```

可以直接输出值：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWQt9FNKBzIJDeJQyIVtdc5a6DCcEXCtYHe9ta6fdSTXOBCia944hMSqQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=96)

`l` 参数也完成了

#### 2.7.5 h 参数

接下来就是最后一个参数 `h` 了

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWOOblChWJPNbibbNSy0ZHW2ppY353HbTePVCWXoiaxAEGGJ9xwcVxicOkQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=97)

```
h = m[$_CAIAK(792)](l)  

```

`l` 参数已经逆向了，进入 `m[$_CAIAK(792)]` 看看

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW4tTWwH385N0yyALBsooDeWk7payFc6plIaX0UXakESzXRjoQicqNnPw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=98)

还是拷贝到本地，折叠，搜索关键代码

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWdz8ECur20ZY3M7RXQKVDqbn4hXfF5kMKVZ6nsYiabzLogrd8G7H8rRg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=99)

将 `m` 对象拷贝下来

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW70qiaFmBGusl9UsRPOCgaIzYuB2bEicXz0ib3Ad8QibcHa03B26jTb54dw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=100)

这种报错不用多说了

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWwd9FaSHbmxSGgibrUNsYPOHX6B8w5u7lLnGYkEp5WZMYFGgTVdsndow/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=101)

成功获取到 `h` 的值

所以 `w` 为：

```
let w = h + u  

```

最后将一些固定的值改为变量即可。

  

三、底图的还原
-------

### 3 .1 缺口图

打开元素检查，点击图片后，会发现图片由 `canvas` 绘制而成

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWPv1AkugEFL0Cu0J6TIXJhocEoJOUicuyv3vsHyey1icXUztbz16TYrFw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=102)

所以可以在 `源代码/来源` 面板，`事件监听器断点` 中画布下打开 `创建画布背景` 断点

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWIqiajTavwv8NVFQ1amn5M47FYmHWVuianOAHnBg2fmC9amjKoIcbMM5w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=103)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWPgzWakziaaHicz49hNHlj7qo9mHx4pqHFLVaLOAX31ynnRIvXibic0Uf9A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=104)

先是创建了一个 `312 × 160` 的二维画布，和乱序图的长宽一样

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWy5cMfPYZmxCiaaXhSylR0nR8BiczlXe3iaLOSJFdbNMMzrQI5R1aNib27g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=105)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW3V5t5ggYrqKq3M38r3j3NRMJnyRO8Ek1aiamnOicKZrZ282GRvQaA4jg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=106)

再看看下面的 case分支，长和宽与还原后的图片一样

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWaDDAhw5iafGr6vAL2dqYdspzicOMgtgIeReE77iaxibOCe60XNB3eGdxRg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=107)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIWdJ9QZTfibuNXqBkeo0555UjumGMVGapDseMricI2aJzURpf0GpB91W4g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=108)

主要的还原代码：

```
for (var a = r / 2, _ = 0; _ < 52; _ += 1) {  
    var c = Ut[_] % 26 * 12 + 1  
      , u = 25 < Ut[_] ? a : 0  
      , l = o[$_CJES(34)](c, u, 10, a);  
    s[$_CJFt(49)](l, _ % 26 * 10, 25 < _ ? a : 0);  
}  

```

*   先来看 `Ut`，Ut里面的数据就是还原的顺序，值是固定的
    
*   `r` 的值为 160，也就是宽， `r / 2` 就是将图片分成上下两部分，后面的 `_` 代表图片总共被分为 52 份，上下各 26
    
*   `c` 表示要提取矩形左上角的 x 轴坐标，`u` 表示要提取的矩形左上角的 y 轴坐标
    
*   `l` 里，`$_CJES(34) = "getImageData"`，返回一个 `ImageData` 对象，用于描述 canvas 指定区域的隐含像素数据，表示从 `c(x)，u(y)` 处取 `10` 矩形的宽度和取 `a(0 or 80)` 矩形的高度
    
*   `s` 里，`$_CJFt(49) = "putImageData"`，用于将数据从已有的 `ImageData`对象绘制到画布上，第一个参数为 `imageData` 对象，也就是 `l`，`_ % 26 * 10` 表示每块的 `x` 取 `10px`，`25 < _ ? a : 0` 表示上下两部分的起始 `y`，刚开始从上半部分拼接，最后再拼接下半部分
    

最后按着这个代码进行还原即可

可以看看这篇还原文章：

```
https://blog.csdn.net/qq_43511026/article/details/128835230  

```

还要注意滑块外围的空白部分，可以用 `cv2` 来识别滑块的边框

![图片](https://mmbiz.qpic.cn/mmbiz_png/fKzkeicO1OVhO2kdqm3b85TiaY1UYZtbIW1Xuw1I98qVxpjQ7uAjNXsZob7Rgp9RrLTHVVlGXrTWTmouyk3uLEJQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=109)

相关资料：

```
https://www.cnblogs.com/DarrenChan/p/5588638.html  
https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/getImageData  
https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/putImageData  

```