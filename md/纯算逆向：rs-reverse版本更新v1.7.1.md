> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/FOuL45epDAbJf4EWlYATTQ)

> rs-reverse介绍: 瑞数vmp网站完全算法逆向，提供npm包及命令方式调用，功能包括动态代码还原、加密cookie生成等
> 
> 同时推荐使用开源补环境框架：sdenv（见github仓库）
> 
> https://github.com/pysunday/rs-reverse

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/bIcvM6RhiathT5xyX6iaTicJmW2FJbSic9ibbOOgKysILoxE8anWy5hXrfcxe7yR7JRTGT2gQyibfAMvw6AhkbmMlUvA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

此次rs-reverse版本更新改动如下：  

1.  新增makecode-high命令；
    
2.  新增basearr命令；
    
3.  修复exec命令版本不兼容；
    
4.  完善makecookie命令中basearr生成正确性。
    

  

**1. 新增makecode-high命令**

        该命令是此次版本的主要更新点，故名思义即为makecode命令的进阶命令，该命令只支持远程请求方式（-u），功能涵盖makecode命令，调用命令示例：

```
npx rs-reverse makecode-high -u https://wcjs.sbj.cnipa.gov.cn/sgtmi
```

        执行完会在output目录下生成makecode-high目录，并产生网站资源文件及解密代码文件与动态代码文件，如：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/bIcvM6RhiathT5xyX6iaTicJmW2FJbSic9ibbB4L7y3JgRpvD8apCzKPX3BLwxdHqmVUEziaf6KbUEx0oskbP351cQQw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

        可以看到makecode-high目录下存在两个子目录，分别是first目录和second目录，了解瑞数的都知道，瑞数网站只有在第一次请求生成正确cookie后才会在第二次请求中返回网站内容。因此这两个子目录分别为两次请求产物，这也标志着makecookie命令产生的cookie是正确可用的！其中对比makecode命令可以发现first目录的文件是与makecode命令产生文件一一对应的，而second目录多出了两个js文件及对应的解密文件，多出来的文件即为网站渲染用的js代码文件。

        需要注意的是，请不要连续请求以免触发风控，如果触发风控请稍等之后再试，或者修改makecookie中config的值后重新尝试，如报错：  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/bIcvM6RhiathT5xyX6iaTicJmW2FJbSic9ibbqnDaSRVLBSagt0WyI4UIuvdCLOiaS5CfSjlUwWn2VzgJbQPt9ybniaOg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

**2. 新增basearr命令**

        该命令与exec命令类型，用于开发中使用，通过传入一个或多个basearr数字数组，并解析数字数组中各数字的含义，将解析结果打印到前台，如果传入的是多个basearr数组则会对比不同，并前台打印差异值与下标。这里不做过多解释，感兴趣可以看命令源码！  

**3. 修复exec命令版本不兼容**

        v1.6.0版本增加多版本适配后出现的遗留问题，在该版本中修复。

**4. 完善makecookie命令中basearr生成正确性**

        由于v1.6.0的cookie生成是在第二次请求的基础上开发，作者在开发makecode-high命令时发起第二次请求仍旧返回第一次请求结果，因此这个问题也是开发makecode-high命令时必须要解决的问题，同时修复应用代码特征码时使用固定值的问题，感兴趣可以看makecookie命令源码。

> 作者创建rs-reverse项目的初衷是为了研究瑞数的加密与反爬原理，通过逆向方式促进技术进步，让对瑞数感兴趣的同学更方便的做研究。作者不建议直接将该项目应用于产线数据采集项目中，由此产生的任何后果与作者无关！！！
> 
> 作者声明

该版本为cookie可用版本，感兴趣的同学可在下次瑞数更新前对比研究。

有任何问题欢迎提issues！