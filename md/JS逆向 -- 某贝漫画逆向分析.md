> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/kisr4iTofspNC7xcdy67Fw)

某贝漫画逆向分析
--------

> 目标网址：  
> 反调试讲解：'aHR0cHM6Ly93d3cubWFuZ2Fjb3B5LmNvbS8='  
> 请求章节响应解密讲解：上面那个网址随便点进去一个漫画的详情页（这里你可以尝试我上面讲解的过反调试的手段哦）

**_本文仅供学习交流，因使用本文内容而产生的任何风险及后果，作者不承担任何责任，一起学习吧_**

**_学学就好，别攻击别人！！！！！！！！！_**

### 文章摘要 and 作者低语

1.  如何分析反调试手段
    
2.  常规逆向的基本流程和手段
    
3.  算法还原
    

### 反调试讲解

进入当前页面，我们会发现，我们的调试工具被断住了，如下图1所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhoGpG3dHaczk8mKiaGLEoLx3un0jibT1ibPBLzeAKTeCkkLzIWI3wgekn1w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

  

一旦我们放开断点，当前页面就会变成空白页面：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhoRQ9L6t4u5Ltu9LOicD1tTbKuLv7Q6FyFdHZLTRKMpsBbiaopdf9UxQtw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

  

  

通过图一我们看到，当前堆栈处于loop,那我们不妨把目光看向这个函数。（在当前空白页回退上一页就可以找到了）如下图3：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhodsNWDfAfTScId3BCSlwjHwenb4kTIPkQlPf5aZCa5t5XJUeiaeMLiaXw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

  

  

这里你可以会疑问，欸？utf8(作者)为什么你图一是堆栈的源代码是空白的，而图三不是呢。

如果你注意到了这点，那么恭喜你你有一个善于观察的眼睛。**这里是因为我们打开开发者工具的时机导致的，我们一开始是先打开网址后打开开发者工具，这导致当前网址的部分文件我们没有缓存，也就是没有被开发者工具捕获。我们在空白页回退，让这个页面资源重新加载，开发者工具又重新捕获到了资源文件，所有就有了，这是一个常见的坑**

言归正传，让我们来逐行详细分析这个函数：

```
 _复制代码_ _隐藏代码_`// 立即执行函数，创建一个私有作用域防止变量泄漏到全局  
((function () {  
    // 存放检测到调试行为后要调用的回调函数  
    var callbacks = [],  
  
        // 设置调试中断允许的最大时间，单位为毫秒  
        timeLimit = 50,  
  
        // 标记是否已经触发过一次调试检测  
        open = false;  
  
    // 每 1 毫秒执行一次 loop 函数，用于反调试检测  
    setInterval(loop, 1);  
  
    // 返回一个监听器注册接口  
    return {  
        // 提供注册回调的功能，外部可以添加要在调试时执行的函数  
        addListener: function (fn) {  
            callbacks.push(fn);  
        },  
  
        // 提供移除回调的功能  
        cancleListenr: function (fn) {  
            callbacks = callbacks.filter(function (v) {  
                return v !== fn;  
            });  
        }  
    };  
  
    // 反调试核心检测函数  
    functionloop() {  
        // 获取当前时间戳  
        var startTime = newDate();  
  
        // 如果调试工具开启，debugger 会中断此处执行  
        debugger;  
  
        // 判断执行完 debugger 后消耗的时间是否超过限制  
        if (newDate() - startTime > timeLimit) {  
  
            // 如果是第一次检测到调试器  
            if (!open) {  
                // 执行所有注册的反调试回调函数  
                callbacks.forEach(function (fn) {  
                    fn.call(null);  
                });  
            }  
  
            // 设置标志为 true，防止回调多次调用  
            open = true;  
  
            // 通常是跳转页面到 about:blank，使页面内容彻底消失  
            aboutBlank();  
  
        } else {  
            // 如果没有检测到调试器，重置标志  
            open = false;  
        }  
    }  
  
})())  
// 注册一个监听器函数：在检测到调试器时执行 aboutBlank()  
.addListener(function () {  
    aboutBlank(); // 页面跳转到 about:blank  
});  
`
```

