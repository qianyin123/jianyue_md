> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iQ1ce2A9Oh_hoj_qiW-p2g)

**提示！本文章仅供学习交流，严禁用于任何商业和非法用途，如有侵权，可联系本文作者删除！**

* * *

前言

#### **又是好久没有写文章了，因为一些很多想写的网站都有人写过，所以就不太想再写了（主要还是懒），这篇文章也是断断续续写出来的（为了水一篇文章不容易），介于之前没写过验证码方面的文章，所以干脆来一篇验证码相关的，今天把某盾当做案例讲一下（网上也能查到很多某盾的文章，不过很多都是之前的没有混淆过的版本），本次主要是分析下无感和滑块这两种**

  

### 网站链接：aHR0cHM6Ly9kdW4uMTYzLmNvbS90cmlhbC9zZW5zZQ==

### 无感验证码：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrrFricxtUT6zxpYzSO6UhjhB3fON9REicbCGFn84icNbIiazySLSiaWE8UpQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

进入网站之后先把**Network**中的抓包信息给清掉，然后这三个验证码中随便点一个，这时能发现右边有四个请求，先看看这四个请求都做了些什么。我们能看到第一第二个请求除了callback这个参数，其余请求时携带的参数是一样的，返回的数据也是一样的，这里就先不管，接着看下面的两个请求

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrEibjkDqWes3qtTiattkia2JGUUcP2duibibdz6W2XibAG2NPRdhNaVfDb8wQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

后面两个请求中返回的数据都有一个**token**参数，然后我们再来看下请求时携带的参数  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr4uW82SoXpDSKYGAmicrjx4qDPb5icWSahKrepfFqQVFnTh2km4f7ibLZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图圈起来的部分就是需要破解的参数，这里先记录下，然后直接点击完成校验  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrzQyXZ4gmib0LmbYonb2odlkenoia4TT8lROEmRicdTKgP6fVtWuTgXpFQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里可以发现请求了一个校验的接口，里面返回了一个**token**和**validate**，有经验的同学一眼就能知道**validate**就是验证码通过之后返回的一个值，之后通过**validate**这个值就可以完成一些后续的请求，这里再来看看请求这个接口时携带的参数，下图圈出来的参数都是需要破解的，其余未圈出来的要么是定值要么可以是定值。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrsUxxD33tAE1HPibb4a54JCiaJL3otJvh9nCXvujAoPARzgXOx1upiaybA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

好了到目前为止需要破解的参数都找出来了，**fp、cb、token、acToken、data**，其中**token**这个参数是前面的请求中返回的，所以实际上需要破解的参数只有四个，这里我们先来看第一个参数**fp**，从参数名上应该就能猜到点什么，**fp**可以理解为**fingerprint**的缩写，也就是指纹的意思，所以这个参数应该是和浏览器指纹有关的，我们直接全局搜索**fingerprint**

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Frh3dsoDSpGQZev3cpyZ6E3L7fmEzBclzHZ9NNl3jb2diaCSffF47yrIg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后发现这个文件中有6处，看下其它几处之后发现只有截图的地方打断点比较合适，这里直接打下断点，然后清除下网站数据，之后随便点击一个验证码，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrvLC4jUhdOywZo3YdPFofw0rUdZR48o1cIG9NPrCYvMSm5dEWs8DfKA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里发现**fingerprint**等于**window['gdxidpyhxde']**，这说明这个值在之前就已经生成了，我们搜一下**gdxidpyhxde**发现只有这么一处，不过既然是**window**下的属性，那么我们直接**hook**就可以了，代码入下:

```
(function() {  
    Object.defineProperty(window, 'gdxidpyhxde', {  
    set: function(val) {  
        debugger;  
        return val;  
    }  
    });  
})();
```

刷新页面直接断住  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrGGsEMaichquJFibjTSqJB68a0Imw5g0XlgLWcTDDcPKN3PHjIZMlyTLg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

之后往前跟一下，发现来到如下位置

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Frh7XeZSXVbdX6zibu0LMLBcVEiaDLhtRDfAQ3LRvC40Y87SOiavCAyDvtQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

