> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MzgzNTU5MA==&mid=2652041121&idx=1&sn=04b9dcbba194e28c256c7c1a2eacafc3&chksm=856f3ef03fc4fd8ff3b80bcaeef0cc5bd950b8f31c902ea88555a74fc7d6f7baa50d068e4d13&scene=0&xtrack=1#rd)

声明

本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关.本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责，若有侵权，请联系作者立即删除。

前言

本人刚学习app方面的逆向几个星期，最近花了四天搞了一个简单的app，写一篇文章分享记录自己的逆向过程，方便自己后续回顾，也方便刚学习app的伙伴们入门练习，同时文章也还遗留了一些问题，如果大佬们有相关的文章和视频可以告送我，我再去学习学习。第一次写文章，大佬们不喜勿喷，欢迎技术指点，万分感谢！

正文

基本信息

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlppuC9TgWLPgewqgbia8NQa1sVaCx7CJXlme0lfbYeROoKEvuL35wOTdg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

建议从谷歌商店下载，小米商城下载的好像有frida检测。

登录注册接口是某验4的一个无感，app端和web端好像都是一样的，本人已搞完了web的，不难，这一部分就不写出教程了，大佬们可以去web端玩玩。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpibacibMFj0bBgCQmo91kIQ5Zz4whrUA2sdhNLInpRknbwdvqkteRNYuA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

通过抓包可以看见，基本大多数的请求头中都会携带一个 x-signature ，因此接下来的文章内容以找到该参数的加密方式为主。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpHgbhNqrOO579a53Hj6iaydFyv0t1ARAwWhFtdnJmDhTbtGTI89aFdmw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

查找参数

1. Frida检测

本人原先的流程是用jadx反编译apk，查看java层代码，写了一点hook脚本，发现无法注入才知道有frida检测的，为了方便文章查看，因此先写了frida检测这部分，后面再讲编写hook脚本部分。（本人没解决该app的frida检测，如果有大佬有类似的文章或者教程可以推荐给我）

首先第一步可以查看加载了哪写so文件，导致断开了frida

 复制代码 隐藏代码

```
`var dlopen = Module.findExportByName(null, "dlopen");//低版本``var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");//高版本``Interceptor.attach(dlopen, {` `onEnter: function (args) {` `var path = args[0].readCString();` `console.log("[dlopen:]", path);` `},` `onLeave: function (retval) {` `}``});``Interceptor.attach(android_dlopen_ext, {` `onEnter: function (args) {` `var path = args[0].readCString();` `console.log("[dlopen_ext:]", path);` `},` `onLeave: function (retval) {` `}``});`
```

执行该脚本，可以看到加载了常见的 libmsaoaidsec.so

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlp4oYKAuWCDTh2zYsHu1QLDu5ZhEDO4ibibnOibVnwLPpXz0oaSlbbqJDzg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

然后我学习类似过该so文件的教程，查看检测的线程和查找偏移

```
`function check_pthread_create(name = null) {` `var pthread_create_addr = Module.findExportByName(null, 'pthread_create');` `var pthread_create = new NativeFunction(pthread_create_addr, "int", ["pointer", "pointer", "pointer", "pointer"]);` `Interceptor.replace(pthread_create_addr, new NativeCallback(function (parg0, parg1, parg2, parg3) {` `var module = Process.findModuleByAddress(parg2)` `var so_base = module.base;` `var off = "0x" + parg2.sub(so_base).toString(16)` `var so_name = module.name;` `console.log('[pthread_create] ',so_name, off, parg3)` `return pthread_create(parg0, parg1, parg2, parg3);` `}, "int", ["pointer", "pointer", "pointer", "pointer"]))``}``setImmediate(check_pthread_create)`
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpvGWrUibFImgCHWe9hiaZCUzPvGqZBg81hamjBtccpfxuC28VvBjM1PmQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

可是我后续 nop 掉相关的地址，还是没用。留着以后再解决吧......

用谷歌商店下载的无frida检测。

2. 参数查找

