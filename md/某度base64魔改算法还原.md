> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/soKf306lXhfMhz-JybHPkA)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

base64是常见的一个算法,但是在逆向某度app的时候发现一个魔改的算法,出于好奇心,来分析下。  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibaMM1XX5UqvEuGcOXrv8GnLZWUSOX4lNfm9DicHF1cU8lY2q0rm7icqt0A/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

  

  

  

  

 目录

  

  

⊙一.apk调用so

⊙二.unidbg主动调用及其trace指令

⊙三.流程分析

⊙四.代码还原

⊙五.总结

**一.apk调用so**

拿到这个样本的第一反应先用ida看了下,加密代码还挺多,思考了下先写个apk调用一下so,看是否能够被调用。

```
`package com.*du.util;``public class Base64Encoder {` `static {` `System.loadLibrary("base64encoder_v1_4");` `}` `public static  final native  byte[] nativeB64Decode(byte[] bArr);` `public static  final native  byte[] nativeB64Encode(byte[] bArr);``}``//测试百度的jni函数``String str = "abcdefghi";``byte[] sb = str.getBytes();``sb = nativeB64Encode(sb);``tv.setText(new String (sb));``Log.e(TAG, "主动调用后：" + new String (sb));`
```

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVM92caHcqBibjiaszfLicUm0paCwfwbs8H6uFcDzTZZ10EGyZ2B8cJMgicA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

发现确实可以被调用,那么就可以在自己的apk加载进行ida动态调试了。  

  

**二.unidbg主动调用及其trace指令**

当我打开ida的时候发现,调试指令时需要重新运行程序,索性再用unidbg试一下主动调用能否复现。

结果如下,因为本文要分析算法所以牵扯到具体工具操作的较少(我懒，前面的没记笔记)。  
  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVkIfComSMXAiaVQquYFn15OgZStO8NZP05LpTIkUlviauWJlWezWC1oFQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

在unidbg增加如下代码进行trace：

```
`String traceFile = "/Users/duzhuangchang/mystudy/unidbg-master/unidbg-android/src/test/java/com/bd/base64/encode.text";``PrintStream traceStream = new PrintStream(new FileOutputStream(traceFile), true);``emulator.traceCode(module.base, module.base+module.size).setRedirect(traceStream);`
```

  

我们已经得到trace的文件了,现在打开ida和unidbg进行调试,分析。

  

**三.流程分析  
**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVNnianDNgcsefO0RdFub7ia3tMCHPD5syTuWvaq6I1kndPAh2aTQjjUdg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

由于前面的逻辑比较清晰我们直接跳转到12行调用加密函数的那个地方

为了验证我们的判断是否正确 在0x6A76下断点。

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVOjZmmsuXakAnj49ia70fxhRW2hLOj3ia7U9oQCxS5MiaUJn7WEal0frOw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

在断点之前验证我们猜想

参数1:  

```
`0000: 61 62 63 64 65 66 67 68 69 00 00 00 00 00 00 00    abcdefghi.......``0010: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................`
```

参数2:9确实是字符串的长度。

```
0x40006a4e: "mov r4, r0" r0=0x9 => r4=0x9
```

  

进入j_GC20函数，根据上方传入的参数，我们可以人为更改函数的参数为我们可以识别的名称

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVpmG50NriaR3jZQzEZI8f3BiaYranpDpya0a442ibl2txibib9lBPzo4J25w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下逐个分析j_B6413,j_B6419,sub_11458函数  

  

j_B6413,j_B6419这两个函数都传入了v24,我们可以看j_B6413执行前后对于v24的影响

之前：

```
`0000: 71 33 10 40 20 A9 13 40 30 20 1C 40 01 00 00 00    q3.@ ..@0 .@....``0010: 30 20 1C 40 00 00 1D 40 00 20 1C 40 08 00 1D 40    0 .@...@. .@...@``0020: 10 00 00 00 94 E5 12 40 01 00 00 00 40 01 18 40    .......@....@..@``0030: DF 45 11 40 00 00 00 00 00 00 00 00 01 00 00 00    .E.@............``0040: ED F0 10 40 01 00 00 00 00 00 00 00 10 00 00 00    ...@............```0050: E5 35 0D 40 14 2F 13 40 E5 35 0D 40 60 FC FF BF    .5.@./.@.5.@`...```0060: 02 00 00 00 FF FF FF FF 0C 2F 13 40 00 00 00 00    ........./.@....`
```

