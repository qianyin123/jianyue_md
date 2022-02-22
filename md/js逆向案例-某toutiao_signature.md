> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/uJrWHTGW1GhHEEqW_xUMgg)

##### 提示！本文章仅供学习交流，严禁用于非法用途，文章如有不当可联系本人删除！

#### 一、jsvmp代码参数特征

① 案例网址是某tou tiao，向下滚动加载页面时的请求参数反爬signature，该加密的结果在`acrawler.js`文件里，该文件js代码有个明显特征`window)._$jsvmprt("`，本案例采用补环境的方法，插桩扣代码待下一篇文章发布  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rpib9Sm6ZpfGJOFra8iayD8akqNBfLn9nRryrz1uxo1T3ZBtKxkKU57FTWRtW3Mt8AQLzJRSs08uic2NQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

#### 二、signature生成逻辑分析

1、定位入口：采用搜索关键词`signature`定位，然后下断点，继续加载页面  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

2、进入S函数，在return o即signature，经分析得最后一句就是`var o = window.byted_acrawler.sign.call(n, i)` , 也就是signature是window.byted_acrawler.sign函数生成的  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

3、继续下一步调试，进入到`window.byted_acrawler.sign`函数，此时进入到一个新的js文件里了`acrawler.js`（只有650行），传参如图，即可生成结果  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

4、本次采用复制全部js代码通过`补环境的方法`来生成signature，而非局部扣函数补函数的形式；所以我们将`acrawler.js`文件全部复制下来，如下，该文件内容放在该网站console界面可以直接运行出结果，那我们直接在本地里面再试试看  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

5、鉴于js的第一句判断了window是否=undefined，如果=undefined则glb=global(node环境global是全局变量)，否则glb=window(浏览器window是全局变量)，所以我们先补个window对象（有两种方法，`①本地js先用jsdom补充一个window，②window=global`），本次采用window=global的方法；  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

6、小插曲知识点：`日志断点又称插桩`，鼠标右击行号选择Add logpoint，可以在console界面输出A变量值以及S[R][A]值；此处可以补很多东西，但是我们先看报错再后面挑着补

 已关注  关注  重播  分享   赞    _切换到竖屏全屏_ _退出全屏_ 逆向OneByOne 已关注 分享 点赞 在看 已同步到看一看[写下你的评论](javascript:;) [](javascript:;) 分享视频  ，时长 00:25

0 / 0

00:00 / 00:25 切换到横屏模式 继续播放 原创 , js逆向案例-某toutiao_signature 逆向OneByOne 已关注 分享 点赞 在看 已同步到看一看[写下你的评论](javascript:;)  进度条，百分之0  [播放](javascript:;) 00:00/00:25  00:25   _全屏_

倍速播放中 [0.5倍](javascript:;) [0.75倍](javascript:;) [1.0倍](javascript:;) [1.5倍](javascript:;) [2.0倍](javascript:;) [超清](javascript:;) [高清](javascript:;) [流畅](javascript:;) <video src="http://mpvideo.qpic.cn/0bc3oeadiaaalmacydistfrfa4odgryqanaa.f10002.mp4?dis_k=282f89c3dcb0f0a5a6fc4639552da8c9&dis_t=1645511669&vid=wxv_2277687124479082497&format_id=10002&support_redirect=0&mmversion=false" control></video>

继续观看

js逆向案例-某toutiao_signature

[视频详情](javascript:;)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

7、小插曲知识点：`条件断点Add conditional breakpoint`，当A变量等于referrer时会自动debugger住，所以下面本地运行调试js报错缺部分参数，可以尝试到281行打条件断点查看结果  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

8、我们继续步骤5的逻辑，第一次运行报错如下，补环境 `document.referrer` ，右击打上条件断点，然后刷新网页  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

9、第二次运行报错如下，缺少`sign`这个函数  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

10、第二次运行报错原因分析，js文件检测了是否是node环境，如`exports`只在node环境下存在，但是浏览器是undefined，所以我们直接把`"undefined" != typeof exports ? exports : void 0` 替换成浏览器输出的结果`undefined`  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

11、第三次运行报错是`href`，调试运行发现需要补`href : 'http...................`  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

 已关注  关注  重播  分享   赞    _切换到竖屏全屏_ _退出全屏_ 逆向OneByOne 已关注 分享 点赞 在看 已同步到看一看[写下你的评论](javascript:;) [](javascript:;) 分享视频  ，时长 00:27

0 / 0

00:00 / 00:27 切换到横屏模式 继续播放 原创 , js逆向案例-某toutiao_signature 逆向OneByOne 已关注 分享 点赞 在看 已同步到看一看[写下你的评论](javascript:;)  进度条，百分之0  [播放](javascript:;) 00:00/00:27  00:27   _全屏_

倍速播放中 [0.5倍](javascript:;) [0.75倍](javascript:;) [1.0倍](javascript:;) [1.5倍](javascript:;) [2.0倍](javascript:;) [超清](javascript:;) [高清](javascript:;) [流畅](javascript:;) <video src="http://mpvideo.qpic.cn/0bc3siaakaaa24abagishrrfbewdawjaabia.f10002.mp4?dis_k=008db74eef52e6975fbb07b512a5ec80&dis_t=1645511669&vid=wxv_2277674688803930114&format_id=10002&support_redirect=0&mmversion=false" control></video>

继续观看

js逆向案例-某toutiao_signature

[视频详情](javascript:;)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

12、第四次运行报错是`length`，调试运行发现需要补`protocol: 'https:'`  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

13、第五次运行报错是`userAgent`，补useragent  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

14、第六次运行，正常了，生成了一个较短的signature，由于检测不严格，所以这个短的也能用  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

15、据前辈介绍signature长短和cookie的长度有关，所以打上条件断点，然后我们再补个cookie，而且document.cookie的位置得往后放，否则可能被删除清空什么的；还有signature如果不成功的话，有可能就是此处281行出了问题，可以对比的继续补环境  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

16、最终的代码逻辑结构![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

#### 三、本案例类似好文章分享

① [某系signature](https://mp.weixin.qq.com/s?__biz=MzUxMjU3ODc1MA==&mid=2247484172&idx=1&sn=bbc018d2fefe9be2e988534aa8f7e166&scene=21#wechat_redirect)

② [某tl的jsvmp逻辑层算法分析‍](https://mp.weixin.qq.com/s?__biz=Mzg2NDY4Njc4MQ==&mid=2247483732&idx=1&sn=fc6c9d4729e49a23aa11ff130f7d84b9&scene=21#wechat_redirect)  

③ https://blog.csdn.net/qq_43454410/article/details/120946279?spm=1001.2014.3001.5501

④ [某新闻signature](https://mp.weixin.qq.com/s?__biz=MzIwNDI1NjUxMg==&mid=2651266972&idx=1&sn=52703f1c62c133a23d546a6d5755a545&scene=21#wechat_redirect)

⑤ 如果觉得文章还不错的话，记得关注支持一下哦~

 ![](http://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9wugYc9YTEUVxnWCQc4iaWCDYPprhWtJvTvG0nUJbsl8XVWSMLW0DC1XY6up4AyHnEZK0MJ9ubZPQ/0?wx_fmt=png) ** 逆向OneByOne ** 逆向案例小笔记 5篇原创内容   公众号