知己知彼，百战不殆。`loop`是干扰我们调试的关键函数，`setInterval`又是反复监控。我们首先想到的方法就是直接替换文件，我们将`loop`函数和`setInterval`函数给注释掉不就行了，我们尝试一下，发现该方法完全可行。

到这里我们就通过了反调试（其实这个通过手段有特别多，你还可以直接hook函数，关键点在于你对反调试代码的理解，我上面对这个代码分析的很详细，相信你可以也可以想出其他的方法）。

### 请求章节响应解密讲解

> 目标接口： `comicdetail`

定位接口之后，简单查看，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhoI6fy9Ervj8mAM4lFfFqwPxHCNDUrAszN9jPgUIRgA6t9Jo0EJv1XSQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

直接查看请求调用堆栈，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhoIWqDxHRMF4kibczdNQGFezSOUGRUw7HceMq4WXqPGDTDV0ZClJxj6fw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

  

  

看到了我们的老朋友，`eval`,这个是逆向加密常用的手段。小萌新可以先记一下小本本。

为了从头展示逆向过程，我们直接进入第一个堆栈，打上断点（xhr断点同理），如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhooA8LsWtA0xqCLNMt8BITEicoX1LLMcOME8VIUKg0QV7oA8xJPS5G5Gw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

  

  

欸，这代码不是人看的，那么恭喜你找到加解密的入口了(代码越抽象，越是在掩饰什么)，这一看就是个ob混淆，给大家讲一下这种混淆的特点：

基于数组映射字符串表的字符串混淆（Array mapping obfuscation）

1.  字符串集中存储：所有字符串（变量名、类名、接口路径、HTML代码等）统一存储在一个数组中（如 var _0xabc = [...]），避免明文出现。
    
2.  索引访问替代明文：所有原始字符串通过一个映射函数（如 _0xfunc(index)）来访问，实现对原始代码的“抽象化引用”。
    
3.  配合数组洗牌：通过 shift()/push() 组合“旋转”数组位置，使索引顺序与原始定义不一致，进一步提升混淆性。
    

如果你会ast那么你可以先还原这个代码，后续调试就会简单方便很多。我这里不展示ast方法直接分析(~绝对不是我不会~)

#### 详细逆向过程

逆向，首先我们要找准我们的切入点是什么？--> 这个是核心思路，这个也是后续你快速分析，快速定位的方法

本次案例我们不讲快速定位，老老实实，采用笨方法，一步步跟，主要是掌握跟栈，技巧是你基本功熟练之后自我总结的，我现在直接告诉你快速定位你是不利于你学习的。关注核心在于**入参口**

最初断点，断下信息如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhoaH7fMJwic2taBI4Jr0oV9YNANjcuVeNiaJoYEF81y1ASV0gHrLMpoe8A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

  

  

要记住一点，响应体解密必然是在请求发出之后才开始(很合理)。所以，在这个断点下，我们老老实实一步步跟(F9)。欸，突然我们发现一个关键点！如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhoBSnNvcCYxPic1G7JviciauW7uHCdj4NPSh4RSmCwW75K69SwpBNmKGoPw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

  

  

这个**key**，有可能是是解密的关键一步，也可能不是，这个时候你可以选择保存一下这个常量，万一后面遇到了呢，复制一下(你可以把调试过程中所有你认为的关键参数记下来)，接着老老实实的跟(F9)

功夫不负有心人啊，终于我们又进入这个混淆文件：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhoib6IibMfibS6PkCnPxe0sdeVAfhQZn3BgeIgK8gaZvGM3OgAzj5lSgHCQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

  

  

但是还是没有看到入参，不急，接着(F9)，欸，好像看到了希望

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOho0kHpFapzszRoY76BXt8IW7gALGvuicGicWqa6z2pqtVrK7694kEvdsJg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

  

  

在这里打上断点记录一下，我们接着F9,看到入参进入函数：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOho7WsLo5xbjnU2AwFckKqIvF5iclyeZo5jruuHtG90RiaxTMgQdyalB8OA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

  

  

