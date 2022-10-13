> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3bnTXCc2DrnPcQao3rKZ0A)

一、什么是WEB前端混淆防护？

  

在介绍“WEB前端混淆防护”之前，我们先来了解一下“WEB前端混淆”。一般来说，WEB前端混淆技术可以在不影响页面呈现和用户交互的情况下，将HTTP会话报文中的关键内容（请求参数、HTML、JS等）转换为难以阅读和修改的形式。

这种做法不仅能够保护前端源码的知识产权，还能妨碍访问者通过浏览器人工交互以外的方式访问WEB站点。长久以来，互联网行业广泛将WEB前端混淆技术运用到反爬虫、防薅羊毛等诸多场景中，展现出了良好的实际价值。

而WEB前端混淆防护，就是将WEB前端混淆技术作为一种应用安全防护措施来使用。由于WEB前端混淆会对浏览器之外的访问方式造成妨碍，因此能够有效阻止各类自动化WEB应用攻击手段（如WEB应用扫描），并迫使那些技艺不精、信心不足、或者不针对特定目标的攻击者放弃进攻，最终达到保护应用安全的目的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZLSJB3iatPnMMyHmQHUDiazvtLDBfgrdjtXtibD1NyZskXWjuJKn4wmlNg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

二、WEB前端混淆防护在企业安全的现状

  

近些年来，企业安全领域中越来越频繁地出现利用WEB前端混淆来妨碍安全测试的案例，甚至出现了一些专门的WEB前端混淆防护产品。他们巧妙地利用了安全测试资源紧张的特点，通过WEB前端混淆来拖延安全测试工作的开展，导致测试人员因无法在短时间在完成测试而被迫承认被测系统是“安全的”。即使测试人员本身技艺纯熟，能够设法绕过前端混淆措施，也至少可以导致很多漏洞利用工具难以使用，显著降低其测试效率。从目前已知的几个WEB前端混淆防护部署案例来看，此种防护方案确实能够有效掩盖应用系统中的漏洞，进而在安全评估和合规检查中取得更好的成绩。

众所周知，在经典安全实践中，安全加固操作最为有效的方法是从根本上修补或移除存在漏洞的功能模块，其次是切断攻击者利用漏洞的途径。但WEB前端混淆既不从根本上修补漏洞，本身也不影响攻击者对漏洞的利用。如果攻击者认为攻击得手后利益庞大，那么WEB前端混淆防护只是一次性地提高了攻击者利用漏洞所需的成本，即攻击者需要先完成对前端混淆代码的逆向分析。由于逆向分析可以离线完成，WEB前端混淆防护甚至可能完全不会对入侵行为产生告警。因此，一般认为WEB前端混淆是一种有价值的补充防护措施，而不适合单独部署使用。

本文将针对性介绍这些混淆防护的机制，并探讨在安全测试中突破这些机制的可行性。

三、WEB前端混淆防护的常见形式

  

笔者收集整理了一些WEB前端混淆防护中常用的技术手段，供参考。需要注意的是，实战中的各种手段往往会结合使用，互相补强，需要灵活应对。

1**通信保护**

不论对于自动扫描还是人工测试来说，HTTP请求的构造、变造、篡改、重放都是非常重要的操作，而下列措施直接提高这些操作的执行难度：

**请求参数编码/加密/MAC**

即在客户端代码中，将提交给服务器的请求参数先进行可逆变换再发送。由于实现简单且不需要过多的安全专业知识，很多老练的开发人员都会在通信逻辑中加入这样的机制。最坏的情况下也能阻止自动化漏洞扫描，极大提高应用系统上线前后安全检查的通过率。

实际的应用系统中，实现方式也是多种多样。

例如，下图请求参数为十进制ASCII编码。此类实现比较简单，熟练的测试人员肉眼观察即可理解并仿制编码，甚至不需要逆向分析：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZaQLaibS4jmV7ichExUpIWpu0l8kWhSSzrGu7Je9yyTY3ZLl850G2xSMQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

