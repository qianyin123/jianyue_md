> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iIXdgPTwmw_nOw6b_aYkEA)

一、前言
====

拖更太久了, 很多读者朋友私信催更。在空闲之余，总结下 akamai 的算法流程。

akamai 我总结为三点  
1. 算法(最为简单)  
2. 流程(流程不对，很难突破难点的网站)  
3. 最后就是 tls 了，想要稳定的并发，tls 必不可少

二、抓包分析 sensor_data
==================

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/5svNlLic6gdvdpn0icNAibIK5WAH02xm8TMKS5WkxNR8A9onaKFmKCDkXJwLs79eOrLxcGuiayXBpcicJzY4tMebbAQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)  
主要还是提交 sensor_data 的 post 请求，sensor_data 包含每个 js 的 key, 浏览器环境。

现在大部分网站应该会加载 2-3 次。

如果是 `3;0;1;2048;`开头, 一般为第一次提交 sensor_data, 且不包含轨迹信息;

如果是 `3;0;1;0;`开头, 包含轨迹信息

三、分析sensor_data
===============

打 xhr 断点，定位到如下位置, a 就是 sensor_data

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/5svNlLic6gdvdpn0icNAibIK5WAH02xm8TMECRBWI2WVuViczEvzr4D5icvvApmdQTBulOSVL4D5GWfeq2foTHaPwicA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

向上追，发现 `BO2` 就是最终的结果, 然后找`BO2`是哪里生成的, 搜索`BO2 =`就可以定位到

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)  
这里加断点刷新网页(下文中 js 和上述可能不同，js 链接是动态变化的)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

可以看到这个对象里包含了很多值。这个对象和 akamai2.0 的大数组是一个作用，其中包含了检测的环境值，以及浏览器的指纹。

sensor_data 包含 3 部分  
  