跟进去，我们的响应参数进入到这个函数，代码如下：

```
 _复制代码_ _隐藏代码_`  
function_0x468bc8(_0x5d7669) {  
                        0xc8 !== _0x5d7669['code'] ? (_0x48fc84['innerText'] = '加載失敗，點擊重新加載',  
                        '加載失敗，點擊重新加載' === _0x48fc84[_0x4e8a('0x54')] ? _0x48fc84['onclick'] = function() {  
                            _0x45ef84()  
                        }  
                        : _0x48fc84[_0x4e8a('0x2')] = function() {  
                            returnnull  
                        }  
                        ) : function(_0x55a4d4) {  
                            var _0x2e033c = _0x522e0c(0x6)  
                              , _0x5e1413 = _0x55a4d4  
                              , _0x5bf283 = _0x5e1413[_0x4e8a('0x21')](0x0, 0x10)  
                              , _0x37be9a = _0x5e1413[_0x4e8a('0x21')](0x10, _0x5e1413[_0x4e8a('0x11')])  
                              , _0x3def88 = _0x2e033c[_0x4e8a('0x59')][_0x4e8a('0x8')][_0x4e8a('0x5c')](dio)  
                              , _0xee2319 = _0x2e033c[_0x4e8a('0x59')][_0x4e8a('0x8')]['parse'](_0x5bf283)  
                              , _0x2fa5a9 = function(_0x5f5c34) {  
                                var _0x2f96b7 = _0x2e033c[_0x4e8a('0x59')][_0x4e8a('0x39')][_0x4e8a('0x5c')](_0x5f5c34)  
                                  , _0x57ae89 = _0x2e033c[_0x4e8a('0x59')][_0x4e8a('0x0')]['stringify'](_0x2f96b7);  
                                return _0x2e033c[_0x4e8a('0x66')]['decrypt'](_0x57ae89, _0x3def88, {  
                                    'iv': _0xee2319,  
                                    'mode': _0x2e033c['mode'][_0x4e8a('0x28')],  
                                    'padding': _0x2e033c[_0x4e8a('0x61')]['Pkcs7']  
                                })[_0x4e8a('0x6')](_0x2e033c['enc']['Utf8'])[_0x4e8a('0x6')]()  
                            }(_0x37be9a)  
                              , _0x14dbee = JSON[_0x4e8a('0x5c')](_0x2fa5a9)  
                              , _0x4a81be = document[_0x4e8a('0x27')](_0x4e8a('0x34'))  
                              , _0x33c653 = (window['location'][_0x4e8a('0xb')],  
                            document[_0x4e8a('0x27')]('.wargin'));  
                            void0x0;  
                            var _0x11cda6 = _0x14dbee['build']  
                              , _0x2e5e7f = _0x14dbee[_0x4e8a('0x3')]  
                              , _0xae5052 = {}  
                              , _0x1ba770 = function_0x4f55e7(_0x207933) {  
                                var _0x32bc38 = _0x2e5e7f[_0x207933]  
                                  , _0x5a1440 = _0x32bc38[_0x4e8a('0x68')]  
                                  , _0x42793b = []  
                                  , _0x586df9 = []  
                                  , _0xf23d1 = [];  
                                _0x5a1440['forEach'](function(_0x2854c5) {  
                                    switch (_0x2854c5[_0x4e8a('0x26')]) {  
                                    case0x1:  
                                        _0x42793b[_0x4e8a('0x53')](_0x2854c5);  
                                        break;  
                                    case0x2:  
                                        _0x586df9[_0x4e8a('0x53')](_0x2854c5);  
                                        break;  
                                    case0x3:  
                                        _0xf23d1[_0x4e8a('0x53')](_0x2854c5)  
                                    }  
                                }),  
                                _0xae5052[_0x207933] = {  
                                    'groups': {  
                                        '全部': _0x5a1440,  
                                        '話': _0x42793b,  
                                        '卷': _0x586df9,  
                                        '番外': _0xf23d1  
                                    },  
                                    'path_word': _0x32bc38[_0x4e8a('0x3b')],  
                                    'name': _0x32bc38[_0x4e8a('0x40')],  
                                    'last_chapter': _0x32bc38[_0x4e8a('0x63')]  
                                }  
                            };  
                            for (var _0x20e11d in _0x2e5e7f) {  
                                _0x1ba770(_0x20e11d)  
                            }  
                            for (var _0x5eb3f7 in _0xae5052) {  
                                var _0x9cee20 = !0x0  
                                  , _0x4ef6c0 = document[_0x4e8a('0x5b')](_0x4e8a('0x1d'));  
                                _0x4ef6c0[_0x4e8a('0x5f')] = _0xae5052[_0x5eb3f7]['name'],  
                                _0x4a81be[_0x4e8a('0x24')](_0x4ef6c0);  
                                var _0x19b0b0 = _0xae5052[_0x5eb3f7]  
                                  , _0xebab21 = document[_0x4e8a('0x5b')](_0x4e8a('0x32'));  
                                _0xebab21[_0x4e8a('0x48')] = 'table-default';  
                                var _0x5da518 = document[_0x4e8a('0x5b')]('div');  
                                _0x5da518[_0x4e8a('0x48')] = _0x4e8a('0x49');  
                                var _0x5957d3 = document['createElement']('ul');  
                                _0x5957d3[_0x4e8a('0x48')] = _0x4e8a('0x46'),  
                                _0x5957d3[_0x4e8a('0x3a')](_0x4e8a('0x4b'), 'tablist');  
                                var _0x4ab2c2 = ''  
                                  , _0x3331db = document[_0x4e8a('0x5b')](_0x4e8a('0x32'));  
                                _0x3331db[_0x4e8a('0x48')] = _0x4e8a('0x14');  
                                var _0x2042a5 = document[_0x4e8a('0x5b')](_0x4e8a('0x32'));  
                                _0x2042a5[_0x4e8a('0x48')] = 'tab-content';  
                                for (var _0x26df19 in _0x19b0b0[_0x4e8a('0x3')]) {  
                                    var _0x33324b = _0x19b0b0[_0x4e8a('0x3')][_0x26df19]  
                                      , _0x3538e9 = document['createElement'](_0x4e8a('0x32'));  
                                    _0x3538e9[_0x4e8a('0x3a')]('id', _0x19b0b0[_0x4e8a('0x3b')] + '' + _0x26df19),  
                                    _0x3538e9[_0x4e8a('0x3a')](_0x4e8a('0x4b'), _0x4e8a('0x3d'));  
                                    var _0x5670ce = document[_0x4e8a('0x5b')]('ul')  
                                      , _0x2bd24c = '';  
                                    for (var _0x33af50 = 0x0; _0x33af50 < _0x33324b[_0x4e8a('0x11')]; _0x33af50++) {  
                                        _0x2bd24c += _0x4e8a('0x51') + _0x11cda6[_0x4e8a('0x3b')] + _0x4e8a('0x38') + _0x33324b[_0x33af50]['id'] + _0x4e8a('0x1') + _0x33324b[_0x33af50]['name'] + _0x4e8a('0xe') + _0x33324b[_0x33af50][_0x4e8a('0x40')] + '</li></a>'  
                                    }  
                                    _0x5670ce[_0x4e8a('0x5f')] = _0x2bd24c;  
                                    var _0x219a7a = document['createElement']('ul');  
                                    _0x219a7a[_0x4e8a('0x48')] = _0x4e8a('0x7'),  
                                    _0x219a7a['innerHTML'] = _0x4e8a('0x3e'),  
                                    _0x9cee20 ? (_0x4ab2c2 += _0x4e8a('0x3c') + _0x19b0b0[_0x4e8a('0x3b')] + _0x26df19 + _0x4e8a('0x36') + _0x26df19 + '</a></li>',  
                                    _0x3538e9[_0x4e8a('0x48')] = _0x4e8a('0x1e'),  
                                    _0x3538e9[_0x4e8a('0x24')](_0x5670ce),  
                                    _0x2042a5[_0x4e8a('0x24')](_0x3538e9),  
                                    _0x3538e9[_0x4e8a('0x24')](_0x219a7a),  
                                    _0x9cee20 = !0x1) : (_0x3538e9['className'] = _0x4e8a('0x1f'),  
                                    0x0 != _0x19b0b0[_0x4e8a('0x3')][_0x26df19]['length'] ? (_0x4ab2c2 += _0x4e8a('0x5') + _0x19b0b0[_0x4e8a('0x3b')] + _0x26df19 + _0x4e8a('0x36') + _0x26df19 + '</a></li>',  
                                    _0x3538e9['appendChild'](_0x5670ce),  
                                    _0x2042a5[_0x4e8a('0x24')](_0x3538e9),  
                                    _0x3538e9[_0x4e8a('0x24')](_0x219a7a)) : _0x4ab2c2 += '<li\x20class=\x22nav-item\x22><a\x20class=\x22nav-link\x20disabled\x22\x20\x20data-toggle=\x22tab\x22\x20href=\x22#' + _0x19b0b0['path_word'] + _0x26df19 + _0x4e8a('0x36') + _0x26df19 + _0x4e8a('0x37'))  
                                }  
                                var _0x2eae1c = document['createElement']('div');  
                                _0x2eae1c['className'] = _0x4e8a('0x52'),  
                                _0x19b0b0[_0x4e8a('0x63')] && (_0x2eae1c[_0x4e8a('0x5f')] = _0x4e8a('0x15') + _0x19b0b0[_0x4e8a('0x63')][_0x4e8a('0x4a')] + _0x4e8a('0x38') + _0x19b0b0['last_chapter']['uuid'] + _0x4e8a('0x2e') + _0x19b0b0[_0x4e8a('0x63')][_0x4e8a('0x40')] + '</a><span>更新時間：</span><span>' + _0x19b0b0[_0x4e8a('0x63')]['datetime_created'] + _0x4e8a('0x22')),  
                                _0x5957d3[_0x4e8a('0x5f')] = _0x4ab2c2,  
                                _0x3331db[_0x4e8a('0x24')](_0x2042a5),  
                                _0x5da518[_0x4e8a('0x24')](_0x5957d3),  
                                _0x5da518[_0x4e8a('0x24')](_0x2eae1c),  
                                _0xebab21[_0x4e8a('0x24')](_0x5da518),  
                                _0xebab21['appendChild'](_0x3331db),  
                                _0x4a81be[_0x4e8a('0x24')](_0xebab21)  
                            }  
                            clickpage(),  
                            activeLink(),  
                            void0x0,  
                            _0x4a81be[_0x4e8a('0x4f')](_0x33c653);  
                            var _0x49f9ae = 0x0  
                              , _0x437599 = setInterval(function() {  
                                _0x49f9ae++;  
                                var _0x4866cb = document[_0x4e8a('0x27')](_0x4e8a('0x1a'));  
                                _0x4866cb && _0x4866cb[_0x4e8a('0x4c')](_0x4e8a('0x60'))[_0x4e8a('0x11')] > 0x0 && (document[_0x4e8a('0x27')](_0x4e8a('0x13'))[_0x4e8a('0x57')](),  
                                clearInterval(_0x437599)),  
                                0xa === _0x49f9ae && clearInterval(_0x437599)  
                            }, 0x3e8)  
                        }(_0x5d7669[_0x4e8a('0x5d')])  
                    },  
