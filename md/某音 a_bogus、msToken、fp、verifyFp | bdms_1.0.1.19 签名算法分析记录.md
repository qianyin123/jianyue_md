> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/WsFEyqmHGeL_2lmkY31Y_g)

> 【作者主页】：小鱼神1024
> 
> 【擅长领域】：JS逆向、小程序逆向、AST还原、验证码突防、Python开发、浏览器插件开发、React前端开发、NestJS后端开发等等

> 本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！若有侵权，请联系作者立即删除！

> 【该文章已同步至星球】：https://articles.zsxq.com/id_khqgib13pvw4.html

### 前言

最近听小伙伴说，`a_bogus` 签名算法又变了，出于学习目的，于是乎，我决定重新分析记录一下，希望能帮助到有需要的小伙伴。

### 前置分析

我们在请求header中发现，有很多请求都带有`a_bogus`、`msToken`、`fp`、`verifyFp`这四个参数的值是动态变化的，所以我们猜测这四个参数应该是加密参数。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDvqnGHpjPiaVgP0X45NXRma0iaAJvRezFTQTACAib90cwoldvhumO7fFiag/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

### 逆向分析

#### fp、verifyFp

从上图可以发现，`fp`、`verifyFp`这两个参数的值是一样的。

全局搜索 `verifyFp`，打上断点，重新请求，发现断住了。如下图：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDq82uI26K7iaibDfkLgMRY6PLfWmwyZXUuGs3sVKmIB0g6jWN8t3wP8qg/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

其中，`fp`、`verifyFp`的值来自 `r`, 往上翻可以看到，`var r = n.getFp()`，打上断点，重新请求。进入`getFp` 函数，如下图：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDVQJGBPFUbibDUhY9pcCYkmt52Nr5736YEibdc8AhDmsXS34dIPibM22mg/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

提取函数为：

```
`function get_fp() {  
    var e = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz".split("")  
      , t = e.length  
      , n = Date.now().toString(36)  
      , r = [];  
    r[8] = r[13] = r[18] = r[23] = "_",  
    r[14] = "4";  
    for (var o = 0, i = void 0; o < 36; o++)  
        r[o] || (i = 0 | Math.random() * t,  
        r[o] = e[19 == o ? 3 & i | 8 : i]);  
    return "verify_" + n + "_" + r.join("")  
}`
```

#### msToken

经过测试发现，它不是前端js生成的，是服务端生成之后，保存到cookie中的，如下图：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDhic3Fsccz5HqyQuYRJgicou1DqCaSQOcPoZ8navJUWGibvqKay7qvQKmQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

经过多次对比发现，它的生成方式和 `base64` 算法生成相似，所以我们可以尝试模拟生成，代码如下：

```
`function get_ms_token(length = 182) {  
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_';  
  let result = '';  
  for (let i = 0; i < length; i++) {  
      result += chars.charAt(Math.floor(Math.random() * chars.length));  
  }  
  return encodeURIComponent(result);  
}`
```

#### a_bogus

重头戏来喽。打起精神来，这部分插桩分析可能有点枯燥，耐心看完，相信你会有收获的。

我也把自己当成小白，从头开始分析，一步步来。

由于平台大部分接口暂时没有对`a_bogus` 参数做强校验，但二级评论接口是个例外，如果不加`a_bogus` 参数，会返回错误。为了验证`a_bogus` 参数生成的是否正确，我们决定先从二级评论接口入手。

通过跟栈分析，发现`a_bogus` 参数是在 `bdms.js` 文件中生成的，点击进去，发现是 `vmp` 代码，如下图：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDwCJv2v14156GaZ9zrzXJwFqoSCSVosnQNB2gfxgoDCBkoVUHSCaW6A/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

有些小伙伴第一时间可能想补环节，但是，尝试过之后的结果会让你失望的，因为你都很难找到生成入口。。

所以，我们换个思路，直接插桩纯算。

说到插桩，你可以尝试一下我之前写的插桩日志框架，可能会让你事半功倍。传送门：终极逆向插桩日志框架，让浏览器崩溃成为历史！

