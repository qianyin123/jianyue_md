> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43411585/article/details/123030973?spm=1001.2014.3001.5502)

### 目录

*   *   *   *   [一、常用的js基础知识](#js_1)
            *   [二、常用的谷歌操作](#_23)
            *   [三、jsvmp的特征](#jsvmp_28)
            *   [四、jsvmp扣逻辑](#jsvmp_37)

#### 一、常用的js基础知识

*   [js各种运算符](https://www.w3school.com.cn/js/js_operators.asp)：比如位运算符：`&、|、^、~、<<、>>、>>>` ，算术运算符：`+、-、*、/、%、++、--`  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/bb594972ccb54fbda1333706c1d2a55c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/f9dc89a2099a424da87fa96589614bd2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   三元运算符又称条件运算符`?`：表达式`结果为true`执行`冒号前的`：value = 表达式 ？为true执行：为false执行 ，比如a = 5 > 3 ? “true” : “false”  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/32b3587ea998412c90f75d78c35fbf22.png)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/d250dde939be491d88b392602d863b23.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/1f0679623ff34e5b9da848c36f1b4b8d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   `String.fromCharCode(num)`：可接受一个指定的 Unicode 值，然后返回一个字符串  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/b17c61c5d4ab4b3db6f9e9fe64a7b60c.png)
    
*   `'str'.charCodeAt(num)`：返回字符串指定位置单个字符串的Unicode编码  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/804161700d884668a3dd8472ed0c17c7.png)
    
*   `parseInt()`：把一个字符串对象转换为一个指定进制的Number  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/420a82e2d41d454384d3e0347ecc199a.png)
    
*   `toString()`：把一个Number对象转换为一个指定进制的字符串  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/92459c9f311a4883b5e0717abd2950a9.png)
    
*   `slice()`：截取指定位置范围的字符串  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/8609c311cd354429a4e42910bcd973c3.png)
    

#### 二、常用的谷歌操作

*   谷歌开发者工具打断点，如鼠标选择行号右击，`插桩断点Add logpoint`可以打出日志，如`条件断点Add conditional breakpoint`可以根据输入的条件表达式为true的时候自动debugger住  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/ff47aebe976244a0a53cfe0e0375b1af.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   xhr断点监控：根据url请求中的链接参数特征，指定监控  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/e515c1b599b3433e9cbdb77a7b3976b6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 三、jsvmp的特征

*   jsvmp的特征：请求参数signature反爬，加密算法文件为`acrawler.js`以及文件里面典型的提示字符`window)._$jsvmprt("`  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/2c3a6f0fdb7046d890008ac0a23d23cd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   [关于jsvmp的代码逻辑有点类似生成器执行，可以看这篇文章简单了解下](https://www.cnblogs.com/luoxiaoer/p/11735768.html)
*   [关于jsvmp生成的逻辑思路看这篇文章，简单了解下](https://bbs.pediy.com/thread-261414.htm)
*   [关于jsvmp虚拟机保护方案](https://mp.weixin.qq.com/s/YDx5Dr-HDfAm-sAqeWW0qg)
*   关于jsvmp的解法一般有3种，`补环境，和插桩扣逻辑，jsrpc`，当然还有自动化等方式可自行研究试试
*   `补环境`的可以看[这篇文章有相关介绍](https://blog.csdn.net/weixin_43411585/article/details/122374204?spm=1001.2014.3001.5501) ，网上有很多，可随意查找
*   `插桩扣逻辑`的可以看这两篇文章：[逆向简史的公众号](https://mp.weixin.qq.com/s/YAL4iT3ECQeK7vZRsLRFaA)，文章前半部分讲了补环境的关键点，文章后半部分讲了如何扣代码的关键点，以及关键的算法函数，强烈推荐，看完实际操作一遍，你会豁然开朗；当你学会了逆向简史的文章，然后再看[小小白的公众号](https://mp.weixin.qq.com/s/sVDHQeRY-774Jw70k7mINQ)，跟着他的思路介绍再巩固一遍

#### 四、jsvmp扣逻辑

*   定位入口：目标参数signature，采用xhr断点定位，发现是由`acrawler.js`这个文件生成的，然后我们插桩日志输出，找出`window.byted_acrawler.sign`函数的传入参数，然后就可以通过补环境的方式生成signature，[补环境和该篇文章的解决思路差不多](https://blog.csdn.net/weixin_43411585/article/details/122374204?spm=1001.2014.3001.5501)，此处视频讲解在公众号：逆向OneByOne有  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/661d6167973b4d408dcd0914327c2f90.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/c9c66faed708487ea0ec628f7fe776d2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   扣代码正式逻辑：如果你看完逆向简史公众号的文章你一定知道signature 是由 9 部分组成的，第一部分是固定参数，剩余的8部分都分别由相应的算法逻辑生成  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/73fe9a29a7d949b784c1b6e0adeb3599.png)
    
*   接着重中之重的是打日志断点插桩的位置：分别在arcawler.js的184行和425行加入日志插桩断点，内容如下
    
    ```
    `"索引j", j,"索引O", O, " 值：", JSON.stringify(S, function(key, value) {if (value == window) {return undefined} return value})` 
    
    *   1
    
    
    ```
    
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/710d9358669e4e99aff597c716380cc3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/95a4f34da6934bf8a49b052d4207498a.png)
    
*   然后我们清除网页缓存，刷新网页，会发现大概有`1.5w的插桩日志`打印出来，而通过ctrl+f搜索，signature也在我们打印的日志里面输出了，也就是`j ===24 && O === 40336`这个位置有了signature以及其对应的值`_02B4Z6wo00f010kDMrgAAIDCwkipWBmxc99JAzYAALBd35`  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/161cda40336941fdbc793e58bae59179.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   我们继续搜索值`_02B4Z6wo00f010kDMrgAAIDCwkipWBmxc99JAzYAALBd35`，会发现signature的最后两位也就是第9部分135`，生成逻辑是`c468ee35`这个字符串的后两位截取生成的，然后我们分别搜索其它②~⑧这几个部分生成的位置
    
    ```
    `① _02B4Z6wo00f01 ② 0kDMr ③ gAAID ④ Cwkip ⑤ W ⑥ Bmxc9 ⑦ 9JAzY ⑧ AALBd ⑨ 35` 
    
    *   1
    
    
    ```
    
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/afe3561c5367423e98861102a453aa72.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   每部分都是由单个字母拼接而成，我们拿第2部分`0kDMr`再详细看下这个流程  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/503e48c4690e4ec78ee7fcee4e745312.png)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/9e177fb377bf435d9ec2c06abc16d4a5.png)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/8034f09270344635a007c6d81d478709.png)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/56305a3b100a40939714f9d883cb9bd6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/6956b5c286fd423bb0efe70e7a1e82d9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   最终第2部分`0kDMr`的生成逻辑就是这个，那其实我们只要继续研究这个[48]、[107]、[68]、[77]、[114]这些数字是怎么生成的即可；这个你在逆向简史的文章里面也可以找到答案，或者在我公众号里面的视频里也能知道如何去逆出这个结果  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/aff20b2c720d46d5854d29a1fed1ea7a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   比如48是(`35394057981102`>>2>>24)&63-4得到的，`35394057981102`其实是由二进制转换而来，由于细节太多，讲不清楚，可以到逆向简史的文章继续看分析逻辑  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/556f8b3f677b47d2afc24ada556c7fe5.png)
    
*   对啦，我开了微信公众号：`逆向OneByOne`，后面有些我以前发的比较好的文章会逐步迁移到公众号下  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/90d4017f12ac4d8cae2c5a15f6623250.png)