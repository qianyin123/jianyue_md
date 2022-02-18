> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/VaxSQqKnPQn4gsvrn2O_MQ)

##### 提示！本文章仅供学习交流，严禁用于非法用途，文章如有不当可联系本人删除！

#### 一、rs5特点

① [关于4代的介绍可以看这篇文章](https://mp.weixin.qq.com/s?__biz=MzU5NTcyMDc1Ng==&mid=2247484428&idx=1&sn=ea6b43a2baf5994a2898482f4e48d01c&scene=21#wechat_redirect)，有些流程和4代是差不多的本文章就省略介绍了

② 服务器响应状态码202或者412返回第一个cookie_S（或者cookie_O）；`然后js混淆生成了第二个cookie_T（或者cookie_P），`只有携带有效的cookie_T（或者cookie_P）才能正确请求页面状态码才是200

③ 如何区分是几代版本，看cookie_T（或者cookie_P）的第一个数字，绝大多数通过该数字就可以确定是几代；cookie_T（或者cookie_P），`其区别含义P代表(https)、T代表(http)其实是区别协议的`，常见的有：gOEoYMGvg36yT、neCYtZEjo8GmP、9CKCOkIaqzqET、DLjfPow8PDr4T；  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdFyVK5icbIgo2yKichJTg42kxTA5EIRS62eh8X8CsqLbU72sfboiaykBow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

④ 加载vm的1万多行代码`入口特性`有3种，如图4代、旧版5代、新版5代，[更多详细版本区分介绍看这篇文章](https://mp.weixin.qq.com/s?__biz=MzU1ODA2Njk0OQ==&mid=2247484290&idx=1&sn=4b5c4a88aa439fb859e36eaf0ae53815&scene=21#wechat_redirect)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdF1MtvNthH2JaPQico50Fs5zXzCVU6R60jzF9Mt8dTkn1TqJv1sv7kdQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdkxjWLWyZX8aiaYFbDKqBnB6OrwIibiacIJPfn7YTlp6iaIqibJAFhDP6d9w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd9AfPzh1jB14pksF63T8HUWibXxa9yUh4jwRMicxRer2EuQq6icCStncJg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 二、cookie正式逻辑分析

① 开始大致是图片这么个流程，先定位到call，然后进入vm代码里面分析

 已关注  关注  重播  分享   赞    _切换到竖屏全屏_ _退出全屏_ 逆向OneByOne 已关注 分享 点赞 在看 已同步到看一看[写下你的评论](javascript:;) [](javascript:;) 分享视频  ，时长 00:20

0 / 0

00:00 / 00:20 切换到横屏模式 继续播放 原创 , js逆向案例-某rs5 逆向OneByOne 已关注 分享 点赞 在看 已同步到看一看[写下你的评论](javascript:;)  进度条，百分之0  [播放](javascript:;) 00:00/00:20  00:20   _全屏_

倍速播放中 [0.5倍](javascript:;) [0.75倍](javascript:;) [1.0倍](javascript:;) [1.5倍](javascript:;) [2.0倍](javascript:;) [超清](javascript:;) [高清](javascript:;) [流畅](javascript:;) <video src="http://mpvideo.qpic.cn/0bc33uacqaaaiyagh6qkrnrfbxodfdoqakaa.f10002.mp4?dis_k=9be329311e8e27163964fe8bb215b540&dis_t=1645148468&vid=wxv_2269111966050975758&format_id=10002&support_redirect=0&mmversion=false" control></video>

继续观看

js逆向案例-某rs5

[视频详情](javascript:;)

② cookie生成起始位置，如搜索`(772, 1`

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdyflWCtNdBrj1tQ4o29a0licYL3HUn2vxfrUyjQEg80TMUC8TntLYIPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

③ cookie结束位置，如搜索`'\\b'`

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdbG1oia12TKsEQT2RQO8A17eb81dXlB80qBqhAibbhCTNTXeQ2ASqGgew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

④ vm当中128位数组生成起始位置生成位置搜索`(128)`  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdJdUm48lymn6gf5ZPuSctIR5wtmNRSJZOLMOGEofn0YvhyrGKqVKbQg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

⑤ vm当中128位数组结束位置，结束位置搜索`([]`  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdZawQRCZE9Knl0UTfK8reFWuu3bknpqNXMPlwmbNJWVYMfic5AConA1w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

⑥ 以下流程你可以理解是带你单步调试扣代码的过程，vm思路扣代码分析前可以先[看看这篇文章](https://mp.weixin.qq.com/s?__biz=MzkzMTE3ODA5NQ==&mid=2247483798&idx=1&sn=fd82f36e3f7b683130d01db2ce48194a&scene=21#wechat_redirect) ；cookie思路可以看本文档末尾推荐的一篇文章

1、先打上script断点再刷新网页，首先会先加载外链的js文件（暂且称之为version.js），此处可能稍卡等一会儿就好，并将js文件的内容赋值给全局变量`$_ts`这个对象  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdqdAkakAazuCT4ibTG8Lf7DnCpGcoBDDKppMXNA7WtqVGV6fCFEdkGEQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2、继续跟着script断点走，会跳到源码的一段自执行函数js这里，暂且称之为main.js  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd6ULKOVibzcIxH7vXSb4PpyZKDMoANtnpTFaqmfg0vuRg5Ksvh016vGw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3、main.js执行到`call`的位置，只做了两件事：`$_ts`对象有了很多属性，`version.js`文件被解密成VM可执行的js代码，进入call函数对于已解密到vm的js代码，我们简称其为`vm.js`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdic8vMzqADZESx6XSowasBdXZhMY6ov0vlaEM9w74tJWT3cq9AlCvYwg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdWUdeXGNMnuEE9Lgj17ibRfnH61b9JDu5KPUzQlrIehV7gHKjwquzPiaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

4、接着通过hook_cookie + 堆栈回溯定位我们知道生成cookie的入口是vm代码里的`_$6p(); ===》 _$co(772, 1) ===》 _$co(742, _$TI, false);`这个位置，到742的控制流就开始生成cookie的逻辑了  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdpueAfkiatLB34NDTNHrSYpymIleLgtAXYzfBCpN789rF8OZOBxbv0pg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdO1sbhzsNcibcsG1aEIg2zvqrs1dkFPO9ONr8QbyJS5qtjcoicg5r07NQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdhydvF7hfNBXjWJRzZONnBbMPSicS257Y23aicwTSaFib50gowiaIicPFtQA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

5、所以我们下次直接搜索`(772, 1)`去找cookie的入口生成逻辑（`注意重点`：从此刻开始你需要保存一份静态的文件用fiddler替换或者overrides替换开始静态分析，防止每次刷新函数变量名都是动态的不好分析，也不好扣）  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdvRxstrVeKNEZ5bMQfiaXSvRxT3mbvXTpkX0AVBH27GjS1tTO6JiaXicUg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdeYpM3MWK5g5Ptn45ln8YEQhiaSXErGDPn7e3icU9AgML7HD3X9fQw7iaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdhydvF7hfNBXjWJRzZONnBbMPSicS257Y23aicwTSaFib50gowiaIicPFtQA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

6、742控制流的第一步代码是跳到这里，首先生成了一个时间戳差值，在后面的128位数组里会转成8位数组push进去  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdu1HPU8OUgVHLv6X2nT6zSRtItaICeZ3qwibicrvbjr9CLTIex1ZtoibsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

7、742控制流的第二步代码是跳到这里，将`_$$d`变量赋给了另一个变量`_$$E`，(中间的_$Wp变量只是用来判断改变控制流走哪个if ~else的，接下来好多流程都会用到类似的判断改变控制流方向，此处直接忽略)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdEq5QjFBuxgTfWOHpZ0sl5gGMqqYNC0WScqqibnhDicCtW0ibr1auvlOeA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdCDgGuPMwujh2QEic9TvjyPIlAJUB9H3g3o2zyOXv755KicO7TUPBKxEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

8、742控制流的第三步代码是跳到这里，进入279的新的控制流，`核心主逻辑在279控制流里`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdrlY9iaD0ibubAUCkpxwoZkw3hE3xt93cxkeWicOEBfrx6f9YicyqJfFxbQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

9、279控制流第一步，先定义了一个局部变量如下  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd3yAaMRe3UYV3GKEntwY1RBnN85fs2eviadyHicujJdgP3grUh2NIDmgw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

10、279控制流第二步代码是跳到了这里 `_$co(157);`，是一些自动化工具特征检测（msCrypto、ActiveXObject、_Selenium_IDE_Recorder,_selenium,callSelenium）等，如果有的话，会有3个全局变量会被改变，默认不发生任何改变，所以这里不用进控制流调试了  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdPXvEJ0RGpvqoZqr2xWzZNia9OkiawNtERPxVl0NNPAuiabVpSjWSotRgA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

11、279控制流第三步代码是跳到了这里`_$Sc(4, _$t2);` 会有个全局变量会改变，在稍后的128位数组里会push进去，其中_$t2这个如果是true的话会改变25165824，默认undefined  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdjyWzw6jZk9BOYiatrqPgjyjIfHUfeNlRia9VowuSneOeFHzj7QdasXGw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdDQiao7p8XZj6YuybtuluNgP3Mnf93V2QvxXLZYWibDsDvO6shtEKVYXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd6BAK9x7Hibguy22y1temiane8xOe44QJDNxwomOjtZsbuhOziaFg6NXXQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

12、279控制流第四步代码是跳到了这里，生成了`_$qC，一个128位空数组`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibddgnJJrGwUyViaPOrjlsyvYG0fnL8O5G3zZSGIHaJbwpxOaBmJ8PTwkw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

13、279控制流第五步代码是跳到了这里，`非常重要，695的控制流将$_ts的5个值生成了20位数组，其中有4个值的顺序非常重要需要映射`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdgBbJPHt2jeDu9j2YTPH0QfdEZp9b4fibJXBMHL2GuAoP7qqYXVLykIg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdt7DPicTa77sWslBpx8G3SHbFCS0u3wbn7pr2Y0mAko2rQ9dmNwySrqg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

14、695控制流第一步代码将`$_ts`的第一个值生成16位数组，第二步代码将16位数组和4个值连接起来  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdMpKVRhzqDibIpMxicupNEbLbAJg7TckWH7uyjfmKuBO8hfnobIovw5ow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdg0Cz29QHlfKeH7WWpvOfibH85GGdNMoVra8HEGiae60FkEJKlyIfk4fw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

15、279控制流第六步代码是跳到了这里，对前面时间戳差值转成了8位数组  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdFciaHBNyWrR0ic956rWzNTOjtcdBoCLPQGIAcxC6Q12AVoVvkfbaY7jg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdOwoujciaVBz61emTY6rUBQJoAiaibtGz4ibhyKFQOgUlTONtUibNa392UeQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

16、279控制流第七、八、九、十步代码是跳到了这里，`_$qC`128位空数组已经push了6个进去了  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd9WvOHyKKibEFRFNnpwiacnYlee54NAzicfMoPAyfDMZKFMMxcPrnR7kyA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdB8qo8SbctWtgtl7xFpGWPibNMUEkkDia1gJoHYCMaxRlJ5yapaCZ63IA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdRgaOIafFpzaco7jlRF4ibepFeRdR41J1ebJx6KPLibEFNViadub3y1AIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdT2Z8Ot3wSHviaNu4Ypk0NDKEujU1RZ1Pht6w1qGZgoSaCS8WEW7Py1Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdbkKqSAITmeWdhChEibskq2bcGWIrWEEDwROXJoTkp4dJUPXfGOgKyDg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

17、279控制流第十一步，push了一个`_$34`的四位数组，这个4位数组是在初始化流程中取_$ts其中一个值生成的数组  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdK2Szn12w6LBicSFGlr1t7icMvkMI6ZzqPbAxBDtcAKicWQtLpaN9VGW6A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdiaJNPtRG1ickLJ9ibhNdzAgfcuDHYvcDibSSWrSDthfgAPx6M9Udia2ljjQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdJAnMbVrtJJTh1x8HiciavnGj3Sm8SJr9jHfnSINHkjKWft0zyfHQootA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

18、279控制流第十二、十三步，push了一个`_$CL`的八位数组，这个8位数组暂且可以写死；然后push了个0也先写死  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdyFCdt1tyu0Ml4kE06rZKCGpkCUzd5DCibhxVISUEz6scciaf6At2WLnA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdzsR8DWF7FrFWRY4hwTv4hfxgs9g0lzPPba0FciaPHXJ5kwZGRia5lwrg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

19、279控制流第十四、十五、十六步，然后push了3代表是Chrome；然后push了一个8位数组（这里用到了一个`_$bJ=25165824`，如果前面selenium等被检测到了，这个值是会发生改变的）；然后push了14固定值  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdRLlOypYbk96V7kX1g03SRCRsllOmcQMRQPO0CZiaEHUvIE4tfPwOFfQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdEwW9wtRSo6P9LMyEfFf3hgIFOwZLVjVQiacrmWBdV2mgkGmiboCXgmTw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdJgbjcnoHawicmqFDCrJdibXsAdBujVXec7wTwNDCRObUMh6yDYnicXgGw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdwJpu4rE38Ilanibr7V5ENibSzjA7kaqjgicuyiczYIDbeCBJKzAahO1udg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

20、279控制流第十八步，索引`_$1g先占位`，push了一个undefined，后面会有值覆盖这个undefined，目前`_$qC`128位数组已经push了13个了  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdDDuMyREwEHdGicl4eHicFqEumxjenTBxllucDtsJhVMuoLNrX6brKuKQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd4cTZAiarshs0VmGp5aOUka61picSjyyVp8MJS6vaPU1icrhicLnTnicJweg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdic5JibkXeRd4rIeqx5Km9u5so0liciaoicHicPzsyElOgtE4AYZuSVibfVTMg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

21、279控制流第十九步，`_$co(58)`这个控制流会返回与`localStorage`相关的值，如果有值的话，则会push一个20位数组，本次未push  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdS4pUlFiabicOqRLkHKJtbWZFXClWUxSW2xTlxD0SeicpjWD3ibyoibOJV3A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdEvOYnWfPehMpVlbOibjPTexJctnuUQ23iaqQxb69yqu2Kt4J5ibc9bicwQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdu6g2bGAib79k9j3F9nbgSes7vhWc8SWsF8MY01GIICENf3AibjErjp2w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

22、279控制流第二十步，`_$co(247, _$_j[580]);`这个控制流会返回`localStorage["$_f0"]`的值，如果有值的话，则会push一个20位数组，本次未push  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd2v2aKT1uYOaJnmRN9ldQ5Hia2bPY3GzmeqpdtP3ibp2ibXSiasDOo6NguA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

23、279控制流第二十一步，`_$co(247, _$_j[154]);`这个控制流会返回`localStorage["$_fh0"]`的值，如果有值的话，则会push一个20位数组，本次未push  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdzs8O54d46CEFAHcjVlibvPJ8KWvsqib3TLkhOXyMWyiboq4fxLVickwF3Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

24、279控制流第二十二步，`_$co(247, _$_j[636]);`这个控制流会返回`localStorage["$_f1"]`的值，如果有值的话，则会push一个20位数组，本次未push  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdSGic32LOFUA3X6DhwH2IPJDibIVVHTsPbImW8OXruaBsbnSG2eF66DNA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

25、279控制流第二十三步，`_$Jk`是网络状态对应的值`NetworkInformation.type`，本次返回是0，然后`_$ty`进行或运算  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdr1XBJiaicm5zia03iaicS4Uy2caBGKM7DvEaCRbUvmxJuOhgRkrUAY1k1zQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdFibHGQOFXrsDUI1xdsMcmu26cVhc7INk6URMibupTd4xicIoicqWNU4Iog/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd0RoWwBh9abuIbTSB4zIaJWahaYia8gEo975POibm0zJJDk94H0hhYv1w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdq4eOUFcq4jQIlY8M70ltR4TfoOxWzEkt091NG30aW8UpwpNvcpWAmw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

26、279控制流又来一波判断，返回`localStorage`相关值，一如既往的undefined，如果有值的话，则会push一个20位数组，本次未push  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdmSujxkaacjD8gAlDxpoibUTntVnQ4V7VFWV6GPda5Vl8y7rkNk08iaZw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdNYGaqCUmVLud4yySsKY1kTFry8x9aMUfLOdX0QEVib2Xng80Q6ibOKSQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdO3aV9XEgI8HnXibl8uN6hOUicFaVEEar4PCia7hLeoGb7h2M4rTvTdsjQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

27、279控制流第二十四步，push一个固定值4，是在初始化777的控制流生成的，然后`_$ty`进行或运算  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd7sZC8bzylF2sAB6j1OfWwsKt7EkTGAwlBNBhgjDYfO2M8jV23km7rA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd764t2Y6IkGmprPUM8CIGwLPq7Aq8ibQZLs8ElkpialoqRJGUEWxlXNJA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)==》![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdOIYFeCicKPShcDrwg4MtMQnsibPDO6ylePRGyJsmJgTeyFibTZNnjVic2g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdh3ARxHGAZFua3AKCia69RhK0vKGUQb64sIP9McuKYXIzOO9Ff8hk77Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdvsicgkl9qLZiawhFWCrh6A5ZqH6BONDQYUiasibsf1PzL9aX3tFIXTe4Jw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

28、279控制流第二十五步，`_$ty`进行或运算，然后push了一个`'https:443'.length`，然后同时将这个协议端口返回9位数组再次push  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdeUiculTKfKHOZFMF2ichoknowBg71Rn6v8IibAwkTB8DBTZ38yCa7Px3w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdKQSMX6W3cyicXWVhLQpeKaEUibaX7ZCbKWKKPbTp7XcbhVo7I416YpXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdOQHKyw4ficTQYJpRgjTc6UJe9iaFHGqtwRdjFAHU1rXada9nt9OgN5ibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd4axHcyhs4pPH6L7CqGJeOU1W81jEhTe2sy3qU2FfoicxKLWKtZ14KIA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

29、279控制流第二十五步，`_$1g先占位`的索引，在这里将多次或运算的`_$ty`给转成4位数组补上了，至此128位数组push完了，17位长度（localStorage有值的话会更长）  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibduO138nia5LLfHa1l2E1ASIgMZupkW0bfKVBuy0cRGibm048ycWdZUpBw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdgX1t7hwfevTaxCa3MuXIbbddtAqNOVicwIYq9VBWkMKDxRkr5jaTEnQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdkpuszQpQz6aOyj1Szq3zbIvxzhwCHdaymdJB6yU1PccIewCblPgovw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

30、279控制流第二十五步，保留截取128位数组前17位已经赋值的  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdA2NoorPf46ib7t4mmEjj1KgFfWPQCSCjiarxianATC9tRN09ScqtqYAGQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdBAwIicmwib9paaP8n6TkOuyJdRL9dg1JUR6yBFm8DQQvDDe5d0aSiaOZg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

31、279控制流第二十六步返回128位数组最终17位全部合并拼接的数组，至此`279控制流对128位数组的操作结束，继续走742的控制流逻辑`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdGicsAfOC9dibuULZic7ES3GUP9eE4EmnGeeia2EYHtPQvB12Q1ibJHYNiaAQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

32、742控制流当时入279控制流的入口，生成的结果给了`_$ty`![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdrlY9iaD0ibubAUCkpxwoZkw3hE3xt93cxkeWicOEBfrx6f9YicyqJfFxbQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

33、742控制流第四步代码，生成了一个32位数组  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdKvf7B8F9GKOryGxhGG6LtSuejWY2X91BSj4eicphphxfMoZlSVPJmJQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

34、742控制流第五步代码，  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdiac6WIxrmE9Ydbtj6PUbpIxhZb3e6U3kAyAm70zMt2mGhHT9iaCD1teQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

35、742控制流第六步代码，那个由17位合并的数组现在又连接了一个_$ts的值，计算出一个大数值，然后大数值再转成4位数组再连接到了17位合并的数组上  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdzB0B9R7LiaSl85cQDTkB43e0w0A9pJKDfVUgCaETIPcCU08b16WVOdg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd01ohXZVD1q75IGYKg2NKJ20kBm35HouQKtpwQiaQiaqYpIKib55c06Jhw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdxmITwdfk69z0SrHT6xWTUbEb9nHGEzN9juHaf4ibgJ1YOiboDYoLKiaAQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

36、742控制流第七步代码，对时间戳，时间戳差值进行了转数组操做，并转成了cookie的前缀字符串  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdRPtP0tfcbXkL8hXcibQayWYav5bvsjzgm2s7gia0IfkGPAbkrgJtYJpw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd0iapia4CzUTQiaTBRqdl0mfaTPl3TSQlnoOwVvotdiaYYIxzkWm8dic1cnQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdth5FZtgyqVxYo5xtXicUlfYvSojmvia3RU6icgTtWibjjBwAXVLdlsJTiaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdETjA3zK4kpTltZagTdrnNzpBE2NmjmpGR8lUicxIax837FAAI1p1Hbw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdicb6Zo0Tjnrd3SAHBYdSpMm5UttbRLDQghmTqeWk6gTlfQ6jcA2x26w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdnMz12VmicUrAdyKniaMNhb70YURu07DPChzXibtoQtlz0ks0Ve1pYycXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdqFwYOVR2gxxibXjbawfHQz2o9MHqQVnyJswVJnkwicibyuCWxKbSiajIRA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

37、742控制流第八步代码，`生成了cookie`，将即 `5 + 时间戳数组字符串 + （128位数组合并的 + 32位数组合并的）转成的数组字符串`，至此742控制流已结束，`回到772的控制流`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibd2WN7Z07mXJII9UgKUdvpqHsTCPvyIv1b5wZHwicOdMz1B79BRaWxDhw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

38、772控制流进入742控制流的入口，生成了`_$$d即cookie`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdhydvF7hfNBXjWJRzZONnBbMPSicS257Y23aicwTSaFib50gowiaIicPFtQA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

39、772控制流接着把_$$d即cookie赋给了document.cookie，至此（772,1）控制流结束  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdnxFDIeUCj33M1EPicGutian54fIHGbiaefIa2XXmIcfrKBTZyw6kibpzRg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdWQdWBiaUqj1Xv6yzzWCA4JOcgmXqrrm9KSEPz8Po0oicK7watvAqf2oQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdCT6UyZjjoTJFicBW0H0KTWHMTh20LiaicAMyN13Vlc2Y5ibibMyAl6uFP5g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

40、最后扣代码逻辑流程大概如下  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdN63rQRHEaMllwnDesQWLZmB22hn7nn9CTdKUSlGgianXTJ0AN6cmAlA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 三、cookie生成后的操作

1、`_$6p()`函数执行生成cookie后，剩余2个函数然后给一些全局对象绑定函数，然后就`返回到cookie到前面的main.js了`![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdr6YDkaomicAdeAmHodicqIpkNF6ylLCwOn0XCk6cRn8ATN6hWThibVyFA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdDzgsbs7jmHxyCTwCX4uQNJzX36O8CHCDXcMYBNlxWpkgqGic0eVNs6Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2、其中`_$6p()`函数生成完cookie后，还给`_$ts`的属性绑定了一个函数；然后就是修改xhr对象了，如图`xhr.open`以被改成另一个函数；修改xhr对象后就是改`Storage.$_nd`值，这个参数在后缀生成可能用到  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdxQNLuMoSEZcFKiaSJ9USjwX3xKbxfWOmW1lB2UOGgWGyicSHQR9nichaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdFhTxVP1wVlv6o8cicgSbI1CACQM7AuQBQ1H2yMdZoCcmAO1rGtnGWCg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdMzia2My9MXEQOlwGZK5uyRGTlxic7vvSeQf8ptLcgSoibxmdzqcmkNoow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

#### 四、后缀生成分析

1、后缀生成推荐先看看本文档末尾推荐的两篇文章，首先确定哪个请求以及什么后缀，如这里search请求，两个后缀参数，其实和4代后缀一样都是xhr.open被修改了函数  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdSeYDE9icLRZkVhibCTgwKRo4btgCaoxar4FnvEia4vxYQ71cziaPn5uBRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 已关注  关注  重播  分享   赞    _切换到竖屏全屏_ _退出全屏_ 逆向OneByOne 已关注 分享 点赞 在看 已同步到看一看[写下你的评论](javascript:;) [](javascript:;) 分享视频  ，时长 02:20

0 / 0

00:00 / 02:20 切换到横屏模式 继续播放 原创 , js逆向案例-某rs5 逆向OneByOne 已关注 分享 点赞 在看 已同步到看一看[写下你的评论](javascript:;)  进度条，百分之0  [播放](javascript:;) 00:00/02:20  02:20   _全屏_

倍速播放中 [0.5倍](javascript:;) [0.75倍](javascript:;) [1.0倍](javascript:;) [1.5倍](javascript:;) [2.0倍](javascript:;) [超清](javascript:;) [高清](javascript:;) [流畅](javascript:;) <video src="http://mpvideo.qpic.cn/0bc3cmacqaaaoeag3oykqnrfae6dfajqakaa.f10002.mp4?dis_k=8612cb1aedbf9647a49150f8e8326aa5&dis_t=1645148468&vid=wxv_2269085234023530509&format_id=10002&support_redirect=0&mmversion=false" control></video>

继续观看

js逆向案例-某rs5

[视频详情](javascript:;)

2、在页面已经加载完成，打开控制台会跳出无限debugger，然后never pause here过掉无限debugger，然后xhr下断点，然后点击搜索按钮触发xhr请求，则会自动断到如下页面，然后开始分析  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibde9geVIxibLeuZibOMoNiaWWatQlYPhXTF0Ep6nTrNUcssictnVsJXIo2gg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3、`_$9v`函数是生成后缀的具体逻辑部分，先对url参数转数组，然后生成后缀hKHnQfLv，再然后生成后缀8X7Yi61c  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdrTZhMEwZGWTURNp47s51I24YdMo5CialapVR9Df80ibdYuL932G2vohA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

4、最后后缀逻辑流程大概如下  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp9icvfErnDTvaxca4xicvOtibdIQbbgQyZk3FBd9R2a5ORibD1ofzErOB85ia3ibicYLSWVm5ugeeETq7uiag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### 五、本案例类似好文章分享

1、通过扣函数推荐以下篇文章

① Cookie思路详细分析：https://blog.csdn.net/qq_35491275/article/details/117390671?spm=1001.2014.3001.5501

② 后缀查找详细思路：https://blog.csdn.net/qq_35491275/article/details/117307069?spm=1001.2014.3001.5501

③ 后缀生成思路：https://blog.csdn.net/weixin_44772112/article/details/122507622?spm=1001.2014.3001.5502

④ [VM代码加密流程分析](https://mp.weixin.qq.com/s?__biz=MzkzMTE3ODA5NQ==&mid=2247483798&idx=1&sn=fd82f36e3f7b683130d01db2ce48194a&scene=21#wechat_redirect)

⑤ [指纹位置生成分析](https://mp.weixin.qq.com/s?__biz=MzkwMTE3NzI5NA==&mid=2247483786&idx=1&sn=3956ec90f707bc42c7ee6675a4947187&scene=21#wechat_redirect)