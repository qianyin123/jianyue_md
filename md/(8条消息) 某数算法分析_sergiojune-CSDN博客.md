> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/sergiojune/article/details/104421946)

今天我们来分析一下某数的js 很多网站都用的，听说是有好几个版本，我也不知道，随便找一个，因为我们之前分析过，那是直接怼混淆，大家也许有点懵，这次我们来细细分析，此次干货多，大家慢慢品。

首先打开页面：  

![](https://img-blog.csdnimg.cn/img_convert/6924b8583a0975fb966f8eda4971342e.png)

这个我之前说过一次，现在再说另几个方法绕过，  

![](https://img-blog.csdnimg.cn/img_convert/682f5b41c30ff003312da2e8b7125ed2.png)

第一个是条件断点，也就是说满足条件会断下，不满足就跳过，

第二个是不断下，也就是说这一行设置好就不会断下来，  

这两个操作前提是js不会变化，因为谷歌的v8引擎对js有优化如果是同一段js他会当成一个来执行，所以循环生成

```
(function() {
    var a = new Date();
    debugger ;return new Date() - a > 100;
}()) 
```

那真是一点用没有，都会优化走第一个生成的，除非你每次生成不一样的我们从引擎角度来分析大家就豁然开朗。

![](https://img-blog.csdnimg.cn/img_convert/a51bdc97ac1c27350b70d288d0791890.png)

现在我们两处都禁止了，然后反过来看主页，我们从头开始看

![](https://img-blog.csdnimg.cn/img_convert/ccd2e145b71c44142a699508d79262ec.png)

我们看到他会加载一段js然后在执行下面这一段，我们看到上面还有一个<meta content=这个标签我们再到他渲染之后的dom看看

![](https://img-blog.csdnimg.cn/img_convert/0a62d10c4aa5eb43004591a3314555cf.png)

奇怪的是他渲染后就消失了，我们先看他加载了什么

![](https://img-blog.csdnimg.cn/img_convert/9f5e037d6104c0c987bdb6f666d06b1b.png)

这个js给win设置了一个对象里面设置了字符串。

因为我们之前被debugger拦住了，所以我们看一下他的文件

debugger文件都是动态代码，动态意味它使用了eval和Function 两种方式，这个你们自己去搜索，首先呢

![](https://img-blog.csdnimg.cn/img_convert/ff843a63fa733b91ac9665204793e686.png)

![](https://img-blog.csdnimg.cn/img_convert/e9d1654de4e4a94a95d71e5b7231ff37.png)

我们分析到现在，发现了两处有着大量字符串的地方，这两处极有可能是两段解密后eval执行，根据我们的推测那下面这一段就是解密执行代码了，大体看一下。

![](https://img-blog.csdnimg.cn/img_convert/b578d79a8d042063d00a27d363ca13be.png)

那是非常眼熟啊，就是开始加载设置的那一段js字符串，我们搜索一下看他干了什么，

![](https://img-blog.csdnimg.cn/img_convert/d23ef4247bd86218f966fac097a14c29.png)

搜了一下发现好几处而且我们看到了execScript和eval

我们猜测他这个_$AI["6ca01ba"];拿到然后解密然后eval，我们发现这一大段都是在拼接代码

![](https://img-blog.csdnimg.cn/img_convert/44163ea3d2d0b590accc49dc171147ca.png)

![](https://img-blog.csdnimg.cn/img_convert/70719828fe707f657f36463cc54813e3.png)

到这里我们可以简单推测一下，他是拿到_$AI["6ca01ba"]然后解密然后和他的代码拼接返回一个js代码段，然后运行再读取content再解密运行，我们推测他的大体流程就是这样，

![](https://img-blog.csdnimg.cn/img_convert/57a524504fa07b1ba5685e3c1ec85241.png)

当我们看到这个那他基本上就是用的eval来运行的第一层代码，这里他判断了functioneval(){[nativecode]} 这是判断有没有被劫持，我们接下来搜eval看看他那里运行的

![](https://img-blog.csdnimg.cn/img_convert/d9370aeb8571573f2ba9952d06750c50.png)

_$cm 这个赋值了，当我们搜_$cm(的时候并没有任何东西，一般js调用除了_$cm()还有call和**apply  
**

![](https://img-blog.csdnimg.cn/img_convert/5c44541b49fd1366aaf395d59adab4c8.png)我们发现了一个调用，因为刷新页面就要变，所以我们保存一下网页，然后代理加载调试，

![](https://img-blog.csdnimg.cn/img_convert/fbf4d8c92dd236024c2bd7b3e9799458.png)

![](https://img-blog.csdnimg.cn/img_convert/202b2b6d6adcfb5628764e2a4f3112a6.png)

![](https://img-blog.csdnimg.cn/img_convert/44e650e58767723038b0611b0b180374.png)

我们分析的没错，他eval执行这一段然后再读取处理content再解密执行。

![](https://img-blog.csdnimg.cn/img_convert/d9b56048c5ced2059ffdd7e59ba38806.png)

既然他这里是eval我们是不是可以把他解密的数据放进去方便调试呢，我们试试  

![](https://img-blog.csdnimg.cn/img_convert/adc80264c83ddc324c53191cec2896f8.png)

可以正常加载，由于篇幅原因我们下一次接着讲