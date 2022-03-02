> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.qinless.com](http://www.qinless.com/150)

### [js/小程序逆向](https://www.qinless.com/series/js-program) 3 / 8

*   [某盛优选 小程序反编译 Api-Sign 加密参数分析](https://www.qinless.com/144)
*   [某多多系列 anti-content 小程序加密逻辑分析](https://www.qinless.com/147)
*   某多多系 websocket 加密逻辑分析
*   [meituan 商家后台 大象 IM SDK JS 分析](https://www.qinless.com/434)
*   [某货优选小程序 sign 参数 js 分析破解](https://www.qinless.com/687)
*   [JavaScript 表情包加密](https://www.qinless.com/1136)
*   [某微信小程序逆向分析 sign hash 参数](https://www.qinless.com/1459)
*   [pdd 缺口滑块验证码逆向分析破解](https://www.qinless.com/1466)

« [上一篇文章](https://www.qinless.com/147 "某多多系列 anti-content 小程序加密逻辑分析")[下一篇文章](https://www.qinless.com/434 "meituan 商家后台 大象 IM SDK JS 分析") »

仅供学习研究 。请勿用于非法用途，本人将不承担任何法律责任。
==============================

前言
==

> 之前写过一篇 [PDD系列 anti-content 小程序加密逻辑分析](https://www.jianshu.com/p/cbd11a47de7d)  
> 这小程序有个坑，每隔几个小时都需要请求个类似于认证的接口，不然请求详情页接口会被风控  
> 经过长时间的尝试测试，最终确定是通过 `websocket` 发送认证信息的，下面开始分析

websocket
=========

小程序反编译出来的代码没有 `websocket` 相关的，无奈就只有去分析调试 `pc` 端的 `js` 了，好在加密逻辑都差不多

![1-chrome-01](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_ff84a63fc9e702e415f1faeaadc3350f.jpg "1-chrome-01")

[点击打开链接](https://ktt.pinduoduo.com/)，这里有 `wss` 连接，收发的消息都是 `arraybuffer` 二进制流数据，主要分析下，二进制的转码逻辑

发送消息编码逻辑分析
==========

![2-chrome-02](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_fbeaeb5458b762b146b5bfbf9133e862.jpg "2-chrome-02")

点到 `Initiator` 窗口跳转到第一个调用栈，通过全局搜索关键词的方式也是可以的

![3-chrome-03](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_c956276abef3a411b6dce125f92bd26a.jpg "3-chrome-03")

点进来之后就看到有 `websocket` 连接的相关代码，下面的 `e.sendSession()` 应该就是发送消息的，先下个端点，刷新页面

![4-chrome-04](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_4c84d517aa693e0eeabc2bd1f351b772.jpg "4-chrome-04")

刷新页面后，就成功断了下来，跟进去，就来到了 `sendSession` 函数，最后调用了 `this.send` 函数，这里有几个参数 `this.appInfo` 不知道是啥，应该是前面初始化好的，后面再去分析。  
`ur, Nn, Bn` 这几个都是固定的，代码抠出来就可以用

![5-chrome-05](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_11a939fe40f1307e9549fd53e24c207b.jpg "5-chrome-05")

来到了 `send` 函数，跳转到 `41225` 行，这里判断了当前环境是否是 `微信小程序`。  
处理方式不同，都是发送了 `a` 数据，`a` 是来自 `a = qn(e)`, `e` 是我们传递进来的 `json` 通过 `qn` 函数转成 `ArrayBuffer` 二进制数据。以上就是 `send` 逻辑，看起来还是比较简单，但是扣代码的时候就比较头疼了，依赖比较多了，自己组合，还是比较消耗时间的。

![6-chrome-06](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_a7cf1d995993c19c82bd5f0017e89d5e.jpg "6-chrome-06")

这里就是 `appInfo` 初始化的逻辑了，代码没有混淆，还是比较容易阅读的

接受消息解码逻辑分析
==========

![7-chrome-07](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_343a7f982b89e358a395a728b03633d2.jpg "7-chrome-07")

经过分析调试，解码逻辑在这里，先在 `41082` 行下断点，刷新页面。  
断下来之后，往下走，遇到函数就跳进去

![8-chrome-08](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_1461dd33f4cc944338f0133a48aa6a17.jpg "8-chrome-08")

最后会来到这个函数，调用 `Mr.decode(t.data.body)` 函数执行解码逻辑，可以看到 `n` 就是最后的结果了

![9-js-01](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_856c55d37dbed23001bf86238b3ea002.jpg "9-js-01")

最后经过长时间调试分析，也是成功把 `JS` 代码扣了出来

![10-python-01](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_39a11b926d891dfce5deda1a1510204a.jpg "10-python-01")

使用 `python` 写个测试脚本，可以正常调用