到这里之后大家自己打上断点单步调试一下，就不细讲了，代码往下拉，直接在如下的地方打上断点，然后释放

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrxpBhLIBdJIu4mECOg8b6Of37J6PLe7481LUNF2b5HiczKdGiaG3f1GWw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里发现参数已经生成了，既然已经找到了参数生成的位置，那么后面就比较简单了，直接开始扣代码即可。大家看我上图截图的都是某盾经过混淆的代码，大家有条件的可以将这份代码通过**ast**进行还原，还原之后再来扣代码就会比较方便，最后扣完代码之后的结果如下:

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrTr4NkA3PqmQuMHHdw1lc0ibKBzLLqbiaSt0Fs7DaynibRrOicWR0q3R4qg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图中圈出来的部分大家自行注意下即可。（**不知道ast是啥的可以自行百度，如果有要想学的可以购买业界大佬蔡老板的星球，大佬公众号里也有相关的文章可以学习，蔡老板公众号：菜鸟学Python编程**）

之后就是第二个参数**cb**，还是直接全局搜索下，这里全局搜索的小技巧，一般由于参数名比较普通，如**m**这种搜索出来肯定一大堆，没法直接定位到参数位置，这个时候可以搜索**"m"、m=、m:**等，这里就是直接搜索**"cb"**，如下图，发现有四个地方，直接全部打上断点。

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrAoGHch5xbyklkTricu46PDPDuibeW0tUJ18urtV4p6jHr3vzfI0DoGKg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

之后重新发起请求，直接断住

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrC0ZLNXPSicX5t7DwVyEaKTTfUaMnP3TfoVcXbK4UoKKwYny6scOC4UQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后这里跟进去，之后再次打上断点，然后释放

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FricEmntbhQBUFw6KzyEfWImD5vMrxsjnLBOPBzAfpYAZZtXseBPNRmQw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

既然已经找到了**cb**生成的位置，很明显接下来就又到了我们最喜欢的扣代码环节，最后扣完之后结果如下:（**注意：这里不想单独扣cb参数的可以先放着，仅仅看下流程就好，后面在扣data参数的时候就直接可以顺带完成cb参数的破解了**）

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrxFFIFe9KhkKkg10ib6ouCtC2wK1uqfFCuciatMMYcAFpiaCeficqeK08lg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

开头的时候我有圈出一个**callback**参数，这个参数不重要，是可以直接写成定值的，这里节约大家时间代码我直接放下面:  

```
function getJSONP(){  
     h ="__JSONP" + "_" + Math.random().toString(36).slice(2, 9) + "_" + 1 //这个1是根据请求次数增加的,直接写成定值就好  
        return h  
    }
```

到这里获取**token**参数的请求里需要的参数都已经破解好了，我们来试一下:

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrAITvjJTAWUXn7rO90S4HvX2vExAMYNPsU8C6sZ4ibNRPeaxkwQD6OqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

成功的拿到了**token**参数，其实如果大家有自己试过的话，就会发现这个接口**fp**参数为空也是可以拿到**token**参数的，而且拿到的这个**token**参数也是可以使用的，这个时候应该有人会问既然**fp**为空也能拿到可以使用的**token**，那么为什么还要花费这么多精力去破解这个参数呢，原因很简单，某盾的无感校验是不严格的，所以这里的**fp**为空是可以成功的，但是后面的滑块验证码部分如果**fp**这个参数为空或者有问题都会导致最终获取不到**validate**这个值的，所以这里将**fp**破解掉也是为了后面省事点。接下来我们看**acToken**参数

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr6eGgTBjpHsAJIwueFQCTp4OVB18zcyLf2cU3SAWJyicOS4PK7uB8JhA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里的这个**acToken**参数，可以通过多次的请求来对照着看，会发现这个值是不变的，此时我们可以搜一下这个值是哪里来的，经过搜索之后可以发现这个值就是上面的接口中返回的数据**token**，所以这个值在这里是不需要破解的，直接写成定值即可。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrQxCoc0Dr1IIklPoJlVe3L1iaXja3vP6XKc214E6Re9FlGOlabMjSI3g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最后再来看**data**参数，首先点击完成校验，发起校验请求，之后鼠标放在该请求的**Initiator**上，然后点击下图中我圈出来的部分即可跳转到**data**参数生成的位置。

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrH0OqufXmJsrYhnkuYMaOeLZcjHashWia9MXqglic84ERiaH6Wpc5AP5wg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

