> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5N0Kp35XGJAuJSiIyLQt1g)

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4LSlgcyicchwNsADGJ0Gf9jUAibRKFnwoibGdYrfuTIuxgsPfWRpBOOAcZicY1rqia13ZUetbWoA297Xag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

声明
--

**本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！**

逆向目标
----

*   目标：加速乐加密逆向
    
*   主页：aHR0cHM6Ly93d3cubXBzLmdvdi5jbi9pbmRleC5odG1s  
    
*   逆向难点：OB 混淆、动态加密算法、多层 Cookie 获取
    

加速乐  

------

加速乐是知道创宇推出的一款网站CDN加速、网站安全防护平台。

加速乐的特点是访问网站一般有三次请求：

1.  第一次请求网站，网站返回的响应状态码为 521，响应返回的为经过 AAEncode 混淆的 JS 代码；
    
2.  第二次请求网站，网站同样返回的响应状态码为 521，响应返回的为经过 OB 混淆的 JS 代码；
    
3.  第三次请求网站，网站返回的响应状态码 200，即可正常访问到网页内容。  
    

逆向思路  

-------

根据我们上面讲的加速乐的特点，我们想要获取到真实的 HTML 页面，需要经过以下三个步骤：

1.  第一次请求网站，服务器返回的 Set-Cookie 中携带 jsluid_s 参数，将获取到的响应内容解密拿到第一次 jsl_clearance_s 参数的值；
    
2.  携带第一次请求网站获取到的 Cookie 值再次访问网站，将获取到的响应内容解混淆逆向拿到第二次 jsl_clearance_s 参数的值；
    
3.  使用携带 jsluid_s 和 jsl_clearance_s 参数的 Cookie 再次访问网站，获取到真实的 HTML 页面内容，继而采集数据。
    

抓包分析  

-------

进入网站，打开开发者人员工具进行抓包，在 Network 中我们可以看到，请求页面发生了三次响应 index.html，且前两次返回状态码为 521，符合加速乐的特点

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxibibCdGq537EgpIIzfhG8cfM0whVFpfgBeGPxjtCxwxyTTDOOk5aXcCg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### **第一层 Cookie 获取**

直接查看 response 显示无响应内容，我们通过 Fiddler 对网站进行抓包，可以看到第一个 index.html 返回的响应内容经过 AAEncode 加密，大致内容如下，可以看到一堆颜表情符号，还挺有意思的：

```
<script>  
    document.cookie=('_')+('_')+('j')+('s')+('l')+('_')+('c')+('l')+('e')+('a')+('r')+('a')+('n')+('c')+('e')+('_')+('s')+('=')+(-~[]+'')+((1+[2])/[2]+'')+(([2]+0>>2)+'')+((2<<2)+'')+(-~(8)+'')+(~~{}+'')+(6+'')+(7+'')+(~~[]+'')+((1<<2)+'')+('.')+((+true)+'')+(~~{}+'')+(9+'')+('|')+('-')+(+!+[]+'')+('|')+(1+6+'')+('n')+((1<<2)+'')+('k')+('X')+((2)*[4]+'')+('R')+('w')+('z')+('c')+(1+7+'')+('w')+('T')+('j')+('r')+('b')+('H')+('m')+('W')+('H')+('j')+([3]*(3)+'')+('G')+('X')+('C')+('t')+('I')+('%')+(-~[2]+'')+('D')+(';')+('m')+('a')+('x')+('-')+('a')+('g')+('e')+('=')+(3+'')+(3+3+'')+(~~{}+'')+(~~[]+'')+(';')+('p')+('a')+('t')+('h')+('=')+('/');location.href=location.pathname+location.search  
</script>
```

document.cookie 里的颜表情串实际上是第一次 __jsl_clearance_s 的值，可以直接通过正则提取到加密内容后，使用execjs.eval()方法即可得到解密后的值：

```
import re  
import execjs  
  
  
AAEncode_text = """以上内容"""  
content_first = re.findall('cookie=(.*?);location', AAEncode_text)[0]  
jsl_clearance_s = execjs.eval(content_first).split(';')[0]  
print(jsl_clearance_s)  
# __jsl_clearance_s=1658906704.109|-1|7n4kX8Rwzc8wTjrbHmWHj9GXCtI%3
```

