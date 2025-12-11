> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/62tutWJrY9jwEDmS6xfXSg)

点关注，不迷路！

> 本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！若有侵权，请联系作者立即删除！

### 前言

最近写了一套 `懒人AST解混淆框架`，无需用户手动编写ast代码，只需要引入封装好通用ast插件，真正做到：**不懂ast，也能轻松解混淆！**

出于学习的目的，今天就使用这套框架，来分析一下某多多 `anti-content` 如何解混淆的，希望能帮助到有需要的小伙伴！

`懒人AST解混淆框架`：https://blog.csdn.net/studypy1024/article/details/154188036

因为之前写过一篇类似的，所以我这里就不过多介绍如何定位找位置的了，不懂的小伙伴可以看下以下文章：

某多多 anti-content 签名算法分析记录 | 扣 webpack 代码实战

这篇文章主要聊下，如何解混淆，以及纯算分析。

### 前置分析

先找到生成位置，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibLJf42iampkagC2nic75tWic7dA70pDxhxOHOcR39NmktnouibMc8Q3oJqw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0 "null")某多多 anti-content纯算、anti_content纯算

翻到文件最上方，发现它是通过 `webpack` 打包加载的。如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibhqhH51b6bngRozvz6jL3OsaZlXhxWVF0FrzM4JKibm6ibvXGkBo6b1icQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1 "null")某多多 anti-content纯算、anti_content纯算

通过 `webpack自动扣代码框架` 扣出代码，然后将其放到 `浏览器代码片段` 里执行一下，是否能正常得到结果呢？

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibYgGgYm73c6B8KahPcBCHmaQFy8icS2PrIX5ZUlpicCjACGTKkNB0mJ4w/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2 "null")某多多 anti-content纯算、anti_content纯算

可以看到，执行后，是可以正常得到结果的。另外，文件行数大概 `3700` 多。

下面就是解混淆环节了。先观察代码，找到解密函数的位置。经分析，发现解密函数有 `3` 个。

且`字符串大数组`在函数的外部，考虑到用 `AST通用插件`不方便找位置，我们可以手动将置换后的`字符串大数组`放到解密函数内，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIib6Z8vpicxGLzheQZP7CanfvXyTNPrjbnbD27nvtc5jJC8OmW1HagmWOg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3 "null")某多多 anti-content纯算、anti_content纯算

用同样的方法，把另外两个也手动改下，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibicicQG1s9FaufnqjyTX4yoBic8bZd7pUY9aZZEn2J93YcUhzHUuxz8bJg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4 "null")某多多 anti-content纯算、anti_content纯算![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibN0EARuBJC1JTZQI71AbuWGNwOEeZPjzKJ2DesW22UoulZiaN56E0s4Q/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5 "null")某多多 anti-content纯算、anti_content纯算

### AST解混淆实操演示

#### 逗号表达式还原

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibZfgPVft9JFgJyONISIqyqsLoKyMTHOia8eJgCcpc8gib7YvI8icvDHTEg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6 "null")某多多 anti-content纯算、anti_content纯算

可以看到代码里，充斥大量的逗号表达式代码，阅读吃力，首先可以用 `SequenceRestore` 通用插件，将其还原，如图所示：

```
`traverse(ast, SequenceRestore);`
```

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibbloomnHHPrqQRyiarPtvlbUkGOcBAibtKaQlkrsyXZ4eicbxrJsib65yYw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7 "null")某多多 anti-content纯算、anti_content纯算

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIib8iaUTdDdDsnEsrQbRtfVMkWwMxYPPMNDluMu1eKYO1Uf3YVNviaZkicUA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8 "null")某多多 anti-content纯算、anti_content纯算

#### 函数别名替换

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibEkhHZ92t1W7MTA7DZ96gTHiafiaUTK4qoNAbR38WApX4mE0Ns1PPT5RQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9 "null")某多多 anti-content纯算、anti_content纯算

就到上述情况，就要用到 `FunctionAliasReplace` 函数别名替换插件了，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibNPKzSaAuAYYs67VWAD7JKAvIiaSsPcwgxxBVsnfdlibgU9p9h7YVondg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10 "null")某多多 anti-content纯算、anti_content纯算

