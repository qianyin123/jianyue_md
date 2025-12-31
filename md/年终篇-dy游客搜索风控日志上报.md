> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0PSdXHP3Lck1_mxLbH3wgg)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnyGDTnD4TtcYkFuhoibFWx8RWqVPxvSUiaJtwGvugCAvQC4R7VFjWjQfA/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)  

过程
--

### 介绍

dy游客登录的ttwid生成就不多讲了，主要是两次请求首页或者目标页面html 外加一些算法 __ac_signature

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAn2erRXK6HA2c4ThGRLO8g8EEXJd1u0CMavx6PoibxWFx9mSyQn4uUxkg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

但是通过协议生成的ttwid配合a_bogus以及代理发现利用率很差，使用几次就失效了, 而且a_bogus是必须的，对于代理的质量要求也很高

```
{"search_nil_type":"verify_check","is_load_more":"first_flush","search_nil_item":"verify_check","text_type":9}  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAn1KxvMZkXkbL5CftfNxaOiaJduABcbcIaLxtM3xzavst4ZHwKBvPicjZg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

神奇的是复制浏览器端生成的ttwid下来使用发现有效期很长，并且**不用带上a_bogus就可以通过验证**

不带ab 10次全部通过![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnnP1G0CR4eX6jk566zMITTECm42icQJYg54mKTL3GODHqIzlOvvOZ4MQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

可以确定是ttwid的问题了

策略
--

*   当然你可以选择用自动化工具去生成ttwid，但是它存在个问题生成ttwid也会不断进入业务页面比如搜索，请求多了还是要要求你过掉网页上面的滑块、点选、拖拽的验证才行！
    
*   通过协议的话可以确认的是ttwid从html页面返回到最后使用没发生改变，那么应该就是带着它请求了某个接口然后有一个**激活**的过程，我们就用代码去发这个激活的包就好了
    

### 测试

从ttwid生成到我们需要的业务请求single/search，发了很多个xhr的包，直接使用二分法下xhr断点减小分析包的数量

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnuY6SYbJCicOMicSPoiabtiaHkXJgAEb5bsGHTnicwBF02h2EvTOT5Tp6pqA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

#### 1 /search/stream/

下这个xhr断点，看下发包到这个前生成ttwid是否可用，注意情况ck生成新的![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnzP0fxYqpOogkSg83x72ib8Z7CzE06KN16UC3Mic2TSmV7vASzDqWdA9g/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

复制ttwid下来![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnepB2MYY6s7Vlh4TxkLpadFaRreeMtUhhDTalRZSL77FAh0WfianZWYw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

发现并不可以使用

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnGQMCk6YmW8UMqqp28yOfzVpoCKIKyZFibnZFHVibP0FOibkDNBg2dEyMA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

#### 2 /web/ab/params/

看url路径有点说法，还是要验证下，注意它发了两次我们直接看第二次的

同样流程再来一遍

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnMgjRKVfrIPwpBmytqwkwE1d1sDM2yQnnKEjkJDUNItgsHuGk4G2MAA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

还是不行![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnmeLYYJTSGWtQqIISLiacP4icfg4huBRVuOBHP8odWSTUwIAPmLEe3gcA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

#### 3 /web/external/notification/

再断住这个，发现可以使用了

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnKX1GF16OTbI4bIWRewXnXyZN6VtBNOJy1jiaBq2tRSHzlp0Nv98HEtg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

总结下就是第二个/web/ab/params/  到  /web/external/notification/ 之间的包

### 定位

chrome有个屏蔽网络请求的功能，我们下把/web/ab/params/ 屏蔽掉看下生成ttwid可以使用不

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnR4B3jTCnmnewynaOKpxE5O6Ifz4CTibSUXicF8c0ibbLqbrBRlrRU5oFA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11)![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnh5GiaULHHl3yiaEM9Zia6S8t2hdvG8zBvQfiaXb4wVFNbzFnKxuhAYnIYw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12)

可以使用，说明不是这个/web/ab/params/

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnzoO1eialwRLlE8I1pNABqbdMoO5jGdfgwmgqDiacZWIaNVnye5FegTPA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13)

接着看下两者中间值得分析的也只有本文的主角了,它虽然没上传ttwid信息，但是请求体的压缩数据倒是值得看的

```
https://mcs.zijieapi.com/list?aid=6383&sdk_version=5.1.24_dy&zip=1  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnTRNHWqic0RIuaou1y2thO45NWfIcsnic3wwz5gSqrYbPP5aljibZibCLdA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14)![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnresZwEYT8bd3mOnGD9lqnzeiatO35nPdSv0S2ibWvyY7p0qicpicAY0hLw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=15)

