> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rC7_bGWeMq1RUtY0GSlkPQ)

##### 提示！本文章仅供学习交流，严禁用于非法用途，文章如有不当可联系本人删除！

#### 一、案例分析

①如图弹幕信息，弹幕信息是protobuf序列化后的数据  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp8tKyicOYbASwwp9gwFApSLlpC13JIEBfkicpnvShNhZKqwXJJ4IibtdPhu1B6dnffNa8onFWp98x4aA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp8tKyicOYbASwwp9gwFApSLlwEILphNJM5RPMqx2669GfWOdOGkIRk5ibibLSQBeYFjTgibicgicyOmBxNw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

② 响应头也有明显特征：`content-type: application/protobuffer`  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp8tKyicOYbASwwp9gwFApSLlFbSOhXYku3S3TqnDF0F4wuEp1bKzpCQicJvGI7DrBNOLKhPjGUnCsFA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

③ 对于protobuf序列化的数据解决方式有两种：

④ 方法一：选择`blackboxprotobuf.protobuf_to_json(content)`，直接获得还原的数据以及消息类型，除非解析失败或异常，才选择方法二的解决方法

⑤ 方法二：分析字节内容.bin文件，然后`还原出.proto文件`，然后编译成python包，然后再解析，这个过程比较复杂，不是很推荐，自行根据网页情况选择  

#### 二、案例解析

① 关于protobuf案例的更详细介绍看[js逆向案例-某wanfang_protobuf](http://mp.weixin.qq.com/s?__biz=MzU5NTcyMDc1Ng==&mid=2247484959&idx=1&sn=a59ac94839579d76f137f65f9d39d46d&chksm=fe6ce944c91b6052d8b72115c1079554612593924eb6fa1c637a9e26136fb61d50836e9ad37c&scene=21#wechat_redirect)

② 这里直接安装`pip install blackboxprotobuf`，先用fiddler抓包，然后选择响应当中的十六进制字节黑色字体右击存为.bin文件，然后按如下脚本解析，测试了多个文件都可以正常解析  
![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp8tKyicOYbASwwp9gwFApSLlPV0QRMQCSfXtmea0dz4pyaf6WOZG9ibWhVxFxrHiamISI867GAuicoIqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZPnQAHJ6rp8tKyicOYbASwwp9gwFApSLlRKN4tibCGFpLPphZFhW9mY926UcxoNU7ibvwib3yLic8BauH6YNaj8ribqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

③ 如果是响应请求的话，就按如下脚本结构格式解析即可

```
import requests  
import blackboxprotobuf  
response = requests.get("填写url", headers={}, timeout=10)  
deserialize_data, message_type = blackboxprotobuf.protobuf_to_json(response.content)  
print(f"原始数据: {deserialize_data}")  
print(f"消息类型: {message_type}") 
```

④ 如果觉得文章还不错的话，记得关注支持一下哦~