> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/h3bfnYtW_Cr8sM43M9xufQ)

【目标】  
https://pc.mijialaw.com/detail/v_61b1c651e4b0e472241603f1/3?fromH5=true  
【尝试1】  
    这个视频是m3u8的形式进行播放的，可以观看，但不允许下载。起先我并不知道视频做了加密，所以第一眼想到的是尝试下载ts文件，如果可以的话最后拼接起来就成了。因为直接在新页面打开ts链接会报302，所以用Fiddler抓包进行保存。如下图所示，但是保存下来的文件用播放器没办法播放。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0pdicZrTPjAImXvkCnVDIsyEgHTfNoE41ibvkicZ5zpLfj7NycoK8A2bYD3mXaCnYJyCF0jzcbwI9ffw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  
    【尝试2】  
    前端的播放器大多都是封装video标签，这个网站也是，而video标签默认是有个下载功能的，所以我在控制台修改了一下video的属性和样式，把下载按钮显示出来，但点击后下载的是视频的m3u8文件（有时也会显示下载失败）。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0pdicZrTPjAImXvkCnVDIsyENSE58boQ4iaqf3n3AVLloUt7O0bToowAB7lm1rEmP7I8g5DFgibxDBNQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  
    【尝试3】  
    接下来，想着查看了一下它是否使用了什么播放器插件，但从文档加载的js名称上没有发现明显的信息，只知道大概是使用了某鹅通的服务。  
    于是，准备调试js文件，结果半途发现了xg-player的字样，搜索了一下，是某瓜播放器的插件，果断找到文档看看有没有什么提供什么机会。发现它有“下载”功能，于是修改js文中的配置内容，把插件的下载按钮显示出来，但遗憾的是，它下载下来的也是m3u8的文件，虽然不同时间链接地址不一样，但内容是相同的。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0pdicZrTPjAImXvkCnVDIsyEDQribQBYf5iaweZcHVe9MujzKsGF6JcI6lMglqc5Jn7pUJolq5WCWtjQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  
    【尝试4】  
    只好打断点调试代码了，在跟踪的过程中碰到几处Array Buffer数据，但直接下载下来都不能播放，也看到了它有个加密解密的一些内容，知道这些视频片段刚下载过来是经过加密的，不能直接用。  
    不过，既然网站内的播放器能播放，说明解密也是在前台做的，但是代码太多，搜索的几个关键词也都数量特别多。但收获也不是没有，发现里面有个hls的变量，下载的ts数据最后都被它用方法摄取过去了，此外还看到很多worker的字样，联想到webworker。猜测ts数据下载下来以后被转交给worker去解码，然后再反馈给主js线程，那最终从webworker出来时，主线程为了接收数据，应该监听了message事件。于是搜索了一下message事件，只有9个，似乎问题被大大简化了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0pdicZrTPjAImXvkCnVDIsyEx01tuROP1ONmbvCpViclePGXLnf5IPR3dt5UQPFH8PibuRJgmSDpPHlA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    经过一番跟踪，找到一个onWorkerMessage的函数（从函数名的语义来看，应该没错了），每次它运行完成后视频就会继续播放，但是代码看起来都有点似是而非，感觉总抓不到关键点。于是，到网上搜索了一下这种视频体系一般是怎么个流程。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0pdicZrTPjAImXvkCnVDIsyENy8nNsUEbEVlTMfFfUV9UBdOQXNmicwExdPXBMgvH0XpWFwxz32zxrA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  
    这里可以看到，在数据到达video标签进行播放的过程中，离它最近的前一站是MSE（MediaSource），且在调试过程中也发现hls中挂了一个MediaSource的对象，看来确实用到了这个API，是这么个体系。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0pdicZrTPjAImXvkCnVDIsyExD9OpSG2Qv0023tibjtpemicszE6gbTrreUjNCQFP0YJzwYjHib7DAGhw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  
    既然最终流到video标签的数据是MediaSource提供的，那么就查询一下这个API的方法，所幸方法不多，只有添加、删除、结束三个。添加方法（MediaSource.addSourceBuffer()）无疑是关键，这个方法是用来摄取外来数据的，到这个环节了，数据必然已经解码了，不然它也不能识别。文档中说addSourceBuffer可以“创建一个带有给定 MIME 类型的新的 SourceBuffer 并添加到 MediaSource 的 SourceBuffers 列表。”，而Array Buffer数据是通过SourceBuffer的appendBuffer来添加的。于是，搜索“appendBuffer”，共找到两处，一个用作判断的条件，排除掉，就只剩一个了。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0pdicZrTPjAImXvkCnVDIsyEtFqxDiaMrcqqfzSO9EyhSqowDXWficw8bGic8zic64NsQfsXsYamWKpZeg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  
    接下来，在此处添加代码将数据data下载下来看看。

```
`let _a_ = document.createElement('a');let _blob_ = new Blob([data], {type:'video/mp2t'});``let _url_ = URL.createObjectURL(_blob_);``_a_.href = _url_;``_a_.download = 'download.ts';``_a_.click();`
```

遗憾的是，下载下来的文件还是不能播放。

    然后在此处打断点调试，发现这里的数据分两种，一种是audio一种是video，这就奇怪了，这里两个资源居然还是分开的，那问题来了，它们什么时候合并呢？但怎么都找不到呢？  
    到这里调试遇阻，于是打算到网上随便看看多了解一下这种模式，无意中看到一个油猴脚本“media-source-extract”，据说它可以下载加密的流数据，而在它的使用说明中发现它是需要视频播放完成才能下载，于是我想可能片段就是不能播放的，只有完整的才行。接下来试了下这个插件，发现它下载下来的文件是两个，一个是视频一个是音频。难道压根视频和音频就是分开的，不合并的？于是我调整了一下代码，将视频片段和音频片段分别储存到变量中，然后让视频播放一遍，播放完成后，我将两个数据下载下来，发现可以成功播放了。

```
`// 用变量将视频片段存储起来window['_chunks_' + e] = window['_chunks_' + e] || [];``window['_chunks_' + e].push(data);`
```

```
 `// 下载代码let _a_ = document.createElement('a');``let _blob_ = new Blob(_chunks_video, {type:'video/mp2t'});``let _url_ = URL.createObjectURL(_blob_);``_a_.href = _url_;``_a_.download = 'download.mp4';``_a_.click();`
```

  最后只需要找个剪辑软件之类的将二者合并就行了。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0pdicZrTPjAImXvkCnVDIsyEgTHVNf6rN92acR8juRTKyFWjsakrYYZdoBVibO4oRgN1xddYTnMH8wA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  
  
    更多细节可查看视频展示：https://www.bilibili.com/video/BV1o841187hu/

  

  

* * *

**关 注 有 礼**

  

  

欢迎关注公众号：逆向有你，

后台回复：20221116

获取每日抽奖送书

本文内容来自网络，如有侵权请联系删除

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0pdicZrTPjAImXvkCnVDIsyEun2wLZTE7X6GlIEADQrNKfBpjyqFAzGLjicjicHu0ENGJvicRBZRJfibMQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)