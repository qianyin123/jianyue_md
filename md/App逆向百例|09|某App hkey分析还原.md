> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qIE3foeuhMCbKfJ-kCzB1Q)

**观前提示：**  

**本文章仅供学习交流,切勿用于非法通途,如有侵犯贵司请及时联系删除**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGUwfiayQcL07lkxJgLEFTNDjuuMCSPTgEGOkKNd4PgnU9WKPl1xyI23Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

```
样本：aHR0cHM6Ly9wYW4uYmFpZHUuY29tL3MvMU1LbEhzbFRoTU8zeHFCODhVbER1NVE/cHdkPWxpbm4=
```

0x1 抓包
======

*   有证书验证
    

使用`objection`过掉证书验证后可正常抓包

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGeIzSMTZjGKs8yIl7KH14ViakroXDPdu5QmnQTmgYZS0KsP237kmSHmQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

本次受害参数为`hkey`

0x2 找加密位置
=========

使用`jadx-gui`打开 发现并没有加壳

直接搜索`hkey`也没有看到有用的地方

那是因为真实的字符串处已经被处理过了

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGpSEc3OAW5QoiaWnkNY7lVM1cSSqcvYic2PGmMcJM9csccibLtekQxRxTg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

通过替换字符最终组成`hkey`

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGZ2zXgxedqibPwnB1em7OTUfZXGkwm3hblh285Qkqu8K5BIAr0ueFRcA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最终定位到

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGXvvFHWiblZFiaEnx6NGqWbwYn4PqPn1AG0SLWL7YneJf25hrt40PvFzQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)传入的参数分别为

*   上下文
    
*   url
    
*   时间戳
    
*   随机32位字符串
    

0x3 Unidbg黑盒调用
==============

先复制粘贴好框架运行 发现并不需要补环境可以直接运行

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGn198EtgcMJ9smTZyNia8EqiaTqJ5xBUIuL1pQbian1Ja6FicickXQ9npMlg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

看到`encode`方法是动态注册的

主动调用

```
public void call_encode(){  
        List<Object> args = new ArrayList<>(3);  
        args.add(vm.getJNIEnv());  
        args.add(0);  
        DvmObject<?> context=vm.resolveClass("android/content/Context").newObject(null);  
        args.add(vm.addLocalObject(context));  
        args.add(vm.addLocalObject(new StringObject(vm,"/bbs/app/api/search/hot_words/")));  
        args.add(vm.addLocalObject(new StringObject(vm,"123456")));  
        args.add(vm.addLocalObject(new StringObject(vm,"GkaZmuMPhIxUBiIjH8P8kmnuA9vcsKU7")));  
        Number number=module.callFunction(emulator,0x3369,args.toArray());  
        System.out.println(vm.getObject(number.intValue()).getValue().toString());  
    }  

```