使用日志框架在`bdms.js` 文件中插桩，如下图：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDbSSJFNXazIAraibIiaRz84fMqMknlyA4Gssdf85zcwXDY6ewoiceSnUqg/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDuGQ2mK0d8TI098Uf98uYyT1Hbs4CK5cq8wsePibg7lDm9HHTNnJW0Ag/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

由于运算符比较多，我就不一一截图展示了。自己把常见的运算符都插桩下即可。

打开日志文件，开始分析。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDywzz0Vdxz1yiaicoLpDE85GUUQAOmEJiagKeyxjojmPpFFYVlszS4A16w/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDICdxBKsGZDuXCpAdQhm3G3GJqjZEE4lhicZEmo97ynIicy85twBrwgcA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

首先，我们发现请求参数通过加盐 `dhzx`, 得到新的值：`device_platform=webapp&aid=6383&channel=channel_pc_web&item_id=7429636428608425267&comment_id=7431107242646258473&cut_version=1&cursor=0&count=3&item_type=0&update_version_code=170400&pc_client_type=1&pc_libra_divert=Mac&version_code=170400&version_name=17.4.0&cookie_enabled=true&screen_width=1280&screen_height=800&browser_language=zh-CN&browser_platform=MacIntel&browser_name=Chrome&browser_version=131.0.0.0&browser_online=true&engine_name=Blink&engine_version=131.0.0.0&os_name=Mac+OS&os_version=10.15.7&cpu_core_num=8&device_memory=8&platform=PC&downlink=10&effective_type=4g&round_trip_time=0&webid=7427765608665040399&uifid=63dd93172fb5fdaa8078f33ae0ba1106c8e3f1f870eecbc4b54b497d452c3517e3b7a5ab3d4ec186d7be006b7a6d53846c6734c86b69acdaa2eb531c0fa92958a8cb940dd76c55017b279b6778ad806977b04a00d235639ba1505bb8c942fc1f1ffdb6b5bed6e26fc7db5b9400d1d9d20450dca0257efb46047e349b49ef1ed79c5ceaff02e8b12e4efffc9114f06bb77651d820b35a2638844fee7a6ac6db85&msToken=ahDrxe9ibTXPfbP25qOkt6QDX0xXfaehFEV-e5XofFV3eQQBqtpPVa_SYmy-Gdi2wSRdz4JN_U1I_fghOF2VNxh_B0A1MlFqT0vsgUqOvv5WRg-W_yIuqJrYt_e-76afo7T6nlzQIiSNZF04A82HjbCnogmaut96WjCkWSJivKn1TdYy04jsXw%3D%3D , + , dhzx , ====> , device_platform=webapp&aid=6383&channel=channel_pc_web&item_id=7429636428608425267&comment_id=7431107242646258473&cut_version=1&cursor=0&count=3&item_type=0&update_version_code=170400&pc_client_type=1&pc_libra_divert=Mac&version_code=170400&version_name=17.4.0&cookie_enabled=true&screen_width=1280&screen_height=800&browser_language=zh-CN&browser_platform=MacIntel&browser_name=Chrome&browser_version=131.0.0.0&browser_online=true&engine_name=Blink&engine_version=131.0.0.0&os_name=Mac+OS&os_version=10.15.7&cpu_core_num=8&device_memory=8&platform=PC&downlink=10&effective_type=4g&round_trip_time=0&webid=7427765608665040399&uifid=63dd93172fb5fdaa8078f33ae0ba1106c8e3f1f870eecbc4b54b497d452c3517e3b7a5ab3d4ec186d7be006b7a6d53846c6734c86b69acdaa2eb531c0fa92958a8cb940dd76c55017b279b6778ad806977b04a00d235639ba1505bb8c942fc1f1ffdb6b5bed6e26fc7db5b9400d1d9d20450dca0257efb46047e349b49ef1ed79c5ceaff02e8b12e4efffc9114f06bb77651d820b35a2638844fee7a6ac6db85&msToken=ahDrxe9ibTXPfbP25qOkt6QDX0xXfaehFEV-e5XofFV3eQQBqtpPVa_SYmy-Gdi2wSRdz4JN_U1I_fghOF2VNxh_B0A1MlFqT0vsgUqOvv5WRg-W_yIuqJrYt_e-76afo7T6nlzQIiSNZF04A82HjbCnogmaut96WjCkWSJivKn1TdYy04jsXw%3D%3Ddhzx`，后续就进入进行两次某种算法加密。

