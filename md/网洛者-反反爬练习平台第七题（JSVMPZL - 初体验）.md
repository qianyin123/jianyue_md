> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/nvQNV33QkzFQtFscDqnXWw)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/QibCkibMaYKNyQdVf4S3YSJpwrgcANHGwN9icFHTib6vktvVoiadNPic82j8RaEau5ZqCRonWL53iafAiaXDN4ic8XCI7ZQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

**链接**：http://spider.wangluozhe.com/challenge/7  (提示：该平台注册需要注册码，可以加这位大佬的vx，记得注明来意，vx号：wx1670044143)

  

**内容分析:**

   1.**本题目标**：采集100页的全部数字，并计算所有数据加和

   2.**接口分析**：老规矩，F12打开调试面板，直接遇到本题第一个反调试：**debugger**，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic3Y4GOxx7fGXia4Q2qy4zSE1qzLgp2d4tXSqmFTCCrHoZAFkicbC4B9IiaElceoYu1p6l969IzDDE9xQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

直接在这一行行标上鼠标右键，选择**Never pause here**，之后直接释放断点即可。

接下来点击下一页，然后出现请求，点上去看一下请求的时候携带了哪些参数

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic3Y4GOxx7fGXia4Q2qy4zSE1uB9YNpib8p7UqauOqeyTwTCWylqNWibsYo8rGo7Czk0McLsuCOuYU9Ow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

这里请求有三个参数：**page**（页码）、**count**（返回数据数量），**_signature**（加密参数）。很明显**_signature**参数就是今天要分析的对象，话不多说，接下来直接开始分析这个值怎么来的。直接点击如下图所示位置跳转到发起请求的位置

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic3Y4GOxx7fGXia4Q2qy4zSE1BYX6dw9LTkMziaNibPrEib8gvGlbtoFSAcML8vEibfISsvmARAtdLLFWJg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic3Y4GOxx7fGXia4Q2qy4zSE1DLIgeLWzGBUfPvicQm3lZl03fykSOjt2Y7SamXxvXl4KuIVOYBS1NPw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

很明显，**_signature**值等于**window._signature**，我们直接**console**中输出下

**window._signature**。

  

直接有值了，跟上面发起请求时携带的参数一样，所以很明显，**_signature**这个参数是在发起请求前生成之后赋值给了**window**，然后在发起请求的时候直接通过**window**来获取的，然后这里有个**window.get_sign()**，那我们是不是可以猜测一下，**_signature**这个值是在这个**window.get_sign()**中生成并完成赋值的，接下来我们验证一下，在控制面板直接**window.get_sign()**，然后再输出**window._signature**，如下图所示:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic3Y4GOxx7fGXia4Q2qy4zSE1niaaxkMH51bD80BORFYIXZf9M5UibuZYgruj8S03woQ8mSN7YhKGdH9g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里我们发现**window._signature**的值变了，证明**window._signature**这个值就是在**window.get_sign()**中生成并完成赋值的，到这里其实有两种方案解决这题了，既然直接运行**window.get_sign()**这个就能直接拿到新的**window._signature**值，那我们可以无需做任何破解，直接通过**jsrpc**来拿到**window._signature**这个参数就好了。

  

**jsrpc方案：**

  

  

这里我通过**jsrpc**直接拿到了请求的参数，下面我们来验证一下是否可行。

  

很明显，这个方案是可行的，而且不需要做任何的破解。

上面只是给大家演示了操作的结果，至于如何使用**jsrpc**，这里我就不做过的讲解了，大家可以直接参考下面这篇文章。

  