主动调用发现也不用补环境即可出结果![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGaO7lVPGd4ibXybG5OW8FvwE7ABo6kja5Lv6uLDzDOtd5dGMicMiaydI8g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

0x4 算法还原
========

将`libnative-lib.so`放到大姐姐看看代码

跳转`0x3369` 先从伪代码分析![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGVm0icHJWoLZDFEQNln2FUqZ5EFtTFrhSiaRqsf30B8VePQRHYMIF5mng/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进入`sub_1B28` 可以知道这里的目的就是签名验证

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGbibfmTEISAbSvaGzEsbdzm6LXEwM05RlelDZXQPk5wUB1BeQGLdZ6Jg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

回到上层继续看 根据if条件判断v15是否小于1 从前面知道传入的v15的长度是32 所以走到如下分支

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHG7frfBtE2xONJpMjmZyVOh5fLUpfnDSibvZRCzobicuNdzALcZWRgMCCg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

v13就是传入的32位随机字符串

而`*v13++`就相当于从前往后取字符

经过计算和判断 最终叠加到v17上

这里的v17其实是指代v47

其中`v19-97`这里 如果结果是负数 py算出来的结果是能直接是负的 但是so算出来的却是正的 所以在代码中还需要做个判断![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGjj5zGBGp2LOgeEb3Dkn5uuakpBslCMbjSniaibOK6Rn7y6ibEVdYzw9OA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所以需要把代码改为`20 < 26 and v20 > 0` 这个是在调试过程中才发现的一个坑

分析完后就可以直接翻译

```
v15 = len(v11)  
v16 = 0  
i = 0  
v47 = "23456789BCDFGHJKMNPQRTVWXY"  
while (v15):  
    v19 = ord(v11[i])  
    v18 = v19  
    v20 = v19 - 97  
    if (v19 - 48) < 10 :  
        v16+=1  
    if v20 < 26 and v20 > 0:  
        v18 -= 32  
    v15-=1  
    v47 += chr(v18)  
    i+=1  

```

上面的代码执行完后v47会叠加延长到58位

并且根据计算得出v16的值

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGH6p7LFNpcvM4VMWsQRzqFszkA5WwicVRqVZkl8tBthh0K8CFClj4waQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里的atoi就是将字符串转化为十六进制 然后再加上前面计算的v16

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGTTP6T96xQnb7UbIFzBDNlXfSpJcIr9RtaEmKmlguEO0J8eQCXUlibiaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后是一些赋值和申请内存的操作

进入`sub_1CF8(url1, url_len, v24)`

进来之后是一堆看不懂的操作 那之前造好的Unidbg就可以上线了

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGFMYqEJW17NKXMBv4iaXgMDrxOjXoMsQAfMXBgLS7yrnBqiarMxY1g8jg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

目前是对`sub_1DBC`这里有疑问 看汇编对应的是`LR`

直接在`0x1D48`下断

断下来后 直接`m+地址`将地址上的数据输出出来

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHG8OnNpPXDAsUGcPJbk1GgzzFoiaUe8FW5EswBpwvMwib6qVWUticib13ZQQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到`LR`其实是一个码表 并非`sub_1DBC` 所以这里的操作有可能就是`BASE64`

验证一下 在`0x1CF8`下断 `mr0`为url

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHG6xJ8KvKF4lBR1JiaUECM3knJ7GIemc4vHsCHCRk23Qr0ibicxLEZ6hJTw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

`mr2`对应地址为`0x401d2000`

记住这个地址 先下个`blr`然后`c`放行

接着输入`m0x401d2000`即可看到方法执行完输出的`mr2`的内容

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGNiaXcoEgCE73l6sLlV0kqXmXYxbo5DBEZzichZE9tkl1oaTxBr5LNibfg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)验证成功![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGwIspZqBpVg8aDLXtIKDiaFeupfCrteoO2tyuGnepib3hicjTcBOtQVtEQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

继续看`sub_1E80(v41, v24, &v42, v26, 8u)`

进来之后一眼看到`0x5c`和`0x36`

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGlJgiavD9Wic1806jFGGlIJCRRhxI6HpRkQSzQA2PccRt1F2icprp5IwvA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

懂的都懂吧 不懂也没关系 继续看`sub_1F68`

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHG2byWuaibl5bNKVu2vx3icxoR45kicXiaP1SW1JSzKCRSgwdGibsST0I3Law/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所以啥都告诉你了 这就是一个`hmac-sha1`

直接hook`0x1E80`

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGtzeK2gwuT7zdc82Y4wxVyVufl6TaOAJC2v4C6btwibsrp1MsXGbibX0g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

`mr1`为KEY 也就是前面BASE64后的url

`mr2`为明文 也就是加上v16后的十六进制时间戳并往前填充`00`到16个字节

对比验证结果一致![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGeBbKQAXepxeGnGoy5ueVKZVKv9Blu7nicZibE19tTgHsGElpBeZcVtcg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGoPrZ884lAk6wlOUGGYFXgw01PJJooGR4GaQkhFN05bm3ACnu6r3BEw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

继续看

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHG6WAEJTULG5Vrd5z3ZCicnwyaU64jXibPicnYVYCCOzknmm7x66mkr955A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

hmac-sha1的结果保存在v41中 而代码中又取出v41中19后的位置 也就是取出`53`![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGFicWqVBAYZ12PP7PnnibxicRuJpb4ibJ8eMia9tUibv7BL9agQLl8kYKvWRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接着进行计算`v41[19] & 0xF`取低位`3`

然后从v41中取出索引`3`的值 所以难道是取出`8E`?

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHG72joZ7o8KAOgW5OfXjibvtwVRHYxbG1icnoeXF8ia2giaC4Bw9m3mtZIVA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其实并不是 转到汇编这里

```
.mytext:0000354A LDR R0, [R5,R0]  

```

可以看到伪代码的翻译并没有太大的问题 但是要注意的是`LDR`的作用![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGQwHgibTBY6IKlOYickUtl619ehp1uQI9Fke6pS5sYYd0ayqwzUGD5bsA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)所以正确的是

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGLNViczSt77GslKkTOGSSz5UDMPm1Y72gO67aNfHIK7DXgK1zpH2pAPw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

后面的`bswap32`的作用是32位寄存器内的字节次序变反 因为读取顺序是从低往高的 所以需要次序变反

我们翻译的话可以直接是`0x8EA583AA` 后面的`0xFFFFFF7F`也需要同时变成`0x7FFFFFFF`

继续往后走就是一段循环操作 生成5位的字符串

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGVPV9ibW4aRKzFdE4ItglSaIpjNt3hoGHLZtLLCCIiajeTJZ0SiaLdIibzQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)直接翻译即可 走到这里 已经完成了前面五个字符的生成 还差后面俩个数字

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGIgm7hp1Zic9HiajqFEwgNdk1F72T1Te268yawIfJFFnxuUN8vKEas63w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里取出了五个字符中的后四个进到了`sub_1900`计算![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGlic7asy838dMmYgEYiajYJyCnsU9WldOzerhP0tjUV4cubskXz8CFx5g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进入`sub_1900`能看到多次出现`sub_1888`

进入`sub_1888`能看到就是一些计算操作

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGkFapk49uZ6tKvfnYN26zcBfcNem4DszbRmT3NsGtFtSFYaKMdiattFA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

那就直接翻译下来就行了吧

```
def sub_1888(a1):  
    v1 = 2 * a1 ^ 0x1B  
    if (a1 & 0x80) == 0:  
        v1 = 2 * a1  
    v2 = 2 * v1  ^ 0x1B  
    if (v1 & 0x80) == 0:  
        v2 = 2 * v1  
    v3 = v2 ^ v1  
    v4 = 2 * v3 ^ 0x1B  
    if (v3 & 0x80) == 0:  
        v4 = 2 * v3  
    v5 = 2 * v4 % 256 ^ 0x1B  
    if (v4 & 0x80) == 0:  
        v5 = 2 * v4  
    return v5 ^ v4  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHG7J5oTf0oibdePCma1AsYyHKEqttqic1ms0d6fu9LYoBXHxWj2vQC0yYA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)对比一下hook结果传出`0x8c`也就是`140`和我们的结果相差甚远![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGSyx5wy5ANtR4Du4Pia4dbShtYr4pYTyOtdsl3iaFoFerUhrjEnQZPt5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)那就直接tracecode吧 选定`0x1888-0x18CE`

通过观察汇编 原来是这里的`uxtb`指令的位置开始出错了

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGBdoicWDYVGHgzuG79IvzAL3F1icBTab8228EZ3O0StwmuaibSTpp7SbIQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

网上的解释是

> UXTB
> ----
> 
> 每个32位整型都有4个字节，该命令主要将4个字节的其中一个字节提取出来，然后转成一个新的32位整型

所以我们要改的话也要根据汇编的实际情况来修改代码`0x160 % 256 => 0x60`

```
def sub_1888(a1):  
    v1 = 2 * a1 % 256 ^ 0x1B  
    if (a1 & 0x80) == 0:  
        v1 = 2 * a1 % 256  
    v2 = 2 * v1 % 256 ^ 0x1B  
    if (v1 & 0x80) == 0:  
        v2 = 2 * v1  % 256  
    v3 = v2 ^ v1  
    v4 = 2 * v3 % 256 ^ 0x1B  
    if (v3 & 0x80) == 0:  
        v4 = 2 * v3  % 256  
    v5 = 2 * v4 % 256 ^ 0x1B  
    if (v4 & 0x80) == 0:  
        v5 = 2 * v4 % 256  
    return v5 ^ v4  

```

修改后验证成功![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGILGZlsG06d322T1rzn7wOw7MOC5Zngo3svVuIoL1NBlicLnMTiaYpfQg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

回到`sub_1900`

还是老规矩先tracecode先留着备用

传入参数为`58 00 00 00 58 00 00 00 4A 00 00 00 54 00 00 00`

传出结果为`78 00 00 00 5B 00 00 00 D9 00 00 00 FA 00 00 00`

跟前面分析的`sub_1888`一样 需要注意处理uxtb 其他大部分可以照着伪代码翻译过去 但是还是有几处错误

`sub_1888(*a1)`中的`*a1`指代`58`

`LDRD.W R2,R4,[R10,#8]`中R10为a1 意思是 将a1偏移+8后存入R2 将a1+8+4后存入R4 所以后续的`v8`为`4A` 而`HIDWORD(v8)`和`SHIDWORD(v8)`为`54`

其他基本照着翻译下来即可 翻译完执行的结果为`[120, 91, 35, 0]`

对比hook的结果可知0x78=>120 0x5B=>91 0xD9=>217 0xFA=>250

所以问题还是出在了2和3里面

```
a1[2] = v4 ^ v24 ^ v22 ^ v8 ^ v26 ^ v13 ^ v14 ^ v25;  
a1[3] = v14 ^ HIDWORD(v8) ^ v22 ^ v8 ^ v10 ^ v17 ^ v27 ^ v18;  

```

直接看tracecode 从`0x1A22`开始看![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGL6klO6ibcumMKm0sIlibKLZse4z3Wb66AqLU0IticIWSDJ2icz7ib0sRdMg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

v4为0xb0 v24为0x58 v8为0xb0 对比到这里问题就出现了

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGjGB3ic96lXibLR352RWMBv9I9uom9G0nwwScrrGzKnJ2y51LuRgGxLAA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

回到汇编v8指R2 所以根据汇编 R2正确的指代是v15

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGHzLnvayWbgTfqMyPjGG7kRDk5PJppyFG9mxFMcuIWzXsFS17gd2KXQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所以整个`sub_1900`翻译完是

```
def sub_1900(a1):  
    v2 = a1[0]  
    v3 = sub_1888(a1[0])  
    v27 = v3  
    v4 = 2 * v2 % 256 ^ 0x1B  
    v24 = a1[1]  
    if (v2 & 0x80) == 0:  
        v4 = 2 * v2 % 256  
    v5 = v3 ^ v2  
    v6 = 2 * v4 % 256 ^ 0x1B  
    if (v4 & 0x80) == 0:  
        v6 = 2 * v4 % 256  
    v7 = v6  
    v26 = v6  
    v23 = sub_1888(a1[1])  
    v8 = a1[2]  
    v9 = v7 ^ v5 ^ v23 ^ a1[3]  
    v10 = 2 * v8 % 256 ^ 0x1B  
    if (v8 & 0x80) == 0:  
        v10 = 2 * v8 % 256  
    v11 = 2 * v8 % 256 ^ 0x1B  
    v22 = a1[2]  
    if (v8 & 0x8000000000) == 0:  
        v11 = 2 * a1[3] % 256  
    v12 = 2 * v10 % 256 ^ 0x1B  
    if (v10 & 0x80) == 0:  
        v12 = 2 * v10 % 256  
    v25 = v12  
    v21 = v12 ^ v9 ^ v10 ^ v11  
    v13 = sub_1888(a1[2])  
    v14 = sub_1888(a1[3])  
    a1[0] = v21  
    v15 = 2 * v24 % 256 ^ 0x1B  
    if (v24 & 0x80) == 0:  
        v15 = 2 * v24 % 256  
    v16 = v24 ^ v2 ^ v4 ^ v23 ^ v13 ^ v11  
    v17 = 2 * v15 % 256 ^ 0x1B  
    v18 = 2 * v11 % 256 ^ 0x1B  
    if (v15 & 0x80) == 0:  
        v17 = 2 * v15 % 256  
    v19 = v16 ^ v17  
    if (v11 & 0x80) == 0:  
        v18 = 2 * v11 % 256  
    a1[1] = v19 ^ v18  
    a1[2] = v4 ^ v24 ^ v22 ^ v15 ^ v26 ^ v13 ^ v14 ^ v25  
    a1[3] = v14 ^ a1[3] ^ v22 ^ v15 ^ v10 ^ v17 ^ v27 ^ v18  
    return a1  

```

运行结果为`[120, 91, 217, 250]` 一模一样

回到上层继续看![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGGcaylXztiaGGrAIy51xcbj8gqBicdZkarB9QL9UKyWibxFeKsvEUqHAiaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里就是直接将返回的结果求和然后%100即拿到最终的俩位数验证一下

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHG2IIulP7xKgiczdHA27xxCYBqGmrSH6h7exD763c2uGAZ9sfTyvXAiaQw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHGkYI3nx7ThNMZJeRyJ2ZQTAWDX5aa8pGefatUyYI6Bgjj80FoBoH0Bg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

到此 所有算法都已经明了啦

![图片](https://mmbiz.qpic.cn/mmbiz_png/Em7AaVMQYsMVicPU5NXGAlOdX0caoejHG1HPlPiaiaWNwaoH6Me4CMNwXzlUSn54QxGA0QpXa7qFF9CDib6xR06mHQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

  

**感谢各位大佬观看**

**感谢大佬们的文章分享**

 **如有错误 还请海涵**

**共同进步**  

  

[完]

* * *

  

点赞 在看 分享是你对我最大的支持

逆向lin狗