再例如下图JS代码为动态密钥的RSA加密。此类实现相比之下就更加复杂，安全测试中需要花费一些时间去还原加密过程：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZHxa0icpDic0ldVwBDvaSt7PyFoEqNNpMbC1lzCMh6YWANmxibVYHzt4hA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

即使测试人员已经阅读代码获知完整的加密流程，要通过MITM工具修改非对称加密的请求参数也是一件非常麻烦的事情。

上文中的截图代码为开发人员自行实现的通信加密。防护产品在实现该功能时，一般通过对XMLHttpRequest、ActiveXObject等Ajax中用到的异步请求接口进行hook来完成。

类似的还有在请求报文中加入消息认证码（MAC）的方法，在此不一一赘述。虽然逆向分析还原并仿制算法理论上是必定可解的，但实践起来难免需要投入很多时间和精力。

**Cookie令牌**

即通过定时器不断更新Cookie，类似于时钟令牌的抗重放措施。

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZrbXvDo85ovkRiaQDHUzyq9cZ0iaN7NzSfw9O8ZEROU7l4gAiboL3kkzow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

相比于直接对请求参数进行加密的方法，动态Cookie的算法逻辑是独立执行的，并不位于发送HTTP请求的过程中，因此在逆向分析中不易通过常规的控制流追踪来定位。但也相对地，Cookie令牌一般来说只起到抗重放作用，本身并不能防止请求参数篡改。

2**抗逆向分析**

由于前端JS代码是在客户端环境中运行的，测试人员不仅能够看到完整的代码，还能干涉其运行过程。通过对JS代码进行逆向分析，测试人员可以获取关于请求参数编码、Cookie令牌的具体算法，并针对性地绕过这些保护。

而下列措施可以妨碍测试人员的分析，使测试人员难以得到具体的算法内容：

**代码混淆**

代码混淆是一种非常成熟的抗逆向分析措施。在实战场景中，JS的代码混淆以命名混淆和控制流混淆为主。前者将代码中的各种变量和函数名替换为无意义的乱码，后者则让代码的文本顺序与执行顺序不一致，从而妨碍阅读。

由于命名混淆原理上是无法还原的，逆向分析中一般需要把握数据流和控制流来还原代码逻辑，因此更加关注控制流混淆。

下图为四叉树随机跳转控制流混淆：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZdiblLUxy3kIa0cOOquZg5Qsd6Q5upJDA551MISeFALbiaT9qdjoTGDnQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZCxsdLNKxf2ropZNfzYygQjOCFWb18ZuS6SVg7Bzgia8icQh2UzBgWOUg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

下为多重分支随机跳转控制流混淆：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZawJ9H0zypLYTAFxuAyZqBzeoJdCr3BNeEfooRJhUvTMPszAb27ib3ow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

控制流混淆的方式也并不局限于随机跳转。例如当数据流为并行时，处理不同数据流的代码之间执行先后不会影响执行结果，可以随意穿插以混淆控制流。

控制流混淆还可以隐藏API调用，例如下面这段原始JS代码：

```
1var input=document.getElementById("username");  

```

可以等价替代为：

```
1var a=document;  
2var b="getElem";  
3var c="entById";  
4var d="username";  
5b+=c;  
6c=a[b];  
7var input=c.call(a,d)
```

这样一来，就可以防止测试人员通过搜索“getElementById”来定位API调用了。但为了进一步妨碍逆向分析，通常还要结合更加复杂的控制流混淆：

```
 1var m=[5,4,7,3,2,1,6];//按照这个顺序去执行下面的switch分支  
 2var a,b,c,d,input;  
 3for(var i=0;i<m.length;++i)  
 4{  
 5switch(m[i])  
 6{  
 7    case 7:  
 8        c="entById";  
 9        break;   
10    case 2:  
11        b+=c;  
12        break;   
13    case 5:  
14        a=document;  
15        break;   
16    case 1:  
17        c=a[b];  
18        break;   
19    case 6:  
20        input=c.call(a,d);  
21        break;   
22    case 3:  
23        d="username";  
24        break;   
25    case 4:  
26        b="getElem";  
27        break;  
28}  
29}
```

