> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/bNdIeaZDaRTJIPUvVVGLgg)

声明：本帖内容仅供参考与学习交流之用，严禁将本帖所述方法、代码、资料等用于任何违法、违规或侵犯他人合法权益的行为。若任何个人或组织擅自将本帖内容用于非法用途，由此产生的一切不良后果及法律责任均由使用者自行承担，与本社区及本帖作者无关。本帖不对因使用或依赖本文内容而可能导致的任何损失、纠纷或责任承担任何直接或间接的法律责任。

最近在使用这软件发现还是很不错的，但由于囊中羞涩故此去寻找方法修改，我发现很多贴子的方法都比较复杂甚至已经无法使用了，所有我在这分享一个简单的，只需要修改四个地方即可  
  
首先是基础工作，使用MT管理器提取安装包，然后对4个class文件使用Dex编辑器+++打开，接着在在常量部分搜素会员，找到永久会员点进去点搜素，再点进去进行编辑 

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HoHOdtz6pO4790hzHQeLiaInehpZSlNl5l0GB9vibTwwC5bXoEm3OC3Sg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HFzGf3icYvQElRshbSk3ufVwic215obGszd5Svbrhu2dd9bVK0rq8Eazg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HgDqOEVMmO4uXjVxpG7SQXFDlFBCXpbDyv6ia2hTuse1wINQEb2bcNoA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HVWxVkdGRRpae7Xed5AcU2CEWia7qkn3HZFPicePgFXPNKSAJYl3iclFmw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

第一个修改的地方就是这永久会员下面的const/4 v0, 0x1  修改为const/4 v0, 0x2 （1改为2就行）

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HsRMCpvDeqUD4Pd0p1c9jkeq8uG3HLZnibzpmoBhicR8CSzUAg23q7bcg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

接着点击上面的getVip Type()  长按再点跳转  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HLRAiazibfuEIEIMWOVT1qxeP8WIibOJAm5UbPRMQgugFToicREP3m0POWA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)  
  
跳转后就是如下所示  我们在这两个红框区域放入两行代码const/4 v0, 0x2   const-wide v0,0x32d57bf5e8L

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HjfDgUyXoMoeSuv88JOeFUn3Jx5jR6KVLGG5wH2lctxIk5bEibzqBzjw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HFJ6vFAOmCoXmtkm4DceRohRbAiatYhPSfgfpt2hbjuJcZdDFuteAu7A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

然后就返回到搜索的位置，发起新搜索（保存），搜索内容是iget (.*), .*, Lcom/wangc/bill/http/entity/User;->vipType:I  选择代码和正则化

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76Htr1GiaEF0QZByZFuszNk3LbakQYPym9YQJzZ7icTSrgtHQ8MjYxUHicEA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HZOj17Weps8KubckJZ3eC0NBKU4qouQd9ibJkY8KqiaNYME68DVp5yENg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

接着替换  const/4 $1, 0x2

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HM7azlsx5hzVPPiah76JAQKE9X2gUQbD1vl7OdicU01qxusrdUIzhyBsA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HgYfliaQptIcLEFMVsRjibTkyiaJw6nN9HmdGEDtudrRKiaJbNcysxAJwuQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11)

  
  
  
替换完成后退出保存就行，如果没有加固就自己加固一下（MT自带）就行  
  
可能教程看着有点多，但是实际操作起来很简单很快  
  
另外，如果自动记账一次就掉，那么你不给这软件使用wifi和流量权限就行

  

  

**·** **今 日 推 荐** **·**

  

  

```
`10月31日更新内容，保存到网盘全免费，保存到网盘全免费，保存到网盘全免费。``在线观看可能有点模糊，下载到本地即可高清观看。``帮您找各类学习资料（影视剧、小初高、公考、教资等等）全免费``联系vx：ivu123ivu``各类资料学习下载合集` `链接：https://pan.quark.cn/s/7c8c391011eb``高清套图合集下载（10月31日更新）``https://pan.quark.cn/s/bd428eb69972``经典车载怀旧音乐合集``链接：https://pan.quark.cn/s/7c9908dbeffd``📒初中九科学霸笔记（无水印）``链接：https://pan.quark.cn/s/a1f27ea4b155``10月27日付费文``链接：https://pan.quark.cn/s/4b2d969efab6``学而思秘籍·初中数学专项突破（含视频讲解）``链接：https://pan.quark.cn/s/32cc2fd33868``🔥 1-6年级学习资料合集``链接：https://pan.quark.cn/s/4d817f989489``北大木兰老师《学霸思维满分数学 (1-6年级) 》``链接：https://pan.quark.cn/s/30f8840a38a3``1025【张可乐家长魔法阅读辅导】``链接：https://pan.quark.cn/s/bb9fdadda0b3``1.吕氏时空人 医算板块-时空本草``链接：https://pan.quark.cn/s/ed2ee02cd13f``强肾教程健体养生健身运动视频全套小白从入门到精通培训课程素材``链接：https://pan.quark.cn/s/849e62119afd``减脂餐教程``链接：https://pan.quark.cn/s/58cc584c6b89`
```

  

本文内容来自网络，如有侵权请联系删除

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oKTNPupib1ZlN3nyubow76HhhCpDoG7PjUnMIBGNNGuKDsaALt6QSgh93WOvHXiaibXH9s7jzB8wL6Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12)