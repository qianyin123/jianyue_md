> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/OkxCbGX5XvlDXzr6mYujsg)

  

免责声明  

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6eccyrNSOic72rU60MknlI9pEoWgY2WcDNygkNG5NtzxaWSibj18LFoScqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

文章中所有内容仅供学习交流使用，不用于其他任何目的，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业和非法用途，否则由此产生的一切后果与作者无关。若有侵权，请在公众号【爬虫逆向小林哥】联系作者

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6ecnicjTbwdy5ze4PutY5ADhJD1xKy7WpcxMqLhyw93ccIQO4QCP1T30Xg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

  

  

01

—

逆向目标  

  
aHR0cHM6Ly9hcHAueWFuZ2tlZHVvLmNvbS8=

  

  

  

02

—  

抓包分析

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK76POGPLXfW7ISZqwriaernEHia9kGVW2pmMPAibqiaTD8Pm2oLcv5QeKM1g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

  

03

—  

逆向过程

通过搜索没搜到，直接xhr断点跟栈吧  

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7iaic9wnGQmQZF0iaeKGBAia9VeRLgpJ7fjicnomjzOBAgibtx6bEZGz5qBJQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从上图看，应该是在下面的异步栈里面，

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7zEXDXYn5WhdqF9VgbNoM9HCWB6wYG4mflPpaWOXYLU30ibrwEiaUcsMA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

发现了一个可疑的地方，可疑在这个地方下断

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7eT65TErHqhZ8KFY8aga3BsibPK0Z0lricWHvCQLjqzj9O14KdrY8aU1g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

看到这里已经生成了，那么我们看它之前的栈

```
`this.httpClient.get("api/caterham/query/fenlei_gyl_group", {``params: o``});``(a = e.sent) && (o.anti_content = a);``at.a.getAntiContent();`
```

我们可以在at.a.getAntiContent();下断分析

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7j5vxo1rmLZ4UiaQm5GVPd2lKE2dSHzn5fashraRfyGOkhdk55weQKAA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

因为这里是一个异步函数，我们需要F11进入函数内部

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7VzV0qCLgGgsictkX2aJUiaqHhjFFed66EAk0IclRDL4AqOFWTvPoUdUQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里返回的是一个异步对象，我们可以通过在s(void 0) 断点，进入耗时任务中

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7m3fpK91ZfYib2pWEL2hrajxlibLHEwUNoktf5sBeCNeTgAx5gQcdPUDg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

一直F11，道上上图另一个promise中，继续F11

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7mXHiaVZ6ibXicIeya2EhyoLj4s8RruFnV0eL6FXLVTggcWKKiaPa8UosCA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

一直到上图位置，promise状态变为fulfilled，并且我们要的anti_content才出来，因此，加密参数的实现就在这个this.riskControlCrawler.messagePackSync中，我们进去

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7dLaHYTwRPFf6InPzq1ciaOiclTmcGNBlSd04sRsKqCic8JDakkNTtialhA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

整个代码是经过混淆的

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7ianQGZ3IkZJXScLI7ibGicDcuHe0BfzkNqYRsDqMQooUTbm7ibiaHSIKkBA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进入了le这个函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7cS7Uia0HDk1ZtVlDq4ibv9FSPsknv4UjvoeDZ7j3IF6eoQvs02Ggtg8w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

又le的返回值  加载出加密函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7lX3MvsxuJjlAKWl8Ry3zTgjWPl9AuBTQpb1WbYwD9avo9oicA7hpzIQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

将整个文件复制到本地进行格式化，发现是分离式webpack

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK781j08BbM8JUjbks2Hh8icURDzflr72HtpTHtAaKXMVVtRtI0TaicQkXA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

而我们需要分析的加密函数又在fbeZ 中， 然fbeZ又是一个webpack

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK77dlfvmGQ6XjsDpicw3v9bEuibCaKcEPibVNcgFbddVXh61PxOj2qjKshQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

下面我们把fbeZ的webpack复制下来

```
`var sign;``!function (e) {``var t = {};``function n(r) {``if (t[r])``return t[r].exports;``var a = t[r] = {``i: r,``l: !1,``exports: {}` `};``console.log(r)``return e[r].call(a.exports, a, a.exports, n),` `a.l = !0,` `a.exports` `}` `sign = n;``}({``})`
```

加密函数所在的函数数组位置为4，我们依次扣下所有函数

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7HOPCyvOiaR00XgqQf6M0Av0icF4qTvB30KukvMA8AD0V1Qkq33bzXHtw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

调用sign(4)，然后跟进栈中

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7QEC5bKm4EouRRQCxz7Xmh6kFy0XpU2d2XQAUKicAau7CHTAJkInO1fg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们这里的arguments没有参数传入 因此是0

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7nxNAeUia54p5zwicDXemibb53TK7H5GeFI0RINgmuQof8C7qhDXSe2uGw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

但是浏览器上面传入了一个包含时间戳的对象，并且下面的逻辑只用到了这个时间戳

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7Uzo8T5MTcicmx8DXWwZlRibxjBZ0Oj1su2XPl51Scow11XqCzbIsFg5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

执行了updateservertime，最后return了Be 也就是上面的qe

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7AWbvMYXB0YMAia2lbPPpxwMteoSH9wkwPaibe822pjjHrAcUnMG02A2w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

然后上面我们分析到的加密函数地方在

```
qe[i(695, "lD!i")][i(539, "lc@H")] = qe["prototype"][""]
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7UEBehvbrMdoQBt870YDV1bdW3MwiaKUlhJW9uTWN3WCLPA9xpHh7rmg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

因此在我们本地只需要下面调用即可

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7PpV9cic0V5v0Lkj1qZaeK9NsrU4nvaGlZJHxqib7suibnyFjqMr1BEcVA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在本地运行并无错误，猜想可能是因为缺少环境，那直接用v神的一键补全环境大法：

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7lmJEBSsl8ay4RnfmxFKuZoF29y6ibNE4CzvvFhyKLtMT8WIEicTzIj2Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

成功返回啦，返回的是promise，我们稍作调整

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyQ1fFJaIicFZDCqydMVgbK7x6ylL1vldsSjIia2TTwJlCQ1f3qDDLRhX81iaY8tLyMInU7q6u2XXP3A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

大功告成！

  

  

  

  

  

  

  

  

04

—  

算法还原

  

公众号回复：【拼夕夕anti_content】  

  

05

—  

归纳总结

  

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6ecwtV3Ot2Y5VCuGU0DibxkkurkYJ2QzbN96L6ibFbBgOEM8TYpH4P8A8eQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuwT0dUxFRAYKJckhBKVG6ecwyVZbsGqS6q1xRoreyqHokuq1KdtUq6A4dkuPFpqVjR0loz0QElpNQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

添加好友回复：交流群