可见混淆后的代码可读性会非常差，很难从单个片段理解其原始含义。

如果想要对代码进行有效的阅读，如果不进行动态调试，就必须将被打乱的控制流还原，对逆向分析造成了不小的困难。

动态代码执行

  

在WEB前端的逆向分析中，DOM内容和HTTP通信内容都是容易被测试人员获取的。因此想要保护前端JS代码不被分析，需要避免关键逻辑代码直接暴露。  

JS是脚本语言，不适用native层面的各种二进制壳。但动态执行JS代码的方式也有很多，常见的包括：

*   eval("XXX")
    

*   setTimeout("XXX",…)
    

*   setInterval("XXX",…)
    

*   document.write("<script>XXX</script>")
    

*   XX.innerHTML="<iframe src=\"javascript:XXX\"></iframe>"
    

*   location="javascript:XXX"
    

*   等等
    

为了防止内层代码被测试人员获取，外壳代码往往会通过非常复杂的逻辑去解密内层代码，然后再去调用类似上述的动态代码执行方法。

此外，内层代码往往也会包含各种混淆措施，甚至还会释放出更内层的代码来执行。

代码在执行完毕后，也可能会删除页面上的<script>标签、清除中间过程中使用的变量等等，进一步妨碍逆向分析。

下图为网站外壳代码完成对内层代码的解密后，调用eval函数动态执行内层代码的瞬间。可见其代码文本特征非常稀薄，淹没在庞大的混淆代码中，笔者花费了不少时间才让调试器中断在这个位置：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZib6by2aSibFUo3UMlicevfqq1aDJtVvicvQbFhcgPYLS0ATFuMGAOI2yWQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这种动态代码执行导致通过HTTP通信和DOM的直接分析收效甚微，是一种经典而有效的抗逆向分析措施。

**反调试**

多数主流浏览器都支持通过F12开发人员工具和远程调试端口对JS代码进行调试。动态调试可以有效应对多数混淆措施，从中还原出运行逻辑，是逆向分析的关键手段。

不同于native代码中反调试与反反调试无止无休的底层对抗，JS层面应对调试器的方法不多。由于JS代码的运行权限很小，也很难访问操作系统API，实战中以debugger指令和运行时间检测为主。

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZibgoaxibsB2R6qsBPUgmnlfnQ6wN9889uOS98RdP29gCDHdJend7yWVg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

对于大部分浏览器来说，如果当前代码没有被调试，则debugger指令没有任何作用；反之，debugger指令就相当于一个调试断点。大部分浏览器都不支持将debugger指令与手动设置的断点区分对待，但手动断点在调试过程中几乎是必要的。

这种措施虽然简单，但还是给逆向分析带来了不小的麻烦。

**hook检测**

hook在逆向分析中也是一种很常用的方法。例如，如果对eval函数的调用进行劫持，很可能非常轻松地实现脱壳。

与反调试类似，JS代码能够实现的hook检测手段也比较有限，主要方法为调用API前进行toString()，以及检测arguments.callee来判断当前函数的调用来源是否合法等。

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZiaicdicXLFpI2AkQYPI09kozOrkv5ZZZXmpeEhJ2XDib0oSQicU1leHqeAw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

虽然JS代码能够实现的hook检测并不复杂，但JS层面能够进行的hook方法也十分有限，或多或少也会造成一些麻烦。

3**其它**

**浏览器指纹检测**

WEB前端混淆防护的一个关键任务是阻止自动化访问（爬虫、暴力猜测、WEB扫描等等），因此代码中通常会试图检测当前运行环境是否为真实浏览器。  

例如下面的代码试图检测当前运行环境是否为Node.js：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZQB3PQv4szlQwoianYct775QZQyOq0EuVLdYEOBgcn0sFuZhTPR8Pn1g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

还有其它各种各样的检测实现：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZdQk4XTZUPTU0r8vRvGUdBhp09VN3yfvmt1wMwbiardyGaDIWo6Sic9qg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZa3vcCnyJtIRo2KBbWXfIKtCvJp7j3bf39Dc8pa77GA9aZ4a6Dksib1Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZxa9EPbialroMufLicRhncWL5P10yeiag8qiaVRd2659VTO5xeU0uWicOL6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果能够知道防护代码的具体检测手段，要针对性地予以伪造也并非不可行。但浏览器指纹检测的实现方法变化万千，且混合在大量混淆代码中，针对性对抗难免陷入被动。

