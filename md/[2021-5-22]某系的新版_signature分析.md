> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6m5bNU1qJlRsSCmAkaJziA)

```
项目地址：aHR0cHM6Ly93d3cudG91dGlhby5jb20vYy91c2VyL3Rva2VuL01TNHdMakFCQUFBQWlBY2U1cWhIMzFUZXVCM1VkcEZNVjh1LXV3eTJMbm9pcUkxMHVaSHFBdDgvP3NvdXJjZT1mZWVk
```

**观前提示：**

**本文章仅供学习交流,切勿用于非法通途,如有侵犯贵司请及时联系删除**

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQH9yE4Wcf1vkBrS6ByI5ZLialn16t7kqX8TibH0OB7bZBhqsx5xJTXyiacw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

**前言**

太久没更新了 差不多半年了吧 突然想起自己还有个公众号 赶紧回来更一篇

这次给的例是挺多人搞过个 听说前段时间更新过一次 就重新看了一下 其实也差不多 万变不离其宗的

  

  

  

  

**定位加密位置**  

直接全局搜索 _signature

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHqWXIfJ5Fic9A27lzLlEwhlKFNy5cpIP7IKiaBib9Wiba917nlBiaN3r9GdQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHPSs7GtlqnGeC0HRApciaS2ZaRubIcWdUxvspMZu2sVRrGicMibNWBzrhA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到变量 n 对应的就是 j 函数 打上断点 跟进去  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHBbibp3YwRiccHxzKuwuaOZUP4hEfxmqEAkNNdW0N2xBvtLMvI2ZHCHXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

进来后步进一步一步看  

```
(null === (a = window.byted_acrawler) || void 0 === a ? void 0 : null === (n = a.sign) || void 0 === n ? void 0 : n.call(a, o)) || ""`br`
```

到这里 就找到我们所需要的加密入口了

  

**代码分析**  

接着上面的位置打上断点 继续步进会进入到一个新的js页面  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHNRicuZLnz5OQx2FatvcUVgSMF3ib4R22rOrRtldw8luicsmFfhxtEwWLA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里就是我们需要分析的代码了 全部复制到node里面运行看看

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHSKqV9L9N6HdNUeRBSrV9Qice0mlvmLYZIRu2XrNxd9DIyLryWwUBGGA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

报错 这不就是要补点东西给它嘛  

直接去浏览器取一个 referrer 补上即可

补完后再次运行发现不报错了 那我们调用函数看看

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHS7c23Dpa54uL6PSb6z37S4ZrRySVicOjGKEn6Sics3XP7BvLJqWgRVkw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

出现了第二个错误 这是为什么呢？

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHFuwufSF5ia5DwlnfgtGgyFy4sViaibiaUtN7yoG5Xic9LdmJKk8zPbDVxEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过观察代码得知 新版的代码尾巴加了点基本检测 整理一下list

```
`[` `,` `,` `"undefined" != typeof exports ? exports : void 0,` `"undefined" != typeof module ? module : void 0,` `"undefined" != typeof define ? define : void 0,` `"undefined" != typeof Object ? Object : void 0,` `void 0,` `"undefined" != typeof TypeError ? TypeError : void 0,` `"undefined" != typeof document ? document : void 0,` `"undefined" != typeof InstallTrigger ? InstallTrigger : void 0,` `"undefined" != typeof safari ? safari : void 0,` `"undefined" != typeof Date ? Date : void 0,` `"undefined" != typeof Math ? Math : void 0,` `"undefined" != typeof navigator ? navigator : void 0,` `"undefined" != typeof location ? location : void 0,` `"undefined" != typeof history ? history : void 0,` `"undefined" != typeof Image ? Image : void 0,` `"undefined" != typeof console ? console : void 0,` `"undefined" != typeof PluginArray ? PluginArray : void 0,` `"undefined" != typeof indexedDB ? indexedDB : void 0,` `"undefined" != typeof DOMException ? DOMException : void 0,` `"undefined" != typeof parseInt ? parseInt : void 0,` `"undefined" != typeof String ? String : void 0,` `"undefined" != typeof Array ? Array : void 0,` `"undefined" != typeof Error ? Error : void 0,` `"undefined" != typeof JSON ? JSON : void 0,` `"undefined" != typeof Promise ? Promise : void 0,` `"undefined" != typeof WebSocket ? WebSocket : void 0,` `"undefined" != typeof eval ? eval : void 0,` `"undefined" != typeof setTimeout ? setTimeout : void 0,` `"undefined" != typeof encodeURIComponent ? encodeURIComponent : void 0,` `"undefined" != typeof encodeURI ? encodeURI : void 0,` `"undefined" != typeof Request ? Request : void 0,` `"undefined" != typeof Headers ? Headers : void 0,` `"undefined" != typeof decodeURIComponent ? decodeURIComponent : void 0,` `"undefined" != typeof RegExp ? RegExp : void 0``]`
```