之后：  

```
`0000: 24 23 38 2D 20 A9 13 40 30 20 1C 40 01 00 00 00    $#8- ..@0 .@....``0010: 30 20 1C 40 00 00 1D 40 00 20 1C 40 08 00 1D 40    0 .@...@. .@...@``0020: 10 00 00 00 94 E5 12 40 01 00 00 00 40 01 18 40    .......@....@..@``0030: DF 45 11 40 00 00 00 00 00 00 00 00 01 00 00 00    .E.@............``0040: ED F0 10 40 01 00 00 00 00 00 00 00 10 00 00 00    ...@............```0050: E5 35 0D 40 14 2F 13 40 E5 35 0D 40 60 FC FF BF    .5.@./.@.5.@`...```0060: 02 00 00 00 FF FF FF FF 0C 2F 13 40 00 00 00 00    ........./.@....`
```

一对比71 33 10 40 改变为 24 23 38 2D

**j_B6413：**

然后进入j_B6413来看  

  

```
`int *__fastcall B6413(int *result, unsigned int a2, int a3)``{` `unsigned int v3; // lr` `char *table_start; // r12` `int v5; // r3` `char *table_end; // r4` `char *v7; // r5` `char v8; // t1` `char *v9; // r3` `int v10; // r4` `signed int i; // r1` `int v12; // r2` `v3 = a2 & 0x3F;` `table_start = B6417;                          // 码表的开始和结尾` `v5 = -v3;` `table_end = B6417 + 63;` `while ( v5 != -64 )                           // 进行64次操作` `{` `v7 = (char *)result + v5;` `v8 = *table_end--;` `--v5;` `v7[0xC3] = v8;` `}` `v9 = (char *)result + 0xC3;` `while ( v3 )` `{` `v10 = (int)&table_start[v3--];` `*v9-- = *(_BYTE *)(v10 - 1);` `}` `*result = (0x2D382324 << ((a2 >> 5) & 0xF)) ^ __ROR4__(a2, 5);` `if ( a3 )` `{` `for ( i = 131; i != 3; --i )` `*((_BYTE *)result + i) = 64;` `v12 = 0;` `do` `{` `*((_BYTE *)result + *((unsigned __int8 *)result + v12 + 132) + 4) = v12;` `++v12;` `}` `while ( v12 != 64 );` `}` `return result;``}`
```

  

  

在代码的这部分存在一个64字节的码表，猜测这个就是base64加密的码表

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVXCibianiapsPEWuazFTYvUdKuDaqWdgDWEZ1cZvBAaDjowMO0hseH5dRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAV3Aiab1ibWE8g7x81ibyrogNyicBRaKnOjfhaFP7WQ6bJuxicYjyKS9N3Cdg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

table = “qogjOuCRNkfil5p4SQ3LAmxGKZTdesvB6z_YPahMI9t80rJyHW1DEwFbc7nUVX2-”

下方的R5其实就是v7

根据下方trace的指令  

```
`0x40006c58: "adds r5, r0, r3" r0=0xbffff624 r3=0x0 => r5=0xbffff624``0x40006c58: "adds r5, r0, r3" r0=0xbffff624 r3=0xffffffff => r5=0xbffff623``0x40006c58: "adds r5, r0, r3" r0=0x bffff624 r3=0xfffffffe => r5=0xbffff622``0x40006c58: "adds r5, r0, r3" r0=0xbffff624 r3=0xfffffffd => r5=0xbffff621``0x40006c58: "adds r5, r0, r3" r0=0xbffff624 r3=0xfffffffc => r5=0xbffff620`
```

  

下方的赋值就是把R6 赋值给R5+0xC3的内存单元

```
`0x40006c60: "strb.w r6, [r5, #0xc3]" r6=0x2d r5=0xbffff624``0x40006c60: "strb.w r6, [r5, #0xc3]" r6=0x32 r5=0xbffff623``0x40006c60: "strb.w r6, [r5, #0xc3]" r6=0x58 r5=0xbffff622``0x40006c60: "strb.w r6, [r5, #0xc3]" r6=0x56 r5=0xbffff621``0x40006c60: "strb.w r6, [r5, #0xc3]" r6=0x55 r5=0xbffff620`
```

  

观察上方的R6 从上到下ascii码为：0x2d=-，0x32=2 ，0x58=X 等等，他们都被加载到  从 0xbffff624+0xC3  至 0xbffff624-0x40-1+0xC3=0xBFFFF6A8逐渐填充,至于为啥百度这么做，不晓得

  

  

我们去debugger去验证下

```
`m0xBFFFF6A7``size: 112``0000: 71 6F 67 6A 4F 75 43 52 4E 6B 66 69 6C 35 70 34    qogjOuCRNkfil5p4``0010: 53 51 33 4C 41 6D 78 47 4B 5A 54 64 65 73 76 42    SQ3LAmxGKZTdesvB``0020: 36 7A 5F 59 50 61 68 4D 49 39 74 38 30 72 4A 79    6z_YPahMI9t80rJy``0030: 48 57 31 44 45 77 46 62 63 37 6E 55 56 58 32 2D    HW1DEwFbc7nUVX2-``0040: 00 00 00 00 86 BE 79 7A 00 00 00 00 00 00 00 00    ......yz........``0050: 00 00 00 00 09 00 00 00 A0 12 FE FF 00 20 1D 40    ............. .@``0060: 28 F7 FF BF 7B 6A 00 40 00 20 1D 40 00 00 00 00    (...{j.@. .@....``^-----------------------------------------------------------------------------^`
```

  

  

对j_B6413 的功能的总结：一个是对j_B6413 的参数的值替换为了 0x2D382324,另一个功能是把码表塞入了0xBFFFF6A8 开始的64个字节

  

_B6419的功能：

  

  

  

根据trace的代码  

```
`第一次``0x40006ce4: "cmp r3, #0" r3=0x2 => cpsr: N=0, Z=0, C=1, V=0``0x40006ce6: "bne #0x40006cda"``0x40006cda: "ldr r4, [r0]" r0=0x401d2000 => r4=0x64636261``0x40006cdc: "subs r3, #1" r3=0x2 => r3=0x1 ；--v6;``0x40006cde: "eor.w r4, lr, r4, ror #3" lr=0x2d382324 r4=0x64636261 => r4=0x1b44f68；*v5 = v3 ^ __ROR4__(*v5, 3);``0x40006ce2: "stm r0!, {r4}" r0=0x401d2000 r4=0x1b44f68 => r0=0x401d2004``第二次``0x40006ce4: "cmp r3, #0" r3=0x1 => cpsr: N=0, Z=0, C=1, V=0``0x40006ce6: "bne #0x40006cda"``0x40006cda: "ldr r4, [r0]" r0=0x401d2004 => r4=0x68676665``0x40006cdc: "subs r3, #1" r3=0x1 => r3=0x0``0x40006cde: "eor.w r4, lr, r4, ror #3" lr=0x2d382324 r4=0x68676665 => r4=0x8034cfe8``0x40006ce2: "stm r0!, {r4}" r0=0x401d2004 r4=0x8034cfe8 => r0=0x401d2008`
```

  

对原始输入的buff，因为本例中使用的input为abcdefghi

相当于把abcd ==>替换为 0x1b44f68

相当于把efgh==>替换为0x8034cfe8

  

```
`0x40006cea: "cmp.w ip, r0, lsr #2" ip=0x2 r0=0xc => cpsr: N=1, Z=0, C=0, V=0``0x40006cee: "it hs"``0x40006cf2: "ldr.w r0, [r1, ip, lsl #2]" r1=0x401d2000 ip=0x2 => r0=0x69``0x40006cf6: "eor.w r0, r0, lr" r0=0x69 lr=0x2d382324 => r0=0x2d38234d``0x40006cfa: "str.w r0, [r1, ip, lsl #2]" r0=0x2d38234d r1=0x401d2000 ip=0x2`
```

hi==>替换为0x2d38234d

  

我们debugger验证下

```
`m0x401d2000``>-----------------------------------------------------------------------------<``[00:35:50 573]RW@0x401d2000, md5=3fcf32469aaaf4c276fcfd33f704efdf, hex=684fb401e8cf34804d23382d00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000``size: 112``0000: 68 4F B4 01 E8 CF 34 80 4D 23 38 2D 00 00 00 00    hO....4.M#8-....``0010: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................``0020: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................`
```

  

对j_B6419的总结：就是对原始的input值进行变换操作，有两种形式，第一种形式是按照4字节正数倍的进行

0x2d382324 ^ ROR4(第n个4的整数倍的数据, 3); ，第二种形式就是将剩下的不足4的字节直接进行：0x2d382324 ^不足4个的字节操作

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVDdYS3KrMH5fDLseWxho7nXHqAgOoa5fiafWT9KbFMZJAw8VjBsMMz5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

sub_11458的功能：

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVcdDlqRliaBEpXf8h0RVvqicyxeJwJeaLWia2uFxFIcHbDJLstG1rdfKiaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVRsIaajoLEFDmicZddG9N6nstuwYNbJWKny4f0CVuT4Biam2vhlhobDkw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

有一些运算,都比较清晰,但是不知道他是干什么的！在逆向我们应该紧紧的关注于，对原始输入的处理先放过去

  

  

然后接着往下看

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVElDGJo3UQZIR6XHicVQC5AQH2EO41wqwbT5YVJ05DWVqomz0KY2LTFA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

j_B6414

首先看这个函数的输入 第一个参数我们知道是老演员v24了,第二个第三个参数我们尝试打印下

可以验证我们上方的注释是正确的

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVl2PM1DFSG6Nh6LvLwBDIuzLiblPicouKp2lvghoIiapYplse9QJqREjGA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVvO6DG4CFPLJnw4aWEdibrElVibpiaIeB40QgdCy6IkdYxACPeXXboUKFg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

```
`int __fastcall B6414(int v24, unsigned __int8 *arg_two, int *arg_three)``{` `int v3; // r0` `int result; // r0` `v3 = v24 + 0x84;` `result = *(unsigned __int8 *)(v3 + (*arg_two & 0x3F)) | (*(unsigned __int8 *)(v3 + (arg_two[1] & 0x3F)) << 8) | (*(unsigned __int8 *)(v3 + (arg_two[2] & 0x3F)) << 16) | (*(unsigned __int8 *)(v3 + (((unsigned int)*arg_two >> 2) & 0x30 | ((unsigned int)arg_two[1] >> 4) & 0xC | ((unsigned int)arg_two[2] >> 6))) << 24);` `*arg_three = result;` `return result;``}`
```

  

这个函数体内明显就是一些移位操作了,看下 v6= v24+0x84 ,在前文中我们提到对老演员v24进行的一波填充base64编码表的操作, 我们还记得前面是0xC3 逐渐减1进行填充的， 0x84+0x39 =0xC3，也就是说v3当前指针的值就是指的asci码也就是“qogjOuCRNkfil5p4SQ3LAmxGKZTdesvB6z_YPahMI9t80rJyHW1DEwFbc7nUVX2-”第一个字符，上面的也就解释通了。

  

我们对这个函数进行trace 内容下方：

```
`0x40006d1a: "add r7, sp, #0xc" sp=0xbffff604 => r7=0xbffff610``0x40006d1c: "str fp, [sp, #-0x4]!" fp=0xc sp=0xbffff604 => sp=0xbffff600``#ip获得第二个参数的0-7``0x40006d20: "ldrb.w ip, [r1]" r1=0x401d2006 => ip=0x34``0x40006d24: "movs r4, #0xc" => r4=0xc``#r3获得第二个参数的8-15位``0x40006d26: "ldrb r3, [r1, #1]" r1=0x401d2006 => r3=0x80``#R0是老演员v24的指针地址 +#0x84正好找到base64第一个偏移的位置``0x40006d28: "adds r0, #0x84" r0=0xbffff624 => r0=0xbffff6a8``#r3获得第二个参数的16-23位``0x40006d2a: "ldrb.w lr, [r1, #2]" r1=0x401d2006 => lr=0x4d` `# ( * arg_two >> 2) & 0x30``0x40006d2e: "movs r1, #0x30" => r1=0x30``0x40006d30: "and.w r6, r1, ip, lsr #2" r1=0x30 ip=0x34 => r6=0x0``#(arg_two[0] & 0x3F)``0x40006d34: "and r1, ip, #0x3f" ip=0x34 => r1=0x34``（arg_two[1] >> 4) & 0xC``0x40006d38: "and.w r4, r4, r3, lsr #4" r4=0xc r3=0x80 => r4=0x8``(arg_two[1] & 0x3F)) << 8)``0x40006d3c: "and r3, r3, #0x3f" r3=0x80 => r3=0x0``# arg_two[2] & 0x3F``0x40006d40: "and r5, lr, #0x3f" lr=0x4d => r5=0xd``( * arg_two >> 2) & 0x30  ｜  （arg_two[1] >> 4) & 0xC``0x40006d44: "orrs r6, r4" r6=0x0 r4=0x8 => r6=0x8``#(v3 + (arg_two[1] & 0x3F)))  == 0x71``0x40006d46: "ldrb r3, [r0, r3]" r0=0xbffff6a8 r3=0x0 => r3=0x71` `((unsigned int)arg_two[2] >> 6)))``0x40006d48: "orr.w r6, r6, lr, lsr #6" r6=0x8 lr=0x4d => r6=0x9``(v3 + (*arg_two & 0x3F))``0x40006d4c: "ldrb r1, [r0, r1]" r0=0xbffff6a8 r1=0x34 => r1=0x45``v3 + (arg_two[2] & 0x3F))``0x40006d4e: "ldrb r5, [r0, r5]" r0=0xbffff6a8 r5=0xd => r5=0x35``(v3 + ((unsigned int) * arg_two >> 2) & 0x30 | ((unsigned int)arg_two[1] >> 4)` `0x40006d50: "ldrb r0, [r0, r6]" r0=0xbffff6a8 r6=0x9 => r0=0x6b``(v3 + (*arg_two & 0x3F)) |(v3 + (arg_two[1] & 0x3F)) << 8)     ==0x7145` `0x40006d52: "orr.w r1, r1, r3, lsl #8" r1=0x45 r3=0x71 => r1=0x7145``(v3 + (*arg_two & 0x3F)) |(v3 + (arg_two[1] & 0x3F)) << 8)  ｜(v3 + (arg_two[2] & 0x3F)) << 16)` `0x40006d56: "orr.w r1, r1, r5, lsl #16" r1=0x7145 r5=0x35 => r1=0x357145``(v3 + (` `((unsigned int) * arg_two >> 2) & 0x30 | ((unsigned int)arg_two[1] >> 4) & 0xC | ((unsigned int)` `arg_two[2] >> 6))) << 24)``0x40006d5a: "orr.w r0, r1, r0, lsl #24" r1=0x357145 r0=0x6b => r0=0x6b357145``0x40006d5e: "str r0, [r2]" r0=0x6b357145 r2=0x401d2008``0x40006d60: "ldr fp, [sp], #4" sp=0xbffff600 => fp=0xc sp=0xbffff604``0x40006d64: "pop {r4, r5, r6, r7, pc}" sp=0xbffff604`
```

  

  

  

已知我们输入的值

abcd efgh  i

经过变换后为

｜1234 ｜5678    ｜  一二三四

你会发现这次处理的是 78一，然后把 处理完的重新存放在一二三四的位置上，

由下图可知v20每次执行-3，因为是char* 指针 ，当每次减1的时候变化为1

  

第二次处理的456 然后把处理的完成后的重新存放在5678上

第三次处理的是123，处理完成之后把处理完成的存放在1234上面 ，通过这样的推理 ，我们知道 sub_11458 就是求出来这个input最后的结尾，猜测成立。

  

  

其实我们看传入的参数第二个参数和第三个参数的时候,我们发现我们原始的input长度就是8,第二个参数正好是经过转换后第七个input中的地址，第三个参数正好是经过转换后的12个值的第九个值的参数,又根据函数接下来的处理,确实印证了我们的想法。

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAValPGzNLsRVfWJkfG2gb5p3jv3tG41SRRcwd5aCasMC1wFpp4YicfibwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

经过验证确实正确，接下来还原算法。

  

**四.代码还原**

  

本次代码还原部分,由于ida生成的伪代码具有迷惑性,几乎逻辑部分完全看汇编trace得到的,花费了大约三天的时间,下方代码的只处理了输入为3字节倍数的部分(保护下百度),但是基本的函数都已经完全还原。

  

  

  

```
`#include <stdio.h>``#include <math.h>``#include <hexdump.h>``int ROR4(int value, int count) {` `count = count % (int) (pow(2, 32));` `value = value % (int) pow(2, 32);` `return (value << (32 - count) | (value >> count)) % (int) pow(2, 32);``}``unsigned int count_leading_zeros(unsigned int num) {` `unsigned int cnt = 0;` `while (!(num & 0x80000000) && cnt < 32) {` `cnt++;` `num <<= 1;` `}` `return cnt;``}``void B6419(char *inputbuff, int inputlen) {` `int v3 = 0x2d382324;` `int v4 = inputlen >> 2;` `int *v5 = (int *) inputbuff;` `int v6 = inputlen >> 2;` `while (v6) {` `v6--;` `printf("0x%x\n", v3 ^ ROR4(*v5, 3));` `*v5 = v3 ^ ROR4(*v5, 3);` `++v5;` `}` `if (v4 < (inputlen + 3) >> 2) {` `*v5 = *v5 ^ v3;` `}``}``unsigned int calc_len(unsigned int input_len) {` `char v2;` `unsigned int v3;` `unsigned int v4;` `unsigned int v5;` `bool v6;` `unsigned int static_3 = 3u;` `if (input_len <= static_3)              // 当输入的长度小于等于三的时候才会被执行` `{` `input_len = input_len == static_3;` `} else if (static_3 & (static_3 - 1)) {` `v2 = count_leading_zeros(static_3) -` `count_leading_zeros(input_len);// 计算前导零指令：（把他们转为二进制 计算零的个数）static_3-iput_len；0x1e-0x1c = 0x2` `v3 = static_3 << v2;                    // v2*8；12 = 0xC` `v4 = 1 << v2;                           // v2*2 ；4` `v5 = 0;` `while (1) {` `if (input_len >= v3)                // 9>12；false` `{` `input_len -= v3;` `v5 |= v4;` `}` `if (input_len >= v3 >> 1)           // 9>= 12/2=6;true ;只要这一项满足   后面的都会满足` `{` `input_len -= v3 >> 1;` `v5 |= v4 >> 1;` `}` `if (input_len >= v3 >> 2) {` `input_len -= v3 >> 2;` `v5 |= v4 >> 2;` `}` `if (input_len >= v3 >> 3) {` `input_len -= v3 >> 3;` `v5 |= v4 >> 3;` `}` `v6 = input_len == 0;` `if (input_len) {` `v4 >>= 4;` `v6 = v4 == 0;` `}` `if (v6)` `break;` `v3 >>= 4;` `}` `input_len = v5;` `} else {` `input_len >>= 31 - count_leading_zeros(static_3);` `}` `return input_len;``}``void j_B6414(char *waitdata, char *resultbuff) {` `char table[64] = "qogjOuCRNkfil5p4SQ3LAmxGKZTdesvB6z_YPahMI9t80rJyHW1DEwFbc7nUVX2-";` `unsigned result = (*(table + (*waitdata & 0x3F)));` `unsigned result1 = (*(table + ((waitdata[1] & 0x3F)))) << 8;` `unsigned int result2 = (*(table + ((waitdata[2] & 0x3F)))) << 16;` `unsigned int result_end1 = result | result1;` `unsigned int result_end2 = result_end1 | result2;` `unsigned int rs3_2 = (((*waitdata) >> 2) & 0x30) | ((waitdata[1] >> 4) & 0xc);//r6=0xc` `    int rs3_3 = ((waitdata[2]) & 0xff) >> 6;` `int rs4_1 = rs3_2 | rs3_3;` `int rs5_1 = *(table + rs4_1) << 24;` `int result4 = result_end2 | rs5_1;` `printf("calc=>0x%x\n", result4);` `//因为参数上的是char* 步长不满足要求 要转换下 再存储` `int *lsresultbuff = (int *) resultbuff;` `*lsresultbuff = result4;``}``int main() {` `char inputbuff[100] = "abcdefghi";` `int inputlen = 9;` `B6419(inputbuff, inputlen);` `unsigned int v7 = calc_len(inputlen);` `int v8 = 3 * v7;` `unsigned v9 = v7;` `int v10 = inputlen - v8;` `int v11 = 4 * v7;` `int v21 = v10;` `int v16;` `if (inputlen == 3 * v7)  //输入的字符是否为三的倍数的情况下` `{` `v16 = 4 * v7;` `} else {` `v16 = 0;` `printf("zijishixian");` `return 0;` `}` `char *v17 = inputbuff + v11 - 4;` `char *v18 = inputbuff + v8;                     // v18=input_buff+0x9` `int v19 = -v9;                                  // v19=-v9=-0x3` `char *v20 = v18 - 3;` `while (v19) {` `j_B6414(v20, v17);` `hexdump(v20, 10);` `hexdump(v17, 10);// 第二次对input_buff中y有选择的加密` `v20 -= 3;` `v17 -= 4;` `hexdump(v20, 10);` `hexdump(v17, 10);` `++v19;` `};` `*(inputbuff + v16) = v21 + 65;` `*(inputbuff + v16 + 1) = 0;` `hexdump(inputbuff, 100);` `return 0;``}`
```

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5YHVYUwZib1PFk3wkOn05EgAgS0H2rAVE7ibbBLRmITZ3Rh0QABTv2HDEhub4NXNcgibyIobdOudtCu5umgsUdcA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

五.总结

  

百度的这个base64加密比常规的魔改要复杂很多,但是缺陷就是在so层可以随意被访问,如果稍微加一些混淆那么难度又会上升一个地步。

  

  

  

我是BestToYou,分享工作或日常学习中关于二进制逆向和分析的一些思路和一些自己闲暇时刻调试的一些程序,文中若有错误的地方,恳请大家联系我批评指正。  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/p5YHVYUwZib3B6WPiavosZFcyzR8y0A5ibalicqxrfTvYLw2zBMWqnyUFTG4vtoJpcjmckN4BNIusIIr8bU57ucmEA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)