四、安全测试中的通用对抗思路

  

综上所述，如果测试人员想要通过代理工具修改请求参数：

1.由于请求参数编码和Cookie令牌的存在，测试人员需要先阅读其具体算法

2.由于算法代码被混淆并动态执行，测试人员需要先进行调试或hook以还原其运行逻辑

3.由于代码中包含反调试和hook检测措施，测试人员需要先应对这些措施

在实际的安全测试工作中，要对于每个被测系统进行花费这么大的成本投入是不可行的。我们必须寻求直接逆向分析以外的方法。

这里引入一个关键的切入点：相比于网络防护系统（如WAF、IPS等），客户端JS代码中实现的过滤策略更容易被绕过，且防护规则泄露的风险要大得多。因此在大部分防护实现中，**混淆代码本身一般不会对输入数据进行过滤。**

实际上，在面对WEB前端混淆防护时，如果加密函数暴露可见，很多测试人员都会采取在控制台中直接调用加密函数的方式来变造请求参数。这正是因为加密函数本身很少附带过滤机制。由此可见，只要能够有效控制混淆代码的输入值，就可以在很大程度上完成安全测试。这样一来，混淆代码的内部运行逻辑以及输出值可能就不那么重要了。

遵循上述结论，我们的核心思路就是要对JS代码的输入值进行劫持。具体来说，这些“输入值”包括随机数、JS字面量、DOM输入、I/O输入（如Ajax返回结果等）、通过接口获取的一些状态和配置信息等（如日期时间、浏览器UA等）。

五、浏览器内核hook案例

  

干涉JS代码输入过程的方法有很多，本文给出了其中的一种方法，通过修改主流开源浏览器的内核代码，在关键函数上加入hook，实现了对部分JS输入值的劫持和混淆措施的应对。

这样做有以下几点好处：

1.不针对特定的混淆方式，只要混淆代码本身没有过滤机制，理论上就可以通杀；

2.JS代码很难直接检测或绕过位于浏览器内核中的hook；

3.浏览器内核native代码容易与外部进行通信，从而带来自动化测试（如使用sqlmap等）的可能性；

4.主流浏览器一般能够通过浏览器指纹检测；

**注：**本文代码基于Chromium开源项目（版本81.0.4000.0，Windowsx64）进行修改，遵循BSD授权（https://chromium.googlesource.com/chromium/src/+/master/LICENSE）。源码获取和编译方法请参考Chromium官方网站（https://www.chromium.org/）。

1**劫持输入值**

**HtmlElement.value**

此处hook的目的在于对DOM元素的值的访问进行劫持。在多数情况下，这是最关键的一个hook位置。由于blink中各个HTML表单元素的valur()函数是分别派生实现，此处hook比较复杂。

推荐hook位置：

src\third_party\blink\renderer\core\html\forms\html_form_control_element.h

```
1//新建静态函数：  
2static String JSTracerFilterGetValue(String);
```

src\third_party\blink\renderer\core\html\forms\html_form_control_element.cc