```
"3;0;1;0;4469043;jTXJqkQ8O2ktqnjTR8NMMcKAizsRr3+1OcHUp5/2Uz4=;14,0,0,0,1,0;\"\"}\"mtX\",c5S[\"cD1\"0(eFzM\"e8;B\"<\">tZ&41amIdN2#]5*cOSN -k\"M\"4&E\"QqQra:3.m0E}v-N44Z[QpR8+da0)(y,\"@\"Ftu\"i\"<UdYf:23TY`*a^LV\"B\"adg\"i\"\"L8*\"Hxh\"h\"vdk!4\"l+i\"gt_D9\"jZ&@saK\"+OR\"E\"Tjc=&\"\"_\"ZZf\"L1zOfuu\"]J+\"8-pOGuSEPK{>>!\"-9G\"9\"\",\"znz\"R5}utvTn8\"R2!\"DhlHF+\"(V`lj7_?z<C\"VKL?KNRE\"]rU\"wPe\"?JZv:\"TB\"f47U,\"T5\"x\"x,\"kQ#\"FvYu\"Xs3S$,BII\"f%Cq)\"w1%2|-VO%x)\"#U\"GcuRv:a4)0vW\"(%ba\"H;VBaK6\"9D[G_aZqHWRAW)01qJ&L:C7!l0I;!=H,C|(+>}-ZGO)q\"X\"dZV\"s\"Of\",\"7`(\"qMJsJL=^W=^\"M;3\" \"D\"h\"zYw\"|\"]s@\"d\"4ZB\"l)8jH6n^{:{QFQtW]U\"\">g;\"w3;\"n7kre\"</d[\"H\"\"s\"y=x\" r8Sp?q\"hvK\"s\"\")\"?hM\"K(uZ0h/\"<oQ\"+\"\"H\"*b*\"CpC\"R_#K\"7\"cSJ1#I-4L3v$3]\"L$g\"ct_\"D.q|9uM*|j\"P!4\"b\"\"x\"S]E\"pF1gP|?^b\",>\"p=;isU3uWB{\"E7\"s};~T\"oVD\"wI5\"]-A\"7\"Bhvg_g;Lp4-r@~ySG2El||X#B8+iL2~o2ce`d%w[zi13LbD!$GhasP5Opu =mxl~p/ooanNF2x?;y@G{<P^DCu%Z~)&e7$w.~]5>BYAU`EQV8SvVYQKFR\"qNj\"|H~\"C\"\"q\"1;P\"k1g\"o8$\"x|phZZxe!qo\"M(\"Xij16\"l%,#\"e\"\"b\"qZ#\"UIBF/\":NPM\"^Kb*H\"{(ZO\"jDVN6z?~\")<C\"vvJ{NW\"p}8Q\"r Ud*51\"g$1\"9Ys\"2HJ\"2\"nV&:8~58l#]|f\"B\"O=+\"D+t!t2Jqy8@B.pKU\"-d \"x\"\"Z\";Pe\"qASP2\"@fC\"w\"\"U\"!8m\"5G[6M:`\"_/L\"5|F#l\"hvGV\"l)!F.\"P**\"o\"_7\"J\">02\"-3d~.u\"Jf+foVn\"O\"xV\"Mz6\"4#R[\"4\"\"}\"Jw4\"q\"N\"=lB\"`yk\"X\"qg/$Z\";9j\"_7L\"`;<HY\"PpU\"!\"i\"a#;\"@C\"V\")\"Z?%\"q 2\"6\"\"i6f\"gFu\"C?E9s\"1c*\"83}&i^,g*A-u*>^7_\"v<:\"XbngC\"mH4y\"=\"aLQ|6*@+1HW4ZI)FeFe~vC3*ze,&ex_{XnJP~ab|P6|*@Ut,\"M5_\"A4~\"H*^D.tqotj\"&R!q\"k\"Ur824\"E2[\"??3\"b\"\"4\"(+;\"w\"M\"3{<+9ydN\"*jr\"m\"^kmB\"LSG\"ZZ$\"`\"\"p\"(CY\"O/-dfL)\"\"z\"2:\"{jTof\"4WX\"3}\",D<\"&<%\""
```

*   第一部分: 
    

`3;0;1;0;4469043;jTXJqkQ8O2ktqnjTR8NMMcKAizsRr3+1OcHUp5/2Uz4=;`

主要是版本信息, js 的 key, 4469043 和最后面这个 key，是每个 js 的定值, 所以不同的 post 请求链接, 这部分是动态变化的

*   第二部分: 
    

`14,0,0,0,1,0;`

主要是时间差值, 没啥好说的。

*   第三部分:
    

``\"\"}\"mtX\",c5S[\"cD1\"0(eFzM\"e8;B\"<\">tZ&41amIdN2#]5*cOSN -k\"M\"4&E\"QqQra:3.m0E}v-N44Z[QpR8+da0)(y,\"@\"Ftu\"i\"<UdYf:23TY`*a^LV\"B\"adg\"i\"\"L8*\"Hxh\"h\"vdk!4\"l+i\"gt_D9\"jZ&@saK\"+OR\"E\"Tjc=&\"\"_\"ZZf\"L1zOfuu\"]J+\"8-pOGuSEPK{>>!\"-9G\"9\"\",\"znz\"R5}utvTn8\"R2!\"DhlHF+\"(V`lj7_?z<C\"VKL?KNRE\"]rU\"wPe\"?JZv:\"TB\"f47U,\"T5\"x\"x,\"kQ#\"FvYu\"Xs3S$,BII\"f%Cq)\"w1%2|-VO%x)\"#U\"GcuRv:a4)0vW\"(%ba\"H;VBaK6\"9D[G_aZqHWRAW)01qJ&L:C7!l0I;!=H,C|(+>}-ZGO)q\"X\"dZV\"s\"Of\",\"7`(\"qMJsJL=^W=^\"M;3\" \"D\"h\"zYw\"|\"]s@\"d\"4ZB\"l)8jH6n^{:{QFQtW]U\"\">g;\"w3;\"n7kre\"</d[\"H\"\"s\"y=x\" r8Sp?q\"hvK\"s\"\")\"?hM\"K(uZ0h/\"<oQ\"+\"\"H\"*b*\"CpC\"R_#K\"7\"cSJ1#I-4L3v$3]\"L$g\"ct_\"D.q|9uM*|j\"P!4\"b\"\"x\"S]E\"pF1gP|?^b\",>\"p=;isU3uWB{\"E7\"s};~T\"oVD\"wI5\"]-A\"7\"Bhvg_g;Lp4-r@~ySG2El||X#B8+iL2~o2ce`d%w[zi13LbD!$GhasP5Opu =mxl~p/ooanNF2x?;y@G{<P^DCu%Z~)&e7$w.~]5>BYAU`EQV8SvVYQKFR\"qNj\"|H~\"C\"\"q\"1;P\"k1g\"o8$\"x|phZZxe!qo\"M(\"Xij16\"l%,#\"e\"\"b\"qZ#\"UIBF/\":NPM\"^Kb*H\"{(ZO\"jDVN6z?~\")<C\"vvJ{NW\"p}8Q\"r Ud*51\"g$1\"9Ys\"2HJ\"2\"nV&:8~58l#]|f\"B\"O=+\"D+t!t2Jqy8@B.pKU\"-d \"x\"\"Z\";Pe\"qASP2\"@fC\"w\"\"U\"!8m\"5G[6M:`\"_/L\"5|F#l\"hvGV\"l)!F.\"P**\"o\"_7\"J\">02\"-3d~.u\"Jf+foVn\"O\"xV\"Mz6\"4#R[\"4\"\"}\"Jw4\"q\"N\"=lB\"`yk\"X\"qg/$Z\";9j\"_7L\"`;<HY\"PpU\"!\"i\"a#;\"@C\"V\")\"Z?%\"q 2\"6\"\"i6f\"gFu\"C?E9s\"1c*\"83}&i^,g*A-u*>^7_\"v<:\"XbngC\"mH4y\"=\"aLQ|6*@+1HW4ZI)FeFe~vC3*ze,&ex_{XnJP~ab|P6|*@Ut,\"M5_\"A4~\"H*^D.tqotj\"&R!q\"k\"Ur824\"E2[\"??3\"b\"\"4\"(+;\"w\"M\"3{<+9ydN\"*jr\"m\"^kmB\"LSG\"ZZ$\"`\"\"p\"(CY\"O/-dfL)\"\"z\"2:\"{jTof\"4WX\"3}\",D<\"&<%\""``  
主要是环境信息, 接下来着重分析下这部分。

