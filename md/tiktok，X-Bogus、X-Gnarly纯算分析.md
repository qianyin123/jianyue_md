> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/UJxdWWtYkjJMXk8TbLRP_g?poc_token=HHGjZWmj5kZbrG8BHQzXqHye9IUHoTv82BY_R4ly)

本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者立即删除！

一.插桩**:**
---------

  

我这里是手动插桩的，主要都是这个F函数，标有序号，后面好查在哪里生成的，大概插了100多个，日志框架也是用前面文章的

这里放一个我插好的，只不过好像被检测了，但是正常出值没问题，有需要的可以试试

https://pan.xunlei.com/s/VOiegkJc-T-OihXTJJSlnVrrA1?pwd=nf6d

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBT5EZ9fmqba1mRRzAx2slD9qhffvm9n5vOHQt5H83LicxrmrPA7aJIicw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBGx1cewcNWc3j2ojy8tpFqibRj3zRpibr3grwg3mUJYGLRnr9Xl747Stg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

二.加密分析**:**
-----------

  

X-Bogus貌似不校验了，这个也就跳过

先找X-Gnarly最后生成的函数，参数字符串，最开始出现的位置在哪里

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKB6phIcmRPBXLdK2G0cSLOkgZH8U8w0x8GGc5FopMNNicC5nWuQgarDyA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

是一个数组的明文生成的，大概猜下，是明文通过某种方法转换为数组，通过fromCharCode来生成

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBLT8hVCXXIfu7rdh3NRt0KWbqlZSFmn5eCYP3eATbcSTBvuflqdcmXQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBKBEhjW26IBeZyiaT8HMzIqibxNe8KA3VMSWlfP7S6kt7rEfuVezaNWjg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

数组最开始的16是明文的长度push进去的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKB9CWEPONdUvrPhfF9nWfJ8c88cmHib194tBBNk0eOibVOhfqicWJPM9odA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBXSz0uvWa2MHkvQiaQA5DaCic6xia6MwxVria3gWia8F75N0lGz4wc7ZkIaA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

把这个数组生成最开始的位置，到最终生成的时候，丢给ai

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBksicUEFuxfak48emQZCICq5RIxkKZGYT5xlklM6TVZxBq4klibYtH7Iw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

很轻松就能搞定

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBp91bVUhSEcveSnnN2ucp6OyHSt406pmUoibicsglpCic3nnEr1UlW6sOg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

接下来看看那个明文，顺序是乱的，这是最开始的顺序

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKB6HSGnXXx3G9EoNFujicgEXB2LYzpCBkfBFQZRnTyPGr5iacos4gZib26g/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

经过一些计算，就变成了这种，对比多份日志，发现每份顺序都不一样

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKB7nmRh7r5LuiahmbTVSfz8tWQ8EEFLeyeLTfbpR3033ql3vDJibPnEeqw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

直接把变化的日志丢给ai，就是取了其中一个值，来进行洗牌算法

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBIu4XY5lCH0vxSY8fldxRmM8ic7ca7IqF0fzR1ngcpOADqT6706H9bEg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11)

接下来看看明文值怎么来的，前面3个貌似是固定的，先不管

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKB1zzqOGlKwpZSKh0ogLm1FN3mAWBicyfEcmGdtgayvf3DdDv41M6eWwA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12)

第4个是url的参数，进行加密，是个md5算法

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBGKIg9mpqdEo2p3J9sCStQoou5QeB3aPRiayCZq0uy8cbsrlWhNUJ8DQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBsxbMqh5nbZvTvbiaF0oAcS4COrfA5SeIgVfkXBWAibNZsD5c44Juj5FQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14)

第5个，貌似也是固定值，第6个ua的md5

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBdz1tsAPWO2Qs0nbCzzk0hYDnKTdA0QLeW5HC7exJibJxQChqoBRpbwg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=15)

第7个是个时间戳，第8个也可以看做是固定的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBTjOibqPdRcx3DWd5xe3l6bd8G6NqNQvateHZwDAbZfUJPiby89YckXRA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=16)

第9个，在日志里面是凭空出现的，这里是参与了生成第15个的算法

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBXuoT4cQ1DkshiaCfqMv2Nia6XhicAPAiaVgPyNibosOxNutmq5Z5jmcCGPw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=17)

这里就比较麻烦了，不是在vmp里面生成的，如果插桩足够多，要好分析一点，我这里没有。在原来的js里面先定位到日志数组19的位置

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBWq4CW6En62KrCiajcu7XzZiblBticNKb7RyZgictlLfJPtTOfw3rStfZmQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=18)