```
 1//实现h文件中新建的静态函数：  
 2String HTMLFormControlElement::JSTracerFilterGetValue(String value) {  
 3  // HTMLFormControlElement::value()返回的是WTF::String，比起Attribute::Value()方便得多  
 4  static std::atomic_int INDEX = {0};  
 5  
 6  char tsource[1024], tdestination[1024], tfilepath[64];  
 7  FILE* tfile = fopen(".\\JSTracer\\getValueReplace.config", "r");  
 8  if (!tfile)  
 9    return value;  
10  //这里使用fscanf的话，没法处理包含空白符的替换配置。后续可能需要注意修改  
11  if (!::fscanf(tfile, "%s%s", tsource, tdestination)) {  
12    ::fclose(tfile);  
13    return value;  
14  }  
15  ::fclose(tfile);  
16  
17  std::string tvalue = value.Utf8();  
18  for (bool treplaced = false; true; treplaced = true) {  
19    size_t tindex = tvalue.find(tsource);  
20    if (tindex == tvalue.npos) {  //没有其它可替换的tsource了，返回结果  
21      if (treplaced) {  //之前发生过替换，需要生成新的WTF::String对象  
22        int tindex = INDEX++;  
23  
24        sprintf(tfilepath, ".\\JSTracer\\Logs\\%lld_getValue_Replaced_%d.log",  
25                time(0), tindex);  
26        FILE* tfile = fopen(tfilepath, "w");  
27        fprintf(tfile, "%s -> %s", value.Utf8().c_str(), tvalue.c_str());  
28        fclose(tfile);  
29  
30        return String::FromUTF8(tvalue.c_str());  
31      } else {  //没有发生过替换，返回原值即可  
32        sprintf(tfilepath, ".\\JSTracer\\Logs\\%lld_getValue_%d.log", time(0),  
33                rand());  
34        FILE* tfile = fopen(tfilepath, "w");  
35        fprintf(tfile, "%s", value.Utf8().c_str());  
36        fclose(tfile);  
37  
38        return value;  
39      }  
40      break;  
41    }  
42    tvalue = tvalue.replace(tindex, strlen(tsource), tdestination);  
43  }  
44}
```

src\third_party\blink\renderer\core\html\forms\html_input_element.cc

```
 1//调用上面新建的静态函数，对value()的返回值进行处理  
 2String HTMLInputElement::value() const {  
 3  switch (input_type_->GetValueMode()) {  
 4    case ValueMode::kFilename:  
 5      return JSTracerFilterGetValue(input_type_->ValueInFilenameValueMode());  
 6    case ValueMode::kDefault:  
 7      return JSTracerFilterGetValue(FastGetAttribute(html_names::kValueAttr));  
 8    case ValueMode::kDefaultOn: {  
 9      AtomicString value_string = FastGetAttribute(html_names::kValueAttr);  
10      return JSTracerFilterGetValue(value_string.IsNull() ? "on"  
11                                                          : value_string);  
12    }  
13    case ValueMode::kValue:  
14      return JSTracerFilterGetValue(non_attribute_value_);  
15  }  
16  NOTREACHED();  
17  return g_empty_string;
```

src\third_party\blink\renderer\core\html\forms\html_select_element.cc

```
1//调用上面新建的静态函数，对value()的返回值进行处理  
2String HTMLSelectElement::value() const {  
3  if (HTMLOptionElement* option = SelectedOption())  
4    return JSTracerFilterGetValue(option->value());  
5  return "";  
6}
```

src\third_party\blink\renderer\core\html\forms\html_text_area_element.cc

```
1//调用上面新建的静态函数，对value()的返回值进行处理  
2String HTMLTextAreaElement::value() const {  
3  return JSTracerFilterGetValue(value_);  
4}
```

完成修改并编译生成后，在当前目录下创建JSTracer文件夹，然后在其中创建getValueReplace.config配置文件和Logs文件夹。getValueReplace.config中用空格分隔，前为查找值，后为替换值。

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZp8LzlVFLSL903q3HkrV8iaxxpKcc6DaiaeWZptfI1NW30BSgrgoaIABQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后用命令行参数“--no-sandbox”启动编译好的浏览器，可见所有对HTML元素值的读操作都会被劫持（基本上不限访问途径，包括表单提交和JS脚本访问等）：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZQKzKrViaic4It7t9icMibrO6l6XicZ8LSh7eyXnYrFCZqaGbCUx0T5qPp0w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

且不论替换与否都会记录到\JSTracer\Logs中：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZmEVjWYfnnjFiaNYltnzdF0ArqzVM2WUjL6GtQwl5QCREkx6qIxZRWDA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**HtmlAttribute.value**

### 此处hook的目的在于对DOM元素的属性的访问进行劫持。  

推荐hook位置：\src\third_party\blink\renderer\core\dom\attribute.h

