> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43411585/article/details/123672621)

### 目录

*   *   *   *   [一、案例分析](#_1)
            *   [二、signature定位与分析](#signature_6)
            *   [三、X-Bogus定位与分析](#XBogus_37)
            *   [四、滑块captchaBody还未研究](#captchaBody_52)

#### 一、案例分析

*   案例网址如图，研究的是这个接口，获取用户视频的接口  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/c97eede94e874ce580503405b7a93327.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   研究的参数是请求参数当中的`X-Bogus和_signature`，并不是所有接口都需要这两个参数，有的并不校验，有的只有cookie即可了，有的有这两个参数即可了，具体接口具体分析，本文只是研究下这两个参数的生成逻辑思路  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/fdd491a6a0ca4760a67762085b4b393b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 二、signature定位与分析

*   采用xhr定位signature，首先清除缓存，然后刷新网页，会定位到如图位置  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/815f476563e64758b732098d2aafd0da.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   往前堆栈回溯定位，发现signature的生成逻辑是在这个`webmssdk.js`里面生成的  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/d2d793b7783b4f78afb56bcbb521db04.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   `webmssdk.js`是混淆的，所以我们直接利用`v_jstools插件`进行一个变量压缩还原，[v_jstools的使用](https://mp.weixin.qq.com/s/LisYhDKK_6ddF-19m1gvzg)，并将初步解混淆的js保存到本地文件并命名为`webmssdk.js`  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/47097f6b5ebb4534b0e19683584e879a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   然后我们用fiddler进行替换，将网页混淆的js替换成我们本地的解混淆的js  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/07e627fc355b4b76ae2e57a6976288ed.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   此时刷新网页，会报错如下`Access-Control-Allow-Origin`跨域错误  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/daf7e14ad0ad4f8fa8bd7daad103058a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   解决`Access-Control-Allow-Origin`跨域错误，可以用fiddler进行如下设置，在Filters下设置Response Headers的跨域问题  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/af8c3af58c7f47feb75d3701728b17db.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   此时我们再次清楚缓存，刷新网页，会发现混淆的`webmssdk.js`变量已经清新了许多  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/e73e0a20956945edaba7c96dd6514c5d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   开始分析signature算法逻辑，首先在js当中插桩打上如下日志断点，`主要是看插桩日志分析逻辑`，所以混淆与非混淆的影响不是很大，二选一，不过混淆的还是稍稍费点眼睛的，之前没有分析过，没有经验的话，还是选择还原后的js比较好，如图在184行和431行插桩打上日志
    
    ```
    `# 混淆的js下打的日志：
    "索引A", _0x1ff777, "索引I", _0x188445, " 值：", JSON.stringify(_0x6f0288, function(key, value) {if (value == window) {return undefined} return value})
    # 非混淆的js下打的日志:
    "索引A", A, "索引I", I, " 值：", JSON.stringify(S, function(key, value) {if (value == window) {return undefined} return value})` 
    
    *   1
    *   2
    *   3
    *   4
    
    
    ```
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/2d3e3e418d95471e9ff764a357793e60.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/32f147870a6a4febbe104e227a889c59.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)

*   最终打印出来的日志差不多1w行，然后通过搜索日志去分析组成逻辑  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/05bac1b490f243598fd686cb7d56ae62.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   signature：`_02B4Z6wo00001ZvTpJQAAIDAEJg.d0yVePmb06AAAATP14`由9部分组成  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/7b9a1afaea204537938a4fd1ce8bf327.png)
*   第一组是固定的，拿第二组、第三组举例，每一组都是由长数字`35392257845541`转换而成，每一个字母都是由`String.fromCharCode`转换而成的，而图中的长数字是怎么来的，char_to_signature函数又是怎么来的，这个在我的[jsvmp纯扣算法](https://mp.weixin.qq.com/s/81sKkysbWyPeHEtuFT_ynw)里面有讲逻辑思路，所以这里不再重复详细介绍，思路流程算法代码几乎一致，只是传参有所变化，至此signature已分析完  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/60370c24428d4fac9a5e78db8fb46a87.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/a5584c1aee4541298adaae7b4fc982eb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 三、X-Bogus定位与分析

*   X-Bogus与signature定位方法是一致的，在前面xhr定位包括插桩日志当中我们其实已经发现X-Bogus的生成结果了，这里的分析算法思路依然是用的[jsvmp纯扣算法](https://mp.weixin.qq.com/s/81sKkysbWyPeHEtuFT_ynw)这篇文章的介绍  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/02f2b0182c184c4d803bdf2bccb122ee.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   X-Bogus的比signature稍难些，难在分析长数字的组成，signature由9部分组成，x-bogus由7部分组成，`DFSzswVOx8zANGR8SRc5oPt/pLv8`，这个算法主流程和[这篇文章的流程是一致的](https://mp.weixin.qq.com/s/BJ2F2o7RXQ3yAVE3-08fcA)，区别在于md5_str的生成不一样，x-bogus的会更复杂一些  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/edd653c09f624bf2aaac4c1eb9076f9a.png)
*   如图第1组和第7组，我们又遇到的长数字`196397`和`934988`，如图第7组则是长数字经过转换而成，而每个字符串是由`固定的字符串取索引`而得，至于这里的长数字和num_to_X_Bogus函数是怎么生成的，我们看接下来的分析  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/61e528961dc34af3a8934e9dc773b115.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/8fde997c844e4a4ab557a42e17801c34.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   然后我们拿第一组的长数字`196397`其实是由`(2<<16)|(255<<8)|45`换算而得，而一些奇怪的数字其实是由乱序的字符`\u0002ÿ-%.*ÔË^–\fñŽšš¿N\u000eDL`转换而得，具体自行领会了  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/31e5eae515f14581bc68bb2e1b5e9577.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   至于生成长数字的乱序的字符串`\u0002ÿ-%.*ÔË^–\fñŽšš¿N\u000eDL`又是怎么生成的，这里比较难暂不公布了，但是只要掌握了前面的分析思路都是能扣出来的，其中往前分析会遇到md5，而md5算法没有做改动，用`var md5 = require("md5")`是一致的，但是用`CryptoJS.MD5(a).toString()`对于Uint8Arrary进行md5加密是不一样的，所以需要注意下，还会涉及一些Uint8Arrary转数组  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/c57a11b6f70b4889839a9baf023a3a5f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)
*   `num_to_X_Bogus函数`如何生成，这里其实也有开源的代码，[看这个文章](https://blog.csdn.net/wdfrog/article/details/6176835)，我就点到为止了，自行根据插桩回溯条件断点等调试领会  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/26eeb5c2ceae4562944844d775c42c93.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 四、滑块captchaBody还未研究

*   通过堆栈回溯的方式找到VM的文件，然后插桩打日志分析逻辑，初步猜测和`AES-GCM`有关，其它后面有空再研究吧  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/5200dd8540274adf85c527a09d1b1f8c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/8308c0d5d598488eb5baeda6ff0db16a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/ca1d1a1d870946beb80b7ffb9d80e568.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2B5LiA5aeQ,size_20,color_FFFFFF,t_70,g_se,x_16)