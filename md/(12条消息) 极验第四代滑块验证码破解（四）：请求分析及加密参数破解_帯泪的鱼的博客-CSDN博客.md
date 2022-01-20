> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_42857999/article/details/122364731)

### 极验第四代滑块验证码破解（四）：请求分析及加密参数破解

*   *   [声明](#_3)
    *   [一、极验请求分析](#_18)
    *   *   [1. 滑块测试网站入口](#1__19)
        *   [2. 滑块验证过程抓包](#2__24)
        *   [3. 请求详解](#3__28)
        *   *   [3.1. adaptive-captcha-demo](#31_adaptivecaptchademo_29)
            *   [3.2. adaptive-captcha-demo.js](#32_adaptivecaptchademojs_44)
            *   [3.3. load](#33_load_60)
            *   [3.4. verify](#34_verify_108)
    *   [二、js破解前准备工作](#js_144)
    *   *   [1. gcaptcha4.js反混淆](#1_gcaptcha4js_145)
        *   [2. 找到w参数](#2_w_148)
        *   [3. 使用chrome插件reres替换js文件](#3_chromereresjs_153)
    *   [三、load请求中challenge参数破解](#loadchallenge_162)
    *   *   [1. 通过python的uuid库代码直接生成](#1_pythonuuid_164)
        *   [2. 通过构造challenge的函数生成](#2_challenge_169)
    *   [四、w参数破解](#w_204)
    *   *   [1. w参数分析](#1_w_205)
        *   [2. e 对象参数详情](#2_e__214)
        *   [3. 序列化函数：d["default"]["stringify"]](#3_ddefaultstringify_232)
        *   [4. 加密函数：h["default"]](#4_hdefault_237)
        *   [5. e 对象参数破解](#5_e__244)
        *   *   [5.1. 参数破解：track](#51_track_249)
            *   [5.2. 参数破解：passtime](#52_passtime_252)
            *   [5.3. 参数破解：setLeft](#53_setLeft_264)
            *   [5.4. 参数破解：userresponse](#54_userresponse_276)
    *   [五、w参数破解完整代码](#w_288)
    *   [六、结语](#_435)
    *   *   [*本期文章结束啦，如果对您有帮助，记得收藏加关注哦，后期文章会持续更新 ~~~*](#__437)

声明
--

**原创文章，请勿转载！**

**本文内容仅限于安全研究，不公开具体源码。维护网络安全，人人有责。**

**本文关联文章超链接：**

1.  [极验第四代滑块验证码破解（一）：AST还原混淆JS](https://blog.csdn.net/qq_42857999/article/details/122364575)
2.  [极验第四代滑块验证码破解（二）：滑块缺口识别](https://blog.csdn.net/qq_42857999/article/details/122364690)
3.  [极验第四代滑块验证码破解（三）：滑块轨迹构造](https://blog.csdn.net/qq_42857999/article/details/122364712)
4.  [极验第四代滑块验证码破解（四）：请求分析及加密参数破解](https://blog.csdn.net/qq_42857999/article/details/122364731)

**本篇文章篇幅较长，但是讲的特别细，小伙伴们耐心看完哦 ~**  
  

一、极验请求分析
--------

### 1. 滑块测试网站入口

```
`https://www.geetest.com/adaptive-captcha-demo` 

*   1


```

### 2. 滑块验证过程抓包

与极验三代滑块验证码相比，极验四代简化了验证过程，加密参数w的生成也变简单了。和极验三代验证码一样的分析流程，撸就完了。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/d259afe7c9ca499bad2a69cbd17ee144.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

### 3. 请求详解

#### 3.1. adaptive-captcha-demo

**请求介绍**  
极验第四代验证码测试主页，主要是获取下个请求中的url(这个url是动态变化的，所以这个步骤必须要)

**请求参数**

```
`没啥参数` 

*   1


```

**请求响应**

```
`响应为html文档，通过正则匹配下个步骤的请求url
href="(.\*?adaptive-captcha-demo\.js)"` 

*   1
*   2


```

#### 3.2. adaptive-captcha-demo.js

**请求介绍**  
获取w参数加密需要的参数captchaId

**请求参数**  
实际破解滑块过程中，此请求可忽略。

```
`没啥参数` 

*   1


```

**请求响应**

```
`响应为js文件，通过正则匹配captchaId参数的值
captchaId:"([0-9a-z]+)"` 

*   1
*   2


```

#### 3.3. load

**请求介绍**  
获取验证码信息，包括：验证码类型、验证码背景图、验证码滑块图、lot_number参数、静态文件url等

**请求参数**

```
`captcha_id: 24f56dc13c40dc4a02fd0318567caef5    // 上个请求中获取
challenge: f8beca82-84a4-4b32-a01d-dae1697f1236 // 由js代码生成，下面会详细讲解生成过程
client_type: web                                // 固定值
risk_type: slide                                // 验证码类型
lang: zh                                        // 固定值
callback: geetest_1641878914316                 // 当前时间戳` 

*   1
*   2
*   3
*   4
*   5
*   6


```

**请求响应**  
响应中的c、s在后续无感验证生成w参数时需要使用，其中c为定值，s为变化值。

```
`{
    "status": "success",
    "data": {
        "lot_number": "c574cd8c30a541b28597fb4582542c61",
        "captcha_type": "slide",
        "js": "/js/gcaptcha4.js",
        "css": "/css/gcaptcha4.css",
        "static_path": "/v4/static/v1.4.4",
        "slice": "pictures/v4_pic/slide_2021_07_14/Group81/slide/019d7acaf9aa4f488a332b6baff7176b.png",
        "bg": "pictures/v4_pic/slide_2021_07_14/Group81/bg/019d7acaf9aa4f488a332b6baff7176b.png",
        "ypos": 116,
        "gct_path": "/v4/gct/gct4.5258a91d0f5f0bb73c65d4d18d48d93f.js",
        "arrow": "arrow_1",
        "show_voice": false,
        "feedback": "https://www.geetest.com/Helper",
        "logo": true,
        "pt": "1",
        "captcha_mode": "risk_manage",
        "language": "zh",
        "custom_theme": {
            "_style": "stereoscopic",
            "_color": "hsla(224,98%,66%,1)",
            "_gradient": "linear-gradient(180deg, hsla(224,98%,71%,1) 0%, hsla(224,98%,66%,1) 100%)",
            "_hover": "linear-gradient(180deg, hsla(224,98%,66%,1) 0%, hsla(224,98%,71%,1) 100%)",
            "_brightness": "system",
            "_radius": "4px"
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


```

#### 3.4. verify

**请求介绍**  
该请求是极验验证请求，gcaptcha4.js收集滑动轨迹，与上个请求中的lot_number参数，加密生成w参数。

**请求参数**

```
`captcha_id: 24f56dc13c40dc4a02fd0318567caef5      // 与上个请求中的captcha_id参数相同
challenge: e29f82f7-78db-42de-913f-fb1b01d3e30b   // 与上个请求中的challenge参数相同
client_type: web                                  // 固定值
lot_number: c574cd8c30a541b28597fb4582542c61      // 上个请求的响应中lot_number参数值
risk_type: slide                                  // 验证码类型
pt: 1                                             // 上个请求的响应中pt参数值
w: c742e66584e3b20ad523c2ddff...                  // gcaptcha4.js收集滑动轨迹加密生成
callback: geetest_1641878916958                   // 当前时间戳` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8


```

**请求响应**  
验证成功，拿到seccode。

```
`{
    "status": "success",
    "data": {
        "lot_number": "5b79a07bfb1640c1955ef28fbe28bef0",
        "result": "success",
        "fail_count": 0,
        "seccode": {
            "lot_number": "5b79a07bfb1640c1955ef28fbe28bef0",
            "pass_token": "f5b3b3d7664e032bc06730d56f83433046af98878bfa796d5e4b5b5f48904e40",
            "gen_time": "1641880037",
            "captcha_output": "2W2T6RrNJ8qVlCuIQxrHVp0imaZt_LrywRPCvYEbTHwQyoZwHIYvpYM5zF0-qSl8LQF_m8ggUDGiA0b8IDdrjji1YjjbEERRWAP9SxWj-G090QRaou4m8NnZL0NVmBie"
        },
        "score": "1"
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


```

二、js破解前准备工作
-----------

### 1. gcaptcha4.js反混淆

通过AST语法树将混淆的gcaptcha4.js文件还原，具体还原方式请看 [极验第四代滑块验证码破解（一）：AST还原混淆JS](https://blog.csdn.net/qq_42857999/article/details/122364575)

### 2. 找到w参数

在反混淆后的gcaptcha4.js文件中搜索 “w”，即可找到今天破解的主角，并加入一行debugger。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/f8a89f2b76ef4f34bcf1fb805bbd13fe.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

### 3. 使用chrome插件reres替换js文件

配置chrome插件reres，将网页加载的js文件替换成对应的反混淆后的js文件。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/9a63d94841c84986b603308a7be5d7c8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

打开开发者工具，刷新网页，触发滑块操作。然后，就可以任意打断点分析啦。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/542d47c987bc43b9ade89620595beea8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

三、load请求中challenge参数破解
----------------------

经过测试，下面有两种方式生成challenge

### 1. 通过python的uuid库代码直接生成

```
`import uuid
challenge = uuid.uuid1().__str__()` 

*   1
*   2


```

### 2. 通过构造challenge的函数生成

第一步：找到构造challenge的函数入口  
![在这里插入图片描述](https://img-blog.csdnimg.cn/50ca73652a3d4d83aa7120356ded653e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

第二步：刷新网页，进入断点分析，找到构造challenge的函数入口  
![在这里插入图片描述](https://img-blog.csdnimg.cn/81744ff6e03f4fdcb749686dda1f1952.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

第三步：进入uuid函数，这个函数特别简单，直接改成py

```
`import random
def uuid():
    """
    var uuid = function () {
        return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
            var r = Math.random() * 16 | 0;
            var v = c === 'x' ? r : (r & 0x3 | 0x8);
            return v.toString(16);
        });
    };
    """

    def __random(c):
        r = int(random.random() * 16)
        v = r if c == 'x' else (r & 0x3 | 0x8)
        return hex(v)[2:]

    string = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'
    ret = ''
    for i in string:
        if i in 'xy':
            i = __random(i)
        ret += i
    return ret` 

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

四、w参数破解
-------

### 1. w参数分析

滑动滑块，触发debugger，从w生成位置开始往上一步一步分析，看看生成w参数需要哪些关键函数。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/14a2c374c55a42c99c800b5a7c4a301d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

下面分析对w参数的分析过程，依照下图所示。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/519dea6626b44e00b0ef7724fe1dd426.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

### 2. e 对象参数详情

通多上面对w参数的分析，可以得出：w参数=加密函数（序列化函数（e对象）  
下面是e对象中各参数的详解

```
`{
    "setLeft": 99,                         // 滑块滑动距离
    "track": [[38, 15, 0], ... ],          // 滑块滑动轨迹
    "passtime": 146,                       // 滑块滑动过程时长
    "userresponse": 98.41476022585691,     // 通过滑块滑动距离计算得到
    "lot_number": "4b4ef3e583444e0fb...",  // load响应中的lot_number参数
    "device_id": "A59C",                   // 可为定值，可省略此参数
    "geetest": "captcha",                  // 可为定值，可省略此参数
    "lang": "zh",                          // 可为定值，可省略此参数
    "ep": "123",                           // 可为定值，可省略此参数
    "nz8c": "255401529",                   // 可为定值，可省略此参数
    "em": {"ph": 0, ...}                   // 可为定值，可省略此参数
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


```

### 3. 序列化函数：d[“default”][“stringify”]

断点进入d[“default”][“stringify”]函数，发现此函数中有外部函数调用，所以将外部函数整个抠出来  
![在这里插入图片描述](https://img-blog.csdnimg.cn/4d827bd2bd4c4f94a6b11cc440ec9d6d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

### 4. 加密函数：h[“default”]

断点进入h[“default”]函数，这里需要注意：此函数中对第二个入参中的值进行了很多判断，实际没有进行运算，这里我们修改下函数，直接绕过判断  
![在这里插入图片描述](https://img-blog.csdnimg.cn/32582137cca847a2821ab62168039885.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/efc39b09b4c34b30803d75520eb0e5e4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

### 5. e 对象参数破解

e对象作为入参，因此需要通过调用栈，跳转到上层函数调用，分析e对象中每个参数的构造函数  
![在这里插入图片描述](https://img-blog.csdnimg.cn/8ea6d3a70a8d4d35a450bb5604e4b3f4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 5.1. 参数破解：track

小伙伴们看这个哦： [极验第四代滑块验证码破解（三）：滑块轨迹构造](https://blog.csdn.net/qq_42857999/article/details/122364712)

#### 5.2. 参数破解：passtime

滑动消耗时长就直接从滑动轨迹中累加了

```
`function get_passtime(track) {
    var passtime = 0;
    for (i = 0; i < track.length; i++) {
        passtime += track[i][2]
    }
    return passtime
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7


```

#### 5.3. 参数破解：setLeft

滑动距离也从滑动轨迹中累加吧

```
`function get_setLeft(track) {
    var setLeft = 0;
    for (i = 1; i < track.length; i++) {
        setLeft += track[i][0]
    }
    return setLeft
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7


```

#### 5.4. 参数破解：userresponse

从上文知道 userresponse = setLeft / t["KaTeX parse error: Can't use function '\]' in math mode at position 7: _BGBb"\̲]̲，那就分析一下t\["_BGBb"]函数  
![在这里插入图片描述](https://img-blog.csdnimg.cn/da0c29061d82462e960343bd023f090a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

```
`function get_userresponse(setLeft, captcha_width) {
    var e = 340
    var i = .8876 * e / captcha_width
    return setLeft / i
}` 

*   1
*   2
*   3
*   4
*   5


```

五、w参数破解完整代码
-----------

感谢小伙伴们看了这么久，直接上干货吧。w参数破解的全量代码奉上，其中只有3个函数需要大家抠出来（原封不动的抠出来就行，太长了没法粘贴进去）

```
`var window = {
    "navigator": {
        "appName": "Netscape"
    },
    "crypto": {
        "getRandomValues": getRandomValues
    }
}
var navigator = window["navigator"]

function randoms(min, max) {
    return Math.floor(Math.random() * (max - min + 1) + min)
}

function getRandomValues(buf) {
    var min = 0,
        max = 255;
    if (buf.length > 65536) {
        var e = new Error();
        e.code = 22;
        e.message = 'Failed to execute \'getRandomValues\' : The ' + 'ArrayBufferView\'s byte length (' + buf.length + ') exceeds the ' + 'number of bytes of entropy available via this API (65536).';
        e.name = 'QuotaExceededError';
        throw e;
    }
    if (buf instanceof Uint16Array) {
        max = 65535;
    } else if (buf instanceof Uint32Array) {
        max = 4294967295;
    }
    for (var element in buf) {
        buf[element] = randoms(min, max);
    }
    return buf;
}


function get_passtime(track) {
    var passtime = 0;
    for (i = 0; i < track.length; i++) {
        passtime += track[i][2]
    }
    return passtime
}

function get_userresponse(setLeft, captcha_width) {
    var e = 340
    var i = .8876 * e / captcha_width
    return setLeft / i
}

function get_setLeft(track) {
    var setLeft = 0;
    for (i = 1; i < track.length; i++) {
        setLeft += track[i][0]
    }
    return setLeft
}

function get_w(track, captcha_width, lot_number) {
    var d = function () {
        // 内容需要自己抠出来
    }();
    d["default"] = d;

    function get_h(e, t) {
        var _ = function () {
            // 内容需要自己抠出来
        }();
        _["default"] = _;
        var guid = function () {
            function e() {
                return (65536 * (1 + Math["random"]()) | 0)["toString"](16)["substring"](1);
            }

            return function () {
                return e() + e() + e() + e();
            }
                ;
        }();

        var i = function () {
            // 内容需要自己抠出来
        }();
        i["default"] = i;

        function arrayToHex(e) {
            for (var t = [], s = 0, a = 0; a < 2 * e["length"]; a += 2)
                t[a >>> 3] |= parseInt(e[s], 10) << 24 - a % 8 * 4,
                    s++;

            for (var o = [], n = 0; n < e["length"]; n++) {
                var r = t[n >>> 2] >>> 24 - n % 4 * 8 & 255;
                o["push"]((r >>> 4)["toString"](16)),
                    o["push"]((15 & r)["toString"](16));
            }

            return o["join"]("");
        }

        var a = guid(),
            o = new _["default"]()["encrypt"](a);


        var n = i["default"]["encrypt"](e, a);
        return arrayToHex(n) + o;
    }

    var passtime = get_passtime(track),
        setLeft = get_setLeft(track),
        userresponse = get_userresponse(setLeft, captcha_width);
    var e = {
        "setLeft": setLeft,
        "track": track,
        "passtime": passtime,
        "userresponse": userresponse,
        "device_id": "A59C",
        "lot_number": lot_number,
        "geetest": "captcha",
        "lang": "zh",
        "ep": "123",
        "nz8c": "255401529",
        "em": {
            "ph": 0,
            "cp": 0,
            "ek": "11",
            "wd": 1,
            "nt": 0,
            "si": 0,
            "sc": 0
        }
    }
    return get_h(d["default"]["stringify"](e))
}

console.log('====================== 测试区 ======================')

var track = [[47, 17, 0],],
    captcha_width = 300,
    lot_number = "13e35cf1c2834cb9afd01b89006c8f41";

var w = get_w(track, captcha_width, lot_number)
console.log(w)` 

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
*   64
*   65
*   66
*   67
*   68
*   69
*   70
*   71
*   72
*   73
*   74
*   75
*   76
*   77
*   78
*   79
*   80
*   81
*   82
*   83
*   84
*   85
*   86
*   87
*   88
*   89
*   90
*   91
*   92
*   93
*   94
*   95
*   96
*   97
*   98
*   99
*   100
*   101
*   102
*   103
*   104
*   105
*   106
*   107
*   108
*   109
*   110
*   111
*   112
*   113
*   114
*   115
*   116
*   117
*   118
*   119
*   120
*   121
*   122
*   123
*   124
*   125
*   126
*   127
*   128
*   129
*   130
*   131
*   132
*   133
*   134
*   135
*   136
*   137
*   138
*   139
*   140
*   141
*   142


```

六、结语
----

**极验第四代滑块验证码破解系列文章就到此结束啦，后期更新更多爬虫知识与破解技术。点赞加关注，免得找不到路。。。**

### _本期文章结束啦，如果对您有帮助，记得收藏加关注哦，后期文章会持续更新 ~~~_

![在这里插入图片描述](https://img-blog.csdnimg.cn/4cc1521b816c495fbb369ea20a4d32a1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)