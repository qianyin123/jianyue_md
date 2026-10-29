> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MBPH8QOmvfdkNt3y09pD0Q)

**本文章中所有内容仅供学习交流使用，不用于其他任何目的，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！若有侵权，请联系作者删除。**  
  
  
每过一段时间就要奖励自己一个逆向分析，分析完这一个网址，感觉自己的逆向水平大概算得上入门了吧，虽然比较费时间，但好在最后是成功了，现在把详细分析过程发出来，相互学习交流；  
  
网址地址：aHR0cHM6Ly9wYXNzcG9ydC56aGlodWlzaHUuY29tL2xvZ2lu  
打开浏览器调试，刷新，随便输入账号密码，出现滑块验证，验证成功后，查看数据提交过程，最终锁定了几个关键网址：  
https:// xxxxx/api/v2/getconf  
https:// xxxxx/v4/j/up  
https:// xxxxx /api/v3/get  
https:// xxxxx /api/v3/check  
https:// passport.xxxxx.com/user/validateAccountAndPassword  
  
##[#1](javascript:;).先看第一个https://xxxxx/api/v2/getconf  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUXMCU6Gu5MVrjCB4xe4A3a61IemDmzE85zylGrWU774u1jNYo3c1FdVzic3l9sALNp1oQLgnzaTqDGwMibG1JFLyfR6GH4Wkntbo/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

  
  
直接搜索关键字getconf很容易找到参数赋值的代码，  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUU3QEV4SYmWeib2LWObfjg5Or1TO3ia5FYFCLx2iaYXo5OjyWaV9IPE70qUwOzBbwQrmCt9TXiblB1heSlVRh2RuqwGtORWQfl9uy4/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

  
  
打断点调试发现，R就是其中4个参数，这里我直接用这4个参数请求了一下，

