> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iHB0NShJ1pClGNz77SJn_g)

点击上方“**Python爬虫与数据挖掘**”，进行关注

回复“**书籍**”即可获赠Python从入门到进阶共10本电子书

今

日

鸡

汤

春风十里扬州路，卷上珠帘总不如。

大家好，我是码农星期八！本教程只用于学习探讨，不允许任何人使用技术进行违法操作，阅读教程即表示同意！  

前言
--

上文：上一篇文章。

上文浅浅的使用了一下frida和简单的hook了一下xx牛的执行流程。

本次就接着上次继续搞，看看如何实现自动登录。

流程分析
----

### login

1.  login->requestNetwork->实例化JsonRequest->调用JsonRequest对象的addRequestMap
    
    如果只看第一层的话，大概是这个流程。
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

最后调用的是addRequestMap，点进去看看addRequestMap都干了什么。

### addRequestMap

addRequestMap就比较简单了，不过是RequestUtil.paraMap的返回值当作了 RequestUtil.encodeDesMap的参数。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

甚至你还可以发现，竟然出现了`Encrypt`，不就是请求里面的关键字吗，wc。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

### RequestUtil.paraMap

查看RequestUtil.paraMap的详细内容，可以发现调用了md5，并且最后返回值都字母全部大写。

![图片](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq54I6ApYuanyw4Gs8UuOYXr7yXHhfqzVc1T8zZpYU4L4XG4QdVcjic32Xffj15pHhmugnvicn1bBXLiaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### md5

这就是一个普通的md5。

![图片](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq54I6ApYuanyw4Gs8UuOYXr7JVnUlxotZpKC24Siblc1pHpcd06OOdqTqGV9AgdUExmh5JSiaGp4mUKQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### RequestUtil.encodeDesMap

分析完了RequestUtil.paraMap继续来看RequestUtil.encodeDesMap，它需要三个参数。

![图片](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq54I6ApYuanyw4Gs8UuOYXr752NEQeuAK2lz1N2Zh2eia9iazLw1avbOtEFftBvALzozrO1kyCUNGXHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

这个`this.desKey`和`this.desIV`是啥呢？

经过寻找

```
this.desKey = 65102933  
this.desIV = 32028092  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq54I6ApYuanyw4Gs8UuOYXr745cpgcWY2CRhFicHvywZ4dXnuXn0MlZcWbxt8j5jkJQouialDjbhEdlw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

继续看逻辑，看一下encodeDesMap的详情，里面又实例化了`DesSecurity`类，并且调用了`encrypt64`方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq54I6ApYuanyw4Gs8UuOYXr7ADEibibViaLw0XZWlFoIq7tLQBBapo7ywc0GIcKmYbrG6wEyicECJicbKHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### DesSecurity

![图片](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq54I6ApYuanyw4Gs8UuOYXr72dVdQc4M4kjlcgwBhy37cKpPqQoOMCm4vNXg2jP2a98Tv6iaWNByXew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

这里是一个DES/CBC算法。

他的逻辑大概是把传入的desKey通过md5哈希一下，然后当作DES的key。

传进来的desIV当作iv向量，最后加密的结果进行base64处理一下。

frida验证流程
---------

经过静态分析，理论上大概是上面的流程，但是。,到底对不对呢？来hook一下不就知道了！！！

### 代码

```
Java.perform(function () {  
    let jsonRequest = Java.use("com.dodonew.online.http.JsonRequest");  
    jsonRequest.addRequestMap.overload('java.util.Map', 'int').implementation = function (addMap, a) {  
        console.log("==== jsonRequest.addRequestMap 被调用了");  
        console.log("addMap:", addMap);  
        console.log("a:", a);  
        return this.addRequestMap(addMap, a);  
    }  
  
    let requestUtil = Java.use("com.dodonew.online.http.RequestUtil");  
    requestUtil.paraMap.overload('java.util.Map', 'java.lang.String', 'java.lang.String').implementation = function (addMap, append, sign) {  
        console.log("==== requestUtil.paraMap 被调用了");  
        console.log("addMap:", addMap);  
        console.log("append:", append);  
        console.log("sign:", sign);  
        return this.paraMap(addMap, append, sign);  
    }  
  
    let utils = Java.use("com.dodonew.online.util.Utils");  
    utils.md5.implementation = function (string) {  
        console.log("==== utils.md5 被调用了");  
        console.log("string:", string);  
        return this.md5(string);  
    }  
  
    requestUtil.encodeDesMap.overload('java.lang.String', 'java.lang.String', 'java.lang.String').implementation = function (data, desKey, desIV) {  
        console.log("==== requestUtil.encodeDesMap 被调用了");  
        console.log("data:", data);  
        console.log("desKey:", desKey);  
        console.log("desIV:", desIV);  
        return this.encodeDesMap(data, desKey, desIV);  
    }  
  
})  

```

### 效果

![图片](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq54I6ApYuanyw4Gs8UuOYXr78YXYdL1gHvMDtvVNpViaA050agrgHAdiaLf7ZxzTLW2k7xL40hz6CoeA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

综上所述，我们分析的是没问题的，当然，如果你发现没有hook到，那就废了，再次尝试即可。

关于算法复现
------

本次通过js来完成算法复现，因为js写完之后，基本能被任何语言调用，更通用，本次使用的是CryptoJS库。

### 获取sign

其实经过研究发现，并不是所有参数都是随机的。

只有那一两个参数是固定的，所以只需要把那几个参数构造好就可以了。

通过观察，在第一次的时候，数据都汇聚到了md5这，所以先hook md5，看一下传入的参数和返回值。

![图片](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq54I6ApYuanyw4Gs8UuOYXr7deIBv3KRSmicWdCgZyHQRicZs0OaHMU4cyGA4q6zzBbRmoJvWSibJV7Sw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**hook md5代码**

```
Java.perform(function () {  
 let utils = Java.use("com.dodonew.online.util.Utils");  
    utils.md5.implementation = function (string) {  
        console.log("==== utils.md5 被调用了");  
        console.log("string:", string);  
        let ret = this.md5(string);  
        console.log("return:",ret);  
        return ret;  
    }  
})  

```

**传入的参数和返回值**

![图片](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq54I6ApYuanyw4Gs8UuOYXr77f0w4zq9eWkoNBwK6oekibLIhAYXozIk0H2tDEYqSYc1lUWslpmfwWw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**js代码**

```
function getSign(user, pass, time) {  
    var signStr = "equtype=ANDROID&loginImei=Androidnull&timeStamp=" + time + "&userPwd=" + pass + "&user;  
    return CryptoJS.MD5(signStr).toString().toUpperCase();  
}  

```

### 获取encrypt

第一层的`RequestUtil.paraMap`执行完了，当作第二层`RequestUtil.encodeDesMap`的第一个参数

所以现在hook一下`RequestUtil.encodeDesMap`看一下参数和返回值

**hook代码**

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**js代码**

ok，知道了动态变量，那就写一下代码吧。

```
function getSign(user, pass, time) {  
    var signStr = "equtype=ANDROID&loginImei=Androidnull&timeStamp=" + time + "&userPwd=" + pass + "&user;  
    return CryptoJS.MD5(signStr).toString().toUpperCase();  
}  
  
function encrypt(user, pass) {  
    var time = '1624379089014';  
    var sign = getSign(user, pass, time);  
    var plainText = '{"equtype":"ANDROID","loginImei":"Androidnull","sign":"' + sign  
        + '","timeStamp":"' + time + '","userPwd":"' + pass + '","username":"' + user + '"}';  
    var key = CryptoJS.enc.Hex.parse(CryptoJS.MD5("65102933").toString());  
    var iv = CryptoJS.enc.Utf8.parse("32028092");  
    var result = CryptoJS.DES.encrypt(plainText, key, {  
        iv: iv,  
        mode: CryptoJS.mode.CBC,  
        padding: CryptoJS.pad.Pkcs7  
    }).toString();  
    return result;  
}  

```

### 完整代码

`xxn.js`

### 运行结果

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

总结
--

本次带着走了一下静态分析和hook静态分析的代码，很庆幸，是正确的。

当然，也可能不正确，所以就要多hook，多尝试。

算法复现这一块，尽量使用js，因为相对而言更通用。

人低为王，水低为海，加油！

我是码农星期八，如果在操作过程中有任何问题，记得下面留言，我们看到会第一时间解决问题。

越努力，越幸运。我是码农星期八，如果觉得还不错，记得动手点赞一下哈。感谢你的观看。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

小伙伴们，快快用实践一下吧！如果在学习过程中，有遇到任何问题，欢迎加我好友，我拉你进Python学习交流群共同探讨学习。

![图片](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq57r8c4APW1Z3gadxvXA9h4oTTklHXdjKyQmwZxLk8EwrGPJQego33ISYib6Yt9jPbhnX1ehMMsWGHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**********---**--************---****---**********---**--********---******************************** End **********---**--************---**--**---**********---**--********-********************************  

往期精彩文章推荐：

*   [requests库请求获取不到数据怎么办？不妨试试看这种妙法](http://mp.weixin.qq.com/s?__biz=MzU3MzQxMjE2NA==&mid=2247500478&idx=1&sn=85d7d7355764d39e202f0a840000a1ea&chksm=fcc08495cbb70d83b8abce4a4bfb083137c7c91c0d57c427d73dc01cc2f0d2974534b350c9a1&scene=21#wechat_redirect)
    
*   [Python网络爬虫之js逆向之远程调用(rpc)免去抠代码补环境简介](http://mp.weixin.qq.com/s?__biz=MzU3MzQxMjE2NA==&mid=2247500475&idx=1&sn=a3eb8c44019f553544acc92354e6eb23&chksm=fcc08490cbb70d868cacc2b2b11b60b349897e7d39797d9b2131494b192e43f7dfd7f23336bf&scene=21#wechat_redirect)
    
*   [盘点一款自研的Python虚拟环境管理器——带GUI界面的那种](http://mp.weixin.qq.com/s?__biz=MzU3MzQxMjE2NA==&mid=2247500471&idx=1&sn=b14111e5a6b77c2341f6f677940aed48&chksm=fcc0849ccbb70d8a9adc1b665e7fa6aa9c4504b9a0ad4303195b3f8e3eacc51a5412c77714aa&scene=21#wechat_redirect)
    
*   [盘点一道Python网络爬虫中使用正则表达式匹配字符的题目](http://mp.weixin.qq.com/s?__biz=MzU3MzQxMjE2NA==&mid=2247500469&idx=1&sn=af139f034262aee27514adff8124983a&chksm=fcc0849ecbb70d881c61aaeba44c02c0de8d84e831ee5974300b7cd7d5809a656db012eeb2a2&scene=21#wechat_redirect)  
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/peDq5Y9hmZaHvs19XSuQEGbkast75g4uziam64mHRseaJibQEIZGUgwWthoqHAiakAXcicszKuT0OgAbXM2k2hiaSyA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

欢迎大家**点赞，****留言，****转发，**转载，****感谢大家的相伴与支持

想加入Python学习群请在后台回复【**入群**】

万水千山总是情，点个【**在看**】行不行

**/今日留言主题/**

随便说一两句吧~~