```
`traverse(ast, SequenceRestore);  
traverse(ast, FunctionAliasReplace);`
```

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibnwQTqoTKDPnicFTCq9IjJgDzO041rz7yiaGmeXrZ9jBIoUwXsu1OTIFQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11 "null")某多多 anti-content纯算、anti_content纯算

发现 `t` 函数的调用，也都换成解密函数 `u` 了。

#### 函数计算

一切准备工作就绪后，就可以执行解密函数了。此时，只需用 `FunctionCalcAdvanced` 通用插件即可。

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibb4ib4faibichBaF09z4uWVktyp150ISVoDevy1xxEgnP0rae4HEPYjklw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12 "null")某多多 anti-content纯算、anti_content纯算

```
`traverse(ast, SequenceRestore);  
traverse(ast, FunctionAliasReplace);  
traverse(ast, FunctionCalcAdvanced);`
```

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibbmjfADH2SIaibnunG4lS8vvYMPk0u7U07iaCvlPOjJRYEdiaMgOicwV5ibA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13 "null")某多多 anti-content纯算、anti_content纯算

可以看到，函数混淆已经全部还原了。

#### 块语句还原

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibZhzicWNyFHwTsnVSKgVcP5ibze9fLibZve3WccibY1fByUtUxDIP39q3vQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14 "null")某多多 anti-content纯算、anti_content纯算

发现有些 `for` 、 `if` 并不是常见的大括号块语句包裹代码，阅读起来也不方便，此时，可以用 `forIfBlockRestore` 块语句还原插件，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibJxvl1DXtyNOoZupgREBTUdMtMvg5wdL0vHf7ppCptD6hl4Acn5ibnmQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=15 "null")某多多 anti-content纯算、anti_content纯算

```
`traverse(ast, SequenceRestore);  
traverse(ast, FunctionAliasReplace);  
traverse(ast, FunctionCalcAdvanced);  
traverse(ast, forIfBlockRestore);`
```

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibtLTIaWqFiciaE0uwbXVSMdgoC9ounfWZzJnDKNeVKqJPBeROKiaXfAzDA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=16 "null")某多多 anti-content纯算、anti_content纯算

可以看到，代码块语句已经全部还原了。

#### 对象属性合并

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibgIQxczzTaIJ5BI8Yew4073HJXWG4PKQs2vZQB7IIazHfffZoAI2lkw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=17 "null")某多多 anti-content纯算、anti_content纯算

发现 `t` 初始值是一个空对象，后续不断填充属性值。遇到这种请求，可以将所有属性值合并到一起，可以使用 `ObjectMergePropertiesRestore` 通用插件，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIiboZAaMIlOK8h3gwHuxn01Lc8LwTnmicibNNxJSAHlMxw6TW5TxmMWd8tQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=18 "null")某多多 anti-content纯算、anti_content纯算

```
`traverse(ast, SequenceRestore);  
traverse(ast, FunctionAliasReplace);  
traverse(ast, FunctionCalcAdvanced);  
traverse(ast, forIfBlockRestore);  
traverse(ast, ObjectMergePropertiesRestore);`
```

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibQ0TuQ6ZcVszzkZWZy9Adia3ZQDKCS4kjQDRm2hcxicuzsSCnPMXjMhhw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=19 "null")某多多 anti-content纯算、anti_content纯算

#### 移除变量重复定义

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibLVpwV4ZfoR9lWqKZwBkrwtuFDgTnx85hFIJV9raaSPtAqtFkGPeEyg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=20 "null")某多多 anti-content纯算、anti_content纯算

将 `t` 函数合并后，发现 `var n = t;` , 它将 `t` 赋值给 `n` 对象，后续也是 `n` 对象调用执行的。遇到这种情况，可以用 `CleanVariableDefinition` 通用插件，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIib5weGc4akGBhv0ZL4tTGV9UXGwGcHYF6n2ibb8Z1qLyIxCZsnTs4vfpg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=21 "null")某多多 anti-content纯算、anti_content纯算

```
`traverse(ast, SequenceRestore);  
traverse(ast, FunctionAliasReplace);  
traverse(ast, FunctionCalcAdvanced);  
traverse(ast, forIfBlockRestore);  
traverse(ast, ObjectMergePropertiesRestore);  
traverse(ast, CleanVariableDefinition);`
```

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibB0MicCMiaUpJyTlzDicfYatb7AM7N6nTMulB64tzBL1SmaInWjKM5ZuIw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=22 "null")某多多 anti-content纯算、anti_content纯算