[JsRpc实战-猿人学-反混淆刷题平台第20题（wasm）](http://mp.weixin.qq.com/s?__biz=MzIxNjQ2NzM3MA==&mid=2247484671&idx=1&sn=eda1e44bf95e119ecc77021ac1d4b77a&chksm=9789d63aa0fe5f2c34ca5fb74e9f7b4f985756c201ae230ab01c0647d2607946473fb089f857&scene=21#wechat_redirect)

  

这里直接给大家贴一下这个方案的代码：

  

js注入的:

```
`function Hlclient(wsURL) {` `this.wsURL = wsURL;` `this.handlers = {};` `this.socket = {};` `if (!wsURL) {` `throw new Error('wsURL can not be empty!!')` `}` `this.connect()``}``Hlclient.prototype.connect = function () {` `console.log('begin of connect to wsURL: ' + this.wsURL);` `var _this = this;` `try {` `this.socket["ySocket"] = new WebSocket(this.wsURL);` `this.socket["ySocket"].onmessage = function (e) {` `console.log("send func", e.data);` `_this.handlerRequest(e.data);` `}` `} catch (e) {` `console.log("connection failed,reconnect after 10s");` `setTimeout(function () {` `_this.connect()` `}, 10000)` `}` `this.socket["ySocket"].onclose = function () {` `console.log("connection failed,reconnect after 10s");` `setTimeout(function () {` `_this.connect()` `}, 10000)` `}``};``Hlclient.prototype.send = function (msg) {` `this.socket["ySocket"].send(msg)``}``Hlclient.prototype.regAction = function (func_name, func) {` `if (typeof func_name !== 'string') {` `throw new Error("an func_name must be string");` `}` `if (typeof func !== 'function') {` `throw new Error("must be function");` `}` `console.log("register func_name: " + func_name);` `this.handlers[func_name] = func;``}``Hlclient.prototype.handlerRequest = function (requestJson) {` `var _this = this;` `var result = JSON.parse(requestJson);` `//console.log(result)` `if (!result['action']) {` `this.sendResult('', 'need request param {action}');` `return` `}` `action = result["action"]` `var theHandler = this.handlers[action];` `if (!theHandler || theHandler == undefined) {` `this.sendResult(action, 'action not found');` `return` `}` `try {` `if (!result["param"]) {` `theHandler(function (response) {` `_this.sendResult(action, response);` `})` `} else {` `theHandler(function (response) {` `_this.sendResult(action, response);` `}, result["param"])` `}` `} catch (e) {` `console.log("error: " + e);` `_this.sendResult(action + e);` `}``}``Hlclient.prototype.sendResult = function (action, e) {` `this.send(action + atob("aGxeX14") + e);``}``// 连接通信``var demo = new Hlclient("ws://127.0.0.1:12080/ws?group=test&);``// 第二个参数为函数，resolve里面的值是想要的值(发送到服务器的)``// param是可传参参数，可以忽略``demo.regAction("_signature", function (resolve, param) {` `window.get_sign();` `_signature = window._signature` `resolve(_signature)``})`
```

python代码:

```
`import requests``def get_params(page):` `params = {` `"group": "test",  # 接口名称` `"action": "_signature",` `"name": "wangluozhe"` `}` `try:` `res = requests.get("http://127.0.0.1:12080/go", params=params).json()` `_signature = res.get("_signature")` `data = {` `"page": page,` `"count": 10,` `"_signature": _signature` `}` `return data` `except:` `return False``def get_value(page):` `data = get_params(page)` `print(data)` `headers = {` `'Origin': 'http://spider.wangluozhe.com',` `'Accept-Encoding': 'gzip, deflate',` `'Accept-Language': 'zh-CN,zh;q=0.9',` `'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3883.400 QQBrowser/10.8.4559.400',` `'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',` `'Accept': 'application/json, text/javascript, */*; q=0.01',` `'Referer': 'http://spider.wangluozhe.com/challenge/4',` `'X-Requested-With': 'XMLHttpRequest',` `'Connection': 'keep-alive',` `}` `cookies = {` `'v': 'xxxxx',``        'session': 'xxxxx',` `}   #此处填自己的cookie即可` `response = requests.post('http://spider.wangluozhe.com/challenge/api/7', headers=headers, cookies=cookies,` `data=data)` `try:` `data = response.json()['data']` `print(data)` `now_page_sum_value = sum([int((i.get("value"))) for i in data])` `return now_page_sum_value` `except:` `print(response.text)` `return 0``get_value(2)`
```

* * *

  

上面给大家讲解了使用jsrpc的方案，下面我们就通过分析js来解决这题：

js逆向:  

前面分析了_signature参数是在哪里生成的，那么我直接跳到生成它的js文件中，如下图所示：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

直接在这一行打上断点，然后不断的释放（每断住一次看一下），知道如下图所示：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里我们能发现gnature = window.byted_acrawler(window.sign()),看到这个我们直接在**console**面板输出试试

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里我们发现得到的结果和_signature参数很相似，其实如果有经验的话，到这里是可以直接确定这个就是生成加密参数的函数。然后我们看看window.sign()输出出来是什么

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里我们能看到window.sign()就是一个时间戳，大胆猜测一下，_signature参数就是当前的时间戳经过md5之后得到的。这里我们直接到用拿一个时间戳去试试，测试结果令人失望，对不上，而且我们固定一个时间值放进去会发现每次的结果都是不相同的

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

那么没办法了，只能继续分析下去。

  

这里为了不浪费时间我就直接说需要在哪些地方打上断点了，具体为什么可以在这些地方打断点，需要自己耐心的单步调试走一下，当你走过几遍之后就会知道在哪些地方下断点比较好，这里的断点都是日志断点，这是因为Jsvpm里面没办法直观的看到要执行的代码（当然可能是我水平不够，哈哈哈），所以通过日志断点来输出出来便于分析，话不多说，直接展示打断点的位置。

  

605行，日志断点直接填 arguments就好，之后617行，如下图所示：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

584行和585打上日志断点，输出__V(_, __)的值

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

509行打上断点，输出__V(_, __)的值

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

266行打上日志断点，输出v_y的值

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

173行和175行打上日志断点，输出 _[V__][0][__]的值

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

打这么多断点只是为了让输出的内容更加的详细，如果有经验的小伙伴只需要在几个关键的位置打上日志断点就好了（这几个断点就已经够分析了，所以还有的断点位置就不放上去了），这些地方打上条件断点之后，我们重新走一遍，根据需要自己可选在不在下图位置打上断点

  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

点击下一页，我们会优先断在上图所示的位置，然后直接释放断点，因为我们上面都是打的日志断点，当我们释放之后会直接在console面板中输出很多内容，直接等待输出完成即可。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

看到上面这个基本上就说明已经打印完成了，由于会自动输出一些内容，这里我们需要在所有内容输出完成之后在151一行打一个断点，防止控制台不断输出一些无用内容，影响调试，接下来我们只需要分析输出在console面板中内容即可

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

这里我们看到打印了很多参数，暂时不用管，继续往下看，（提示：由于我们是从外往里面打印的，所以会优先输出最终的结果）

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

看到我圈起来的参数了吧，结合起来看看：

```
`"window" +"._si" +"gnature = window.byted_acrawler(window.sign())"` `code="window._signature = window.byted_acrawler(window.sign())"``eval(code)`
```

  

我们把上面的代码拿到console面板中输出试试

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

这里就看到了window._signature这个是如何得到的。我们接着继续往下看

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

上面我也说了，由于我们是从外往里面打印的，所以会优先输出最终的结果。

从上图可以看到，sign输出的最终结果是：1645496445000，之后继续往上看，**Date**、**parse**、**Date**、**toString**，其实这里很明显就能看出，如果猜不到，也可以百度一下，Date，parse这两个关键词，然后我们可以很容易得出下面的代码

```
`t = Date.parse(Date())``t= t.toString()`
```

验证一下

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

继续往下看

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**hex_md5**、**str2binl**，很明显啊，最终的结果就是时间戳然后md5一下得到的，不过我们拿这个时间戳去md5一下会发现根本对不上，那么也很容易想到，这个md5是魔改过的，那么没办法，只能继续往下看了，这里为了方便大家对比调试，我直接将md5的js代码放这里（其实大家自行百度就能得到，大家要学会百度啊）

```
`/*` `* A JavaScript implementation of the RSA Data Security, Inc. MD5 Message` `* Digest Algorithm, as defined in RFC 1321.` `* Version 2.1 Copyright (C) Paul Johnston 1999 - 2002.` `* Other contributors: Greg Holt, Andrew Kepert, Ydnar, Lostinet` `* Distributed under the BSD License` `* See http://pajhome.org.uk/crypt/md5 for more info.` `*/``/*` `* Configurable variables. You may need to tweak these to be compatible with` `* the server-side, but the defaults work in most cases.` `*/``var hexcase = 0;  /* hex output format. 0 - lowercase; 1 - uppercase        */``var b64pad = ""; /* base-64 pad character. "=" for strict RFC compliance   */``var chrsz = 8;  /* bits per input character. 8 - ASCII; 16 - Unicode      */``/*` `* These are the functions you'll usually want to call` `* They take string arguments and return either hex or base-64 encoded strings` `*/``function hex_md5(s) {` `return binl2hex(core_md5(str2binl(s), s.length * chrsz));``}``/*` `* Calculate the MD5 of an array of little-endian words, and a bit length` `*/``function core_md5(x, len) {` `/* append padding */` `x[len >> 5] |= 0x80 << ((len) % 32);` `x[(((len + 64) >>> 9) << 4) + 14] = len;` `var a = 1732584193;` `var b = -271733879;` `var c = -1732584194;` `var d = 271733878;` `for (var i = 0; i < x.length; i += 16) {` `var olda = a;` `var oldb = b;` `var oldc = c;` `var oldd = d;` `a = md5_ff(a, b, c, d, x[i + 0], 7, -680876936);` `d = md5_ff(d, a, b, c, x[i + 1], 12, -389564586);` `c = md5_ff(c, d, a, b, x[i + 2], 17, 606105819);` `b = md5_ff(b, c, d, a, x[i + 3], 22, -1044525330);` `a = md5_ff(a, b, c, d, x[i + 4], 7, -176418897);` `d = md5_ff(d, a, b, c, x[i + 5], 12, 1200080426);` `c = md5_ff(c, d, a, b, x[i + 6], 17, -1473231341);` `b = md5_ff(b, c, d, a, x[i + 7], 22, -45705983);` `a = md5_ff(a, b, c, d, x[i + 8], 7, 1770035416);` `d = md5_ff(d, a, b, c, x[i + 9], 12, -1958414417);` `c = md5_ff(c, d, a, b, x[i + 10], 17, -42063);` `b = md5_ff(b, c, d, a, x[i + 11], 22, -1990404162);` `a = md5_ff(a, b, c, d, x[i + 12], 7, 1804603682);` `d = md5_ff(d, a, b, c, x[i + 13], 12, -40341101);` `c = md5_ff(c, d, a, b, x[i + 14], 17, -1502002290);` `b = md5_ff(b, c, d, a, x[i + 15], 22, 1236535329);` `a = md5_gg(a, b, c, d, x[i + 1], 5, -165796510);` `d = md5_gg(d, a, b, c, x[i + 6], 9, -1069501632);` `c = md5_gg(c, d, a, b, x[i + 11], 14, 643717713);` `b = md5_gg(b, c, d, a, x[i + 0], 20, -373897302);` `a = md5_gg(a, b, c, d, x[i + 5], 5, -701558691);` `d = md5_gg(d, a, b, c, x[i + 10], 9, 38016083);` `c = md5_gg(c, d, a, b, x[i + 15], 14, -660478335);` `b = md5_gg(b, c, d, a, x[i + 4], 20, -405537848);` `a = md5_gg(a, b, c, d, x[i + 9], 5, 568446438);` `d = md5_gg(d, a, b, c, x[i + 14], 9, -1019803690);` `c = md5_gg(c, d, a, b, x[i + 3], 14, -187363961);` `b = md5_gg(b, c, d, a, x[i + 8], 20, 1163531501);` `a = md5_gg(a, b, c, d, x[i + 13], 5, -1444681467);` `d = md5_gg(d, a, b, c, x[i + 2], 9, -51403784);` `c = md5_gg(c, d, a, b, x[i + 7], 14, 1735328473);` `b = md5_gg(b, c, d, a, x[i + 12], 20, -1926607734);` `a = md5_hh(a, b, c, d, x[i + 5], 4, -378558);` `d = md5_hh(d, a, b, c, x[i + 8], 11, -2022574463);` `c = md5_hh(c, d, a, b, x[i + 11], 16, 1839030562);` `b = md5_hh(b, c, d, a, x[i + 14], 23, -35309556);` `a = md5_hh(a, b, c, d, x[i + 1], 4, -1530992060);` `d = md5_hh(d, a, b, c, x[i + 4], 11, 1272893353);` `c = md5_hh(c, d, a, b, x[i + 7], 16, -155497632);` `b = md5_hh(b, c, d, a, x[i + 10], 23, -1094730640);` `a = md5_hh(a, b, c, d, x[i + 13], 4, 681279174);` `d = md5_hh(d, a, b, c, x[i + 0], 11, -358537222);` `c = md5_hh(c, d, a, b, x[i + 3], 16, -722521979);` `b = md5_hh(b, c, d, a, x[i + 6], 23, 76029189);` `a = md5_hh(a, b, c, d, x[i + 9], 4, -640364487);` `d = md5_hh(d, a, b, c, x[i + 12], 11, -421815835);` `c = md5_hh(c, d, a, b, x[i + 15], 16, 530742520);` `b = md5_hh(b, c, d, a, x[i + 2], 23, -995338651);` `a = md5_ii(a, b, c, d, x[i + 0], 6, -198630844);` `d = md5_ii(d, a, b, c, x[i + 7], 10, 1126891415);` `c = md5_ii(c, d, a, b, x[i + 14], 15, -1416354905);` `b = md5_ii(b, c, d, a, x[i + 5], 21, -57434055);` `a = md5_ii(a, b, c, d, x[i + 12], 6, 1700485571);` `d = md5_ii(d, a, b, c, x[i + 3], 10, -1894986606);` `c = md5_ii(c, d, a, b, x[i + 10], 15, -1051523);` `b = md5_ii(b, c, d, a, x[i + 1], 21, -2054922799);` `a = md5_ii(a, b, c, d, x[i + 8], 6, 1873313359);` `d = md5_ii(d, a, b, c, x[i + 15], 10, -30611744);` `c = md5_ii(c, d, a, b, x[i + 6], 15, -1560198380);` `b = md5_ii(b, c, d, a, x[i + 13], 21, 1309151649);` `a = md5_ii(a, b, c, d, x[i + 4], 6, -145523070);` `d = md5_ii(d, a, b, c, x[i + 11], 10, -1120210379);` `c = md5_ii(c, d, a, b, x[i + 2], 15, 718787259);` `b = md5_ii(b, c, d, a, x[i + 9], 21, -343485551);` `a = safe_add(a, olda);` `b = safe_add(b, oldb);` `c = safe_add(c, oldc);` `d = safe_add(d, oldd);` `}` `return Array(a, b, c, d);``}``/*` `* These functions implement the four basic operations the algorithm uses.` `*/``function md5_cmn(q, a, b, x, s, t) {` `return safe_add(bit_rol(safe_add(safe_add(a, q), safe_add(x, t)), s), b);``}``function md5_ff(a, b, c, d, x, s, t) {` `return md5_cmn((b & c) | ((~b) & d), a, b, x, s, t);``}``function md5_gg(a, b, c, d, x, s, t) {` `return md5_cmn((b & d) | (c & (~d)), a, b, x, s, t);``}``function md5_hh(a, b, c, d, x, s, t) {` `return md5_cmn(b ^ c ^ d, a, b, x, s, t);``}``function md5_ii(a, b, c, d, x, s, t) {` `return md5_cmn(c ^ (b | (~d)), a, b, x, s, t);``}``/*` `* Calculate the HMAC-MD5, of a key and some data` `*/``function core_hmac_md5(key, data) {` `var bkey = str2binl(key);` `if (bkey.length > 16) bkey = core_md5(bkey, key.length * chrsz);` `var ipad = Array(16), opad = Array(16);` `for (var i = 0; i < 16; i++) {` `ipad[i] = bkey[i] ^ 0x36363636;` `opad[i] = bkey[i] ^ 0x5C5C5C5C;` `}` `var hash = core_md5(ipad.concat(str2binl(data)), 512 + data.length * chrsz);` `return core_md5(opad.concat(hash), 512 + 128);``}``/*` `* Add integers, wrapping at 2^32. This uses 16-bit operations internally` `* to work around bugs in some JS interpreters.` `*/``function safe_add(x, y) {` `var lsw = (x & 0xFFFF) + (y & 0xFFFF);` `var msw = (x >> 16) + (y >> 16) + (lsw >> 16);` `return (msw << 16) | (lsw & 0xFFFF);``}``/*` `* Bitwise rotate a 32-bit number to the left.` `*/``function bit_rol(num, cnt) {` `return (num << cnt) | (num >>> (32 - cnt));``}``/*` `* Convert a string to an array of little-endian words` `* If chrsz is ASCII, characters >255 have their hi-byte silently ignored.` `*/``function str2binl(str) {` `var bin = Array();` `var mask = (1 << chrsz) - 1;` `for (var i = 0; i < str.length * chrsz; i += chrsz)` `bin[i >> 5] |= (str.charCodeAt(i / chrsz) & mask) << (i % 32);` `return bin;``}``/*` `* Convert an array of little-endian words to a string` `*/``function binl2str(bin) {` `var str = "";` `var mask = (1 << chrsz) - 1;` `for (var i = 0; i < bin.length * 32; i += chrsz)` `str += String.fromCharCode((bin[i >> 5] >>> (i % 32)) & mask);` `return str;``}``/*` `* Convert an array of little-endian words to a hex string.` `*/``function binl2hex(binarray) {` `var hex_tab = hexcase ? "0123456789ABCDEF" : "0123456789abcdef";` `var str = "";` `for (var i = 0; i < binarray.length * 4; i++) {` `str += hex_tab.charAt((binarray[i >> 2] >> ((i % 4) * 8 + 4)) & 0xF) +` `hex_tab.charAt((binarray[i >> 2] >> ((i % 4) * 8)) & 0xF);` `}` `return str;``}``/*` `* Convert an array of little-endian words to a base-64 string` `*/``function binl2b64(binarray) {` `var tab = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";` `var str = "";` `for (var i = 0; i < binarray.length * 4; i += 3) {` `var triplet = (((binarray[i >> 2] >> 8 * (i % 4)) & 0xFF) << 16)` `| (((binarray[i + 1 >> 2] >> 8 * ((i + 1) % 4)) & 0xFF) << 8)` `| ((binarray[i + 2 >> 2] >> 8 * ((i + 2) % 4)) & 0xFF);` `for (var j = 0; j < 4; j++) {` `if (i * 8 + j * 6 > binarray.length * 32) str += b64pad;` `else str += tab.charAt((triplet >> 6 * (3 - j)) & 0x3F);` `}` `}` `return str;``}`
```

  

先从md5的js中找找看，发现hex_md5、str2binl都有，那我们这里直接一一测试即可。

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

结果一致，那么这里没有做过改动，继续往下看。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

继续测试

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

这里很明显能看出和上面的结果对不上，那么魔改的地方肯定就是这个core_md5函数了,既然如此，我们继续往下看

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

这里是传入的x和len

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

结果一致，说明这里也没有魔改，继续往下看

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

结果一致，也没有魔改 ，之后就是这样重复的调试工作了，大家按这样对比调试即可

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

这里说一下，我这里为了方便对比调试，所以以a为分界线，每4个为一组（a，b，c，d）,因为是从上往下执行的，所以如果第一个值不对的话，那么最后一位的值肯定是对不上的，所以大家只需要对比每组的第一个和最后一个的参数即可，由于都是这种对比调试，这里我就不一一写下去了，经过调试发现魔改的地方如下

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

圈起来的就是被魔改后的位置，注释掉的是原本原有的值。下面贴一下代码：

  

```
`var hexcase = 0;  /* hex output format. 0 - lowercase; 1 - uppercase        */``var b64pad = ""; /* base-64 pad character. "=" for strict RFC compliance   */``var chrsz = 8;  /* bits per input character. 8 - ASCII; 16 - Unicode      */``/*` `* These are the functions you'll usually want to call` `* They take string arguments and return either hex or base-64 encoded strings` `*/``function hex_md5(s) {` `return binl2hex(core_md5(str2binl(s), s.length * chrsz));``}``function core_md5(x, len) {` `/* append padding */` `x[len >> 5] |= 0x80 << ((len) % 32);` `// x[len>>5] |=` `x[(((len + 64) >>> 9) << 4) + 14] = len;` `var a = 1732584193;` `var b = -271733879;` `var c = -1732584194;` `var d = 271733878;` `for (var i = 0; i < x.length; i += 16) {` `var olda = a;` `var oldb = b;` `var oldc = c;` `var oldd = d;` `a = md5_ff(a, b, c, d, x[i + 0], 7, -680876936);` `d = md5_ff(d, a, b, c, x[i + 1], 12, -389564586);` `c = md5_ff(c, d, a, b, x[i + 2], 17, 606105819);` `b = md5_ff(b, c, d, a, x[i + 3], 22, -1044525330);` `a = md5_ff(a, b, c, d, x[i + 4], 7, -176418897);` `d = md5_ff(d, a, b, c, x[i + 5], 12, 1200080426);` `c = md5_ff(c, d, a, b, x[i + 6], 17, -1473231341);` `b = md5_ff(b, c, d, a, x[i + 7], 22, -45705983);` `a = md5_ff(a, b, c, d, x[i + 8], 7, 1770035416);` `d = md5_ff(d, a, b, c, x[i + 9], 12, -1958414417);` `c = md5_ff(c, d, a, b, x[i + 10], 17, -42063);` `b = md5_ff(b, c, d, a, x[i + 11], 22, -1990404162);` `a = md5_ff(a, b, c, d, x[i + 12], 7, 1804603682);` `d = md5_ff(d, a, b, c, x[i + 13], 12, -40341101);` `c = md5_ff(c, d, a, b, x[i + 14], 17, -1502002290);` `b = md5_ff(b, c, d, a, x[i + 15], 22, 1236535329);` `a = md5_gg(a, b, c, d, x[i + 1], 5, -165796510);` `d = md5_gg(d, a, b, c, x[i + 6], 9, -1069501632);` `c = md5_gg(c, d, a, b, x[i + 11], 14, 643717713);` `b = md5_gg(b, c, d, a, x[i + 0], 20, -373897302);` `a = md5_gg(a, b, c, d, x[i + 5], 5, -701558691);` `d = md5_gg(d, a, b, c, x[i + 10], 9, 38016083);` `c = md5_gg(c, d, a, b, x[i + 15], 14, -660478335);` `b = md5_gg(b, c, d, a, x[i + 4], 20, -405537848);` `a = md5_gg(a, b, c, d, x[i + 9], 5, 568446438);` `d = md5_gg(d, a, b, c, x[i + 14], 9, -1019803690);` `c = md5_gg(c, d, a, b, x[i + 3], 14, -187363961);` `b = md5_gg(b, c, d, a, x[i + 8], 20, 1163531501);` `a = md5_gg(a, b, c, d, x[i + 13], 5, -1444681467);` `d = md5_gg(d, a, b, c, x[i + 2], 9, -51403784);` `c = md5_gg(c, d, a, b, x[i + 7], 14, 1735328473);` `b = md5_gg(b, c, d, a, x[i + 12], 20, -1926607734);` `a = md5_hh(a, b, c, d, x[i + 5], 4, -378558);` `d = md5_hh(d, a, b, c, x[i + 8], 11, -2022574463);` `c = md5_hh(c, d, a, b, x[i + 11], 16, 1839030562);` `b = md5_hh(b, c, d, a, x[i + 14], 23, -35309556);` `a = md5_hh(a, b, c, d, x[i + 1], 4, -1530992060);` `d = md5_hh(d, a, b, c, x[i + 4], 11, 1272893353);` `c = md5_hh(c, d, a, b, x[i + 7], 16, -155497632);` `b = md5_hh(b, c, d, a, x[i + 10], 23, -1094730640);` `a = md5_hh(a, b, c, d, x[i + 13], 4, 681279174);` `d = md5_hh(d, a, b, c, x[i + 0], 11, -358537222);` `// c = md5_hh(c, d, a, b, x[i + 3], 16, -722521979);` `c = md5_hh(c, d, a, b, x[i + 3], 16, -722521939);` `// b = md5_hh(b, c, d, a, x[i + 6], 23, 76029189);` `b = md5_hh(b, c, d, a, x[i + 6], 23, 76029185);` `a = md5_hh(a, b, c, d, x[i + 9], 4, -640364487);` `d = md5_hh(d, a, b, c, x[i + 12], 11, -421815835);` `c = md5_hh(c, d, a, b, x[i + 15], 16, 530742520);` `b = md5_hh(b, c, d, a, x[i + 2], 23, -995338651);` `a = md5_ii(a, b, c, d, x[i + 0], 6, -198630844);` `d = md5_ii(d, a, b, c, x[i + 7], 10, 1126891415);` `c = md5_ii(c, d, a, b, x[i + 14], 15, -1416354905);` `b = md5_ii(b, c, d, a, x[i + 5], 21, -57434055);` `a = md5_ii(a, b, c, d, x[i + 12], 6, 1700485571);` `d = md5_ii(d, a, b, c, x[i + 3], 10, -1894986606);` `c = md5_ii(c, d, a, b, x[i + 10], 15, -1051523);` `b = md5_ii(b, c, d, a, x[i + 1], 21, -2054922799);` `a = md5_ii(a, b, c, d, x[i + 8], 6, 1873313359);` `d = md5_ii(d, a, b, c, x[i + 15], 10, -30611744);` `c = md5_ii(c, d, a, b, x[i + 6], 15, -1560198380);` `b = md5_ii(b, c, d, a, x[i + 13], 21, 1309151649);` `a = md5_ii(a, b, c, d, x[i + 4], 6, -145523070);` `d = md5_ii(d, a, b, c, x[i + 11], 10, -1120210379);` `c = md5_ii(c, d, a, b, x[i + 2], 15, 718787259);` `b = md5_ii(b, c, d, a, x[i + 9], 21, -343485551);` `a = safe_add(a, olda);` `b = safe_add(b, oldb);` `c = safe_add(c, oldc);` `d = safe_add(d, oldd);` `}` `return Array(a, b, c, d);``}``/*` `* These functions implement the four basic operations the algorithm uses.` `*/``function md5_cmn(q, a, b, x, s, t) {` `return safe_add(bit_rol(safe_add(safe_add(a, q), safe_add(x, t)), s), b);``}``function md5_ff(a, b, c, d, x, s, t) {` `return md5_cmn((b & c) | ((~b) & d), a, b, x, s, t);``}``function md5_gg(a, b, c, d, x, s, t) {` `return md5_cmn((b & d) | (c & (~d)), a, b, x, s, t);``}``function md5_hh(a, b, c, d, x, s, t) {` `return md5_cmn(b ^ c ^ d, a, b, x, s, t);``}``function md5_ii(a, b, c, d, x, s, t) {` `return md5_cmn(c ^ (b | (~d)), a, b, x, s, t);``}``/*` `* Calculate the HMAC-MD5, of a key and some data` `*/``function core_hmac_md5(key, data) {` `var bkey = str2binl(key);` `if (bkey.length > 16) bkey = core_md5(bkey, key.length * chrsz);` `var ipad = Array(16), opad = Array(16);` `for (var i = 0; i < 16; i++) {` `ipad[i] = bkey[i] ^ 0x36363636;` `opad[i] = bkey[i] ^ 0x5C5C5C5C;` `}` `var hash = core_md5(ipad.concat(str2binl(data)), 512 + data.length * chrsz);` `return core_md5(opad.concat(hash), 512 + 128);``}``/*` `* Add integers, wrapping at 2^32. This uses 16-bit operations internally` `* to work around bugs in some JS interpreters.` `*/``function safe_add(x, y) {` `var lsw = (x & 0xFFFF) + (y & 0xFFFF);` `var msw = (x >> 16) + (y >> 16) + (lsw >> 16);` `return (msw << 16) | (lsw & 0xFFFF);``}``/*` `* Bitwise rotate a 32-bit number to the left.` `*/``function bit_rol(num, cnt) {` `return (num << cnt) | (num >>> (32 - cnt));``}``/*` `* Convert a string to an array of little-endian words` `* If chrsz is ASCII, characters >255 have their hi-byte silently ignored.` `*/``function str2binl(str) {` `var bin = Array();` `var mask = (1 << chrsz) - 1;` `for (var i = 0; i < str.length * chrsz; i += chrsz)` `bin[i >> 5] |= (str.charCodeAt(i / chrsz) & mask) << (i % 32);` `return bin;``}``/*` `* Convert an array of little-endian words to a string` `*/``function binl2str(bin) {` `var str = "";` `var mask = (1 << chrsz) - 1;` `for (var i = 0; i < bin.length * 32; i += chrsz)` `str += String.fromCharCode((bin[i >> 5] >>> (i % 32)) & mask);` `return str;``}``function binl2hex(binarray) {` `var hex_tab = hexcase ? "0123456789ABCDEF" : "0123456789abcdef";` `var str = "";` `for (var i = 0; i < binarray.length * 4; i++) {` `str += hex_tab.charAt((binarray[i >> 2] >> ((i % 4) * 8 + 4)) & 0xF) +` `hex_tab.charAt((binarray[i >> 2] >> ((i % 4) * 8)) & 0xF);` `}` `return str;``}``function get__signature() {` `t = Date.parse(Date())` `_signature = hex_md5(t.toString())` `return _signature``}`
```

这里直接验证一下

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

结果一致，本题_signature参数的破解完成。接下来我们验证一下是否能拿到数据

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic3Y4GOxx7fGXia4Q2qy4zSE1ajHSXaBZmPGs7Oic16cLOOiax69NwXKz7v6hiaU9n9wFYxvGSIdI1WiaUg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

成功拿到数据，本题完成。