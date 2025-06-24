> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2_NF8lc1BjAxGHse3Q4OEg)

逆向目标
----

*   网址：`aHR0cHM6Ly93d3cubGFuZGNoaW5hLmNvbS8jLw==`
    
*   目标：反调试及请求头`Hash`
    

反调试
---

进入网站，我们会发现`F12`无法打开控制台。那就换一种方式，以谷歌浏览器为例`设置 -> 更多工具 -> 开发者工具`，然后网页就会跳到空白页了，这是除了`debugger`外一种经典的反调试手段。

那么我们如何过掉这个反调试呢？目前，我遇到的就两种情况，第一种就是`window.close()`，第二种就是`location.href = ''`，还有其他情况的话欢迎各位大佬补充。

下面直接开始分析：

我们用无痕浏览器分析（因为在`Script`断点时不会加载插件的JS代码），打开控制台，勾上`Script`断点，输入网址，成功断住。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8aJbwgmkz5wJ6vOSfzcsE8y889SP9Vka0UibtCyQ92aEK73zDP8xDicJQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

然后我们先搜索`window.close`，`F8`一直走，之后某个文件出现了相关代码，直接下断。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8uxdgScaQeichs8BmpTXIJZ9e7plAsNfCJTmCp0RkVE5oLPHCDDffRyg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

然后放开`Script`断点，直接`F8`过掉，成功在断点处停住。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8ewDXyAzcfu00O3GVxB6muic65BPEHuO7ezcnV5pMabriakWIicGianiaK4w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

我们往前跟栈，看网站通过什么手段进行的反调试。可以看到，是`e > 10 * this.maxPrintTime`这个条件成立才触发`this.onDevToolOpen`，从而触发后续的`window.close`

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR88xJGetzvfiaHDGQwgJgzRpr2GDflAwOr9lQg2ul3U1bmET0iaIfuYPaw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

那我们就看看代码具体做了什么，我们在前面的`e`和`r`处下断，重新请求，成功断住。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8HpdYraAtaua1Tic0lWuicickaGmLTpWTPVd2UIlzfSrxgdnPyOVJuq1mw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

然后我们单步跟进去`m`函数，可以看到，先取了一下时间戳，然后调用传进的参数，最后再返回参数调用的时间差，那猫腻肯定是出现在传入的函数中了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8261ud16XHObHvd4KmT6m6vzibYPZZ8cIdVHxhNLSf2IZ8l3m2moRTxg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

我们直接进入被调用的参数函数中，可以看到`t.largeObjectArray`是一个大对象，`b`其实是`console.table`函数，也就是在控制台打印一个大对象。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8sxhUtpR6rd4KaqwnJ5icH8sEgnJesx7W9cCTWY7ZMIqRBqib81fZ3o2g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

那反调试的逻辑大体清楚了：如果打开了控制台，就会通过`console.table`打印一个很大的对象，这就需要消耗时间，后续`e > 10 * this.maxPrintTime`就成立，就会进入最终`window.close`的逻辑。

我们可以通过`hook`，返回一个固定的时间戳来过掉反调试（油猴或者直接在控制台注入都可以）。

```
 _复制代码_ _隐藏代码_`Date.prototype.getTime = function(){  
    return1741067137856  
}`
```

`hook`掉时间戳之后，我们会发现控制台一直会被清空，也不利于调试，我们直接把清空控制台的代码置为空函数`E = function(){}`。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8EJdcDeLTlSUCumL6h3LPiaxpI8xpAaicIoicRgfaEicZnyzjOZWOWkYaaw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

同理，再把和控制台打印相关的`b`和`w`函数也置空（在对应函数断住的时候去置空）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8XyyKib0f8DCcEgrNdVx7jkLVibicdjp4D83DVrsGx0WRu6A4EYrjx4mhA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

最后，能够在控制台输入并输出表示我们成功过掉了反调试。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8B8pmXnfEiaHAiakicyYgj1O1zslqJOTYZ6DNictnEPFXAPiaaHxbibDTttEQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

抓包分析
----

我们直接点击`土地供应 -> 供地计划`，会发一个`list`的包，请求头中的`Hash`参数就是需要逆向的参数。

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8UhmctiaSqUAPHeO643nI0KOQAriaFGJG6SPI8T6BcU6gKjrSkj2y7E6A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

逆向分析
----

一般请求头参数加密，我们可以通过`hook`的手段去找加密位置（可以去找现成的`hook`代码）。我选择直接从启动器去找了，一般都会看异步之前的`request`。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8LniadyoXiaC8ZcHCYZQJUkichRQSFlq6eOAolTUNgU33nUJn0nMoq0fTw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

下断，发包，然后找请求拦截器。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8sCGNoBxJiajpE0G6rxCuCARn3rtJGhCoaTgzhSXhzaDckrZoDdicC99w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

很显然，下图大概率就是加密参数的生成位置。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8mLdrpjnLxO3SEt7aZ9GMRckWNsz3ZnibsnWpYH4v5TJnSJD1BSrn7DA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

我们在控制台输出一下结果，可以看到密文的长度为`64`，看起来很像`SHA256`。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8ZzOLqO6aXxLmlZYYaMxibUe4EI7T3UXFNibf8y3YCEZRicJEoUXWTBibpQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

那我们就去尝试一下，看是否为标准的`SHA256`，字符串`'1'`的摘要为`6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b`。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8xaGmqCfBFQTE1OnTOoaDsTkVahgqocgsfqZeeLzjcC73QSMRPgB0nA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

再去控制台对字符串`'1'`进行摘要，经对比，算法是标准的，没有魔改。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8VIuzlzXelNqmlFA3SibpwRabYBtDKQRic6FghJWAkVNbt0VrKHUoCgoQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

那加密过程大概清楚了，对`userAgent`、`今天的日期（日）`以及`list`进行`SHA256`得到`Hash`。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8xxlQgqRbms2zia3miaspiaOh0OCJfeHCiaERVN2uCP9CcSmG3mT8VFqnMg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

那我们直接对接口进行请求模拟。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qAnxIUgvqkLvODoicYiabVR8LJUqzLbEyNoyyJAfsjiblWCteTv0sDh0H8YGSjvCv6zr3aum5eYJR3A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

成功！！！

  

  

  

  

**·** **今 日 推 荐** **·**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oRzx6ROGOUNQGH1hiaWR9tGJticmcp8YLSxmicQssTOsMjCsa8mGYDJLaBznezsxoN1RZ3AyXLQjA1A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

本文内容来自网络，如有侵权请联系删除