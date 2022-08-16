> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/auRSwS6QVnna9DiS2nqnRA)

  

郑重声明：本文只为大家学习交流，如若侵犯贵司权益请联系本人删除，严禁用于商业用途，产生严重后果与纠纷与本人无关

  

测试链接   aHR0cHM6Ly9kdW4uMTYzLmNvbS90cmlhbC9qaWdzYXc=

  

某盾流程

  

目前文件版本  案例版本 滑块拼图，  watchman 2.7.5_602a5ad7 ， core.v2.20.0.min.js

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlC0DJIt2C7qQhwsgb4cvRH6peG2H0hrDpTiaam12bvfvXx5jRHbjATzfw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

api/v3/check 请求

  

check请求验证是否通过，参数主要有data,cb

cb 随机即可

data 参数和轨迹相关，涉及到轨迹加密， js流程在core.js 中

请求结果能拿到validate 即可

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlCr0CW0rtlkRXhVicSbvdib3Y3RLBEAAMqy6LdicRv2QicXJN9MG8o44dEuQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

在 core.v2.20.0.min.js 中搜索 'data'  几处 打断点，滑动滑块，如图即可找到生成位置

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlCJJRYhQ3k1Sia7s12xADhqiaCFyT4U8vPaU9iabVjgCBBbc4Jeicg8uAG9g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

_0x3d43cb  是滑动点位加密的结果，最多取50位

_0x9a0013   是get请求返回的token

_0x11fd60   是token 和点位 加密生成的结果，扣js 即可

  

api/v3/get 请求

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlCtk1aOyyv2VCR1tXpnCpKbvXypkUZNFVQ8QaANUibk2icmLyaR8TRx2Ug/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最主要的参数是 acToken,  fp

  

acToken参数

  

在core.v2.20.0.min.js 中搜索 acToken  ，打上断点，滑动滑块 ，停在断点处，如图

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlC4HEfIUOdGvJwM7f6OvBbIpmbX6WBmvguk1D1qCrPLKIKG5LibVXRNibg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

查看堆栈，追溯进watchman  js, 即可看到， 之后js 调试缺啥补啥

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlCltWbbkCI5TlfBvjJibIuOiasMe1WgoxoicdCSVMCPF6ib9PPDNPUOpsW3g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

FP参数

在core.v2.20.0.min.js 中搜索 'fp'  ，打上断点，滑动滑块 ，停在断点处，如图

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlCt2tBTCmTBDic3Qib1FpdsZXsNPovUR0j9wWIHYWriadLCHozfuecMoQbA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

搜索 fingerprint ， 打上断点，调试

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlC3Lma4genRqQ0xCG2cFBYP2qw2VcX7lMkovicnAibNp7Lfa7FTyKGZJuQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

发现fp 值来源于 window.gdxidpyhxde ,  通过js hook 脚本 hook window.gdxidpyhxde 生成 找到对应位置，进js堆栈

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlCWYMv5ZicOC0iaMfmoqFK57aibhU06TmgkHXtw9Z6wqVXTnoIb5eAUw2Hw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下来 在该方法进去位置打断点，清空缓存重新加载

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlCib8AkRoPGQ8UsRt1qKtAIcPEQTGtDstgI0wEUkDp0jHnwVTYwXW7wjQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

参数 _0x5b5411 中

fp 是拿到浏览器属性生成的， 里面有 sessionStorage， localStorage， canvas指纹， UA, 屏幕高度宽度等等

h 是网站域名， u 前三后三是随机值，中间是时间戳 和最后结果fp 的时间戳一致，v固定即可

其他部分扣js代码即可 生成fp

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlCfEx4OhuVWPuPssD36vhlVcSZ9L4KOo9WhlOMricgPukggoX1HgTjU0g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

最后请求网站，拿到validate结果

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZhJZCItkPvd94laYfftQ47Sd1MbgnNlCUd0Kb9c5y4s1WblYZVbccibtUXEJyqft4KAG9FzguIaZKbyktiabtcPA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

有点难的就acToken 和 fp， 其他参数比较简单，有时间后续文章更加详细介绍