### **第二层 Cookie 获取**

抓包到的第二个 index.html 返回的是经过 OB 混淆的 JS 文件，我们需要对其进行调试分析，但是直接在网页中通过 search 搜索很难找到该 JS 文件的位置，这里推荐两种方式对其进行定位：

**1.文件替换**

右键点击抓包到的第二个状态码为 521 的 index.html 文件，然后按照以下方式将其保存到本地：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxDP7qnkibTtWZJgrFnKwZktxWBpyynLiaLFJ08CAtl2wlHLbEPPVdKfKw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

保存到本地后会发现 JS 文件被压缩了不利于观察，可以通过以下网站中的 JS 格式化工具将其格式化：https://spidertools.cn/#/formatJS，将格式化后的代码粘贴到编辑器中进行处理，可能需要一些微调，例如首尾 Script 标签前后会多出空格，在 < script > 后添加debugger;如下所示：

```
<script>  
debugger;  
var _0x1c58 = ['wpDCsRDCuA==', 'AWc8w7E=', 'w6llwpPCqA==', 'w61/wow7', 
```

最后通过 Fiddler 对其替换，点击 Add Rule 添加新的规则，如以下步骤即可完成替换：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxg5ZymFLonI4AhiarUJicEDkMJbBwOnspNsgKia0Y8icqDXS4BqDqsJ0WVQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

以上操作完成后，开启 Fiddler 抓包（F12 左下角显示 Capturing 即抓包状态），清除网页 Cookie，刷新网页，会发现成功断住，即定位到了 JS 文件的位置，可断点调试：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxWG97lDC8mhvy1mXXmRzNM1MiaPBuYeypwNJeQzzZFGWhYT1DNguiaibmw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**2.Hook Cookie 值**

因为我们获取到的 JS 文件生成了 Cookie，其中包含 jsluid_s 和 jsl_clearance_s 参数的值，所以我们不妨直接 Hook Cookie 也能断到 JS 文件的位置，对 Hook 方法不了解的可以看看 K 哥往期的文章，以下是 Hook 代码：

```
(function () {  
    'use strict';  
    var org = document.cookie.__lookupSetter__('cookie');  
    document.__defineSetter__('cookie', function (cookie) {  
        if (cookie.indexOf('__jsl_clearance_s') != -1) {  
            debugger;  
        }  
        org = cookie;  
    });  
    document.__defineGetter__('cookie', function () {  
        return org;  
    });  
})();
```

Hook 注入的方式有很多种，这里通过 Fiddler 中的插件进行注入，该插件在 K 哥爬虫公众号中发送【Fiddler 插件】即可获取：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxNJC7ptMOcsjicKJUWoTDDTojY5puDQh22M1iaJDrwLHRUZKdRnq81uZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

同样，设置完成后开启抓包，清除网页缓存，刷新网页，页面也能被顺利断住，上半部分就是我们通过 Hook 方式注入的代码段，显示出了 Cookie 中 __jsl_clearance_s 关键字的值，下面框起来的部分格式化后会发现就是之前经过 OB 混淆的 JS 文件内容：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxCQ6EhHRThTUoZ8NylDNzictXzx15SRpiafJezgA6eb9EkS0u54Bo6UCw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**调试分析 JS 文件**

经过 Hook 之后，往前跟栈就能找到加密位置，我们知道 JavaScript 中一般使用 document.cookie 属性来创建 、读取、及删除 cookie，经过分析 JS 文件中的一些参数是在动态变换的，所以我们使用本地替换的方式固定一套下来，然后在该 JS 文件中通过 CTRL + F 搜索 document，只有一个，在第 558 行打断点调试，选中_0x2a9a('0xdb', 'WGP(') + 'ie'后鼠标悬停会发现这里就是 cookie 经过混淆后的样式：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxiaK1icvULSmmf9IXWgdVicNMyZa4Hk3kG1wgmwujqo0FibjaxckzjuEq9g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

将等号后面的内容全部选中，鼠标悬停在上面可以发现，这里生成了 Cookie 中 __jsl_clearance_s 参数的值：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxrRNufj7Alds3HJH2cIny5oz17qDZy2W3RsqtQyk6TibPpZnzBzR67uA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

