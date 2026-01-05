> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/na6O86mTikkgl5k-LXM8ig)

抓包
==

注意：sign值不统一 注意甄别

* * *

版本：6.5.2

找到如下搜索请求。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nA39m5hRmbZgibGm6gePJvCVwI9NYWZu6pX3YcBUeMhceSo4TfWYoMAUw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

可以发现请求头和请求题有很多值都需要。

分析
==

这里发现一些参数生成逻辑如下所示

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAYAaibpM6zNRm3vichLgP2KmPcGTwsYvst5UnNlPULgiaJGmToYiaA2LLRw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

Version
-------

写死 版本号

noncestr
--------

随机值 逻辑如下

```
for _ in range(i):  
  if random.randint(0, 1) == 0:  
      if random.randint(0, 1) == 0:  
          sb.append(chr(random.randint(65, 90)))  # A-Z  
      else:  
          sb.append(chr(random.randint(97, 122))) # a-z  
  else:  
      sb.append(str(random.randint(0, 9)))  

```

timestamp
---------

开机时间+偏移

cuid
----

用户User的id

如果没有就为0

cfrom
-----

写死

wm
--

基于厂商判断的一个缓存值。

Huawei / Nubia / other

可以默认写死

其他
--

其他值有的是设备注册带过来的值，有的是固定值

可暂时写死

sign
----

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAPgSJ8VIMuPYpNZia40Mnl7qqX0PicYvrRPNtfJooKbfp5Lxaj7XhSicCw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAXXNk5fwsaEd6OEUBLLyIReIzCw6Ficm0Ko36FY63ic72icuwjoDMzOSSw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

那目标就是这个s了 在oasiscore这个类中。

Frida 一下 果然如此

```
Java.perform(()=>{  
    let NativeApi = Java.use("com.weibo.xvideo.NativeApi");  
NativeApi["s"].implementation = function (text, isDebug) {  
    console.log(`NativeApi.s is called: text=${text}, isDebug=${isDebug}`);  
    let result = this["s"](text, isDebug);  
    console.log(`NativeApi.s result=${result}`);  
    return result;  
};  
})  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAUIiahZ8Ujx2fdH8ctsd1Ck2fAgXXibRdZd2MZnzVgLKVtlSqowBWBiatQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

可以发现这一坨是什么呢。其实我们看到了 97 105 和 100 就可以自己算下

97 => a

105 => i

100=> d

那其实这一坨就是params的传餐

使用如下命令转化结果。

```
var jsText = str1.toString();  
    return jsText  
        .split(",")  
        .map(function (num) {  
            return String.fromCharCode(parseInt(num.trim()));  
        })  
        .join("");  
}  

```

再使用如下代码转换为主动调用

```
Java.use('java.lang.String').$new(str).getBytes();  

```

unidbg
======

这里我们提前用frida hook下地址。

```
var addrRegisterNatives = null;  
  
var symbols = Module.enumerateSymbolsSync("libart.so");  
for (var i = 0; i < symbols.length; i++) {  
    var symbol = symbols[i];  
    if (symbol.name.indexOf("art") >= 0 &&  
        symbol.name.indexOf("JNI") >= 0 &&  
        symbol.name.indexOf("RegisterNatives") >= 0 &&  
        symbol.name.indexOf("CheckJNI") < 0) {  
  
        addrRegisterNatives = symbol.address;  
        console.log("RegisterNatives is at ", symbol.address, symbol.name);  
        break  
    }  
}  
if (addrRegisterNatives) {  
    Interceptor.attach(addrRegisterNatives, {  
        onEnter: function (args) {  
            var env = args[0];        // jni对象  
            var java_class = args[1]; // 类  
            var class_name = Java.vm.tryGetEnv().getClassName(java_class);  
            var taget_class = "修改这个为类的地址";   //111 某个类中动态注册的so  
            if (class_name === taget_class) {  
                //只找我们自己想要类中的动态注册关系  
                console.log("\n[RegisterNatives] method_count:", args[3]);  
                var methods_ptr = ptr(args[2]);  
                var method_count = parseInt(args[3]);  
                for (var i = 0; i < method_count; i++) {  
                    // Java中函数名字的  
                    var name_ptr = Memory.readPointer(methods_ptr.add(i * Process.pointerSize * 3));  
                    // 参数和返回值类型  
                    var sig_ptr = Memory.readPointer(methods_ptr.add(i * Process.pointerSize * 3 + Process.pointerSize));  
                    // C中的函数内存地址  
                    var fnPtr_ptr = Memory.readPointer(methods_ptr.add(i * Process.pointerSize * 3 + Process.pointerSize * 2));  
                    var name = Memory.readCString(name_ptr);  
                    var sig = Memory.readCString(sig_ptr);  
                    var find_module = Process.findModuleByAddress(fnPtr_ptr);  
                    // 地址、偏移量、基地址  
                    var offset = ptr(fnPtr_ptr).sub(find_module.base);  
                    console.log("name:",class_name +" "+ name, "sig:", sig,'module_name:',find_module.name ,"offset:", offset);  
                }  
            }  
        }  
    });  
}  
//frida -U -f com.sina.oasis -l RegisterNatives.js  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAFDeeX3kUjUCsGCYzDKLCMk1icrmE0mbjDVKQEp850LVO6jog9akuSVg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