之后在下图所示位置打上断点，然后重新完成一次校验过程，之后断点断住，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr0kibbJ3SWWhJGxPRsbEkSUt8OibMiaPsR6SnQ9TiaXF0EaZYbEmTHkr79Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里我们可以看到**data**参数中有四个参数，**d、m、p、ext**，其中**d**参数为空，所以只需要解决**m、p、ext**这三个参数即可，这里简单的带大家扣一遍代码，先把整个**data**参数扣下来，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrWnzufks3mFnIRcGP6W8Gpb9kN3MhNxdnDBMzz529bycjiajU0qEgj0g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后为了增加代码可读性，大家可以先将变量名还原（还是那句话，有条件的可以使用**ast**将整个文件还原，这样方便分析）,直接在控制台输出这些变量名即可，如下图所示 :

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrTpeicGPkfst8M89pfqwxIhmrwgrPPs9lCKE0hQxslYYAKPKx4tHpA4Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

之后在代码中进行替换，如下图所示：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrY3vm5VmaVjaqr6g4GoHlm6LGHpcGdXFlCsumzibHQfjGG2AXvnLbYNg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后运行代码，之后会报如下错误：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FricZLX4NnWhufCuLBtRwicpBElqY1QqRuYicolkpkzb04pgSbUufPqT1Fw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个时候开始缺啥补啥即可，直接鼠标放在函数上，然后点击圈出来的位置跳转到函数位置，也可以直接在控制台输出该函数，然后点击跳转也行，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrBFTL1TrPf7trVR0RQicEpTCBXhkmNExRIfvkhe8nCgQS7Y5fiaIicdF1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里说一下这部分代码怎么扣比较好，首先我们先把解密函数扣出来（在整个文件的最上面），下图中我们抠出来是右侧文件这样的，这里会报错，所以改成左侧这样，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrJoiaFfZGuLdbibhiadrOejrWO7TG9AGrWx49rgkfy0O4aDwnOZ9hSQjjw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

之后跳转到要扣的第一个函数的位置，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrUkEEr3ZsBjWibYiaERCc3fZqpiaDjFKpVe8mIOa9uRevDcH8VibJQXJ7ag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里直接将代码拉上去，将整个函数内的代码全部扣下来，如下图所示：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrEw3mHoaMWXnYKz2p8zK0PicoO4UePH3xBm24njXAXwau8GmtibYWbtRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

之后将这个函数开头和结尾部分修改一下即可，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrzN7XibnfLX3gQY6Ud2sQNjE3o4RgTFuh0DHscRicuIDIPic5LFIXHvxGw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrffMXXHfJEFwNQCafghLC2ch3H6vOVlw5qyVxXAv71yoWywRmKibwVgA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

扣完之后打印下，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr0QrDFC7nQ1SDy8kGTSXH9PdicDPQxsTHV6MQZYHHdMaL7fzMnH6yJIA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这样我们之后使用的时候只需要调用函数里的方法就行了，我们可以看到，前面报错的地方是**_0x49d32f**这个函数没有定义，然后我们跳转过去可以发现**_0x49d32f**函数就是**_0x371e10**，上图又可以看到**_0x371e10**就是**eypt**，所以我们只需要这样使用即可，如下图所示 :

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr7cHXBBIJpNNm9RFuhDqUqianfApjBYV9aVMx4JMR2BVtZwAGebsoytQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下来**_0x76aa8['sample']**也是用同样的方法扣代码，扣完之后如下：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrxqfibkJzTOZ3n21FyGzoxibyKzYX2Bs1aibGKvepb6nr5kS2UjbKkWRMw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

之后替换到代码中即可，如下图所示：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Frryrfia6r4Wh06VUmNICoJQ1THxsWdS1PIPr8tm5KfZuOGVvFI8aQLiaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

