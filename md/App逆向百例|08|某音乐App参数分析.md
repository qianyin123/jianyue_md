> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/8SU7hOUKHkEGBeis_VbK3g)

**观前提示：**  

**本文章仅供学习交流,切勿用于非法通途,如有侵犯贵司请及时联系删除**

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1ibibSicdgxscAL02hW7AOwBun1wAYmc3Z2aeONRK9l6Kms6hZ7vNAciaWg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

```
样本：aHR0cHM6Ly9wYW4uYmFpZHUuY29tL3MvMTV3cl9BOGlOb3NGMDlzNkZWM25Ia0E/cHdkPWxpbm4=
```

  

0x1 抓包
======

打开Charles抓包

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1Nua2FiafLbTrpMNdslMIU8iaX113j17lia9VXdYUviaIdNvJIOQEKI4MSg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)提示`请先通过验证`却没有弹出其他验证窗口 而没抓包的时候正常 这里可能就是遇到证书验证了

这里直接使用`objection`

```
android sslpinning disable --quiet  

```

Pass掉证书验证后就能正常抓包

```
{  
 "support_third": "3",  
 "params": "23e489ecaf982f609fdd1e0dfcbd1993697efa6972c66137c7ca942578d932c15b05511fc6031d3439b1da752c0a03f96c969fc13a52fcdcfddeab59cb8e15dc",  
 "clienttime_ms": "1658552431536",  
 "dfid": "0QcooS19wtf02lKOV14fyS4X",  
 "dev": "Pixel",  
 "plat": 1,  
 "pk": "8087088AFF4397949CAB324500E75A4E471D520A5A31EB489202E9587C7925D013B50326F654E67B5D13021E610806B16DE91758C2597F00229F7BF963A2CE0C3A62E194201CE3B11BE3E95FD2FBA790487D1D74DCF6212688969AD03DB1129B2E328A4C26FB719CD5EC9A024B07F13F3B927848518E280D43E8384253DD87EE",  
 "t1": "987fc21df4cac6d3389781844574b561",  
 "support_multi": 1,  
 "support_verify": 1,  
 "gitversion": "8c4e364",  
 "t2": "4b940068884a2bf5cf3cd31c6096114dbbd2c1a0a2526b53a1b5d5cc6632a84387313f1df71ef71f9abe3d156d1eea5b2d6bf0356649b8fee93fe5bc201ed828a02dc11e1aafd3bfc871552c4d84fe6904d538c2cff69eb5f7bb3a5ea4cafe8da419359d136982399269cbfebf3a70a3",  
 "key": "290f4f6c76b25fd6db4afecc2b6f2e6f",  
 "t3": "MCwwLDAsMCwwLDAsMCwwLDA=",  
 "username": "uuuuuuuu"  
}  

```

0x2 参数分析
========

params
------

