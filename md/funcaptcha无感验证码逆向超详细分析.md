> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/s1jD4CErXxqtOujyURSOGw)

本次分析对象: 格罗航空 https://www.zipair.net

1.前言

    先上个正常的func有感的样子  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmfr4EOiaXswfXI3dnhO3HvOgGbbia6yKgpU0vCGxiaoUs7dtzvU0Ly2etA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

    再说说无感验证码， 当前浏览器环境及IP没啥问题的情况下，通过js加密算法生成的参数 提交请求得到的一个token去验证，通过后前端没有任何感觉，反之浏览器弹出上面的验证码 必须通过选项验证。

    **接手了一个网站 听说是这个验证码 以为没得搞(毕竟图片更新快 打码平台都难) 后面听****LeeGene****说是假的(无感) 可以逆向过去后 开始着手分析一波~**

2. 无感流程查看

        2.1查看发包，所需一个captchaToken

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmWShUs4kmyLNmYuonsiaO9jvjQv1AKibxOFDBS65APHFH3AiaSRtibEwKsQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

直接搜索 这个token是上面接口 public_key/D4643E9F-7FF0-4299-9FE7-095F928F84A1 返回

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmqztd8qWia2mV7Tln5VpAgTD4NCYoUQbBmUGMMtdeP23zkgQVklPFPMw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

    先说一下这个接口所需要几个参数:

        bda(浏览器环境加密后的值),

        public_key 网页固定

        data[blob] 上面data-exchange接口返回

    很明显该找bda参数的生成过程，开始分析

3. js逆向分析-bda参数

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagm1QycIE1icdsMpQfvHGVGoftZJMgiarp6jqqEB8Ej1uh7ENluBu71Fwhw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

全局搜索打开(快捷键ctrl+shift+f/⌘+⌥+f)搜索bda

一共就搜出3个bda 全部打断点，刷新网页

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmOtSqtwErHwjk9peufviaYOjmicqdc7iczs1h7wH0rCWHXCVNdwq5yYy5A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

断住发现bda = e.fpData 好多需要的参数都是这个e带来的，e是上级函数传过来的，以及上面的代码可以看出 这又是个经典的异步框架(混淆?webpack打包?)，switch case一直跳啊跳 通过sent拿上次值的方式，看的太多了 不知道叫啥 随便叫吧。 

该找e了,找e的fpData方法有两个，1直接全局搜fpData下断点 然后刷新等他进来分析，2往上跟栈点几下也能找到(给的arguments容易被忽视)

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmgUreDIia16QsvedWFC3k2fiblG3dm12Hp81kbHKpwmzMcJ5MjMa1yoXQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

tips:这里看到的switch 可以加个日志断点打印它的.next来查看一下过程

到上级的for循环地方发现 case 0是直接判断or有没有fpData的 说明还要找，直接搜索 "or.fpData = " 成功定位到，继续下断刷新

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagm5JGD06omkv9B3jfskcKOP4maib0mPy7B6dDCCgetpcEXjUUK5F0Q3bA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

又是一个for switch case 直接进mn函数 里面就是生成fpData的  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmK0sKvU9rU3HkEaT8kYsrBdpvJmH1ByDUwYymxDZ4MsLuHvmwsxLBIg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此次进来发现代码有点轻度小混淆了 这次只有3个case  这个a, c, u, s, f都是生成一些小东西的 Me不用看，直接跳到case7 就能看到这几个小东西了

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmCsYCuUTKYq0CrRP9cgggwCQ8CJBx8L5PoWvsQLYcqXWicqu2K70qLuQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

然后先不看这个小的，把mn函数代码折叠一下 最后需要s 和f这两个异步的结果 f就是上面看的那个小函数，s=gn()是个好东西 所有的获取浏览器环境的都写在了里面

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmLLMlDicmfwNO82yibsj9tMlQv7KujiaXOIe3Myot2dVickJnmRvvoujmng/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

gn函数 有兴趣的自己去看下它获取了浏览器的哪些环境 这里不多看了(图中代码折叠了)  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmd4fiazJLZEicsbDIiad0rGuXfQic2icNUXMgc3vDLrmMKwtswsPxTawedBA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmYybn0riapWS9YdricaaFGc2Pbje3sErLe9oT60OMo1nhERya9TKicicib6Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

最后看到 要的就是这些东西，至此 fpData分析完成，开始分析把fpData加密成bda

查看之前搜索的fpData，有判断if(or.fpData)的地方，下个断点调过来

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagm3TvYcfo24orrvyrGyYibicggHgaSib70fxy86qCIicTWWUEn3icVQ7z5xGQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

往上翻一番，还是经典的for switch case 

前面的case代码都是废话，直接看case10最后return处，拿fpData的一些东西生成另外一些东西，再经过qn函数 生成最后的bda了

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmaBjQke3O5RofK4FrPEoTWMOD2PhRprgMnCQIcEvlAEM1PbicxiabUaTw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

qe单步就能走到最后的大boss地方

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagm7Ug9mRGf8Mw9yQmDx3yNQ2ibjLre7dyMPpXEE89l5Q7z5t1iaaXvOZtw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

懒得讲最后的一个qn加密了，根据上面的经验很容易看出这个qn干了些什么 根据case走一下，也留给自己动动手脑

😊 直接贴上最后的加密明文图 

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14fNRADrL9hlhrQRfLoSKiagmU08HrxKytWa9iaVT2T4MBBH6RRuV6pqcpkkb9GE6YI1Kexia3gFyJaIg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

生成bda后基本完事，发包拿token。完结~