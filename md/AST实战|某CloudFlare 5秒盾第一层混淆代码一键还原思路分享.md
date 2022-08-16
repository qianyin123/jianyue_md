> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwNTY1OTg0MQ==&mid=2647563690&idx=1&sn=d1bee149d5633e225116ae63a0fb3404&chksm=83237004b454f9125ebff1d0a29494e42f541076ec70681bc43254e63ddf251772366add706e&scene=126&&sessionid=1660619592#rd)

**关注它，不迷路。**

  

  

*   本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者立即删除！
    
      
    
      
    

一.实战地址
------

```
https://www.e-food.gr/
```

  

加载的混淆js地址，但是是动态的哈:

```
https://www.e-food.gr/cdn-cgi/challenge-platform/h/g/orchestrate/jsch/v1?ray=7334095989227e52
```

  

混淆代码如图所示:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR323oeG8y9Zq3HlSGGd50y41NpfJgvHicwJj4GAZFTYgpeDHrBAT16iaaXhf3bfgiceekiaIab9JZnxCw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

这种一看就是变种的ob混淆，一个大数组 + 移位函数 + 解密函数。  

二.还原分析
------

还原第一步:  

**实参为字面量的函数调用，能还原，先还原**:  

c('0x1b7')     ===>   'document'

还原后的结果如下:  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR323oeG8y9Zq3HlSGGd50y4cGO8NCPhsIpJHy0WAoia3ibVLnOFMBTFcsCHVoSpfxt8vsRTFFguwICg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

还原第二步:

**代码结够能调整的，先调整:**  

一般调整的是逗号表达式，但是在处理逗号表达式之前，我们需要先处理一些block，好多都没有{}，这是不规范的，得先给它补上:  

```
`if (0 == J && (J = Math.pow(2, L),``L++),``I[V])` `V = I[V];``else {` `if (V === K)` `V = B["VwayE"](N, N["charAt"](0));` `else` `return null;``}`
```

===>  

  

```
`if (0 == J && (J = Math.pow(2, L),``L++),``I[V]) {` `V = I[V];``} else {` `if (V === K) {` `V = B["VwayE"](N, N["charAt"](0));` `} else {` `return null;` `}``}`
```

  

还原第三步:

**逗号表达式的还原:**

```
`if (A = {}, A["wzYNQ"] = function (K, L) {` `return K * L;` `}, A["qobwJ"] = function (K, L) {` `return K == L;` `}, A["jCUes"] = function (K, L, M, N) {` `return K(L, M, N);` `}, A["MUwqS"] = function (K, L) {` `return K !== L;` `}, A["vhpCJ"] = function (K, L) {` `return K + L;` `}, A["PfjYi"] = function (K, L) {` `return K === L;` `}, A["JusLJ"] = "McOvX", A["WdCju"] = "lbAdP", A["DBACO"] = "&#x8BF7;&#x542F;&#x7528;Cookie&#x5E76;&#x91CD;&#x65B0;&#x52A0;&#x8F7D;&#x9875;&#x9762;&#x3002;", A["HBUlA"] = function (K, L) {` `return K(L);` `}, A["UxpOE"] = "cf_chl_", A["IhiBw"] = function (K, L) {` `return K + L;` `}, A["xOnNk"] = function (K, L) {` `return K + L;` `}, B = A, B["MUwqS"](d["_cf_chl_opt"]["cLt"], 'd')) {` `d["_cf_chl_opt"]["cLt"] = 'd';` `} else {` `return void 0;` `}if (A = {}, A["wzYNQ"] = function (K, L) {` `return K * L;` `}, A["qobwJ"] = function (K, L) {` `return K == L;` `}, A["jCUes"] = function (K, L, M, N) {` `return K(L, M, N);` `}, A["MUwqS"] = function (K, L) {` `return K !== L;` `}, A["vhpCJ"] = function (K, L) {` `return K + L;` `}, A["PfjYi"] = function (K, L) {` `return K === L;` `}, A["JusLJ"] = "McOvX", A["WdCju"] = "lbAdP", A["DBACO"] = "&#x8BF7;&#x542F;&#x7528;Cookie&#x5E76;&#x91CD;&#x65B0;&#x52A0;&#x8F7D;&#x9875;&#x9762;&#x3002;", A["HBUlA"] = function (K, L) {` `return K(L);` `}, A["UxpOE"] = "cf_chl_", A["IhiBw"] = function (K, L) {` `return K + L;` `}, A["xOnNk"] = function (K, L) {` `return K + L;` `}, B = A, B["MUwqS"](d["_cf_chl_opt"]["cLt"], 'd')) {` `d["_cf_chl_opt"]["cLt"] = 'd';` `} else {` `return void 0;``}`
```