至此，我们知道了 Cookie 生成的位置，接下来就需要了解其加密逻辑和加密方法，然后通过 python 对其进行复现了，document 部分完整的代码如下：

```
document[_0x2a9a('0xdb', 'WGP(') + 'ie'] = _0x2228a0[_0x2a9a('0x52', '$hOV') + 'W'](_0x2228a0[_0x2a9a('0x3', '*hjw') + 'W'](_0x2228a0[_0x2a9a('0x10b', 'rV*F') + 'W'](_0x60274b['tn'] + '=' + _0x732635[0x0], _0x2228a0[_0x2a9a('0x3d', 'QRZ0') + 'q']), _0x60274b['vt']), _0x2228a0[_0x2a9a('0x112', ']A89') + 'x']);
```

OB 混淆相关内容可以观看 K 哥往期文章，这里等号后面的内容比较冗杂，其实我们想要获取的是 jsl_clearance_s 参数的值，通过调试可以看到其值由0x60274b['tn'] + '=' + _0x732635[0x0]生成:

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxjqwZt0UDRgbt7oj3vyWJuIDorNLIbW0Lh4O8T98iazwiceSeaRWRcDzQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

由上可知0x60274b['tn']对应的部分是 __jsl_clearance_s，而其值是0x732635[0x0]，因此我们需要进一步跟踪 0x732635生成的位置，通过搜索，在第 538 行可以找到其定义生成的位置，打断点调试可以看到，0x732635[0x0]其实就是取了 0x732635 数组中的第一个位置的值：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcx52gyyYUuadtIuz3gnVaN1AAtnf65XdDsApPb5JgwbQUtMjNj2oLUqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们来进一步分析 0x732635 后面代码各自的含义，_0x14e035(_0x60274b['ct'])取的是 go 函数传入的字典中 ct 参数的值：

```
go({  
    "bts": ["1658906704.293|0|YYj", "Jm5cKs%2B1v1GqTYAtpQjthM%3D"],  
    "chars": "vUzQIgamgWnnFOJyKwXiGK",  
    "ct": "690f55a681f304c95b35941b20538480",  
    "ha": "md5",  
    "tn": "__jsl_clearance_s",  
    "vt": "3600",  
    "wt": "1500"  
})  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxPNzicZtjadhVo38yDic5UeNOwYUcV2U0G8eTKYx64VLENHdf1rwwveDw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

分析可知将_0x60274b[_0x2a9a('0xf9', 'uUBi')]数组中的值按照某种规则进行拼接就是 __jsl_clearance_s 参数的值，并且_0x2a9a('0xf9', 'uUBi')对应字典中 bts 的值：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxXPk4uII409TY7licV1UVM6keqNj8v4Au94bhazicdSgbd7kiaJ1R51HzQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxSX3Ly2ntYicgVibdIMVRDNYRlkROVpQYw9NKbcIX6aFrvhvmwZOgthKw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下来先进一步跟踪 _0x14e035，可以发现其是个函数体，第 533 行 return 后的返回值就是 __jsl_clearance_s 参数的值：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcx05iaKUgsoVDZEV2rNZndmWCp3bKibTaZ55q7CUUXqpyLaZNB4Qg2ZaPg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在第 532 行打断点调试，能知道 hash 后 _0x2a7ea9 为 __jsl_clearance_s 参数的值：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxibEyCXgQGL1829mlddgRJXxWxicVB3HuNQwbhX4xMtAF8I3fbFuIYiacw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

hash( _0x2a7ea9 ) 的值为 _0x2a7ea9 经过加密后的结果，在本例中，加密结果由 0-9 和 a-f 组成的 32 位字符串，很明显的 MD5 加密特征，找个在线 MD5 加密进行验证，发现是一致的，这里加密的方法即 hash 方法不全是 MD5，多刷新几次发现会变化，实际上这个 hash 方法与原来调用 go 函数传入的字典中 ha 的值相对应，ha 即加密算法的类型，一共有 md5、sha1、sha256 三种，所以我们在本地处理的时候，要同时有这三种加密算法，通过 ha 的值来匹配不同算法。

进一步观察这里还有个 for 循环，分析发现每次循环 hash(_0x2a7ea9) 的值是动态变化的，原因是 _0x2a7ea9 的值是在动态变化的，_0x2a7ea9 中只有中间两个字母在变化，不仔细看都看不出来：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxicXricZnZMZOLvISMQrlawSyicna8hWQBH38Q99rVWicI4WlYabxbxqjKQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

跟进 _ 0x2a7ea9 生成的位置，分析可知 _0x2a7ea9 参数的值是由 0x5e5712 数组的第一个值加上两个字母再加上该数组第二个值组成的结果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxLKnhcgkPiazYS7WeQZrPE621hibo32Jh22a0YLXzSMWDbsKU6PcW5HPg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

中间两个字母是将底下这段写了两次生成的，即 _0x60274b['chars']['substr'][1]， 取字典中 chars 参数的一个字母，取了两次，这里通过 for 循环在不断取这两个值，直到其值加密后与 _0x56cbce（即 ct）的值相等，则作为返回值传递给 __jsl_clearance_s 参数：

```
_0x60274b[_0x2a9a('0x45', 'XXkw') + 's'][_0x2a9a('0x5a', 'ZN)]') + 'tr'](_0x8164, 0x1)
```

0x56cbce 为 ct 的值：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxt0rBiczkcTYggj7fWWcs8aXyxr0ibhEWy8VjKDJdQSBiaHibTW8txkx5Hg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最前面0x2228a0[_0x2a9a('0x6d', 'U0Y3') + 's']是个方法，我们进一步跟进过去，看这个方式里面实现了什么样的逻辑：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4K4xsicZibPichq5ogI5qAQZcxuoV1LwwrlD6NugjQo7ibwicIPVicZRakcc86Ct97l59vNMN82LzkfbicVA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其内容如下，可以看到这个方法返回的值是两个相等的参数：

```
_0x560b67[_0x2a9a('0x15', 'NwFy') + 's'] = function(_0x4573a2, _0x3855be) {  
 return _0x4573a2 == _0x3855be;  
}  