对了，**_encrypt_1、_encrypt_2**这两个函数扣完之后相当于**cb**参数也破解了，下图就是上面扣完**cb**之后的图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fra1GL7apefpX3ibt65P0kwuAvZBiahhacdwsH2buI45XsIsaWWGJia0XwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

好了，接着说刚才的问题，之后运行代码，报错之后继续缺啥补啥，为了增加代码可读性，该还原的变量可以还原下，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrkcFMHWpGHXiaWZB85UpTKBx6ibNJEgbXB7g5ksEdrhwHJ9uOGxA8PXAg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里主要是对一些参数进行伪造，比如轨迹等，另外有些可以直接写成定值的就直接写成定值，比如**_0xdb3e79**参数，这里的无感校验轨迹啥的校验不是特别严格，所以自己写一下，或者百度一个即可，最终完成后如下：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrxM0anuEC6ZqH2cPyMic1tGcicogXLypkPwwKHS1Th6B2ibfOLv5y2vwkQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最后我们来试试能否成功通过这个无感校验，我们可以从下图看到是可以通过，所以理论上只要轨迹啥的模拟的没有问题基本上都是能通过的，至于成功率应该跟你自己的模拟轨迹有关了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrcUiaANCPL3T4hIEQfbhFZPziaBjs0XtSO4FmDkDRBzAQlsFreQbdYA0A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

到这里智能无感验证码的分析就完成了，接下来再来看看滑块拼图的。

**滑块拼图验证码**：

网上其实有很多相关的文章了（在文末我会放上推荐文章），所以就先直接说下它的一个流程 (最近有更新，所以细节上会有些变动)

```
1.初始请求"https://ac.dun.163.com/v3/d"接口 （姑且称为 d 请求接）  
  这一步会获取两个后面会用到的参数 -> DID、TID，这两个参数分别对应返回的数据中序号2，3，其中2对应的TID，3对应的是DID  
  这一步请求有三个参数 d、v、cb  
  d参数需要破解  
  v参数是个定值 （好像是会根据版本变化的，网站用的什么你就用什么好了）  
  这里的这个cb参数可以直接写成定值  
2.请求"https://ac.dun.163.com/v3/b"接口 姑且称为 b 请求 (这一步是可有可无的)  
  这个请求中所需的参数和上一步一样，只是生成d参数所需要的参数不同  
3.请求"https://c.dun.163.com/api/v3/get"接口  
  这一步主要是获取滑块拼图的相关的图片以及后面需要使用到的token参数  
  这一步所需要破解的参数 fp、acToken、cb，其中 fp、cb我们在无验证码中已经破解过，所以这一步只需要破解acToken即可  
  这一步不仅仅是拿到图片和token，之后还需要图片识别获取到滑块需要滑动的距离，后面演示的时候会讲到如果获取滑块滑动距离。  
4.滑块校验完毕之后请求"https://c.dun.163.com/api/v3/check"  
  这一步主要是进行滑块的校验，完成之后返回一个 validate 参数，用于后续请求的校验。  
  这一步中需要破解的参数 data （主要是根据上一步中的token及滑块轨迹等生成的）  
  该参数破解的逻辑和无感验证码中相同，只是所传参数不同而已，所以只需要找到加密位置，分析出所传参数就差不多了
```

首先来分析第一个请求中的**d**参数，大家可以自己打断点分析**d**参数生成的位置，这里我就直接指出来了，（这是出现**d**请求后）直接点击下图我圈出来的部分（大家在无法通过搜索直接定位加密参数位置的时候，可以通过该方式在逐一打上断点，然后缩小范围最终确定加密参数的位置）

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrsFt89LKXXPOM9NUzIRDYEANd6DnNOLO3vFwpTHMGIicxvRBtqjFnlRw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

之后在如下图所示的位置打上断点，清除浏览器数据，然后刷新下页面（重新发起一遍**d**请求），断到这个地方之后单步跟一下  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FruYCxWWAMYkanMdpIbAS0ArRewN0jPj9qjicoRSYtyx2dO9G6p6z0Utw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

