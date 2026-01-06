> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hKK_Bz5Z_rvshcnm2ZGvtg)

本文章中所有内容仅供学习交流，不可用于任何商业用途和非法用途，否则后果自负，如有侵权，请联系作者立即删除！

一.插桩**:**
---------

  

目标接口是搜索，很轻松就能找到最终的加密位置

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8Mum5ibG1E5Pic5VxtQhnU67vZibaF2yq0HcdZb8kALUEb0xOycMQ3Bars9FQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

进方法发现是普通的混淆代码和很多段vmp组合的，手动插桩vmp太耗时间了，就让ai写了个针对这段vmp的ast

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumRhiaOUicH4eOKgRnFPRHXfjX1AtzCwKv4g5ic3mAFodaS60VibAHGiaBFkA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

ai写的，写得很一般，配合之前文章的插桩框架勉强能用，有需要的可以优化下

```
`let functionCallIndex = 0;``function isInsideAddlog(path) {` `return !!path.findParent(p =>`  `p.isCallExpression() && types.isIdentifier(p.node.callee, { name: "addlog" })` `);``}``const visitor = {` `ForStatement(path) {` `if (path.node.init === null && path.node.test === null && path.node.update === null) {` `path.traverse({` `// 1. 二元运算：不标序号，结构为 (L, op, R, res)` `BinaryExpression(binPath) {` `if (binPath.node._processed || isInsideAddlog(binPath)) return;` `const { left, right, operator } = binPath.node;` `if (types.isNumericLiteral(right) && right.value <= 2) return;` `const idL = binPath.scope.generateUid("l");` `const idR = binPath.scope.generateUid("r");` `const idRes = binPath.scope.generateUid("res");` `const opNode = types.binaryExpression(operator, types.identifier(idL), types.identifier(idR));` ``const binTemplate = template(` `` `(function() {` `var %%ID_L%% = %%LEFT%%;` `var %%ID_R%% = %%RIGHT%%;` `var %%ID_RES%% = %%OP_NODE%%;` `if (typeof addlog !== 'undefined' && !addlog.lock) {` `addlog(%%ID_L%%, %%OP_STR%%, %%ID_R%%, %%ID_RES%%);` `}` `return %%ID_RES%%;` `}.call(this))` `` `);`` `const newNode = binTemplate({` `ID_L: types.identifier(idL),` `ID_R: types.identifier(idR),` `ID_RES: types.identifier(idRes),` `LEFT: left,` `RIGHT: right,` `OP_NODE: opNode,` `OP_STR: types.stringLiteral(operator)` `}).expression;` `newNode._processed = true;` `binPath.replaceWith(newNode);` `binPath.skip();` `},` `// 2. 函数调用：【标序号】，结构为 (序号, obj, method, args, res)` `CallExpression(callPath) {` `if (callPath.node._processed || isInsideAddlog(callPath)) return;` `if (types.isIdentifier(callPath.node.callee, { name: "addlog" })) return;` `const node = callPath.node;` `const currentIndex = ++functionCallIndex; // 仅在这里递增序号` `const idRes = callPath.scope.generateUid("ret");` `const idObj = callPath.scope.generateUid("obj");`  `// 预捕获参数，防止 _$fU++ 副作用` `const argVars = node.arguments.map((arg, i) => ({` ``id: callPath.scope.generateUid(`p${i}`),`` `node: arg` `}));` `let callerNode = types.identifier("undefined");` `let methodLogName = types.stringLiteral("DirectCall");` `let callExpression;` `if (types.isMemberExpression(node.callee)) {` `callerNode = node.callee.object;` `methodLogName = types.isIdentifier(node.callee.property)`  `? types.stringLiteral(node.callee.property.name)`  `: node.callee.property;`  `callExpression = types.callExpression(` `types.memberExpression(` `types.identifier(idObj),`  `node.callee.property,`  `node.callee.computed || !types.isIdentifier(node.callee.property)` `),` `argVars.map(v => types.identifier(v.id))` `);` `} else {` `callerNode = node.callee;` `callExpression = types.callExpression(` `types.identifier(idObj),`  `argVars.map(v => types.identifier(v.id))` `);` `}` `const iifeBody = [` `types.variableDeclaration("var", [types.variableDeclarator(types.identifier(idObj), callerNode)]),` `...argVars.map(v => types.variableDeclaration("var", [types.variableDeclarator(types.identifier(v.id), v.node)])),` `types.variableDeclaration("var", [types.variableDeclarator(types.identifier(idRes), callExpression)]),` `types.ifStatement(` `types.logicalExpression("&&",`  `types.binaryExpression("!==", types.unaryExpression("typeof", types.identifier("addlog")), types.stringLiteral("undefined")),` `types.unaryExpression("!", types.memberExpression(types.identifier("addlog"), types.identifier("lock")))` `),` `types.expressionStatement(` `types.callExpression(types.identifier("addlog"), [` `types.numericLiteral(currentIndex), // 传入序号` `types.identifier(idObj),` `methodLogName,` `types.arrayExpression(argVars.map(v => types.identifier(v.id))),` `types.identifier(idRes)` `])` `)` `),` `types.returnStatement(types.identifier(idRes))` `];` `const newNode = types.callExpression(` `types.memberExpression(` `types.functionExpression(null, [], types.blockStatement(iifeBody)),` `types.identifier("call")` `),` `[types.thisExpression()]` `);` `newNode._processed = true;` `callPath.replaceWith(newNode);` `callPath.skip();` `},` `// 3. 复合赋值：不标序号，结构为 (old, op, val, res)` `AssignmentExpression(assignPath) {` `if (assignPath.node._processed || assignPath.node.operator === '=' || isInsideAddlog(assignPath)) return;` `const { left, right, operator } = assignPath.node;` `const idOld = assignPath.scope.generateUid("old");` `const idVal = assignPath.scope.generateUid("val");` `const idRes = assignPath.scope.generateUid("res");` `const opNode = types.binaryExpression(operator.slice(0, -1), types.identifier(idOld), types.identifier(idVal));` ``const assignTemplate = template(` `` `(function() {` `var %%OLD%% = %%LEFT%%;` `var %%VAL%% = %%RIGHT%%;` `var %%RES%% = %%OP_NODE%%;` `if (typeof addlog !== 'undefined' && !addlog.lock) {` `addlog(%%OLD%%, %%OP_STR%%, %%VAL%%, %%RES%%);` `}` `%%LEFT%% = %%RES%%;` `return %%RES%%;` `}.call(this))` `` `);`` `const newNode = assignTemplate({` `OLD: types.identifier(idOld),` `VAL: types.identifier(idVal),` `RES: types.identifier(idRes),` `LEFT: left,` `RIGHT: right,` `OP_NODE: opNode,` `OP_STR: types.stringLiteral(operator)` `}).expression;` `newNode._processed = true;` `assignPath.replaceWith(newNode);` `assignPath.skip();` `}` `});` `}` `}``};`
```

日志全部拿下来，大概就有个20w行左右，最终生成的结果可以分为这几段，一段一段来分析

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8Mum3ibuf7Zomwz9SYBYVrcVeNBEVf2m8hcHiaRJpNyVNnhKJIEfoygn0XZA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

二.加密分析**:**
-----------

  

第一段就是个时间，第二段直接在日志里面搜索第一次出现的位置，往上稍微分析下

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumWUhib6ayjvRPz7sxeJAwI2ynpwZ6dlKvsuicMyARsIFxyumhJDtUlbfw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

发现是从这个字符串，经过一些运算，变成最终结果mz9iw6m3aj0jdd63

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8Mum12LnJlfez17QbRqrAuiaYHHoibyaPbEphdn7cCmiciccL29kFsfslmH4zg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

直接把这个开始的地方，和最终生成的地方，日志单独拿出来，丢给ai，这个是我之间搞的所以结果不一样，很轻松就把这个搞定了

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8Mumfsg6IR13WjymVWfcFNSGYAjhElNibvM8HhQv1HVgv23PNUbznibOnsXA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

后面验证过也没问题，利用ai来分析尽量一小段一小段的来，多了容易出错

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumhmOLMSuRiclsnk1Rxibj6LiacqATyo5QicNe6ibrzf9qZ2CNeia6WzXjicIyQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

接着分析这段是怎么来的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumofGmxu4sKV5I0461cGIrZvdp4UcAu2UfmdAft8Qbe3a3DL0QCNYPRA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

把流程记录下来，先看看pwd是怎么来的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumeKKWxkTlUUbze59PpHuLPeUHvnrs1KPC8REwtmNfqnkAkGc54L13Dw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

是通过不是vmp里面的代码生成的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumZ1vtcrszQK27NAg09icDbPRaHcbibB3qX8uPkKXE9hZkszozb9HxPZKw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)

我在每段插桩日志前面标注的代码具体的位置，可以直接找到位置

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumMxpU5Imlia7snzF78E6krzgRVZH4Sib6iaLricaW70QDa1z3grLGlupicJA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=10)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumQdKfPEfwmxnRv3d13ussy5ZNWfJfia8K1cnRwk0jBgdC1MjjFuzdYAg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=11)

进方法，代码扣下来就行了

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8Mum0UYDejdkpqoVb8gaEwlvoCJlRAkhfNABf7c1mpuK3WDtGQr8uWeuxw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=12)

找找这个num值怎么来的，还是跟之前一样，往上找一找，丢给ai，就是从这里开始的size值怎来的都一样能给你分析出来，t6d0jhqw3p是个固定值

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumXAgLia1qlxeXRmzWfHc5UwwI7CCPnG5nyTdySrjk1p25O4X9L4WwVhQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=13)

这个值大概就是这个流程就能搞定，第三段是个appid，不同的接口id不一样，看看第四段，直接日志搜索第一次出现的位置

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumP33icZ9QMD5jRCibfZnFsMgUV7ibVCuvlCv0LEwS3ibHQVZuW3RPmqdmIQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=14)

好几段拼接的，先看看M3gxKzNrMktZ怎么来的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8Mumhf1JG7pkMxH3VLEEnNs5nm14NNpmUTtY8ib4hTlPwpnbK5JReia5AVRw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=15)

经过了一个一大段运算来的，也可以跟上次一样直接丢ai，分析下

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumshrLYtWbRNt4arqzyKK1FmNRjHdWNAWNJyBFYCAunByCosbicUm6Ohg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=16)

这里其实就是根据这个字符串，进行了标准的base64，明文字符串也是两段拼接，后面一段是12位随机数，取了一个前4位

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumCNtia2juqdKesh1TXia8FaU3wYRycaicCJVHgjqog8x6ibw8wPQiaueLxNA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=17)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumvfL3clic5q05CoFqpLT9xibtXGG4GKxyzuagprsPa2sKr3bPm48eqkkg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=18)

3x1+3，是根据两个数组，进行随机拼接，具体算法可以丢ai分析

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumkhcZKvf87S1lbQlDb9P1jfed1M0P4e6DuxxHDUQCicdRfZgPicar6w2A/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=19)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8Mumtxl1hF54tOO59s1emJHZFDL48qDXbrDNFJpKw79iaw8Xmbrq6ZQfQyA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=20)

生成出来为了补齐9位，才会取一个随机值

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8Mumx4Z7wds541MeMgps8w0XjAtgTaNJbdxgBejiaMOic99PSA7D8PHzBkog/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=21)

tk05w41l貌似是个固定值，看第三段是怎么来，一步一步往上关键字搜索，看看怎么来的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumRD2mxEM2qu3ULqsDY0ib5BsgInHziag0slBPkUGtAoRkyrqic69Zfbk9Q/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=22)

发现是个十六字符串，生成的，其实就可以从这里开始，到生成的地方，丢给ai，就能分析出是个魔改的base64

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumCiaxqP41lubZOwS8SxnMIjKetwsqPibF4QTZoGLMnc7VqibYibfAMdnib1Q/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=23)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumMex5nqtNicXQ74gnpFBFtsJicAC8bdnqlgmypdYZC3yQIicz7ia8qve1OQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=24)

唯一需要注意的就是，中间有个补位算法

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MummB5sNzvZchhKx77CS0sBaWHvbt2ZlD1Ko5JicqmKzXxYj5WK2PTQ4Ww/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=25)

分析这个十六进制怎么来的，大概的流程图，主要还是看md5的明文值是什么

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8Mumd0FJ9jpicKP5QhTobmDOIV7aJathdcAF1jUKrDvnL4jl94OfsAibWZJQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=26)

最后会定位到一个数组

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumnbatufmyIWnKpo0Lut7OBF0nGch1wibx9p2XBTKqNuP3JS6mca5Dq2w/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=27)

数组基本上就是通过这样来的

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumVXBeiclibXXzNSicrafKjDubds61zib8oZ4YoBvKnSHTrXZB0nMHR1FE8w/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=28)

这个md5是魔改的，可以直接扣代码，就能扣下来，或者想具体哪里变了分析的，也可以通过ai来分析

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumLZtfjD0on5LS92CcOx0w7HiasU5phH7gVFH3BQDYq95JLZY0mN2qlEA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=29)

第8段也是个魔改的base64生成的，可以通过ai写个解码base64，明文就是一些环境

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumImzyRhsUiclILiaXlYDQpYpT7uAxaqZMT0ODpZslKia7PbUFJRcxpJWSw/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=30)

后续参数就不说了，按照以上流程很容易分析出来，核心就这几个算法

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/rs4p9qo1ibotaVjdPW3Y9SxuicX6lN8MumHJxKz1xibA2QEtr2wibxurDcfeKTcVSI4uqZ37qFEM6WLVWmFuicJ2laA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=31)