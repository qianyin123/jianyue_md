> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/lz7Oy4RvEekA5o9TSoqesA)

> 这是一篇完整实战复盘：从“看不懂的 VMP 混淆 JS”，到“可运行、可对拍、纯 Python 实现”。

* * *

一、背景：为什么这个 sign 难
-----------------

QQ 音乐web接口的 `sign` 参数不是普通加密函数，而是藏在 **JS VMP（虚拟机保护）** 里。  
你打开源码后，会看到非常长的一段 `switch-case` 指令分发、字节码解码、动态函数构造，看起来几乎不可读。

核心挑战有三个：

1.  不是直接算法，而是“解释器执行字节码”；
    
2.  混淆后变量无语义，静态阅读成本极高；
    
3.  直接抄 JS 执行可以跑，但不等于“还原”。
    

我的目标不是“能跑就行”，而是：

*   读懂关键执行路径；
    
*   还原出可解释的真实流程；
    
*   最终给出纯 Python 版本，并与 JS 多组对拍一致。
    

* * *

二、工具组合：Trae + AI 模型全家桶
----------------------

这次我采用的是“人给目标，AI 做分层推进”的方式，且针对不同环节动用了不同的 AI 能力：

*   **Trae (IDE)**：负责在工程内快速读文件、改文件、运行验证；
    
*   **Chrome DevTools MCP**：负责在网页调试时定位 sign 参数加密代码；
    
*   **Gemini-3-Flash-Preview (AI)**：负责**环境补全（Mocking）**，解决本地执行与网页输出不一致的问题；
    
*   **GPT-5.3 (AI)**：负责核心策略：先定位 VM 入口，再做动态插桩，最终实现python纯算法。
    

* * *

三、前置：环境补全（Environment Mocking）
------------------------------

在本地获取到 VMP 算法代码后，直接运行往往会因为缺少浏览器环境（window, navigator, location 等）而报错，或者产出的结果与网页完全不同。

