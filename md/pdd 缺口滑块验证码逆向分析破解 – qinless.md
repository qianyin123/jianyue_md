> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.qinless.com](https://www.qinless.com/1466)

### [js/小程序逆向](https://www.qinless.com/series/js-program) 8 / 8

*   [某盛优选 小程序反编译 Api-Sign 加密参数分析](https://www.qinless.com/144)
*   [某多多系列 anti-content 小程序加密逻辑分析](https://www.qinless.com/147)
*   [某多多系 websocket 加密逻辑分析](https://www.qinless.com/150)
*   [meituan 商家后台 大象 IM SDK JS 分析](https://www.qinless.com/434)
*   [某货优选小程序 sign 参数 js 分析破解](https://www.qinless.com/687)
*   [JavaScript 表情包加密](https://www.qinless.com/1136)
*   [某微信小程序逆向分析 sign hash 参数](https://www.qinless.com/1459)
*   pdd 缺口滑块验证码逆向分析破解

« [上一篇文章](https://www.qinless.com/1459 "某微信小程序逆向分析 sign hash 参数")

前言
==

> **_地址：[https://app.yangkeduo.com/](https://app.yangkeduo.com/)_**  
> **_注意是缺口滑块，而不是点选的_**

![https://www.qinless.com/wp-content/uploads/2022/02/620618c5490c2.png](https://www.qinless.com/wp-content/uploads/2022/02/620618c5490c2.png "pdd 缺口滑块验证码逆向分析破解")

就是上图这种滑块

抓包分析
====

![https://www.qinless.com/wp-content/uploads/2022/02/620618cb03562.png](https://www.qinless.com/wp-content/uploads/2022/02/620618cb03562.png "pdd 缺口滑块验证码逆向分析破解") ![https://www.qinless.com/wp-content/uploads/2022/02/620618cf3744f.png](https://www.qinless.com/wp-content/uploads/2022/02/620618cf3744f.png "pdd 缺口滑块验证码逆向分析破解")

先来分析第一个可疑的请求包，有个 `captcha_collect` 加密参数，响应猜测是图片，估计是经过处理过的一串编码，后面来分析一下

![https://www.qinless.com/wp-content/uploads/2022/02/620618d32bf9b.png](https://www.qinless.com/wp-content/uploads/2022/02/620618d32bf9b.png "pdd 缺口滑块验证码逆向分析破解")

拖动下滑块触发了校验接口，参数多了个 `verify_code` 猜测是缺口距离

获取滑块图片
======

前面有个 `obtain_captcha` 接口返回 `pictures` 数组，全局搜索一下，点进 `js` 文件，搜索 `pictures` 关键词，结果不多，就全部下断点，刷新图片

![https://www.qinless.com/wp-content/uploads/2022/02/620618dddfa13.png](https://www.qinless.com/wp-content/uploads/2022/02/620618dddfa13.png "pdd 缺口滑块验证码逆向分析破解")

成功断下，这里看不出啥，往下继续分析

![https://www.qinless.com/wp-content/uploads/2022/02/620618e2ce213.png](data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== "pdd 缺口滑块验证码逆向分析破解")

主要下面有 `src` 属性，值调用了 `Y.a.decode` 函数，有点可疑，直接下断点

![https://www.qinless.com/wp-content/uploads/2022/02/620618e87a139.png](data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== "pdd 缺口滑块验证码逆向分析破解")

跟进函数，直接跳转到 `return` 处，这是结果已经出来了正是图片的 `base64` 编码，直接把这个函数复制到本地，运行一下

![https://www.qinless.com/wp-content/uploads/2022/02/620618ed739e9.png](data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== "pdd 缺口滑块验证码逆向分析破解")

成功计算出图片的 `base64`

captcha_collect 加密参数分析
======================

这里指分析图片接口的 `captcha_collect` 加密参数，其他接口的流程相同参数不同，老套路全局搜索关键词，不知道在哪里就全部打上断点

![https://www.qinless.com/wp-content/uploads/2022/02/620618f425837.png](data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== "pdd 缺口滑块验证码逆向分析破解")

断下来跟进函数

![https://www.qinless.com/wp-content/uploads/2022/02/620618f8c16ac.png](data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== "pdd 缺口滑块验证码逆向分析破解")

跳到函数尾部，查看 `e` 参数，是一些浏览器指纹信息，接着调用了 `h -> p` 函数

![https://www.qinless.com/wp-content/uploads/2022/02/620618fded581.png](data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== "pdd 缺口滑块验证码逆向分析破解")

`h` 函数，猜测是 `gzip` 数据压缩算法

![https://www.qinless.com/wp-content/uploads/2022/02/62061902a1459.png](data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== "pdd 缺口滑块验证码逆向分析破解")

`p` 函数有 `key iv` 盲猜是 `aes` 算法，后面进行验证

captcha_collect 算法验证
====================

加密：`aes + gzip + base64`  
解密：`base64 + gzip + aes`

![https://www.qinless.com/wp-content/uploads/2022/02/62061eaace726.png](data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== "pdd 缺口滑块验证码逆向分析破解")

按照以上思路，解密成功