可以看到，变量重复定义已经全部移除了，同时，`n` 对象调用也相应的更换成了 `t` 对象调用。

#### 对象属性还原

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIib9hyqhNEyw7LDhddwHxvYFzGPkpibDFWwaCzcCaia1UaCcxIg7EN52Bmw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=23 "null")某多多 anti-content纯算、anti_content纯算

既然有了对象以及对象的调用，那就可以使用 `ObjectPropertiesRestore` 对象属性还原通用插件，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibIdOu4YYYWZvQDZKUUGtlPxEKlZ65sEcDOicv1KnC9Yt0zHHpR08oKjg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=24 "null")某多多 anti-content纯算、anti_content纯算

```
`traverse(ast, SequenceRestore);  
traverse(ast, FunctionAliasReplace);  
traverse(ast, FunctionCalcAdvanced);  
traverse(ast, forIfBlockRestore);  
traverse(ast, ObjectMergePropertiesRestore);  
traverse(ast, CleanVariableDefinition);  
traverse(ast, ObjectPropertiesRestore);`
```

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIib7tIIrNPS2OwG7AC8ibrFF3H4Jo0GZNNiaVIwKgGXjHOkAmPhUA16Bzgg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=25 "null")某多多 anti-content纯算、anti_content纯算

可以看到，对象属性已经全部还原了，此时代码逻辑已经很清晰了。

#### 变量还原

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibiaiar5NwWxQBrMRTvHlamf4C2kEEuv0YJvTHIPdYnSqLGhAQpMmPjlPw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=26 "null")某多多 anti-content纯算、anti_content纯算

发现代码中也有很多变量，看代码逻辑时，还找找到定义的地方看，非常不方便，此时，可以用 `VariableRestore` 变量还原通用插件，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibdHW6tosjic0lQxqZwnWzF1xE5oaSwjap3ZDccFm29OrFpVTQxcGeTibw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=27 "null")某多多 anti-content纯算、anti_content纯算

```
`traverse(ast, SequenceRestore);  
traverse(ast, FunctionAliasReplace);  
traverse(ast, FunctionCalcAdvanced);  
traverse(ast, forIfBlockRestore);  
traverse(ast, ObjectMergePropertiesRestore);  
traverse(ast, CleanVariableDefinition);  
traverse(ast, ObjectPropertiesRestore);  
traverse(ast, VariableRestore);`
```

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibicibu5IzEZicplibWgUoxWibXAxAy72s9ENZClEibHQbDLu548uV5hXj4pQg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=28 "null")某多多 anti-content纯算、anti_content纯算

可以看到，变量已经全部还原了，此时代码逻辑已经很清晰了。

#### 调用表达式还原

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibEVTaqduIT2q7UEicUWU8JTZQwkD6nKw8kgudXQ8VxaqsTzOHvh0iaMpQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=29 "null")某多多 anti-content纯算、anti_content纯算

一般在控制流代码处，会有类似的链式表达式调用，为了后续控制流平坦化还原做准备，此时，可以用 `CallExpressionRestore` 调用表达式还原通用插件，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIib4ib51qg7XZfP7tOZXVvcQW8lrqpiafol6icjpOXnGSuOG5tszpmAO68JA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=30 "null")某多多 anti-content纯算、anti_content纯算

```
`traverse(ast, SequenceRestore);  
traverse(ast, FunctionAliasReplace);  
traverse(ast, FunctionCalcAdvanced);  
traverse(ast, forIfBlockRestore);  
traverse(ast, ObjectMergePropertiesRestore);  
traverse(ast, CleanVariableDefinition);  
traverse(ast, ObjectPropertiesRestore);  
traverse(ast, VariableRestore);  
traverse(ast, CallExpressionRestore);`
```

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibVD1Mu3uzCbL7CgmCD4pqXaTaVia5owNyyMDJZ3VibQEzW7aaqs2lA22A/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=31 "null")某多多 anti-content纯算、anti_content纯算

可以看到，调用表达式已经全部还原了。

#### 控制流ForSwtich还原

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibuT1IyUnUdChoHbGsA9MBGIOzkibvwGZzjnfV5CZmyuz2Dk3vbaUPUjg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=32 "null")某多多 anti-content纯算、anti_content纯算