```
{  
    "ver": "MDbyIBAJ6vyb6goh8rhG3dAY5TnQPZIcpOMtS0mmYeQ=",  
    "fpt": "-1",  
    "fpc": "94",  
    "ajr": "6bcf4731|20|28",  
    "din": [  
        {  
            "wdr": 0  
        },  
        {  
            "ua": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36"  
        },  
        {  
            "ucs": "8756"  
        },  
        {  
            "pha": 0  
        },  
        {  
            "asw": 1512  
        },  
        {  
            "hal": 874200715085.5  
        },  
        {  
            "nap": "Gecko"  
        },  
        {  
            "dau": 0  
        },  
        {  
            "ash": 944  
        },  
        {  
            "wih": 150  
        },  
        {  
            "tsd": 0  
        },  
        {  
            "ran": "0.392453782196"  
        },  
        {  
            "wiw": 1512  
        },  
        {  
            "she": 982  
        },  
        {  
            "nps": "20030107"  
        },  
        {  
            "xag": 12147  
        },  
        {  
            "npl": 5  
        },  
        {  
            "hz1": 430189  
        },  
        {  
            "wow": 1512  
        },  
        {  
            "ibr": 0  
        },  
        {  
            "nal": "zh-CN"  
        },  
        {  
            "adp": "cpen:0,i1:0,dm:0,cwen:0,non:1,opc:0,fc:0,sc:0,wrc:1,isc:0,vib:1,bat:1,x11:0,x12:1"  
        },  
        {  
            "swi": 1512  
        }  
    ],  
    "eem": "do_en,dm_en,t_en",  
    "ffs": "",  
    "vev": "",  
    "inf": "",  
    "ajt": "0,0",  
    "kev": "",  
    "dme": "",  
    "mev": "",  
    "doe": "",  
    "pur": "https://www.ferragamo.com/index.html",  
    "pev": "",  
    "mst": [  
        {  
            "kevl": 1  
        },  
        {  
            "mevl": 32  
        },  
        {  
            "tevl": 32  
        },  
        {  
            "devl": 0  
        },  
        {  
            "dmvl": 0  
        },  
        {  
            "pevl": 0  
        },  
        {  
            "tovl": 0  
        },  
        {  
            "delt": 1  
        },  
        {  
            "it": 0  
        },  
        {  
            "sts": 1748401430171  
        },  
        {  
            "fct": -999999  
        },  
        {  
            "dd2": 18703  
        },  
        {  
            "kc": 0  
        },  
        {  
            "mc": 0  
        },  
        {  
            "ww8": 0  
        },  
        {  
            "pc": 0  
        },  
        {  
            "tc": 0  
        },  
        {  
            "ssts": 2  
        },  
        {  
            "tst": 0  
        },  
        {  
            "rval": "-1"  
        },  
        {  
            "rcfp": "-1"  
        },  
        {  
            "nfas": 30261693  
        },  
        {  
            "jsrf": "PiZtE"  
        },  
        {  
            "jsrf1": 31955  
        },  
        {  
            "jsrf2": 94  
        },  
        {  
            "signals": "0"  
        },  
        {  
            "mwd": "0"  
        },  
        {  
            "hea": ""  
        },  
        {  
            "dvc": "aclfmfacf9acldarsQuQ,10,i+b+a+e+c+l+j+d+g+f+k+"  
        },  
        {  
            "srd": "0"  
        }  
    ],  
    "o9": 0,  
    "tev": "",  
    "sde": "0,0,0,0,1,0,0",  
    "pmo": "",  
    "dpw": "",  
    "pac": "",  
    "per": "8",  
    "pde": "",  
    "oev": "",  
    "if": "",  
    "fwd": [  
        {  
            "fmh": ""  
        },  
        {  
            "fmz": ""  
        },  
        {  
            "ssh": "0"  
        }  
    ]  
}
```