`
```

前面是一个很长的三元表达式：

```
 _复制代码_ _隐藏代码_`0xc8 !== _0x5d7669['code'] ? (_0x48fc84['innerText'] = '加載失敗，點擊重新加載',  
                        '加載失敗，點擊重新加載' === _0x48fc84[_0x4e8a('0x54')] ? _0x48fc84['onclick'] = function() {  
                            _0x45ef84()  
                        }  
                        : _0x48fc84[_0x4e8a('0x2')] = function() {  
                            returnnull  
                        }  
                        ) : function(_0x55a4d4) {在这里哦}`
```

我们调试发现，加密参数进入了这个函数，所以聚焦点在`function(_0x55a4d4)`里面，其实到了这里基本上逆向就很明确了，这里你可以直接补环境，也可以直接代码。但是我想带你一步步看看，这里到底做了什么。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOho4fyMZwGTMGOib2Vhd7DiahhoXI2icYpTDFicmKHW6JX406XwQeAwYH2jicA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11)

  

  

我配合着代码一起讲解吧：

```
 _复制代码_ _隐藏代码_`function(_0x55a4d4) {  
    var _0x2e033c = _0x522e0c(0x6)  //初始化加密函数对象  
        , _0x5e1413 = _0x55a4d4  // 入参  
        , _0x5bf283 = _0x5e1413['substring'](0, 16)  // 字符串操作  
        , _0x37be9a = _0x5e1413['substring'](16, _0x5e1413['length'])  // 字符串操作  
        , _0x3def88 = _0x2e033c['enc']['Utf8']['parse'](dio)  // dio 是密钥  
        , _0xee2319 = _0x2e033c['enc']['Utf8']['parse'](_0x5bf283)  
        , _0x2fa5a9 = function(_0x5f5c34) {  
        var _0x2f96b7 = _0x2e033c['enc']['Hex']['parse'](_0x5f5c34)  
            , _0x57ae89 = _0x2e033c['enc']['Base64']['stringify'](_0x2f96b7);  
            // 这里我留点空间给你思考。  
        return _0x2e033c['AES']['decrypt'](_0x57ae89, _0x3def88, {  
            'iv': _0xee2319,  
            'mode': _0x2e033c['mode']['CBC'],  
            'padding': _0x2e033c['pad']['Pkcs7']  
        })['toString'](_0x2e033c['enc']['Utf8'])['toString']()  
    }(_0x37be9a)//后面被我截断了，至于为什么你动动手就知道了  
    }`
```

