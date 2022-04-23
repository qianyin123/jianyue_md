> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GttIFJv4yCKmTDWtnBllUg)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A4g05V4rHL4vZMyGTE8ic691mOejIJzAEUvdWQ5sxxrH4ASUa54ugFpMJ985UJn2vkiaCpSD4LFSr8Q/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

     时隔多日，我们又见面了。从今天开始，将会有一个新的逆向章节《App逆向》陪伴我们，希望大家能够喜欢。文章若有错误的地方，欢迎各位读者多多交流指正，希望通过分享这些文章能让大家一起成长！☀️

  

**特别声明**：本公众号文章只作为学术研究，不用于其它用途。

  

  

  

 目录

  

  

①    环境准备

②    参数分析

③    FridaHook

④    算法还原

⑤    总结分享  

  

  

一、环境准备

  

本次分享内容需要我们具备以下环境和依赖：

  

*   Charles
    
*   Jadx
    
*   Frida
    

  

**说明：**因为我们的重点不在环境如何安装这块，所以大家可自行百度安装即可；如果某些环境无法安装，也可联系我帮忙解决。(附上frida安装流程)

  

* * *

  

**Frida安装：**

*   安装Frida模块，命令为：pip install frida
    
*   安装frida-tools模块，命令为pip install frida-tools
    
*   Android版本去下载相应frida-server，然后去git地址https://github.com/frida/frida/releases下载相应的版本，我这里使用的是pixel手机，选择frida-server-15.1.17-android-arm64.xz。
    
*   解压后将文件传输到手机中的/data/local/tmp目录下，设置可执行权限并运行。
    
*   输入`frida-ps -U`命令后输出手机进程，说明frida安装成功。如下图所示：
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7ZxFbIgWOJ4uM55WwXyxYNcboBnCYyqMvestbRicmzXADibbCpPQwO7j6g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

* * *

  

  

二、参数分析

  

**1、使用charles进行app抓包，配置好本地证书，打开指定App，抓包截图如下所示：**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7ZTgGoIO0HQxoUvboGYBpzOZLC4wWibESnYBNmEUt5ichYq4ud4MG5y2jA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

**2、加密参数初判断**

观察上图，我们本次需要还原的参数为：sign；通过查看sign参数的长度位数，初步怀疑为md5加密！此刻我们先标记一下这个判断。  

  

  

**3、jadx确定加密参数位置**

1）使用jadx打开指定apk，截图如下(电脑内存要足够大，不然会卡死)：

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7Z65aOOPJkxOPGXyYgbLnlicPtsgFkMmNbTJF3Gq87PPgzfWvUWc5ybjw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

2）等待jadx反编译完毕后，进行指定关键字搜寻，截图如下：  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7ZaUVbfRg4r1o1Pe4rSPdEzf1SEPu8nY6zoSrSEVWQL4NCFejibG3UgLg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

3）查看java代码，找到加密参数的函数，如下：

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7Zkh7TEYy81wXKHhZiajvv6Jr81VC25pKSKNzsNbz9faYHgQ3aicSibtG9w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

4）追踪到最后加密参数的函数，截图如下：  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7Z9KWyCcXHrbeJOAydolK3c7icS1Uu4GGSRrRExmthkFFqeicoXmgl4giaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

**总结：**初步确定入参及出参的函数后，接下来进行frida hook调试吧！  

  

* * *

  

  

三、FridaHook

  

*   编写frida脚本js代码如下：
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7ZEGktcelmaicrZjd7A5v2iaqyFMQQNpefh1YvOHoTgD7iachqmIHO1AGhA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**说明：**我们只需要在终端关注下入参str、出参ret的value值即可。  

  

*   启动frida-server代码，截图如下：
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7ZXp2LS1rggz8m6flyvPkTibcxEvEYD5unu6IzYksQGV4MZkxlp1iabicpQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

*   启动frida脚本，截图如下：  
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7Zvuj6oZcic07r2bibSS2s086fFI7iaG6xthiaDgvnqnIHnD33dTglt3PWpw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

*   手机端刷新app页面，截图如下：
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7ZztKaicBSzZpCQ0vzpQnhtjNMek8suOoKhgK8SvLC4e5uXFy9PEBk8jQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

*   将参数str复制到md5在线工具中，进行加密截图如下：
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7ZZcI2kOz26wxC8KDLhaUgTjpiadlF5ibDJNFiaiaicdSRVibP7NAiavXiaJN97g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

**总结：**观察步骤4和5中ret变量及在线还原后的值，我们发现结果一致；既然加密手段是md5，出参的加密方式我们已经知道！接下来需要解决入参问题，通过阅读java源码，我们需要对加密前的参数进行逐一模拟和排序拼接，接下来让我们进行算法还原吧！

  

* * *

  

  

四、算法还原

  

*   nonce参数还原  
    