着重说下这个对象中比较重要的部分, 其他的很好找

ver: js 的 key

ajr: 动态函数, 大概一周变一次, 用于计算环境

din: 主要是 screen 信息

mev: 轨迹信息

mst: 主要是包含轨迹相关的信息，比如鼠标移动总时间, 总和, 轨迹信息, 最后一个点的信息，里面还有个比较重要的 dvc 参数，下文着重讲解下

四、dvc中vmp参数分析
=============

dvc = "aclfmfacf9acldarsQuQ,10,i+b+a+e+c+l+j+d+g+f+k+"  
可以看出, dvc 也是包含 3 部分

*   第一部分，vmp 函数生成
    
*   第二部分, 时间差值
    
*   第三部分, 每个 js 规定值，不同 js 变化
    

所以主要介绍下 dvc 中 vmp 这部分的生成

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

这里可以看出，nKA 就是 vmp 的结果。

然后看一下它的入参, 依次是时间差 、ajr、ajt(第几次生成 sensor_data), 轨迹和

然后就是单步进入`gO`函数，看下 vmp 的情况。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)  
这里 vmp 执行初始化，并入栈到 `Dh[183]`

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

vmp 初始化完成后，这个函数里才是 vmp 的主要计算逻辑，一般 vmp 都会有一个大循环, 比如 for 循环 | while 循环等等。这是 vmp 的特征之一。

