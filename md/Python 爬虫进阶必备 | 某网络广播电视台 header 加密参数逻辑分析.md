> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ufVgRQsEAiNXEJbB3Icm-A)

点击上方“咸鱼学Python”，选择“加为星标”

第一时间关注Python技术干货！

![图片](https://mmbiz.qpic.cn/mmbiz_png/jqCHqBaKLl1EjpvMJawicrfVtdEebUA2ibMAARaOuRUma2RtqlDRJiakAnZ9KicBAsUuyw3iafJWgGJWBJOzqtibjd2Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图源：极简壁纸

今日网站
----

aHR0cHM6Ly93d3cuZ2R0di5jbi9zZWFyY2g/a2V5PSVFNSVBNCVBNyVFNiU5NSVCMCVFNiU4RCVBRQ==

### 抓包分析

搜索关键词，可以抓到上面的请求

可以看到在 header 中的几个字段应该都是加密的

### 加密定位

直接检索关键词 `x-itouchtv-ca-signature`

可以找到下面的文件

在文件中的位置上直接断点，然后点击下一页，就可以看到已经断上了

### 加密分析

#### X-ITOUCHTV-Ca-Timestamp

这个见名知义，就是时间戳，并且在上面也可以看到代码

```
d = (new Date).getTime()  
"X-ITOUCHTV-Ca-Timestamp": d  

```

#### X-ITOUCHTV-Ca-Signature

这个字段，涉及到了 `o.default` 还有 `l.default.stringify` 还有 `v`

```
l.default.stringify((0,o.default)(v, "dfkcY1c3sfuw0Cii9DWjOUO3iQy2hqlDxyvDXd1oVMxwYAJSgeB6phO8eW1dfuwX"))  

```

传入的参数 v 是由下面的代码计算出来的

```
var v = "".concat(t, "\n").concat(n, "\n").concat(d, "\n").concat(_);  

```

由上下文可以知道

```
t = 当前请求的方法 post/get  
n = 当前请求的链接  
d = 时间戳  
_ = l.default.stringify(p))  
p = (0,r.default)(a)  
a = 当前的请求提交的参数  

```

所以目前需要知道的是参数 a 经过的两个方法 `l.default.stringify` 还有 `r.default` 分别为什么

单步进入可以看到，这里的 r.default 是一个 md5

经过计算之后的结果，传入 `l.default.stringify`，继续调试

可以看到这里的 `l.default.stringify` 就是一个 base64 的编码

经过这样计算之后的结果，和上面的 t，n，d 用 \n 拼接到一起，得出参数 v

然后这个 v 经过了 o 之后得到最终的结果

单步进去可以看到

先是进行了一次 hmac

这里的 key 是 `dfkcY1c3sfuw0Cii9DWjOUO3iQy2hqlDxyvDXd1oVMxwYAJSgeB6phO8eW1dfuwX`

加密的 msg 是 `POST\nhttps://gdtv-api.gdtv.cn/api/search/v1/news\n1663207996688\nfGfnMG3h1j/3YxyPfIJ2Ag==`

ps：注意这里的 msg 拼接使用的是 `\n`

在计算之后使用了 base64 也就是 `l.default.stringify`进行了编码

最终得出请求需要的  X-ITOUCHTV-Ca-Signature

带入 python 测试结果如下

End.

以上就是全部的内容了，咱们下次再会~

备注【**咸鱼666**】，入群交流

**我是没有更新就在摸鱼的咸鱼**

**收到请回复~**

**我们下次再见。**

**对了，看完记得一键三连，这个对我真的很重要。**