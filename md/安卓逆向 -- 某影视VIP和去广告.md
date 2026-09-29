> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/x6p0qTmTLk3UkBKITXYGMA)

前言：

起因是最近从其他地方搞得去广告破解版的影视APP强制更新了，找了一圈，不是资源卡的飞起就是广告满天飞。

本来没打算写这篇文章的，昨天也遇到了点问题，求助了一下论坛大佬，（这里再次感谢@move 大佬指点）看到评论区还有和我一样的小白在问全过程，因此写下了这篇文章。

本次使用到的工具为 MT管理器 没有的自行百度下载

准备工作：打开MT管理器，提取目标应用安装包，定位到提取的安装包位置。

1.  点击安装包，点击查看，找到 classes.dex ,点击选择Dex编辑器++ ，然后全选.dex文件，并确定，找到搜索，选择发起新搜索 ，
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUVZpBhCsBibru2laudkPxDZauSO0GEoWvTIDO0vCf4icCCUS6M83QicOkWVuqMeEvRO5j5rbVGKev1TH4rECzW3236j6VAGZxSyWQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

  

  

内容输入VIP，选择方法名

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Gv6JExJQjUXaF2OaC1ev0uqLcYO4yGaibLS1nusw8txIfrKTjdeQa2tQ1ia1u6FkzqhEFQd7jDVr3OlqeibT5LpGbiaXd8OeZr9BjZp2BK9Khn8/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

  

  

根据结果，我们看到有个setGet_vip_day()和getGet_vip_day() ,通过方法名不难看出，这是设置vip天数和获取vip天数的方法，先把这里get，set方法都设置成-1（后续有写为什么）-1对应的smali语法为：const/4 v0, -0x1

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

  

设置完成后，保存退出，然后重新进入方法打开，右上角选择转Java，发现get方法已经是return -1;  
但是set方法成为了

```
 _复制代码_ _隐藏代码_`public void setGet_vip_day(int i) {  
  // 这里就是我昨天疑问的点，为何this会变成（-1），知道的大佬评论区解惑一下  
   (-1).get_vip_day = -1;  
}  
  
// 这是我对应修改的代码  
.method public setGet_vip_day(I)V  
    .registers 2  
  
    .line 1  
    const/4 v0, -0x1  
    iput v0, p0, Lcom/http/api/MineInviteCodeApi$Bean;->get_vip_day:I  
  
    return-void`
```

然后保存，退出，更新.dex文件，重新签名，安装测试，发现并无卵用，哈哈哈，看到这是不是发现看了个寂寞。。。

补充为什么修改-1图，这是转Java后的代码  
  

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

好的，来回折腾了几圈，各种尝试无果，我以为是setGet_vip_day()方法在作妖，我就来论坛求助了，感兴趣的可以看我上一个帖子。

OK，废话一堆，以下进入本次破解和去广告的正经核心流程~~~

经过询问得知，这类软件是通过邀请人数和看广告赠送的时长控制的套壳软件，所以上面的vip方法并没有用到，好吧，大佬指出了核心方法：getfreetime，好的，有了目标，用MT发起新搜索，选择方法名，找到getfreetime方法，直接设置到2300年，注意这里赋值的时候的是取时间戳，转16进制  
大佬指点图  
  

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

我们按照思路去走搜索：  
  

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

第二个核心方法：getInvite_num 这个是统计邀请人数的，按照上面搜索，直接修改成99的16进制（看代码得知大于50就是永久），

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

  

OK，满心欢喜，保存推出，重新签名安装，打开APP我们看到破解成功！！！  
  

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

随便找个视频播放，我勒个豆，提示无网络或者服务器异常，天塌了撸，呜呜呜，难道一切努力都白费了吗？？？OK，回忆了一下，我是直接签名安装的，以前看的知识模糊的出现，应该要去签名然后重新签名，OK，签名好重新安装并测试

测试成功！！！！！！！！！！！！！！！！！

好的，VIP破解成功，以下是去广告，去广告说实话我也不会啊，我啥也不知道啊，于是我就论坛搜去广告，找到了另外一个大佬的正则去广告大法,以下是大佬的帖子内容：  
https://flowus.cn/mengqing/share/39515bac-e86a-4c47-b56f-e207c62e2f3a （非本站内容，如违规，管理帮忙ban以下）

于是按照流程替换内容，替换完，重新去签名，再签名，再次测试是否去广告

okk 测试成功！！！！！！！！！！！！！！！！！！！！！！！！！

  

  

  

  

**·** **今 日 推 荐** **·**

  

<table><tbody><tr><td data-colwidth="268" width="268" valign="top" style="word-break: break-all; width: 862.5px;"><p style="text-align: center;"><img data-src="https://mmbiz.qpic.cn/sz_mmbiz_jpg/Gv6JExJQjUV4w04qRwvRIiaRSwvF5lLgnGr7fYib0ricDYiaibiawNwj0VcicFQhx1ib98UwT8KxMfAbah0hrUnvySCpdt9sGwo4iayZ2YEicQaFZ9vyE/640?wx_fmt=jpeg&amp;from=appmsg#imgIndex=8" class="rich_pages wxw-img js_img_placeholder wx_img_placeholder" data-ratio="1.4015518913676042" data-s="300,640" data-type="jpeg" data-w="1031" type="inline" data-imgfileid="504558410" data-aistatus="1" data-original-style="null" data-index="10" src="data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E" _width="677px" style="aspect-ratio: calc(0.713495) / 1; width: 317px !important; height: auto !important;" alt="图片"></p></td><td data-colwidth="268" width="268" valign="top" style="word-break: break-all; width: 862.5px;"><p style="text-align: center;"><img class="rich_pages wxw-img js_img_placeholder wx_img_placeholder" data-aistatus="1" data-imgfileid="504558080" data-ratio="1.368421052631579" data-s="300,640" data-src="https://mmbiz.qpic.cn/sz_mmbiz_jpg/Gv6JExJQjUWRT04uLSbQ1VblTb1nLKDjzUDXD4aI87NBBFCAnRG39AKV9opm5CsQQeH5pgUtwGKtA3tMPRaQv4hU61MYDR3lkYEocA6RCE4/640?wx_fmt=jpeg&amp;from=appmsg#imgIndex=9" data-type="jpeg" data-w="950" data-original-style="null" data-index="11" src="data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E" _width="677px" style="aspect-ratio: calc(0.730769) / 1; width: 317px !important; height: auto !important;" alt="图片"></p></td></tr></tbody></table>