> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wzWj3Vs8G_yo655eJ99BHw)

在日常生活中，随着短视频的发展，大家使用抖音进行数据搜索，也占了一大部分，今天给大家带来的文章是，**如何在**抖音上**根据关键词进行视频搜索下载？**

  

抖音根据关键词进行视频下载有什么作用呢？其实很多时候我们制作视频，写脚本，都需要大量的素材与文本参考，在我们没有思路的时候，参考同行是一个很好的方向。

  

这个时候利用关键字下载视频就有着很大的作用，能够帮我们快速的定位到自己想要的素材，拿到数据后我们可以根据自己的需要做一些数据分析，比如文本提取，视频分割之类的。

  

由于现在抖音开放了根据关键词下载的接口，这样的话我们不用像以前那样需要逆向什么的，直接去请求接口即可。具体示例代码如下：

  

```
`import json``import os``import re``import time``import requests``import urllib.parse``nn = 1``sun_s = 0``gjc_name = input("输入关键词")``max_bofangliang = int(input('点赞量'))``key = urllib.parse.quote(gjc_name)``# print(key)``while True:` `url = f'https://www.douyin.com/aweme/v1/web/general/search/single/?device_platform=webapp&aid=6383&channel=channel_pc_web&search_channel=aweme_general&sort_type=0&publish_time=0&keyword={key}&search_source=normal_search&query_correct_type=1&is_filter_search=0&from_group_id=&offset={sun_s * 10}&count=10&search_id=202209151332480101402051633D0E8650&pc_client_type=1&version_code=170400&version_name=17.4.0&cookie_enabled=true&screen_width=2560&screen_height=1080&browser_language=zh-CN&browser_platform=Win32&browser_name=Chrome&browser_version=105.0.0.0&browser_online=true&engine_name=Blink&engine_version=105.0.0.0&os_name=Windows&os_version=10&cpu_core_num=12&device_memory=8&platform=PC&downlink=10&effective_type=4g&round_trip_time=100&webid=7129806389195458082&msToken=20jBGIfrrkKSgtlRmqkkoaFZIj-hQEwWI2LVMn4kASh_Jg_VAJCVGW9q5gwmCLXQnEFn8KdqlEJxrjF7geVghbpbUDCgZS5GJhVjGsTSrXE382FG5H-sKFM=&X-Bogus=DFSzswVLF50ANydASsRgAKXAIQ-S'` `headers = {` `"Cookie": '',  # 登录后输入自己的Cookie` `"referer": "https://www.douyin.com/search/%E5%84%BF%E5%AD%90?aid=165d20aa-17b3-4b63-b831-645b2eb7f064&publish_time=0&sort_type=0&source=normal_search&type=general",` `"User-Agent": "user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36",` `}` `#` `resp = requests.get(url, headers=headers)` `respss = json.loads(resp.text)` `for i in respss['data'][:-1]:` `try:` `url = i['aweme_info']['video']['play_addr']['url_list'][0]` `name = i['aweme_info']['desc']` `aweme_id = i['aweme_info']['aweme_id']` `bofangliang = i['aweme_info']['statistics']['digg_count']` `if bofangliang > max_bofangliang and gjc_name in name:` `if not os.path.exists(f'./{gjc_name}'):  # 如果作者文件夹不存在，就创建` `os.mkdir(f'./{gjc_name}')  # 如果作者文件夹不存在，就创建一个` `video_name = name  # 获取视频名称` `video_name = video_name.replace('\n', ' ')  # 吧\n替换成空格` `video_name = re.sub(r'[\/:*?"<>|]', '-', video_name)  # 替换文件名中的特殊字符` `resp = requests.get(url)` `file_object = open(f'./{gjc_name}/{bofangliang}_{video_name}.mp4', mode='wb')` `file_object.write(resp.content)` `file_object.close()` `print(f'第{nn}个视拼，名称：{bofangliang}_{video_name}')` `nn += 1` `except:` `pass` `time.sleep(5)` `sun_s += 1` `print(sun_s)`
```

  

**右击运行代码，即可实现在抖音根据关键词下载视频**，还不快来试试，代码获取，回复：“**抖音关键词下载**”。  

  

以上就是今天给大家分享的内容，更多精品教程请关注**公众号****SpiderBy**，回复“Python教程”，即可获取*智基础+就业班课程。