单步跟进去之后我们来到如下位置，图中圈出来的位置就是**d**参数生成的位置，接下来就是逐步分析加扣代码的环节了  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrJGU4eznicgDpibYnc1eo45FaGiaZpLA1320omx1TCR4unFFMvlq5hjCEA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个**d**参数除了自己生成的参数外是不需要额外传任何值的，该写成定值的直接写成定值即可。为了方便大家扣代码，大家可以通过**ast**将该文件进行还原，之后替换该文件后再进行调试 。（在文末我会放上关于**ast**还原这个文件的推荐文章）

最终完成之后结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Frm9FQl5nFnIH9GmBVCmicayJyomjSOOM7D2GrelHCElMpYBiazOXRQ5bg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

第二步**b**请求中**d**参数也是在这个文件中生成的，这里直接同样直接指出来，如下图所示：（由于**b**请求不是必要的，所以可以不用分析这一步中的**d**参数是如何来的）

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrELiaYGAlSjtzOEdOmTCWGO7xxlaCC2tT7CsyHEX5NwFoZgjH1OqhImA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

直接点击跳转之后，下图圈出来的位置就是**b**请求中**d**参数生成的位置

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr83MousQicEeicPnoNRaSvibegR7LaCXzNJsN9icYG7OAzJWsOBzs1mS0Vg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里需要注意的是几个参数问题，上面**d**请求中**d**参数是除了自己生成的参数外是不需要额外值的，但这里需要传**TID、DID、yd_id（自己命名的）**，其中**TID、DID**是通过**d**请求获取到的，这个**yd_id**在获取背景图及校验请求的时候都会用到，一般会根据网站的不同而变化，如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrXhfJn0kB6oOJ67rcmPu07xiceklCPaXQM3NqU5x5fd6Z4V4dllAqT4w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrBOFCuZ4OGh0qT61SwESG4tdwX9UoL2lNbibqQ5ttf7AicUYUA7iaUrAaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最终扣完代码之后如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr2iaF5XEqmTJOCibB4Tic6tdH5gO9yRzsghZR5E6Z6DPOmk2icmlv5UPhjQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

再来分析第三步中的**acToken**参数，这个参数原本是在最后校验的时候才需要的，但是版本更新之后这个参数加在这一步了，最后校验那步反而不需要了，话不多说直接开始定位该参数，先全局搜索，然后发现只有一个js文件中，且在这个js文件中只有四处，介于不知道是哪个，索性全部打上断点，之后从新刷新页面断点断住，如下图：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrjMaIxDibpJ4vUcMHtugsM1icOcBB3SqZia51iafaK8541nCh47OKMenhdQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图我们发现该参数在这里已经生成，直接看一下前面的堆栈

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrDBXm9H70VZLhUyyb5icKPfhDmKKqd1LWDiaTqQyEISHcrLQM8lziaBMPQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

发现在这里一步也已经有了，继续往前看，这里直接关注**（anonymous)**就好了，然后继续一个一个打断点往前看，到下面这步打上断点

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrfKbvuetyCwWNwYiaU5eibRNs98bjnOrvdzicWiaM1YsfR7U8bfyOcbalZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里能发现这个**acToken**也已经生成了，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrzqCqPI5R7ibMAE5nkM6xlCQArnySVQYyloreUNcfPZw5UtYhGFdfCuQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

再往前跟一步，然后直接就可以看到**acToken**的是怎么生成的了，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrlKSSpQmdy4pIwCnIKfHH3yuq83J2cE6zkH4Sxcia056AS4kTicRuibiamA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里可以看到**acToken**是在这个**dc**里面生成的，接下来验证下，直接跟进去然后在return处打上断点，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr91If4cNsOmCdPwAbcKJIgAicX2aqSQRG1xtNNiaKAVLYp2Jn7lZlD6XQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