下面我们可以看到, 入参 push 到 `Dh[183]`  
![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

`rP(EG, d5.b)`来判断是否跳出 vmp 循环  
`this[EG](this);`来执行对应的函数或者 vmp 参数压入栈操作、 参数释放等等

`this[45]();`这里主要是 vmp 指向的运行位置变化，跟进这个函数就可以看到下图所示, this[175][this[11][d5.P]++] 来改变指向的地址，选择是否继续执行 vmp 或者跳出。

那么接下来我们就可以在这个 while 循环的地方对 vmp 进行插桩分析

1.1 插桩点记录
---------

### part1: vmp 寄存器插桩

1.  while 大循环对每次调用函数，及 opcode 插桩
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

插桩内容如下：

```
"opcode: ", this[11][d5.P], "索引: ", EG, "栈数据: ", JSON.stringify(this[183], function (key, value) { if (value == window) { return undefined}return value})
```

1.  get、set 方法插桩
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

1.  String.fromCharCode 生成变量插桩
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

1.  变量赋值插桩
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

1.  数组赋值插桩
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

1.  push 进栈插桩
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

### part2: 运算插桩

通过正则快速找到下列计算点

`return\s+([A-Za-z_$][\w$]*)\s*\^\s*([A-Za-z_$][\w$]*)\s*;`

1.  `|` 位
    
    运算插桩点
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

1.  `+`
    
     加法运算插桩点
    
2.  `%`
    
     求余运算插桩点
    
3.  `^`
    
    异或运算插桩点
    
4.  `<<`
    
     左移运算插桩点
    
5.  `-`
    
     减法运算插桩点
    
6.  `>>`
    
     右移运算插桩点
    
7.  `+`
    
     加法运算插桩点
    
8.  `/`
    
     除法运算插桩点
    
9.  `*`
    
     乘法运算插桩点
    

1.2 插桩分析
--------

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)  
首先我们可以看出 vmp 参数分为两部分组成

`acjfkfadiiakda`和 `wc1uwe`

然后分别去分析对应的生成。

拿第一部分`acjfkfadiiakda`生成举例, 直接对生成的字符串进行反推，就可以很容易推出算法。

该算法主要用于生成一个设备指纹字符串，流程如下：

我重新插桩了一份生成，vmp 的结果是`ackfmimmamiljaacswyx`，现在说下第一部分长字符串的生成`ackfmimmamilja`, 短字符串生成是同理的。

```
TTIB:1554 this[11][d5.P]:  2346 EG:  21 this[183]:  [{"r":"a"},{"r":"ackfmimmamilj"}]  
TTIB:1567 GET方法: {zp: 'acswyx', Pc: div, F9: 5, FU: 1, NA: 2, …} [ SA ] -> ackfmimmamilj  
TTIB:1567 GET方法: (22) ['a', 'c', 'd', 'f', 'i', 'Y', 'j', 'k', 'l', 'm', '7', 'p', 'q', 'r', 's', '1', 'w', 'Q', 'x', 'y', 'B', '2'] [ 0 ] -> a  
TTIB:1554 this[11][d5.P]:  2347 EG:  25 this[183]:  ["ackfmimmamilja"]
```

从插桩结果可以看出是从数组`['a', 'c', 'd', 'f', 'i', 'Y', 'j', 'k', 'l', 'm', '7', 'p', 'q', 'r', 's', '1', 'w', 'Q', 'x', 'y', 'B', '2']`取索引 0 处的值得到`a`, 这个可以看多次循环印证，然后就是找索引`0`是怎么来的。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)  
往上找到可疑位置后，可以单步 debug 可以看到是 bmak_str 这个字符串，也就是01739499094860 取索引 `13`位置得到 0 的，这个多次对比可以印证。