首先打开jadx，可以搜索查找相关关键词，如我搜索了 x-signature

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpGdxu599qMhQguukoVeicehftAPRuxkJrkQEhIscrbIBO6Z10n7AFibUQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

```
builder.add("X-Signature", KLookSignatureToolUtil.sign(bArr2, "kMtbID/p1?eWAsQ+5A3g="));
```

这里搜索到的大小写有点差异，不过问题不大，我们可以复制frida脚本，试试究竟会不会走这对应的函数。这两个都是走的KLookSignatureToolUtil.sign，双击进入

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpNLnmUw9CicO6tHZhZhKI6gTbIiaDWiaz7eNQd3qut0OSBFnMyS5AXXBPA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

```
String sign = KLookSignatureTool.sign(AppContextHolderKt.getAppContext(), data, key);
```

这里走的 KLookSignatureTool.sign，一样双击进入

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpibsWKQfAUnFQUVxpWmwPRRhcNQCIwMTRMR0atJiaJch6clWbnrdu80ZQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

到此就可以先编写hook脚本查看是否是这几个进行的调用：

```
`function hookSign(){` `console.log("开始注入")` `let SignatureHandlerImpl = Java.use("com.klooklib.flutter.bridgeimpl.SignatureHandlerImpl");` `SignatureHandlerImpl["sign"].implementation = function (argument) {` ``console.log(`SignatureHandlerImpl.sign 调用: argument=${argument}`);`` `let result = this["sign"](argument);` ``console.log(`SignatureHandlerImpl.sign result=${result}`);`` `return result;` `};` `let KLookSignatureToolUtil = Java.use("com.klook.base_platform.security.KLookSignatureToolUtil");` `KLookSignatureToolUtil["sign"].implementation = function (data, key) {` ``console.log(`KLookSignatureToolUtil.sign is 调用: data=${data}, key=${key}`);`` `let result = this["sign"](data, key);` ``console.log(`KLookSignatureToolUtil.sign result=${result}`);`` `return result;` `};` `let KLookSignatureTool = Java.use("com.klook.base_platform.security.KLookSignatureTool");` `KLookSignatureTool["sign"].implementation = function (context, bArr, str) {` ``console.log(`KLookSignatureTool.sign is called: context=${context}, bArr=${bArr}, str=${str}`);`` `let result = this["sign"](context, bArr, str);` ``console.log(`KLookSignatureTool.sign result=${result}`);`` `return result;` `};``}``function main(){` `Java.perform(function (){` `hookSign()` `})``}``setImmediate(main);`
```

把上述脚本进行注入，可以看到日志内容如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpgFaf1Fek9ibibzaAzzUDu0xy5vBP5smtpbp7la2HgWc39noXkicIHDFfw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

通过日志可以看到是 KLookSignatureTool.sign 被成功调用的，说明该函数确实是最后生成的地方，因此可以确定到是 libklooktool.so 这个文件实现了这个方法。

使用ida打开该so文件，左侧搜索一下sign可以看到对应的函数方法：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlp0xeaib8w6sSPIK8j3ons3NLfUictWxts1vFtGtFQz8C7sIiaOpCBP7IDQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

双击后按F5看相关代码，然后导入jni.h文件，再切换参数可以看到部分代码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpE2M8fJyicaYyAKVosJe4Gu6S2Z7lZRwawKS4DxFtoXPMVKo27Rb9IMA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

分析（看不懂可以直接丢给AI）：

通过v8的返回值，判断是否需要继续拼接一段固定盐 a2xvb2thbmRyb2lk

再把v14传入到 klook_api_sign_v2 中，生成签名转换成二进制最后返回java层

因此，我们可以跳过v8的返回值，直接去hook 函数 klook_api_sign_v2  的入参，就可以知道有没有拼接这段盐值。

  

进入函数 klook_api_sign_v2 可以看到：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpIO1ibAlZ3TaPKbXTrmoBnXjicOc7cyo2BvhwbUcdDWkZPico95fqzVjxg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11)

  

  

