> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_23262677/article/details/135992308)

  声明:

        本文章中所有内容仅供学习交流，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！

  前言:

        听说最近某宝水果[滑块](https://so.csdn.net/so/search?q=%E6%BB%91%E5%9D%97&spm=1001.2101.3001.7020)出了一个bx-pp参数 是wasm加密,抱着好奇的心态打开的网站.

        打开网站可以看到这个新的参数bx-pp

![图片](https://i-blog.csdnimg.cn/blog_migrate/506f03360c7ec813e883aaab387d7d66.png)

        我们直接搜bx-pp可以定位到位置接下来跟栈的过程就略过直接到重要的部分,直接定位到下面这个位置,可以看到b应该就是加密的入口 其他则是加密的参数,我们看看参数都传了一些什么.  
u:0  
f:0  
g:"a"  
h:"1706839682508"  
m:"md5"  
k:'qweasdzxcg'  
y:'673b248503cfe1dc61cbbe3feb7b6924f9b8ce84081dffc66e3175cb138da6c7db927017d1627c374db7fcb0b9a1931a'  
a:'1'  
i:'1'  
r:'1  
w:3000  
        重复观察几次后发现会变动的参数是h、y 其他参数

![图片](https://i-blog.csdnimg.cn/blog_migrate/8f152769db7e690342dd231ee9b2840d.png)

        我们继续跟进b方法看看,可以看到wasm的身影了,好接下来掏出反编译的工具进行分析

![图片](https://i-blog.csdnimg.cn/blog_migrate/c0870bfbce95834347a1293f6442a971.png)

  
 

        打开Ghidra把wasm下载下来之后拖入分析

![图片](https://i-blog.csdnimg.cn/blog_migrate/671f1aad3773d5174fe0082ce2014773.png)

        我们看到调用的是o.h的方法,同理我们在Ghidra反编译的方法里面找到h方法如下图:

![图片](https://i-blog.csdnimg.cn/blog_migrate/078c1835e6bd2b683f141d7f6b9afb62.png)

        接下来我们据根据网页上的wasm跟Ghidra反编译的c代码进行联调,这里先拿一个func13方法进行讲解

![图片](https://i-blog.csdnimg.cn/blog_migrate/b1f7aff30e654cb51a66d33107f9fe8b.png)

        我们可以看到这个部分是取了一个时间戳然后定义了一个1000的值两者进行除的操作我们这个可以在反编译的代码中也看到这个部分

![图片](https://i-blog.csdnimg.cn/blog_migrate/f9837739125f36a3003c5916ac02ee0d.png)

![图片](https://i-blog.csdnimg.cn/blog_migrate/3dd7f5e0a50aaa60cbd46b4d9e7ca92d.png)

        同理下面的代码就是11对应所以我们就可以用代码进行还原

![图片](https://i-blog.csdnimg.cn/blog_migrate/b4c41db7cfb4304dd84e59791e7e2dca.png)

![图片](https://i-blog.csdnimg.cn/blog_migrate/1b9f2ba104f7dfdbdae28a82e9d3dc9a.png)

        一一对应进行还原成js代码最后可以发现func13主要的操作就是把时间戳分成前十位和后三位,实现的js代码如下:

```
`// js代码实现``t = 1706842045600``QY = t%1000``QZ = parseInt(t/1000)`
```

        这里我们可以看出func13处理出来的两个值的用途这里我分开去说先对QZ进行-1操作再经给下面进行循环*0x5851f42d4c957f2d后+1循环的次数就是QY的次数

![图片](https://i-blog.csdnimg.cn/blog_migrate/ab276cae752bfed8861081b371be7895.png)

        这里为了方便我直接用c++进行接下来的还原操作

![图片](https://i-blog.csdnimg.cn/blog_migrate/ddcfb5fba3125699370fce1f0ded7174.png)

        先定义入口然后再定义一个方法把我刚才说的操作用c++代码进行还原接下来的部分还是和一开始的部分一样根据Ghidra进行还原

![图片](https://i-blog.csdnimg.cn/blog_migrate/312f00b330fa850c3d7f7baeb7f97ecc.png)

        还原后执行结果如下:

![图片](https://i-blog.csdnimg.cn/blog_migrate/15df21e47219a398058b3f234dd423ee.png)

        那上面这串字符串什么用呢？我们接着往下看跳过繁琐的过程我们讲最后是如何生成的,我们定位到下面这个部分在浏览器上我们输出一下这个位置的字符串(func6函数作用是获取了内存中这个位置存在字符串的长度)

![图片](https://i-blog.csdnimg.cn/blog_migrate/996ac6e6b0e9361821c9d396ba1258db.png)

通过输出我们可以看到三个参数  
第一个:32位的值  
第二个:明文  
第三个:就是我们上面一开始的y值

![图片](https://i-blog.csdnimg.cn/blog_migrate/27be81166af3daae0f959cea81e00f5e.png)

        我们一开始可以看到有个md5的字眼再根据反编译中一段散列的hash算法我们可以大概率猜测第一个值是[md5加密](https://so.csdn.net/so/search?q=md5%E5%8A%A0%E5%AF%86&spm=1001.2101.3001.7020)值,我们用工具进行校验看看,用加密工具加密明文后可以看到这个值刚好对上32位的值,成功验证了我们的猜想

![图片](https://i-blog.csdnimg.cn/blog_migrate/95987ef381fa8eb488ccf985f3cbc031.png)

![](https://i-blog.csdnimg.cn/blog_migrate/cb29bd11e971397102113c8aba5b5216.png)

        接着往下看我们可以看到这里有一个类似占位符的代码那是不是这里是把刚才的几个值都填进去呢？

![图片](https://i-blog.csdnimg.cn/blog_migrate/21cde87f40ed988974ee0484e2437c78.png)

        带着猜想我们继续控制台输出,可以看到这不就是我刚才输出的那一段么,到此距离成功复现加密值就差最后一步了

![图片](https://i-blog.csdnimg.cn/blog_migrate/f33b58d568198a7c0c8094ef8768ca61.png)

        我们接着往下看的时候可以看到这一段代码,熟悉的人可能一眼就看出来这段代码做异或和移位操作去生成字符串

![图片](https://i-blog.csdnimg.cn/blog_migrate/7b4f642355872c1ca3542b8b049dd0a8.png)

        我们写一个算法来实现一下,上面那个是网页生成的下面是根据我们算法生成的我们可以看到在我画红点的部分charcode为10的时候他会处理成a、code为12的时候处理成c 那同理我们也对应处理

![图片](https://i-blog.csdnimg.cn/blog_migrate/287b3253401f423bb661e84cb5239d90.png)

        修改代码之后再次验证,这次我们可以看到等于true说明加密值对了,至此对于bx-pp的加密整体流程就算是结束了

![图片](https://i-blog.csdnimg.cn/blog_migrate/23e565fbde6f5a28b1ffdb87fa8656d7.png)

        最后可以直接用python代码还原算法

      ![](https://i-blog.csdnimg.cn/blog_migrate/699a06fdb383fc00bdc393bbdf74cd38.png)

        有兴趣的可以加入我的星球后续会持续分享和更新

![图片](https://i-blog.csdnimg.cn/blog_migrate/f43613afb6fa6d13253416e1bbee42f1.jpeg)