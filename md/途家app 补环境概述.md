> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/adfMcqdmrlcE8jJwaKPn0A)

                                    文章写得烂不要紧 封面选的好就行
====================================================

水文一篇～大佬绕过～

**前言**
======

本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！

目标网站
====

> aHR0cHM6Ly93d3cud2FuZG91amlhLmNvbS9hcHBzLzEyMzU5NjM=

抓包
==

这里省略

最终发现 有如下图一堆加密值。这里我们只挑重点。搞下最重要的加密X-TJH即可。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWI1hLvMcZ6iasRvDYPUFibAGFscGlu81LZaaq5Cic2VRSnuaT3GI7GDyTg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

其实这个加密参数。在一年前我搞过m端的，但是一年后。之前APP端好像是可以通用的。但是几天前请求了。貌似是被拒绝了。感兴趣的同学可以看下之前web端的解析。https://mp.weixin.qq.com/s/HLt0CspGtx8toG7hhIKjxg

补充
==

这里感觉写的东西太少了。索性把其他能分析的也都分析一下吧。

挨个来

如下图 已经展示了四个加密了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWovVzecCMxrwdic1wYDPxmf51EhuCH3y8VkOmDMFiaUss7FngDP0nCZ9g/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

分别是

**G-TJS：**

这里代码指的是这个单实际抓包 却不是 但是实际测试好像是都行。所以。这个值应该还能重点研究下

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWl3FYuGqEPRCHibjomyNPk4HSwDu31iakFlWPeQxkTDKaBlcI1Ifar9WA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

**G_TJAP:**

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWFD2QbsAmeMQRhXPyFwbLmJaCJjkUBXskVia18zu38iazExnDICsZpjBw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

然后发现这个带下划线的setApi_Level 最终来自不带下划线的getApiLeveL，而这个不带下划线的getApiLevel 来自不带下划线的setApiLeveL，最终等于 392

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWURSiaMNRlC2zqd4TdSgSV09SQJTGa48QKCicq7vMUlmE0FOKMnFh27jw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

**G_TJU**：

与上个参数一样 层层套娃。

这个UID 其实就是 devices ID 第一次规则生成的。具体逻辑点如下图 自己去研究下

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWThxN3bOibWR6D1Zmvz3KTnJdcECHFCcMNiawRQoffDIsMUCnMdA8m9LA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

**G_TJSE**：

其实就是sessionid 。

具体规则如下图。不细说了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWPTXGHro2D2IhMZvFIepKf9HZRHJxeUic72oibm0u7rmxcj4K6sDXRrSQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

**X-TJP**：

这个参数其实就是基于平台最终判断的值。默认写死也行。

具体如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWGbeXBBY93dM0LCySbvU1sED8Rk44xjrmZiaKvSeqhVM6ic43Ju9mt9Nw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

**X-App-Traceid**：

具体如下。言简意赅

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEW7U6hQPsL8SwQqxtgBqC4ngoWb73wia5QiaiaG5w5gEicujjTgkI78aibQ4g/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

**X-App-Stats**：

最后一个值。其实也很简单。看一眼就出来了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEW3ZiaLDsbfp9UYlPfItqJRNxtvzQojmmiak18NP3C4GSIWmoSjFqON4oQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

反编译
===

这里直接反编译下APK。来看看具体接口吧。

这里直接搜索关键参数名：X-TJH

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWtphZlelRaysZSibNHiaibeH5b5h68ySDlTw1zPBNAibTddKlDibPGBApxDg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

这里发现 三个地方都是调用  Gundam.a 这个方法

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWP1iaibSjupcwUF0zf6CXSxic0UvOspYIkmf6ehRYqmwib9ISTbrD5uUfYQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

点进去看看具体的场景。

然后发现是So层里的加密。 这里就不搞算法了。毕竟我也不会，而且好像也不是常规的算法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWk8ibzt582jiav1WmPKyoF0wxdzpibL52booeOom2Z62mc7Q8m2ZIXf9JA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

这里可以考虑直接补环境吧。

So层 UniDbg
==========

第一步，我们先来frida 调用下这个方法。

发现确实是这个地方。

frida脚本如下。之所以写成JSON只是方便我复制和查看参数。

