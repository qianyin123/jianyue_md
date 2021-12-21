> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GuVq3FJ-qyp2dAQFTTN_Mg)

### 1. ios包解密

fridadump-ios: 先运行app，在进行dump，不然会卡在0B/0S这里。

https://github.com/AloneMonkey/frida-ios-dump

dump完之后就如图所示。安装到iphone中，我是iphone7，

![图片](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5VmkGePwkRGibuwyOyaIsSmstxGhHK0Yaj4QCBOnK3kibeE24OeRVHBtR4cxWvrvqhic7mRoTufEQmw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

直接定位这两个框架，不知道用哪个的话，objection看使用了那个。

![图片](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5VmkGePwkRGibuwyOyaIsSmzc6YZZaYafwo39nETsIuOg1CPl6ibTxQQVJfibQzUSEsr0vgzX5Lnniag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

只使用了上面那个框架，将其中的二进制文件拖进ida去。

```
id __cdecl +[XYSSHandle reqAuthorityWithHeaders:request:bodyData:](XYSSHandle_meta *self, SEL a2, id a3, id a4, id a5)  
{  
  return sub_581C(a3, a4, a5);  
}
```

hook下这个函数就知道，就是主逻辑。字面意思就没必要杠了。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在581c中有这些用法，就是ios的语法。

  

```
`function get_func_name(module, name) {` `var func_addr = Module.findExportByName(module, name);` `console.log("base_addr: " + func_addr);` `if (Process.arch == 'arm')` `return func_addr.add(1);  //如果是32位地址+1` `else` `return func_addr;``}``function hookxhs() {` `var res;` `var func_addr = get_func_addr('XYSecureShield', 0x581c)` `console.log("func_addr", func_addr);` `Interceptor.attach(ptr(func_addr), {` `onEnter: function (args) {` `console.log("onEnter");` `res = args[0];` `console.log("param:" + args[0],args[1],args[2],args[3]);` `            console.log("arg1->",new ObjC.Object(args[0]));` `},` `onLeave: function (retval) {` `console.log("onLeave");` `console.log("param:" + res + " type:" + typeof res);` `}` `});``}`
```

fridahook一下看看参数。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

一些头和url的信息。

           console.log("arg1->",new ObjC.Object(args[0]));

这样打印retain。有的会闪退，自行测试下。

代码跟到底部有个5e48。

建议用trace工具先trace一遍流程，静态看太累了。

```
`__int64 __fastcall sub_54E8(__int64 byte_buffer)``{` `__int64 v1; // x19` `int *v2; // x0` `__int64 v3; // x20` `_QWORD *v4; // x22` `__int64 v5; // x0` `__int64 v6; // x20` `void *v7; // x21` `int v8; // w22` `__int64 v9; // x0` `__int64 v10; // x0` `__int64 v11; // x0` `__int64 v12; // x0` `__int64 v13; // x0` `__int64 v14; // x19` `__int16 v16; // [xsp+8h] [xbp-88h]` `char v17; // [xsp+Ah] [xbp-86h]` `_QWORD *v18; // [xsp+10h] [xbp-80h]` `__int64 v19; // [xsp+18h] [xbp-78h]` `void **v20; // [xsp+20h] [xbp-70h]` `__int64 v21; // [xsp+28h] [xbp-68h]` `__int64 (__fastcall *v22)(); // [xsp+30h] [xbp-60h]` `void *v23; // [xsp+38h] [xbp-58h]` `__int16 *v24; // [xsp+40h] [xbp-50h]` `unsigned int v25; // [xsp+4Ch] [xbp-44h]` `__int64 v26; // [xsp+50h] [xbp-40h]` `__int64 v27; // [xsp+58h] [xbp-38h]` `v1 = objc_retain(byte_buffer);` `v19 = 0LL;` `v16 = 0;` `v17 = 0;` `v18 = e32acff3_3394_42c6_a408_79837f6b1a6f(); // 从keychain里面读取main_hmac` `v2 = sub_7BAC();                              // 计算出keyhash 96字节` `v3 = v2;` `if ( !v2 )` `{` `sub_7404(2, &off_14768);``LABEL_7:` `v4 = v18;` `goto LABEL_8;` `}` `v4 = v18;` `if ( v18 && !v16 )` `{` `if ( choice_md5(v18, *v2, v2 + 1, 64) == 1 )`
```

你们看到的应该全是uuid的函数，我这里稍微改动了下函数名。

熟悉android的应该知道，这个算法和android大致一样，先从xy-ter-str读取一串base64的密文，在经过aes解密拿到md5check和hmackey的值。

sub_7BAC 就是计算这串密文的函数，跟进去，由于这个要初始化才能完成，或者你自行修改keychain。

我这里说一下思路。

![图片](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5VmkGePwkRGibuwyOyaIsSmtibzyhyqzRqaXuA0Cp8EgKn8WXdTKicjD1ibiam94XYqFn423OdT9Xu6yw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**这是keychain的图**

![图片](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5VmkGePwkRGibuwyOyaIsSm3uGYRlnJJg0rX7qGSNLLHzfMsl53qEUPP9RrqwxMYibCI8H0SZ90KsQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

进入5220：

![图片](https://mmbiz.qpic.cn/mmbiz_png/icBZkwGO7uH5VmkGePwkRGibuwyOyaIsSmVukVUDGYRgAwIRjVDheIjUIcBPyLPJxZTBmIbup42bhfZFiaZMJh3kg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

一看就是aes的初始化。128位。就是aes-128。再看看他的导出函数，完全是openssl风格。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

直接openssl-github：https://github.com/openssl/openssl

怕你们不理解aes：https://blog.csdn.net/qq_28205153/article/details/55798628

上面074cd开头的，类似秘钥扩展，因为只传递了deviceid的前16位。跟进去。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

里面还有一层。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

判断完轮数之后，进行xor，这里有修改源码。0xFF001123这种，

一下结合了一些t表进行加速处理。

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

chioce_aes(v10, v12, v11, &v15, &v18, 0);

之后在回到这里，进行正常的aes解密，aes/cbc/128。前面有个iv的复制。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```
`unsigned __int64 __fastcall 81968886_6b80_4522_a871_316b3d1926f5(unsigned int *a1, _DWORD *a2, _DWORD *a3)``{` `unsigned int v3; // w17` `unsigned int v4; // w3` `unsigned int v5; // w5` `unsigned int v6; // w0` `__int64 v7; // x10` `signed __int64 v8; // x11` `_DWORD *i; // x12` `unsigned __int64 v10; // x8` `unsigned __int64 v11; // x9` `unsigned int v12; // w19` `unsigned __int64 result; // x0` `unsigned __int64 v14; // x4` `unsigned __int64 v15; // x17` `_DWORD *v16; // x10` `int v17; // w13` `int v18; // w13` `v3 = bswap32(*a1) ^ *a3;` `v4 = bswap32(a1[1]) ^ a3[1];` `v5 = bswap32(a1[2]) ^ a3[2];` `v6 = bswap32(a1[3]) ^ a3[3];` `v7 = ((a3[60] >> 1) - 1);` `v8 = 8 * v7;` `for ( i = a3 + 6; ; i += 8 )` `{` `v10 = (dword_109F4[(v6 >> 16) & 0xFF] ^ dword_105F4[v3 >> 24]) ^ dword_10DF4[v5 >> 8] ^ dword_111F4[v4] ^ *(i - 2);` `v11 = (dword_109F4[(v3 >> 16) & 0xFF] ^ dword_105F4[v4 >> 24]) ^ dword_10DF4[v6 >> 8] ^ dword_111F4[v5] ^ *(i - 1);` `v12 = dword_105F4[v6 >> 24];` `result = (dword_109F4[(v4 >> 16) & 0xFF] ^ dword_105F4[v5 >> 24]) ^ dword_10DF4[v3 >> 8] ^ dword_111F4[v6] ^ *i;` `v14 = dword_109F4[(v5 >> 16) & 0xFF] ^ v12 ^ dword_10DF4[v4 >> 8] ^ dword_111F4[v3] ^ i[1];` `v15 = v10 >> 24;` `if ( !v7 )` `break;` `v3 = dword_109F4[(v14 >> 16) & 0xFF] ^ dword_105F4[v15] ^ dword_10DF4[result >> 8] ^ dword_111F4[v11] ^ i[2];` `v4 = dword_109F4[(v10 >> 16) & 0xFF] ^ dword_105F4[v11 >> 24] ^ dword_10DF4[v14 >> 8] ^ dword_111F4[result] ^ i[3];` `v5 = dword_109F4[(v11 >> 16) & 0xFF] ^ dword_105F4[result >> 24] ^ dword_10DF4[v10 >> 8] ^ dword_111F4[v14] ^ i[4];` `v6 = dword_109F4[(result >> 16) & 0xFF] ^ dword_105F4[v14 >> 24] ^ dword_10DF4[v11 >> 8] ^ dword_111F4[v10] ^ i[5];` `LODWORD(v7) = v7 - 1;` `}` `v16 = &a3[v8];` `v17 = byte_115F4[v11 >> 24];` `*a2 = bswap32((((byte_115F4[v15] << 24) & 0xFF00FFFF | (byte_115F4[(v14 >> 16) & 0xFF] << 16)) & 0xFFFF00FF | (byte_115F4[result >> 8] << 8) | byte_115F4[v11]) ^ v16[8]);` `a2[1] = bswap32((((v17 << 24) & 0xFF00FFFF | (byte_115F4[(v10 >> 16) & 0xFF] << 16)) & 0xFFFF00FF | (byte_115F4[v14 >> 8] << 8) | byte_115F4[result]) ^ v16[9]);` `v18 = byte_115F4[v14 >> 24];` `a2[2] = bswap32((((byte_115F4[result >> 24] << 24) & 0xFF00FFFF | (byte_115F4[(v11 >> 16) & 0xFF] << 16)) & 0xFFFF00FF | (byte_115F4[v10 >> 8] << 8) | byte_115F4[v14]) ^ v16[10]);` `a2[3] = bswap32((((v18 << 24) & 0xFF00FFFF | (byte_115F4[(result >> 16) & 0xFF] << 16)) & 0xFFFF00FF | (byte_115F4[v11 >> 8] << 8) | byte_115F4[v10]) ^ v16[11]);` `return result;``}`
```

这些看起来就特别熟悉了。

不多bb，直接扣代码，扣完之后对比一下hook的结果

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**hash对得上，这里我还省略了一点，这串hash解出来前16位，是为了选择md5的某一种算法，没错，md5也修改了**

只不过我看了一下，android有不同，ios2套算法完全一样，应该是开发偷懒了。下面会提到

继续跟进choice_md5(v18, *v2, v2 + 1, 64) == 1

v2 :就是刚刚提到了选择哪一个算法，v2+1开始就是这hmackey，64位。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

将这个值hook了之后，是走1，有时候也会走2，这个可以根据md5check那部分的值，第一个48就是1，49就是2.

跟进md5这个struct。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

signed **int64** fastcall sub_AF10(__int64 a1){  return md5_init(*(a1 + 16), &old_k);}

这里将k表复制进去，就是这个k表会更改。但是ios的一样，开发偷懒，再次石锤。

然后md5init，向量也进行了更改

```
xmmword_12900   DCB 0x76, 0x54, 0x32, 0x10, 0xFE, 0xDC, 0xBA, 0x98, 0x89  
__const:0000000000012900                                         ; DATA XREF: md5_init+3C↑r  
__const:0000000000012900                 DCB 0xAB, 0xCD, 0xEF, 1, 0x23, 0x45, 0x67
```

正好反过来了。

hook这个里的k。是正常值，android有部分不正常的。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在看其余的md5update-subaf20  md5final-subaf28这种函数，对一个常规的final和update做了修改，

也就是说魔改点至少2点：

运算和向量。这也是openssl的源码。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

先不扣代码，

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

看看特征，hamc_md5。

hookxhs_block_data_order这个东西。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

每一次的向量都会是上一次md5 update的值作为初始向量在进行运算。

而且md5transform的时候，顺序也更改了

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

xhs_md5_block_data_order 这种函数名，一看就是openssl，不多bb，直接github一波openssl源码。扣代码。

怕你们不会扣，附件附上md5部分代码，这是github参考：https://gist.github.com/HoLyVieR/11e464a91b290e33b38e

具体这一整串buffer怎么运算的，我就不说了，hook一下就知道了。看雪上也有很多

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里计算出32位的shield之后，

在8024里面进行封装buffer。

```
`std::__1::basic_string<char,std::__1::char_traits<char>,std::__1::allocator<char>>::basic_string(&v42, &xmmword_14F50);// 7200370 version` `v6 = dword_14F68;                             // 02 af fa ec 01 appid` `sub_4D90(qword_14F38, &v40);` `bcff0575(2, &v42, v6, v3, &v40, v5, v4, &v44);// append buffer v5=32位的shield` `if ( v41 & 0x80000000 )` `operator delete(v40);` `if ( v43 & 0x80000000 )` `operator delete(v42);` `v7 = v44;`
```

  

这里有一个flag是2，android的是1，在拼接buffer的时候，放在appid的前面，

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

对整体buffer进行封装，形成一个83位的byte

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

进入自定义的buffer xor。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

很熟悉了，进行表xor。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这部分比较简单，

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

最后这个点进行base64之后，xy拼接起来。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

还原成python

对得上了。

最后一张流程图：

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

总体的算法和android几乎一模一样，不过算法层没有很复杂的操作，像加壳，ollvm这些。这也是ios比android简单的原因。

但是ios的体积太大了，oc语言和swift逆向工具不太完善，很头疼。

请勿拿代码去做任何违法事情，不但任何责任，只进行逆向研究。

参考：https://bbs.pediy.com/thread-267748.htm

花了一周时间熟悉了下ios的流程，骨头爹给了极大的帮助。

下一篇：dy五神。

附件(暂时不放全部的，考虑到责任问题)：aes部分的代码，看雪上一大堆，自行扣一下。

放md5和xor部分。

https://github.com/10code15bugs/xhs_ios