===>  

```
`A = {};` `A["uVDHf"] = "gncnt";` `A["Ehaxm"] = "7|1|2|6|8|3|0|5|4";` `A["iIBCc"] = "data-translate";` `A.NwOfM = "error code: 1020";` `A["FQgtt"] = function (H, I) {` `return H < I;` `};` `A["dpEeO"] = "none";` `B = A;` `C = e.getElementById("challenge-form");` `if (C) {` `if ("JwQCM" !== B["uVDHf"]) {` `D = B["Ehaxm"]["split"]('|');` `for (E = 0; !![];) {` `switch (D[E++]) {` `case '0':` `F["setAttribute"](B["iIBCc"], "error");` `continue;` `case '1':` `G["style"]["display"] = "none";` `continue;` `case '2':` `C["appendChild"](G);` `continue;` `case '3':` `F["className"] = 'text-gray-600';` `continue;` `case '4':` `G["appendChild"](F);` `continue;` `case '5':` `F["innerText"] = B["NwOfM"];` `continue;` `case '6':` `F = e["createElement"]("span");` `continue;` `case '7':` `G = e["createElement"]("span");` `continue;` `case '8':` `B["FQgtt"](Math["random"](), .25) && (F.style["display"] = B["dpEeO"]);` `continue;` `}` `break;` `}` `} else {` `function H() {` `return void 0;` `}` `}` `}`
```

  

还原第四步:

**object对象的还原。**

从上面的代码就知道下一步应该还原object了。  

===>  

```
`if (C) {` `if ("JwQCM" !== "gncnt") {` `D = "7|1|2|6|8|3|0|5|4"["split"]('|');` `for (E = 0; true;) {` `switch (D[E++]) {` `case '0':` `F["setAttribute"]("data-translate", "error");` `continue;` `case '1':` `G["style"]["display"] = "none";` `continue;` `case '2':` `C["appendChild"](G);` `continue;` `case '3':` `F["className"] = 'text-gray-600';` `continue;` `case '4':` `G["appendChild"](F);` `continue;` `case '5':` `F["innerText"] = "error code: 1020";` `continue;` `case '6':` `F = e["createElement"]("span");` `continue;` `case '7':` `G = e["createElement"]("span");` `continue;` `case '8':` `Math["random"]() < .25 && (F["style"]["display"] = "none");` `continue;` `}` `break;` `}` `} else {` `function H() {` `return void 0;` `}` `}` `}`
```

  

如果直接用星球里的插件，会发现还有一个object没有还原:    

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Lll8tx0MDR323oeG8y9Zq3HlSGGd50y4wolXMtfYxlN5JiaK5Uw0GxIWTib47GibecpnElnV38qiaCzQSHUwLPLQLw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

分析后发现，它是这样的:  