01739499094860 很好找到生成，就是 0 拼接外部传入的 startTs

接下来找`['a', 'c', 'd', 'f', 'i', 'Y', 'j', 'k', 'l', 'm', '7', 'p', 'q', 'r', 's', '1', 'w', 'Q', 'x', 'y', 'B', '2']`数组生成就可以了

我们搜最前面`['a', 'c']`这两位如何生成的

```
TTIB:1554 this[11][d5.P]:  839 EG:  101 this[183]:  ["1",{"r":"1011001000110110010110000111101"},{"r":31},{"r":3}]  
TTIB:1567 GET方法: {fX: 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) Ap…KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36', Sc: 'a3cd9efghiYjklm7opqrs1uvwQxyBz2', UU: '01739499092136) Chrome/130.0.0.0 Safari/537.360', IX: 1494953021, S4: '1011001000110110010110000111101', …} [ VI ] -> 3  
TTIB:1567 GET方法: 1011001000110110010110000111101 [ length ] -> 31  
TTIB:430 DI % hD: 3 31 = 3  
TTIB:1554 this[11][d5.P]:  840 EG:  3 this[183]:  ["1",{"r":"1011001000110110010110000111101"},3]  
TTIB:1567 GET方法: {fX: 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) Ap…KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36', Sc: 'a3cd9efghiYjklm7opqrs1uvwQxyBz2', UU: '01739499092136) Chrome/130.0.0.0 Safari/537.360', IX: 1494953021, S4: '1011001000110110010110000111101', …} [ S4 ] -> 1011001000110110010110000111101  
TTIB:1946 this[183] push--> {"r":"1"}  
TTIB:1554 this[11][d5.P]:  842 EG:  40 this[183]:  ["1",{"r":"1"}]  
TTIB:1567 GET方法: 1011001000110110010110000111101 [ 3 ] -> 1  
TTIB:1554 this[11][d5.P]:  843 EG:  238 this[183]:  [true]  
TTIB:759 Ol << Xl: 0 24 = 0  
TTIB:759 Ol << Xl: 0 16 = 0  
TTIB:313 fU | GI: 0 0 = 0  
TTIB:759 Ol << Xl: 3 8 = 3  
TTIB:313 fU | GI: 0 768 = 768  
TTIB:313 fU | GI: 768 1 = 769  
TTIB:1554 this[11][d5.P]:  770 EG:  155 this[183]:  []  
TTIB:1554 this[11][d5.P]:  771 EG:  25 this[183]:  []  
TTIB:759 Ol << Xl: 0 8 = 0  
TTIB:313 fU | GI: 0 2 = 2  
TTIB:1618 String.fromCharCode得到-> Dc  
TTIB:1897 BL[W1]= (31) ['a', '3', 'c', 'd', '9', 'e', 'f', 'g', 'h', 'i', 'Y', 'j', 'k', 'l', 'm', '7', 'o', 'p', 'q', 'r', 's', '1', 'u', 'v', 'w', 'Q', 'x', 'y', 'B', 'z', '2']  
TTIB:1554 this[11][d5.P]:  776 EG:  25 this[183]:  [{"r":["a","3","c","d","9","e","f","g","h","i","Y","j","k","l","m","7","o","p","q","r","s","1","u","v","w","Q","x","y","B","z","2"]}]  
TTIB:759 Ol << Xl: 0 8 = 0  
TTIB:313 fU | GI: 0 2 = 2  
TTIB:1618 String.fromCharCode得到-> VI  
TTIB:1897 BL[W1]= 3  
TTIB:1554 this[11][d5.P]:  781 EG:  3 this[183]:  [{"r":["a","3","c","d","9","e","f","g","h","i","Y","j","k","l","m","7","o","p","q","r","s","1","u","v","w","Q","x","y","B","z","2"]},{"r":3}]  
TTIB:1567 GET方法: {fX: 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) Ap…KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36', Sc: 'a3cd9efghiYjklm7opqrs1uvwQxyBz2', UU: '01739499092136) Chrome/130.0.0.0 Safari/537.360', IX: 1494953021, S4: '1011001000110110010110000111101', …} [ VI ] -> 3  
TTIB:1567 GET方法: {fX: 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) Ap…KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36', Sc: 'a3cd9efghiYjklm7opqrs1uvwQxyBz2', UU: '01739499092136) Chrome/130.0.0.0 Safari/537.360', IX: 1494953021, S4: '1011001000110110010110000111101', …} [ Dc ] -> (31) ['a', '3', 'c', 'd', '9', 'e', 'f', 'g', 'h', 'i', 'Y', 'j', 'k', 'l', 'm', '7', 'o', 'p', 'q', 'r', 's', '1', 'u', 'v', 'w', 'Q', 'x', 'y', 'B', 'z', '2']  
TTIB:1946 this[183] push--> {"r":"d"}  
TTIB:1554 this[11][d5.P]:  783 EG:  52 this[183]:  [{"r":"d"}]  
TTIB:1554 this[11][d5.P]:  785 EG:  25 this[183]:  [{"r":"d"},0]  
TTIB:759 Ol << Xl: 0 8 = 0  
TTIB:313 fU | GI: 0 2 = 2  
TTIB:1618 String.fromCharCode得到-> Zk  
TTIB:1897 BL[W1]= (2) ['a', 'c']
```