根据这段代码稍微补一下  

最终得出  

```
`var exports = undefined,` `module = undefined,` `define = undefined;`
```

可能目前还没用到其他检测 不代表以后风控不会加强 这三行还可以再简化的  

v8环境可以不用补这几段  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHzw5IzrvV0ibnHpT9d1Czua0jeFbMg0KUZG7e1ewJJVqtwzrdB1n7b7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

加上这一段后成功拿到运行函数

接着就是带上参数跑起来看看啦

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHibPovAYLNp3M8qMJiau2XB6vWKN8AYRl7m7He4ibnXRzLDcY11yUvTmYA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

又是一处报错 还是老规矩 报啥错就补啥

最终 补了 href protocol userAgent

可能有人会说:欸 小lin啊 你怎么知道需要这些环境的 它没报错这些啊？

不要慌 可以用代理大法or插桩大法

找到指定位置 打上console 看看调用了啥就行啦

搜索 S[R] = S[R][A] 可以搜索到俩处 都可以插桩

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHDG8feUDfxicfCYhEqRA7Zc81CFhK3Yd631ib7GS3zvGpSxvaDURhJxxA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHllfq3at0LeoZC4S9TCGDOMyOmKIakyEykblJricyNn7QGHUP9KMHtzQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

补完便可以拿到值了  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHk27C9MlWkxzeeOVH9BpibkIOaR1OGAEVZVFIwibfKdKqL5eIic7hia92wg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

拿到的值再postman里面测试一下看看  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHFXmD2PKRbXGibkyrlxqY16Zz7cTufsugW7txO3NGTdianhJic5akLE6Mg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

成功拿到数据 完美撒花？？？？？

有人又要问了：小lin啊，你这怎么这么短啊 我看别人都好长 就你的这么短

wocao 怎么可以被说短呢 那可不行啊  

用插桩or代理可以看到 其实 这js里面还取用了cookie值

那不是补上就好了吗 这简单  

某个人胸有成竹的补上下面的代码并运行

```
window.document.cookie="tt_webid=6963122183099254302; ttcid=ae060c139b024662a70f3f5c7efd19ed50; csrftoken=25b800355bc69db20102b071c3e7de5d; __ac_signature=_02B4Z6wo00f01WaVS3gAAIDAsVNmA2eZ1WFmsU.AADkI7yNYhjiAwFahZq5.fikL135xcMOnizL69KL-qaL2IXBoHlxsgXmUcQhrJCbv27uj.DSexI7E4xFkhJHZ-eQ0YsZWIOXJA8LdEm9v0a; _S_WIN_WH=1920_937; _S_DPR=1; _S_IPAD=0; s_v_web_id=verify_koy65lqk_AAlwyoBG_5x4R_4Zpk_A4oP_vr5u0TzrzdJb; MONITOR_WEB_ID=eab8ca9e-33f9-46c3-ad16-4b35d4696e87; ttwid=1%7CaViRLWI19OZq4nUGH_emezypLNf4sD0OHrJvplMlFVw%7C1621594797%7C509d995bfabe7218415528577a976413c3898116ea6754d78532ae64ca993512; tt_scid=zklw0zHPt-DKwJmAGLQCxJTDlquEzMCO-Nd8g2S3mInLIwTlXdsCcQw34GSAbf7b17c9"
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHHmEpgsmfdSGQbUcuj1cLExTm4h8SqHsLcAbr6aKDjfZ7JNIwtyJYxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

emmmm..........  

cao！

这不是玩嘛  

仔细思考了一波 该不会cookie在整段函数运行的时候给重写了吧  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHQ0eGY02CGLSmtkibpwaEQ6xuJoVOYBy4XTIa1W1cwDyiaAgMGrgsQ1LA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

害 太聪明了我 果真是给重写了 既然如此 我就稍后写入cookie不就行了嘛

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQH5kpWVIC4WEntibQRWvQUKKiaa7OzOic6NI3icbHPVOj80KDwX20suEwhrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这不就行了嘛 我也很长的

最后再放到postman里面测试一下

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsPoOria1d04CyCficPxExUTQHCuhwTXSyhJO3IHicn927ZxqsuFkgrCSLneXps157ZQic7UNLsZVBOAKA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

正常拿到数据 牛啊牛啊

  

  

  

  

**完美撒花 感谢各位大佬观看**

[完]

* * *

  

长按点击关注

逆向lin狗

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOTMgbfKeAjJUe3dWvWvbL11ra5SAZDMOjU0r74uA9luNjj8lS9B67dbic0E5jicVygXhLc6dialAgiaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)