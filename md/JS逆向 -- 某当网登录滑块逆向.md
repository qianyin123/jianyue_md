> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/58t4yug3YqR6WNRMLhL4qQ)

本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！  
  
  
目标：实现账号的登录，滑块验证。  
网站：aHR0cHM6Ly9sb2dpbi5kYW5nZGFuZy5jb20vIw==  
  
  
说明 本文主要是对该网站实现登录，逆向其中遇到的滑块。  
  
1. 随便输入手机号和密码，手动滑动滑块实现登录全过程，在控制台可以看到一些主要的请求，从后往前分析，看看每个请求需要的参数，主要四次请求。 

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVL75fIPjArUmOibOPAukRGgN9JH1PNu6oCkiaicSdj4uUwAGs8QGazHiboKg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLv0WaE3wt3l20WJGBuHzo8s50kTAAvFEgtdV0pwdUA5OwucsgDRXS0Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLCibAgHRtKoOFr8JcQBda6Blb1aBu3g3hVwfyMfAWscH25SwUC9pxa3Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

  
  
  
isShowSlide 原本以为是对参数的验证，结果是无需理会  

getSlidingVerifyCode 主要是获得滑块验证的图片  

checkSlidingVerifyCode 是对滑块的校验  

accountLogin 登录接口

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLdQA39wGf8eTAicSyMNDT8no7xPLJezx8HENLkaoeOjOVT3Kz8jgTPlg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

2.登录需要的参数有很多需要分析，主要分析是sign的生成过程，token是getSlidingVerifyCode 返回结果数据，check_token 是验证滑块后checkSlidingVerifyCode 返回结果数据。  

3.首先是先获得滑块的图片，可以看到主要有三个参数。前两个permanent_id requestid  在这四次请求中同时存在，值还是一样的。直接搜索sign或者hook JSON，xhr断点指定接口等  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVL19ZichPlquAGW2og5va3EbXMr16woobMU55fJ91DrggSXrYekicrJ2Pg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVL4klH2TP6DbQrum0kW9bV45Uo80T8OHooEstx9VLVmPHrh8KgFRic8Uw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

4.直接搜索sign 返回不多，主要两个打上断点，再次请求，可以看到变量n 就是一些请求参数，r就是最终sign的值，加密有的结果在赋值到n中，permanent_id 等于e ，e的生成在上面 e = G(), G函数，

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLdN3pkR1ibnLskLWGsoxJkgLPN8BL97qARRuLr3nCu6D2Yh3CQan6RJg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLia3nnNr9nhd6qxlakc7l7XBJDibNVEHxMWZfpBwDlpw3paNUFxFYaC2A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

  
  
  
5. D.state.requestId 返回requestId  刚开始我以为这个值是js生成的，在往上跟栈找着找着发现是接口返回的数据，首先重新刷新网站，再次断到当前位置，直接打印requestid的结果，直接搜索。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLibfUtE3Z7Ue9fteh9BHdKGLzh2zG1gxqxib8PxTicjDII8STbV4YtM50Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVL4APFSNn4iaYKzRHcsjzha4r27zSv2vaJBz2ucTWtqEAmqlzp5niaNEPA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVL9ibiawTFNoGwt7pXxAosz5YbYwpUbBywKeKxaFqAqFwQ9tHEc68BrOvQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

6.requestId 直接是一个接口返回值，请求参数也是先获得permanent_id 和sign， 所以先分析，函数G里面生成规律，在看sign。直接控制台打印G，跳转到函数里面。  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLIyvh9gRNSbJUyRUBKrZziaSImshWZW73bafjZD75wPFsV2qPibnSZQNQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11)

  

  
  
7.大致分析一下这个函数，先是定义一个t变量，在获得当前时间时分秒等，随机生成两个参数，一大堆拼接成d，然后函数J对d转换，明显是一个md5，测试一下是一个标准的md5。后面有定义一个h函数对p转换。  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLtqZ4R4monaIq6ELafa2FBa2YnFgnJasqT6ZLm1jLicJ5IGp9RqGdfQw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVL7KudhibasTqcbyNzzpbpiaPCzAiasspuXBwE7oiad35M7dHqsmIYssBNrA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLAn1MP0uEOsxpUFJ8RAvEwI0A2icNiaUFqzP3auZhYN9f3YLQBsuUnxyg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14)

  

  
  
8.可以直接把G扣下来的改一下J函数，也可以直接用python实现还原过程，自己写或者扔给ai一键转换也可以，一下是py代码。  
  

