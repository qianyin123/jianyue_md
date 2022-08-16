> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwNTY1OTg0MQ==&mid=2647563627&idx=1&sn=b3ea92deda78329e29e1cece3b1433eb&chksm=832370c5b454f9d3ee9e748db825a6a53d9b5fe8b7d16acbaeeba15b5f8d892f3447815c2457&scene=126&&sessionid=1660619935#rd)

**关注它，不迷路。**

  

  

*   本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者立即删除！
    

  

应客户要求，写下ti线上调试的步骤，感谢大佬们的支持！  

一.工具准备
------

1.  浏览器映射插件: **Netify**,下载地址:
    
      
    
    ```
    https://t.zsxq.com/023nY7qVz
    ```
    
2.  过掉Akamai的toString检测，下载地址:
    
      
    
    ```
    https://t.zsxq.com/02BMBiyFE
    ```
    
3.  还原后的Akamai脚本;
    
4.  谷歌浏览器;
    
      
    
      
    

二.还原后文件修改
---------

1.  使用一键自动还原脚本还原混淆的js，自动保存为 decodeResult.js
    
2.  使用 一键生成toString检测 脚本，生成toString环境，并添加到上面的 decodeResult.js 文件中，加到最开始那行就行。
    
3.  全局搜索 "2;" ，在加密senser_data的语句后面添加打印，看结果。
    
    比如现在的代码定位在这里:
    
    ```
    Dh = "2;" + cL[0] + ";" + cL[1] + ";" + DL + ";" + Dh;
    ```
    
    在后面添加:
    
      
    
    ```
    console.log(Dh);
    ```
    
      
    

三.使用Netify进行映射
--------------

在谷歌浏览器添加好Netify插件后，按下F12，切换到Netif面板，添加规则:

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR0e7tkgMEZaHiaucWCB1EFGFSgjvTRegYLKajaE57xK8YSjtG9NcQuaUd1dviaMGdvCEQzGc6syVJGg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

点击加号后，按如下方式添加:  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR0e7tkgMEZaHiaucWCB1EFGFn7onyiaicmgXWYiaSZufLu0sDQEvpbLlwt6hKiamV02A9oBObxHhhFFXxA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

①，填入需要映射的js Url  

②，根据网站抓包分析，这里选择 Script

③，根据网站抓包分析，这里选择 Get,这样就不会映射post请求了。  

  

切换到 Local response，  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR0e7tkgMEZaHiaucWCB1EFGFTkj6gBsTz9fBWUibh2wwrpEYyl8IVNOeffkHqslSPXCtwTliaiagYd0fg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

在body位置，可以选择Text，既将整个js复制进去，也可以选择File，直接映射本地文件，我这里选择File。  

  

选择好以后，点击下面的保存按钮即可，

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR0e7tkgMEZaHiaucWCB1EFGFUSicty83WKBy89fxxiadyQNou0qwD9xb8CPBJRyIolKTH8gr2Q5ib3Q7g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

这里就是添加好了的规整。

  

  

四.结果比对
------

打开Akamai 2.0 的网站，并打开调试工具，点击 如下所示的按钮:  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR0e7tkgMEZaHiaucWCB1EFGFEJOxgLs5J3M9srHPabEZBq6qNcMInKaib4op4ThAv3CYYT29nxAM1LA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

如果出现右上角的提示，说明已开启映射。  

  

刷新后，可以看到 senser_data是和控制台打印一样的了。  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR0e7tkgMEZaHiaucWCB1EFGFpPrFNu56F1M13jCApRUQQQibkyt5nqlMW7ZsyC2ptydrCcAhic729KMg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

这样，可以在代码的任意位置添加debugger，愉快的进行调试了。  

  

今天的内容就介绍到这里，如果你有更好玩的工具，欢迎分享给我们，感谢。

  

五.交流学习
------

  

最近我的4个群都加满了，准备建个5群，想进群的可以加我好友，拉你进群，注意，严禁讨论破解相关的话题。  

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Lll8tx0MDR0kCx9rvzFflV5uIzLHZwQ7BxgpibUhX1j1fD0KU6TUmYIiaxBP5ibqWibBqOTGn5cj200eW6MEZEB7Qw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)