> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gjDIf30ffH9ixD2OjaS5_g)

点击上方“Python爬虫 js逆向”，选择“加为星标”  

第一时间关注爬虫技术干货！

  

距离上一次某音乐更新之后，已经有5个月接近半年没写过文章了，主要是因为太懒了，感觉没啥好写的，所以今天呢，为了表示我还健在，特来水一篇某音的一个小参数，当然，蚊子再小也是肉嘛，也是vmp里面生成的，溜了溜了

前言
--

本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者(wx：xxb00414)立即删除！

网站
--

aHR0cHM6Ly93d3cuZG91eWluLmNvbS92aWRlby82OTYxOTk0NDA4NTYwNDYzMTE4

目标参数及定位
-------

打开目标网站，往下拉，加载评论，在开发者工具里面用comment关键字筛选就可以看得到一些数据包，随便点开一个，就是rcFT小参数，别看他那么短，算法还是有点意思的，so，本文的目标就是他的算法了

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k2y3UtUwEFfdtYFgdPJbsib1RpT2GbDMNZiboXqgGSV4Bhia027o621PnV6o1LOjrBQib492J4r1qiaVog/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

关于定位呢，老规矩，搜索rcFT，来到下面位置，打上断点，向下拉加载评论，就会在断点处断下，单步跟，就会进入字节系vmp

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k2y3UtUwEFfdtYFgdPJbsib11IStsAuJKUQoAicZdDftho7mcdQswwRMibpbezSxdQOhoQux5oGSTibXg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

建议进入r.Z函数，在那里面随便打一个断点，防止vmp参数log输出结束了，还去执行后面的流程再进入vmp里面输出，就不好分析了  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k2y3UtUwEFfdtYFgdPJbsib1lZxFp4msu1V6sGAFvG7ichlyQL69JDoibQ4rfXJsJV1tcTTlibjWtMAmA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

算法分析
----

现在vmp算法还原的文章也有那么多了，相信大家也会一些小技巧了，第一步终归是离不开插桩，一个好的插桩，能在还原算法过程里面起到至关重要的作用，反正据1900大佬跟我说，新版某乎vmp是比较特别的，基于寄存器的vmp，其他的大多都是基于栈式，咱们也不管啥啥寄存器也好，栈式也好，我就只知道他vmp里面每个指令操作的都是哪个变量，我们就尽可能把那个变量的值都打印出来，就可以分析算法了，将下面的log断点内容分别插在vmp上下两个循环的第一个if处，分别在webmssdk.js的177行跟475行

```
_0x6f0288[1], "<<->>", _0x6f0288[2], "<<->>", _0x6f0288[3], "<<->>", _0x6f0288[4], "<<->>", _0x6f0288[5], "<<->>", _0x6f0288[6], "<<->>", _0x6f0288[7], "<<->>", _0x6f0288[8], "<<->>", _0x6f0288[9]
```

  

然后呢，就放开断点，静静等待log输出结束，断到刚刚说的r.Z函数里面，大概，1.4w行左右的亚子吧，还好还好

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k2y3UtUwEFfdtYFgdPJbsib1CibnicdLcodhiatIydHuC2IKp2orRulcCgJRzUFm7rzKwYmdsq5hOYAAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

最后的参数值呢，就是AADqmZvSQ，通过每次评论数据抓包，都可以发现，AA是一个固定的前缀，不用管，所以只需要分析DqmZvSQ

  

1. D

老规矩，直接搜'D'，记得加上引号，不然一搜很多，可以看到有fromCharCode函数，还有一个[68]，随便猜一猜

