> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/FN7gJ0NTsA-9mrcbtoJbEQ)

前言
--

*   本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，如有侵权，请联系作者立即删除！！！(我这种垃圾分析估计也不会联系我了)
    
*   已经有好久没更新了，最近太忙了（啊呸，就是太菜了，没东西来写了）
    
      
    
*   目前走的另一套方案，已有成品，断断续续的能并发抢购（主要是几天升级一次风控，生产队的驴都不敢这么搞）
    
      
    
*   如有理解错误的地方，还请指出，以免导致以讹传讹的局面。  
    

网站
--

aHR0cHM6Ly93d3cudGkuY29tLw==

  

个人心得
----

对于js逆向，个人所知的就两种：还原算法和补环境。

*   还原算法
    
    优点：直接，能跳过大部分的指纹检测等；
    
    缺点：复杂（对于那种使用原生加密或者加密流程不穿插的情况当我没说）
    
      
    
*   补环境（顾名思义就是模拟一个它在浏览器中所能运行需要的环境）
    
    优点：忽略几乎大部分的魔改算法，跳过上面所说的还原算法的缺点等；
    
    缺点：对浏览器的属性了解度深，更新频率快（通常他动你就要跟着动）等。
    
      
    

  

请求流程分析

* * *

  

对于Akamai是什么本人就不介绍了，网上的大把文章分分钟能打死我，直接分析流程吧，暂时就分析到第一次验证加密参数的地方吧，后面的根据每个人不同业务路线就不一样了。

    1.先看一下整体：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTYpQpzrpYqDkqmE56HTueQicd7wibEOWbPrRlph8TNXvJSHaxyWB30yh3w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

目前就关注两个请求，上图中的1和2。

1.  请求1中会返回三个东西（abck，bm_sz，下一次请求的url，下一次请求的url是几天变化一次的）
    
      
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTYD4rp2oIoKy4mbwHqWbSC2WPRoTia1b4ofMDdB4ictK9WEJtv9mYXzDaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)  
    
2.   携带上一次返回的两个cookie去获取新的abck，用于后面加密。
    

  

第三请求：带上计算出来的加密去请求验证并获cookie用于后面的操作。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTYHXO81JxiatdLWlHNRnkH9wg5icbZ5BtTPiaW9iaQic75C5FvlAQwz2Wcm5w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

第一步加密分析就到这了，后面的就先不说了，因为后面的加密真的都是大同小异，只是里面加密用到的值不一样而已，不过要想把整个业务走通，仅仅解决加密是完全不可行的，不然大名鼎鼎的Akamai不可能这么low，他后面的TLS（如想详细学习的，华总[公众号：编脚料]的那篇ja3文章可以了解一下），waf啥的，风控比国内的玩得花哨，玩得细节，玩得真实，总之他就是会玩，哎，就是玩！

  

  

加密分析

* * *

其实另一套能运行的方案是通过补环境，包括写了些过指纹检测的框架和过风控的方案（其实是我另外几个大哥写的，他们太强了）。挺稳的，就是他一更新风控，我们就要跟着动（此处省去好几万个羊驼），随着不断对他的js代码熟悉，发现和国内知名的安全公司（某数）太像了，所以就萌发了这个思路。

开始！直接定位到他的关键部位：  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTYRtIOODsqOBzeMr4l0qdEU33xZmxAicZh91lfE6btroUvq3dLRhvxnyA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

对，没错，就是补齐这个58位数组（2.0版本早期是50位数组，我估计后面还会增多）。

如果真的想很愉快的且后续更新以后想快速的修复的话，真的有必要把这个数组里面的每一位元素代表的是什么意思搞得明明白白的，这里就是体现出了还原算法的优点，他检测的东西，我直接给你写死（当然有些是不能写死的），我不就有这个你要检测的信息了吗？再多废话一句，他检测设备信息，无非就是用来加密或者作为代码执行流程的逻辑路线判断（国内的某多多pc端就是这么做的，然而这里的却是用来加密）。下面就针对这个问题来看几个元素的组成。

  

这里的代码是混淆的，如果走算法还原的路线，建议借助AST解混淆以后在分析，会轻松很多，对于没有AST基础的大佬，推荐两个本人曾经看过的教程,一个是猿人学（也是我入门js，安卓逆向的启蒙教程，里面大佬巨多），还有一个 蔡老板（研究AST在国内有排名的那种，公众号：菜鸟学Python编程），对于这个网站解混淆只能用于静态分析，他有好几次对他原生的函数进行签名。

  

