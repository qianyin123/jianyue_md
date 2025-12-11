> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/B938R4_KFw1B9nrJ_NAxkw)

点关注，不迷路！

> 本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！若有侵权，请联系作者立即删除！

### 前言

距离上次分析 `x-s` 也有一年了，该篇文章阅读量也有 `11万` 多，感谢小伙伴的一路支持。

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIHDdHJyGcyt4UfosvHfdrQQpV8ia3P5IXtrCy6kib7jRqib5r8yhe8q5qg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "null")某书 x-s、mnsv2、window.mnsv2

最近，它又从 `wasm` 方式加密更改为 `js` 加密了，即： `mnsv2` 从原来的 `mns0101` 升级为 `mns0301`。其实目前网络上也有该算法相关的分析，但是整体看下来不够全，不够细。

所以，出于学习目的，我决定手把手分析记录一下，希望对小小伙伴有所帮助。

### 前置准备

懒人AST解混淆框架：https://blog.csdn.net/studypy1024/article/details/154188036

它不仅能解混淆，还能自动插桩，保存日志文件，方便我们分析。

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIW5oB218M1pWXkMP6VOKh1dFevUicuicKOEDbhr5lIN48xKd5dGibLaApA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1 "null")某书 x-s、mnsv2、window.mnsv2

### 分析过程

全局搜索 `XYS_`，打上断点，然后刷新页面，断点触发，如图所示：

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITInB1lPiaUCicomBDcEDfEjuJjwXHgPeNp2MrKUWxDJpffgnP5Nfic57CNQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2 "null")某书 x-s、mnsv2、window.mnsv2

发现 `x-s` 是通过:

```
`{  
    "x0": "4.2.6",  
    "x1": "xhs-pc-web",  
    "x2": "Windows",  
    "x3": "mns0301_fZaKqslIYHwR62e9nR/vWclLGdIMR85Vm/jxYj8I2j5l7I4dQ/Hj+Dq9k+DQypUTtJ2+mmhzg0njIDU/4GFsPoY6vSF69C0JdzZvA3R92OUEVAuKAqjA0JHKXS0YLGS70+MCnpsRzRJPUaT3lHnKSndVZlciE0JRIk0OHF==",  
    "x4": "object"  
}`
```

经过 `JSON.stringify(f)` 转成字符串后，通过 `(0,p.lz)` 等加密的得到的。如图所示：

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIOgBD9nYoDiaoVViaU1Q1wh8U1p5wpmUspBI9q5oVg9Mzwv66QRNzics3Q/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3 "null")某书 x-s、mnsv2、window.mnsv2

这里直接手动扣下代码就行。没啥难度，就不在这里啰嗦了。

大部分小伙伴应该是在 `x3` 的值上卡住了，其实它就是 `window.mnsv2(c, d)` 得到的。

所以，我们这次着重分析 `window.mnsv2`。首先在控制台输入 `window.mnsv2`，然后双击进入它的代码，如图所示：

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIWl21pKyYLzic6CL1p7Mnv8j7y9ergMrBZJBHD9IQzib2oqnCS5LVMsyA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4 "null")某书 x-s、mnsv2、window.mnsv2

复制全部混淆代码，然后粘贴到 `懒人AST解混淆框架` 的 `source.js` 中，在 `fix.js` 引入以下 `AST通用插件`：

```
`traverse(ast, LiteralRestore);  
traverse(ast, FunctionAliasReplace);  
traverse(ast, FunctionCalcFn({  
globalProgramBodyIndex: [0, 1, 2]  
}));  
traverse(ast, ObjectPropertiesRestore);  
traverse(ast, forIfBlockRestore);  
traverse(ast, CleanUnreferVarsFuns);  
traverse(ast, AddXHSLogger);  
traverse(ast, AddXysLogSaver);`
```

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIAdE0ibKA2oAscHiauv6IKtHg85wJHmZtd7T5jibY2S5wtvhfUo6V7zqBA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5 "null")某书 x-s、mnsv2、window.mnsv2![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIVQ827ZHUTib3cib4aSLDcFcqtBI1hBCVmdA4Ucp4JBiakicEAA11ykxcWw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6 "null")某书 x-s、mnsv2、window.mnsv2

如图，以上插件，我都有详细的使用场景和说明，直接引入就行。我就不一一截图了。