```
`0.12641606836915464(random.random()) * 25 + 65 = 68.16040170922886``Math.floor(68.16040170922886) = 68``String.fromCharCode.apply(String, [68]) = 'D'`
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k0cB3c3VrbcRibhE49YS87LqCVdOvaicMkaZvibwT3YAD19EonFbPicKuQG8GIEVAgJBiaoS9kHicy5oKNg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k0cB3c3VrbcRibhE49YS87Lq1c7Xkkkb2X7mGB4h2zRRv3W1vcJvQbh6Z6xV9ricz1z0RrNdpzgSp8Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2. 后序算法流程

    1) 准备一个从0 ~ 255 的256位数组  

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k0cB3c3VrbcRibhE49YS87Lqia7d3XdKmn8RicxZsgiblgdicUzd0EDZox03kUgFKEaibSIQM4QnuXhoW3Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

    2) 根据一开始的随机数，对0 ~ 255 的256位数组进行洗牌，大致算法流程如下，这一部分算法结束之后就会得到一个打乱了的256位数组，说到了洗牌算法，顺带提一下吧，国外某f5的shape也有洗牌算法，感觉差不多，也是对数组根据下标两两互换  

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k0cB3c3VrbcRibhE49YS87LqIcJshKT25wsmxbKK2nBexAd1mkQLJgs8TpdyDbFJoeIE2wRrSWBxaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

算法还原后对比前100位如下：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k0cB3c3VrbcRibhE49YS87Lqvb4nAdats9Fibw3iad7hPrBiaFQ1V0YhlL0Mv4tBT1g9cqgSA5j7KRJLA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

    3) 根据乱序之后的256位数组 arr_256 与一个一开始根据user字符串初始化的4位数组 user_arr_4 运算得到最终四位数组，再对最终的四位数组 last_arr_4 进行fromCharCode得到最终字符串  

*   最开始通过"user"字符串得到一个四位数组
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k0cB3c3VrbcRibhE49YS87Lqs07bpdcyd7T6OsKZXfSJQWpIxBgaibtKXl2UCwOyiazbic5sWpvIViaQCQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    117 ^ 160 = 213

    115 ^ 160 = 211

    101 ^ 160 = 197

    114 ^ 160 = 210

    user_arr_4 = [213, 211, 197, 210]

*   4位数组与256位数组运算得到最终4位数组，进行fromCharCode得到4位字符串  
    

    index_1 = (arr_256[1] + 0) % 256 = 90

    arr_256[1], arr_256[90] =  arr_256[90], arr_256[1]  

    last_arr_4[0] = user_arr_4[0] ^ arr_256[(arr_256[1] + arr_256[90]) % 256] ^ 10 = 170

  

    index_1 = (arr_256[2] + 90) % 256 = 30  

    arr_256[2], arr_256[30] = arr_256[30], arr_256[2]  

    last_arr_4[1] = user_arr_4[1] ^ arr_256[(arr_256[2] + arr_256[30]) % 256] ^ 10 = 102

  

    index_1 = (arr_256[3] + 30) % 256 = 250  

    arr_256[3], arr_256[250] = arr_256[250], arr_256[3]  

    last_arr_4[2] = user_arr_4[2] ^ arr_256[(arr_256[3] + arr_256[250]) % 256] ^ 10 = 111

  

    index_1 = (arr_256[4] + 250) % 256 = 88  

    arr_256[4], arr_256[88] = arr_256[88], arr_256[4]  

    last_arr_4[3] = user_arr_4[3] ^ arr_256[(arr_256[4] + arr_256[88]) % 256] ^ 10 = 73

  

    last_str = String.fromCharCode.apply(String, last_arr_4) = "ªfoI"  

4) 对最终字符串进行类似b64的编码得到qmZvSQ，最后将固定前缀AA，与随机数得到的字符D拼接得到最后密文AADqmZvSQ

emmmm，已经写了这么多了，大体算法都完成了，有点懒得写了，手动狗头，最后的类似b64编码算法就由看官们自行分析吧，思路都一样，没啥难度，这部分编码算法在很多vmp里面都有使用，比如某音的x-bogus、mstoken，某滑块vdata参数等等

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k0cB3c3VrbcRibhE49YS87LqObBkAr4lLApcuOicX0szotUAgrITPERIAicicqw0H3Go00pYicN6wVCibnQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k0cB3c3VrbcRibhE49YS87Lq5X7w7S8CZDo4SKrs8vSnhc8ic0trUpicSl0DhzpGdTg4VjbCO1qSw5UA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

整体算法py还原下来，也就70来行，难度一般般，最后付费区就是算法源代码，需要的可自取，完结撒花![图片](https://mmbiz.qpic.cn/mmbiz_png/EiaqCy3pb5k0cB3c3VrbcRibhE49YS87LqHlhqgUJ2VkPcWY8LsoW9HorV8vibm2rScd5N1J5LibPHVDBlkguQdIMQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)