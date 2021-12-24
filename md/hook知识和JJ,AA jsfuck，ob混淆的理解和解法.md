> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JjmagOPPGQzHRlsmAw9THA)

声明：
===

以下内容来自成都逆向团成员之一 RY

  

一. 前言
=====

这里主要是写对于hook知识的一个理解和对于市面的混淆做出一个他的解法， 相当于一个总和

二. hook
-------

什么是hook? Hook技术又叫做钩子函数，在系统没有调用该函数之前，钩子程序就先捕获该消息，钩子函数先得到控制权，这时钩子函数既可以加工处理（改变）该函数的执行行为，还可以强制结束消息的传递。简单来说，就是把系统的程序拉出来变成我们自己执行代码片段。在程序中我们可以把他理解为劫持。

在js中的hook，替换原函数的都可以理解为hook，那么我们就有个疑问了，我们为什么可以hook呢， 原因就是我们客户端拥有最高解释权，可以决定在任何时候注入hook，而服务器无法阻止，只能通过检测hook或者混淆，让其难度加大。这里我强烈推荐一篇文章, 可以让你理解hook知识：[https://mp.weixin.qq.com/s/IYFyjVrVkHtUdCzn9arkJQ](https://mp.weixin.qq.com/s?__biz=Mzg5NzY2MzA5MQ==&mid=2247486172&idx=1&sn=0434718daf570df32d98582d79a42eaf&scene=21#wechat_redirect "JS 逆向之 Hook")。hook当然还有一个插件，那就是油猴，关于油猴就不在这里面过多描述了，可百度。

  

**fillder中的JS替换**

**![图片](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXX1MnHAhVBxGQk7rMFBGeP6mItPokJ4UlUiaZr4OKibz0Ff7GrdE55fRQoJ0kaAMKaTjhs0K6SJWbCg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**

这里hook需要着重理解的就是**object.defineProperty()**:Object.defineProperty(obj, prop, descriptor)是基本语法，作用就是在一个对象上面定义一个新的属性，或修改一个对象现有的属性。

  

接收三个参数如下：

*   obj: 需要定义属性的对象.
    
*   prop: 当前需要定义的属性名， 也就是你要hook的属性　　　　　
    
*   descriptor：属性的描述符， 可以取以下的值：
    

　　　　　　　　![图片](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXX1MnHAhVBxGQk7rMFBGeP6c1NVKEaIUKrQ1MHD5k5gQ1Fs6nFQHR4khHTlVFDREnN9oerl0opibmg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里写一个hook cookie的一个列子：　

```
`Object.defineProperty(document, 'cookie', {``set:function(val){``debugger;``return val``}``})`
```

这里第一个就是当前需要定义的对象， 第二个就是你要hook的属性， 第三个就是描述符了。

  

在 Hook 中，使用最多的是存取描述符，即 get 和 set。

**get**：属性的 getter 函数，如果没有 getter，则为 undefined，当访问该属性时，会调用此函数，执行时不传入任何参数，但是会传入 this 对象（由于继承关系，这里的 this 并不一定是定义该属性的对象），该函数的返回值会被用作属性的值。

**set**：属性的 setter 函数，如果没有 setter，则为 undefined，当属性值被修改时，会调用此函数，该方法接受一个参数，也就是被赋予的新值，会传入赋值时的 this 对象。

例子的话大家可以在我贴的上面代码实现， 需要注意的是，网站加载时首先要运行我们的 Hook 代码，再运行网站自己的代码，才能够成功断下，或者用fillder替换。这个过程我们可以称之为 Hook 代码的注入，下面写几种注入hook获取属性方法。

  

### Hook Cookie

  

```
`Object.defineProperty(document, 'cookie', {``set:function(val){``debugger;``return val``}``})`
```

### Hook Header

Header Hook 用于定位 Header 中关键参数生成位置，以下代码演示了当 Header 中包含 `Authorization` 关键字时，则插入断点：

```
`(function () {``var org = window.XMLHttpRequest.prototype.setRequestHeader;``window.XMLHttpRequest.prototype.setRequestHeader = function (key, value) {``if (key == 'Authorization') {``debugger;` `}``return org.apply(this, arguments);` `};``})();`
```

### Hook URL

URL Hook 用于定位请求 URL 中关键参数生成位置，以下代码演示了当请求的 URL 里包含 `login` 关键字时，则插入断点：

```
`(function () {``var open = window.XMLHttpRequest.prototype.open;``window.XMLHttpRequest.prototype.open = function (method, url, async) {``if (url.indexOf("login") != -1) {``debugger;` `}``return open.apply(this, arguments);` `};``})();`
```

  

  

> 编著注：下面的图微信好友Rui哥发我的，他顺便说了句“用这个直接就出链接了，这还扣什么js啊，写个post转发到flask就够了”。总之Y总牛逼

![图片](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXX1MnHAhVBxGQk7rMFBGeP6JlIa9WgzZ7ibia88bGg4zlgAZTcqGFYCPgbVNY88nmn5RKJFOZXxrnXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

### Hook JSON.stringify

`JSON.stringify()` 方法用于将 JavaScript 值转换为 JSON 字符串，在某些站点的加密过程中可能会遇到，以下代码演示了遇到 `JSON.stringify()` 时，则插入断点：

```
`(function() {``var stringify = JSON.stringify;``JSON.stringify = function(params) {``console.log("Hook JSON.stringify ——> ", params);``debugger;``return stringify(params);` `}``})();`
```

### Hook  JSON.parse

`JSON.parse()` 方法用于将一个 JSON 字符串转换为对象，在某些站点的加密过程中可能会遇到，以下代码演示了遇到 `JSON.parse()` 时，则插入断点：

```
`(function() {``var parse = JSON.parse;``JSON.parse = function(params) {``console.log("Hook JSON.parse ——> ", params);``debugger;``return parse(params);` `}``})();`
```

ob混淆
====

ob混淆呢具有以下特征：

1.  具有大数组的情况
    
2.  数组移位（有内存泄露风险、建议不格式化），自执行函数，进行移位操作，有明显的 push、shift 关键字
    
3.  解密函数（有内存泄露风险、建议不格式化）------可能有定时器--------（看加密的开关开启数量）
    
4.  实际代码+控制流平坦化（整体ob的强度几乎完全取决于这段的代码强度，这里面是加密前的逻辑）
    
5.  控制流平坦化+无限debugger自执行函数+死代码注入。一般情况下不会有业务逻辑所以不要问，ob、sojson如何破解。这些东西只是一层壳，破解强度完全取决于第四段代码，也就是其他网站作者写的代码强度！
    
6.  函数名和变量名通常以 _0x 或者 0x 开头，后接 1~6 位数字或字母组合；
    

  

解决方法呢：上面也讲了，还有就是AST解码混淆 推荐和猿人学的解混淆http://tool.yuanrenxue.com/，或者你硬刚也是可以的。

  

例如猿人学的第二题

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXX1MnHAhVBxGQk7rMFBGeP6HibIazTl6btv9Typn9NPy2AoIyN8TSbr8iaia4FVpeNIGibxpaT6FZw8NQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 ![图片](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXX1MnHAhVBxGQk7rMFBGeP6U2Zlj1WtuGO71qmce4iaQtRyu9n7niaXwibtxqJm865iaLm39MeT5Ol4Vw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过上面的列子我们可以知道运用hook的知识， 然后硬刚基本还是可以的， 不过遇到了复杂的混淆， hook是hook不到的 只能从头开始跟。

JJEncode混淆
==========

jjencode是日本的Yosuke HASEGAWA在2009年开发的，它可以将任意 JavaScript 编码为仅使用 18 个符号的混淆形式 []()!+,\"$.:;_{}~=。

缺点：JJEncode 易于解码，它不是实用的混淆，只是一个编码器，JJEncode很容易被检测，而且还浏览器依赖，它的缺点是压栈很严重，如果 JS 很大，去做加密可能内存溢出，所以只适合核心功能加密，事实上 JJEncode 商用的还是很少，不过认识一下并没有什么坏处。

  

举例：

![图片](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXX1MnHAhVBxGQk7rMFBGeP63aBhaPBZic5c0GiaFELjMFgmdvjiaiadNHpX2ia1iaicXuyO2pjZo0X8U5hKg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这一个正常的代码。然后这是混淆过后的代码

![图片](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXX1MnHAhVBxGQk7rMFBGeP6jak5ibbico1CtzWhk4jDZ0L11RwxGMLEORUiceEBW829hPmQnAtEvhicww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

解决办法：

1.  控制台执行报错，且非unsafe错误：点击报错信息即可还原代码
    
2.  控制台执行不报错，删除一些语法（以括号为主），强制令其报错
    
3.  控制台报unsafe错误：自写个html运行，然后删除语法令其报错即可还原 
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/l4m5icTfxSXX1MnHAhVBxGQk7rMFBGeP6Qz3lmRsbLiaichqFOY5jh1GNibDnbAia9NkCx244vlhib3fSATu0ey5C3Ng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 AAEncode混淆
===========

跟JJencode , jsfuck一个作者,方法都是差不多 在控制台进行打印。上面那几种方法都适用

解决方法：

*   去掉代码最后一个符号 ('_') 后，放到浏览器里面去直接执行就可以看到源码
    
*   在线调试，在 AAEncode 代码第一行下断点，然后一步一步执行，最终也会在虚拟机（VM）里看到源码
    

  

jsFuck混淆
========

jsfuck都是差不多的，方法跟上面都适用， 不过会有一种jsfuck技术在控制台打印他也显示不出来，他是在jsfuck套了一层，比如你想知道a  但是你控制台打印出来是b.

这种的解决办法是：在混淆前面打上debugger，然后在控制台一步一步的调试。跟到生成a的代码里面。这是唯一需要注意的一个点。其他基本差不多

  

> 编著注：卞大看完说的，jsfuck套一层的这种已经没用了，function的hook过时了，其他是对的