往上看几层堆栈

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBaiaSoxwZErBK0dbUibxEYFPfT4DNHLYVfZ58GhdYIWVr62dI615iay1Vg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=19)

  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKB4jWIGIrYRO4ia6ysg2JOEicJg22ib5FTKLeSgw9hYyK3cZPseuMhVExeg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=20)

发现，赋值给了n.o[143]的位置，我们直接在给和这个对象赋值的地方添加代码下断点

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBzcw2nO02ruXulSnpsmjJkeLohHVm09CNalhtf2paJiavPqL44nwdLzw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=21)

很轻松就能定位到

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBE9W9XuuRSiaX5I9GC9kDmXjG0Ho2dChUbjtmFiceE0FhfcWXrmgdzzcw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=22)

往上看一层堆栈，是这样生成

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKB4g84fVr78WxeEzmicWQrIAmYnYksgX8YXicicom7cLP1B0TNWxL24EPibA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=23)

进这个函数，发现t=964的时候取到的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBgJaysq3xG3YAicqsa0dPysa3fEsGwur3HKqpLef0PkViaFyJvT0h1TYg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=24)

直接搜索这个964，就能定位到生成的位置，后续有找不到的，都可以通过类似的方法来找，插足够多的桩也行

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBibI0yj7vn1HWw6w2001hvvlh5IuibvKtia3fIgRNqSRyB1jsJsAuKPibNg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=25)

这3个貌似也可以是固定值，想找怎么生成的可以可以通过上面说的方法来找，其中中间那个3是通过，下面第二张图来的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKB4Zk1icF6C4m4QUDBU34vcUCQGJPKAPyDrqaSYXRqy8fSOaJNE5QWoiaw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=26)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBUaYicO8JVVxEic42xPQypkryj2L63WCgnAIPo1UhvkbicVibJDYaW3Yp7Q/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=27)

第一位和第15位都比较简单，直接丢给ai就行了

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBrF9HHRrJM9QS6icqqadjXwBpZjqZ1brqAibHuzSZSGSLvia5cgMeb02UQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=28)

接下来看看这行，这行字符串是通过魔改的base64生成的我们最终想要的参数X-Gnarly

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBWEstkdpYsgmicz0p22sst7CacXUA3tzQCWRSjTSrfsFjUWPD19WSLLw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=29)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKB1x34SX424rb9NSYsPPyvbsPsvlOg72fBsicggOyiccS2zjyT2nf4yFicg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=30)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBoyrBoPmLSLiaz6atBibpV9PibCIiaEA0djjl4dKvxuTibcccDLic08icp1zUg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=31)

这个字符串是，前面加上了一个固定的k来的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBKqCNGCU3z4CRVZHyic0afWHVM7Raiah84XMy3mRscvTeTVP4ibBg1iaHgQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=32)

同样也是两段合并的，先看最后一段

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBprfxYba6ejwMXtyK2xETMfWUcnl6mu3iaeIC9rbh5s5lgaSXVAsAvbw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=33)

是substring(130)来的，对比多份日志发现130还不是固定值，这个轮数的生成就在前面，可以直接丢给ai进行算法还原

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBFMGnwicQX5D738EJyHj8OxMlpZgtq3V5dWBGnMzydUAowqr2cN3wZJQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=34)

前面的字符串也是两段拼接  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBustLQOLBh4OQ0fKHaDsOH9PgKBmGVZUTu6ibSialg2NlbKibic6PLbaDCw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=35)

这两段前面一段，是也是同一个字符串substring(0，130)来的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBOlckKfa4Usvib7lYzYTkC9D2y2qyuZibdlW71SVsMwe1fVGvMcs7WR4A/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=36)

后面一段是个数组fromCharCode来的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBPLYYOEI6R2b84zc2kMeByhuv3VXAHHsWnT94LgJ3qh0GQUYwSgkDUQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=37)

整体思路就是，把一段字符串分成了两段，k +第一段+ 数组 +第二段 进行base64，先找下这段字符串是怎么来的，在日志里面又是凭空出现的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBJUqvj54nV7Z3vcHysGnefamPbhI1QAvKZI4CDMJa2O3ao6vZu3lpJA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=38)

数组的生成比较明显是个随机数进行计算来的，还是用之前办法，很容易定位到生成的位置，参数就是文章最开始讲的明文，和随机数，和生成随机数的时候，加密的轮数，利用ai很容易分析出来

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBhQAJg9WTR3kKibWib1X9ED0K2biaFxH6RY6AZBVSd3PePZqBWSKSZzTbg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=39)

这个函数可以直接把他扣下来补全，比较简单，如果想保险一点生成的随机数函数也可扣出来，是根据一个时间来生成的随机数

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibouevibhYVM9GzGzJRYXVNzKBUgUQMoTYpLzmjFNEkVFiaibxD50ko6iccVmpUjULHcBNLsH8jsvrVRucg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=40)