初始化了一个数组`['a', '3', 'c', 'd', '9', 'e', 'f', 'g', 'h', 'i', 'Y', 'j', 'k', 'l', 'm', '7', 'o', 'p', 'q', 'r', 's', '1', 'u', 'v', 'w', 'Q', 'x', 'y', 'B', 'z', '2']`

然后循环对`1011001000110110010110000111101`对应每位的 `0`或`1`进行判断的, 如果是 位置是 0, 则 push 进数组, 或者索引%3==0，也 push 进数组。

*   `1011001000110110010110000111101`
    
    的生成
    

```
TTIB:1618 String.fromCharCode得到-> toString  
TTIB:1554 this[11][d5.P]:  171 EG:  3 this[183]:  [2,0,{"r":1494953021},"toString"]  
TTIB:1567 GET方法: {arguments: Array(1), BU: 1494953021} [ BU ] -> 1494953021  
TTIB:1946 this[183] push--> {}  
TTIB:1554 this[11][d5.P]:  173 EG:  194 this[183]:  [2,0,{}]  
TTIB:1567 GET方法: 1494953021 [ toString ] -> ƒ toString() { [native code] }  
TTIB:532 函数: ƒ toString() { [native code] } 参数:  
TTIB:1554 this[11][d5.P]:  177 EG:  164 this[183]:  [{"r":"1011001000110110010110000111101"}]
```

由`1494953021`toString 得到

*   `1494953021`
    
    生成
    