找到S的位置。然后我们去伪造unidbg

使用如下方法

```
module.callFunction(emulator, 0x基值, args.toArray())  

```

最后成功出值。

什么都不用补

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nA1TYrWQibH9Kaeiarb5MgwxpFMOsGKWw2jfHG3533xbQDRTOHPR0Ekdbg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

对后对比下so结果也是一样的

SO层
===

但是这里我们看下So

看下整体视图。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAgvbm6Ix8bx8DWXEXPKdDKwg1Evzzzziaz3vpzdcXEDAldYxhiaCwnSqA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

可以看到这算是个比较明显的OLLVM

右上角主分发器 然后跳转执行。

我们看看分发器逻辑。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAZRibIs01LNhib5wkDw7gxwQHRKrcu04vgJUKamqW99ZL0Q3hzTTjjiaeg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

通过修改V15的 然后跳转逻辑块。

然后通过修改v15 然后嵌套大量if else 来完成整个流程。

这里我们可以简单看下so。

```
while ( 1 )  
  {  
    while ( 1 )  
    {  
      while ( 1 )  
      {  
        while ( (int)v15 > 399029958 )  
        {  
          if ( (int)v15 <= 1006975887 )  
          {  
            if ( (int)v15 > 495949214 )  
            {  
              if ( (_DWORD)v15 == 495949215 )  
              {  
                v23 = &v52[a5];  
                for ( i = 1093145561; ; i = -514631672 )  
                {  
                  while ( i == 1093145561 )  
                    i = v32;  
                  if ( i == -514631672 )  
                    break;  
                  v42 = v23;  
                  memcpy(v23, a8, a7);  
                  v23 = v42;  
                }  
                v15 = 37573277;  
              }  
              else  
              {  
                v15 = v33;  
              }  
            }  
            elseif ( (_DWORD)v15 == 399029959 )  
            {  
              v51 = v54 + 1;  
              for ( j = 954318854; j != -1656068248; j = -1656068248 )  
                ;  
              v26 = operatornew(v54 + 1);  
              v15 = v30;  
              v52 = (char *)v26;  
            }  
            elseif ( v36 == 23 )  
            {  
              v15 = 3602604382LL;  
            }  
            else  
            {  
              v15 = 144921241;  
            }  
          }  
          elseif ( (int)v15 > 1476513897 )  
          {  
            if ( (_DWORD)v15 == 1476513898 )  
            {  
              for ( k = 1093145561; ; k = -514631672 )  
              {  
                while ( k == 1093145561 )  
                  k = v31;  
                if ( k == -514631672 )  
                  break;  
                memcpy(v52, v50, a5);  
              }  
              v15 = 994172386;  
            }  
            else  
            {  
              if ( v53 )  
                v19 = -761086421;  
              else  
                v19 = -514631672;  
              for ( m = 1093145561; ; m = -514631672 )  
              {  
                while ( m == 1093145561 )  
                  m = v19;  
                if ( m == -514631672 )  
                  break;  
                v43 = v19;  
                memcpy(&v52[a5 + a7], &v50[a5 + a6], v53);  
                v19 = v43;  
              }  
              v15 = 495930070;  
            }  
          }  

```

这里只展示部分。

可以看到 代码中部分

例如：

v26 = operator new(v54 + 1); 申请了部分内存。再通过memcpy 进行拷贝。最后进行拼接。

基于这块去找下函数。总体还是太乱了 这里可以使用插件解一下混淆，比如：D810或者deflat

从这里慢慢去看。

找到对应的值。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAibrLsbYo3icIeeb9R312RgogwicdZfCibicRogFMOjRzLrR95GKIVtDMlFw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

我们可以看到文中开头生命了 四个变量 v3，v2，v4，v5

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAa07j7SpqTraPXzFHgialUAOtHxm7ntWEeicS1Du2usVwL1V9nAeXCBpQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

下文 n0x40 < 0x40

代表了循环处理 **64 字节** (0x40) 的数据块

很像md5.

所以我们这个直接frida 调用这个方法看看传参和结果。

使用frida hook下。

> 搜索下我们的传餐 然后反复对比前面的值

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAV8aP4VekwCIkVlUpYCicn5xhv4L0BdlANicOwS37uzhajCwAtDwoibTTA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11)

最后放到网站上对比下和unidbg对比下。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAVeOVUsoTtiafP12P4z4XFChYdenoicyibe9icaesqYhMuBlNG7tlsNyYicQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12)![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAjaiboayibLdTq4F3P56aaX3shViaPf6V8WGDNCNAPHiapcnvCBTnLj7Ckw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13)

然后就完美解决了

最后试下接口

![图片](https://mmbiz.qpic.cn/mmbiz_png/UNlsdSygkENaDnkGZPOzmhDYnibIKL0nAfDtAibsG9MKCbA4KicBI5CfMdbgaSBTW3yYX1yR2abbCL9T9h9uOTic2g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14)

结尾
==

今天已经是2025年最后一天了。最后两个月确实没怎么发文了，今天浅浅水一篇文章。提前祝大家2026新年快乐