搜索`params` 结果很多![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1uTPPn7CV4OGicHwBibL99cCn6AF2Hy2zPd8A60KD81ib1TP50fCMw7ohQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

但是往下翻还是让我注意到关键的信息

```
jSONObject.put("params", com.xxx.common.useraccount.utils.a.b(jSONObject2.toString(), a2));  

```

随便点击一个跳转![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo19ho2PJWrRX9nHemXXJ4X57TSD7UkytpA8dktoXDYibgFZvSVpvX8XRw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)继续走![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1fWp3hsOibbWSibJp4fyEG2U84K15yib5hocPhgTGNiam5ES4d72kxdMhnw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)最终到达加密地方![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo180qOKH1qpJxwpibVIlwWHyP1bTNtZ3jbrUumJ6VK1YBRUSMCnEGuGzw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)就是一个普普通通的`AES/CBC/PKCS5Padding`

key和iv是上层传入的 a方法还不知道

```
key->a(str2).substring(0, 32)  
iv->a(str2).substring(16, 32)  

```

跳转了几次a方法后可以知道 就是一个普普通通的`MD5`![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1qjia6o7aqsh8kuGYXicepn12iaUVxW39dhMIIJarcCXPSMbfQJMiazteicA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)所以现在压力来到了`str2` 是哪里来的

先上`objection`hook 打印堆栈

```
com.xxx.android on (google: 8.1.0) [usb] # (agent) [834523] Called com.xxx.common.useraccount.utils.a.b(java.lang.String, java.lang.String)  
(agent) [834523] Backtrace:  
        com.xxx.common.useraccount.utils.a.b(Native Method)  
        com.xxx.common.useraccount.b.ad$b.qr_(SourceFile:1196)  
        com.xxx.common.useraccount.b.ad.a(SourceFile:507)  
        com.xxx.common.useraccount.b.ad.a(SourceFile:130)  
        com.xxx.common.useraccount.b.ad$2.run(SourceFile:345)  
        java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1162)  
        java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:636)  
        java.lang.Thread.run(Thread.java:764)  
  
(agent) [834523] Arguments com.xxx.common.useraccount.utils.a.b({"clienttime_ms":"1658482674312","pwd":"ppppppppppp"}, 21c6f58bd3c36f1b)  
(agent) [834523] Return Value: ff6883e9160413724281c365d083769ef02ba9045eb2b53e4698945bbbb7885caf91795602018d603e6bd71fd236f087c8250664f8b7e1741b994ba3112e24bb  

```

通过对`Backtrace`的打印观察 可以定位到正确的业务逻辑`qr_`方法

然后可以知道`str2`来自于`this.g`

```
this.f163691a.put("params", com.xxx.common.useraccount.utils.a.b(jSONObject.toString(), this.g))  

```

最终定位到`this.g`生成位置

```
KeyGenerator instance = KeyGenerator.getInstance("AES");  
str = a(instance.generateKey().getEncoded());  
str = str.substring(0, 16);  
public static String a(byte[] bArr) {  
  StringBuffer stringBuffer = new StringBuffer();  
  for (byte b2 : bArr) {  
    String hexString = Integer.toHexString(b2 & GZIPHeader.OS_UNKNOWN);  
    if (hexString.length() == 1) {  
      hexString = '0' + hexString;  
    }  
    stringBuffer.append(hexString.toLowerCase());  
  }  
  return stringBuffer.toString();  
  }  

```

也就是说 `this.g`就是随机的16位字符串

*   MD5后截取(0,32)为AES的key
    
*   MD5后截取(16,32)为AES的iv
    

pk
--

在`qr_`里找到`pk`的逻辑部分

```
JSONObject jSONObject2 = new JSONObject();  
jSONObject2.put("clienttime_ms", this.p);  
jSONObject2.put("key", this.g);  
String b2 = com.xxx.common.config.c.a().b(com.xxx.common.config.a.lq);  
if (TextUtils.isEmpty(b2)) {  
  b2 = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDIAG7QOELSYoIJvTFJhMpe1s/gbjDJX51HBNnEl5HXqTW6lQ7LC8jr9fWZTwusknp+sVGzwd40MwP6U5yDE27M/X1+UR4tvOGOqp94TJtQ1EPnWGWXngpeIW5GxoQGao1rmYWAu6oi1z9XkChrsUdC6DJE5E221wf/4WLFxwAtRQIDAQAB";  
}  
this.f163691a.put(LeftBottomIconsEntity.ICON_PK, com.xxx.common.useraccount.utils.h.a(jSONObject2.toString(), b2));  

```

其中`com.xxx.common.useraccount.utils.h.a`就是一个普普通通的`RSA/ECB/NOPADDING`

```
(agent) [708059] Arguments com.xxx.common.useraccount.utils.h.a({"clienttime_ms":"1658485309160","key":"05f3263011b110d5"}, MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDIAG7QOELSYoIJvTFJhMpe1s/gbjDJX51HBNnEl5HXqTW6lQ7LC8jr9fWZTwusknp+sVGzwd40MwP6U5yDE27M/X1+UR4tvOGOqp94TJtQ1EPnWGWXngpeIW5GxoQGao1rmYWAu6oi1z9XkChrsUdC6DJE5E221wf/4WLFxwAtRQIDAQAB)  

```

所以这里就是把随机的`this.g`带进去RSA加密作为`pk`上传

key
---

同样的可以在`qr_`里面找到

```
this.f163691a.put("key", com.xxx.common.useraccount.utils.d.a(this.q, this.r, this.s, String.valueOf(this.p)))  

```

这里传入了四个参数

*   this.q→appid
    
*   this.r→appkey
    
*   this.s→clientver
    
*   this.p→clienttime_ms
    

翻译下来就是`MD5(appid+appkey+clientver+clienttime_ms)`

t1
--

搜索`t1`找到

```
String token = NativeParams.getToken(a2);  
this.f163691a.put("t1", token == null ? "" : token);  

```

最后到达

`public native String _d(Object obj);`

从初始化中可以看到加载了俩个so![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo12ONwKriaD6XOw9cwqRKDEFEv4OuIE25iarSFWYatW80FRB4utnw0J4FA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

但测试下来只有`libj.so`是所需要分析的

打开大姐姐 简单搜索是不是静态注册

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1FuibeXVL7RvOHII6lKdvOI5ACrzctdxmPn7MubXXIp75YezawYvoSxQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)没有结果 那应该是动态注册了 从`JNI_Onload`开始看起

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1WqHazib6UW8WybqlDolF7mJjPmVWvKJBnwH3gjdQrUtdapticUNYsRRA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1bvf7LAbQbOyx2TzzCTHlIPhpjMHIQKJojPTWAQXibOmDibFGY5l3BnVg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1WgPNcUnbaL4UNbO7nWOMKialokPetEzrLwvuia02dUlkUPibE7LWWibpsw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)没有混淆 很直白的就看到了需要的地址偏移了