![图片](https://mmbiz.qpic.cn/mmbiz_png/s64PlbPUu29VVj0NEnojZhGNdiaKIBuFcOsiclmct2iaPlt1ic2rBk20tXLwkPPuPKfSXXbEeMjeoN6blSnoagGQjtZI4aCbPByI5ME6I4a5E7A/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

这时候我先引导 **Gemini** 帮我生成了一份环境补全代码：

```
global.window = {  
    document: {},  
    navigator: {  
        userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'  
    },  
    location: {  
        host: 'y.qq.com',  
        protocol: 'https:'  
    }  
};  
  
// 暴露给全局作用域，确保 VM 指令能正常读写  
global.document = global.window.document;  
global.navigator = global.window.navigator;  
global.location = global.window.location;  

```

补全环境后，本地执行结果终于能和线上对齐，这是后续所有还原工作的基础。

![图片](https://mmbiz.qpic.cn/mmbiz_png/s64PlbPUu2ibZSNSDzJSsF9hAJI4K5h7OLxUrk2ubONrEQicibfgUuQicpmARiaOZGdX7ys7voRARpF6WtxjnRgCQsgXbf5GREkmT8TCwQ1XqErA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

* * *

四、第一阶段：确认 VMP 结构
----------------

一开始我用的提示词是：帮我用python实现这个source.js(补充环境后的代码)的算法。最终结果确实是在python里使用node调用……

![图片](https://mmbiz.qpic.cn/mmbiz_png/s64PlbPUu2icyqcEkXoQxCnJfnFNAcLE4QWcuYGDZQqbtG0azBRiasC4lQUVRiauJCMJGlQibkVN2RvwSfKs8pib8ax9Ypz2z8pt12HQgDUNzicM0/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

    调整提示词为：你需要先读懂这个js里面vmp的算法，然后再用python    去实现。

![图片](https://mmbiz.qpic.cn/mmbiz_png/s64PlbPUu2icGzIJaoIQBjndjbSBcaWPU7TV8HsX6fdJAfMDkSGibCsiallL6Xem8wpTbAFjZ0cOJDDtUJvhlvKvf1ydxyQudHfavBpFoTALW4/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

    从AI的回答中得知它能够完整读懂vmp的代码，并能大概知道什么算        法，这时候就要继续引导AI使用插装的形式进行调试，推理出算法。

    提示词如下：你可以在vmp里某个执行位置打上log，然后进行调试，      看看能否还原出实际的算法？

![图片](https://mmbiz.qpic.cn/mmbiz_png/s64PlbPUu28DYZqngPcZibQe8yHUE0cAhmwY5STllaDwv33IXJDrKkIVdAM4tt8JOoicdNS0u44M8OYichNCVtpVq16gUc4GgMibIIAt6ahrHiaY/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

    从这次AI的回答里就可以看出已经知道其算法了，只要继续推进即可！

![图片](https://mmbiz.qpic.cn/mmbiz_png/s64PlbPUu28PpTo4BvGzCaCSWeYWP54HQCpgiagzzCUSXMPlIOq8Q0sMdse3aq2hfZjp59bx8yhrIYf9QhBBxbx5EaGMdXJa3XYIbwWRaibwg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

    后面的阶段都是详细讲AI如何进行还原的。

五、第二阶段：在 VMP 执行位点打日志（关键突破）
--------------------------

我新增了一个调试脚本 `vmp_trace.js`，做三件事：

1.  注入 `switch (n[++p])`，记录 opcode 执行；
    
2.  拦截 `case 1 return`，采样返回值；
    
3.  记录关键字符串槽位变化，捕获中间态。
    

最终，这个日志策略帮我定位到三个关键片段：

*   `A`：7位十六进制片段；
    
*   `B`：27位 base64 风格片段（中间可能有 `+` `/`）；
    
*   `C`：8位十六进制片段。
    

签名结构被拆成：

```
sign = "zzc" + A + B + C  

```

* * *

六、第三阶段：从日志反推真实规则
----------------

### 1）先拿最稳定的 C 片段

通过多组 payload 对比，我定位出：

*   `C` 来自 `SHA1(payload).upper()` 的固定索引重排：
    
*   索引为：`[16, 1, 32, 12, 19, 27, 8, 5]`
    

### 2）再定位 A 片段

*   `A` 也是同一个 SHA1 十六进制串重排，长度 7；
    
*   索引为：`[23, 14, 6, 36, 16, 7, 19]`
    

### 3）最难的 B 片段

通过抓到 `rawBOrig`（含 `+ /`）、`rawBClean`（清洗后），再和 SHA1 原始字节做比对，最终确认：

*   先对 `sha1_bytes` 与固定 20-byte key 做 XOR；
    
*   XOR 后 bytes 做 base64（去尾 `=`）；
    
*   再删除 `+ / =`；
    
*   得到 `B`。
    

固定 key 为：

```
5927b396da523afcb134ba7b7840f2858fa179b3  

```

* * *

七、第四阶段：落地纯 Python 实现
--------------------

最终我把 `security_sign.py` 改成了纯 Python，不再依赖 Node 子进程。

核心实现逻辑（精简版）：

```
PART_A_INDEXES = (23, 14, 6, 36, 16, 7, 19)  
PART_C_INDEXES = (16, 1, 32, 12, 19, 27, 8, 5)  
PART_B_XOR_KEY = bytes.fromhex("5927b396da523afcb134ba7b7840f2858fa179b3")  

```

```
sha1_bytes = hashlib.sha1(payload.encode("utf-8")).digest()  
sha1_upper = sha1_bytes.hex().upper()  
part_a = "".join(sha1_upper[i] for i in PART_A_INDEXES)  
part_c = "".join(sha1_upper[i] for i in PART_C_INDEXES)  
part_b_bytes = bytes(b ^ PART_B_XOR_KEY[i] for i, b in enumerate(sha1_bytes))  
part_b_base64 = base64.b64encode(part_b_bytes).decode("ascii").rstrip("=")  
part_b = re.sub(r"[+/=]", "", part_b_base64)  
sign = f"zzc{part_a}{part_b}{part_c}".lower()  

```

* * *

八、验证
----

补环境的代码、网站源代码、python纯算结果均一致。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/s64PlbPUu28OopzqrdaXY1qalUFLgIXNj6ZoZzxhiblGGTMFZazeaqplqicUX9HnCX7PdpOPFibI6QQAPF5zMJF1L1icESS8wQ9Ix8OtGTKamCg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/s64PlbPUu29qq5GMk25gatibeP3c9GbHhsf3gJ0pYavfvIEM3kUdATXUfx9OMcX7r6YRlPsg6jAB0gY8QOfcYxTpTwKGdbcVWAWsnaJiaibELo/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

九、经验之谈
------

可以让AI先跑一份补环境的代码，然后告诉AI再结合log插桩，进行动态调试，找出使用的参数和算法，如果你提前能够知道网站用了哪些算法，可以直接提供AI，这样能更快得到你想要的结果。

总之逆向这块如果你让AI直接得到你想要的结果，可能会不尽如人意（标准算法除外），需要一步一步引导它去实现。

这个站点的vmp其实相对简单，AI整体实现出python算法大概就半小时搞定！AI辅助能够大大提升逆向的效率！