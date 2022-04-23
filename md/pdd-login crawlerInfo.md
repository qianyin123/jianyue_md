> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hOw2lVb4Eps9TqXWgrYzGg)

> login - crawlerInfo      个人笔记

目标网站：aHR0cHM6Ly9tbXMucGluZHVvZHVvLmNvbS9sb2dpbi8=

> 1，定位

直接搜 `crawlerInfo`，可以看到 `getCrawlerInfo` 方法， 这里可以直接控制台搜索，或者`xhr`断点之后直接搜索，文件很大，忍一下， 经过`webpack`打包了

其实这里定位是取巧了，如果方法名稍微改一下，混淆一下，我很菜，就定位不到了， 来个大佬指点我一下吧![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKrSicib8TpWt6F4NH3BytY8PMqqxibIia3D8SmibQwkMrWNBYaSticdqficlHQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKrSicib8TpWt6F4NH3BytY8PMqqxibIia3D8SmibQwkMrWNBYaSticdqficlHQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKrSicib8TpWt6F4NH3BytY8PMqqxibIia3D8SmibQwkMrWNBYaSticdqficlHQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKCibOZAHc8owS4AwPM7bAW3sMjU5lKgYZNmQvnysAgYsL5m8hO14vbwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个`then`，其实就是回调，前面执行完之后再去执行`then`内部的逻辑， 注意：then()方法是异步执行

`t.getServerTime()` 其实就是获得了`13位时间戳`

那主要逻辑就是`B`了

> 2，B

进入`B`之后，可以发现 参数`e` 其实就是上面的时间戳

可以发现 `n` 就是加载器，加载了大数组里面的下标为 `153` 包

`new` 了一个对象，将时间戳传进去，并调用了 `messagePack` 方法

![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKYzhkajwGNWZZdzbibjr7xMkXDyLnoAOczYYk3GMxLRKnicM7j0y6ZfZQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> 3，new n(153)

单步执行，经过加载器之后，就会来到这个地方， 可以发现 `f("0x1b", "bpr9")`为`exports` 可以理解为暴露出去的一个接口，导出方法

注意：此处使用了`逗号表达式`, 其实是`return` 了 `De`

`De = new Me`

![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKts4kSBmfh24cTNutBJiaI5GItBGkWjhZJJuR1HnDvIicNZMRc70jFCsg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> 4， Me

继续单步，可以发现其实就是执行了`Ne`

![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKiaJJAKPmrcV9AvS5qeKUk7uqiaSgEdnXicAXYJibvYMUVEeWdeicbIjj3yA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> 5，Ne

进入`Ne`之后 上面在初始化对象赋值， 下面开始计算，这些地方要小心，检测了鼠标的点击位置，以及点击的停留时间，我感觉这个停留时间应该检测了你是不是在`debuggger`状态

![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKX4k7ARDSqQuyxomru9IaAoiclhHqqtFfMwvMhEByOoR9vyQdlxbtRZQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

`T[_]`我是直接写死了，也可以在区间内随机

![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKeBOLWpTZsMloNYiawsTBSlPARvX7fsia33MSD4DFsWrP8chALHyoQymA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

别的好像也没啥了，整体复制出来，缺啥补啥就好了， 有独立的加载器，那我们只需要把有加载器的上一层复制出来就好了， 稍微改一下，全局导出加载器

![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKuLfOhYSUcr8RtSFazbPryJjF72z8ibEegFPvrnEurAJD9ibqmJDicfBAA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

补环境代理：[Proxy代理（二次修改）](http://mp.weixin.qq.com/s?__biz=MzIyMjQ3OTE5MA==&mid=2247483738&idx=1&sn=1d53007e7805d88e5841a582718f0e16&chksm=e82d9763df5a1e75e59b27c3b6772b78eeb2b9ccd219df09362b8a4ac7f455766d8e76f6b362&scene=21#wechat_redirect)

补完环境之后，如果加密结果和浏览器加密结果长度不一样，先不要慌，写一下代码测试一下自己生成的加密参数可不可用， cookie的地方可能会稍微有点问题，注意下就好了

总结：三千多行，代码量不大，难点在于异步的定位

![图片](https://mmbiz.qpic.cn/mmbiz_png/tQ4OibXVicnhnsd4wwfqAcO9nnBl9CibVmKiaEaQsiaMmFGTibI0vSMA0ZiaK9BU2VO7x5m1A8lbaeB3FWqicIvBJMYL9A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)