```
TTIB:1554 this[11][d5.P]:  277 EG:  140 this[183]:  [{"r":46},0,{"r":"01739499092136) Chrome/130.0.0.0 Safari/537.360"}]  
TTIB:759 Ol << Xl: 0 8 = 0  
TTIB:313 fU | GI: 0 10 = 10  
TTIB:1618 String.fromCharCode得到-> charCodeAt  
TTIB:1554 this[11][d5.P]:  290 EG:  3 this[183]:  [{"r":46},0,{"r":"01739499092136) Chrome/130.0.0.0 Safari/537.360"},"charCodeAt"]  
TTIB:1567 GET方法: {arguments: Array(1), WR: '01739499092136) Chrome/130.0.0.0 Safari/537.360', WO: -214999443, Ir: 46} [ WR ] -> 01739499092136) Chrome/130.0.0.0 Safari/537.360  
TTIB:1946 this[183] push--> {}  
TTIB:1554 this[11][d5.P]:  292 EG:  194 this[183]:  [{"r":46},0,{}]  
TTIB:1567 GET方法: 01739499092136) Chrome/130.0.0.0 Safari/537.360 [ charCodeAt ] -> ƒ charCodeAt() { [native code] }  
TTIB:1567 GET方法: {arguments: Array(1), WR: '01739499092136) Chrome/130.0.0.0 Safari/537.360', WO: -214999443, Ir: 46} [ Ir ] -> 46  
TTIB:532 函数: ƒ charCodeAt() { [native code] } 参数:  
TTIB:1554 this[11][d5.P]:  296 EG:  52 this[183]:  [{"r":48}]  
TTIB:1554 this[11][d5.P]:  298 EG:  25 this[183]:  [{"r":48},33]  
TTIB:759 Ol << Xl: 0 8 = 0  
TTIB:313 fU | GI: 0 2 = 2  
TTIB:1618 String.fromCharCode得到-> WO  
TTIB:1897 BL[W1]= -214999443  
TTIB:1554 this[11][d5.P]:  303 EG:  58 this[183]:  [{"r":48},33,{"r":-214999443}]  
TTIB:1567 GET方法: {arguments: Array(1), WR: '01739499092136) Chrome/130.0.0.0 Safari/537.360', WO: -214999443, Ir: 46} [ WO ] -> -214999443  
TTIB:1554 this[11][d5.P]:  304 EG:  30 this[183]:  [{"r":48},-7094981619]  
TTIB:1583 get XP= 48  
TTIB:752 kU ^ c1: -7094981619 48 = 1494953021
```

可以看出-7094981619 ^ 48 = 1494953021

48 => 对应'01739499092136) Chrome/130.0.0.0 Safari/537.360'索引上的 charCodeAt 值

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

-7094981619 对应 -214999443 * 33 得到

`-214999443`追踪到循环开始，就是初始值 bigNumber=5381

`01739499092136) Chrome/130.0.0.0 Safari/537.360'`对应"0" + String(startTs) + userAgent.slice(-32) + "0";

总结如下:

1.  初始化参数
    

设定 startTs（时间戳）、userAgent（浏览器 UA）、param_arr（参数数组）。

  2. 生成 bmak_str

通过 bmak_str = "0" + String(startTs + parseInt(param_arr[0])) 拼接得到。

1.  生成 time_ua 字符串
    

拼接 startTs 和 userAgent 的后 32 位，前后各加一个 "0"。

1.  计算 bigNumber1
    

调用 get_big_number(time_ua)，对 time_ua 每个字符做 hash 计算，得到 bigNumber1。

1.  将 bigNumber1 转为二进制字符串 bits_str
    

bits_str = bigNumber1.toString(2)

1.  生成 chars1 字符集
    

调用 get_chars(bits_str)，根据 bits_str 和预设字符表，按规则选取字符组成 chars1。

1.  生成 part1_str
    

用 bmak_str 的每一位作为索引，从 chars1 取字符拼接。

vmp 插桩主要讲究的就是反推+条件断点，这样就很容易找到算法的生成。

五、结尾部分
======

akamai 算法整体不算很复杂，主要是靠风控+tls。

tls 部分我是使用了`陈不不`的浏览器转发 tls 来测的并发，试了几个网站并发成功率还可以， 有需要也可以联系他。

akamai 网站简单的比较简单，难的风控又很高, 在测试多数网站后, 得出如下结论。

*   ua 不对应, 算法被风控 、ajr 算法更新都可能会引起 403
    
*   难点的航司网站，如果 akamai 校验不过, 在请求接口的时候会卡住
    
*   现在多数网站对流程校验很严格，一次生成的 abck 很多网站是过不去的。
    

网站并发就看运气了

  

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)