经过测试发现，这个是标准的 `sm3` 算法，即：`sm3(sm3(搜索参数+dhzx))`。当然了，你可以扣 `sum` 函数，也可以计算出值。

日志继续往下翻，发现盐值 `dhzx` 也进行两次 `sm3` 算法加密，即：`sm3(sm3(dhzx))`。

日志继续发下翻，发现 `user-agent` 进入我们的视野，同时发现一个长度为3的数组，即：`[0.00390625,1,12]`，经过测试，Windows 系统下，这个数组为 `[0.00390625,1,8]`，都是固定的。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnD2GGXZsbbNSULSOwwAHlCFATeG9SZYq5DJ8MGzpAhvt7PNyGew0IfFw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

后续就进入了一个循环计算，一开始我也不清楚它是什么算法，但是它有一个特点，就是先初始化一个从 `255 - 0` 的数组，然后又是一个 `0 - 255` 的循环，而且每次还进行相同的运算，如下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnD7aUibAnrpmtWHibMTfvgITgQ6lSSFSIRq080ujcvnBjCxn4SK9z6oLLA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

看它的运算规则，很像 `rc4` 算法，如下：

```
`function rc4(key, str) {  
    var s = [], i = 0, j = 0, x, res = '';  
    for (var k = 0; k < 256; k++) {  
        s[k] = k;  
    }  
    for (k = 0; k < 256; k++) {  
        j = (j + s[k] + key.charCodeAt(k % key.length)) % 256;  
        x = s[k];  
        s[k] = s[j];  
        s[j] = x;  
    }  
    for (var k = 0; k < str.length; k++) {  
        i = (i + 1) % 256;  
        j = (j + s[i]) % 256;  
        x = s[i];  
        s[i] = s[j];  
        s[j] = x;  
        res += String.fromCharCode(str.charCodeAt(k) ^ s[(s[i] + s[j]) % 256]);  
    }  
    return res;  
}`
```

所以，我们可以大胆猜测，它就是 `rc4` 算法，只不过它的 `key` 是一个数组，如下：

```
`var key = [0.00390625, 1, 12];`
```

经过代码验证，它不是标准的 `rc4` 算法，而是 `rc4` 算法的变种，即魔改的 `rc4` 算法。

`遇到类似的运算加密，尽量不要根据日志去还原代码，很麻烦的，亲身经历过的。。。，而是尽量能找到类似结构的标准算法上去改造，这样对于插桩纯算，很节约大量时间。`

所以根据日志，很容易就得到魔改后的 `rc4` 算法，如下：

