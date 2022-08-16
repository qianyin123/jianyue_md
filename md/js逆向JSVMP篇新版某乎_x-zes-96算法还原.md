> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/d8FYaAp4tfHdR8c2YwypJA)

****提示！本文章仅供学习交流，严禁用于任何商业和非法用途，如有侵权，可联系本文作者删除！****  

* * *

### **前言**

    这两天有人在[jsvmp-某乎_x-zes-96参数算法还原（手把手教学）](https://mp.weixin.qq.com/s?__biz=MzIxNjQ2NzM3MA==&mid=2247485119&idx=1&sn=297c7b85cbcbb19897f5d89ceada680c&scene=21#wechat_redirect)CSDN上的发布文章下说跟某乎的加密不一样，然后就去看了下，发现某乎更新了加密算法，具体啥时候更新的就不太清楚了，之后发现通过**jsdom**来补环境的方式已经行不通了，同时抠出的加密函数只有在知乎页面生成的加密参数才有效，因为暂时不清楚要补哪些环境，所以补环境到方案就先放一放，本文呢主要给大家介绍两种方式，一种是比较简单的方式通过**jsrpc**获取加密参数，另一种就是本文的重点，直接还原出它的算法。

* * *

**网站链接**

aHR0cHM6Ly93d3cuemhpaHUuY29tL3NlYXJjaD9xPXB5dGhvbiZ0eXBlPWNvbnRlbnQ=

* * *

#### **jsrpc方案获取加密参数**

    如何定位加密参数，又如何抠加密函数就不说了，可以看[jsvmp-某乎_x-zes-96参数算法还原（手把手教学）](https://mp.weixin.qq.com/s?__biz=MzIxNjQ2NzM3MA==&mid=2247485119&idx=1&sn=297c7b85cbcbb19897f5d89ceada680c&scene=21#wechat_redirect)这篇文章，首先把加密函数抠出来，然后测试一下抠出的函数是否有用，在前言中也有提到**jsdom**已经行不通了，所以直接将抠好的代码放到浏览器中运行，上一篇还原文章中说到最终是把一个**md5**值进行加密，所以这里抠完代码之后也直接拿**md5**值进行测试，这里说一下测试的时候注意点：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGo4lU3DHZsqh8uic7PAFuVib6q1Hx4QYoaOJUkPPeFUPMmiaZak2g6vibVBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    如上图，要注意点就是**params、cookie**这两个参数一定要和生成**md5**值之前传的参数一致，还有要注意的是**headers**中是否有传**x-zst-81**值，我这里因为是举例找的是初始的**search_v3**请求，所以没有携带**x-zst-81**值，大家测试时候自己注意下，否则即使最后参数生成的是正确的也无法拿到数据的，之后来看看测试结果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoFpyyaZ0al0ENpstUrpNpnPbSuHNRkbq9w4L9G10SpPLA42La0SH6QA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里先拿到**md5**值，之后生成加密参数：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoEnibVvp8fCFRG1fcdOqgDqiaLETPlWGU5SOxUAuGdicyLQ3ShUFqbFYBA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoIyqZ8jlhECTynSUXSYU0GiamvBAcqQ7rslwF5F57bX6J2LNR9T0CIQw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    上图可以看到成功拿到数据，所以证明抠出来的加密函数是正确的，接下来就是完善一下代码，将代码完善成如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGo6NiaCfSsmz94CPErvXAV4XXzLOpjQFF7qgN522Pq7IQLpm7QzCYic0lQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    这样的话就只需要传**params、cookie、x-zst-81**就好了，而**get_x_zse_96**这个函数就是抠的下图中的：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGogicCuoj9X61vONxdy9DIBHENAH4WLHUtkIroicBFHo3LAK9Fuo8SCnNA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    抠完之后将上图中划线的部分改成自己抠出来的加密函数就好了，因为是在浏览器中运行的，会发现浏览器中没有**md5**这个方法，不过这里直接给大家简单封装了一下，放到代码中直接使用就好了：

```


 function md5(val) {  
     var hexcase = 0;  /* hex output format. 0 - lowercase; 1 - uppercase        */  
     var chrsz = 8;  /* bits per input character. 8 - ASCII; 16 - Unicode      */  
   
     function hex_md5(s) {  
         return binl2hex(core_md5(str2binl(s), s.length * chrsz));  
     }  
   
     /*  
      * Convert an array of little-endian words to a hex string.  
      */  
     function binl2hex(binarray) {  
         var hex_tab = hexcase ? "0123456789ABCDEF" : "0123456789abcdef";  
         var str = "";  
         for (var i = 0; i < binarray.length * 4; i++) {  
             str += hex_tab.charAt((binarray[i >> 2] >> ((i % 4) * 8 + 4)) & 0xF) +  
                 hex_tab.charAt((binarray[i >> 2] >> ((i % 4) * 8)) & 0xF);  
         }  
         return str;  
     }  
   
     function core_md5(x, len) {  
         /* append padding */  
         x[len >> 5] |= 0x80 << ((len) % 32);  
         x[(((len + 64) >>> 9) << 4) + 14] = len;  
   
         var a = 1732584193;  
         var b = -271733879;  
         var c = -1732584194;  
         var d = 271733878;  
   
         for (var i = 0; i < x.length; i += 16) {  
             var olda = a;  
             var oldb = b;  
             var oldc = c;  
             var oldd = d;  
   
             a = md5_ff(a, b, c, d, x[i + 0], 7, -680876936);  
             d = md5_ff(d, a, b, c, x[i + 1], 12, -389564586);  
             c = md5_ff(c, d, a, b, x[i + 2], 17, 606105819);  
             b = md5_ff(b, c, d, a, x[i + 3], 22, -1044525330);  
             a = md5_ff(a, b, c, d, x[i + 4], 7, -176418897);  
             d = md5_ff(d, a, b, c, x[i + 5], 12, 1200080426);  
             c = md5_ff(c, d, a, b, x[i + 6], 17, -1473231341);  
             b = md5_ff(b, c, d, a, x[i + 7], 22, -45705983);  
             a = md5_ff(a, b, c, d, x[i + 8], 7, 1770035416);  
             d = md5_ff(d, a, b, c, x[i + 9], 12, -1958414417);  
             c = md5_ff(c, d, a, b, x[i + 10], 17, -42063);  
             b = md5_ff(b, c, d, a, x[i + 11], 22, -1990404162);  
             a = md5_ff(a, b, c, d, x[i + 12], 7, 1804603682);  
             d = md5_ff(d, a, b, c, x[i + 13], 12, -40341101);  
             c = md5_ff(c, d, a, b, x[i + 14], 17, -1502002290);  
             b = md5_ff(b, c, d, a, x[i + 15], 22, 1236535329);  
   
             a = md5_gg(a, b, c, d, x[i + 1], 5, -165796510);  
             d = md5_gg(d, a, b, c, x[i + 6], 9, -1069501632);  
             c = md5_gg(c, d, a, b, x[i + 11], 14, 643717713);  
             b = md5_gg(b, c, d, a, x[i + 0], 20, -373897302);  
             a = md5_gg(a, b, c, d, x[i + 5], 5, -701558691);  
             d = md5_gg(d, a, b, c, x[i + 10], 9, 38016083);  
             c = md5_gg(c, d, a, b, x[i + 15], 14, -660478335);  
             b = md5_gg(b, c, d, a, x[i + 4], 20, -405537848);  
             a = md5_gg(a, b, c, d, x[i + 9], 5, 568446438);  
             d = md5_gg(d, a, b, c, x[i + 14], 9, -1019803690);  
             c = md5_gg(c, d, a, b, x[i + 3], 14, -187363961);  
             b = md5_gg(b, c, d, a, x[i + 8], 20, 1163531501);  
             a = md5_gg(a, b, c, d, x[i + 13], 5, -1444681467);  
             d = md5_gg(d, a, b, c, x[i + 2], 9, -51403784);  
             c = md5_gg(c, d, a, b, x[i + 7], 14, 1735328473);  
             b = md5_gg(b, c, d, a, x[i + 12], 20, -1926607734);  
   
             a = md5_hh(a, b, c, d, x[i + 5], 4, -378558);  
             d = md5_hh(d, a, b, c, x[i + 8], 11, -2022574463);  
             c = md5_hh(c, d, a, b, x[i + 11], 16, 1839030562);  
             b = md5_hh(b, c, d, a, x[i + 14], 23, -35309556);  
             a = md5_hh(a, b, c, d, x[i + 1], 4, -1530992060);  
             d = md5_hh(d, a, b, c, x[i + 4], 11, 1272893353);  
             c = md5_hh(c, d, a, b, x[i + 7], 16, -155497632);  
             b = md5_hh(b, c, d, a, x[i + 10], 23, -1094730640);  
             a = md5_hh(a, b, c, d, x[i + 13], 4, 681279174);  
             d = md5_hh(d, a, b, c, x[i + 0], 11, -358537222);  
             c = md5_hh(c, d, a, b, x[i + 3], 16, -722521979);  
             b = md5_hh(b, c, d, a, x[i + 6], 23, 76029189);  
             a = md5_hh(a, b, c, d, x[i + 9], 4, -640364487);  
             d = md5_hh(d, a, b, c, x[i + 12], 11, -421815835);  
             c = md5_hh(c, d, a, b, x[i + 15], 16, 530742520);  
             b = md5_hh(b, c, d, a, x[i + 2], 23, -995338651);  
   
             a = md5_ii(a, b, c, d, x[i + 0], 6, -198630844);  
             d = md5_ii(d, a, b, c, x[i + 7], 10, 1126891415);  
             c = md5_ii(c, d, a, b, x[i + 14], 15, -1416354905);  
             b = md5_ii(b, c, d, a, x[i + 5], 21, -57434055);  
             a = md5_ii(a, b, c, d, x[i + 12], 6, 1700485571);  
             d = md5_ii(d, a, b, c, x[i + 3], 10, -1894986606);  
             c = md5_ii(c, d, a, b, x[i + 10], 15, -1051523);  
             b = md5_ii(b, c, d, a, x[i + 1], 21, -2054922799);  
             a = md5_ii(a, b, c, d, x[i + 8], 6, 1873313359);  
             d = md5_ii(d, a, b, c, x[i + 15], 10, -30611744);  
             c = md5_ii(c, d, a, b, x[i + 6], 15, -1560198380);  
             b = md5_ii(b, c, d, a, x[i + 13], 21, 1309151649);  
             a = md5_ii(a, b, c, d, x[i + 4], 6, -145523070);  
             d = md5_ii(d, a, b, c, x[i + 11], 10, -1120210379);  
             c = md5_ii(c, d, a, b, x[i + 2], 15, 718787259);  
             b = md5_ii(b, c, d, a, x[i + 9], 21, -343485551);  
   
             a = safe_add(a, olda);  
             b = safe_add(b, oldb);  
             c = safe_add(c, oldc);  
             d = safe_add(d, oldd);  
         }  
         return Array(a, b, c, d);  
   
     }  
   
     /*  
      * These functions implement the four basic operations the algorithm uses.  
      */  
     function md5_cmn(q, a, b, x, s, t) {  
         return safe_add(bit_rol(safe_add(safe_add(a, q), safe_add(x, t)), s), b);  
     }  
   
     function bit_rol(num, cnt) {  
         return (num << cnt) | (num >>> (32 - cnt));  
     }  
   
     function md5_ff(a, b, c, d, x, s, t) {  
         return md5_cmn((b & c) | ((~b) & d), a, b, x, s, t);  
     }  
   
     function md5_gg(a, b, c, d, x, s, t) {  
         return md5_cmn((b & d) | (c & (~d)), a, b, x, s, t);  
     }  
   
     function md5_hh(a, b, c, d, x, s, t) {  
         return md5_cmn(b ^ c ^ d, a, b, x, s, t);  
     }  
   
     function md5_ii(a, b, c, d, x, s, t) {  
         return md5_cmn(c ^ (b | (~d)), a, b, x, s, t);  
     }  
   
     function safe_add(x, y) {  
         var lsw = (x & 0xFFFF) + (y & 0xFFFF);  
         var msw = (x >> 16) + (y >> 16) + (lsw >> 16);  
         return (msw << 16) | (lsw & 0xFFFF);  
     }  
   
     /*  
      * Convert a string to an array of little-endian words  
      * If chrsz is ASCII, characters >255 have their hi-byte silently ignored.  
      */  
     function str2binl(str) {  
         var bin = Array();  
         var mask = (1 << chrsz) - 1;  
         for (var i = 0; i < str.length * chrsz; i += chrsz)  
             bin[i >> 5] |= (str.charCodeAt(i / chrsz) & mask) << (i % 32);  
         return bin;  
     }  
     return hex_md5(val)  
   
 }


```

到此为止在浏览器中运行的条件已经准备完毕，接下来测试一下是否正确：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoKicp6A64G17QUYcH07SVf10ialm7eXBRm7lnnZLvZnk37JTPPDSibnQJQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoXmf6kibnibv41YgGJhR5xFPIMfCLcL18o70ickceJBfsVTVNib2icEsWz4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    发现也是能用的，接下来就是按照**jsrpc**的方式，将**js**代码放入**jsrpc**框架中，至于**jsrpc**如何使用，大家可以到公众号中看[JsRpc实战-猿人学-反混淆刷题平台第20题（wasm）](https://mp.weixin.qq.com/s?__biz=MzIxNjQ2NzM3MA==&mid=2247484671&idx=1&sn=eda1e44bf95e119ecc77021ac1d4b77a&scene=21#wechat_redirect)、[网洛者-反反爬练习平台第七题（JSVMPZL - 初体验）](https://mp.weixin.qq.com/s?__biz=MzIxNjQ2NzM3MA==&mid=2247484740&idx=1&sn=be10e2180b8064fef08db3239d3eff8d&scene=21#wechat_redirect)这两篇以及**k哥**写的更加详细的[RPC 技术及其框架 Sekiro 在爬虫逆向中的应用，加密数据一把梭](https://mp.weixin.qq.com/s?__biz=MzIxNjQ2NzM3MA==&mid=2247484747&idx=1&sn=8fd2f3964a5e856d4769edc6b262df20&scene=21#wechat_redirect)这篇文章，最后将代码整合完成之后测试结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoBEaiaFkVVLdyL92oBVPvlRc95adormyS6ZoIxxaaDrl3ekNoNFaK92w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGohIlOS5ogcLOGNrpEDPSnCfIV28mRnm3gkYD1FzHrDEcbQg1rtsHVbw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

经过多次替换**params**参数来测试也是能够拿到数据的，通过**jsrpc**的方式获取加密参数就说到这里了，接下来就开始说重点部分，如何还原这个加密算法。

* * *

### **加密算法还原**

    既然已经抠出了加密算法，且在浏览器中生成的加密参数也能够拿到数据，那么我们接下来就直接在浏览器中调试就好了，在调试之前，先做一些准备工作，把调试中的干扰给去掉，我们能看到这个更新之后的**jsvpm**里面有大量的三元表达式，只要有一个地方判断不对了，那么后面肯定就全部错了，所以要先排除一些能够排除的干扰项，比如里面有判断运行时间的，只要一打断点肯定会超时，就会导致分支发现变化，所以先找到以下地方，将其修改一下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoKBmQR4sxng5Qlym6rx6heib8WywMQsn94ZrhibXKntWialcuburLXI3nQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

之后找到**jsvmp**循环一条条执行指令的位置，直接在该位置插桩：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoSGJRR3ld9g3XKic10icJj3WEURzl2ribVyjbtF6SmMVkfkm1c9ia8FxGFw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

插桩内容：

```


 console.log("索引:", this.s, "值：", this.C.toString())


```

    其‍‍‍实看过[jsvmp-某乎_x-zes-96参数算法还原（手把手教学）](https://mp.weixin.qq.com/s?__biz=MzIxNjQ2NzM3MA==&mid=2247485119&idx=1&sn=297c7b85cbcbb19897f5d89ceada680c&scene=21#wechat_redirect)这篇文章的对这一步应该挺熟悉了，剩下的插桩内容其实跟那篇文章里面的差不多，所以本次的插装完全可以参考那篇文章，这里就不做过多说明了。

    先说一下更新之后的算法，与上一次相比首先是可以看出本次更新之后生成的加密参数，在传入**md5**值不变的情况下生成的加密参数是不一样的，而一般导致这种情况的无非是加密参数里面加了**时间戳**或者**随机值**，至于是哪种待会儿带着大家见分晓，细心的同学还会发现以前版本生成的加密参数长度是**44**位，现在生成的加密长度是**64**位，这些是从表面上看出来的，接下就直接来分析算法，为大家解开本次算法神秘的面纱。

    话不多说，插完桩后直接运行（因为会检测环境，所以需要在打开知乎页面下），如图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoD1X3v3FuicibwQO2xNsku6CfKVBRKJoF7hD9ySiceBVJdTvDibbQnrw6pQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    看到这个图就不用我说怎么开始分析了吧，在讲旧版算法的时候已经说的是相当详细了，所以这里直接跳过这一段，先来到生成这串加密参数的第一个值的位置：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGosb7duMhFHoNOyriaezJDIuDWR1GXVxD14cjXnBwNjcm77LSmecicO5dw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    大家看图，日志信息都输出的很详细了，还记的旧版本的算法里面会有一个初始的字符串，然后加密参数就是从里面依次获取的，这次也是一样，直接开始记录重要的步骤，因为是从下往上看的，所以做记录的时候也按这个步骤：（无关或者不重要的步骤就直接跳过了，至于为什么还是参考之前那篇文章）

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGo2EMGoLyiaGUNfpQUptbiaaIeHc74XtQtKBRsUbdVibZ5ffJTm7H6faTLw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从记录中能看到得出一个值的过程，接来下看看后面几个值怎么生成的并记录下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoV8rEiaybibuc2GpW6ia6Tvvwsh90dz24dZbTnYLrgvEnj7faBgwADaL1A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoiaTLTYOupDmicQjia2poWX4VUKelRtu7AWw3ibvGceNpb0yBuPkGl7U1aw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    看到这个地方，还原过旧版本的同学就能看出来，这里是和旧版本一样的方式，在旧版本中是总共分成**11**组，每组**4**个值，每组每次计算出**第一个值**，之后**第二到第四个值**就是通过**第一个值**中最后计算出来的一个数值来生成，最终生成的加密参数长度是**44**，本次算法中也是一样，不过这次是分成了**16**组，每组**4**个值，最终生成的加密参数长度是**64**，所以后面的值怎么生成的也就不用继续说了吧，不知道看看旧版的分析文章就好了。

    到目前为止能够看出的也就是**第二到第四个值**怎么来的，而**第一个值**我们虽然看到了计算过程，但是参数计算的数值从哪里来的其实是不知道的：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoos3UY00twRvL17fOUmw8oJcFZMOUXq8yn762T1cU9ws14Yb3a4Wypw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    上图中划出来的三个值就是目前不知道哪里来的，不过不用慌，日志继续往上看，然后看到如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoFnicCVHA1qac0CgJ13aKGhpNrYGb4Y1ibHibmWw1IUNazqEWKbMcENYNw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    对数字比较敏感的同学一般看到这个地方就能发现点什么了，圈出来的三个值不就是我上面划出来的吗，对数字不是那么敏感的同学，可能会因为这个颜色不太显眼而忽略这个地方，不过也肯定会继续往上看的，然后看到如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoaBc1fVmBe1cLgOYGYNCmxbp5DaPibhLnPrB6CSQ9tO3VsAjB3GlREiaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    圈出来的这行数组该显眼一点了吧，此时总该注意到这一行了吧，这是一个**48位的数组**，到这里应该会有怀疑了吧，计算出第一组加密值中第一个值的参数来自于这个**48位数组**，如果到这里还是不太明白为什么会注意这个**48位数组**的，可以从刚才记录分析**第一组**加密值那里开始继续记录分析剩下的**十五组**加密值，当分析的差不多的时候在回过头来看看这个**48位数组**，之后得出的结论就是：**每一组加密值中第一个值都是由这个48位数组中的三个元素参与计算的**，之前也有说过本次加密参数总共分成**16**组，每组中有**3**个元素参与，正好对应这个**48位数组**，接下来就是探究这个**48位数组**如何来的。

    在逆向的过程中我们需要有怀疑的精神，我自己反正就有怀疑，那么怀疑什么呢？不管是老版本还是现在的新版本，都是传入了一个**md5**值，最终生成了加密参数，但是到此时都没有看到这个**md5**值，所以当时我就直接去搜索这个**md5**值了，直接找到了**md5**值最开始出现的地方：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoic4q5W68Ep3bDBRQTkic60EmKhup7ibScy3V1PNMrcG8WuXNtvyQ3OuSQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

到这里之后就直接开始记录这个**md5**值变化的过程了，记录如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoGiaCsnbISJ4ElyThTYRJTF142k5dzAWNv9ia14cxIQWGqEKOgvr4fJvA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    上面是**md5**值的过程记录，接下来继续：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoWMTKYia8r0LTU5AJWmYa9crjDL2o7LUSOSPic0SkT4eDDxx2Snwic0iamQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    上图圈出来的两个部分不正好就是开始分析的时候说的，加密参数变化无非是里面加了**时间戳**或者**随机值**，这里刚好两个都出现了，接着记录:

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoxxJjhbAT1C8AmolNUOALMDZ3GDWhAflOap32IvZGaZibw6ZYY7ia72ZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    上面的**127**是一个定值，多次对比之后就能确定下来了，之后接着往下看：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGorBbXtJCGdwoiaUV5OjyXmdWPlct4hLJwOdkTtciccSwmsm0fkP28574g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoQnxbSsnXnD8wia0YwbImJp6hMsM9ib2AHyrDXLAA0BrqJdf1mq9J7XXg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoWMQFjbNavfhMxibZibbBzPFqibb5ERU4ATqbAYsWrZcsWbmo17dT2NXQw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoRpW6XzCicF2paaozRpdibYhj7HR9wibYJQfesiadPia0qO7llFCnm60CrYA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里**往数组中放15个14**也是可以经过多次验证的发现是固定的逻辑，接着继续往下看：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGohSGcnyesVmmJkiaQicGbKibHfC9OibnaASlhs8CSzHbuBCBonJrJevtfWQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoz4fxrWOUy9lVJW3oKRSZNSSfhTsJVpEvfDx79duCc3ISfusQfRnHng/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoICKWQSGabwIzp71kN8nAAJ0p8c0PDRZGicE9tzsP9r5gCdH54nG5EeA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    注意上图，这里开始就又要进行计算了，这里由于记录比较长，就只记录前面和后面几步了：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGo8I3NEbkyIYj9czeia58ngSoFFWsbQeOTqtI1QMRXvr2sLtFR09oEJ4w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    经过上面一系列运算之后就来到了下面这步：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoN2TCAtt2VXtrP0eQp3qicicvz00CFI9Z5RBibwJmT3qnXP5SG3LWvYuibg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    这里是将这个**新组成的16位数组**放入函数中进行计算，然后又生成一个**新的16数组**，直接直接将这个函数贴出来：

  

```


 function (e) {  
     var t = new Array(16)  
         , n = new Array(36);  
     n[0] = B(e, 0),  
         n[1] = B(e, 4),  
         n[2] = B(e, 8),  
         n[3] = B(e, 12);  
     for (var r = 0; r < 32; r++) {  
         var o = G(n[r + 1] ^ n[r + 2] ^ n[r + 3] ^ h.zk[r]);  
         n[r + 4] = n[r] ^ o  
     }  
     return i(n[35], t, 0),  
         i(n[34], t, 4),  
         i(n[33], t, 8),  
         i(n[32], t, 12),  
         t  
 }


```

 这个函数是加密函数中的，直接抠出来就好了，剩下的就是缺啥补啥就行了。接着继续看：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoOy4ZTiaYYt6ia8yOdMH6spjFz7ya4eDoBrGPysyUmUXm30ojAxu85Qcw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

```


 //取md5_charCodeAt_arr中的16-48位 ,也就是之前前面得到的48位数组  
 _16_48_arry = md5_charCodeAt_arr.slice(16, 48)  
 //之后又是将这个截取出来的32位和上面刚生成的16位数组放入下面函数中  
 function(e, t) {  
     for (var n = [], r = e.length, i = 0; 0 < r; r -= 16) {  
         for (var o = e.slice(16 * i, 16 * (i + 1)), a = new Array(16), c = 0; c < 16; c++)  
             a[c] = o[c] ^ t[c];  
         t = __g.r(a),  
             n = n.concat(t),  
             i++  
     }  
     return n  
 }


```

    这个函数也是从加密函数中抠出来的，还是缺啥补啥，这一步完成之后会得到一个**新的32位数组**，最后将刚才生成的**16位数组**和这个**32位数组**通过**concat**合在一起就生成了开始需要参数计算的**48位数组**，到此**48位数组**如何生成的就讲解完成了。

    再说一下刚才的**时间戳**和**随机值**，在后面生成**48位数组**的过程中会发现这个时间戳其实并没有参与进去，所以其实导致最终加密参数变化的就是这个**随机值**。本篇文章的分析到这里就差不多了，这篇文章也主要是为了分析这个**48位数组**怎么来的了，至于最终的加密参数如何还不知道如何分析的直接看之前的分析旧版的那篇文章就好了。

    最后验证一下还原之后的算法是否正确，首先看看**js版本**的:

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoo9rjsywS0t3y2ns0VtfqYdgUgUR03Fpgb2CPJEqeuL3I63Teiag9x9g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGouicuIsg8UMJzEMQRvAdQ4erBagVsibz4dgsdEFkyCbZ4lnHiaq30BLH3w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

再来看看**python版本**的:

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoF8ZDI9mcvEj4fJYq1cjjCH3keSKpIqRs22LMUlqiae1cz1ZPM04ZIuQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic0qrAf8gBTibMMH01pPVmmGoIjw5VD2vASTkZvqc7U7L2GkYnp75ia8TpNHCb6V7XvxDj0zbTM7J1Dg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

能够看到不管是**js版本**还是**python版本**的都能够拿到数据。

* * *

**旧版某乎算法分析文章**  

[jsvmp-某乎_x-zes-96参数算法还原（手把手教学）](https://mp.weixin.qq.com/s?__biz=MzIxNjQ2NzM3MA==&mid=2247485119&idx=1&sn=297c7b85cbcbb19897f5d89ceada680c&scene=21#wechat_redirect)