跳转过来后 第一眼并没有啥太关键的信息

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1w7h1ZCTlMxPuOlEgQbNib6m6GvpU1544VhTnWLtjFzol5hibyVMQ2zPA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

继续进入`f4(v8)` 进来之后比较重点看`f5` `f9` `h15`![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1EamvYbQHuTr4PCDqHzjbwq7ibuvLRQPxLXZC93HXxQ4X9AV9mWxhtpQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进入`f5(&v42)` 还没看到特别的信息![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1G7LSrbxODk3ztUUoZP8qicQSdx4sv6LeWXhTbmbJ8tjfriaMK0PAqYhg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)进入`h9()` 所以就是一个取时间戳的方法![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1HMcELk5GBhgfx8BERtDticia8picxeJib8icSgXpuPIQdHbQEYvhmuH20zw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)进入`f9(v45, v35)`整个循环下来就是读`a2`的地址做偏移然后拼接![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1StwuNibY4Oh5RsUUTIIibrdRK4zfUuD1BC5ObnpoyruI4opXV0qJnJhw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)最后从`a1`传出结果

通过hook的结果为`|1657866669000`

进入`h15(&v46, v45)` 进来之后流程较长 直接找到`h14`

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1RZg4ic4broCJZRKuExxZn6pYibREibq0CicpOxPAmlwiaVXclY1ibmqlasZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)进入`h14(1, inbuf, out, key, iv)` 发现了普普通通的AES字眼![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1RVkjj7stwn5tFUzyyoC7QS2Fp7c17HL4qbgxafAPSCC6lf6RVGSq3Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)所以现在就是要找`key`和`iv`

回溯一下![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1JGDSKib800IqlyOmRRXsDw6ZFq5obiaSl2P4FpT1oRURq2CLZH8icsyxg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)简单翻译一下算法

```
var v26 = [0x66, 0x64, 0x33, 0x38, 0x37, 0x38, 0x39, 0x31, 0x32, 0x35, 0x34, 0x65, 0x36, 0x63, 0x65, 0x3C, 0x21, 0x29, 0x4D, 0x43, 0x36, 0x4C, 0x58, 0x33, 0x7B, 0x00, 0x68, 0x64, 0x54, 0x28, 0x7B, 0x39]  
var unk_39078 = [0x58, 0x42, 0x1d, 0x7d, 0x72, 0x0f, 0x2f, 0x39, 0x03, 0x4b, 0x36, 0x59, 0x01, 0x35, 0x1e, 0x1f]  
var v8 = v26.length  
var v9 = v8 - 2  
var key = ''  
while (v9 != v8 - 18) {  
    if (v9 < 0) {  
        break  
    }  
    //v9-v8+17 相当于倒取  
    v26[v9] = v26[v9] ^ unk_39078[v9 - v8 + 17]  
    --v9  
}  
for (var i of v26) {  
    key += String.fromCharCode(i)  
}  
console.log(key)  

```

最终拿到key为`fd387891254e6cedc4019ca0061ea6d9`

在iv这里只是取出了`dword_39088`的16位 并没有做计算操作![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo15aNQUkAyj05IGicxyiaxibprGDtd34nIXNAYwmCmibxze0uzGcrnGJoJqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)也就是iv为`63 34 30 31 39 63 61 30 30 36 31 65 61 36 64 39`

即为`c4019ca0061ea6d9`

验证结果 能够成功解密出来![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1Hib2sSwMw8wjObuibwoPaF0eDxIRZFPRuPTfHZxQl1KKljvRUDxIkteQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

t2
--

搜索`t2`找到

```
String machineIdCode = NativeParams.getMachineIdCode(a2);  
this.f163691a.put("t2", machineIdCode == null ? "" : machineIdCode);  

```

最后到达

`public native String _e(Object obj);`

根据前面的动态注册位置跳转到`cc::e`

看了一下大概逻辑和前面的`cc::d`的逻辑差不多

*   `cc::h4(&v38, a1)`获取ANDROID_ID
    
*   `cc::h5(&v44, a1)`获取DeviceId
    
*   `cc::h6(&v50, a1)`获取HardwareAddress
    
*   `cc::h8(&v56, a1)`获取MODEL
    
*   `f5(&v62)`获取timeofday
    

然后在`f9(v65, v34)`拼接出明文

```
d0c2e9f3ecc7523f9122caca02c99d72|0f607264fc6318a92b9e13c65db7cd3c|02:00:00:00:00:00|Pixel|1657866669004  

```

`f6(&v66, v65)`为AES加密

进到`f6` 老规矩 找`key`和`iv`

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1PAdxNoOtUnpib5XMPlWBOBDFLBPYjupyH0LnPK7GnP2Md1ISudXxMtA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)根据伪代码还原出key的计算过程

