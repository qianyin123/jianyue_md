> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/bAoikyrfDuuUtnOvefmD7Q)

首先从海外平台下载此应用到测试机中然后通过mt管理器提取出他的安装包，发现安装包是apks的格式的

网上搜索了一下发现apks是某海外平台的格式，直接将apks修改后缀名为zip后解压发现如下几个apk子包体内容

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oOl9CYcfyFiaDusBTTl7ibj98BTevT3lhhvV7PgQ5DMR4gD6ibD5QDrDwCsogygv0LmJIuYvJjicFCJw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

其核心代码都放在base.apk中，所以我们只要先分析里面的代码

通过jadx打开后发现其代码已经被混淆了

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oOl9CYcfyFiaDusBTTl7ibj9UNlJUsHrwNuxuHKiaK0ExJlG2dVlLBITCGMqakqkiaVRl0aibDIxCzAGg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

于是通过jadx的反混淆功能后，也于事无补

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oOl9CYcfyFiaDusBTTl7ibj9BRAn13ZXJxibqVcfGfAp0F7Lich9ESH6U86ibudpAMAQh3o9tOXx4ADHA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

那先不管，先通过开发者工具测试 找到app的当前activity，如下图：![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oOl9CYcfyFiaDusBTTl7ibj9uuGzhqmC5SyDMfpgvLic6970VYt9f5FPDDJpG8ItE73EIYaticPd3Ngg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

大概看了一遍后也没啥用，这里知识好奇想看看启动的入口而已

然后继续回到jadx找相应的代码，根据多个小时的代码分析和测试

最终发现如下几个代码文件信息量非常大，估计可能就是播放源的操作

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oOl9CYcfyFiaDusBTTl7ibj9xJuwJpdOfOeXYPlicjeJlj9jazBmCCkfKQk5a9qLpxtw2H3L6GgTJYQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oOl9CYcfyFiaDusBTTl7ibj9wc8JE7clNQibicEnW7DxSEPtODRZX7UngLr5goujNAFCp7sgKL2iccCBg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

最后通过frIDA的hook这几个方法后，发现来回播放视频都会调用TXVodPlayer的startVodPlay(String str)方法，

其中str就是播放源的uri链接地址

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oOl9CYcfyFiaDusBTTl7ibj9a1daTt3rbguhhDianic5S8Tp7IibOBBZy0R4cvPZ5JSWJ90ZiaINv926bw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

返回的时候调用了aVar.a(str)，继续跟踪其方法

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oOl9CYcfyFiaDusBTTl7ibj9wjekB2WB0vBpzYb0YfGnorHPP5mvm27iaheweVlMdt5HntR78Xrskdg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

发现a方法内部有控制其播放的操作，有此可以判定已经找到了下载源和播放视频的调用方法

最后frida去hook此调用的startVodPlay方法

```
`let TXVodPlayer = Java.use("com.tencent.rtmp.TXVodPlayer");``TXVodPlayer["startVodPlay"].overload('java.lang.String').implementation = function (str) {` `console.log('startVodPlay is called' + ', ' + 'str: ' + str);` `let ret = this.startVodPlay(str);` `return ret;``};`
```

结果出现了加密的uri下载源

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oOl9CYcfyFiaDusBTTl7ibj9h9Z4EDDQ9yh38N0FzC3PhfQ0DiaTrA8lryx3HdViaYfiaxqRcEz9YHHLg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

将其下载源放入浏览器后发现没权限等提示，很恼火。  
  
于是灵机一动通过ai写一段python读取下载源的代码并保存为mp4格式

```
`import requests``import os``import re``from urllib.parse import urlparse``from datetime import datetime``def download_video(url, output_dir="downloads"):` `"""` `下载带有签名的视频文件` `参数:` `url (str): 视频URL` `output_dir (str): 保存目录` `"""` `# 创建保存目录` `os.makedirs(output_dir, exist_ok=True)` `# 从URL中提取文件名` `parsed_url = urlparse(url)` `filename = os.path.basename(parsed_url.path)` `# 检查URL中的Expires参数` `match = re.search(r'Expires=(\d+)', url)` `if match:` `expires_timestamp = int(match.group(1))` `expires_date = datetime.utcfromtimestamp(expires_timestamp)` `now = datetime.utcnow()` `# 检查链接是否过期` `if now > expires_date:` `print(f"警告: 链接已于 {expires_date} 过期!")` `return False` `# 设置请求头` `headers = {` `"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",` `"Accept": "video/webm,video/ogg,video/*;q=0.9,application/ogg;q=0.7,audio/*;q=0.6,*/*;q=0.5",` `"Referer": "https://xxxxxx.com/",  # 假设来源网站` `}` `# 文件保存路径` `save_path = os.path.join(output_dir, filename)` `try:` `print(f"开始下载: {filename}")` `# 使用流式下载` `response = requests.get(url, headers=headers, stream=True)` `response.raise_for_status()  # 检查HTTP错误` `total_size = int(response.headers.get('content-length', 0))` `downloaded = 0` `# 写入文件` `with open(save_path, 'wb') as f:` `for chunk in response.iter_content(chunk_size=8192):` `if chunk:  # 过滤掉保持连接的新块` `f.write(chunk)` `downloaded += len(chunk)` `# 显示下载进度` `if total_size > 0:` `percent = (downloaded / total_size) * 100` `print(f"\r下载进度: {percent:.2f}% ({downloaded}/{total_size} bytes)", end='')` `print(f"\n视频已成功保存到: {save_path}")` `print(f"文件大小: {downloaded / (1024 * 1024):.2f} MB")` `# 验证文件完整性` `if total_size > 0 and downloaded != total_size:` `print("警告: 下载文件大小与预期不符，文件可能不完整")` `return False` `return save_path` `except Exception as e:` `print(f"下载失败: {e}")` `return False``if __name__ == "__main__":` `# 您的视频URL` `video_url = "https://xxxxx.xxxx.com/4c1b3983vodhk1310397514/4f3f05db3560136622446212854/f0.mp4?Expires=1752042081&Signature=YrMXtvSb8me66QuH05nMW48rmS6h-~P7xAQ21JlPxFGpS1FGKyFI1gzKz05--dotLXEwhPM4a920SGh2jfCXTdqswVn7KCn-LPz1u30xHX2XnqcRf5HfWkRCwfu4MTLRgZloq4oIwQsdnPGjf86t830kD6pT4KW9wJByfHRc4sFZd~aFidrSiR3gEvWJ5VgtaJ7uJjMrnCM3vwRVC~TzXQOHaZFp67q4qn0o~EKahTBxFEgKI7Vh-uxWvMUOBXhqrKUMp9zWSVF5GwdRjk6kbFBEvIUlhV527RUZzPpcM0cOwmzC8PopzOo4jS~~lxmIg~fNjPAs9sNbyhgMgExlyw__&Key-Pair-Id=K3BZ1QB768QHY7"` `# 下载视频` `result = download_video(video_url)` `if result:` `print("下载完成!")` `else:` `print("下载失败!")`
```

运行后成功下载mp4格式视频

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oOl9CYcfyFiaDusBTTl7ibj97JAtFPCrNhgpfTG0ZAOhiaicTIe5iaZ9QZ6JAnrt86379aNWSfjd4sWmQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

至此完成此app的播放逆向分析。

最后叠甲

【宇宙免责声明】

本文内容仅供学习研究，请勿用于非法途径。

  

**·** **今 日 推 荐** **·**

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHOaTlVncsyCic5iaBibcrQcZR7B7UOQMQxjgCibxvEjFQoQU43ibiaWkpp9og/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

本文内容来自网络，如有侵权请联系删除