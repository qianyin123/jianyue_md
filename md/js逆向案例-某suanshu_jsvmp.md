> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/81sKkysbWyPeHEtuFT_ynw)

##### 提示！本文章仅供学习交流，严禁用于非法用途，文章如有不当可联系本人删除！

#### 一、常用的js基础知识

① js各种运算符：比如位运算符：`&、|、^、~、<<、>>、>>>` ，算术运算符：`+、-、*、/、%、++、--`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9YkN9fXCFFjqo4ic4IKR9cZDS4eoGiaXgYyOwRZW7L6UBYfiahBDxf5cQjyaicKg9z9yrJ3l6xHRZQFQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9YkN9fXCFFjqo4ic4IKR9cZGHoaxpBf83AY5BAgNJC1sQsicVCuZ1sQEdsMXlyq4Tpm47DNwC0vkWQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

② 三元运算符又称条件运算符`?`：表达式`结果为true`执行`冒号前的`：value = 表达式 ？为true执行：为false执行 ，比如a = 5 > 3 ? “true” : “false”  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTf8KYHcjKBC7m8L3C6OuVPBHYfEZXiah06BEqWenOJ8gUUGmhApc90ow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTd91pcyQ2unJyL7wXd7icHv78XBcaUt4a72Qnfd6wo9XQZ1KY90YdsVQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

③ `String.fromCharCode(num)`：可接受一个指定的 Unicode 值，然后返回一个字符串  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTKLtHujsVoaZQt6micjMbYJYpNo7aB825gxQ8CVpiaRGSUWTCaKNgcOCw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

④ `'str'.charCodeAt(num)`：返回字符串指定位置单个字符串的Unicode编码  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTwyrEcImEHEca4POH8RlyW7G7oqUiaTQicZQlySoAicW25xeWnGOKPa0Fw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

⑤`toString()`：把一个Number对象转换为一个指定进制的字符串  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9YkN9fXCFFjqo4ic4IKR9cZS0fKtWQicCu49akbYlNUXIdaia707m4rqf4lpMO9yGCACwZ8YnGoAkVg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

⑥ `parseInt()`：把一个字符串对象转换为一个指定进制的Number

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9YkN9fXCFFjqo4ic4IKR9cZZs4qCxecP9usjgqz4qpN3avvxWWFkq5FltDDUq0kBibY8Jmf4l2Ve6g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

⑦`slice()`：截取指定位置范围的字符串  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTCI0OnDLzGE130NUL9YTWxabX2icWexmKFNSuGf9icmh8icca0TicJupWsA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 二、常用的谷歌操作

① 谷歌开发者工具打断点，如鼠标选择行号右击，`插桩断点Add logpoint`可以打出日志，如`条件断点Add conditional breakpoint`可以根据输入的条件表达式为true的时候自动debugger住  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTPwpjpGCzLjc7Ne0cTxVGjp4auiaQUkQ0zYo3h6klWffZzDPQMhfsB0g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

② xhr断点监控：根据url请求中的链接参数特征，指定监控  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTUBIZicsaSSiaaGVqjsJ8xJhTg530qiaQynicnzwtJhoibGkxqeXbV8QWyHw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 三、jsvmp的特征

① jsvmp的特征：请求参数signature反爬，加密算法文件为`acrawler.js`以及文件里面典型的提示字符`window)._$jsvmprt("`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTImswRFJtgVHPCBN06XzVgGwFeqPiaYBIlhGfIgyiaQ1xlk4QsdNtK7zQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

② 关于jsvmp的一些了解可以看这3篇文章：https://bbs.pediy.com/thread-261414.htm ， [jsvmp虚拟机保护](https://mp.weixin.qq.com/s?__biz=MzU3MDc0MTY1MA==&mid=2247483878&idx=1&sn=8189762977d777ccac1e056511c3ed0d&scene=21#wechat_redirect) ，https://www.cnblogs.com/luoxiaoer/p/11735768.html

③ 关于jsvmp的解法一般有3种，插桩补环境`，插桩扣逻辑，jsrpc`，当然可能还有其它的自动化等方式可自行研究试试