```
`function rc4_magic_encrypt(plaintext, key) {  
  var s = [];  
  for (var i = 0; i < 256; i++) {  
    s[i] = 255 - i;  
  }  
  var j = 0;  
  for (var i = 0; i < 256; i++) {  
    j = (j * s[i] + j + key.charCodeAt(i % key.length)) % 256;  
    var temp = s[i];  
    s[i] = s[j];  
    s[j] = temp;  
  }  
  
  var i = 0;  
  var j = 0;  
  var cipher = [];  
  for (var k = 0; k < plaintext.length; k++) {  
    i = (i + 1) % 256;  
    j = (j + s[i]) % 256;  
    var plaintext_charCodeAt = plaintext.charCodeAt(k);  
    var temp = s[i];  
    var t = (temp + s[j]) % 256;  
    s[i] = s[j];  
    s[j] = temp;  
    cipher.push(String.fromCharCode(plaintext_charCodeAt ^ s[t]));  
  }  
  return cipher.join("");  
}`
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDNDzmOibiaznoEg9PVODII1gWKv02O1SV0mlLPCjibADPnryiaVnrQ8ZDgQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

继续往下翻，可以看到，上述魔改后的 `rc4` 算法加密后，再进行三个字符一组进行循环，而且还看到：`a << 16` 、`a << 8` 等关键运算。此时，我暗自窃喜，因为这种运算，很容易想到 `base64` 算法。标准如下：

```
`function standardBase64Encode(inputString) {  
    // 标准Base64字符集  
    var base64Chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";  
  
    // 处理一个3字节组的函数，将其转换为4个Base64字符  
    function processGroup(a, b, c) {  
        // 组合三个字节为一个24位整数  
        var combined = (a << 16) | (b << 8) | c;  
  
        // 从24位整数中提取4个6位部分  
        var part1 = (combined >> 18) & 63; // 提取最高的6位  
        var part2 = (combined >> 12) & 63; // 提取次高的6位  
        var part3 = (combined >> 6) & 63;  // 提取次低的6位  
        var part4 = combined & 63;         // 提取最低的6位  
  
        // 将每部分映射到标准Base64字符集  
        return [  
            base64Chars.charAt(part1),  
            base64Chars.charAt(part2),  
            base64Chars.charAt(part3),  
            base64Chars.charAt(part4)  
        ];  
    }  
  
    // 将输入字符串转为字节数组  
    var inputArray = [];  
    for (var i = 0; i < inputString.length; i++) {  
        inputArray.push(inputString.charCodeAt(i));  
    }  
  
    var encodedStr = ""; // 存储最终编码结果  
    for (var i = 0; i < inputArray.length; i += 3) {  
        // 每次处理三个字节  
        var group = inputArray.slice(i, i + 3); // 获取3字节组  
        var paddedGroup = group.slice(0); // 复制数组  
        while (paddedGroup.length < 3) {  
            // 不足3字节时用0填充  
            paddedGroup.push(0);  
        }  
        // 处理组并将结果拼接到最终字符串  
        var encodedGroup = processGroup(paddedGroup[0], paddedGroup[1], paddedGroup[2]);  
        encodedStr += encodedGroup.join(""); // 将编码的字符数组拼接为字符串  
    }  
  
    // 处理末尾填充（如果有的话）  
    var paddingLength = (3 - inputArray.length % 3) % 3;  
    if (paddingLength > 0) {  
        encodedStr = encodedStr.slice(0, -paddingLength) + Array(paddingLength + 1).join("=");  
    }  
  
    return encodedStr; // 返回编码后的字符串  
}  
  
// 使用示例：  
var encoded = standardBase64Encode("Hello, World!");  
console.log(encoded); // 输出：SGVsbG8sIFdvcmxkIQ==`
```

经过验证，它又不是标准的 `base64` 算法，而是魔改后的 `base64` 算法。此时，我内心是崩溃的。。

没办法，只能再继续分析日志，在原有的 `base64` 算法基础上，再进行魔改。经过一番努力，终于找到了规律，并还原。

上述有了 `rc4` 魔改算法的经验，这个我就不贴出来了，毕竟我们学习插桩技巧的。不难的，多尝试，努力提升自己。

继续往下翻日志，发现魔改 `base64` 加密后的结果，又进行了 `sm3` 加密。日志如下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDfspLFicT5hlFO4gm43aSgSeYcXYuZkB5OuB3iaFaGgF43zF19Ivuxbzg/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

继续往下翻日志，很难找到规律了。此时，我内心又是崩溃的。

当时，我想着既然正向找不到规律，那我就逆向分析，看看能不能找到规律。于是，我就找到生成 `a_bogus` 的地方，进行逆向分析。

全局搜索日志中的 `a_bogus`，找到生成 `a_bogus` 的地方，如下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDyOH3PJhgh4eD8weuqZvsASrocrgqmtv8uwQUjJXNFJB9RBTTMswibibA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

从下往上翻日志，发现 `a_bogus` 是从长度为 `134` 位数组得到的，如下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDrE75K9jV57hkOF5dXDELQ3xJatyDvuRYx1nNPOe6odiazSjGL9lYJUw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDL6y1dXMQd15h65MPEtQ164OGzSkf9pFZL6AzpMXmtWn2rj4AmvGKxA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

当然了，不同浏览器，环境也不一样，加密参数也不一样，所以，你的浏览器未必是 `134` 位数组。这是为啥有的人得到结果是：`184`、`188`、`192` 长度 `a_bogus` 的原因。

经过日志分析得到，`a_bogus` 是从长度为 `134` 位数组通过魔改的 `rc4` 算法得到的。

那问题又来了。这个长度为 `134` 位数组又是从哪里来的呢？

继续往上翻日志，如下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDhhZibw3PA7Epxyfc60vJ2cIRqF1AqolnOTCPgA26OiccRMn5Zich5XsFw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

经验证，数组长度为 `8` 的数组是随机生成的。所以固定写死也行。如下：

```
 `const a8 = [171, 68, 168, 1, 35, 64, 162, 16];`