```
`def random_string():` `# 基础字符串` `base_string = "DDClick521"` `# 获取当前时间` `now = datetime.datetime.now()` `year = str(now.year)` `month = str(now.month).zfill(2)  # 确保月份是两位数` `day = str(now.day).zfill(2)` `hour = str(now.hour).zfill(2)` `minute = str(now.minute).zfill(2)` `second = str(now.second).zfill(2)` `millisecond = str(now.microsecond)[:3].zfill(3)  # 取毫秒的前三位` `# 生成随机数` `random_num1 = str(random.randint(100000, 999999))` `random_num2 = str(random.randint(100000, 999999))` `# 组合字符串` `combined_string = year + month + day + hour + minute + second + millisecond + random_num1 + random_num2 + base_string` `# 创建 MD5 哈希` `md5_hash = hashlib.md5(combined_string.encode()).hexdigest()` `# 处理哈希前八位` `def process_hash(hash_str):` `hex_value = int(hash_str[:8], 16)` `str_value = str(hex_value)[:6]` `if len(str_value) < 6:` `str_value += '0' * (6 - len(str_value))` `return str_value` `processed_hash = process_hash(md5_hash)` `# 最终组合字符串` `final_string = year + month + day + hour + minute + second + millisecond + processed_hash + random_num1 + random_num2` `return final_string`
```

9.接下来主要看一下sign的生成过程，在getRankey 请求断住，看一下请求加密参数需要哪些，主要是一个permanent_id，N.a.stringify(a);   把a 转换成url查询字符串，在decodeURIComponent编码，在使用函数J转换，这个J似曾相识，就是和之前的一样，也是md5转换，主要看一下函数Y的功能，传入两个参数，第一个参数是明文md5值，第二个参数是密钥，因为getRankey接口请求前，rankey和requestId 都还为空，所以第一次生成sign是密钥为空。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLmXMXLpgJt7T79PbQbibhCb4CuuicT4FnyuB0v2cQaWy1EE3o4emGfs8g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=15)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLRLfYQBhuwXUmMeJBtdP9pFTicUqtMwt3CPkgkqIdC2ALKqiblkIhRdHA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=16)

10.进入到函数Y中可以看着这是一个aes加密，测试过了也是一个标准的加密，具体加密还原使用js还是python当是看你的心情了

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLvaaEKYr0KZibrJVpIiaTlECFTtj9upLc8SYEXn1pytpxrMaTcrvsdxUQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=17)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLiajHfIwOktYplMywCBLfHyX5icow1iamurYuw3UGvc5MNtQFZWfuBcolA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=18)

11.  哦对，还有一个t参数，是时间戳，这样就可以拿到rankey和requestId，需要注意的是，在请求getSlidingVerifyCode 接口时，注意一下参数，密钥就是rankey。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLmLw5AWKGy2fc8bZ9WnjOyWCB4OEDvwSIOkr0kT5oVic1aK57hn5rwjg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=19)

12.获得图片后，就是要是别滑块的距离了，我使用的是ddddocr验证滑块的参数有点多。  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVL1sD8EqgClsQOib6uel4Msj6sqOdCr4lfBhbyXAeMpJ7QGoe22BFJdCQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=20)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVL3OtdGXsQbfDsgelHycKxGEWegz4XiaIM2t6icOibzZWbt7Z8sJW3unlxw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=21)

point_json 是加密过的  
slide_cost_time 滑动时间  
verifyToken 验证token，获得图片时一起返回值  
其他没有什么特殊参数了  
  
13，现在主要看一下point_json  怎么生成的，直接搜索打下断点，还是aes加密，其中不同的是key值，t.left就是滑动的距离，t.json.y 是请求图片时返回的y，t.json.encryptKey是返回的encryptKey

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLag5pZye4n4ej8HiaTrNs0seJBibsSy8QpOkL0xnzQXSO4bKxibMf7d0ZQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=22)

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVL7reFb40Lx0rgBfUpztfgciagyPW2ScdiacYKhoRUCw2WElULfduCYcNw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=23)

4.滑块验证成功后会返回 checkcode，至此登录接口还有一个password的生成规则，直接搜也能搜到就是有一点多，可以直接使用xhr断点accountLogin，找到password生成地方。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVL3K1gicWPj9s6mibgJhruQIo5ALDYd8KKHuu4eEDCVsB0oN3c8Suict3icw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=24)

15. 同样也是Y函数，传入你输入的密码，key的值是固定的。到此所有参数都分析完了，使用代码测试测试。  
  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLTrXT1EcmuWV6DsL8ttE6F0yHX6IW6FSgoEWXBdM5OOPlVmoXUAoibaw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=25)

  

**·** **今 日 推 荐** **·**

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0rkjtS9ngP4EPbXbbzAQaQYjjzMsjqZprnu0jicBY6X5ia0BRCSaN8tEmFDv0libdv9XGCwO1VHDF4Zw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=26)

  

本文内容来自网络，如有侵权请联系删除

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEh0DpROskzTgmeK6XspVLy1skUJibe2a2cI1502uBYzcs5JrhnxWXSE5JcA7GLupNF4X9EmDvfVQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=27)