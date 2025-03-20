> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/WqSldKZ5tmlVRoWadt3_dg)

url: aHR0cDovL21tcHoudHR6aHVpanViYS5jb20vP3I9L2wmY2lkcz0xJnNpdGU9Y2xhc3NpZnkmc29ydD0w

分析过程
----

抓取流量包。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBt56FjuR09oSLFOxyDyg9N4oqjic4vH6uILicn0FNbjMWCGvO74IibROicw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
 主要关注图中框起来这条流量包，因为这条流量包返回的是当前页面数据。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBvVq9iaLibWrgdKUbJib8ibF2m5zW0KicOZA1MBeDA23MlDFlhQET2Jv2WbA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
 该流量包的url地址有个加密的参数`sign`，目的就是找到`sign`参数的加密过程。

按照常规思路会去搜索url中的关键词，但找到对应的地方不能很好的定位到`sign`参数，所以需要换一种思路。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBxMPibTHT7DQhia154TAQAZ0AkjmQ6pgorvrZefVXyovkK1U4QCEoVLPw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

我们直接搜索`sign`关键字，有很多条记录，需要稍微筛选一下。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBTzmHy06V89ibbL1t1AR3vdnc3YdOpcOx2jmqibQzxgXVLtVRnlaauPiaA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

只有两条数据符合我们的要求。先试`sign: n("7bc9")`，打断点。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBfSqdAciaIiaW1fgZ80aR0KFCHJgickohS2TibQONI0j2bbhXehib05a8a2A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
 输出一下`n("7bc9")`。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBRkJZSusCOqQy5VPwb1ib2uEZmYZlIxVFib9ngelgVyeJ1zcZeKRhlzLw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 发现输出是个原生代码，所以此处不是我们要的，可以排除。

  

找到另一条符合要求的记录，打断点。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBUMvzOxjApVa5YSyBtKeibNkfuHzWPJ42AwT7SHKCjLgfd1dcPSCK7ww/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

刷新界面，触发该断点，查看每一个处的值。  
`makeSign`是个函数。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBWlEjG81NEARWvR2KrpXmia9xt94N6K4RclvoSODYtwy5qZ3f3T6Iuow/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
`s`和`this.config.appSecret`都是变量。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBNm2oLP6lNVc2icfWdSaib470MRMdpqvEuwgGVLhpeHiaMCjlfNHCZ7VcQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQB1MibPpp56K4mf00chY8GF1ArVKv2dgBxM6w2sANbId5kyiaiaYYxv2wMw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

先搞清楚两个变量的来源。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQB6HV2sbuxdDGw9q4fickV04oOtQoRQtdJahXlJzjeg6mxzVC63olX4ng/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

```
`// 创建包含版本和 appKey 的对象``const configParams = {` `version: this.config.version,` `appKey: this.config.appKey``};``// 第一次调用 p 函数``const intermediateResult = p({}, configParams);``// 第二次调用 p 函数，合并 r 参数``const s = p(intermediateResult, r);`
```

`s`也涉及到了`this.config`变量，那就先确定`this.config`变量是什么。在该文件中搜索`this.config`，定义处如下。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBItcE1AfLMmzmjTEqlsGO9zfWpmdWjRaGItASw5CkANQT996SwfWHSw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
`this.config`的初始化涉及到传参`e`，看哪里调用的该函数（看call stack找）。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBEwCQTiaqtU6rtZ9SmUJH6mfYpG6KlAp6Uxcx0mSrtw6AjC6zRcCEX2A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
`e`是个固定值，`Object.assign`函数是一个用于复制的函数，故`this.config`也是一个定值，`this.config.appSecret`的值就显而易见了。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBia3SwN6QCBw9qpQGBVMtx4GSRByVkP0GLQZiccCkbNicInpKMvqDVhbMQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
 再回去看`s`变量的定义。`p`函数定义如下。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBp31bh9AG5geaibgbEPScQhbamy8XN3HZMDmriaQHzmNYT2p7a6dL6ysw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
 都是一些原生代码，看不懂也没关系，直接运行一下即可。  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBTgFcbTRX48kDjFY9tFj6svPVmTlXkADPabGicucFo9N72KjGOApWLLA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
 相当于就是给`version`和`appKey`赋值。

`makeSign`

函数的定义如下。

![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBfbAqaQ4rRD6IylKPicSrcEfvAp5lfgiblpL61nAnDGkicaic95nHVEtjug/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

 看到有md5摘要算法了，那么只要搞清楚要加密的数据就可以了。

`n`

的值如下。

  

  

![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBzGS3kEylvZYqUBvCrkkgibSGIIAovGAJprNnnMYxJVUlYtl1muGmfeQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

`e`

就是传过来的

`this.config.appSecret`

，接下来就简单了，将两个值进行拼接即可。

  

  

![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBfsbl0fbv3LTLOLqHOicJKbvv6icp8jFPZnDI6x93tNBaaM3exiavicbkibQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

整个解析步骤结束了。那么可以通过直接编写python代码来爬取数据了。

```
`import requests``import hashlib``def generate_md5_signature(app_key: str, version: str, key: str) -> str:` `""" 生成 MD5 签名 """` `data = f"appKey={app_key}&version={version}&key={key}"` `md_hash = hashlib.md5()` `md_hash.update(data.encode("utf-8"))` `return md_hash.hexdigest()``def fetch_goods_list(page_id: int, app_key: str, version: str, key: str):` `""" 获取商品列表 """` `sign = generate_md5_signature(app_key, version, key)` `url = (` `f"https://openapi.dataoke.com/api/goods/get-goods-list?"` `f"version=v1.2.4&appKey={app_key}&pageId={page_id}&pageSize=100"` `f"&sort=0&cids=1&tmall=1&commissionRateLowerLimit=3"` `f"&hasCoupon=1&keyWords=&sign={sign}"` `)` `headers = {` `"User-Agent": (` `"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 "` `"(KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36"` `)` `}` `response = requests.get(url, headers=headers)` `return response.text``if __name__ == "__main__":` `APP_KEY = "612bcfe884763"` `VERSION = "v1.0.2"` `KEY = "5dd20b08158402032845b45f5b67a349"` `try:` `page_id = int(input("请输入你要查第几页: "))` `result = fetch_goods_list(page_id, APP_KEY, VERSION, KEY)` `print(result)` `except ValueError:` `print("请输入有效的数字页码！")`
```

  

运行结果如下：  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBA1XOsRT46C3vBvibmNRDDQBCmEgEFicicPhjRm7ZUmCGa06Rm6slkYZ40ET06HnwRsAlW8UkibEjiaAIw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBDqj6QnUvCNGYdPKcFaXsJVN34VKtyptJSicszC21gxKSoU1HqicG8QsXrtWxC3TGMFu8jFjqT8rG2A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)