```
`(function (B, H, I, J, K, L, M, R, Q, P, O, N, C) {` `B["hgTsd"] = "0|13|5|7|8|10|1|11|2|6|14|9|12|4|15|16|3";` `......` `})({}, /^[\],:{}\s]*$/, /\\(?:["\\\/bfnrt]|u[0-9a-fA-F]{4})/g, /"[^"\\\n\r]*"|true|false|null|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?/g, /(?:^|:|,)(?:\s*\[)+/g, /[\\"\u0000-\u001f\u007f-\u009f\u00ad\u0600-\u0604\u070f\u17b4\u17b5\u200c-\u200f\u2028-\u202f\u2060-\u206f\ufeff\ufff0-\uffff]/g, /[\u0000\u00ad\u0600-\u0604\u070f\u17b4\u17b5\u200c-\u200f\u2028-\u202f\u2060-\u206f\ufeff\ufff0-\uffff]/g);`
```

  

难怪没办法还原，B的赋值是在参数里面，因此，需要点小技巧,变成:  

```
`(function (B, H, I, J, K, L, M, R, Q, P, O, N, C) {` `B = {};` `B["hgTsd"] = "0|13|5|7|8|10|1|11|2|6|14|9|12|4|15|16|3";` `......` `})({}, /^[\],:{}\s]*$/, /\\(?:["\\\/bfnrt]|u[0-9a-fA-F]{4})/g, /"[^"\\\n\r]*"|true|false|null|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?/g, /(?:^|:|,)(?:\s*\[)+/g, /[\\"\u0000-\u001f\u007f-\u009f\u00ad\u0600-\u0604\u070f\u17b4\u17b5\u200c-\u200f\u2028-\u202f\u2060-\u206f\ufeff\ufff0-\uffff]/g, /[\u0000\u00ad\u0600-\u0604\u070f\u17b4\u17b5\u200c-\u200f\u2028-\u202f\u2060-\u206f\ufeff\ufff0-\uffff]/g);`
```

  

这样，就可以一起还原啦。  

**tips:当你发现你的插件没办法兼容所有的情况，那你可以使用点小技巧，让插件能将其还原，但是不能影响它的执行逻辑。**  

还原第四步:

**控制流的还原。**

**===>**

```
`if (C) {` `if ("JwQCM" !== "gncnt") {` `G = e["createElement"]("span");` `G["style"]["display"] = "none";` `C["appendChild"](G);` `F = e["createElement"]("span");` `Math["random"]() < .25 && (F["style"]["display"] = "none");` `F["className"] = 'text-gray-600';` `F["setAttribute"]("data-translate", "error");` `F["innerText"] = "error code: 1020";` `G["appendChild"](F);` `} else {` `function H() {` `return void 0;` `}` `}` `}`
```

还原到这里，基本就差不多了，剩下的是一些收尾工作，例如删除deadcode等。  

还原第五步:

删除deadcode等收尾工作。

```
`function E() {` `if ("vXRMB" === "gIBma") {` `function S(a5, a4, a3, a2, a1, a0, Z, Y, X, W, V, U, T) {` `V = D[0];` `a5 = E[1];` `a0 = F[2];` `a1 = G[3];` `a2 = H[4];` `a3 = I[5];` `W = J[6];` `a4 = K[7];` `for (a0 = 0; 64 > a0; (a5 = V, (a0 = a5, (a1 = a0, (a2 = a5(a1, X), (a3 = a2, (W = a3, (a4 = W, (Y = a5(a4, V & a5 ^ V & a0 ^ a5 & a0), (a4 = a2(a4, 2) ^ a2(a4, 13) ^ a2(a4, 22), (a4 = V, (X = a5(a5(a5(a5(a4, X), a2 & a3 ^ ~a2 & W), V[a0]), Y[a0]), (X = a2(X, 6) ^ a2(X, 11) ^ a2(X, 25), (X = a2, (Y[X] = Y, (16 > a0 ? Y = a3[a0 + a4] : (Z = a2(Z, 7) ^ a2(Z, 18) ^ Z >>> 3, (Z = Y[a0 - 15], (Y = a5(Y, Y[a0 - 7]), (Y = a2(Y, 17) ^ a2(Y, 19) ^ Y >>> 10, (Y = Y[a0 - 2], Y = a5(a5(Y, Z), Y[a0 - 16])))))), (X = a0, V = a5(X, Y))))))))))))))))), a0++) {` `;` `}` `N[0] = a5(V, O[0]);` `P[1] = a5(a5, Q[1]);` `R[2] = a5(a0, S[2]);` `T[3] = a5(a1, U[3]);` `V[4] = a5(a2, W[4]);` `X[5] = a5(a3, Y[5]);` `Z[6] = a5(W, a0[6]);` `a1[7] = a5(a4, a2[7]);` `}` `} else {` `return this["valueOf"]();` `}` `}`
```

===>

  

```
`function E() {` `return this["valueOf"]();` `}`
```

  

到这里，基本就还原了。

整个还原的完整代码我放星球了，供大家参考学习，地址:

```
https://t.zsxq.com/04j6a6Qzb
```

  

  

三.交流学习
------

  

加我好友，拉你进群，现在快开6群了，学习氛围浓，注意，严禁讨论破解相关的话题。  

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Lll8tx0MDR0kCx9rvzFflV5uIzLHZwQ7BxgpibUhX1j1fD0KU6TUmYIiaxBP5ibqWibBqOTGn5cj200eW6MEZEB7Qw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)