代码也有很多类似的控制流代码，此时，可以用 `ControlFlowForSwtichRestore` 通用插件将其还原，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIib3mYuxXiaRbiaE96VlHBIzBDCrWetl9t9TFkWP98d2wLnVpKatQaSM0XQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=33 "null")某多多 anti-content纯算、anti_content纯算

```
`traverse(ast, SequenceRestore);  
traverse(ast, FunctionAliasReplace);  
traverse(ast, FunctionCalcAdvanced);  
traverse(ast, forIfBlockRestore);  
traverse(ast, ObjectMergePropertiesRestore);  
traverse(ast, CleanVariableDefinition);  
traverse(ast, ObjectPropertiesRestore);  
traverse(ast, VariableRestore);  
traverse(ast, CallExpressionRestore);  
traverse(ast, ControlFlowForSwtichRestore);`
```

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibMe0ef9icicrxycIs3rWsc76RUOXfvjjR861U1JFqyppYJK8aIjNMnxhw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=34 "null")某多多 anti-content纯算、anti_content纯算

可以看到，控制流代码已经全部还原了，阅读起来就非常方便了。

#### 移除未引用的函数和变量

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIib4Rria91S7nWgkBrFXicAcxvYibYxmbPu9kVFsKsSVNmTa22878ohDZk4A/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=35 "null")某多多 anti-content纯算、anti_content纯算

代码中有很多类似未引用的函数和变量，可以清理一下，让代码看起来更简洁，此时，可以用 `CleanUnreferVarsFuns` 通用插件，如图所示：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibEJDVrAA7umFycAnxz6psjbSH8rZUkHFgLnFyZjIxQ8S6xNwJXDkB4w/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=36 "null")某多多 anti-content纯算、anti_content纯算

```
`traverse(ast, SequenceRestore);  
traverse(ast, FunctionAliasReplace);  
traverse(ast, FunctionCalcAdvanced);  
traverse(ast, forIfBlockRestore);  
traverse(ast, ObjectMergePropertiesRestore);  
traverse(ast, CleanVariableDefinition);  
traverse(ast, ObjectPropertiesRestore);  
traverse(ast, VariableRestore);  
traverse(ast, CallExpressionRestore);  
traverse(ast, ControlFlowForSwtichRestore);  
traverse(ast, CleanUnreferVarsFuns);`
```

还原后：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIib3flqyBACdk5zAr95lvJFoYYphuyMdUdNl392PmCdiaE6Bib8ibqKyYiaZg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=37 "null")某多多 anti-content纯算、anti_content纯算

解混淆后，代码缩减到了 `2400` 多行，运行之后，也依然可以得到结果。说明AST通用插件没问题。

解混淆只需要引入相应的通用插件，就能快速还原混淆代码，大大节省了逆向的时间。

### 纯算分析

搜索 `0aq`，就能定位到生成位置，打上断点，如下：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibKJq4fhq0MdweaXVpMUcSqLmoibUTE3LibnnYZiaRmtibTtTwnO5nVWqsRw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=38 "null")某多多 anti-content纯算、anti_content纯算

它有两部分组成，其中 `0aq` 是头部标识，是固定的。但是不同的平台标识可能不一样，比如：`0as`、`0ap` 等等。

参与运算的函数有两个，`encode`、`budget` 都在 `d` 对象里。找到位置如下：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibbhvEGTAcnTSIQt9AuW6icz2v2Zp7B6hlURez6nXZVxeIvqUibCiciaqDHw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=39 "null")某多多 anti-content纯算、anti_content纯算

函数有了。那接下来看加密数据从哪里来？

先从简单 `te` 数组的开始，如下：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibhdWUY6lvwsRwI8rn3BSgib4EUpGibeAH2iakK0icNJtVezKZKBJEr8aEibw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=40 "null")某多多 anti-content纯算、anti_content纯算

找到它赋值的地方，如下：

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibibcYNLOZydxttJMBacaGyAT0dTJRpHURcUbE8jGEdLvcW7mC8VxTyHw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=41 "null")某多多 anti-content纯算、anti_content纯算

发现它校验了代码的 `toString`，然后生成 `[92, 134, 175, 140]` 字节数组，然后通过 `String["fromCharCode"]` 转成字符。

