> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.qinless.com](https://www.qinless.com/147)

### [js/小程序逆向](https://www.qinless.com/series/js-program) 2 / 8

*   [某盛优选 小程序反编译 Api-Sign 加密参数分析](https://www.qinless.com/144)
*   某多多系列 anti-content 小程序加密逻辑分析
*   [某多多系 websocket 加密逻辑分析](https://www.qinless.com/150)
*   [meituan 商家后台 大象 IM SDK JS 分析](https://www.qinless.com/434)
*   [某货优选小程序 sign 参数 js 分析破解](https://www.qinless.com/687)
*   [JavaScript 表情包加密](https://www.qinless.com/1136)
*   [某微信小程序逆向分析 sign hash 参数](https://www.qinless.com/1459)
*   [pdd 缺口滑块验证码逆向分析破解](https://www.qinless.com/1466)

« [上一篇文章](https://www.qinless.com/144 "某盛优选 小程序反编译 Api-Sign 加密参数分析")[下一篇文章](https://www.qinless.com/150 "某多多系 websocket 加密逻辑分析") »

仅供学习研究 。请勿用于非法用途，本人将不承担任何法律责任。
==============================

前言
==

> 本次目标小程序，快团团。  
> 前面就不去分析请求包了，直接去反编译小程序，分析JS

反编译 wxapkg
==========

找一台 root 手机  
路径在 `/data/data/com.tencent.mm/MicroMsg/hash值/appbrand`  
打开之前建议先把 路径下所有 hash值 文件都给删了，剩的其他的小程序文件混淆视听  
找到之后使用 adb 命令 pull 到电脑上，开始反编译，具体反编译代码可自行查找，也可联系本人分享给你

分析 JS 加密逻辑
==========

反编译成功之后直接使用 微信开发者工具打开，第一次打开可能会报一些，缺失插件，语法错误之类的，自己修修补补就行了，可以删的就直接删了

![1-wxtools-01](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_c7e1f69367bc54918a5ba8df3a786817.jpg "1-wxtools-01")

改好之后成功运行，就是上图这种

![2-wxtools-02](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_872c4484c56641fc97c51e61bf001539.jpg "2-wxtools-02")

接着在全局搜索 `anti_content` 关键字，定位到这里，经过调试发现调用 `s.messageDepacketize()` 函数，返回的就是加密结果

![3-wxtools-03](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_84803adce725dde3353023600fee19c6.jpg "3-wxtools-03")

在全局搜索 `antiSdk` 定位到这里，调用 `default.turtlemirror` 函数返回，网上可以看到 `require("./libs/lightMiniApp")` 引用了这个文件，现在就可以大胆猜测加密逻辑应该就是在这个里面，直接跟进去看看

JS 环境补充
=======

经过上面的分析已经确定了 加密逻辑就在 `lightMiniApp.js` 这个文件里，我么直接全部复制放到本地去调试  
本地调试js就是 补环境，缺啥补啥，这里贴上一段代码，可以使用 `Proxy` hook 出代码里使用了哪些环境

```
`window = new Proxy(window, {
    get: function (target, key, receiver) {
        console.log('window.get.key: ', key);

        if (target[key] instanceof Object) {
            return new Proxy(target[key], {
                get: function (a, b, c) {
                    console.log('window.get.instance.key: ', key, b);
                    return a[b];
                },
                set: function (a, b, c, d) {
                    a[b] = c;
                }
            })
        }

        return target[key];
    },
    set: function (target, key, value, receiver) {
        console.log('window.set.key: ', key);
        console.log('window.set.value: ', value);
        console.log(target, key, value, receiver);
        target[key] = value;
    }
});`Copy
```

![4-pycharm-01](https://www.qinless.com/wp-content/uploads/2021/09/wp_editor_md_34bf698d7f811451c4e4b94f262e0658.jpg "4-pycharm-01")

最后环境补充完后，也是成功运行出了结果