```
 1const AtomicString& Value() const {  
 2    static std::atomic_int LOG_INDEX = {0};  
 3    static std::atomic_int BUFFER_INDEX = {0};  
 4    //因为是常函数又必须返回引用，只能期待实际修改的属性访问不多或者使用短暂...  
 5    static AtomicString* BUFFER = new AtomicString[4096];  
 6  
 7    std::string tname = name_.LocalName().Utf8();  
 8    if (tname != "id" && tname != "name" && tname != "style")  
 9      return value_;  // DOM属性的访问操作实在太多，挑些关键的处理吧  
10  
11    char tsource[1024], tdestination[1024], tfilepath[64];  
12    FILE* tfile = fopen(".\\JSTracer\\getAttrReplace.config", "r");  
13    if (!tfile)  
14      return value_;  
15    //这里使用fscanf的话，没法处理包含空白符的替换配置。后续可能需要注意修改  
16    if (!fscanf(tfile, "%s%s", tsource, tdestination)) {  
17      fclose(tfile);  
18      return value_;  
19    }  
20    fclose(tfile);  
21  
22    std::string tvalue = value_.Utf8();  
23    size_t tpos = 0;  
24    for (bool treplaced = false; true; treplaced = true) {  
25      tpos = tvalue.find(tsource, tpos);  
26      if (tpos == tvalue.npos) {//没有其它可替换的tsource了，返回结果  
27        if (treplaced) {//之前发生过替换，需要用掉一个全局缓冲区来生成AtomicString引用  
28          int tindex = BUFFER_INDEX++;  
29          BUFFER[tindex % 4096] = AtomicString::FromUTF8(tvalue.c_str());  
30  
31          sprintf(tfilepath, ".\\JSTracer\\Logs\\%lld_getAttr_Replaced_%d.log",  
32                  time(0), tindex);  
33          tfile = fopen(tfilepath, "w");  
34          fprintf(tfile, "%s -> %s -> %s", tname.c_str(), value_.Utf8().c_str(),  
35                  tvalue.c_str());  
36          fclose(tfile);  
37  
38          return BUFFER[tindex % 4096];  
39        } else {//没有发生过替换，返回原值即可  
40          sprintf(tfilepath, ".\\JSTracer\\Logs\\%lld_getAttr_%d.log", time(0),  
41                  LOG_INDEX++);  
42          tfile = fopen(tfilepath, "w");  
43          fprintf(tfile, "%s -> %s", tname.c_str(), value_.Utf8().c_str());  
44          fclose(tfile);  
45  
46          return value_;  
47        }  
48        break;  
49      }  
50      //在tvalue中找到了tsource，执行替换并继续搜索  
51      tvalue = tvalue.replace(tpos, strlen(tsource), tdestination);  
52      tpos += strlen(tdestination);  
53    }  
54  }  //{ return value_; };
```

但因为blink::Attribute::Value()方法以常函数形式直接定义在h文件中，如果直接修改h文件，每次生成时会花费非常长的时间，调试起来很不方便。因此也可以考虑将实现代码放在cc文件中。

但这样的话又需要注意，attribute.h同时在blink_core和blink_modules两个模块中被引用。除了在\src\third_party\blink\renderer\core\dom\中新建attribute.cc并实现函数之外，src\third_party\blink\renderer\modules\accessibility\中也要实现一份，否则可能会链接失败而无法正常生成。此外，添加cc文件时不要忘记同步修改build.gn：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZlxEsnakCK7ClLoM8bHfjsW7sLcIv9n0RSjly6GxUzgM9LUibzeria4Gw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

用法与前文HtmlElement.value类似，但配置文件变成了getAttrReplace.config。

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZo9UA5dXzj3lBNRCHzRc0hxGMoP3eAIQYWQOUm6UaXRn1qeWqdald4w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZr05KU1JxoEm0VCjpOwLVV9j5ELPPu0lt7ibU1Lu3PQExYqesiaWtKeOA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

不要忘记命令行参数需要加“--no-sandbox”。

目前为止所做的都是针对DOM输入的劫持（因为实战场景中需求最多）。

