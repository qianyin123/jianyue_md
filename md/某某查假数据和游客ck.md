> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/FOo-xNThwIMA7ExFCoc52A)

    最近某DN网站删文章蛮凶的，目前有两篇去年写的文章已经GG了，搞事情，...废话不多说，今天这个问题主要是总看到有些群里问到这个，看完之后你也根本不用去扣什么js，还原什么python代码了。  

问题：xhr返回数据中的只要是数字类的与真实的数据不一致导致不能用，包括链接的id，日期，地址里的号码等等  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/OUuGDHj1ueGHJicU12P210BzBpCibbOxdQJNZGNScQWAb6Oo1enaBEMCPovxDbDJNibV0GRKIB9hFhebBmeib3nC7g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

断点找问题  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/OUuGDHj1ueGHJicU12P210BzBpCibbOxdQ3sMC8MV8HgrtcmqrneZuOlu6BmTPZiaLVzxFyKcpyLZPdCFhqz19tiaw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

异步的话直接追内部，这个就不再细说了，找到如下加密位置  

这里可以看到t其实就是xhr返回的数据，这里提前注意个东西t.ddw，后面要考，之后主要就是S这个函数咯，追进去看下  

这里又要追什么呢，啥b、A这些函数方法，其实可以不用盲目追，看看生成值是什么，直接用返回值替代函数方法即可，没必要扣那么多，挺累的，我们看看b()(t)  

这里用到了两次，返回值其实就是返回值中字典里的key值，因为xhr返回的key值是固定不变的，所以这里没必要去扣js。  

只剩一个A函数咯，接着看  

这里需要注意两点

第一点，为什么我们b()(t)有两个值，因为上图又调用了S这个函数，所以两次S函数中的b()(t)的返回值不一样，这里自己稍微修改下将b()(t)的返回值当作参数参进去就行了  

 第二点，s = T["code" + e]，T是啥

返回值既然是固定的，那也没必要去扣了  

这里如果正常的有哪些做法，正常扣各个方法，当然你要扣各个函数的话这个网站是可以直接wbpack的，要么就像我一样，看看方法返回，能不多扣就尽量偷懒

最终结果

上面水完了，讲最直接的，之前让注意的t.ddw的值，可能是1或者2，看你xhr返回数据的实际情况，然后对应到之前的T的值（固定的）

然后看code+t.ddw，对应去将返回数据里的数字对应还原回去就是了，就这么简单，这里追流程的时候就把它的套路看得清清楚楚了。

最后附上游客的ck  

js爬虫版

python爬虫版  

也不难，就MD5这类的。  

大家努力工作，让老板尽快换车换房

喜欢的就一键三连啦~~~