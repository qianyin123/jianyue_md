> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4c-VoxkmHgplkU9aNrbrVA)

> 【作者主页】：小鱼神1024
> 
> 【擅长领域】：JS逆向、小程序逆向、AST还原、验证码突防、Python开发、浏览器插件开发、React前端开发、NestJS后端开发等等

前言
--

首先问小伙伴一个问题，你们在逆向工程时，有没有遇到过以下问题？

*   • 1、如何快速定位到加密参数位置？
    
*   • 2、如何判断还原的算法是否正确？
    
*   • 3、如何快速过debugger？
    

...

为了解决以上问题，我们通常的解决方案是 `Hook`。但是，大量的 `Hook` 又难以维护和操作；同时，`Hook` 还可能被反调试工具检测到，导致 `Hook` 失效。

那有没有一款好看且实用的浏览器插件来解决呢？

答案是：有的，它就是 `Reverse DevTools | 逆向调试工具`。我会定期更新该插件，欢迎小伙伴们关注。

谷歌商店扩展地址：https://chromewebstore.google.com/detail/reverse-devtools-%E9%80%86%E5%90%91%E8%B0%83%E8%AF%95%E5%B7%A5%E5%85%B7/ahbegcapccekgadkepgaebigbccaggjl

最新的 `crx` 文件，请移步到知识星球下载。同时有好的建议或问题，欢迎在知识星球中提出。

![Reverse DevTools | 逆向调试工具](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcu6LjjoEe5971ZphcJ02cUKNibibykQfd9VT4p15JARrtq8JZcbeXUdpUgBSe2KqLyJ8qYt3FnkU6Jg/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")Reverse DevTools | 逆向调试工具

使用说明
----

安装插件后，点击插件图标，即可打开插件面板。

![Reverse DevTools | 逆向调试工具](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcu6LjjoEe5971ZphcJ02cUKOiaqu4R4tIStW8p6icw4GlAsDib5xH6miaEqMEZIPMWQHiapsAXknGLGeicA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")Reverse DevTools | 逆向调试工具

打开一键开关按钮，即可开启插件功能。

由于后续插件会不断更新，功能会越来越多，所以如果只想使用部分功能时，可在 `配置` 里开启具体的功能模块。

![Reverse DevTools | 逆向调试工具](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcu6LjjoEe5971ZphcJ02cUKC1NEEfnEgTkVLu1iaMAUIHLYaJKPbVmcXDtNQQIuRXTHfwbrXaTKteg/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")Reverse DevTools | 逆向调试工具

### 变量固定

为了什么要使用 `变量固定` 功能呢？

在逆向过程中，我们经常需要调试一些加密参数，比如：`sign`、`token`等，这些可能是通过标准或者非标准的算法生成的，那如何验证我们的算法是否正确呢？

通常在算法加密是有 `时间戳` 和 `随机数` 参与计算的。如果不做变量固定，即使入参相同，每次加密的结果也会不同，所以我们需要固定 `时间戳` 和 `随机数`，这样就可以验证我们的算法是否正确了。

![Reverse DevTools | 逆向调试工具](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcu6LjjoEe5971ZphcJ02cUKDBlL2r5kVQbX5lCXhjLobrbLSSGaMPFTDIsQHicYnoGyxCF2LLAqlicw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")Reverse DevTools | 逆向调试工具

以 `Date.now` 为例：

![Reverse DevTools | 逆向调试工具](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcu6LjjoEe5971ZphcJ02cUKns6vWEYCDiaJmdiaGN6ckiaNXUoBd4ps7xbTScuAibyATp7o8MMia7d4CDQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")Reverse DevTools | 逆向调试工具

从上述图中可以看出，`Date.now` 的值一直是 `1715937679000`，也就是说，`Date.now` 的值被固定了。同时，`Date.now.toString()` 和 `Function.prototype.toString.call(Date.now)` 的值都为 `function now() { [native code] }`，可以看出，我们也做了 `反Hook检测` 处理。

这样的话，参数参数，随机变量也相同，如果加密结果也是也一样的，就说明我们的算法是正确的。否则，算法就是错误的。

举个案例说明：

用 `零点` 大佬写的 `HOOK对抗` 小游戏，网址：深邃矿洞（第二关）

![Reverse DevTools | 逆向调试工具](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcu6LjjoEe5971ZphcJ02cUKicJEYNVhNYictGHuPUp7FTLOU0L4AA6jJ64hRfKAZsZibZCibtkuibASehA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")Reverse DevTools | 逆向调试工具

这个小游戏的规则很简单，`通过HOOK方式实现挖矿【1次】通关`。通过点击 `开始挖矿` 发现，它的金币是随机生成的，所以我猜测是通过 `Math.random()` 生成的。

Reverse DevTools | 逆向调试工具

通过查看源码，发现源码使用了控制流平坦化，搜索找到了 `Math.random()`。

游戏规则是：`累计金币达到100即可通关`，所以，需要将 `Math.random()` 固定为 `100`。如下：

Reverse DevTools | 逆向调试工具

刷新网址后，分别点击 `开始挖矿`、`通关验证` 各一次。如下：

Reverse DevTools | 逆向调试工具

从上图可以看出，在不打开浏览器控制台的情况下，我们实现了Hook小游戏的通关。

### Hook注入

使用 `Hook注入` 功能前，需要先在配置中开启 `Hook` 功能模块。

Reverse DevTools | 逆向调试工具

#### 移除debugger

在逆向过程中，我们可能会遇到一些网站使用了 `debugger` 来防止调试。比如：`aHR0cHM6Ly9kZXd1LmNvbS8=`

Reverse DevTools | 逆向调试工具  
Reverse DevTools | 逆向调试工具

打开浏览器插件，开启 `移除debugger` 功能，刷新网址后，`debugger` 被移除。简直不要太爽了。

#### JSON.stringify

在逆向过程中，如果遇到 `json` 序列化后的加密参数，可以尝试使用它，快速定位到加密参数位置。比如：`aHR0cHM6Ly9oNS53YWltYWkubWVpdHVhbi5jb20vd2FpbWFpL21pbmRleC9ob21l`

Reverse DevTools | 逆向调试工具  
Reverse DevTools | 逆向调试工具

刷新网页后，发现瞬间 `debugger` 到了加密参数位置。通过堆栈就可以一步一步分析了。

#### XMLHttpRequest.open

它的作用，类似 `xhr` 断点：

Reverse DevTools | 逆向调试工具

一起试试吧。hook关键词设置为：`shopList`

Reverse DevTools | 逆向调试工具

刷新网页后，发现瞬间 `debugger` 到了请求发送位置。然后通过堆栈就可以一步一步分析了。

`Hook注入` 其他参数就不一一举例了。玩法很多，大家自己探索吧。

### 其他功能

新功能，还在持续更新中...，

功能更新后，在知识星球通知大家哦！

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)