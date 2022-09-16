> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/x6gmo2iK5euw1vt1cIyuCA)

**关注它，不迷路。**       

本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者立即删除！

1. 需求
-----

  

众所周知，某数的js不仅混淆了，而且还是动态的，非常的难以阅读，能让小白直接劝退。

我是研究js混淆代码还原的，希望有一种方法能动态调试还原后的VM，经过一番摸索，它来了。

2. 准备
-----

  

因为需要替换html响应，需要安装 mitmproxy 这个库:

```
pip install mitmproxy
```

  

这个库是 **@渔滒** 推荐的，在此感谢！  

这个库需要配合代理，因此需要修改系统的代理，win系统打开 **网络和Internet** 设置 ，如下图设置代理:

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2jIcesKH7DL2m0mSzLwLzmCq693VmAxGkW0zxK6uUcEV3ImbTrDic3r4ffmbOLVeEjnaNQJCZJ87Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

注意，**设置好后一定要点击保存按钮**。  

3. 编写脚本，替换响应
------------

  

编写如下脚本，保存为 main.py :  

```
`import mitmproxy.http``print('脚本初始化成功')``def request(flow: mitmproxy.http.HTTPFlow):` `pass``def response(flow: mitmproxy.http.HTTPFlow):` `if '某数4代URL' == flow.request.url:` `html = flow.response.text` `print("ret=" in html)` `html = html.replace("{ret=","{debugger;ret=")` `flow.response.text = html`
```

  

这里将 **"{ret="** 替换成 **"{debugger;ret="** 这里将关键位置进行插桩，便于快速定位，至于为什么是这里，请参考其他大佬的文章。

运行如下命令:  

```
 mitmdump -q -p 8888 -s main.py
```

  

打开目标网站和调试面板，清空所有cookie和其他的缓存，再次刷新，停到这里:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2qnAoEr5cRxDXBqecz2P7gERQG7KvOnhGib5zMn2Ffz1c392QiatynLlIZLa0avHoTNVeGEzUwygdQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

控制台输出 _$OF 的值 ： 

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2qnAoEr5cRxDXBqecz2P7gwLGGQ7eDbteu3hAdKzNAkLJH8m40UTGu5eD3K4kNYRibl6Sorp7ynyw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

聪明的你，肯定发现了它就是VM里的代码，并且它是被eval的对象。注意它是字符串，并非代码，因此，复制后，还需要将其 console.log ，转换成代码再进行还原:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2qnAoEr5cRxDXBqecz2P7gZqtOu814usCwsQDqkIeFnXVmZ2WhibkDbC8GsRDj01v6pKE2K40wlAg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

复制保存，并对其进行还原处理，复制还原后的代码，赋值给_$OF :

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2qnAoEr5cRxDXBqecz2P7gialRkibetwnayUnIPq7VHFoiaHrquZ27EuHRhG78fSiaQBQ85icFaN77zfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

注意在第一行加入 debugger;  回车后再次F8运行，停在第一行的debugger位置:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2qnAoEr5cRxDXBqecz2P7g3VZibeHoD1yPYU5rrpOVicfJRI11M2yeLv8cL6IibDMruaSiaoroA3yCZQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

全局搜索cookie,在下面这个位置打上断点:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2qnAoEr5cRxDXBqecz2P7g0Fmld4ZaicGN50CVaDGBMbs85ib7NcTDhiaMjdCWNqB0kalmRwSjgiaheQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

再次F8运行,停在断点位置:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2qnAoEr5cRxDXBqecz2P7gK3P39EQia6lLjKKQQZu405fTLBKYVIj0srCpoa4zd4OpYaWX2wLqnBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

堆栈往上追，进入假cookie生成的地方:

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2qnAoEr5cRxDXBqecz2P7gfa4b9Dpg7xFJuXXqgYkaIcpUDr8zS5mJ3sUn3J3rTsAGWkggvywK6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

再次F8运行,J继续停在断点位置:

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2qnAoEr5cRxDXBqecz2P7g5JmOLZUDDVp4EaMwxQYrEmictEDqd3lRwqLlTxseWY3kY4EbHhqoGBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

堆栈往上追，进入真cookie生成的地方:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR2qnAoEr5cRxDXBqecz2P7gCAf0mexWhQIQFfY2STwpO27frA1Lic2mV6naTxrEib0uyZEB8ZwnxzrA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

说明替换是可以走正确逻辑的。  

4. 后记
-----

  

遇到了两个小问题，记录一下。

1.  需要将字符串转换成代码，使用console.log；
    
2.  需要将还原后的代码转换成字符串，使用了 JSON.stringify，在此感谢 @Echowxsy,后来发现保存为文件后复制，还是会报错，于是干脆直接复制到剪切板，使用了 copy-paste 这个库，问题解决。
    

今天的文章就分享到这里，后续分享更多的技巧，敬请期待。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Lll8tx0MDR0xtvs5q4zuW5BXvXzbAAAdAxXH7PSebBWJT3U9dXG1XtOSKRVDQqictGWKznl3rusg5MAsGO0D6Lw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

  

欢迎加入知识星球，学习更多AST和爬虫技巧。