感兴趣的读者可以自行尝试劫持其它输入。

2**应对抗逆向分析措施**

**动态代码执行**

此处hook的目的在于记录脚本执行过程中动态编译JS代码并执行的操作。在多数情况下，这样做可以轻松脱掉混淆代码的外壳。

推荐hook位置：src\v8\src\codegen\compiler.cc

```
 1//新增函数StartWith，用来过滤掉一些我们不关心的动态脚本编译操作  
 2bool StartWith(char* a, char* b) {  
 3  for (int i = 0; a[i] != 0 && b[i] != 0; ++i)  
 4    if (a[i] != b[i]) return false;  
 5  return true;  
 6}  
 7MaybeHandle<JSFunction> Compiler::GetFunctionFromEval(  
 8    Handle<String> source, Handle<SharedFunctionInfo> outer_info,  
 9    Handle<Context> context, LanguageMode language_mode,  
10    ParseRestriction restriction, int parameters_end_pos,  
11    int eval_scope_position, int eval_position) {  
12  Isolate* isolate = context->GetIsolate();  
13  int source_length = source->length();  
14  isolate->counters()->total_eval_size()->Increment(source_length);  
15  isolate->counters()->total_compile_size()->Increment(source_length);  
16  
17  //在此处添加hook代码  
18  static std::atomic_int INDEX = {0};  
19  int tlength = 0;  
20  std::unique_ptr<char[]> tsource =  
21      source->ToCString(DISALLOW_NULLS, FAST_STRING_TRAVERSAL, &tlength);  
22  char* ttsource = tsource.get();  
23  //看上去应该是DevTool内置的一些脚本。因为数量非常多所以专门屏蔽一下。  
24  if (!StartWith(ttsource, (char*)"Root.Runtime.cachedResources") &&  
25      !StartWith(ttsource, (char*)"import('./") &&  
26      !StartWith(ttsource, (char*)"TextEditor.CodeMirrorTextEditorFactory")) {  
27    char tfile[64];  
28    sprintf(tfile, ".\\JSTracer\\Logs\\%lld_eval_%d.log", time(0), ++INDEX);  
29    WriteChars(tfile, ttsource, tlength, true);  
30  }  
31  
32  ……
```

不要忘记命令行参数需要加“--no-sandbox”。无需特别配置，所有动态编译的代码会被输出到JSTracer\Logs文件夹中：

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZ9nHwiaIUrz3ibwxiaDnGxSVecicoKxh2YRLwQL1MCiaNrAdQbnu1ESWibckQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUZlrCTVq5m7hhlA1liaaOgiaZCOVuVxIXRvWXnOVwc9qbwjDKIZ2T8SRiceLHTDeu7b11NGNCBXsQSwA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上述方法实测对eval、setTimeout、setInterval的动态执行均有效，但不能捕获到DOM中的JS代码执行。如果需要关注DOM中的代码执行，则要将hook代码移动到更加深层的函数中。本文推荐在这个位置进行hook，因为DOM中的JS代码实在太多了，且大多并非我们关注的脱壳。

与前面的两个hook位置不同，此处修改不再位于blink模块，而是v8引擎中。此外，由于JS外壳释放的代码可能也是动态变化的（并可能带有运行时间检测），要修改这些代码还是比较困难的。我们姑且先只进行记录，而不进行修改。

另外需要注意，对于setTimeout和setInterval中第一个参数为函数（而非字符串）的情况，由于不需要动态编译JS代码，上述位置的hook不起作用。但既然不涉及脱壳，hook与否相对而言也不那么要紧。

但这里还是给出blink中它们的实现位置，供确实需要hook这些功能的读者参考：src\third_party\blink\renderer\core\frame\window_or_worker_global_scope.cc

**反调试**

此处hook的目的在于对抗JS代码中的反调试措施。

推荐hook位置：src\v8\src\parsing\parser-base.h