④ `插桩补环境`的可以看 [逆向OneByOne的公众号](http://mp.weixin.qq.com/s?__biz=MzU5NTcyMDc1Ng==&mid=2247484729&idx=1&sn=6f83dc946b27a875fe421711a794533f&chksm=fe6cea62c91b6374684f596e14babe65015e4d3f46068941ea1f96be694317f7aed8b6d42efe&scene=21#wechat_redirect) 或者是 [爬虫术与道的公众号](http://mp.weixin.qq.com/s?__biz=MzkwMTE3NzI5NA==&mid=2247483839&idx=1&sn=17a8b2b3a2e3a588587c8d19ffba8076&chksm=c0b98b1ef7ce0208d087ba8242c7f611b6976646f3af64e38dea9da1df02db3ce4903f839bfd&scene=21#wechat_redirect)有相关思路，网上也有很多，可随意查找

⑤ `插桩扣逻辑`的可以看这篇文章：[逆向简史的公众号](http://mp.weixin.qq.com/s?__biz=Mzg2NDY4Njc4MQ==&mid=2247484192&idx=1&sn=1db676ecaff6aca791782fefacecf829&chksm=ce64c4f2f9134de47f09e1366134211d46ad9860226f17b2f23724c99407b67f3b4283073ad6&scene=21#wechat_redirect)，文章前半部分讲了补环境的关键点，文章后半部分讲了如何扣代码的关键点，以及关键的算法函数，强烈推荐，看完实际操作一遍，你会豁然开朗；以及这篇文章：[Python爬虫 js逆向的公众号](http://mp.weixin.qq.com/s?__biz=Mzg5NTY3MTc2Mg==&mid=2247483704&idx=1&sn=8e08162f0768fa6a793de5d761eff369&chksm=c00d8d85f77a04939dd5ce0481c8a8ca4426c25c7a1cd037f4568803f479bef2b355b80e6ee5&scene=21#wechat_redirect)

⑥ `jsrpc的逻辑`的可以看这篇文章：[时光python之旅的公众号](https://mp.weixin.qq.com/s?__biz=MzIxNjQ2NzM3MA==&mid=2247484740&idx=1&sn=be10e2180b8064fef08db3239d3eff8d&scene=21#wechat_redirect)

#### 四、jsvmp扣逻辑

① 定位入口：目标参数signature，采用xhr断点定位，发现是由`acrawler.js`这个文件生成的，然后我们插桩日志输出，找出`window.byted_acrawler.sign`函数的传入参数，然后就可以通过补环境的方式生成signature![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTGcXePe8FzwicT5CqWxlfUGazs9PgNKzy7EtkAT5ApiaIAypWoeXvibibXg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

① 补环境和[该篇文章](http://mp.weixin.qq.com/s?__biz=MzU5NTcyMDc1Ng==&mid=2247484729&idx=1&sn=6f83dc946b27a875fe421711a794533f&chksm=fe6cea62c91b6374684f596e14babe65015e4d3f46068941ea1f96be694317f7aed8b6d42efe&scene=21#wechat_redirect)的解决思路差不多，当然视频里举例的sign传入的参数可能不准确，具体的有兴趣的需要自己去插桩测试验证下，本篇文章重点在扣代码逻辑

 已关注  关注  重播  分享   赞    _切换到竖屏全屏__退出全屏_逆向OneByOne已关注分享点赞在看已同步到看一看[写下你的评论](javascript:;)[](javascript:;)分享视频，时长03:45

0/0

00:00/03:45 切换到横屏模式 继续播放进度条，百分之0[播放](javascript:;)00:00/03:4503:45_全屏_

倍速播放中 [0.5倍](javascript:;) [0.75倍](javascript:;) [1.0倍](javascript:;) [1.5倍](javascript:;) [2.0倍](javascript:;) [超清](javascript:;) [高清](javascript:;) [流畅](javascript:;) <video src="http://mpvideo.qpic.cn/0b2ezyaaaaaaqiaoxdaslbrfbtwdadhaaaaa.f10002.mp4?dis_k=c36e5365b63d1e3223acde9856d9c0a3&dis_t=1651820612&vid=wxv_2278593930235641863&format_id=10002&support_redirect=0&mmversion=false" control></video>

继续观看

js逆向案例-某suanshu_jsvmp

原创,js逆向案例-某suanshu_jsvmp逆向OneByOne已关注分享点赞在看已同步到看一看[写下你的评论](javascript:;)

[视频详情](javascript:;)

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTQgtG1ZnsUTdRKRJc7C7uuT1s93TVbAamEturQcnObFBtDJYPXEUEPg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

② 扣代码正式逻辑：如果你看完逆向简史公众号的文章你一定知道signature 是由 9 部分组成的，第一部分是固定参数，剩余的8部分都分别由相应的算法逻辑生成  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTLAuzz75zoRsnYrNBJg1YlQmXAicB2y9MulMaICJ9ib6W2t9BzRhsv36w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

③ 接着重中之重的是打日志断点插桩的位置：分别在arcawler.js的184行和425行加入日志插桩断点，内容如下:

"索引j", j,"索引O", O, " 值：", JSON.stringify(S, function(key, value) {if (value == window) {return undefined} return value})

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTANhb2rHcIYjbiaibal9icFJgbrzS9Va2JO7SUjCkdzfkG8T4cJ1O2z5tg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

④ 然后我们清除网页缓存，刷新网页，会发现大概有`1.5w的插桩日志`打印出来，而通过ctrl+f搜索，signature也在我们打印的日志里面输出了，也就是`j ===24 && O === 40336`这个位置有了signature以及其对应的值`_02B4Z6wo00f010kDMrgAAIDCwkipWBmxc99JAzYAALBd35`

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTNsQxiad8UcaLWibuu3PxlOXgxuHJ7YFO8aT1lHFLF8j67WxzFiaYR17AQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)  

⑤ 我们继续搜索值`_02B4Z6wo00f010kDMrgAAIDCwkipWBmxc99JAzYAALBd35`，会发现signature的最后两位也就是第⑨部分是35，`生成逻辑是`c468ee35`这个字符串的后两位截取生成的，第一部分是固定参数，所以我们先分别搜索其它②~⑧这几个部分生成的位置看看规律

① _02B4Z6wo00f01 ② 0kDMr ③ gAAID ④ Cwkip ⑤ W ⑥ Bmxc9 ⑦ 9JAzY ⑧ AALBd ⑨ 35

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTyES6iaLryMvbJmLyTTbUaSwNMVtiaL2PxP39uQia3lu3UKpTw0iaO65Hlg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

⑥ signature的第8部分，`AALBd`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTPe2cK8t3Dpmj0z9jXdb5wnicPGPF3g7ANqlO8AXLJR3rPt6O1KrQC4g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

⑦ signature的第7部分，`9JAzY`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTnKEgl1agOWjq4K6jyeaYlSy37IY6wQNUtaxsN6lDWgnavVfXFBzKtA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

⑧ signature的第6部分，`Bmxc9`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTyugpxhes06iaP0xTVDAjSZz61dQh6WHoeZPaiakaiaKsEvytQeC8RfQFA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

⑨ signature的第5部分，`W`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTlEJYXvqmacg8x9RqDyOIvMo3aMG4R9KJMM2x7mXM5yh2BiaSicJLutow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

⑩ signature的第4部分，`Cwkip`  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

11、signature的第3部分，`gAAID`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTiaQm6JqMH3OFPLFab3hDUibsUZmRx0xWYjQ1KpDN7amibJ8s0Hq1hcjhw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

12、signature的第2部分，`0kDMr`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTiclzAcyEYxnJMXuf5uiaC4Y8ScmyRvTOIlgQtiaibjDGaoSVia0hvtBqNZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

13、通过前面②~⑧ 部分的图片逻辑，你应该已经发现了每部分都是由单个字母拼接而成，我们拿第2部分`0kDMr`再详细看下这个流程，可以发现第2部分的0kDMr确实一个字母一个字母拼接而成，而每个字母都是由后面的数字转换而成  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhT4Xe7Zq7FYsprdI2iapeibQvYVnhCs59WBFEoyL4nMHQHZQ5vIApFBVJA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

14、也就是2部分`0kDMr`的生成逻辑就是数字通过`fromCharCode()`转换成字母然后再拼接，那其实我们只要继续研究这个[48]、[107]、[68]、[77]、[114]这些数字是怎么生成的即可，这个你在[逆向简史的公众号](https://mp.weixin.qq.com/s?__biz=Mzg2NDY4Njc4MQ==&mid=2247484192&idx=1&sn=1db676ecaff6aca791782fefacecf829&scene=21#wechat_redirect)里面也可以找到原因  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhTC9pe7BmG4CKxEZFt070RDqqOsW71SY48SGRIsib0XkufdEwZHiaJA92A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

15、先看下结果，[48]、[107]、[68]、[77]、[114]这些数字的生成方式如下，它们都共用了35394057981102这个14位长度的数字，并且按一定的运算方式顺序获得  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpictALFBGiczZs54Su3qJpjhT4gCJeLLxrnTauxeHEvnvtMxHKByT36j4xibh2wFzRKV5zhdZLxDVrYA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

上面的9部分signature的每个字母拼接都有可能由长数字经过一定的运算转换而来，所以我们要研究`35394057981102(长数字)`是怎么生成的，以及对应的每个字母的运算方式，比如哪些数字是固定的，又该用哪种运算符，这里建议直接研究[逆向简史的公众号](https://mp.weixin.qq.com/s?__biz=Mzg2NDY4Njc4MQ==&mid=2247484192&idx=1&sn=1db676ecaff6aca791782fefacecf829&scene=21#wechat_redirect)的付费文章部分（`真的超牛，无比敬佩`），看完后如果还不会再选择是否看本付费区的进一步讲解