```
`# 一个长度为8位的int随机数``def random_randint(self, i):` `nonce = ''.join(random.sample(string.digits, i))` `return nonce`
```

*   accessSecret、timestamp参数还原
    

```
`# 通过阅读java代码，该值为一个常量，属于salt``accessSecret = "3refdgdEDCRTGFDSFASDFASoiwqeurcvVASDCZX456yhs"``# 13位时间戳``timestamp = str(int(time.time() * 1000))`
```

  

*   所有参数排序后算法还原  
    

```
`def get_param_str(self, timestamp, params, nonce):` `_params = list(params)[:-1]` `_params.sort()` `_params.append(('nonce', nonce))` `_params.append(('pcursor', '0'))` `_params.append(('timestamp', timestamp))` `_params.append(('accessSecret', '3refdgdEDCRTGFDSFASDFASoiwqeurcvVASDCZX456yhs'))` `params = urlencode(_params)` `return params`
```

  

*   Python还原后完整代码  
    

```
`import random``import string``import requests``import hashlib``import time``from urllib.parse import urlencode``class AppSpider(object):` `def __init__(self):` `self.url = "https://xxxxxx/rest/n/kmovie/app/template/photo/getTemplateInfoList"` `def get_md5(self, str1):` `md5 = hashlib.md5()` `md5.update(str1.encode())` `sign = md5.hexdigest().upper()` `return sign` `def get_headers(self, timestamp, params_str, nonce):` `headers = {` `'Cache-Control': 'no-cache',` `'mod': 'Pixel 2',` `'memory': '3.5714128',` `'os': 'android',` `'ip': '',` `'language': 'zh-hans-cn',` `'version': '5.54.0.554005',` `'installAge': '0',` `'cpuBoard': 'msm8998',` `'channelName': 'MYAPP',` `'region': 'CN',` `'networkType': 'wifi',` `'kprojectVersion': '310',` `'timestamp': timestamp,` `'nonce': nonce,` `'Host': 'api.xxxxx.com',` `'User-Agent': 'okhttp/3.14.7',` `"sign": self.get_md5(params_str),` `}` `return headers` `def get_params(self):` `params = (` `('classId', '0'),` `('limit', '20'),` `('kprojectVersion', '310'),` `('flow', 'outflow'),` `('pcursor', '0'),` `)` `return params` `def random_randint(self, i):` `nonce = ''.join(random.sample(string.digits, i))` `return nonce` `def start_requests(self):` `timestamp = str(int(time.time() * 1000))` `params = self.get_params()` `nonce = self.random_randint(8)` `params_str = self.get_param_str(timestamp, params, nonce)` `headers = self.get_headers(timestamp, params_str, nonce)` `try:` `response = requests.get(self.url, headers=headers, params=params)` `print(response.text)` `except Exception as e:` `print(e)` `def get_param_str(self, timestamp, params, nonce):` `_params = list(params)[:-1]` `_params.sort()` `_params.append(('nonce', nonce))` `_params.append(('pcursor', '0'))` `_params.append(('timestamp', timestamp))` `_params.append(('accessSecret', '3refdgdEDCRTGFDSFASDFASoiwqeurcvVASDCZX456yhs'))` `params = urlencode(_params)` `return params``if __name__ == '__main__':` `spider = AppSpider()` `spider.start_requests()`
```

  

*   结果输出如下：
    

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A4icrtCkic1hoKZ4Ubt0cbY7Zu3hhNWibOWt7vwMJjq6vcrTicQSSicLGREUfYic7ZYb1g2E3NnNQsia5AOg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

* * *

  

  

五、总结分享  

  

回顾整个分析流程，本次难点主要概括为以下几点：

  

*   如何快速定位加密参数的位置
    
*   如何通过frida代码去写hook代码
    
*   有阅读Java源代码的能力  
    
*   能够通过Python还原加密算法
    

  

本篇分享到这里就结束了，欢迎大家关注下一期，我们不见不散！！！☀️

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7revypRO1iacSSjh6m3iaeZ7k7QiaRDzFktiaSbkClw0pXa6NV1Q9ge9a6D5nxGOojicqVQUQqQK0NOHg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

_**END**_

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7revypRO1iacSSjh6m3iaeZ7zpB5TQPCHMeJVyX5BicWRibtHzfCIJvrJRAiaLC9akyJxXrfKVMnUS6rw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A4g05V4rHL4vZMyGTE8ic691Wt6FFglTFeeibsPZT5F1vAiafn06J37WwvPkkGVX2B14Qh3gpPmic5Dpw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

    我是**TheWeiJun**，有着执着的追求，信奉终身成长，不定义自己，热爱技术但不拘泥于技术，爱好分享，喜欢读书和乐于结交朋友，欢迎加我微信与我交朋友。

    分享日常学习中关于爬虫、逆向和分析的一些思路，文中若有错误的地方，欢迎大家多多交流指正☀️

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/m5qEELWt8A4g05V4rHL4vZMyGTE8ic691PicricStHwRzqmIO1cPGTPsCk5SmfU2AZQLL2B6KSpxaHGguqZjXnjiaw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)