好，打住！由于之前一段时间在学习安卓，所以AST忘得差不多了，没办法，这次只能看混淆的了，因为真的一时半会写不出来了，比如这里：

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTYrh1zonOJ5FM8MG9J6QYWhayWAZzEZfr3j89fa9m5dMEL2RDBmoQia9Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里就是检测你有没有他要的这个东西，有就是1，没有就是0，然后得出来的值会拼接在一起，组成这个58位数组的第二个元素，用于后面加密，这是一种用 1/0作为结果的方式，还有这种是具体值的方式：

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTY31mvv8ia5CGvoKzmibRic5BGSV180OIPZibWMUHPYfQSSWtF8WVRSjXPicg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

这里就是检测是否有陀螺仪，这里一共检测了三个，另外两个：设备的位置和方向的改变速度，触摸功能。然后这三个得出的值拼接在一起作为58位数组的第8个元素。

后面的基本都是这样，这回应该知道为什么这个数组里面的大多数元素是不会变的了吧？

提示：开头就说这个很像某数，那他的那几个时间戳记得留意一下哦，它里面一共用到了5次时间戳差值，和很多次stratTs的计算，应该就是来监测它几步计算的时间，间接性的去检测代码是否被调试，这个也会用于加密的哦。

其实一开始我准备了很多写文章的素材，但是我觉得我都说了是某数的思路了，我觉得再讲细一点就有点啰嗦了，好多大佬现在差不多已经可以搞了，所以就不罗嗦了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTYdFmZ7pugf4qOnQDjSA8Cd7hFPNk971yzreNhQbYOGiaTST3muo3JVmQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

虽然有58位元素，经过多次对比就知道哪些是动态的了，把数组补齐以后就可以愉快的进行还原加密部分了。比如这里就58位数组就刚好组装完成了：

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTYYVblCSJYFASCibNCseY1WT2ibia1NsNydwS7V264JLl3BYC9TRGqkS4gQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

加密部分：58位数组组装完成后，第一步就是通过方法（dX = d3(KY, [bX, h1])）使用58位数组去生成一个随机的字符串用于后面将58位数组转为字符串以后作为间隔符号将他们串起来。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTY059ckFmU3hJNTrm8ib35brK7rn4SYthNLuk0T3OmGI9trV6RaOR99Kg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下来就是各种对这个Cn（也就是将58位数组join成的字符串进行各种转换）操作，流程很清晰跟几遍就明了，接下来操作都是各种算法细节还原，说白了就是他是怎么操作的你就怎么还原就好了，没啥细节，唯一的细节就是那个startTs（开始时间）他是你第一次请求首页的时间。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTYgjmDvztC9zeuiakXkyjxJV8vHhBRYQtbNicRnw06Y78E4cg8wxjUN70Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

不要放过它里面每一次对时间的操作，基本都会将结果在半道加入加密计算的。后面其实真的没啥好说的了，因为第一层加密校验的东西真的不多，在通过看他的加密逻辑还原的时候细心点就完事了，在加密的时候无非就是把外层得到的 startTs abck，bm_sz拿进来参与计算就好了。后面的步骤会校验的东西多一点，包括每次更新的abck和bm_sz（这个变化得比较慢），他的每一次请求加密关联性很强，所以好多东西都要实时变化，就是考虑到这一点，所以我觉得对于这个网站只从加密角度来讲的话，还原算法优于补环境，因为补环境要处理这个问题的话是对你对js这个语言得理解深度很有要求。最后还原出来的算法代码大概也就几百行的样子。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTY9ZwxicSu84SicJuT6XRHWYZyfiaw6XWagEGjJRatC9PCibAgQkJEHHNHOw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

最后验证：

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTYVOt3I7ibVGJBiacbqASwPRQogYtibCmYwbPgkkghEUUU9CBD476pDpGtw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

好了，第一次加密就没了，确实本人最近时间很紧，所以只简单的介绍了下加密部分，后面的加密差不多都类似吧，不过确实要复杂一些。  

  

结语：对于对抗这个东西，真的要花很多精力，时间，他的风控更新太快了，比我那几个大哥还快，由于本人太菜了，准备不搞了，跟不上脚步了，以后再搞是狗，至于现在成熟的这一套让我那几个大哥搞吧，致敬所有（js逆向，App逆向）在对抗最前线的大佬，瑞思拜！

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicChT5PFHy6ATBuSS5PArOTRmnkm2iaTYqHA6hg9dzWhwrwicqS5yfIPVIlv2pr2UAFhPt2IZlQLz81td4rWGdzw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)