```
var v25 = [0x67, 0x18, 0x6B, 0x0B, 0x31, 0x78, 0x27, 0x7B, 0x54, 0x37, 0x13, 0x25, 0x6B, 0x53, 0x6D, 0x45, 0x64, 0x62, 0x66, 0x62, 0x62, 0x39, 0x31, 0x61, 0x66, 0x30, 0x65, 0x61, 0x39, 0x63, 0x61, 0x37]  
var byte_390F8 = [0x04, 0x7D, 0x5A, 0x6E, 0x09, 0x40, 0x43, 0x4C, 0x6C, 0x53, 0x75, 0x43, 0x59, 0x62, 0x5E, 0x77]  
var key=''  
for (var i = 0; i != 16; ++i) {  
    v25[i]=v25[i]^byte_390F8[i]  
}  
for (var i of v25) {  
    key += String.fromCharCode(i)  
}  
console.log(key)  

```

得到key结果为`ce1e88d78dff2132dbfbb91af0ea9ca7`

iv的部分还是跟前面的一样截取16位

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1zvNVlOPEQXrzwniabHc5Plic7URTIicsnic5zv7QqpibBdphibKgyFVlgL3Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)即iv为`dbfbb91af0ea9ca7`

验证结果 能够正常解密![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1NUgKibmJevhicSa9fRxXuVicarFeRIjshMYMhibTyF4O0Ghicybl8prPO2A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

t3
--

直接开启搜索大法搜索`t3` 显示的结果刚好只有一个

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1ysR8gOKL6QoQSAJ4GHicf4fyo6faV0avBjRslISKDm9Aue077M6cynQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

跳转过来可以看到代码中一堆拼接操作 然后进入`c.a`中计算

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1xKd8OTubtemqUJ3XAKHsicmYbGrY1trKickG8D8icys8ngd6Fpibrw1ZXg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进来之后 能看到这可能是base64

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1gkiavYXSibxfA3Bkuycy6KlsaXUqz7FcYoTdOKnO9iap7ibljs1ibnmF5lA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所以验证一下结果![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1icZDeaWZ7x9vOWvlFHO1ryd6ibeiaZ3WicIK75oD9GAXWWLxmeez2X9Hrg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

结果能正常解密 而且也能看到前面拼接的明文结果

0x3 Unidbg黑盒调用
==============

先造个基础框架运行

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1ScsC8Sv1VjBb3wicibkShqxUWkFVicQAcMqoicSSeQCeWOcphEH81CYd5g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)根据报错补上

```
emulator.getSyscallHandler().setEnableThreadDispatcher(true);//多线程  

```

补上之后不报错开始call方法

```
public void call_e(){  
    List<Object> args = new ArrayList<>(3);  
    args.add(vm.getJNIEnv());  
    args.add(0);  
    DvmObject<?> context=vm.resolveClass("android/content/Context").newObject(null);  
    args.add(vm.addLocalObject(context));  
    Number number=module.callFunction(emulator,0x123e5,args.toArray());  
    System.out.println(vm.getObject(number.intValue()).getValue().toString());  
}  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1otB109IEt6bvEWJo8ARVWvrNUjwbRZPj3jYhvnNtj5Y9jwc7dFJia1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)提示缺少`getSafeDeviceId()`

```
@Override  
public DvmObject<?> callStaticObjectMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {  
    switch (signature){  
        case "com/kugou/common/utils/SecretAccess->getSafeDeviceId()Ljava/lang/String;":{  
            return new StringObject(vm,"8eea3d202e42");  
        }  
    }  
    return super.callStaticObjectMethodV(vm,dvmClass,signature,vaList);  
}  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1xnI53yyTfBdgiaLyXlBqGuV9RjlibUjD4rbfvuUkvzrkc6MPKCiaEPeJw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

提示缺少`MODEL`

```
@Override  
public DvmObject<?> getStaticObjectField(BaseVM vm, DvmClass dvmClass, String signature) {  
    switch (signature) {  
        case "android/os/Build->MODEL:Ljava/lang/String;":{  
            return new StringObject(vm,"Pixel");  
        }  
    }  
    return super.getStaticObjectField(vm,dvmClass,signature);  
}  

```

补完后正常返回值![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsOFGUkJbWjdwtLVibRBHMRo1yc3J2pc2XA1vcvsywCTLPMbOyj6AicKiaLoBeicLTbIicN6w4hMrMnRHyQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

**感谢各位大佬观看**

**感谢大佬们的文章分享**

 **如有错误 还请海涵**

**共同进步**  

  

[完]

* * *

  

点赞 在看 分享是你对我最大的支持

逆向lin狗