```

### **模拟执行**

综上所述，_0x14e035 函数中的逻辑就是判断 _0x2a7ea9 的值经过 hash 方法加密后的值，是否与 ct 的值相等，若相等则将返回值传递给 __jsl_clearance_s 参数，循环完后还未有成功匹配的值则会执行第 509 行提示失败，传入参数中 ha 的值是在变化的，即加密算法也是在变化的，有三种加密方式 SHA1、SHA256 和 MD5，我们可以扣下三种 hash 方法，也可以直接使用 crypto-js 库来实现：

```
var CryptoJS = require('crypto-js');  
  
  
function hash(type, value){  
    if(type == 'md5'){  
        return CryptoJS.MD5(value).toString();  
    }  
    if(type == 'sha1'){  
        return CryptoJS.SHA1(value).toString();  
    }  
    if(type == 'sha256'){  
        return CryptoJS.SHA256(value).toString();  
    }  
}  
  
  
var _0x2228a0 = {  
    "mLZyz" : function(_0x435347, _0x8098d) {  
        return _0x435347 < _0x8098d;  
    },  
    "SsARo" : function(_0x286fd4, _0x10b2a6) {  
        return _0x286fd4 + _0x10b2a6;  
    },  
    "jfMAx" : function(_0x6b4da, _0x19c099) {  
        return _0x6b4da + _0x19c099;  
    },  
    "HWzBW" : function(_0x3b9d7f, _0x232017) {  
        return _0x3b9d7f + _0x232017;  
    },  
    "DRnYs" : function(_0x4573a2, _0x3855be) {  
        return _0x4573a2 == _0x3855be;  
    },  
    "ZJMqu" : function(_0x3af043, _0x1dbbb7) {  
        return _0x3af043 - _0x1dbbb7;  
    },  
};  
  
  
function cookies(_0x60274b){  
    var _0x34d7a8 = new Date();  
    function _0x14e035(_0x56cbce, _0x5e5712) {  
    var _0x2d0a43 = _0x60274b['chars']['length'];  
    for (var _0x212ce4 = 0x0; _0x212ce4 < _0x2d0a43; _0x212ce4++) {  
        for (var _0x8164 = 0x0; _0x2228a0["mLZyz"](_0x8164, _0x2d0a43); _0x8164++) {  
            var _0x2a7ea9 = _0x5e5712[0] + _0x60274b["chars"]["substr"](_0x212ce4, 1) + _0x60274b["chars"]["substr"](_0x8164, 1) + _0x5e5712[1];  
            if (_0x2228a0["DRnYs"](hash(_0x60274b['ha'], _0x2a7ea9), _0x56cbce)) {  
                return [_0x2a7ea9, _0x2228a0["ZJMqu"](new Date(), _0x34d7a8)];  
            }  
        }  
    }  
    }  
    var _0x732635 = _0x14e035(_0x60274b['ct'], _0x60274b['bts']);  
    return {'__jsl_clearance_s' : _0x732635[0]};  
}  
  
