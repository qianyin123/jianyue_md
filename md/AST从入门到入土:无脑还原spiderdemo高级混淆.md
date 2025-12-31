> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ayTbCZgXcDu0lSlZRSB2GA)

#### 声明

###### 本文写于2025年12月29日,仅做技术学习交流。本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！

#### 一、前言

早上看到大佬出了自写的高级混淆.

趁着中午休息时间试试还原

输入如下代码

```
console.log("Hello World");  

```

我们选择高级混淆,配置全选

![图片](https://mmbiz.qpic.cn/mmbiz_png/1pcia7zHaice3cIuia406kVfYKqBrE052uL1X1DQ0knQJn6XKFu3oM94oKqDmIEpnM0rvqqDQepkqeFHibmItMJvOQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

  

运行后正常

![图片](https://mmbiz.qpic.cn/mmbiz_png/1pcia7zHaice3cIuia406kVfYKqBrE052uLuSz5gMr3ZQ5iatqfUkqVBJfHLNxP7YpR0Tjyfurxf7r8lIsjF6UjxPw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

  

接下来我们开始还原

#### 二、还原

本来想自己写还原代码,突然想到在蔡老板星球里面一年了.还没咋用过插件.

因此本篇基本全用蔡老板的星球插件还原

可以看到代码变量全是不宜读的字符.

![图片](https://mmbiz.qpic.cn/mmbiz_png/1pcia7zHaice3cIuia406kVfYKqBrE052uLy4vAmxKUZApiaiaE8ueqIyiaecniab942u05YhhRDzVomicoiadSZWp3MibDg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

因此第一步我们使用星球的变量重命名插件

![图片](https://mmbiz.qpic.cn/mmbiz_png/1pcia7zHaice3cIuia406kVfYKqBrE052uLicOO4r5iaCYH2BXSroDn2icvCCcdn0UnS8ajrhGFVicHmRnDN8Vc1FpPOA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

可以看到还原后的可读性就变得很高了.

接下来,我们可以看到有大量的变量定义

```
var bF = Object;  
var uO = name;  
var zB = String;  
var qZ = RegExp;  
var oE = Boolean;  
var kH = Math;  
var gI = window;  
var aP = document;  
var tU = decodeURIComponent;  
var dJ = escape;  
var nA = atob;  
var xR = parseInt;  
var iM = isNaN;  
var cX = NaN;  
var mH = Array;  
var iC = Number;  
var lL = console;  
var uY = Date;  
var eB = Function;  
var rL = setInterval;  

```

那么就需要使用变量定义还原插件,加上配套的常量折叠插件

![图片](https://mmbiz.qpic.cn/mmbiz_png/1pcia7zHaice3cIuia406kVfYKqBrE052uLbJCgXGtGiaBnhhjhgqDrs6cmKDaIsTv5vCbtMKiar0iaY546bLk89aWzQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

可以看到两个插件运行后,还原进了一步

接下来我们需要写一点代码,把如下的调用还原回去

```
hW("ﱟﱞﱠ", 0x11, 0xed)  

```

我们先收集switch的参数

```
let swtichcall = {};  
traverse(ast, {  
    SwitchStatement(path) {  
        let {node, scope} = path;  
        let {cases, discriminant} = node;  
        if (!types.isIdentifier(discriminant, {name: "operator"})) {  
            return  
        }  
        for (let case1 of cases) {  
            let {consequent, test} = case1;  
            let {left, operator, right} = consequent[0].argument;  
            let value = test.value;  
            let operator_ = operator  
            swtichcall[value] = operator;  
        }  
    }  
})  

```

接下来匹配还原计算即可.

可以看到计算已经还原了

![图片](https://mmbiz.qpic.cn/mmbiz_png/1pcia7zHaice3cIuia406kVfYKqBrE052uLbia92G4LHKibjttQQCIjtPgAtGIG9CTg00Q3QR9FUz39XLakxia6p4kMw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

接下来我们用星球的`变量定义还原特殊版`和`变量定义还原`两个插件都运行一次,

运行完后我们在运行`删除垃圾代码`插件

![图片](https://mmbiz.qpic.cn/mmbiz_png/1pcia7zHaice3cIuia406kVfYKqBrE052uLFxrlpJjYIA6x3VhhVpn6eMKoovH7jtfibQqRcxuz3pptlcZHibuObIVg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

可以看到运行完后我们的代码减少了一百多行,并且已经很规整了

代码的理解程度也很明了了

```
function hW(operator, a, b) {  
    switch (operator) {  
        case"ﱟﱞﱠ":  
            return a * b;  
        case"ﱟﱞﱣ":  
            return a - b;  
        case"ﱟﱟۥ":  
            return a + b;  
        case"ﱟﱟۦ":  
            return a ^ b;  
    }  
}  
  
var fP = Object["create"](null);  
var kC;  
  
functiongR(uO, pQ, sN, fnLengths = {}) {  
    var eO;  
    var xL = {  
        "ﱟۥ": function () {  
        },  
        "ﱟۦ": function () {  
            var [nM, iF] = kC;  
            var tE = Object["create"](null);  
            var iK;  
  
            functionfJ(uO, fM, pI, fnLengths = {}) {  
                var tZ;  
                var eI = {  
                    "ۦﱟ": function () {  
                        var [gC] = iK;  
                        var xZ = ["\u2027", "\u2028", "\u2029"];  
                        var cG = xZ.length;  
                        var wG = {};  
                        for (var i = 0; i < xZ.length; i++) {  
                            wG[xZ[i]] = i;  
                        }  
                        var pW = '';  
                        for (var pos = 0; pos < iF.length; pos += 6) {  
                            var chunk = iF.substr(pos, 6);  
                            if (chunk.length < 6) break;  
                            var charCode = 0;  
                            for (var j = 0; j < chunk.length; j++) {  
                                var idx = wG[chunk[j]];  
                                if (idx === undefined) return iF;  
                                charCode = hW("ﱟﱟۥ", hW("ﱟﱞﱠ", charCode, cG), idx);  
                            }  
                            pW += String.fromCharCode(charCode);  
                        }  
                        var pT = '';  
                        for (var i = 0; i < pW.length; i++) {  
                            pT += String.fromCharCode(hW("ﱟﱟۦ", pW.charCodeAt(i), 85));  
                        }  
                        return pT;  
                    }  
                };  
                if (fM === "ۥﱣ") {  
                    iK = [];  
                }  
                if (fM === "ۦۥ") {  
                    functioncreateFunction() {  
                        var qA = function (...bE) {  
                            iK = bE;  
                            return eI[uO].apply(this);  
                        };  
                        var hX = fnLengths[uO];  
                        if (hX) {  
                            kC = [qA, hX], gR("ﱟۥ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"];  
                        }  
                        return qA;  
                    }  
  
                    tZ = tE[uO] || (tE[uO] = createFunction());  
                } else {  
                    tZ = eI[uO]();  
                }  
                if (pI === "ۦۦ") {  
                    return {"ۦﱞ": tZ};  
                } else {  
                    return tZ;  
                }  
            }  
  
            var lT = function (kY) {  
                this['_seed'] = kY;  
                this['_state'] = [1, 0, 0];  
                this['_check'] = function () {  
                    return'protected';  
                };  
                this['_pattern1'] = "\\w+ *\\(\\) *{\\w+ *";  
                this['_pattern2'] = "['|\"].+['|\"];? *}";  
            };  
            lT['prototype']['verify'] = function () {  
                var rX = newRegExp(hW("ﱟﱟۥ", this['_pattern1'], this['_pattern2']));  
                var tM = rX['test'](this['_check']['toString']()) ? --this['_state'][1] : --this['_state'][0];  
                returnthis['_validate'](tM);  
            };  
            lT['prototype']['_validate'] = function (rM) {  
                if (!Boolean(~rM)) {  
                    return rM;  
                }  
                returnthis['_execute'](this['_seed']);  
            };  
            lT['prototype']['_execute'] = function (kJ) {  
                for (var i = 0, len = this['_state']['length']; i < len; i++) {  
                    this['_state']['push'](0);  
                    len = this['_state']['length'];  
                }  
                returnkJ(this['_state'][0]);  
            };  
            var iH = newlT(function (kW) {  
                return kW === 1;  
            });  
            try {  
                if (!iH['verify']()) {  
                    return"\0";  
                } else {  
                    try {  
                        if (!(window["location"] === document["location"])) {  
                            return"\0";  
                        }  
                    } catch (e) {  
                        return"\0";  
                    }  
                }  
            } catch (e) {  
                return'';  
            }  
            if (iF) {  
                var firstChar = iF.charCodeAt(0);  
                if (firstChar === 8231 || firstChar === 8232 || firstChar === 8233) {  
                    iF = (iK = [iF], fJ("ۦﱟ"));  
                }  
            }  
            var hT = false;  
            if (nM.length > 0) {  
                var firstCharCode = nM.charCodeAt(0);  
                if (firstCharCode === 8231 || firstCharCode === 8232 || firstCharCode === 8233) {  
                    hT = true;  
                }  
            }  
            var jV = '';  
            if (hT) {  
                var unicodeChars = ["\u2027", "\u2028", "\u2029"];  
                var base = unicodeChars.length;  
                var charToIndex = {};  
                for (var i = 0; i < unicodeChars.length; i++) {  
                    charToIndex[unicodeChars[i]] = i;  
                }  
                for (var pos = 0; pos < nM.length; pos += 6) {  
                    var chunk = nM.substr(pos, 6);  
                    if (chunk.length < 6) break;  
                    var charCode = 0;  
                    for (var j = 0; j < chunk.length; j++) {  
                        var idx = charToIndex[chunk[j]];  
                        if (idx === undefined) {  
                            return"\0";  
                        }  
                        charCode = hW("ﱟﱟۥ", hW("ﱟﱞﱠ", charCode, base), idx);  
                    }  
                    jV += String.fromCharCode(charCode);  
                }  
            } else {  
                jV = nM;  
            }  
            try {  
                var xorEncrypted;  
                try {  
                    xorEncrypted = decodeURIComponent(escape(atob(jV)));  
                } catch (e) {  
                    xorEncrypted = Buffer.from(jV, 'base64').toString('utf8');  
                }  
                var result = '';  
                for (var i = 0; i < xorEncrypted.length; i++) {  
                    result += String.fromCharCode(hW("ﱟﱟۦ", xorEncrypted.charCodeAt(i), iF.charCodeAt(i % iF.length)));  
                }  
                return result;  
            } catch (e) {  
                return"\0";  
            }  
        },  
        "ﱟﱞ": function () {  
            var [yN, qC] = kC;  
            let zO = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")));  
            let fA = 0;  
            while (true) {  
                switch (+zO[fA++]) {  
                    caseparseInt(hW("ﱟﱟۥ", (kC = ["‧ ‧‧‧‧‧ ‧   ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]), (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")))) === 10 && 0:  
                        if (parseInt(hW("ﱟﱟۥ", (kC = ["‧ ‧‧‧‧‧ ‧   ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]))) === 10) {  
                            returnfunction (mV) {  
                                let zW = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")));  
                                let lZ = 0;  
                                while (true) {  
                                    switch (+zW[lZ++]) {  
                                        case0:  
                                            if (String((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))).charCodeAt(0) === 120) {  
                                                return mV = function (iN) {  
                                                    let vM = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")));  
                                                    let bD = 0;  
                                                    while (true) {  
                                                        switch (+vM[bD++]) {  
                                                            case0:  
                                                                iN = function (kP) {  
                                                                    let mM = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")));  
                                                                    let pO = 0;  
                                                                    while (true) {  
                                                                        switch (+mM[pO++]) {  
                                                                            case0:  
                                                                                return kP = newRegExp(hW("ﱟﱟۥ", (kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧  ‧‧‧ ‧‧‧‧‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]))), kP[hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧   ‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧    ‧ ‧   ‧‧    ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), (kC = ["‧‧   ‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")))](mV);  
                                                                        }  
                                                                        break;  
                                                                    }  
                                                                }, function () {  
                                                                    let hK = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")));  
                                                                    let dP = 0;  
                                                                    while (true) {  
                                                                        switch (+hK[dP++]) {  
                                                                            case0:  
                                                                                if (String((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])).charCodeAt(0) === 120 && iN()) {  
                                                                                    (function () {  
                                                                                        let jW = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]));  
                                                                                        let xT = 0;  
                                                                                        while (true) {  
                                                                                            switch (+jW[xT++]) {  
                                                                                                caseArray.isArray([]) && 0:  
                                                                                                    if (typeofdocument !== hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧   ‧‧ ‧   ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["HRQMAgwcAQ==", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), (kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])) && typeofdocument.createElement === hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧    ‧ ‧‧‧‧‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]), (kC = ["‧‧   ‧‧ ‧   ‧‧    ‧‧   ‧‧‧   ‧‧‧  ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), (kC = ["‧‧    ‧ ‧   ‧‧ ‧‧ ‧‧  ‧‧", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")))) {  
                                                                                                        try {  
                                                                                                            for (qC = 0; qC < Number.MAX_VALUE; qC++) {  
                                                                                                                window[hW("ﱟﱟۥ", hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧    ‧‧    ‧‧ ‧‧ ‧‧   ‧", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧    ‧ ‧   ‧‧   ‧‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), (kC = ["‧‧    ‧‧  ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), qC)] = newArray(hW("ﱟﱞﱣ", 4294967296, 1)).fill(0);  
                                                                                                            }  
                                                                                                        } catch (e) {  
                                                                                                            console.error((kC = ["", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]), e);  
                                                                                                        }  
                                                                                                    }  
                                                                                                    continue;  
                                                                                            }  
                                                                                            break;  
                                                                                        }  
                                                                                    })();  
                                                                                }  
                                                                                continue;  
                                                                        }  
                                                                        break;  
                                                                    }  
                                                                }.call(this);  
                                                                continue;  
                                                        }  
                                                        break;  
                                                    }  
                                                }, mV();  
                                            } else {  
                                                returnhW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧   ‧‧ ‧   ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧  ‧‧‧‧  ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), (kC = ["‧‧  ‧‧‧ ‧‧‧ ‧‧   ‧‧ ‧ ‧ ‧‧    ‧ ‧   ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]));  
                                            }  
                                            continue;  
                                    }  
                                    break;  
                                }  
                            }(), yN = 0, function () {  
                                let dN = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]));  
                                let sU = 0;  
                                while (true) {  
                                    switch (+dN[sU++]) {  
                                        casenewDate().getTime() > 0 && 0:  
                                            for (qC = 0; qC < 100; qC++) {  
                                                (function () {  
                                                    let hQ = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]));  
                                                    let uV = 0;  
                                                    while (true) {  
                                                        switch (+hQ[uV++]) {  
                                                            case0:  
                                                                return namedFunction = function () {  
                                                                    let rB = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")));  
                                                                    let jC = 0;  
                                                                    while (true) {  
                                                                        switch (+rB[jC++]) {  
                                                                            case"undefined" === hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧   ‧‧ ‧   ‧‧    ‧‧   ‧‧‧  ‧ ‧ ‧‧‧‧‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]), (kC = ["‧‧    ‧ ‧‧‧ ‧ ‧   ‧‧  ‧‧‧‧  ‧ ‧ ‧‧‧‧‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), (kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])) && 0:  
                                                                                test = function () {  
                                                                                    let fB = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")));  
                                                                                    let aV = 0;  
                                                                                    while (true) {  
                                                                                        switch (+fB[aV++]) {  
                                                                                            case(kC = ["‧‧   ‧‧ ‧   ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")) === (kC = ["‧‧   ‧‧ ‧   ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")) && 0:  
                                                                                                return regExp = newRegExp(hW("ﱟﱟۥ", (kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧  ‧‧‧ ‧‧‧‧‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]))), regExp[hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧   ‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧    ‧ ‧   ‧‧    ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), (kC = ["‧‧   ‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]))](namedFunction);  
                                                                                        }  
                                                                                        break;  
                                                                                    }  
                                                                                }, function () {  
                                                                                    let eX = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]));  
                                                                                    let fX = 0;  
                                                                                    while (true) {  
                                                                                        switch (+eX[fX++]) {  
                                                                                            case0:  
                                                                                                if (test()) {  
                                                                                                    (function () {  
                                                                                                        let aB = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]));  
                                                                                                        let zF = 0;  
                                                                                                        while (true) {  
                                                                                                            switch (+aB[zF++]) {  
                                                                                                                case0:  
                                                                                                                    if (parseInt(hW("ﱟﱟۥ", (kC = ["‧ ‧‧‧‧‧ ‧   ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]), (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")))) === 10 && typeofdocument !== hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧   ‧‧ ‧   ‧‧    ‧‧   ‧‧‧  ‧ ‧ ‧‧‧‧‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]), (kC = ["‧‧    ‧ ‧‧‧ ‧ ‧   ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), (kC = ["‧‧  ‧‧‧ ‧‧‧ ‧ ‧‧  ‧‧   ‧", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])) && typeofdocument.createElement === hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧    ‧ ‧‧‧‧‧ ‧‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧  ‧‧‧ ‧‧‧ ‧‧    ‧ ‧ ‧ ‧‧    ‧ ‧‧‧‧‧  ‧ ‧‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), (kC = ["‧‧  ‧‧‧ ‧‧‧‧‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")))) {  
                                                                                                                        try {  
                                                                                                                            for (qC = 0; qC < Number.MAX_VALUE; qC++) {  
                                                                                                                                window[hW("ﱟﱟۥ", hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧    ‧‧    ‧‧ ‧‧ ‧‧   ‧", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧    ‧ ‧   ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), (kC = ["‧‧  ‧‧‧ ‧  ‧‧‧ ‧‧ ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), qC)] = newArray(hW("ﱟﱞﱣ", 4294967296, 1)).fill(0);  
                                                                                                                            }  
                                                                                                                        } catch (e) {  
                                                                                                                            console.error((kC = ["", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), e);  
                                                                                                                        }  
                                                                                                                    }  
                                                                                                                    continue;  
                                                                                                            }  
                                                                                                            break;  
                                                                                                        }  
                                                                                                    })();  
                                                                                                }  
                                                                                                continue;  
                                                                                        }  
                                                                                        break;  
                                                                                    }  
                                                                                }.call(this);  
                                                                                continue;  
                                                                        }  
                                                                        break;  
                                                                    }  
                                                                }, namedFunction();  
                                                        }  
                                                        break;  
                                                    }  
                                                })();  
                                                yN += qC;  
                                            }  
                                            continue;  
                                    }  
                                    break;  
                                }  
                            }.call(this), console.log(hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧  ‧ ‧‧  ‧ ‧ ‧‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]), (kC = ["‧‧  ‧‧‧ ‧   ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), (kC = ["‧ ‧‧‧ ‧ ‧‧‧‧‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), yN);  
                        } else {  
                            returnhW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧  ‧‧‧ ‧‧‧‧‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧  ‧‧‧‧  ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), (kC = ["‧‧    ‧‧    ‧‧    ‧ ‧   ‧‧   ‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")));  
                        }  
                        continue;  
                }  
                break;  
            }  
        }  
    };  
    if (pQ === "ﱞﱞ") {  
        kC = [];  
    }  
    if (pQ === "ﱞﱟ") {  
        functioncreateFunction() {  
            var tK = function (...oU) {  
                kC = oU;  
                return xL[uO].apply(this);  
            };  
            var bT = fnLengths[uO];  
            if (bT) {  
                pC(tK, bT);  
            }  
            return tK;  
        }  
  
        eO = fP[uO] || (fP[uO] = createFunction());  
    } else {  
        eO = xL[uO]();  
    }  
    if (sN === "ﱞﱠ") {  
        return {"ﱞﱣ": eO};  
    } else {  
        return eO;  
    }  
}  
  
(function (yG, bJ) {  
    let oB = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")));  
    let kA = 0;  
    while (true) {  
        switch (+oB[kA++]) {  
            case0:  
                yG = hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧    ‧    ‧‧ ‧‧  ‧‧    ‧‧   ‧‧ ‧‧‧‧‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧    ‧‧  ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), (kC = ["‧‧    ‧‧   ‧‧ ‧‧  ‧ ‧   ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), bJ = function () {  
                    let yM = (kC = ["‧ ‧‧‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")).split((kC = ["‧‧    ‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]));  
                    let oY = 0;  
                    while (true) {  
                        switch (+yM[oY++]) {  
                            casenewDate().getTime() > 0 && 0:  
                                Function(hW("ﱟﱟۥ", hW("ﱟﱟۥ", yG, (kC = ["‧ ‧‧‧ ‧‧  ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])), Date.now()))();  
                                continue;  
                        }  
                        break;  
                    }  
                }, setInterval(bJ, 1000);  
                continue;  
        }  
        break;  
    }  
})();  
console[hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧  ‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]), (kC = ["‧‧  ‧‧‧‧  ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), (kC = ["‧‧    ‧‧  ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]))](hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["6K6W5LqT56Go55WV", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]), (kC = ["‧‧  ‧ ‧‧  ‧ ‧‧  ‧ ‧‧  ‧ ‧‧  ‧ ‧‧  ‧ ‧‧  ‧ ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), (kC = ["ARQMCQrovqnooKjpmZfmi4k=", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], newgR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])));  
console[hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧  ‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]), (kC = ["‧‧  ‧‧‧‧  ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), (kC = ["‧‧    ‧‧  ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]))](hW("ﱟﱟۥ", hW("ﱟﱟۥ", (kC = ["‧‧    ‧    ‧‧ ‧‧  ‧‧    ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ")), (kC = ["‧‧  ‧‧‧    ‧‧‧ ‧‧ ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ"))), (kC = ["‧ ‧‧  ‧     ‧ ‧ ‧‧‧‧    ‧‧    ‧ ‧   ‧ ‧   ‧ ‧‧ ‧", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"])));  

```

剩下的就剩如下类型的混淆

```
kC = ["‧‧  ‧‧‧   ‧ ‧‧ ‧  ‧‧ ‧  ", "‧‧  ‧ ‧‧  ‧ ‧‧ ‧ ‧‧‧    ‧‧   ‧‧‧   ‧‧‧    ‧‧   ‧‧‧ ‧‧ ‧‧ ‧  "], gR("ﱟۦ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"]  

```

这种也很简单,但是我们得写一点点代码

我们把gR先单独复制出来,在浏览器和node测试

![图片](https://mmbiz.qpic.cn/mmbiz_png/1pcia7zHaice3cIuia406kVfYKqBrE052uLFGd0o6QINn1Mdpe7iaHjEslMZ5U556XJBK6cSIESQysp6tvBQvnJ4FA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

  

可以看到结果是一摸一样的,因此我们直接调用还原

ast代码如下

```
traverse(ast, {  
    SequenceExpression(path){  
        let {node,scope}=path  
        let {expressions}=node  
        if(!types.isAssignmentExpression(expressions[0]) && !types.isCallExpression(expressions[1])){  
            return  
        }  
        if(types.isCallExpression(expressions[1])){  
            let {callee,arguments}=expressions[1]  
            if(!types.isIdentifier(callee,{name:'gR'})){  
                return;  
            }  
            if(arguments.length === 1 && types.isStringLiteral(arguments[0])){  
                let data=eval(path.toString());  
                path.replaceWith(types.valueToNode(data))  
            }  
        }  
        if(types.isMemberExpression(expressions[1]) && types.isAssignmentExpression(expressions[0])){  
            try{  
                let data=eval(path.toString());  
                path.replaceWith(types.valueToNode(data))  
            }catch(e){}  
        }  
    }  
  
});  

```

```
function hW(operator, a, b) {  
switch (operator) {  
    case"ﱟﱞﱠ":  
      return a * b;  
    case"ﱟﱞﱣ":  
      return a - b;  
    case"ﱟﱟۥ":  
      return a + b;  
    case"ﱟﱟۦ":  
      return a ^ b;  
  }  
}  
var fP = Object["create"](null);  
var kC;  
functiongR(uO, pQ, sN, fnLengths = {}) {  
var eO;  
var xL = {  
    "ﱟۥ": function () {},  
    "ﱟۦ": function () {  
      var [nM, iF] = kC;  
      var tE = Object["create"](null);  
      var iK;  
      functionfJ(uO, fM, pI, fnLengths = {}) {  
        var tZ;  
        var eI = {  
          "ۦﱟ": function () {  
            var [gC] = iK;  
            var xZ = ["\u2027", "\u2028", "\u2029"];  
            var cG = xZ.length;  
            var wG = {};  
            for (var i = 0; i < xZ.length; i++) {  
              wG[xZ[i]] = i;  
            }  
            var pW = '';  
            for (var pos = 0; pos < iF.length; pos += 6) {  
              var chunk = iF.substr(pos, 6);  
              if (chunk.length < 6) break;  
              var charCode = 0;  
              for (var j = 0; j < chunk.length; j++) {  
                var idx = wG[chunk[j]];  
                if (idx === undefined) return iF;  
                charCode = 0 / 0;  
              }  
              pW += String.fromCharCode(charCode);  
            }  
            var pT = '';  
            for (var i = 0; i < pW.length; i++) {  
              pT += String.fromCharCode(hW("ﱟﱟۦ", pW.charCodeAt(i), 85));  
            }  
            return pT;  
          }  
        };  
        if (fM === "ۥﱣ") {  
          iK = [];  
        }  
        if (fM === "ۦۥ") {  
          functioncreateFunction() {  
            var qA = function (...bE) {  
              iK = bE;  
              return eI[uO].apply(this);  
            };  
            var hX = fnLengths[uO];  
            if (hX) {  
              kC = [qA, hX], gR("ﱟۥ", "ﱞۦ", "ﱞﱠ")["ﱞﱣ"];  
            }  
            return qA;  
          }  
          tZ = tE[uO] || (tE[uO] = createFunction());  
        } else {  
          tZ = eI[uO]();  
        }  
        if (pI === "ۦۦ") {  
          return {  
            "ۦﱞ": tZ  
          };  
        } else {  
          return tZ;  
        }  
      }  
      var lT = function (kY) {  
        this['_seed'] = kY;  
        this['_state'] = [1, 0, 0];  
        this['_check'] = function () {  
          return'protected';  
        };  
        this['_pattern1'] = "\\w+ *\\(\\) *{\\w+ *";  
        this['_pattern2'] = "['|\"].+['|\"];? *}";  
      };  
      lT['prototype']['verify'] = function () {  
        var rX = newRegExp(0 / 0);  
        var tM = rX['test'](this['_check']['toString']()) ? --this['_state'][1] : --this['_state'][0];  
        returnthis['_validate'](tM);  
      };  
      lT['prototype']['_validate'] = function (rM) {  
        if (!Boolean(~rM)) {  
          return rM;  
        }  
        returnthis['_execute'](this['_seed']);  
      };  
      lT['prototype']['_execute'] = function (kJ) {  
        for (var i = 0, len = this['_state']['length']; i < len; i++) {  
          this['_state']['push'](0);  
          len = this['_state']['length'];  
        }  
        returnkJ(this['_state'][0]);  
      };  
      var iH = newlT(function (kW) {  
        return kW === 1;  
      });  
      try {  
        if (!iH['verify']()) {  
          return"\0";  
        } else {  
          try {  
            if (!(window["location"] === document["location"])) {  
              return"\0";  
            }  
          } catch (e) {  
            return"\0";  
          }  
        }  
      } catch (e) {  
        return'';  
      }  
      if (iF) {  
        var firstChar = iF.charCodeAt(0);  
        if (firstChar === 8231 || firstChar === 8232 || firstChar === 8233) {  
          iF = (iK = [iF], fJ("ۦﱟ"));  
        }  
      }  
      var hT = false;  
      if (nM.length > 0) {  
        var firstCharCode = nM.charCodeAt(0);  
        if (firstCharCode === 8231 || firstCharCode === 8232 || firstCharCode === 8233) {  
          hT = true;  
        }  
      }  
      var jV = '';  
      if (hT) {  
        var unicodeChars = ["\u2027", "\u2028", "\u2029"];  
        var base = unicodeChars.length;  
        var charToIndex = {};  
        for (var i = 0; i < unicodeChars.length; i++) {  
          charToIndex[unicodeChars[i]] = i;  
        }  
        for (var pos = 0; pos < nM.length; pos += 6) {  
          var chunk = nM.substr(pos, 6);  
          if (chunk.length < 6) break;  
          var charCode = 0;  
          for (var j = 0; j < chunk.length; j++) {  
            var idx = charToIndex[chunk[j]];  
            if (idx === undefined) {  
              return"\0";  
            }  
            charCode = 0 / 0;  
          }  
          jV += String.fromCharCode(charCode);  
        }  
      } else {  
        jV = nM;  
      }  
      try {  
        var xorEncrypted;  
        try {  
          xorEncrypted = decodeURIComponent(escape(atob(jV)));  
        } catch (e) {  
          xorEncrypted = Buffer.from(jV, 'base64').toString('utf8');  
        }  
        var result = '';  
        for (var i = 0; i < xorEncrypted.length; i++) {  
          result += String.fromCharCode(hW("ﱟﱟۦ", xorEncrypted.charCodeAt(i), iF.charCodeAt(i % iF.length)));  
        }  
        return result;  
      } catch (e) {  
        return"\0";  
      }  
    },  
    "ﱟﱞ": function () {  
      var [yN, qC] = kC;  
      let zO = "0".split("|");  
      let fA = 0;  
      while (true) {  
        switch (+zO[fA++]) {  
          caseparseInt("10") === 10 && 0:  
            if (parseInt("10") === 10) {  
              returnfunction (mV) {  
                let zW = "0".split("|");  
                let lZ = 0;  
                while (true) {  
                  switch (+zW[lZ++]) {  
                    case0:  
                      if (String("x").charCodeAt(0) === 120) {  
                        return mV = function (iN) {  
                          let vM = "0".split("|");  
                          let bD = 0;  
                          while (true) {  
                            switch (+vM[bD++]) {  
                              case0:  
                                iN = function (kP) {  
                                  let mM = "0".split("|");  
                                  let pO = 0;  
                                  while (true) {  
                                    switch (+mM[pO++]) {  
                                      case0:  
                                        return kP = newRegExp("\\n"), kP["test"](mV);  
                                    }  
                                    break;  
                                  }  
                                }, function () {  
                                  let hK = "0".split("|");  
                                  let dP = 0;  
                                  while (true) {  
                                    switch (+hK[dP++]) {  
                                      case0:  
                                        if (String("x").charCodeAt(0) === 120 && iN()) {  
                                          (function () {  
                                            let jW = "0".split("|");  
                                            let xT = 0;  
                                            while (true) {  
                                              switch (+jW[xT++]) {  
                                                caseArray.isArray([]) && 0:  
                                                  if (typeofdocument !== "undefined" && typeofdocument.createElement === "function") {  
                                                    try {  
                                                      for (qC = 0; qC < Number.MAX_VALUE; qC++) {  
                                                        window["__mem_undefined"] = newArray(4294967295).fill(0);  
                                                      }  
                                                    } catch (e) {  
                                                      console.error("", e);  
                                                    }  
                                                  }  
                                                  continue;  
                                              }  
                                              break;  
                                            }  
                                          })();  
                                        }  
                                        continue;  
                                    }  
                                    break;  
                                  }  
                                }.call(this);  
                                continue;  
                            }  
                            break;  
                          }  
                        }, mV();  
                      } else {  
                        return"uonQwV";  
                      }  
                      continue;  
                  }  
                  break;  
                }  
              }(), yN = 0, function () {  
                let dN = "0".split("|");  
                let sU = 0;  
                while (true) {  
                  switch (+dN[sU++]) {  
                    casenewDate().getTime() > 0 && 0:  
                      for (qC = 0; qC < 100; qC++) {  
                        (function () {  
                          let hQ = "0".split("|");  
                          let uV = 0;  
                          while (true) {  
                            switch (+hQ[uV++]) {  
                              case0:  
                                return namedFunction = function () {  
                                  let rB = "0".split("|");  
                                  let jC = 0;  
                                  while (true) {  
                                    switch (+rB[jC++]) {  
                                      case"undefined" === "undefined" && 0:  
                                        test = function () {  
                                          let fB = "0".split("|");  
                                          let aV = 0;  
                                          while (true) {  
                                            switch (+fB[aV++]) {  
                                              case"a" === "a" && 0:  
                                                return regExp = newRegExp("\\n"), regExp["test"](namedFunction);  
                                            }  
                                            break;  
                                          }  
                                        }, function () {  
                                          let eX = "0".split("|");  
                                          let fX = 0;  
                                          while (true) {  
                                            switch (+eX[fX++]) {  
                                              case0:  
                                                if (test()) {  
                                                  (function () {  
                                                    let aB = "0".split("|");  
                                                    let zF = 0;  
                                                    while (true) {  
                                                      switch (+aB[zF++]) {  
                                                        case0:  
                                                          if (parseInt("10") === 10 && typeofdocument !== "undefined" && typeofdocument.createElement === "function") {  
                                                            try {  
                                                              for (qC = 0; qC < Number.MAX_VALUE; qC++) {  
                                                                window["__mem_undefined"] = newArray(4294967295).fill(0);  
                                                              }  
                                                            } catch (e) {  
                                                              console.error("", e);  
                                                            }  
                                                          }  
                                                          continue;  
                                                      }  
                                                      break;  
                                                    }  
                                                  })();  
                                                }  
                                                continue;  
                                            }  
                                            break;  
                                          }  
                                        }.call(this);  
                                        continue;  
                                    }  
                                    break;  
                                  }  
                                }, namedFunction();  
                            }  
                            break;  
                          }  
                        })();  
                        yN += qC;  
                      }  
                      continue;  
                  }  
                  break;  
                }  
              }.call(this), console.log("Sum:", yN);  
            } else {  
              return"noONHS";  
            }  
            continue;  
        }  
        break;  
      }  
    }  
  };  
if (pQ === "ﱞﱞ") {  
    kC = [];  
  }  
if (pQ === "ﱞﱟ") {  
    functioncreateFunction() {  
      var tK = function (...oU) {  
        kC = oU;  
        return xL[uO].apply(this);  
      };  
      var bT = fnLengths[uO];  
      if (bT) {  
        pC(tK, bT);  
      }  
      return tK;  
    }  
    eO = fP[uO] || (fP[uO] = createFunction());  
  } else {  
    eO = xL[uO]();  
  }  
if (sN === "ﱞﱠ") {  
    return {  
      "ﱞﱣ": eO  
    };  
  } else {  
    return eO;  
  }  
}  
(function (yG, bJ) {  
let oB = "0".split("|");  
let kA = 0;  
while (true) {  
    switch (+oB[kA++]) {  
      case0:  
        yG = "debugger", bJ = function () {  
          let yM = "0".split("|");  
          let oY = 0;  
          while (true) {  
            switch (+yM[oY++]) {  
              casenewDate().getTime() > 0 && 0:  
                Function("undefined;undefined")();  
                continue;  
            }  
            break;  
          }  
        }, setInterval(bJ, 1000);  
        continue;  
    }  
    break;  
  }  
})();  
console["log"]("该代码由spiderdemo进行防护");  
console["log"]("Hello World");  

```

###### 本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责，若有侵权，请在公众号联系作者立即删除！