很清晰了，我们知道这就是一个AES加密，同时模式是CBC,填充是Pkcs7，那么我们随便选择一个解密网址去验证一下我们的想法是否正确？

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhoemYrhppozDFX2EjuypYjK2bbzFNhdurmy0B6jxE0uB47ewAAuVrWLg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12)

  

  

逆向成功，接下来你可以直接让ai给你写出还原代码。他详细图片的加密也是这个流程。这里就不讲了。

#### 小结

这个网址的整体难度不高，属于是入门级别，但是又包含了很多知识点，补环境，webpack，标准算法,轻微混淆。希望看完本篇文章之后，你可以自己去尝试一下，**_！！！严禁网络攻击！！！_**，只有自己做一遍才知道自己是否掌握。为什么说是入门级别呢，因为这个定位确实难度不大。我跟堆栈是为了让大伙更加清楚的了解到底是怎么来的，然后你自己去总结经验，慢慢的，你也就可以快速定位了。初学者我还是强烈建议好好跟堆栈，弄清楚每一步才是学习。

#### 作者骚话

一个礼拜差不多更新了3篇(其实有不少存货，但是写文章确实不容易，可能你认为很简单，对别人来说很复杂)，我希望文章确实可以帮助到大家打开思路(**我不会放成品的**)，如果有不足的地方欢迎大佬们指出。看不懂不怕，多练多问，谁都是从小菜鸡过来的。后续更新看看有什么好玩的网址吧(当然你有好玩的也可以分享给我)，(头部的几家加密不太敢写后面最多提供一下思路)。看看能不能坚持学习，至少每个月更新一篇吧(画大饼ing)，后续可能更新一个`protobuf`的协议逆向

  

**·** **今 日 推 荐** **·**

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOho74rcpXItxzTy4SoKveYvc3hXC9yXTF8EtJpmLj2CzUfA3t0XtAveZA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13)

  

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0oEfL9jHRlABh5fK7xxeOhoNnDHMjBOoibRYOFnXibDbibumcyxwk9rJgejiar99khrxXYSCbZ4pxAibuQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14)

  

本文内容来自网络，如有侵权请联系删除