```
Java.perform(() => {  
let Gundam = Java.use("com.xxx.gundam.Gundam");  
  Gundam["encrypt"].implementation = function (str, str2, str3, str4, i2, j2) {  
    console.log(  
      JSON.stringify({  
        str: str,  
        str2: str2,  
        str3: str3,  
        str4: str4,  
        i2: i2,  
        j2: j2,  
      })  
    );  
  
    let result = this["encrypt"](str, str2, str3, str4, i2, j2);  
    console.log(  
      JSON.stringify({  
        result: result,  
      })  
    );  
    return result;  
  };  
});  
  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWcP8P33asza0GsuP5pVRR1LwAsVapmW3rzib341vs1jvfa9Va2UZ5YZQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

然后就可能去研究这个so了。APP也没啥别的反扒 整体还是练手的好项目。

先报了个 getInstance 缺失的问题。我们就frida去hook一下

```
Java.perform(() => {  
  let TuJiaApplication = Java.use("com.xxx.xxx.xxx");  
  TuJiaApplication["getInstance"].implementation = function () {  
    console.log(`TuJiaApplication.getInstance is called`);  
    let result = this["getInstance"]();  
    console.log(`TuJiaApplication.getInstance result=${result}`);  
    return result;  
  };  
});  
  

```

然后直接补就好了

```
vm.resolveClass("com/xxx/xxx/xxx").newObject("com.xx.xxx.xxx@58d3ff7");  

```

然后继续补

报了个 getPackageName

直接补包名就完事了

然后又报了个 getPackageManager 的错误。

这里固定写死 如下

```
return vm.resolveClass("android/content/pm/PackageManager").newObject(null);  

```

补完之后还有个错

callStaticObjectMethod的 java/security/MessageDigest->getInstance(Ljava/lang/String;)Ljava/security/MessageDigest;的错误

这个方法我们新手不会补 咋办。直接问GPT

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEW2PBgE3FIxZIiaic8bfLLaM6Dxl8qtEyyibDHfkqE1cglhaJ4rgmM2Xia1A/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

```
StringObject algorithm = (StringObject) vaList.getObjectArg(0);  
String algorithmName = algorithm.getValue();  
try {  
    // 生成MessageDigest实例并包装为DvmObject  
    MessageDigest digest = MessageDigest.getInstance(algorithmName);  
    return dvmClass.newObject(digest);  
} catch (NoSuchAlgorithmException e) {  
    // 若算法不存在，可返回默认或抛出异常  
    throw new IllegalStateException("Unsupported algorithm: " + algorithmName);  
}  

```

搞完之后还有最后一个参数 java/security/MessageDigest->digest([B)[B

下图是他给的答案。明显这个DvmArray 没有。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWsZG1mKRvwBG99ofaXfveEIIpeiaD1OnNLcuJj8hMSkxkD9p5wrB2wXQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

所以我们适当修改下。

```
// 获取MessageDigest实例  
MessageDigest digest = (MessageDigest) dvmObject.getValue();  
  
// 获取输入字节数组  
ByteArray inputArray = varArg.getObjectArg(0);  
byte[] input = inputArray.getValue();  
// 执行哈希计算  
byte[] result = digest.digest(input);  
// 返回Byte数组对象  
return new ByteArray(vm, result);  

```

然后运行

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEW54C26fScl3VTrCXCR3iag1DPSfVnQLX8wMteAcMPCJwL31KLiaIh4vUQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

这里就正式出值了。然后我们去frida 一下。找个值比对下。

先拉个 值 目标值 ： 58d00104c05237eff83ac9e39ea705543cf15789

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWse1kQYC4t6C6PVPm9N9LNlTDBS7ahTDQ6vvuR1HwpUxmRkkwALyTrg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

然后我们把上文传参也复制到unidbg中，

然后运行下unidbg

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWO18JZiaMwO6lOVbrGd1IYMZrcm04IYAdstSic2IoU9Rl4PqyZ4JszA9Q/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

58d00104c05237eff83ac9e39ea705543cf15789

58d00104c05237eff83ac9e39ea705543cf15789

嗯 完全一样。完美的入门项目。

兄弟们 加群的可以加我好友拉你进群。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/UNlsdSygkEO3ZVV5fpCe44ZdURym5icEWABiciaID4cklr3ib0MiaL2ibugeiaCVkIaSGeMzLnFBklmibaQyBpic5SXjkcg/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)