既然位置已经定位到了，那么接下来就又是扣代码的环节了，这里就只简单说一下好了，首先打上断点，然后断住后如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr8JQ1zwIjWNqLqjMMdMglhurLHvTTNFgeKcujzzVib7fEAy4yibQOKLgg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图中这个**q = ea().k(ta)**参数就是一开始的时候**d**请求返回的**DID（序号为3的）**，**f = a.C**这个参数是生成的，代码往前跟一下就能找到了，就是下图中的这个**bc**函数生成的：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrhNn8ic4OYdbvia8dY3v6ffo6eGq0Eot9hAZF27TtZgdNXEs7CxMDib2gQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最终扣完代码之后结果如下：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrNT8sTeJwEFczpQt1KUFM1PZHanHJulcMic67ia7S9yVLZ02M3UqsoicZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下来就只剩下最后的**data**参数了，话不多说，直接开始，这里我直接说下两个关键位置：

```
生成轨迹的地方 ->onMouseMove  
data参数生成位置 -> onMouseUp
```

所以后面直接在全局搜索就好了，先看看**data**参数生成的位置，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrQZ5JD973QRQ80ZAEJ3QxtJZAibdLAAReWqpiaGPkaFL42TOjy8jwibS8w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里的**data**参数加密方式和之前无感验证码的加密方式是一样的，加密函数直接复用就好了，然后轨迹的加密函数也是一样的，这里主要解决的就是一个轨迹生成的问题，轨迹的话大家可以自己模拟生成也可以自行百度，网上有很多，最后演示一下整个流程：  

##### 第一步发起d请求:

前面有说**d**请求中的**cb**参数是可以写成定值的，不过这里还是把**cb**参数的代码贴一下：

```
function ab() {  
    return "xxxxxxxxxxxx4xxxyxxxxxxxxxxxxxxx".replace(/[xy]/g, function (c) {  
        var e = Math["random"]() * 16 | 0;  
        return (c === "x" ? e : e & 3 | 8).toString(16);  
    });  
}  
  
function cb() {  
    return "__wmjsonp_" + ab().slice(2, 9)  
}
```

再来看看能不能用破解好的**d**参数拿到数据，下图可以看到是成功拿到数据的

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrjicBsojhQhE8ESsfhl1uM2K3welW9pD64DDp9N0iaibxwVbxymoN7amow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

##### 第二步发起b请求(这一步是可有可无的)：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrLysXnrtQ7icJTgtTVaLYC6gRHhW5Mib7RjGldibZTfbNPEr1W1J19KnFA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

能看到这一步也是成功拿到数据的，说明这一步的**d**参数也是没问题的。  

##### 第三步获取滑块拼图及token:

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr0RsYZEMicCsceAB1LgicjjSXLdw2u5yHszzS0eiaF4SkaVQicWp0OlBNicA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这一步也是成功拿到滑块拼图及token，但是这里拿到滑块拼图后并没有完，我们还需要拿到滑块需要滑动的距离，网上可以百度到很多方法，这里推荐一个超级靠谱且强大的识别库：**ddddocr**（ **文末会放上项目地址**），**这个库是业界大佬文安哲开源的一个免费识别库，具体用法github上都讲的很清楚，可以用于某验、某盾等，大家可以动动自己的小手给哲哥点个star**。下面就来演示一下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrwsUWGEGc0kN1OPyYXjrpryPSGIduL0rW5iaE4zVWM8O7qpv4AYbIRtw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

首先我们找一个验证码自己滑动一下，看到这里的滑动距离是**136**，然后我们利用识别库看看识别出来是多少，首先找到本次验证码滑块拼图图片链接，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr4iaNsLewe6Yjf3DicZlbxXAibNMVHSNxdibFYQ9Qh7fZEjic4mG6DBpSczA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后把链接拿过来试一下看看，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0FrxBCT9XBMrNXZesnlLlne8ASzSLVYyEIg6Hw4kkL7TSscQUyicM6KzoA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里我们能够看到识别出来的距离是相当的准确，下面就直接把代码贴出来：

