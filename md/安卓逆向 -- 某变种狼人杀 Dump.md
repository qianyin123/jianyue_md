> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/jgqGJ11dTbOiBBfebv_40g)

一，前言
----

### 研究仅供学习交流，如有侵权请联系删除

#### 目前主流dump分下列这二种:

##### 1， 提取游戏metadata和il2cpp 之后使用Il2cppDumper进行Dump

##### 2， 使用zygisk-Dumper 进行动态Dump

###### 但是前者大多游戏都有对**metadata** 与 **il2cpp.so** 进行加密

###### 后者由于是透过符号查找il2cppAPI进行动态dump

###### 但有的游戏会隐藏或加密导致获取不到il2cppAPI

##### 这款游戏都没有办法使用上面两种直接Dump

二，分析
----

使用静态Dump发现失败了!  
  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0otrCpGqjhxdVkS6FcCcwq03qpe2TibkB74yu6zN8Uk4FKAHtcY2KwjaUHy5KJ7RMGoDpAeMUzyLKA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  
  
但好像可以更改il2cppDump去修复(但我打算使用动态Dump)  
之后我使用动态Dump发现也同样Dump不出来  
只能去看看他的SO了  
我先使用readelf查看符号，发现并没有Dump需要的il2cppAPI符号  
  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0otrCpGqjhxdVkS6FcCcwq0JtdXDpTShY6CHO3PD6g30yeTNRsd6XTdNMHrgQ9MR2uYuUMv36Armg/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  
  
那我们把il2cpp拖入IDA看看怎么了  
跑完SO直接Shift+F12查看字串  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0otrCpGqjhxdVkS6FcCcwq0M99ibZuVt3uMQiak1X8RgteGmb0NRSOiauwlLVF9FDQib68hKnfogj0doQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

搜索il2cpp_domain_get会发现有函数用到直接进去F5!!  
  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0otrCpGqjhxdVkS6FcCcwq0BMvTQtrDxSNEKatk8LMLwjZ9X7lgLeG8atJxoUCyt0K855sfPde37A/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0otrCpGqjhxdVkS6FcCcwq0cJCrWibA8T0YE20yb2l5UUiccW9rPHJxJNQMy8F64TibvdsulHldxIeRg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

三，查找Il2cppAPI函数
---------------

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0otrCpGqjhxdVkS6FcCcwq0uKLpx2zTk9ktf0B2NLmhsZNz7MEybxmfZnXng5A2EEauPhxFItHTpA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  
  
我们发现他把字串赋值给v0 v0+1是一个函数  
我就猜这个函数就是API了  
我们直接查看其他游戏的API函数内部实现去对比  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0otrCpGqjhxdVkS6FcCcwq0SNsKqgmYuoE23NHo8qChDKDERXQ0GhcmMJ5YQb6n4hRfDqOA3yDGRQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  
  
我们发现除了函数名不一样其他大致一样  
至此我们就找到il2cppAPI了  
剩下就写个工具把需要的API提取出来就可以了  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0otrCpGqjhxdVkS6FcCcwq0j8qQADpAnibTv3icpvf1Bkd0fMolSPO1Oh4uB4Rg7dJDyZcibwOPoDprQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

四，Dump
------

找到API就轻松了，可以修改Zygisk-Dumper源码  
或者使用Frida-Dump  
这里我使用imy大佬的frida脚本进行Dump(还可以dump某盾)  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0otrCpGqjhxdVkS6FcCcwq0hcsJHo9I0hiaVQfg24FvITssYl0KNUlicSicnxIVxJunLwRbTWWax4vyQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

**·** **今 日 推 荐** **·**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0otrCpGqjhxdVkS6FcCcwq0wQ4QDsHLNUqG2eg5FFlbUdpmZXaaKsttKOia7hBTCegYQ73nXBic2iabQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

本文内容来自网络，如有侵权请联系删除