显然这个 `[92, 134, 175, 140]` 可能是不对的。为啥呢？

因为是解混淆后的代码 `toString` 得到的。最好去未解混淆的代码中生成，后续固定即可。

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibaicBvVoKAv1P6SE4qqxOwB1d1aZuWkCI4icW3WIBKT17k6wGPoKg9vHQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=42 "null")某多多 anti-content纯算、anti_content纯算

这里的 `m` 环境数组，又是如何来的呢？

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibeCxLEAInA83eDuzYbWO9p2YqR2hHFdLzib5uV19PKs52NpUibacMOgIA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=43 "null")某多多 anti-content纯算、anti_content纯算

从 `s` 对象的方法可以推断出，它使用的 `pako` 库中的 `deflate` 方法。

这个方法的作用也很明显，就是将数组进行压缩，方便数据传输。

这里将长度从 `323` 压缩到了 `283`，下面我写个例子，如何引入 `pako` 库，如下：

```
`// 准备原始数据  
const originalData = "这是一个用于测试压缩的字符串，包含一些重复内容以展示压缩效果。";  
console.log("原始数据:", originalData);  
console.log("原始数据长度（字节）:", new TextEncoder().encode(originalData).length);  
  
// 使用pako.deflate进行压缩  
const compressedData = pako.deflate(originalData); // 返回Uint8Array  
console.log("压缩后数据:", compressedData);  
console.log("压缩后数据长度（字节）:", compressedData.length);`
```

当然了，如果不想使用第三方 `pako` 库。扣代码也是不难的，毕竟都解混淆了。

OK，那继续...

这些数组又从哪里来呢？

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibsvibDGI0HdHYunHkag1kialvJMF3CBhUJmhPlguqS49mqPxAYEcu7Gmg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=44 "null")某多多 anti-content纯算、anti_content纯算

可以看到，`i` 的值是通过多个对象中 `packN` 方法调用得到的。

由于，对象 `packN` 方法比较多，所以，我就举例说下方法，剩下的自己按照方法去分析就行。

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibM7GXfNL1My2ibg9ERYgwa6vKCkt0ZWmLicb9k2TSk7SFicsbvwXpShhOA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=45 "null")某多多 anti-content纯算、anti_content纯算  
![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibd64vteLrIw2fkWBgrSFmN4ictjwWLr0ljpuaPNPQRBumoLtKicwh4FMg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=46 "null")某多多 anti-content纯算、anti_content纯算

比如这个，可以在方法里断住，既然 `re["location"]["href"]` 是 `https://xxx.com/`，根据提示，将相应变量替换成字符串即可。

![某多多 anti-content纯算、anti_content纯算](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4MctQU9uGb1WTehnUQj9vXmIibIlwWicRm0mpcQv2cA0OcfpOYYGibPmkUZTQBelictp3Sptu24hD5RvUicw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=47 "null")某多多 anti-content纯算、anti_content纯算

大部分都差不多，大部分都是环境相关的，不算很难，但是对象中 `packN`比较多，需要多断点调试，比较耗时。

我把解混淆后的js贴到星球，仅供学习使用，严禁其他用途！剩下的自己分析即可。

### 写到最后

逆向难或不难，关键是看你是否有耐心，是否有归纳总结能力，是否有趁手的工具。

*   • 有**耐心**，能让你有问题不后退!
    
*   • 有**归纳总结**，能让你知识成体系！
    
*   • 有**工具**，能让你事半功倍！
    

星球有大量逆向工具，比如：`webpack自动扣代码框架`、`插桩日志框架`、`懒人AST解混淆框架`、`逆向调试工具浏览器插件`等等；干货有点多，绝对值得你拥有！

一个人或许很难坚持，但一群人，绝对能走的更远！欢迎加入我们，一起学习，一起进步！

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4Mcu6lQT1qme8ia3VQr0lZ6Dl0wnsSOAvPE55mU0nQVrqib5tDWf56cNQuonclE7MAxmwb6O9oiaib7z3XQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=48)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/vxZ2DaY4McuvCjebbu0lQ6QG7z4GPITIOeA4xwbziamuNLMzuRfMhxibn2053XSp2StOxgR2OphVQj2Mxv953KFg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=49)