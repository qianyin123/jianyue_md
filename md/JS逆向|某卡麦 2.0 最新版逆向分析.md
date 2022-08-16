> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wPA0VhrAq1gUYeFnaSUR_w)

**关注它，不迷路。**

  

  

本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者立即删除！

  

一.前言
----

研究akamai已经有些时日了，市面上akamai分为两个版本：1.75和2.0，其实也不叫2.0吧，因为新版的没有像旧版的一样指定版本号，为了区分方便暂且叫它2.0。  

1.75版本就不用说了，逆向起来很简单并且代码不会再更新，国外的航司用的比较多。重点是2.0，不仅每周都会更新，而且最近加强了风控和混淆方案。

**目前1.75和2.0我这边都能并发，需要的可以联系我 v: 523176585**

二.akamai js部分
-------------

混淆前的代码长这样:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR24xdclWIDEYusoE6UDc7leHgrjjrdnTyh9W0QYlR7yt1iakInCg9mE1qTGcv6ODHfn9WQI7nmj2pw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

---

目前没有好的办法来还原，因为有些是事件触发型，在没有触发前，还原出来的字符串乱码,如图:

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR24xdclWIDEYusoE6UDc7leeTXmT41OAyIaTjuKxI14DB34zvpdm3H4exRH6m069050qck12EfKLg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

很多字符串没法还原(我相信有人可以)，但是绝大部分还是可以还原，我使用了一点小的技巧，node配合浏览器断点来执行并替换:

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR24xdclWIDEYusoE6UDc7leg8xrfbibNHeMxzP7Qic9b9ic1cpzPVa9hz4WW2Of4Lq58lYYMHtHKOU1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

当然了，你做过上个版本的，不还原也没事，它一般不会大改，经常改的就那几个地方。  

没搞过的也不用慌，我使用插件，直接定位到senser_data生成的地方，找到还是很容易的。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR24xdclWIDEYusoE6UDc7leM4XX9HDpAKRKMax14ZbH9I4sZEWWwZOEuaL2BpXMPXUqWNwtRJqicxw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

这里就是那56个数组元素生成的地方，全局搜索，然后打断点调试即可。抠出js后，生成senser_data，请求多次(每次校验的点不一样)后，能看到大佬们常说的破盾标志(cookie的_abck字段):

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR24xdclWIDEYusoE6UDc7leE0iaTqte76qOPHU2MnEDdpxeCRkBibhibiatsiaboEGQeokPSxGibDcbuCOg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

拿到这个后，某芯片购买网站能够加购物车了:

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR24xdclWIDEYusoE6UDc7leYbicia0okV8yyaprrYroR6EyxicOXMVZ8fgcchtmmUqGzr4picRDCTibicqw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

解决IP,浏览器指纹后的并发情况:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR24xdclWIDEYusoE6UDc7le647DTyobHWKKeqdH030E75Ksicic2CJqTic7jZkiaXHoa9bibc9CzTibRjtA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

三.说说它的风控
--------

关于akamai的风控，这个是老生常谈了，毕竟js逆向难度也只能算个中上，难点全在风控，**风控没有解决就不能并发**。

*   首先要过tls检测，当然在测试过程中，不过tls也能正常过盾，但肯定上不了并发。
    
*   浏览器环境要用真实的，不能简单随机，比如canvas、ua、webgl数据。它检测了大量的浏览器指纹，极有可能一个不正确，就返回经典的403。并且还检测了自动化框架，所以用selenium等自动化框架得先过掉它的检测。
    
*   行为数据必须要带上，如果是pc端需要模拟鼠标轨迹以及键盘事件，移动端需要螺旋仪、触摸事件等等。  
    
*   不同网站的风控级别是不同的，可以过某些站不代表可以过所有站。目前看来，某芯片网站的风控等级还比较高。
    
*   风控是在长久对抗中才能摸清门路的，这个主要靠经验，只有测试的情况足够多，才能摸清它的风控点，所以需要足够的耐心。
    

  

经过测试，捷星jetstar.com，德州仪器 ti.com，w6 wizzair.com，还有dhl，spirit这些站都可以并发，目前还没有遇到过不去的站。

**如果需要，可以联系我 v: 523176585**