```
 1template <typename Impl>  
 2typename ParserBase<Impl>::StatementT ParserBase<Impl>::ParseStatement(  
 3    ZonePtrList<const AstRawString>* labels,  
 4    ZonePtrList<const AstRawString>* own_labels,  
 5AllowLabelledFunctionStatement allow_function) {  
 6  
 7 ……  
 8  
 9  switch (peek()) {  
10  
11   ……  
12  
13    case Token::DEBUGGER:  
14      //return ParseDebuggerStatement();  
15      Next();  
16      return factory()->EmptyStatement();  
17  
18……  
19  }  
20}  
21  
22  ……
```

如此修改后，即使在浏览器中开启F12开发人员工具中的调试器，也依然会忽略JS代码中的debugger指令。此处修改发生在JS代码编译阶段，对调试器中手动设置的断点没有影响，仍然可以正常触发。

六、后记

  

通过本文所介绍的方法，实测可以对使用了部分商业级WEB前端混淆防护的网站功能进行手工测试。该方法理论上也具备自动测试的可行性（例如，用按键精灵等模拟操作的方法反复提交请求，每次提交请求前修改配置文件）。读者可以按照自己的喜好任意设计用户交互。

如果您发现文中描述有不当之处，还请留言指出。在此致以真诚的感谢~

关于天枢实验室

天枢实验室聚焦安全数据、AI攻防等方面研究，以期在“数据智能”领域获得突破。

  

内容编辑：天枢实验室 吴复迪  责任编辑：肖晴

**往****期回顾**

*   [2020年Q1 WS-Discovery反射攻击观察](http://mp.weixin.qq.com/s?__biz=MzIyODYzNTU2OA==&mid=2247487691&idx=1&sn=d3dc9c8c110b87d415a8a8ed6e38ce66&chksm=e84fb614df383f02d53e73331b8b3a3156e5c216ff80d73e812caa36673fda659c49eae92408&scene=21#wechat_redirect)  
    
*   [物联网卡业务安全分析发展之路](http://mp.weixin.qq.com/s?__biz=MzIyODYzNTU2OA==&mid=2247487671&idx=1&sn=3e253b55411e11e2f3071b2257a678f9&chksm=e84fb668df383f7ef94e46c3ed319afdd3c96bb42ebb8b97555bef51aa08754c7cb3037ec069&scene=21#wechat_redirect)  
    
*   [物联网安全始于资产识别](http://mp.weixin.qq.com/s?__biz=MzIyODYzNTU2OA==&mid=2247487662&idx=1&sn=e0a13d0c7b1b7d6c7dbe0e6ed27be49f&chksm=e84fb671df383f67b07afb6ecaf4f45a980f225028728c927f8e3dcefeac1440b0568bdaf68a&scene=21#wechat_redirect)  
    
*   [首发—攻击者开始攻击Edimax WiFi桥接器](http://mp.weixin.qq.com/s?__biz=MzIyODYzNTU2OA==&mid=2247487656&idx=1&sn=6bc98e1dbc3640468713371aa0756950&chksm=e84fb677df383f61fce9597a08e883b06bdf220acad7572f32084067baafcf2db990129c871b&scene=21#wechat_redirect)  
    

本公众号原创文章仅代表作者观点，不代表绿盟科技立场。所有原创内容版权均属绿盟科技研究通讯。未经授权，严禁任何媒体以及微信公众号复制、转载、摘编或以其他方式使用，转载须注明来自绿盟科技研究通讯并附上本文链接。

**关于我们**

  

绿盟科技研究通讯由绿盟科技创新中心负责运营，绿盟科技创新中心是绿盟科技的前沿技术研究部门。包括云安全实验室、安全大数据分析实验室和物联网安全实验室。团队成员由来自清华、北大、哈工大、中科院、北邮等多所重点院校的博士和硕士组成。  

绿盟科技创新中心作为“中关村科技园区海淀园博士后工作站分站”的重要培养单位之一，与清华大学进行博士后联合培养，科研成果已涵盖各类国家课题项目、国家专利、国家标准、高水平学术论文、出版专业书籍等。

我们持续探索信息安全领域的前沿学术方向，从实践出发，结合公司资源和先进技术，实现概念级的原型系统，进而交付产品线孵化产品并创造巨大的经济价值。

**长按上方二维码，即可关注我们**