// console.log(cookies({  
//     "bts": ["1658906704.293|0|YYj", "Jm5cKs%2B1v1GqTYAtpQjthM%3D"],  
//     "chars": "vUzQIgamgWnnFOJyKwXiGK",  
//     "ct": "690f55a681f304c95b35941b20538480",  
//     "ha": "md5",  
//     "tn": "__jsl_clearance_s",  
//     "vt": "3600",  
//     "wt": "1500"  
// }))  
  
// __jsl_clearance_s: '1658906704.293|0|YYjzaJm5cKs%2B1v1GqTYAtpQjthM%3D
```

完整代码  

-------

bilibili 关注 K 哥爬虫，小助理手把手视频教学：https://space.bilibili.com/1622879192

GitHub 关注 K 哥爬虫，持续分享爬虫相关代码！欢迎 star ！https://github.com/kgepachong/

**以下只演示部分关键代码，不能直接运行！** 

完整代码仓库地址：

https://github.com/kgepachong/crawler/

```
# =======================  
# --*-- coding: utf-8 --*--  
# @Time    : 2022/7/27  
# @Author  : 微信公众号：K哥爬虫  
# @FileName: jsl.py  
# @Software: PyCharm  
# =======================  
  
  
import json  
import re  
import requests  
import execjs  
  
  
cookies = {}  
headers = {  
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36"  
}  
url = "脱敏处理，完整代码关注 https://github.com/kgepachong/crawler/"  
  
  
def get_first_cookie():  
    global cookies  
    resp_first = requests.get(url=url, headers=headers)  
    # 获取 cookie 值 __jsluid_s  
    cookies.update(resp_first.cookies)  
    # 获取第一层响应内容, AAEncode 加密  
    content_first = re.findall('cookie=(.*?);location', resp_first.text)[0]  
    jsl_clearance_s = execjs.eval(content_first).split(';')[0]  
    # 获取 cookie 值 __jsl_clearance_s  
    cookies['__jsl_clearance_s'] = jsl_clearance_s.split("=")[1]  
  
  
def get_second_cookie():  
    global cookies  
    # 通过携带 jsluid_s 和 jsl_clearance_s 值的 cookie 获取第二层响应内容  
    resp_second = requests.get(url=url, headers=headers, cookies=cookies)  
    # 获取 go 字典参数  
    go_params = re.findall(';go\((.*?)\)</script>', resp_second.text)[0]  
    params = json.loads(go_params)  
    return params  
  
  
def get_third_cookie():  
    with open('jsl.js', 'r', encoding='utf-8') as f:  
        jsl_js = f.read()  
    params = get_second_cookie()  
    # 传入字典  
    third_cookie = execjs.compile(jsl_js).call('cookies', params)  
    cookies.update(third_cookie)  
  
  
def main():  
    get_first_cookie()  
    get_third_cookie()  
    resp_third = requests.get(url=url, headers=headers, cookies=cookies)  
    resp_third.encoding = 'utf-8'  
    print(resp_third.text)  
  
  
if __name__ == '__main__':  
    main()
```

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/iabtD4jabia4IQ0KuGaLtko0OtDicjUiaibualkmdtQMyz6p7sYmSv9g7oJGEZiaPxRia4jrTNEMumFQW5TqDiae8qoM1g/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)