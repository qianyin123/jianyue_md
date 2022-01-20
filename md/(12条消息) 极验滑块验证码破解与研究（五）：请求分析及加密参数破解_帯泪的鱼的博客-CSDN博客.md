> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_42857999/article/details/121674134)

### 极验滑块验证码破解与研究（五）：请求分析及加密参数破解

*   *   [声明](#_3)
    *   [一、极验请求分析](#_19)
    *   *   [1. 滑块测试网站入口](#1__23)
        *   [2. 滑块验证过程抓包](#2__28)
        *   [3. 请求详解](#3__32)
        *   *   [3.1. register-slide](#31_registerslide_33)
            *   [3.2. gettype.php](#32_gettypephp_53)
            *   [3.3. get.php](#33_getphp_92)
            *   [3.4. ajax.php](#34_ajaxphp_148)
            *   [3.5. get.php](#35_getphp_175)
            *   [3.6. ajax.php](#36_ajaxphp_256)
        *   [4. 相关js文件介绍](#4_js_283)
        *   *   [4.1. fullpage.9.0.7.js](#41_fullpage907js_284)
            *   [4.1. slide.7.8.4.js](#41_slide784js_286)
    *   [二、js破解前准备工作](#js_289)
    *   *   [1. slide.7.8.4.js反混淆](#1_slide784js_290)
        *   [2. 找到w参数](#2_w_293)
        *   [3. 使用chrome插件reres替换js文件](#3_chromereresjs_297)
    *   [三、w参数破解](#w_305)
    *   *   [1. w参数分析](#1_w_306)
        *   [2. _ 参数分析](#2____314)
        *   *   [2.1. 参数 _ 源码分析](#21_____315)
            *   [2.2. 扣扣扣，改改改（构造 _ 参数）](#22_____341)
        *   [3. l 参数分析](#3_l__377)
        *   *   [3.1. 参数 l 源码分析](#31__l__378)
            *   [3.2. 参数 o 源码分析](#32__o__382)
            *   [3.3. 扣扣扣，改改改（构造 l 参数）](#33__l__451)
        *   [4. 最后一步咯：w = l + _](#4_w__l____519)
    *   [四、结语](#_537)
    *   *   [*本期文章结束啦，如果对您有帮助，记得收藏加关注哦，后期文章会持续更新 ~~~*](#__539)

声明
--

**原创文章，请勿转载！**

**本文内容仅限于安全研究，不公开具体源码。维护网络安全，人人有责。**

**本文关联文章超链接：**

1.  [极验滑块验证码破解与研究（一）：AST还原混淆JS](https://blog.csdn.net/qq_42857999/article/details/121586389)
2.  [极验滑块验证码破解与研究（二）：缺口图片还原](https://blog.csdn.net/qq_42857999/article/details/121628908)
3.  [极验滑块验证码破解与研究（三）：滑块缺口识别](https://blog.csdn.net/qq_42857999/article/details/121635961)
4.  [极验滑块验证码破解与研究（四）：滑块轨迹构造](https://blog.csdn.net/qq_42857999/article/details/121659205)
5.  [极验滑块验证码破解与研究（五）：请求分析及加密参数破解](https://blog.csdn.net/qq_42857999/article/details/121674134)

**本篇文章篇幅较长，但是讲的特别细，小伙伴们耐心看完哦 ~**  
  

一、极验请求分析
--------

首先来看一张图，极验的官方请求流程  
![在这里插入图片描述](https://img-blog.csdnimg.cn/4336fadc894147398bf028a61aca533a.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

### 1. 滑块测试网站入口

```
`aHR0cHM6Ly93d3cuZ2VldGVzdC5jb20vZGVtby9zbGlkZS1wb3B1cC5odG1s` 

*   1


```

### 2. 滑块验证过程抓包

![在这里插入图片描述](https://img-blog.csdnimg.cn/3178c8c8e71842c09242630f9956059c.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

### 3. 请求详解

#### 3.1. register-slide

**请求介绍**  
首次获取gt（验证 id，极验后台申请得到）、challenge（验证流水号，服务端 SDK 向极验服务器申请得到）参数。

**请求参数**

```
`t: 1638435080881` 

*   1


```

**请求响应**  
gt、challenge非常重要，后面的请求参数都会携带这两个参数，其中gt为定值，challenge会变化。

```
`{
    "challenge": "b6ae4c5464ef63f4564073ce0c504b9f",
    "gt": "019924a82c70bb123aae90d483087f94",
    "new_captcha": true,
    "success": 1
}` 

*   1
*   2
*   3
*   4
*   5
*   6


```

#### 3.2. gettype.php

**请求介绍**  
该请求的目的是获取极验核心js文件版本信息，js文件地址信息。

**请求参数**  
实际破解滑块过程中，此请求可忽略。

```
`gt: 019924a82c70bb123aae90d483087f94
callback: geetest_1638435083096` 

*   1
*   2


```

**请求响应**

```
`{
    "status": "success",
    "data": {
        "type": "fullpage",
        "static_servers": [
            "static.geetest.com/",
            "dn-staticdown.qbox.me/"
        ],
        "beeline": "/static/js/beeline.1.0.1.js",
        "voice": "/static/js/voice.1.2.0.js",
        "click": "/static/js/click.3.0.2.js",
        "fullpage": "/static/js/fullpage.9.0.8.js",
        "pencil": "/static/js/pencil.1.0.3.js",
        "slide": "/static/js/slide.7.8.6.js",
        "geetest": "/static/js/geetest.6.0.9.js",
        "aspect_radio": {
            "slide": 103,
            "click": 128,
            "voice": 128,
            "pencil": 128,
            "beeline": 50
        }
    }
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24


```

#### 3.3. get.php

**请求介绍**  
该请求主要是通过执行fullpage.js文件，收集浏览器指纹数据数据，加密生成w参数，如果该请求伪造得足够好，会触发极验无感验证。

**请求参数**  
本章只研究极验滑块，因此构造此请求时，w传空字符，不触发无感验证。

```
`gt: 019924a82c70bb123aae90d483087f94
challenge: b6ae4c5464ef63f4564073ce0c504b9f
lang: zh-cn
pt: 0
client_type: web
w: 
callback: geetest_1638435082792` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7


```

**请求响应**  
响应中的c、s在后续无感验证生成w参数时需要使用，其中c为定值，s为变化值。

```
`{
    "status": "success",
    "data": {
        "theme": "xxx",
        "theme_version": "1.5.8",
        "static_servers": [
            "static.geetest.com",
            "dn-staticdown.qbox.me"
        ],
        "api_server": "api.geetest.com",
        "logo": true,
        "feedback": "https://www.geetest.com/contact#report",
        "c": [12, 58, 98, 36, 43, 95, 62, 15, 12],
        "s": "744a7751",
        "i18n_labels": {
            "copyright": "由极验提供技术支持",
            "error": "网络不给力",
            "error_content": "请点击此处重试",
            "error_title": "网络超时",
            "fullpage": "智能检测中",
            "goto_cancel": "取消",
            "goto_confirm": "前往",
            "goto_homepage": "是否前往验证服务Geetest官网",
            "loading_content": "智能验证检测中",
            "next": "正在加载验证",
            "next_ready": "请完成验证",
            "read_reversed": false,
            "ready": "点击按钮进行验证",
            "refresh_page": "页面出现错误啦！要继续操作，请刷新此页面",
            "reset": "请点击重试",
            "success": "验证成功",
            "success_title": "通过验证"
        }
    }
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35


```

#### 3.4. ajax.php

**请求介绍**  
该请求是极验验证请求，fullpage.js将再次收集浏览器指纹数据，加密生成w参数，若浏览器指纹构造的足够好，无感验证通过，直接返回validate；若无感验证未通过，则需要加强验证，返回验证类型，其中包括滑块、图形点选、文字点选等。

**请求参数**  
本章只研究极验滑块，因此构造此请求时，w传空字符，让无感验证失败。

```
`gt: 019924a82c70bb123aae90d483087f94
challenge: b6ae4c5464ef63f4564073ce0c504b9f
lang: zh-cn
pt: 0
client_type: web
w: 
callback: geetest_1638435091829` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7


```

**请求响应**  
返回极验验证码类型slide（滑块验证）。

```
`{
    "status": "success", 
    "data": {
        "result": "slide"
    }
}` 

*   1
*   2
*   3
*   4
*   5
*   6


```

#### 3.5. get.php

**请求介绍**  
该请求获取滑块验证需要的缺口图地址、带缺口的背景图地址、c、s、gt、challenge参数。

**请求参数**

```
`is_next: true
type: slide3
gt: 019924a82c70bb123aae90d483087f94
challenge: b6ae4c5464ef63f4564073ce0c504b9f
lang: zh-cn
https: true
protocol: https://
offline: false
product: popup
api_server: api.geetest.com
isPC: true
autoReset: true
width: 100%
callback: geetest_1638435082551` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14


```

**请求响应**  
响应中s和challenge参数发生变化，gt和c值不变，这4个参数非常重要，在生成w加密参数是需要用到它们。

```
`{
    "gt": "019924a82c70bb123aae90d483087f94",
    "challenge": "b6ae4c5464ef63f4564073ce0c504b9fdx",
    "id": "ab6ae4c5464ef63f4564073ce0c504b9f",
    "bg": "pictures/gt/d401d55fc/bg/f9e8594ad.jpg",
    "fullbg": "pictures/gt/d401d55fc/d401d55fc.jpg",
    "link": "",
    "ypos": 72,
    "xpos": 0,
    "height": 160,
    "slice": "pictures/gt/d401d55fc/slice/f9e8594ad.png",
    "api_server": "https://api.geetest.com",
    "static_servers": [
        "static.geetest.com/",
        "dn-staticdown.qbox.me/"
    ],
    "mobile": true,
    "theme": "xxx",
    "theme_version": "1.2.6",
    "template": "",
    "logo": true,
    "clean": false,
    "type": "multilink",
    "fullpage": false,
    "feedback": "https://www.geetest.com/contact#report",
    "show_delay": 250,
    "hide_delay": 800,
    "benchmark": false,
    "version": "6.0.9",
    "product": "popup",
    "https": true,
    "width": "100%",
    "show_voice": true,
    "c": [12, 58, 98, 36, 43, 95, 62, 15, 12],
    "s": "74632f4e",
    "so": 0,
    "i18n_labels": {
        "cancel": "取消",
        "close": "关闭验证",
        "error": "请重试",
        "fail": "请正确拼合图像",
        "feedback": "帮助反馈",
        "forbidden": "怪物吃了拼图，请重试",
        "loading": "加载中...",
        "logo": "由极验提供技术支持",
        "read_reversed": false,
        "refresh": "刷新验证",
        "slide": "拖动滑块完成拼图",
        "success": "sec 秒的速度超过 score% 的用户",
        "tip": "请完成下方验证",
        "voice": "视觉障碍"
    },
    "gct_path": "/static/js/gct.e7810b5b525994e2fb1f89135f8df14a.js"
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48
*   49
*   50
*   51
*   52
*   53
*   54


```

#### 3.6. ajax.php

**请求介绍**  
该请求是极验验证请求，slide.js收集滑动轨迹，与上个请求中的gt、challenge、c、s参数，加密生成w参数。

**请求参数**  
gt、challenge参数在上个请求的响应中，w参数是本文章的分析对象，构造过程继续往下看哦。

```
`gt: 019924a82c70bb123aae90d483087f94
challenge: b6ae4c5464ef63f4564073ce0c504b9fdx
lang: zh-cn
pt: 0
client_type: web
w: NkI64(ZcHfs3JycKL35lnaf762N7xGhvcy)NV1ozrmUUYMEG5Ymn90E8N68t1osufnaPZf7iDs)UM3NSL7L8uP)DJdN0g8JNVf13NXMYiY4vbx4WbNFqX)gi3vo)HyDLaqlt7d3d9YLmLBcmLO4q9VGVIDJ9SetiIu2onjaDLW69FNr38GiLCXmeYjMPFxfN7UZ5fp217VVvbyaPCnacWkuj49yF67NRzwcHA)o2TWmnFGdplaahfE)tT5Zle6MeFv3xolFZ(7YgZgvtC2OP3h35sxvHObEdrMQBggKO96ne4A)Wcsy1DcGkAdF5wcpi2HTKSqLkpV(1(pOGizo0yf7IpjozhMNttEmyWxLe8eOq3Yae1YGKz5UQPgWLrnx8ZHPn9ZwS)HRCuvy6L2sKb7BOQ(EkaKRUfZ)XoeuIEJmrTkqI(kBhoQIytNg)KPTNzq1pQuVo)3RfTpy2L(tQYeq38XR5sl4etkJ84Kk5OZRgFzta9R2mD6rGS0a0aEYhO5EHcgv)STJ08Qk4pch4D7ToF7bFAhTTGDIsfIDAEPYX(QtqNf1V2csyj)D2X2l(4FJTZcKF0xR1Y2UpBMFI3n7hntZ8raKs3frabKsw5W)uuS1yIVkU22zlnHoA4vBDcDpiV81wtIAEbFq4X8K7wtf08Eu1uJc(bqqp5lAMG1H2YxxZH5RTj8zLRUwRlnK70sUHFT5)2WkfbTkjWqNgmn5w(NaQeMfsghBr4eOZl)4kro0ui4MzoZCEt960hcLL4KbRDW6MNx1py3A3n7f7BjCL4QfAKw8WmtB3eRmwxMfUGReu)zUBMiOxxOujFIWIkX9iInDeb68ujMzUSloUQ5uVO(1(PRbbDDawKbO7poo.3b3906f41fc68bc5724d4628d02ebfe941a9fcf7c60b51534d3d4f413ec4120fac2af09363d56dfb4548139f1f39e618cf8df2da148d9eb23f4fb2e1b5b2a67548ad169620b0c1a1037c5786aba3f483c2248c0fce7611cb785edc748be1b0cf89390a86d1624b3cdd6582e5dad9549ec0e1d0ee9bbe6028611b5308698520bc
callback: geetest_1638435090054` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7


```

**请求响应**  
验证成功，拿到validate参数。

```
`{
    "success": 1,
    "message": "success",
    "validate": "c563114bf49f259172e087fbdb9bbb61",
    "score": "1"
}` 

*   1
*   2
*   3
*   4
*   5
*   6


```

### 4. 相关js文件介绍

#### 4.1. fullpage.9.0.7.js

极验的滑块、点选、语序等都会存在的js文件，主要是用于获取浏览器指纹信息，进行无感验证及相关加密。

#### 4.1. slide.7.8.4.js

极验滑块验证核心文件，主要是用于收集用户滑块轨迹，进行滑块验证，本文研究重点js文件。

二、js破解前准备工作
-----------

### 1. slide.7.8.4.js反混淆

通过AST语法树将混淆的slide.7.8.4.js文件还原，具体还原方式请看 [极验滑块验证码破解与研究（一）：AST还原混淆JS](https://blog.csdn.net/qq_42857999/article/details/121586389)

### 2. 找到w参数

在反混淆后的slide.7.8.4.js文件中搜索 “w”，即可找到今天破解的主角，并加入一行debugger。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/f48975d0e65c4c73b9ff9e4d444ba835.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

### 3. 使用chrome插件reres替换js文件

配置chrome插件reres，将网页加载的js文件替换成对应的反混淆后的js文件。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/fdba3e5561ed462aab9b50138378ea47.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

打开开发者工具，刷新网页，触发滑块操作。然后，就可以任意打断点分析啦。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/054f4cd3e21045208215bb19277ad0b0.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

三、w参数破解
-------

### 1. w参数分析

从w生成位置开始往上一步一步分析，看看生成w参数需要哪些关键函数。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/c8b175312ced43dbac9fabe3e120d1a6.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

下面分析对w参数的分析过程，依照下图所示。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/5cd346fd85444377ac467abfd1afc318.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

### 2. _ 参数分析

#### 2.1. 参数 _ 源码分析

第一步：找到 _参数生成源码

```
`var _ = n["rdgJ"]();` 

*   1


```

第二步：进入 n[“rdgJ”] 函数，扣出需要的Q、t[“skuf”]函数代码

```
`"rdgJ": function (e) {
    var t = this;
    var r = new Q()["encrypt"](t["skuf"](e));  // 需要Q函数和t["skuf"]函数
    return r;
}` 

*   1
*   2
*   3
*   4
*   5


```

第三步：进入 t[“skuf”] 函数，扣出 t[“skuf”] 函数

```
`"skuf": function () {
    var t = ne();  // 需要ne函数
    return function (e) {
        if (e === true) {
            t = ne();
        }
        return t;
    };
}()` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9


```

#### 2.2. 扣扣扣，改改改（构造 _ 参数）

```
`// 抠出第三步相关函数
var ne = function () {
    function S4() {
        return ((1 + Math["random"]()) * 65536 | 0)["toString"](16)["substring"](1);
    }

    return function () {
        return S4() + S4() + S4() + S4();
    };
}()

// 抠出第二步相关函数
// 因为f = te["encrypt"](xe["stringify"](o), n["skuf"]())，所有此函数在破解f参数时也会用到
var skuf = function () {
    var t = ne();
    return function (e) {
        if (e === true) {
            t = ne();
        }
        return t;
    };
}()
// 这个函数太长了，源码就不粘贴出来了，小伙伴们自己扣哦
var Q = function () {}()

// 最后，将第二步中函数修改下，作为获取 _ 参数的主体函数
var rdgJ = function (e) {
    var r = new Q()["encrypt"](skuf(e));  // 需要Q函数和t["skuf"]函数
    return r;
}
console.log("_ 参数: "+ rdgJ())` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31


```

**到这儿，_ 参数就构造好啦，有没有很简单的样子。一不小心 w 参数破解一半 …**

### 3. l 参数分析

#### 3.1. 参数 l 源码分析

l参数的构造比较复杂，破解过程会分的比较细，下图详细分析了l参数的构造主体。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/4717ffeb98cb4e2780f13a1a997bc130.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 3.2. 参数 o 源码分析

o参数源码如下，下面一个一个分析。

```
`var o = {
    "lang": i["lang"] || "zh-cn",
    "userresponse": $(e, i["challenge"]),
    "passtime": r,
    "imgload": n["aFie"],
    "aa": t,
    "ep": n["quyY"]()
};` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8


```

**o[“lang”]=i[“lang”] || "zh-cn"**  
该参数取 i[“lang”] || “zh-cn” 的结果，所以可以默认为：o[“lang”] = “zh-cn”

**o[“ep”]=n[“quyY”]()**  
这个参数可以为空对象，所以本文不深究，令 o[“ep”] = {} 即可

```
`"quyY": function () {
    return {
        "v": "7.8.4",
        "te": Ee["touchEvent"],
        "me": Ee["mouseEvent"],
        "tm": new iRjl()["wCnl"](),  // 鼠标事件的一些时间戳
        "td": this["td"] || -1
    };
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9


```

**o[“imgload”]=n[“aFie”]**  
网页加载滑块验证码图片的加载时长，值可以自己构造，30-80左右（随便），单位毫秒  
想研究的小伙伴可以在slide.js源码中检索"aFie"，下面是获取imgload的部分源码

```
`var s = oe();  // 获取当前时间戳
r["smhf"]()["ECoQ"](function (e) {
    n["ZrSs"](e);  // 乱序的验证码背景图还原
    r["aFie"] = oe() - s;  // 获取当前时间戳 - s，得到imgload
    i["tSpO"](Ae);
}, function () {
    return Z(J("url_picture", r));
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8


```

**o[“passtime”]=r**  
此参数为滑动滑块过程消耗的时长，单位毫秒。通多调用栈跟踪参数r，找到生成源码。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/d7cef32ce5bd49509a55d730fb50eb09.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/a0a6198c99f04603ae6abb00d572ed91.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

通过上图跟踪，找到 n[“Ibbj”] = oe()，oe函数返回值是当前时间戳。重点在于 n[“Ibbj”] = oe() 在触发滑块时就执行，r[“QPWo”] = oe() - r[“Ibbj”]在松开滑块时执行的，因此passtime近似等于滑动过程总消耗时长。

**o[“userresponse”] = $(e, i[“challenge”])**  
userresponse生成过程需要函数 $（自己抠出来就行了），参数e：滑动的距离，参数challenge。  
通多调用栈跟踪参数e，方法同o[“passtime”]  
![在这里插入图片描述](https://img-blog.csdnimg.cn/04d695cd833340b49379ed6dcf5c88a6.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

**o[“aa”]=t**  
通多调用栈跟踪参数t，方法同o[“passtime”]。t生成方式如下

```
`var f = r["LxVe"]["pMye"](r["LxVe"]["Umdc"](), r["wrnn"]["c"], r["wrnn"]["s"]);` 

*   1


```

r[“wrnn”][“c”]：get.php请求返回值c  
r[“wrnn”][“s”]：get.php请求返回值s  
r[“LxVe”][“pMye”]：一个静态方法，可以直接扣出来  
r[“LxVe”][“Umdc”]()：对滑动轨迹进行加密，返回加密后的结果  
![在这里插入图片描述](https://img-blog.csdnimg.cn/d6d81f5df19e4e49b22f962826a7d8fc.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 3.3. 扣扣扣，改改改（构造 l 参数）

通过上面的分析，扣出相关的源码，把生成l参数的主体函数写好，之后再慢慢分析参数o。

```
`// 源码太长了，小伙伴们自己扣出来哦
var d = {};
var xe = function() {}();
var te = function() {}();
function K(e) {}
var $ = function (e, t) {}
function angq(e) {}
angq["prototype"] = {}
angq["vgQA"] = function (e) {}
var pMye = function (e, t, r) {}

// 有修改的函数
var Umdc = function (arr) {
        // 这三个自己扣哦
    function e(e) {}
    function r(e) {}
    function n(e) {}
    var t = e(arr);
    var i = [],
        o = [],
        a = [];
    new angq(t)["ofuT"](function (e) {
        var t = n(e);

        if (!t) {
            i["push"](r(e[0]));
            o["push"](r(e[1]));
        } else {
            o["push"](t);
        }

        a["push"](r(e[2]));
    });
    return i["join"]("") + "!!" + o["join"]("") + "!!" + a["join"]("");
}

// 根据分析写出参数l的主体函数
function get_l(c, s, gt, challenge, slide_track, imgload) {
    /* 参数说明
        c: 图片信息的 c; eg: [12, 58, 98, 36, 43, 95, 62, 15, 12]
        s: 图片信息的 s; eg: '4671736a'
        distance: 滑动总距离; eg: 100
        passtime: 滑动过程总消耗时长, 毫秒; eg: 1000
        imgload: 验证码图片加载时长, 30-80左右（随便）, 毫秒; eg: 37
        slide_track: 构造的滑动过程数组: eg: [[x, y, time], ...]
    */
    var lang = "zh-cn"
    var distance = slide_track[slide_track.length - 1][0]
    var passtime = slide_track[slide_track.length - 1][2]
    var o = {
        "lang": lang || "zh-cn",
        "userresponse": $(distance, challenge),
        "passtime": passtime,
        "imgload": imgload,
        "aa": pMye(Umdc(slide_track), c, s),
        // "ep": n["quyY"](),
        "ep": {},
    }
    o["rp"] = K(gt + challenge["slice"](0, 32) + o["passtime"])
    var f = te["encrypt"](xe["stringify"](o), skuf())
    var l = d["VIIJ"](f)
    return l
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48
*   49
*   50
*   51
*   52
*   53
*   54
*   55
*   56
*   57
*   58
*   59
*   60
*   61
*   62
*   63


```

### 4. 最后一步咯：w = l + _

```
`function get_w(c, s, gt, challenge, slide_track, imgload) {
    var l = get_l(c, s, gt, challenge, slide_track, imgload);
    var _ = rdgJ()
    return l + _
}
console.log('====================== 测试区 ======================')
var c = [12, 58, 98, 36, 43, 95, 62, 15, 12]  // 图片信息的 c
var s = '4671736a' // 图片信息的 s
var gt = "019924a82c70bb123aae90d483087f94"
var challenge = "ccb5c33fb1498f2fb861468321d4954fgf"
var slide_track = [[0,0,0]]
var imgload = 37  // 验证码图片加载时长,30-80左右（随便）,毫秒
console.log("w 参数: "+ get_w(c, s, gt, challenge, slide_track, imgload))` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14


```

四、结语
----

**极验滑块验证码破解系列文章就到此结束啦，后期更新极验点选验证码的破解方案，包含机器学习验证码识别技术。点赞加关注，免得找不到路。。。**

### _本期文章结束啦，如果对您有帮助，记得收藏加关注哦，后期文章会持续更新 ~~~_

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0d8925da99d43aabd76522e03bfdc87.jpg?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)