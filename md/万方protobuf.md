> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/s7VZt93mRa0a_YV6zkaKqQ)

前言

    本篇文章仅供交流学习，侵权删(满满的求生欲)

    看了3篇有关万方的文章，下面是链接，不过我也会讲的很详细滴。遇到的坑我会一一说出来。

    https://blog.csdn.net/weixin_43411585/article/details/123151047

    https://blog.csdn.net/qq_35491275/article/details/111721639

    https://blog.csdn.net/m0_67391270/article/details/123349453

    那么就开始吧。首先网址是:

    aHR0cHM6Ly9zLndhbmZhbmdkYXRhLmNvbS5jbi9wYXBlcg==

  

如何模拟请求

![图片](https://mmbiz.qpic.cn/mmbiz_gif/bL2iaicTYdZn44gvyOMEqwMzru2icHdoQKMejH40H5iba9IWKAAlz9ZjRrOgFdRJnEkiatdD3Lp8vN3Hichjnib7NZmhg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

随便搜索一点东西 

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIbfJYibVPEZpBxrGeFhumEziaFTRtwxPUiar1sKCBP22V12Z7tY0bCQPWJOlticKCcauQicn2CyYL54ANA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIbfJYibVPEZpBxrGeFhumEziaeBCHBSWyS28L5Ad1msJ1GIn3W0WqA0CIgCc6ZhXxr26JnEy87jCXaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

那么我们就要先解决发请求的问题

### 1.首先打开fiddler抓包，请求查看选择HexView，然后选择一定范围黑色的十六进制数，然后右击选择save selected bytes保存为.bin文件，比如保存为source_form_data.bin文件，放到桌面上

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIbfJYibVPEZpBxrGeFhumEzia6motiborKkeSbtvOicgsdQmf8Fg5Ne7o91PLwf6XWPrnVOZxiayqGfd0Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后打开pycharm

```
import requests,blackboxprotobuf  
  
with open(r"C:\Users\Administrator\Desktop\source_form_data.bin", "rb") as fp:  
    data = fp.read()  
    deserialize_data, message_type = blackboxprotobuf.protobuf_to_json(data)  
    print(f"原始数据: {deserialize_data}")  
    print(f"消息类型: {message_type}")  # 消息类型  

```

### 这个时候，一定范围的作用就出来了，根据我的试验，如果选择全部黑色的字节保存，运行上面的代码会报错，如果报错了，就比图片上多往前选择一个字节，多尝试，这是第一个坑

运行完会发现打印出了这样的数据。

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIbfJYibVPEZpBxrGeFhumEziaeRIAsuuxF1IJPHBvLmBnaKdK2JpFfmektJDNlZWoZFPSRR4OlQibEvw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 直接把上面的复制下来，然后定义两个变量(我这里不一样是因为后期有修改，不重要)

```
deserialize_data = {  
  "1": [  
    {  
      "1": "paper",  
      "2": "dd",  
      "5": "1",  
      "6": "20",  
      "8": "\u0000"  
    }  
  ]  
}  
message_type = {'1': {'type': 'message', 'message_typedef': {'1': {'type': 'bytes', 'name': ''}, '2': {'type': 'bytes', 'name': ''}, '5': {'type': 'bytes', 'name': ''}, '6': {'type': 'bytes', 'name': ''}, '8': {'type': 'bytes', 'name': ''}}, 'name': ''}, '0': {'type': 'bytes', 'name': ''}, '2': {'type': 'bytes', 'name': ''}}  

```

### 第二个坑是要把原始数据和消息类型后面数据的type对应上，是字符串，就改成bytes就好了，像上面这个图里面就全是字符串嘛，所以消息类型中的int就都要改成bytes，改完后就是上面的代码

2.现在我们可以发请求了

```
form_data = bytes(blackboxprotobuf.encode_message(deserialize_data,message_type))  
  
response = requests.post(url, headers=headers, data=form_data,timeout=10,verify=False)  
response_data,response_type = blackboxprotobuf.protobuf_to_json(response.content)  
print(response_data)  

```

###   

如何正确请求

![图片](https://mmbiz.qpic.cn/mmbiz_gif/bL2iaicTYdZn44gvyOMEqwMzru2icHdoQKMejH40H5iba9IWKAAlz9ZjRrOgFdRJnEkiatdD3Lp8vN3Hichjnib7NZmhg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

### 但是你会发现请求有时候成功有时候不成功，而且就算成功返回也是空

### 这个时候就要构造字节序列的头部，修改后代码

```
form_data = bytes(blackboxprotobuf.encode_message(deserialize_data,message_type))  
bytes_head = bytes([0, 0, 0, 0, len(form_data)])  
response = requests.post(url, headers=headers, data=bytes_head+form_data,timeout=10,verify=False)  
response_data,response_type = blackboxprotobuf.protobuf_to_json(response.content)  
print(response_data)  

```

### 第三个坑来了！！！！现在每次请求都会成功，但是还是不返回数据！在反复观摩3篇文章后都没有发现有用的信息，大佬们好像都是直接请求就成功了，反复思考后，我决定使用fiddler抓包自己python发出的请求，然后和网站发出的请求做对比，看看到底有哪里不同！

图一，是我python发出的请求

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIbfJYibVPEZpBxrGeFhumEziar6uus5VMicH0e9eo4bgy6NcoDichSqW7qCN9GB8E0gHoVCfJVZniaBgoA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图二，这是我python构造的原始数据

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIZX6wKP1Bhv5yUspicSV88FUIv945BlyjPDvnUaQ1YSnCiaTI9nwNE2rzduWzs2Kas2ZVKLicPsQxD1A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图三，是网站发出的请求

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/wvdY3149yIZX6wKP1Bhv5yUspicSV88FU4ciaZcJGUd8aKsa3eFBmQIyzlR1LFsibMcUFDDYtY0JwpUicG83ViaaMkw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图一和图三，一比较，就很明显了，好像前后多了点东西，最后多的0…22…1，是不是就是图二中的最后"0"和"2"对应的数据，既然多了，我们就把它删掉！最终我构造的原始数据

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/wvdY3149yIZX6wKP1Bhv5yUspicSV88FUbaic4R6Mcgy1iaUQibrbRCsibjLibqOibiaic1LR27Sibky0ayXaDE6HzCdETjQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

就是这样了，打印一下response.content，然后你就会发现，请求返回了数据！密密麻麻一大堆。但是转成json又会报错

```
ValueError: Invalid wiretype for field number 0. int is not wiretype 1  

```

### 这个的原因看文章所说可能是前后有一些没用的字节，要把它去掉

首先我们选择响应数据的前5个字节

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/wvdY3149yIZX6wKP1Bhv5yUspicSV88FUESQzaXAJxGKZPs0VPbADvWUa0nKsKQUl2YtibXQp2YmaldyeRxqthiaw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

然后去16进制转10进制网站转一下

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/wvdY3149yIZX6wKP1Bhv5yUspicSV88FUoGXQicBvoYy3Tr1YpVo8GBEbiakT9NBmbLbCH0IlxDnZibKsiaok0WnTVw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

那么数据对应的位数就应该是63630(至于为什么前5个字节就是数据的长度，我也还不了解，如果有大佬可以解答一下，感谢！)

### 而我们看下图中响应数据的长度是63655，去掉前5位，63655-63630-5 = 20，所以后面20位也是没用的字节，我们可以请求完对response.content进行切片去掉前后没用的字节。这是第四个坑，也是最后一个坑

所以最后的代码就是

```
form_data = bytes(blackboxprotobuf.encode_message(deserialize_data,message_type))  
bytes_head = bytes([0, 0, 0, 0, len(form_data)])  
  
response = requests.post(url, headers=headers, data=bytes_head+form_data, cookies=cookies,timeout=10,verify=False,proxies={'http': 'http://127.0.0.1:8888', 'https': 'http://127.0.0.1:8888'})  
print(response.content)  
response_data,response_type = blackboxprotobuf.protobuf_to_json(response.content[5:-20])  
print(response_data)  
print(response)  

```

结果也出来了，好耶！

后记

![图片](https://mmbiz.qpic.cn/mmbiz_gif/bL2iaicTYdZn44gvyOMEqwMzru2icHdoQKMejH40H5iba9IWKAAlz9ZjRrOgFdRJnEkiatdD3Lp8vN3Hichjnib7NZmhg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)   

    文章发完后，有大佬对文章中的问题进行了解答  

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIZX6wKP1Bhv5yUspicSV88FUNV6mg9bTydBMGMrGVAs0Htibs8u3ibichYomJyENvCE1hYrrjiaG7mcsZQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIZX6wKP1Bhv5yUspicSV88FUjibWm6GNm8R8LjSCTQmF8qyicXqscCmG9IMZpbNgA5fCCso02HuWD0QQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIZX6wKP1Bhv5yUspicSV88FU98wD6AcpUiaicMVH6KPzjfuic8On8ucpYPRA9uZA9TCMYBtNgwez2yCQg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIZX6wKP1Bhv5yUspicSV88FUsbSFwJXFH93D1yEibnXrxomfgwibCKMSqUrf9sZCI5anbkVr8bTXV3PQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

想要翻页就要修改这两个地方  

![图片](https://mmbiz.qpic.cn/mmbiz_png/wvdY3149yIZX6wKP1Bhv5yUspicSV88FUv5xEibFibibproO1IrFNtAic4rdT6UuXMic0WeRhibOkCn5sHiaKw3GOd9Z1Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

**END                          感谢大家观看**

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/bL2iaicTYdZn44gvyOMEqwMzru2icHdoQKMOMTRgWx4HDEQU7oW7OJhltxeQr8vsdcXw3cW37nnLa3t26QYqonIGQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)