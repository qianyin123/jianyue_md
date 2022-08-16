> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/8CwMnMmQGIOfbg6rGRS0ag)

  

     大家好，我是TheWeiJun。时间过得飞快，距离上一篇B站protobuf协议还原文章已经过去了一个多月。今天我们继续分享protobuf协议还原分析，教你如何在无结构的数据中抽取并定义proto文件。全程高能，在阅读的同时不要忘记点赞+关注哦⛽️

![图片](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A6N3l9obtATicYg0vSreUhmEsiciaibJfHjG8MEHndnoJycRKpgeeK3LkxKu6qlz7oyC2Gelexa4W4Bfw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

  

立即加星标

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A6N3l9obtATicYg0vSreUhmEG7mwRfEUOichNPSVJkJl4IDcjeqvJyO5GGxL9CaYcRQqg13wjyXVzicQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

每天看好文

  

 目录

  

  

一、什么是protobuf？

二、protobuf堆栈输出

三、proto文件定义及编译

四、完整代码实现

五、心得分享及总结

  

* * *

  

趣味模块

  

      小红是一名爬虫开发工程师，自从上次小红解决了字体反爬、websocket协议、B站protobuf协议后，小红一直所向披靡，过五关斩六将，在一个多月的时间里一直没有遇到过有难度的问题。但是今天，小红遇到了某音App端的protobuf协议加密，不再是js端一样去找定义的类型id了。那么今天我们去分析下小红同学遇到的新问题吧，如何通过无结构定义proto文件还原某音的protobuf协议！

  

* * *

  

一、什么是protobuf协议？

  

**前言：**Protobuf (Protocol Buffers) 是谷歌开发的一款无关平台，无关语言，可扩展，轻量级高效的序列化结构的数据格式，用于将自定义数据结构序列化成字节流，和将字节流反序列化为数据结构。所以很适合做数据存储和为不同语言，不同应用之间互相通信的数据交换格式，只要实现相同的协议格式，即后缀为proto文件被编译成不同的语言版本，加入各自的项目中，这样不同的语言可以解析其它语言通过Protobuf序列化的数据。目前官方提供c++，java，go等语言支持。

  

* * *

  

二、protobuf堆栈输出  

  

1、首先从charles应用中保存我们抓到的某音数据包，截图如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A74kWpDsIE054ISNQMAO924sibibpHOIIu4UlhXKSWSG7FzrOD1dIARLDJPPkXOLEP04LvdkxYhgfoQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

2、访问协议接口，将response响应内容保存为.bin格式文件，截图如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A74kWpDsIE054ISNQMAO9245bEx7T7W8aLHKTtcjZwejFbjVhJYqMDZXzbqV6ClEReCZgsn63pMvw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

3、使用**protobuf_inspector** python第三方包打印protobuf文件堆栈信息，截图如下所示：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A74kWpDsIE054ISNQMAO924dj9DuFm7mFIicWPsnv4jKllBWP0ibkq56qCPZt4aeCJu0MSdsZYnyMuA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**总结：**打印protobuf数据堆栈信息后，接下来我们只需要根据打印的堆栈信息定义proto id类型文件并编译为python可执行文件即可完成对某音protobuf协议还原。

  

* * *

  

三、proto文件定义及编译

  

1、通过分析打印的堆栈信息，确定我们本次要提取的文字内容，截图如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A74kWpDsIE054ISNQMAO924raibLPp7TZtI7dQEw3NEOcdtzbQQDXXNibwFD5cFWibd07eaYmicQOGPEA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

2、确定我们要提取的文字内容后，定义proto文件格式，定义后的结构体如下所示：

```
`syntax = 'proto3';``message FeedData {` `message Message{` `string url = 2;` `}` `message Info{` `string           user_id   = 1;` `string           user_name = 3;` `repeated Message data      = 7;` `}` `message FeedView{` `string        item_id = 1;` `repeated Info info    = 4;` `}` `repeated FeedView message = 5;``}`
```

  

3、执行如下命令，编译为python protobuf可执行文件：

```
protoc  --python_out=. *.proto
```

  

4、运行命令后，生成python protobuf py文件，截图如下所示：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A74kWpDsIE054ISNQMAO924YZicp19jMgHa62YR6kC8SJTy3Ko02HenmjuKHZ2Q5hpH6Zpiav9XP5Mg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

**总结：**走到这里protobuf协议就完全还原了，接下来让我们进入完整代码实现环节吧。

* * *

  

四、完整代码实现

  

1、整个项目完整代码实现如下：

```
`from dyFeed_pb2 import FeedData``from google.protobuf.json_format import MessageToDict``def main():` `with open('feed2.bin', 'rb') as f:` `_info = FeedData()` `_info.ParseFromString(f.read())` `_data = MessageToDict(_info, preserving_proto_field_name=True)` `items = _data.get("message", [])` `for item in items:` `item_id = item.get("item_id")` `info = item.get("info")[0]` `user_id = info.get("user_id")` `user_name = info.get("user_name")` `icon = info.get("data")[0].get("url")` `print(item_id, user_id, user_name, icon)``if __name__ == '__main__':` `main()`
```

  

2、代码编写完成后，我们运行代码，代码输出截图如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A74kWpDsIE054ISNQMAO924e7gk6pFJriaoeo2u3gwI9RqPOvjn6EuESsXR8KbzqpNKr6cQDNcicGxA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

* * *

  

五、心得总结分享

  

      通过本次案例分享，我发现有些技术不是难，而是没有找到好的方案去解决；有的问题需要思考3个小时甚至更久，但是解决只需要10分钟。今天分享到这里就结束了，欢迎大家关注下期文章，我们不见不散⛽️

  

* * *

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A43BLL5j8ShhkUSRLT9ayEsmde5yxQmlKq8qZgsltoYaFpliaXKlmJUtNr6uZHibrn2xgIsQJUXu2nA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

关注我们获得更多精彩内容

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7revypRO1iacSSjh6m3iaeZ7k7QiaRDzFktiaSbkClw0pXa6NV1Q9ge9a6D5nxGOojicqVQUQqQK0NOHg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

_**END**_

![图片](https://mmbiz.qpic.cn/mmbiz_png/m5qEELWt8A7revypRO1iacSSjh6m3iaeZ7zpB5TQPCHMeJVyX5BicWRibtHzfCIJvrJRAiaLC9akyJxXrfKVMnUS6rw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/m5qEELWt8A4g05V4rHL4vZMyGTE8ic691Wt6FFglTFeeibsPZT5F1vAiafn06J37WwvPkkGVX2B14Qh3gpPmic5Dpw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

    我是**TheWeiJun**，有着执着的追求，信奉终身成长，不定义自己，热爱技术但不拘泥于技术，爱好分享，喜欢读书和乐于结交朋友，欢迎加我微信与我交朋友。

    分享日常学习中关于爬虫、逆向和分析的一些思路，文中若有错误的地方，欢迎大家多多交流指正☀️

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/m5qEELWt8A4g05V4rHL4vZMyGTE8ic691PicricStHwRzqmIO1cPGTPsCk5SmfU2AZQLL2B6KSpxaHGguqZjXnjiaw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**点分享**

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**点收藏**

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**点点赞**

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**点在看**