```
`url = "https://xxxxx/api/v2/getconf"``params = {` `"id": "75f9f716460a422f89a628f50fd8cc2b",` `#initNECaptcha中的预设值captchaId` `"ipv6": "false", # 未定义` `"runEnv": "10", # 预设值 WEB: 10,` `"iv": "5", # 预设值 v.IV_VERSION``}``response = requests.get(url, params=params)`
```

出现了返回值为

 callback isinvalid ，然后搜JSONP直接就找到参数生成的位置：  
var l,p, f = Math.random().toString(36).slice(2, 9), h = a.prefix ||"__JSONP", d = a.name || h + ("_" + f) + ("_" +i++), v = a.param || "callback", y = a.timeout || 6e3, m = window.encodeURIComponent,g = document.getElementsByTagName("script")[0] || document.head;  
简化一下就是d ="__JSONP"+ ("_" + Math.random().toString(36).slice(2, 9)) +("_" + i++)  这里的i是当前网址的请求序号，从0开始，所以取0；  
使用python编程：

```
`import random``import string``I = 0``def callbak():` `global I` `m = ''.join(random.choices(string.ascii_lowercase + string.digits, k=7))` `result = "__JSONP" + "_" + m + "_" + str(I)` `I += 1` `return result``params = {` `"id": "75f9f716460a422f89a628f50fd8cc2b",` `#initNECaptcha中的预设值captchaId` `"ipv6": "false", # 未定义` `"runEnv": "10", # 预设值 WEB: 10,` `"iv": "5", # 预设值 v.IV_VERSION` `"callback": callbak()   # 随机值``}`
```

  

直接请求成功：  
__JSONP_ztyfoeu_0({"data":{"dt":"W93/vPVL……..net"]},"error":0,"msg":"ok"});  
好像不需要referer、zoneId和loadVersion，“referrer”就是网站网站址，“zoneId”为空不用管，  
我还是找了一下"loadVersion":"2.5.4",  是预设值代码赋值r.loadVersion= t._captchaConfig.loadVersion；  
第一个网址分析到此结束  
  
====================================================================================================  
  
##[#2](javascript:;).接下来是这个网址 https://xxxxx/v4/j/up  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUVztXNBHNhJKJuc2S1PePeJRB1eWZYqSHL3qv65TwfibGVIJDc2QJcZYyHa8TNMO0icMusldibqM4LmfvbsZxwDjnPGgP0YYGOp9o/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

  
  
总共5个参数，首先可以看到p在上一个网址请求的结果中，  
p = result["data"]["ac"]["pn"]，  
这里搜up显然不管用，搜yanzhengma也没戏，搜了下vk，很容易找到参数位置，  
  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Gv6JExJQjUViblxPKGXXkaep6ibxnRehGiaFL7H1aQjSdcSsibX9Cjeq1HgKJSNgN5F974b1LlUBXCgezIeIbiahPc5LQ2K2L3JlyleBEaZ1DROk/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

  
  
还原一下代码  

```
`p = a = (h = n["context"])["appId"]``vk = s = h["versionKey"]``v = O = h["sdkVersion"]`
```

  

O和s

感觉像是预设值，然后我根据调用堆栈找了一下源头，发现最开始赋值的位置，  
  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUWfIqIb7SP1Hia7aFEib0wJWia5HLw0gOAqkuBicT7uzjM67SPNdrtibEG13j7ichWwZf97sd6wbIQrDylVzf8rz2TMqQWSYRdP3Plfg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUXOvpmsJRiaHcP5ndZzdP1jb2oGPwNUsLA4icMYFpicnk4kI9fs5BB7uendduzmZ1t9Gg7GVX6wa6VR7o1nicpfGRngCpiccxSu4UuE/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

逐一进入函数发现各个参数的赋值：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Gv6JExJQjUVfcspeFVib20FcIY13on8vf2hJIpBrgVYKAj0jfX0tgPpiaggLbXvnm0ia0YODLR21cADjHEmRKCuiadZu7JabTYkjLZkcm39tDtk/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

  
  
还原一下看的更清楚，

```
`function n(n) {` `var t = {};` `t['cacheKey'] = 'ntes_utid',` `this['appId'] = n,` `this['collectDuration'] = 0,` `this['visitDuration'] = 0,` `this['nonce'] = "",` `this['accessInfo'] = 'init:1-gts:1',` `this['timestamp'] = "",` `this['visitTime'] = O(),` `this['collectTime'] = 0,` `this['tkCacher'] = new hu(t),` `this['cacher'] = new hu({` `cacheKey: this['getCacheKey'](),` `simple: !0` `}),` `this['sdkVersion'] = '2.0.13_yanzhengma',` `this['versionKey'] = 'd44593ca',` `this['sdkType'] = 1` `}`
```

这里的n就是上一个网址提取的result["data"]["ac"]["pn"]，然后vk和v参数也知道了是预设值；剩下n=G()，d=v()，  
直接打断点然后控制太看G的函数位置，得到G()：  

  

```
`function G() {` `return 'xxxxxxxxxxxx4xxxyxxxxxxxxxxxxxxx'['replace'](/[xy]/g, function(n) {` `var t = 16 * Math['random']() | 0;` `return ("x" === n ? t : 3 & t | 8)['toString'](16)` `})` `}`
```

  

Python写法如下：

```
`import random``def G():` `def replace_char(n):` `t = int(16 * random.random())` `return format(t if n == 'x' else (3 & t | 8), 'x')` `return ''.join(` `replace_char(c) if c in 'xy' else c` `for c in 'xxxxxxxxxxxx4xxxyxxxxxxxxxxxxxxx'` `)`
```

  

  

d=v()也是同样的方法，稍微还原简化后也就300行，所以这个解析的过程我帮大家跳过了，我就把主代码放上来吧，

```
`def v(n):` `t = N('fd6a43ae25f74398b61c03c83be37449')` `r = [Z(math.floor(256 * random.random())) for i in range(4)]` `t = A(R(t),R(r))` `t = R(t)` `i = [t, r]` `u = i[0]` `e = i[1]` `c = N(crc(n))` `def inner_func(n):` `if not len(n):` `return l(64, 0)` `t = []` `r = len(n)` `i = 64 - r % 64 - 4 if r % 64 <= 60 else 128 - r % 64 - 4` `P(n, 0, t, 0, r)` `for u in range(i):` `while len(t) <= r + u:` `t.append(0)` `t[r + u] = 0` `P(p(r), 0, t, r + i, 4)` `return t` `def outer_func(n):` `if len(n) % 64 != 0:` `return []` `t = []` `r = len(n) // 64` `u = 0` `for i in range(r):` `t.append([])` `for e in range(64):` `t[i].append(n)` `u += 1` `return t` `o = outer_func(inner_func(n+c))` `f = [] + e` `v = u` `m = 0` `h = len(o)` `while m < h:` `def func(n):` `t = [q, K, x, V, Y, W, H]` `r = '037606da0296055c'` `i = 0` `u = len(r)` `while i < u:` `e = r[i:i + 4]` `c = B(e[0:2])` `o = B(e[2:4])` `n = t[c](n, o)` `i += 4` `return n` `def func1(n, t):` `i = len(t)` `e = len(n)` `r = []` `for u in range(e):` `r.append(Z(n + t[u % i]))` `return r` `a = A(func(o[m]), u)` `a = A(func1(a, v),v)` `v = S(S(a))` `P(v, 0, f, 64 * m + 4, 64)` `m += 1` `return M(f,list('MB.CfHUzEeJpsuGkgNwhqiSaI4Fd9L6jYKZAxn1/Vml0c5rbXRP+8tD3QTO2vWyo'), "7")`
```

  

我们来分析一下它的参数，它传入的参数是一个列表，

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUUjdj3BC6GQDkLW7FYGJQJk3CjBz3zO6aOVaVDlzcQQVStWrnYeiaDSmd5KJY0PmCE5KibV3epDeBrlGqmShV0ZslotocBGfNXhI/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

  

  

意思是t+o+f的列表，o和f的赋值在v=前面有：

```
`c = m['sent']()``o = c[0]``f = c[1]`
```

  

所以要知道c值，然后看到m函数里面，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUUszqTQbOGTeEiayf7JngPclvJv0RsD4MSP37iaHe189zDTKcPnPBSdu8cJ5LR5ia3YBQCA0CLHbBItlg4kTvoXjj4SRJJW9uK7WU/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

  

往上翻一下发现e的赋值跟r有关，而r是传入的参数

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUW4rOTTwZgOwZVoQjAOhSdb2wxIqbKIpg7AE16vBFRyK516dZAANwQlUhicmOTaN0pZ6mVGWNHfqT3T9fT4W0LmrnKzKGGp5ccQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

  

然后走一遍看e是如何生成的，接着根据调用堆栈一路往上追到了这里，发现到头了；

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUV9csVGTxUIDe9756Vaj9eHK8qte29FxwM24MyEexCKMOLJPGMOzbLMs7GGPuSbmtiazdsg40EpM2o0mKEc3VLqlBJrmDTLDNlk/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

  

f的值是o[“value”]，而o是c的next下一个，然后就找不到这个数组从哪来的了，但多次调试发现他是预设值，所以大概是初始化的时候就已经生成了；然后我某次刷新的时候发现断在这里的r运行多次后会有变化，不知道是多少次，反正挺多的；

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUWnvfm8mVoiaY8VicFib1iat3yibN5ibr3BgGg5dAh9peIYBH4ffcNQFHdp09pSAic8dyEJsU332vBkJZlPtgpNYLbsPCKPiagEqv9wE0Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11)

  

然后这里的值函数内运算后会变成数组，然后这个数刚好就是Array(75)其中的一个，就是这么幸运，

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Gv6JExJQjUX1PZCtOfF2Vmu3egQotoLwibPyfDt359Jh40vewichM3WqtGnOTxNuUXlibYlibKakqvAj94CGOUCIzFylITP3wqUrjWWA1EicnRbw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12)

  

所以接下来在这里单步调试，发现是在这行代码中变化，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUXzvbxv5Juu3918DRSVO7obM2qty21gbPQXXA9Mmzpeq3UZGr9YicUPapXwEjLkt7CsWdibRpC36wiaZuvtia4nPXM1fUc5euPgArA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13)

  

单步进去看到这里有个变换，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUVFtN4Megngs5oFhLyppbia6LqGwMUib1AH4Ojoo8FfeQqnsBy01iaXfppzyQefceolIqmLicg9tFFQAHomfZ74RPgnvjr87W7p8oY/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14)

  

然后看到上面闭包有三个参数，于是搜了一下t的值254，

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Gv6JExJQjUW38P15QJzepWw5r46pmpvSN7zXibpYYEu2VHrHaFpnTicVC8uibVqKHQDb3kKHcgZODHMN5MNL6MGuXHic4Npn1UUtT2nKGat8BC4/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=15)

  

结果发现这里三个字刚好对应n,t,r；

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUWYibAImyzKDu6YkqJsN3bGe2rrNnoyFrNGQoc4jBDvyap0ic4J7iaSiceUdShTNEMia4vJC9W0e4qMwcpDNsQYsr1ibAtCiaWxoLorps/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=16)

  

然后在这里打上断点刷新看他怎么走的，这里确实有点烦人，又是异步又是混淆，好在经过我的不懈努力，最后把代码简化成这样：

```
`# 关键函数简化后``def L(n, t, r):` `u = r(0, n())` `p_t = p(t)` `p_len = p(len(u))` `return p_t[-2:] + p_len[-2:] + list(u)``# 测试1``def Vt(): # 这个是传入的第一个参数，只需要返回值` `return 2``def tn(n,t,r):` `def function(n, t):` `if n == 0:` `if t:` `temp = 1` `else:` `temp = 2` `else:` `temp = n` `return D(p(temp), r)` `result = L(n, t, function)  # 虽然都是调用L，但每个function都不一样` `print(f"测试结果：{result}")``tn(Vt, 218, 1)  # 三个参数对应``# 测试2``def jn(): # 这个是传入的第一个参数，只需要返回值` `return '48000,2,1,0,2,explicit,speakers'``def un(n,t,r):` `def function(n, t):` `if n == 0:` `return D(N(t), r)` `else:` `return D(N(""+n), r)` `result = L(n, t, function)  # 虽然都是调用L，但每个function都不一样` `print(f"测试结果：{result}")``un(jn,252,100)  # 三个参数对应`
```

  

随便测试了两个用例发现没有问题，传入参数需要用到的其他函数需要单独扒；由于这个Kr()里面数组太多了不想一个一个简化分析，不过大致思路都差不多。分析完Kr然后我们回到之前t+o+f的列表，发现其实出发点有点问题，但结果好像不差，其实这个Kr赋值给了t，而o和f是Kr下面两个Vr和Wr的值，稍微还原一下第一个参数值，就能发现Vr中的第一个参数就是之前还原代码n(n)中赋的值，Wr是鼠标滑动相关的值；然后我们来看Wr：因为他是混淆后的代码，所以直接搜混淆，找到了这里，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUWM5AUh3iakcHeYLMuxCaPgjHI4BMaxLrYZVXiaXiar2iaF8Za7JfDib4a13mXLv0HTl0icuicT1DDyiaustXuT0aL1iaT5Oo0b9jztB7wk/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=17)

  

还原一下看的更清晰：

```
`function n() {` `this['moveCount'] = 0,` `this['clickCount'] = 0,` `this['downCount'] = 0,` `this['upCount'] = 0,` `this['motionCount'] = 0,` `this['orientationCount'] = 0,` `this['keypressCount'] = 0,` `this['focusCount'] = 0,` `this['blurCount'] = 0,` `this['scrollCount'] = 0,` `this['popstateCount'] = 0,` `this['trustedCount'] = 0,` `this['unTrustedCount'] = 0` `}`
```

  

13个参数刚刚好，而且这个Gu很熟悉啊，不就是前面Wm代码里的嘛，创建监听鼠标然后执行开始，后面还有个执行结束，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUUdFqKmTt4eO6MbUibsX6m5CFxFI41NS7Nwsibe4uIQ9698SDe9ORQGufuU83c2miaZ3kZYfBPSFgkpIK0icvYRrCyujvDicUDCicIico/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=18)

  

这里就凑齐了所有参数了，经测试发现全部为零也没有任何影响，我就直接拿生成好的三个列表展平为一维列表传入事先还原好的v()中，检查一下v函数有问题没，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUWzj8alI1LM2OVgJznyicAlAxVV7uRkI4Kl3BBWU7mT4kpImrJPgYoT4CCRUYDcJdWyuo36Ek1M6HibamL4OFUltSsGvPHw6ofBE/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=19)

  

  

成功获取！

{"code":200,"data":{"ed":"","ed2":"","es":"b32833xxxxx9def4c7","td":"bLR+GSxxxxxlzISnT1","tk":"c1lrkAxxxxxo5EstI","ts":178xxxxx196},"msg":"成功","ok":true}

  

====================================================================================================

  

##[#3](javascript:;).终于到第三个了https:// xxxxx/api/v3/get先看参数，这里这个referer其实一直没啥用，然后很多老熟人，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUXz9YLwx1ST022tIzxGPXzIb71vW0yCYqmku9MdYtyHicU9McpEJq2MyfvFnKuK3v71V45hCFy3O5Lz51YJnfNfs7V2I2TqZ9fQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=20)

  

那就随便搜一个把'sdkVersion'，找到一个相似度最高的；为啥不搜混淆后的this[y(724)]呢，因为已经搜过了就只有之前那一个；

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUXlVAzCd2gtyiaKPf4xLqJjsK2JjzM8nJxfXbjQEZicUSicZJ9qZy9ntoqQZSLWRWQG296OyAicNgNvMhNujF3RnIMaBkJWhOQkXp0/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=21)

  

  

  

id是之前分析过了，zonId和dt在getconf的返回值中和，irToken在up返回值，所以来看fp，发现他是fingerprint赋值，然后搜索fingerprint，找到了他赋值的位置，

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Gv6JExJQjUW3Q4k4TnVgFtQWwYS07Whu1na9kk5u74e0icrLaHlO8zS2vvich7r3wVz4XyD0n9upCCiatZUodypYFgL1MLXntR6kX0CZSFPKpg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=22)

  

然后就找不到这个参数从哪来的了，几经周折发现他在cookie中自动就生成了，实在是找不到地方，于是就解锁了新技能hook大法，

使用事件监听器断点，在控制台输入下面的代码，

```
`(function() {` `'use strict';` `Object.defineProperty(window, 'gdxidpyhxde', {  // 修改goldlog为你需要查询的window属性` `get: function() {` `// debugger;` `return "";` `},` `set: function(value) {` `debugger;` `return value;` `},` `});` `// Your code here...``})();`
```

  

最终获得了值的来源；

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUWWFvdjT2c4hNy9bEdzcJK7Kcw5sAluyAibuAMdUHSNxxOdSqbRGtVICgr09Gx3Se6gzoAKCt1WKAzNTJReSfCmPS4S3W5LtHmw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=23)

  

  

好了又是繁琐的分析阶段，跳过分析，结果就是元素参数就4个，只有u是变化的，对这个用了一系列加密，最后结果再拼接一个时间戳，

```
`dict1 = {` `'v': 'v1.1',` `'h': 'passport.xxxxx.com',` `'u': _0x3f6f3f(3) + str(int(time.time() * 1000) + 900000) + _0x3f6f3f(3),` `'fp': '418xxxxx53,3610xxxxx3142',` `}``===省略===``result = ''.join(chunks)``final_time = int(time.time() * 1000) + 900000``window['gdxidpyhxde'] = f"{result}:{final_time}"`
```

  

fp结束再看cb，cb也是一抹多混淆，但在我解的过程中越看越熟悉，这不就是之前的v()嘛，然后回过头一看fp的核心加密方式也是一样的，不知道为啥都用同一种加密方式，cb就用现成的，

```
`def cb():` `code = 'vfnv46'` `pos = [1, 10, 12, 13, 26, 31]` `uuid = _0x2b4c4d(32)` `uuid = list(uuid)` `for i in range(len(pos)):` `uuid[pos[i]] = code[i]` `return vv.v([ord(i) for i in uuid])` `#这里调用的之前的v``()``def _0x2b4c4d(n):` `m = list('0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz')` `t = len(m)` `rt = [m[int(random.random() * t)] for i in range(n)]` `return ''.join(rt)``print(cb())`
```

  

其他参数不用怎么管，比较常规，然后测试一下，

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Gv6JExJQjUWib1w2iajIQge4LTHY5ovvuoqGX8bicWTHn5Y2jBf1CIrciaYiaV8m4icoFkqLTq2gLQf0n4c86nJQMOAuWicNykicXrUEBNEibMWSmxpM/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=24)

  

请求成功！

  

====================================================================================================

  

##[#4](javascript:;).接下来是这个网址 https:// xxxxx /api/v3/check

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUWne7ZY61wex0d4zrDnYs914MibkZFhia0DwRO2DzaTpbVxFfXbKa5bKjsDicianqN1RRFDZ7HFTiboQ8VO79cgdxblokRxbiaQEUPB4/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=25)

  

  

dt、id已知， token是get返回值，cb后面查看发现调用的同一个函数，callback最开始分析过了，主要就是data中的几个参数，然后这里多了个extraData的参数，那就直接搜这个，直接就定位到了这里，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUVWlDCq6U2TcMGG37HxOXk6VKpLls4mfdssGysticzNgGsGicFQKEjsPuiaicgwLPPYgVZKmt25gQPoQKl3ylPeEz2mZQNeTpTLVuQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=26)

  

查看堆栈找到了data的几个参数，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUXaaIUT794LTHoNXl3buCP6oyia2cmhic8keyd8aniaoaVSbia7TZxtIO3ZRxVeJ45O34Gv7cpzHibDbLZWtAILk2TpjYNWGQeQmHN8/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=27)

  

参数源头都在上面，需要一层一层的分析函数，比较繁琐，经分析得知，

m为空，d、p、f、ext最外层加密一致，他们参数不一样，

d主要是两个参数token之前up获取的返回值，和单组鼠标轨迹的列表加密，

f的一个参数是token，另一个参数是鼠标轨迹列表，每组列表由四个参数构成，也要套加密，

模拟了一个鼠标轨迹，

```
`X_end = int(input("输入X轴偏移量："))``st = random.randint(60, 90)``tm = []``length = 0``dragX = []``X_init = 0``while True:` `temp = random.randint(2, 7)` `X_init += temp` `st += random.randint(6, 9)` `tm.append(st)` `dragX.append(X_init)` `X_end -= temp` `length += 1` `if X_end < 0:` `dragX[-1] += X_end` `break``dragX.sort(reverse=True)``Y = [random.randint(0, 10) for i in range(length)]``Y.sort()``traceData = []``mouse_list =[]``for i in range(length):` `m_da = [dragX[i], Y[i], tm[i], 1] #[拖拽X偏移距离，拖拽Y偏移距离，距开始的时间差，真实判断]` `mouse_list.append(m_da)` `traceData.append(p_data(token, ",".join(map(str, m_da))))`
```

  

p是token和滑块X轴偏移的像素，需要通过图片识别得到，图片识别就直接跳过了，本来想训练个大模型识别，奈何自身的技能点数不够，所以后面直接获取图片人工测试；

  

```
token, str(X_end / 320 * 100)
```

  

ext是token和”鼠标按下次数”+“,”+”轨迹列表长度”的拼接，套个加密；好了参数全了，最后测试一下，完美！

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUXRtUJHqdQGrKkknKibNES9jIekEC7BdxCftmttmfskFUlEqJaNMxH2dpS1NJFv5S1fJm77vhVkdeFaJg1ibNtoEYT3Cuf5Hwn7E/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=28)

  

  

====================================================================================================

  

##[#5](javascript:;).最后一个就不那么难了，https://passport.xxxxx.com/user/validateAccountAndPassword

  

参数就一个secretStr，

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Gv6JExJQjUUlEJtuZ7l7u02j2rbJp5b8ma7M27t0Segsk9wwvDDE7SAoSxqb3iagabmics38sdtqUYB4MjQ82xJcFkZIofcu9tdQneVK1wuKA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=29)

  

就一行，是不是过于简单了点，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUUXgDaY4iaOicEibvuqkCUPSETsibLfRZYTgyuuC2QLicXTicE4bH6Q73bI1xB3eWQHAReIfPBpBryDdibDNhtuRI9YZ5UtSpr4iaJibIA8/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=30)

  

转为python实现：

```
`import base64``from urllib.parse import quote``word = {"account":"xxx","password":"xxx","validate":"CN31_b.xxxxx7_v_i_1"}``word = json.dumps(word,separators=(',',':'),ensure_ascii=False)``encoded = quote(word, safe=':,')``secretStr = base64.b64encode(encoded.encode()).decode()`
```

  

其实word里面还有validate一个加密参数，搜索关键词通过堆栈跟，还是不难找，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUUKIzvGFvXjVnFF9fV1WlxVjmEeU8jex0ydbv7Abd2iaNjeM3jF6Zd2OyfvAY6cn62B3MpoPL1Yjem6HQ8gwRGVopJ9xy6fmFRA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=31)

  

三个参数，第一个是返回的validate，第二个是之前的fp，第三个zoneId，然后核心还是同一个加密函数，但凡多搞几种都没有心情分析了，整体连贯起来尝试，

![图片](https://mmbiz.qpic.cn/mmbiz_png/Gv6JExJQjUWdp3NnJkWy9ukS5K6iaPEEusGlfzM1fs5s5mWMeYooy1hAzbWLNljIbVcYxefoaBbXGf9qsliahHtYHTT7icargDvJCN97N6czcg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=32)

  

登录成功！以上就是全过程了！

  

  

**·** **今 日 推 荐** **·**

  

<table><tbody><tr><td data-colwidth="268" width="268" valign="top" style="word-break: break-all; width: 863px;"><p style="text-align: center;"><img class="rich_pages wxw-img" data-aistatus="1" data-imgfileid="504558545" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_jpg/Gv6JExJQjUVKl4icL6H19Uyyibj5sXcyXRhC2CicictHcy20TPuz6k8KI0gqQfecfeoOEZSngGNIyvz6ibI0gqibcBKnNXhOpeqybU9tQjWXiaRtfg/640?wx_fmt=jpeg&amp;from=appmsg#imgIndex=33" data-type="jpeg" type="inline" data-original-style="null" data-index="35" data-ratio="1.4015518913676042" data-w="1031" src="https://mmbiz.qpic.cn/mmbiz_jpg/Gv6JExJQjUVKl4icL6H19Uyyibj5sXcyXRhC2CicictHcy20TPuz6k8KI0gqQfecfeoOEZSngGNIyvz6ibI0gqibcBKnNXhOpeqybU9tQjWXiaRtfg/640?wx_fmt=jpeg&amp;from=appmsg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1#imgIndex=33" _width="677px" style="height: auto !important; visibility: visible !important; width: 316.992px !important;" alt="图片" data-report-img-idx="34" data-fail="0"></p></td><td data-colwidth="268" width="268" valign="top" style="word-break: break-all; width: 863px;"><p style="text-align: center;"><img class="rich_pages wxw-img" data-imgfileid="504558080" data-ratio="1.368421052631579" data-s="300,640" data-src="https://mmbiz.qpic.cn/sz_mmbiz_jpg/Gv6JExJQjUWRT04uLSbQ1VblTb1nLKDjzUDXD4aI87NBBFCAnRG39AKV9opm5CsQQeH5pgUtwGKtA3tMPRaQv4hU61MYDR3lkYEocA6RCE4/640?wx_fmt=jpeg&amp;from=appmsg#imgIndex=34" data-type="jpeg" data-w="950" data-original-style="null" data-index="36" src="https://mmbiz.qpic.cn/sz_mmbiz_jpg/Gv6JExJQjUWRT04uLSbQ1VblTb1nLKDjzUDXD4aI87NBBFCAnRG39AKV9opm5CsQQeH5pgUtwGKtA3tMPRaQv4hU61MYDR3lkYEocA6RCE4/640?wx_fmt=jpeg&amp;from=appmsg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1#imgIndex=34" _width="677px" style="height: auto !important; visibility: visible !important; width: 316.992px !important;" alt="图片" data-report-img-idx="33" data-fail="0"></p></td></tr></tbody></table>