```
import ddddocr, requests  
  
def get_distance(bg_url, front_url):  
    slide = ddddocr.DdddOcr(det=False, ocr=False)  
    background_bytes = requests.get(bg_url).content  
    target_bytes = requests.get(front_url).content  
    res = slide.slide_match(target_bytes, background_bytes, simple_target=True)  
    slider_distance = res.get("target")[0]  
    # 这里需要在识别出来的距离上+10，应该是跟图片大小有关导致的，需要自己去进行调整  
    slider_distance = slider_distance + 10  
    return slider_distance  
  
bg_url = "https://necaptcha.nosdn.127.net/bdfcbb7edd324a6ea3bdefd12c3e7ad7.jpg"  
front_url = "https://necaptcha.nosdn.127.net/b25c58f6c7b5457e82de930298a5a3bb.png"  
slider_distance = get_distance(bg_url, front_url)  
print(slider_distance)
```

既然滑块滑动距离也搞定了，那么我们就继续下面的步骤。

##### 第四步滑块验证码校验：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Friatx8Fvr2KEbWgaic1VrWdhGl5qlpSydLqlShbgWCgkbmCtaXRicVaEcA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到是成功的，说明我们前面说有破解出来的参数都是没有问题的，接下来连续测试看看：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic1pdzMXRaABlQ7GcK1HN0Fr2syHIgbGr8subj6LG06zjnbo8qsVibBpKOfxeLHnXulBwhRehZ0RZYA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从上图可以看出成功率是非常高的，成功率大概有98%左右，这也说明了哲哥的识别库是真的牛批。

最后总结一下一些会导致获取**validate**失败的原因：

```
1.初始必须要获取到DID，同一个DID可以多次使用，但是一旦DID失效就会导致最后获取不到validate值  
2.cb参数不能复用，在获取图片及token时复用会显示参数错，但是在校验接口中复用则是获取不到validate值  
3.获取图片链接及token时不带fp参数，或者fp参数有问题都会导致获取不到validate值  
4.生成data参数中的width参数必须和请求时的width参数一致否则获取不到validate值，即使获取到也只是碰巧  
5.获取的acToken参数必须根据DID来生成否则获取不到validate值  
6.轨迹生成的有问题或者滑块移动的距离有问题这个肯定是获取不到validate值
```

到此为止无感验证码和滑块验证码就全部分析完成了，然后点选、空间推理他们的整个流是大同小异的的，无非是一个**data**参数的变化，而**data**参数加密逻辑也是一样，只是传入的参数不同，加密逻辑不变，把传参改一下，像点选、空间推理也自然而然就能过了。

**相关文章及使用库推荐**：

[js逆向案例-某yd](http://mp.weixin.qq.com/s?__biz=MzU5NTcyMDc1Ng==&mid=2247484396&idx=1&sn=e3babb919a772f23c4f7f302b1197f9c&chksm=fe6cecb7c91b65a17ae876e81f264f6721c063bc631b10d907e3d7a48f1bf4525f9c2e6901d0&scene=21#wechat_redirect) -> 这篇是时一姐写的关于某盾滑块拼图验证码比较详细的文章（**公众号：逆向OneByOne**）

https://blog.csdn.net/qq_38999456/article/details/110184643?spm=1001.2014.3001.5502 -> 这一篇里面有关于js文件还原的步骤及代码

https://github.com/sml2h3/ddddocr -> 哲哥的开源识别库项目地址（强烈推荐，大家可以动动小手给个 star）

**推荐文章**

某验篇：

[js逆向案例-某jy](http://mp.weixin.qq.com/s?__biz=MzU5NTcyMDc1Ng==&mid=2247484397&idx=1&sn=e62bf44584403591ef0033dc93bd606b&chksm=fe6cecb6c91b65a0f20009697758e18fc5ca679f7709a01cee1e30a97ae544fafd792e198eff&scene=21#wechat_redirect) 、[js逆向案例-zzjg之jy3/ast/woff.2](http://mp.weixin.qq.com/s?__biz=MzU5NTcyMDc1Ng==&mid=2247485270&idx=1&sn=27db55b20d847fc0d1f148e39ba61ace&chksm=fe6ce80dc91b611b1d4632536bce75beed3695341c075d4dbf47206b0e1124408d00cbd064b7&scene=21#wechat_redirect)