运行代码后，它会自动解混淆，自动插桩。并保存在在 `out.js` 中，如图：

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIaiakibSibOMAhwvvzxX9JMz4a3FWicBjteLicByKJbR8bG1a9X9lKg643Lg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7 "null")某书 x-s、mnsv2、window.mnsv2

替换代码后，在 `window.mnsv2(c, d)` 上打上断点，然后刷新页面，断点触发。

然后，开启日志打印：

```
`xysLogSaver.switch = true; // 打开日志开关`
```

然后放开断点，此时会发现，会输出`1182`条左右日志，如图所示：

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIjc4nhPaUzxCaXib6BQ5aZJF1DaFtCmBD7hQDsVGHK04L7HwZCTiaxGNw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8 "null")某书 x-s、mnsv2、window.mnsv2![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIoIW2iax7QkccEOLUGaicib0WeyoXoZLqXycpb5Jl36g9saJcP4Nj4L0bA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9 "null")某书 x-s、mnsv2、window.mnsv2

可以清楚的看到，`fZaKqMlTfvolxQnGnR/vWt8hedNMR85Vm/jxYj8I2j5l7I4dJiJw1eFEUbPQypUTtJ2+mmhzg0njIDU/4GFsPoY6vSF69C0JdzZvA3R92OUEVAuKAqjA0JHKXS0YLGS70+MCnpsRzRJPUaT3l2nKSndVZlciE0JRIk0OHF==` 是通过上述函数得到的。

传参有两个：

*   • `124` 长度的数组
    
*   • **MfgqrsbcyzPQRStuvC7mn501HIJBo2DE** 经分析，这个是定值
    

那现在只需分析 `124` 长度数组是怎么来的？

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITI3kx4XUJGgJLdjJd1mffSKzXG3gdPXL9r2oECoSwOKaasXSt7OicFIDQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10 "null")某书 x-s、mnsv2、window.mnsv2

这里，`124` 长度数组通过一个函数，得到相同长度的数组，那这个函数是如何实现呢？

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIZnmM5tNEMLO9SXFLo3RiaetaBGQDNIEFZJztWln0Kh8fT64G8TSw9KA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11 "null")某书 x-s、mnsv2、window.mnsv2

这里我列出了三组，说明它是一个循环，通过异或操作，得到相同长度的数组，我暂且叫它 `数组置换`函数。

小伙伴可以先动手试试看，不算难，这里就不一一截图了。

那原始的 `124` 长度数组是怎么来的呢？

总共可以分为 `11` 部分：

`数组置换`函数和获取原始 `124` 长度数组函数，我会贴到星球文章末尾，供大家参考。

#### 1、头部标识固定值

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITI0xq8yS6Cl4bv4F4Oq2icPP7OzicjCTQeszwo60ibvlcIOxu5PqDKEuKQg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12 "null")某书 x-s、mnsv2、window.mnsv2

这个可以固定，直接写死 `[119, 104, 96, 41]`。

#### 2、随机数Math.random字节

![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIhgufYvZCxEeicYFkKRw65GrQia3IzzNvEcCVcicpwmbXG85OiaQTtNoMhw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13 "null")某书 x-s、mnsv2、window.mnsv2![某书 x-s、mnsv2、window.mnsv2](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITILd1e5o33gRLlsF9oX8MQBazhBxmiaJSUcbUbFiackicr3kfzwicToU4qIg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14 "null")某书 x-s、mnsv2、window.mnsv2

这个是通过 `Math.floor(Math.random() * 4294967295)` 然后取字节数组。具体实现：

```
`const v1 = Math.floor(0.5507453723052297 * 4294967295);  
const v2 = v1 & 255;  
const v3 = (v1 >>> 8) & 255;  
const v4 = (v1 >>> 16) & 255;  
const v5 = (v1 >>> 24) & 255;`
```

其中 `v2、v3、v4、v5` 就是。

#### 剩余九种情况数组生成，仅限星球内可见！

以上 `11` 种环境加密就是 `124` 原始数组组成了。

拥有 `懒人AST解混淆框架`，真的可以节省很多时间，大家一定要学会使用。

### 总结

`window.mnsv2`具体实现等js手稿，我会贴到星球文章末尾，供大家学习参考，严禁用于其他用途！

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4Mcu6LjjoEe5971ZphcJ02cUK6V5P9IChhX4D6jvsmAJUexSAYlLOhCs3n5ky4PNQGql8XZcWTQcuRg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=15)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIOeA4xwbziamuNLMzuRfMhxibn2053XSp2StOxgR2OphVQj2Mxv953KFg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=16)