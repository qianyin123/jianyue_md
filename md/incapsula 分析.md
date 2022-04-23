> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-s0fafxe5fGKYrg4-Qi67A)

  

##### **该文章主要提供交流学习使用，请勿利用其进行不当行为！**

##### **如本篇文章侵犯了贵公司的隐私，请联系我立刻删除！**

**如因滥用解密技术而产生的风险与本人无关！**

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/qZWs8x7TdWppQ30r2GFI3v76aq0ia8ia2z6Z2IT38qhIys9jCtEyBQCbS72d7iaNj4QucwHAbRiaVZSiaQXHlwcoaTw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

三个月没打开微信公众号了。。。  

  

距离上一篇文章5s 已经三个月了

  

那我们就来聊聊之前5s文章里面提到过的 incapsula 

  

可以百度一下这个东西。此处不多介绍。  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/qZWs8x7TdWppQ30r2GFI3v76aq0ia8ia2znx8ZmeEb9z12IeEgZiaqlJ3cqQzAXNfNduVdY0HRs9GqZ8CEWeF9rEw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

有兴趣的同学可以根据这个网站进行学习。

YUhSMGNITTZMeTkzZDNjdWNISnBZMlZzYVc1bExtTnZiUzVoZFM4PQ==

  

清空cookie刷新几次，就会看到 incapsula 

  

  

我搞这个东西，也是我三月前搞得。。  

我就不调试了，，，折磨。。。。

  

所以，下面全是我本地代码。

  

- 流程。

     请求首页。获取_Incapsula_Resource的链接！！

    请求上面链接。获取一份eval的代码。目的获取cookie ___utmvc  以及 src

    get 请求 Cawdor-asse-my-Nightning-we-from-Dealell-Come-Ty 这个获取一份4000左右的 js代码， ob混淆

    上面js 获取  15000行长的参数

    然后post请求 Cawdor-asse-my-Nightning-we-from-Dealell-Come-T 这个，然后获得 名为 reese84 的参数。

    然后携带 请求首页，获得数据。  

      

  

此处我们看第一次eval的js  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/qZWs8x7TdWppQ30r2GFI3v76aq0ia8ia2zfugyng5EHaRuibHvicOQLBe2pLTsibnqBxtFnZvmeIO8H41UNibJyiaibXkQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

那我们先看看eval之前是啥样子！

直接改成

```
for (var i=0;i<b.length;i+=2){z+=String.fromCharCode(parseInt(b.substring(i,i+2),16));} console.log(z);
```

  

然后得到 800 多行的ob。

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/qZWs8x7TdWppQ30r2GFI3v76aq0ia8ia2zQTfFBJN8tPyRurwXa1rZVU3icCXRriaiaZmX6Rc6bDcXAoAuDBDcDXicRw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

里面就是检测nodejs的一些特征，toString  之类的js特征！！

  

然后就会得到。 名为 ___utmvc 的cookie

和 src ，，据图src。。我忘了在哪里用了。哈哈哈哈，尴尬

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/qZWs8x7TdWppQ30r2GFI3v76aq0ia8ia2zYZAnaRgq4ASO1hqZayzH7y0LpZoamibQVtvGNylJjDNJRib5iaqrHwzHg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

然后下份js。

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/qZWs8x7TdWppQ30r2GFI3v76aq0ia8ia2zvuz51xhDAbuWJKevXc58qJ6PyUgIMicuzPgPChA8vePthP0m9hWVzRw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

大约4000行左右的js。  

  

嗯，，直接补环境好了。。

毕竟我不是小和尚。。卷

  

这份js是事件触发的。  

不是直接运行的。  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/qZWs8x7TdWppQ30r2GFI3v76aq0ia8ia2zrltCfcdhibq2IJjluw7VrvOWVgAiaVCYWghlFqXa404SulnKMY4A3T0g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

此处，你可以在事件函数处，下个断点，，看浏览器会先执行哪个事件。

  

或者哪个事件是走的我们加密的流程，。

  

然后事件搞定后。  

  

所有的流程都在这个里面。。。

  

细节多到恐怖。

但是幸亏可以一步一步的调试。。。

如果是vm？？？  

哈哈哈哈哈哈啊哈哈哈哈哈。

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/qZWs8x7TdWppQ30r2GFI3v76aq0ia8ia2zXiaTTeSu4x9Am6LkJfoO9IsdQhaS8wU0KJibaktmFO62lEhBHTLMTcVA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

所有的流程都在我红框的框住的函数里面。  

所有的环境都在里面。  

  

里面大量校验了canvas 2d  以及 canvas  webgl

其他都是一些正常的校验，，toString  Object  

  

还有就是现在经常见到的主动抛异常。

try的逻辑是错的。

正确的走catch

然后获取你的报错信息。  

然后检测你的报错信息，以及报错路径。  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/qZWs8x7TdWppQ30r2GFI3v76aq0ia8ia2zXibVL2aCicJKnW4AKwsdwgoWWjCO6D7VFUibdPyeYFUGSYGgOmmclloEQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

然后就开搞吧。。

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/qZWs8x7TdWppQ30r2GFI3v76aq0ia8ia2zZX1TavE9icOZ8RNtdrvdsBPrtmEY3yK9hWmL7al2yziarxdJx4f8TOTQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

记得最后发包校验。成功后会获得 一个 token。。这个token就是cookie的值reese84.。

  

  

记得关注公众号。  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/qZWs8x7TdWpO4ObjI8yxX1khOSs70ZscCw0Tp3BYicyJcGhYD7T7ju2mbLP2yw3H9YDER5J0LSsjYem4j2SOtIg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)