```

剩下的长度 `126` 数组是由：

```
`a126 = a124 + a95最后2位，其中a124数组也是根据a95位数组生成的  
a95 =a50 + a40(环境：663|730|1536|816|1536|816|1536|864|Win32) + a4(时间戳) + a1(通过a8和a50异或^得来)`
```

按照上述思路，可以得到 `a40` 数组，如下：

```
`function getA40CharCodeAts() {  
    var str = "663|730|1536|816|1536|816|1536|864|Win32";  
    var charCodes = [];  
  
    for (var i = 0; i < str.length; i++) {  
        charCodes.push(str.charCodeAt(i));  
    }  
    return charCodes;  
}  
console.log(getA40CharCodeAts())`
```

这个是 `Windows` 环境，以下是 `mac` 环境的：

```
`function getA43CharCodeAts() {  
    var str = "592|613|1280|734|1280|734|1280|800|MacIntel";  
    var charCodes = [];  
  
    for (var i = 0; i < str.length; i++) {  
        charCodes.push(str.charCodeAt(i));  
    }  
    return charCodes;  
}  
console.log(getA43CharCodeAts())`
```

这个就是 `a43` 数组，这也是为啥，不同环境，`a_bogus` 长度不一样的部分原因。

那 `a4` 数组呢？其实叫 `a4` 有点不严谨，往下看就知道了。它其实就是时间戳得到的，如下：

```
``function getA4OrA3OrA2CharCodeAts() {  
    // const timestamp = 1733376433069;  
    const timestamp = Date.now();  
    const tString = `${(timestamp + 3) & 255},`;  
    var charCodes = [];  
    for (var i = 0; i < tString.length; i++) {  
        charCodes.push(tString.charCodeAt(i));  
    }  
    return charCodes;  
}  
console.log(getA4OrA3OrA2CharCodeAts())``
```

这个也是一个坑点，因为 `tString` 得到的字符串长度不固定，可能为 `4` 或者 `3` 或者 `2`，所以，它会导致 `a95` 数组长度不固定，进而可能导致 `a_bogus` 长度不固定。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDCtjJQUQD69uicSKZBzuhyhsKUccMJeE63Jb2zteGZPEichHMA0G8MNbA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

从图中可以发现，`a1` 是根据 `a8` 和 `a50` 异或得到的。

至此，`a_bogus` 的生成逻辑就分析完了。

### 参数验证

将 `a_bogus`、`msToken`、`fp`、`verifyFp` 四个参数整理后放到 `get_dy_sign` 函数中，直接生成四个参数，方便后续调用。

写个小例子，验证下生成的参数是否正确，如下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/vxZ2DaY4Mcv6cHcdVE9ASKT14UIBmicnDNne4PjuGLSEUGdia3xWJWJKyt6c4icPojct8MianEA9AT0xhkNej8ibIxQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "null")抖音a_bogus、msToken、fp、verifyFp

搞定！！

如果还有什么疑问，欢迎留言讨论！