我们屏蔽它看下生成的ttwid还可用吗 不可用说明就是它了

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnp4LTNCMC0tyLA4JSIl3jHYLqFjiahOZyUHb51GS8icQdVfree1O5FFzg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=16)

**不可用啊！终于找到它了！**![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAn3R6DKmVX0efNwNxUicgb8icHx2lCUQnB0M9X60WQDbGoWwvSia0xrGEiaQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=17)

跟栈
--

因为目标的包这次完整的请求发了很多个  因此我们要先断到/web/ab/params/ 然后在下它的断点  减少分析量

请求体加密位置再下面

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAn8KoaoibibWhDMLBNz6K2deKBIvDsZL0tcrQ5uC6GBpFcNXRv2JIo63hg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=18)

注意一定要断到第二处ab/parmas再下目标断

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAn6qAtFvjrJFcUl8ickcHxSoMKjJae430Qv51pjibxom60zZf1c1u8cVxg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=19)

来了

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnib0spMPiaibQHP6r82dXo78IoPCApkPO6SSPsjdZF3vdJW4fz46EiciaS2g/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=20)

这里还是跟之前一样逐个测试

经过几轮验证发现，只要events里面包含整个search发过之后几秒后  ttwid就被激活了

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAndHx2hFiaEiaUhFOgcp8adZzS4b1I0ibpl1QIeqbSavP5zQYSMfqJBMRDw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=21)

复制这个明文请求体下来分析

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnGkaMicYQt3P8Ra2H8Lyc1hkRlibWgcpTswULrDImhtG89j8mBvLmAgsw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=22)

分析
--

### 目标

*   信息里面没用ttwid，那么服务端如何去激活ttwid的
    
*   events是什么如何去构造
    

### ttwid绑定user_unique_id

全局搜下这个user_unique_id，发现它是html页面跟ttwid一起返回的，所以它俩是绑定的关系！

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnbBB4Jc3KiaUS2If9iciawOMIQaEEBgezwvR5ia9CvkiafUtTpCyiawsbic4Yg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=23)

### 构造events

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAngcSia7iabeVqOhAWX5D6ibzuLdRDW1r17Q7XyV5m6L69PsqN4elVCVJibQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=24)

这里主要展示页面的信息以及行为特征，细节结合Ai去了解下就可以了

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnuje0RBTMLdGyBRYYUIMmia79n3uC5HytkRhU9eibH8dOPEagT2Xbbrbg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=25)

最后自己构造个动态的就可以了

大功告成，激活后的ttwid就不用a_bogus也可以请求直播搜索了![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnRXZ17VLzPzE41shdNn02Uu4doJTkanoVodVSPicSjGJBWH1NoG4LhrQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=26)

广子
--

> ❝
> 
> 有需要用于爬虫、逆向、抓包的测试机又不想花时间折腾Root、刷机、环境配置可以来看下下面联系方式，或者B站视频评论区见

![图片](https://mmbiz.qpic.cn/mmbiz_png/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnUoLXMn3p5Uojb1S10r0X4dpmuibibiaUgILsk43sGfWibu1ib6GZXZNRZqg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=27)![图片](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuyFssrgp68cUIV2Xynx3SAnHtQ1F5e0ZMjszK4fOpklL3JdpA9bS1GqMFEVU7Ap9syWXzQMnRTE7g/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=28)