到这后可以先编写hook脚本，尝试打印 klook_api_sign_v2  的入参和返回值：

```
`var targetSo = 'libklooktool.so'``var pollIntervalMs = 500;``var maxWaitMs = 60 * 1000;``function toHex(byteArray) {` `if (!byteArray) return '';` `var u8 = new Uint8Array(byteArray);` `var out = [];` `for (var i = 0; i < u8.length; ++i) {` `var s = u8[i].toString(16);` `if (s.length === 1) s = '0' + s;` `out.push(s);` `}` `return out.join('');``}``function tryHookModule(mod) {` `try {` `console.log(' 尝试在模块 hook:', mod.name, mod.path, ' base=', mod.base);` `var klook_api_sign_v2_addr = null;` `klook_api_sign_v2_addr = Module.findExportByName(mod.name, 'klook_api_sign_v2');` `try {` `if (klook_api_sign_v2_addr) console.log(' findExportByName 找到 klook_api_sign_v2 ->', klook_api_sign_v2_addr);` `} catch (e) {}` `Interceptor.attach(klook_api_sign_v2_addr, {` `onEnter: function (args) {` `// ARM64 convention: x0.. -> args[0], args[1], ...` `this.input_ptr = args[0];` `// 可能是 size_t，尽量兼容` `try { this.len = args[1].toInt32(); } catch (e) { try { this.len = args[1].toInt64(); } catch (e2) { this.len = 0; } }` `this.out_ptr = args[2];` `try { this.out_len = args[3].toInt32(); } catch (e) { try { this.out_len = args[3].toInt64(); } catch (e2) { this.out_len = 0; } }` `console.log('\n---- klook_api_sign_v2 ENTER ----');` `console.log(' addr:', klook_api_sign_v2_addr, ' input_ptr=', this.input_ptr, ' len=', this.len, ' out_ptr=', this.out_ptr, ' out_len=', this.out_len);` `// 读取 input，按 len 读取（限制上限防止 OOM）` `try {` `if (this.len > 0) {` `var raw = Memory.readByteArray(this.input_ptr, this.len);` `try {` `var sutf = Memory.readUtf8String(this.input_ptr, this.len);` `console.log(' input (utf8):', sutf);` `} catch (e) {` `console.log(' input (hex):', toHex(raw));` `}` `} else {` `// 长度为 0 时尝试读取预览（64 bytes）` `var pre = Memory.readByteArray(this.input_ptr, 64);` `console.log(' input (preview hex):', toHex(pre));` `}` `} catch (e) {` `console.log('[!] 读取 input 失败：', e);` `}` `},` `onLeave: function (retval) {` `console.log('---- klook_api_sign_v2 LEAVE ----');` `// retval 一般是 int` `try {` `var r = retval.toInt32();` `console.log(' retval (int):', r);` `} catch (e) {` `console.log('[!] 读取 retval 失败：', e);` `}` `// 读取 out buffer（优先 out_len，再 fallback 32）` `try {` `var outbytes = Memory.readByteArray(this.out_ptr, this.out_len);` `var hex = toHex(outbytes);` `console.log(' out bytes (hex len=' + this.out_len + '):', hex);` `} catch (e) {` `console.log('[!] 读取 out buffer 失败：', e);` `}` `console.log('---- klook_api_sign_v2 END ----\n');` `}` `});` `console.log(' klook_api_sign_v2 hook 安装成功 :', klook_api_sign_v2_addr);` `return true;` `} catch (e) {` `console.log('[!] tryHookModule 异常:', e);` `return false;` `}``}``function pollForModule() {` `var start = Date.now();` `var timer = setInterval(function () {` `var mods = Process.enumerateModulesSync();` `for (var i = 0; i < mods.length; i++) {` `var m = mods[i];` `if (m.path && m.path.indexOf(targetSo) !== -1) {` `console.log(' 发现模块 by path:', m.name, m.path);` `clearInterval(timer);` `tryHookModule(m);` `return;` `}` `}` `if (Date.now() - start > maxWaitMs) {` `clearInterval(timer);` `console.log('[!] 超时未发现目标模块（' + targetSo + '）。');` `}` `}, pollIntervalMs);``}``setImmediate(function () {` `console.log(' wait_and_hook 启动, targetSoHint =', targetSo);` `// 先尝试一次立即查找` `var mods = Process.enumerateModulesSync();` `for (var i = 0; i < mods.length; i++) {` `var m = mods[i];` `if (m.path && m.path.indexOf(targetSo) !== -1) {` `console.log(' 初次发现模块:', m.name, m.path);` `if (tryHookModule(m)) return;` `}` `}` `// 否则轮询等待 dlopen` `pollForModule();``});``// frida -U -f com.klook -l hook_test.js -o text.log`
```

  

这里我做了一个定时循环的操作，我不能很精准的把控hook时机，大佬们如果有好的写法也可以告送我一声。

  

看到结果如下，传入参数似乎是 时间戳 + android + 设备id + 设备id + url（不同的接口组成方式好像不同）

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlplXr7GSUn8Bte9k1nNINtupb4rJTGVtEQ4NdxC7bayOS0kXLKy4b07Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12)

  

然后现在回过头去看 klook_api_sign_v2 的具体实现方式：

分析（看不懂就丢给AI）：

v10 一顿猛虎操作，但是似乎没有传入下一个函数步骤？所以暂时不用理会。

dest声明了一个512字节的缓冲区，然后把 unk_2DF20 其中的部分字节赋值给了dest。

最后把参数传入klook_api_sign_base中拿到结果。

unk_2DF20肯定是常量，取了其中512个字节，然后传入给了klook_api_sign_base。想拿到完整的unk_2DF20，然后再取512字节对我来说挺难的，所以我直接去hook klook_api_sign_base的入参似乎也能达到相同的效果。在刚刚的代码块中新增代码：

```
`var klook_api_sign_base_addr = null;``var dump_context_len = 512; // klook_api_sign_v2 里 dest 长度``try {` `klook_api_sign_base_addr = Module.findExportByName(mod.name, 'klook_api_sign_base');` `if (klook_api_sign_base_addr) console.log(' findExportByName 找到 klook_api_sign_base ->', klook_api_sign_base_addr);``} catch (e) {}``Interceptor.attach(klook_api_sign_base_addr, {` `onEnter: function(args) {` `// args[0] 应该是 context (512 bytes copy of unk_2DF20)` `this.ctx_ptr = args[0];` `this.input_ptr = args[1];` `// len may be args[2] or args[2] is some jint, we will try both when reading` `try { this.len = args[2].toInt32(); } catch(e) { try { this.len = args[2].toInt64(); } catch(e2) { this.len = 0; } }` `this.out_ptr = args[3];` `try { this.out_len = args[4].toInt32(); } catch (e) {` `console.log("入参第四个参数转换错误:",e)` `console.log("args[4]:",args[4])` `this.out_len = 32;` `}` `console.log('---- klook_api_sign_base ENTER ----');` `console.log(' ctx_ptr=', this.ctx_ptr, ' input_ptr=', this.input_ptr, ' len=', this.len, ' out_ptr=', this.out_ptr, ' out_len=', this.out_len);` `try {` `var fullctx = Memory.readByteArray(this.ctx_ptr, dump_context_len);` `console.log('dest | 512 (hex):', toHex(fullctx));` `} catch (e) {` `console.log('[!] 读取 dest 失败:', e);` `}` `},` `onLeave: function(retval) {` `console.log('---- klook_api_sign_base LEAVE ----');` `var outBuf = null;` `try { console.log(' retval (int):', retval.toInt32()); } catch (e) { /* ignore */ }` `try {` `outBuf = Memory.readByteArray(this.out_ptr, this.out_len);` `if (outBuf) {` `var outHex = toHex(outBuf);` `console.log(' out (hex full):',outHex);` `} else {` `console.log('[!] out 读取到空 buf');` `}` `} catch (e) {` `console.log('[!] 读取 out buffer 失败:', e);` `}` `console.log('---- klook_api_sign_base END ----');` `}``});``console.log(' klook_api_sign_base hook 安装成功 :', klook_api_sign_base_addr);`
```

利用上述代码，可以看到dest的值都是相同的

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpHI9xpcH05mz3LVWDYIdw2icAqdJc3qzicjCiafmss4cU5ZOO375yjibPJQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13)

  

但是klook_api_sign_base并不只是传入了这一个参数，同时可以看看别的参数具体是什么，所以可以再次补充代码：

```
`// 读取 input``try {` `if (this.input_ptr && this.len && this.len > 0 ) {` `try {` `var s = Memory.readUtf8String(this.input_ptr, this.len);` `console.log('完整 input (utf8):', s);` `var raw = Memory.readByteArray(this.input_ptr, this.len);` `var hex = toHex(raw);` `console.log('完整 input (hex):', hex);` `} catch (e) {` `// 如果不是纯 UTF-8 文本（可能是二进制）` `var raw = Memory.readByteArray(this.input_ptr, this.len);` `hex = toHex(raw);` `console.log('完整 input (hex):', hex);` `console.log('klook_api_sign_base onEnter input Error :',e)` `}` `} else {` `console.log('[!] input_ptr 或 len 无效, input_ptr=', this.input_ptr, ' len=', this.len);` `}``} catch (e) {` `console.log('[!] 读取 input 失败:', e);``}`
```

  

得到日志内容如下。可以看到除了刚刚的dest，同时也把klook_api_sign_v2的参数一同传入。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpfibjhGAadnJzFxEJyIqpwCEicNPJGvqqSU60Jlp800vpjaiaicMasTRFlQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14)

现在进入klook_api_sign_base内部，查看实现逻辑：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpLpU6ZSEMSQaiaHpWULkXNyecUA24ibpNGQicqyEibyHdJBtmGxfSvibJKKA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=15)

  

分析：

  

到这就有个很亮眼的hmac_sha256，大概可以知道是什么算法了，现在去尝试找一下密钥和待加密字符串。

a2 a3 a4都是参数，中间没做过处理就传入hmac_sha256了，因此主要看v8 v16的生成。

v8就是把a2指向的那个内存中的前10个当作 ASCII 十进制字符逐位解析得到的十进制整数，a2是第二个参数，也就是我们刚刚日志打印中的那个input_ptr参数，这个地址对应的utf8的格式我们刚刚也是打印出来了，要取前10个字符，也就是时间戳的前10位。

v16就是利用v8做了一些一些取模、按位运算后，以这些结果作为索引偏移，然后去a1指向的那个地址，每次取4个字节的内容，一共取8次，最终组成一个32字节的数组。a1是第一个参数，也就是我们刚刚得到的那个512字节的dest。

分析到这，可以对klook_api_sign_base的hook脚本进行补充，打印v8和v16的值分别是什么？同时去打印hmac_sha256的入参，可以确保分析的是否正确。

```
`var klook_api_sign_base_addr = null;``var dump_context_len = 512; // klook_api_sign_v2 里 dest 长度``try {` `klook_api_sign_base_addr = Module.findExportByName(mod.name, 'klook_api_sign_base');` `if (klook_api_sign_base_addr) console.log(' findExportByName 找到 klook_api_sign_base ->', klook_api_sign_base_addr);``} catch (e) {}``Interceptor.attach(klook_api_sign_base_addr, {` `onEnter: function(args) {` `// args[0] 应该是 context (512 bytes copy of unk_2DF20)` `this.ctx_ptr = args[0];` `this.input_ptr = args[1];` `// len may be args[2] or args[2] is some jint, we will try both when reading` `try { this.len = args[2].toInt32(); } catch(e) { try { this.len = args[2].toInt64(); } catch(e2) { this.len = 0; } }` `this.out_ptr = args[3];` `try { this.out_len = args[4].toInt32(); } catch (e) {` `console.log("入参第四个参数转换错误:",e)` `console.log("args[4]:",args[4])` `this.out_len = 32;` `}` `console.log('---- klook_api_sign_base ENTER ----');` `console.log(' ctx_ptr=', this.ctx_ptr, ' input_ptr=', this.input_ptr, ' len=', this.len, ' out_ptr=', this.out_ptr, ' out_len=', this.out_len);` `try {` `var fullctx = Memory.readByteArray(this.ctx_ptr, dump_context_len);` `console.log('dest | 512 (hex):', toHex(fullctx));` `} catch (e) {` `console.log('[!] 读取 dest 失败:', e);` `}` `// 读取 input` `try {` `if (this.input_ptr && this.len && this.len > 0 ) {` `try {` `var s = Memory.readUtf8String(this.input_ptr, this.len);` `console.log('完整 input (utf8):', s);` `var raw = Memory.readByteArray(this.input_ptr, this.len);` `var hex = toHex(raw);` `console.log('完整 input (hex):', hex);` `} catch (e) {` `// 如果不是纯 UTF-8 文本（可能是二进制）` `var raw = Memory.readByteArray(this.input_ptr, this.len);` `hex = toHex(raw);` `console.log('完整 input (hex):', hex);` `console.log('klook_api_sign_base onEnter input Error :',e)` `}` `} else {` `console.log('[!] input_ptr 或 len 无效, input_ptr=', this.input_ptr, ' len=', this.len);` `}` `} catch (e) {` `console.log('[!] 读取 input 失败:', e);` `}` `// 这里解析出 v8 和 v16` `var v8 = 0;` `for (var i = 0; i < 10; i++) {` `var byte = Memory.readU8(this.input_ptr.add(i));` `v8 = byte + 10 * v8 - 48;` `}` `console.log("v8 :",v8);` `var v16 = [];` `for (var i = 0; i < 8; i++) {` `// 从 ctx_ptr（也就是 dest）里按偏移读取 4 字节` `// 注意 Frida 是按字节地址，所以要乘 4` `var offset;` `switch(i) {` `case 0: offset = (v8 % 0x409) & 0x7F; break;` `case 1: offset = (v8 % 0xCEB) & 0x7F; break;` `case 2: offset = v8 & 0x7F; break;` `case 3: offset = (v8 & 0x7F) ^ 0x7F; break;` `case 4: offset = (v8 % 0x177B) & 0x3F; break;` `case 5: offset = (v8 % 0x178D) & 0x3F | 0x40; break;` `case 6: offset = (v8 % 0x25D9) & 0x3F | 0x40; break;` `case 7: offset = (v8 % 0x26B3) & 0x7F; break;` `}` `var val = Memory.readU32(this.ctx_ptr.add(offset*4));` `v16.push(val);` `}` `// 打印 v16` `console.log('v16 :',v16);` `console.log('v16 (derived key):', v16.map(x => ('00000000' + x.toString(16)).slice(-8)).join(' '));` `// 打印完整 32 字节 hex（小端顺序拼接）` `var keyHex = v16.map(x => {` `var b0 = (x & 0xFF).toString(16).padStart(2,'0');` `var b1 = ((x >> 8) & 0xFF).toString(16).padStart(2,'0');` `var b2 = ((x >> 16) & 0xFF).toString(16).padStart(2,'0');` `var b3 = ((x >> 24) & 0xFF).toString(16).padStart(2,'0');` `return b0 + b1 + b2 + b3;` `}).join('');` `console.log('v16 hex (32 bytes):', keyHex);` `},` `onLeave: function(retval) {` `console.log('---- klook_api_sign_base LEAVE ----');` `var outBuf = null;` `try { console.log(' retval (int):', retval.toInt32()); } catch (e) { /* ignore */ }` `try {` `outBuf = Memory.readByteArray(this.out_ptr, this.out_len);` `if (outBuf) {` `var outHex = toHex(outBuf);` `console.log(' out (hex full):',outHex);` `} else {` `console.log('[!] out 读取到空 buf');` `}` `} catch (e) {` `console.log('[!] 读取 out buffer 失败:', e);` `}` `console.log('---- klook_api_sign_base END ----');` `}``});``console.log(' klook_api_sign_base hook 安装成功 :', klook_api_sign_base_addr);``var hmac_sha256_addr = null;``try {` `hmac_sha256_addr = Module.findExportByName(mod.name, 'hmac_sha256');` `if (hmac_sha256_addr) console.log(' findExportByName 找到 hmac_sha256 ->', hmac_sha256_addr);``} catch (e) {}``Interceptor.attach(hmac_sha256_addr, {` `onEnter: function(args) {` `try {` `console.log('---- hmac_sha256 ENTER ----');` `this.a0 = args[0];` `this.a1 = args[1].toInt32 ? args[1].toInt32() : parseInt(args[1]);` `this.a2 = args[2];` `this.a3 = args[3].toInt32 ? args[3].toInt32() : parseInt(args[3]);` `this.a4 = args[4];` `this.a5 = args[5].toInt32 ? args[5].toInt32() : parseInt(args[5]);` `console.log(' arg0(ptr)=', this.a0, ' len=', this.a1);` `console.log(' arg2(ptr)=', this.a2, ' len=', this.a3);` `console.log(' arg4(ptr)=', this.a4, ' len=', this.a5);` `var data0 = Memory.readByteArray(this.a0, this.a1);` `console.log(' arg0(hex)=', toHex(data0));` `var data2 = Memory.readByteArray(this.a2, this.a3);` `console.log(' arg2(hex)=', toHex(data2));` `var s = Memory.readUtf8String(this.a2, this.a3);` `console.log(' arg2(utf8):', s);` `var data4 = Memory.readByteArray(this.a4, this.a5);` `console.log(' arg4(hex)=', toHex(data4));` `} catch (e) {` `console.log('[!] print args failed:', e);` `}` `},` `onLeave: function(retval) {` `try {` `console.log('---- hmac_sha256 LEAVE ----');` `try { console.log(' retval (int):', retval.toInt32()); } catch (e) { /* ignore */ }` `// 尝试读取输出缓冲区` `var outData = Memory.readByteArray(this.a4, this.a5);` `console.log('hmac_sha256 output(hex)=', toHex(outData));` `console.log('---- hmac_sha256 END ----');` `} catch (e) {` `console.log('hmac_sha256 ERROR :',e);` `console.log('hmac_sha256 onLeave retval (raw) =', retval);` `}` `}``});``console.log(' hmac_sha256 hook 安装成功 :', hmac_sha256_addr);`
```

查看日志：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpwo5ibs6P1fVV5axxoZwemq9JU03rD68I8V3L8T7m3prViadFPdn7EDGA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=16)

  

  

分析日志：

查看klook_api_sign_base部分的v8和v16跟刚刚的分析相同，32字节的v16也是传入到了hmac_sha256中，现在可以去一些在线的加密网站中去验证加密是否正确，防止对算法进行了一些魔改什么的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpicWkcXR97Zx8ia91z9SV162amxSFiaeYWczP7zNGvyCSPliaGYHc8KicJlA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=17)

  

  

完美复现，是标准的加密，然后就可以使用python去还原加密流程了

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpPEN1ian55pv8lTa5TqPw7BKLhMJMcMumTyia7KX3k6Dnwf9qROuDdVwA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=18)

算法还原部分就不提供了，拿到完整的dest后可以让AI帮忙生成对应的代码就行了，后续接口请求也不再多说了。

  

  

  

  

**·** **今 日 推 荐** **·**

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0pL7HZQ4fuoRHsiaNedu7NiaVac9X7TCtCx1voA4qQqXRLkiaTIBmXViboLMicuSnYqsdq0cTVk6V4GiaQQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=19)

  

  

本文内容来自网络，如有侵权请联系删除

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0rMmacqPTL1CgX6qeOiaSDlpc6sVRO2CCqYbE5MFribncKIk5IdKibO0JRaCmYHcLjpbsl3CNYuvnYMA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=20)