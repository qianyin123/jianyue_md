> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/CZxpFjta7ihLsLyXKYgsWg)

````

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBD5Ar6P3bw9e5olgZcvuSS88QYqVSbP5Dqdg0Geys37LyqTXTS7uOv0BYwIm7UFyRLqAv9yfHhl7g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过搜索关键字 `sign:`，定位到如下所示的代码位置。在该位置设置断点后，刷新页面可以成功拦截请求，由此确认了代码的具体位置。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBD5Ar6P3bw9e5olgZcvuSS8K0WibuIkIWsgBcyUiaU3rh2drjM2VibOLHSuPds54lgWsS0vGkq1FRW9Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

分析实现算法

代码中代码可简化成如下伪码：

```
`// p 是一个函数，传入参数 e，通过调用 function(e){} 对输入的字符串进行处理``// 其中 function(e){} 是具体的加密算法，text 是待加密的字符串``// 最终输出结果是一个 32 位的十六进制密文``// 将代码拆分成两部分分别进行分析``_hadoken = function(e){} // 定义加密算法的具体实现``text = o.token + "&" + a + "&" + s + "&" + n.data // 拼接字符串作为待加密的输入`
```

函数分析

根据加密后字符串的特性，推测该算法为 MD5。将函数赋值给 `_hadoken` 变量后，对测试字符 `"1"` 进行加密，发现其加密结果与标准 MD5 算法的输出一致，因此可以确认该函数实现的是标准的 MD5 算法。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBD5Ar6P3bw9e5olgZcvuSS8Eds6RZGpCqnvZ1Nlw2I3WZPEsg2un1b18M90NsicjXRyMyneBqicjx2Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

参数分析

接下来对参数 

```
o.token + "&" + a + "&" + s + "&" + n.data
```

 的值进行分析，其实际值如下图所示：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBD5Ar6P3bw9e5olgZcvuSS8oV91Vo8jJe7VsbC4GIp0g66MW3OGTrbF4vXmOSIlnfXGZsSH70KiamQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

观察得知：

o.token为cookie里面的_m_h5_tk参数，访问网站后会自动获得，且值短期内不变。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBD5Ar6P3bw9e5olgZcvuSS8wwCPyoo5ShCapHumZMiaX9VESfL8DpdHK54oeG9ErICWeQeDQKsMzZA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

*   `a` 是时间戳，可以直接生成。
    
*   `s` 是 `appKey`，用于标识应用 ID，固定值为 `12574478`。
    
*   `n.data` 的值与查询表单的数据一致，可以直接构造。
    

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBD5Ar6P3bw9e5olgZcvuSS8tvRIHwibfIewMwbsia6Lrd981jZACiafRbFqUZicpmatDIysCCtib3BBfCA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

至此， 模拟算法已分析完成，可用python实现代码如下：

```
`import requests``import hashlib``import time``# 请求头``HEADERS = {` `"authority": "h5api.m.taobao.com",` `"accept": "application/json",` `"accept-language": "zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7",` `"cache-control": "no-cache",` `"content-type": "application/x-www-form-urlencoded",` `"origin": "https://zc-paimai.taobao.com",` `"pragma": "no-cache",` `"referer": "https://zc-paimai.taobao.com/",` `"sec-ch-ua": "\"Not_A Brand\";v=\"99\", \"Google Chrome\";v=\"109\", \"Chromium\";v=\"109\"",` `"sec-ch-ua-mobile": "?0",` `"sec-ch-ua-platform": "\"macOS\"",` `"sec-fetch-dest": "empty",` `"sec-fetch-mode": "cors",` `"sec-fetch-site": "same-site",` `"user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"``}``# Cookies``COOKIES = {` `"cna": "WHVfHEC+s0gCAXFo75AtW31b",` `"cookie2": "2684da5df27f5ac5f9d7f3332dad27af",` `"t": "e643beab1e45cf6e24d842d88d81c2d1",` `"_tb_token_": "f08437ebe3eae",` `"xlly_s": "1",` `"_m_h5_tk": "44faf6e4294578f14c7f8662a602ec23_1679041878217",` `"_m_h5_tk_enc": "38b523e9efd3584ecdfa56c775e020f6",` `"isg": "BPb2ETm6jlJDr319Fxfsn6IkRyr4FzpRCPe4eGDH_Fl0o5M9yafSYaUSu3_PCzJp",` `"l": "fBTFDzPcTWQP8CT1BO5Zourza77O3EAfCsPzaNbMiIEGa6a5GFwugNCsnHK6PdtjgTfxBeKy58T6YdIow34NixDDBe45AzmfSc99-bpU-L5..",` `"tfstk": "cKt1Bw1oeIKEaXm0j1ME3pR4JTIdCze5hV1MCffI3djEH-nfTo1cQu57W7T0MiWAd"``}``# 请求 URL``REQUEST_URL = "https://h5api.m.taobao.com/h5/mtop.taobao.datafront.invoke.auctionwalle/1.0/"``# 请求参数``PARAMS = {` `"jsv": "2.6.1",` `"appKey": "12574478",` `"t": "",  # 时间戳` `"sign": "",  # 签名` `"bxPageId": "1910955",` `"api": "mtop.taobao.datafront.invoke.auctionwalle",` `"v": "1.0",` `"type": "originaljson",` `"dataType": "json",` `"requiredParams": "dfApiName,dfUniqueId"``}``# 请求数据``REQUEST_DATA = {` `"data": "{\"dfApp\":\"auctionwalle\",\"dfApiName\":\"auctionwalle.datou.getPageModulesData\",\"dfVariables\":\"{\\\"pageId\\\":1910955,\\\"moduleIds\\\":\\\"4394607430,4489817680,4480874310,2004318340,4529967930,2708524060,global\\\",\\\"context\\\":{\\\"keyword\\\":\\\"二手房\\\",\\\"page\\\":\\\"1\\\",\\\"userInfo\\\":\\\"{}\\\",\\\"sceneCode\\\":\\\"20210823QCG72BUD\\\",\\\"firstScreen\\\":\\\"true\\\",\\\"device\\\":\\\"pc\\\"}}\",\"dfUniqueId\":\"1910955.4394607430,4489817680,4480874310,2004318340,4529967930,2708524060,global\",\"dfVariablesRecover\":\"{}\"}"``}``def generate_timestamp():` `"""` `生成当前时间戳并更新请求参数` `"""` `current_time = int(time.time() * 1000)  # 毫秒级时间戳` `PARAMS['t'] = str(current_time)``def generate_sign():` `"""` `根据请求参数生成签名` `签名规则：token + "&" + timestamp + "&" + appKey + "&" + data` ``token：从 cookies 的 `_m_h5_tk` 中获取`` `timestamp：当前时间戳` `appKey：固定值 12574478` `data：请求体数据` `"""` `generate_timestamp()` `token = COOKIES['_m_h5_tk'].split('_')[0]  # 提取 token` `raw_sign = f"{token}&{PARAMS['t']}&12574478&{REQUEST_DATA['data']}"  # 拼接字符串` `return hashlib.md5(raw_sign.encode(encoding='utf-8')).hexdigest()  # 生成 MD5 签名``def send_request():` `"""` `发送 POST 请求并返回响应结果` `"""` `response = requests.post(` `REQUEST_URL,` `headers=HEADERS,` `cookies=COOKIES,` `params=PARAMS,` `data=REQUEST_DATA` `)` `return response.text``if __name__ == '__main__':` `# 生成签名并更新请求参数` `PARAMS['sign'] = generate_sign()` `print("请求参数：", PARAMS)` `# 发送请求并打印响应数据` `response_data = send_request()` `print("响应数据：", response_data)`
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBD5Ar6P3bw9e5olgZcvuSS8pSOqJAxq2jxU3SeIPwTmljxaHajyICTIv5LdJsYhVYpBYHEItiaQ7Ug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
a为时间戳，可以直接生成。  
s为appKey，应该是标识应用id，固定不变为12574478。  
n.data的值跟查询表单相同，可以直接构造。
```

```
定位代码位置  
搜索sign:关键字，定位到下图位置，设置断点后，刷新页面可以正常拦截，可以确定代码位置。  
  
  
  
分析实现算法  
代码中代码可简化成如下伪码：
```

````

```
定位代码位置  
搜索sign:关键字，定位到下图位置，设置断点后，刷新页面可以正常拦截，可以确定代码位置。  
  
  
  
分析实现算法  
代码中代码可简化成如下伪码：
```