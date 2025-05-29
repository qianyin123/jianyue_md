> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/H0zUjLragjS40L93gFpypA)

  

  

先写个大致提纲

1.使用浏览器本地替换html加浏览器插件替换js

2.提取akamai变量数字运算的两种方法

3.函数(1个数字参数)还原

4.运算操作优化(函数里的加减乘除位移取反等)

5.最重要的"函数名()(多个数字参数)/函数名().call(null,多个数字参数)"还原

  

  

01

—

  

使用浏览器本地替换html加浏览器插件替换js

  

这里用chrome自带的本地替换，把html里面的Akamai链接固定住

再用插件netify替换js内容让它固定

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8plTTibpASfxGZEMSfPa859554PH8rwQ0biaVhR2YvQpHd6gAtPTcjVyuTw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

02

—  

提取akamai变量数字运算的两种方法

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8plc8hibrQicUMqvB679Vy5OrC7glyjKXoib3hicVOsI7hcRKcNOlBsaXLSQw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

这里写两种方法 

第一种，只是针对变量的结果为数字 把它一开始的运算规则都复制下来，然后在node js里就能拿到变量的结果 只针对提纲3的话(比如br(hS)还原) 可以选择第一种对简单的方法

第二种，把整体代码复制下来一份，然后适当缝补和改点东西 把所有作用域变量都拿到后 在后面写ast代码，这种针对第一种 要复制一万多行代码 vscode和pycham会一卡一卡的 但是这种也把最后一个函数名()(多个数字参数)作用域都提取到了 可以直接还原(有个问题就是直接还原有问题 所以我后面讲的是其他思路) 有兴趣的可以按照这种东西 继续研究另外的解法

  

02.1

——

  

方法1：只提取数字变量 比如hs 以及前面的更多变量xF,kn等等

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8pl7TMNEwwr8DAIlbrC7p3HcQCvcGJg4ysWhHfGb5qstaA7Dvf6ibeBOFA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

把最开头的前三个执行函数直接复制下来 以及这个控制流赋值数字的函数 vl

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8plibXmN5yEibJO1oIPqBdr05rZ1JCxYibcqchW0l8vmeA43iaiaZIDezX3TPQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

在浏览器里 vl函数第一行代码下个断点，刷新网页 让它进来，进来后往上走一个堆栈，看怎么调用的，上级堆栈显示代码是 "vl(sK, []);"

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8plFW7NIFOrPVq9CHry7u7xTozICCCWtVqtzf5esmNr9pEaaEPBVRDiaFg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

直接复制下来 运行一下 可以，已经把所有数字变量拿到了

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8pljrKq83yXNo5nGygInroh2x5o45YG2ib3qocETxibAcgqxbibAcQnlmqqA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

02.2

方法2：复制所有代码，缝补两下，让node.js的变量 作用域等等和浏览器一样，能拿到所有东西

先折叠 观察整体代码，2个自执行函数，直接复制到浏览器的话，没有任何报错，第二个自执行里面有很多变量参数，这是我们需要的东西，因为函数作用域问题，我们拿不到 那就直接把它拆了，变成全局代码，然后函数里的return改成直接调用

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8playpCOmcJRdtcXVzGiaXnekECKTiaMDBiatGE7bVnonvOWJNOlmibnCL4Cw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

变化如下 左边变右边

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

然后直接在node.js运行，会提示document不存在，需要补点东西。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

这里我之前单步调试看了一下，是在获取'document.currentScript.src'可以直接用jsdom给他模拟了

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

jsdom补上之后，在这段全局代码后面加个debug调试，得到的东西已经和网页一样了 至此 准备工作已经做完了 开始还原。

  

03

—

函数(1个数字参数)还原

讲究快准狠，这里就不细写ast去动态提取最外层数字函数定义了，毕竟逆向一个东西 给ast的时间不太多 能快速解下来就是关键 所以直接复制2个这种数字定义的函数名和几个引用函数 再直接写ast还原

先观察结构  GJZ是关键 它里面是常见的数组定义

```
`lW(Ob);``function lW(TDZ) {` `return GJZ()[TDZ];``};`
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

直接我们直接手动复制2个这种函数名 再给它把对应的几个引用函数名(lW)对应上 网页直接 "搜= [' "就能搜到GJZ和另外一个 lkZ 然后"搜GJZ()[" 找到它的引用函数

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

写个对应关系 再写个CallExpression就能解了

```
`// 第1步 函数数字``callback_object = {` `// 搜 = ['` `"lkZ": ["ZF", "R7", "br"],` `"GJZ": ["lW", "wW", "bF","gJ"]``}``let Replacement = {` `"CallExpression"(path) {` `let {node} = path;` `let {callee} = node;` `if (!callee) return;` `let arguments_ = node.arguments;` `if (!callee.type || callee.type !== "Identifier" || !arguments_) return;` `if (callee.name.length > 3) return;` `if (arguments_.length !== 1 || arguments_[0].type !== "Identifier" || arguments_[0].name.length !== 2) return;` `// console.log(callee.name);` `for (c in callback_object) {` `if (callback_object[c].includes(callee.name) === false) {` `continue;` `}` `// debugger;` `let path_content = path.toString();` `decrypt_result = eval(c + '()[' + arguments_[0].name + ']')` `console.log(path_content, "结果：", decrypt_result)` `path.replaceWith(types.valueToNode(decrypt_result));` `}` `}``}``traverse(ast, Replacement);``//`
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

好了开始下一个

  

04

—

  

运算操作优化(函数里的加减乘除位移取反等)

```
和步骤3一样，为了节省时间，找对应关系
```

```
在网页用正则 搜 'return .*? 运算符 .*?'，比如 return .*? !== .*? 搜出来几个，但是只要最外层的函数
```

```
  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8plUVRTGgBNnbGXog1QUxRutntGOlCcVXsmd1nJGRv0pVUgGMt6cXjqCA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

复制好了如下 直接运行

```
`const functionMap = {` `EJ: {type: 'binary', operator: '=='},` `H5: {type: 'binary', operator: '!=='},` `// W5: {type: 'binary', operator: '!'},` `zj: {type: 'binary', operator: '<='},` `// L64: {type: 'binary', operator: '^'},` `Wv: {type: 'binary', operator: '-'},` `gI: {type: 'binary', operator: '/'},` `vr: {type: 'binary', operator: '*'},` `pL: {type: 'binary', operator: '<<'},` `XL: {type: 'binary', operator: '>>'},` `N5: {type: 'binary', operator: '<'},` `U5: {type: 'binary', operator: '&'},` `qp: {` `type: 'special', handler: (args) => types.binaryExpression('in', args[0], args[1])` `}``};``traverse(ast, {` `CallExpression(path) {` `const callee = path.node.callee;` `if (!types.isIdentifier(callee)) {` `return` `}` `const operator = functionMap[callee.name];` `if (!operator) {` `return` `}` `const args = path.node.arguments;` `switch(operator.type) {` `case 'binary':` `if (args.length === 2) {` `path.replaceWith(types.binaryExpression(` `operator.operator,` `args[0],` `args[1]` `));` `}` `break;` `case 'special':` `path.replaceWith(operator.handler(args));` `break;` `}` `}``});`
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8plZmpgfvfGgFKl8MXcdNPVrE9LF1UGObKg0BbpKZWGU5hEtSiaTX4iat7w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

这里已经还原了两处了，复制还原的代码 到浏览器去替换测试一下能不能用 只要能发包 能拿到0的cookie就表示没问题

替换后 结果如下图(左变右) 明显的很多东西替换完成 并且发包没有问题

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8ploj1ngde29QuC4TsMTX1dDt2ffcWXIAiazGY3moRyEHg0Azmwv7Ruibxw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

05

—

最重要的一个字符串还原，把最后一个还原了 就能直接搜到akamai字典的明文了

混淆代码格式：

```
`Or()["JT"](gG, UK)``PD()["bZ"].call(null, JCZ, sb, VJ, tJ, Bp);`
```

先讲一下我一开始的尝试过程和思路

尝试过程 我先用2.2的方法，在node.js已经拿到了所有的东西，以为简单的写个ast 遍历所有CallExpression就可以了，实际操作有问题。 

主要问题 执行这个解密代码 有的会操作一些内部的数组 导致变化后的解密东西不一样，并且很多分支不会运行上，如果去调用了没用的分支会报错 或者执行顺序不一致也有问题吧 所以放弃这种方法，想尝试的可以用我下面的ast定位代码 然后自己修改玩一玩

```
`"CallExpression"(path) {` `const {node} = path;` `// 检测第一种格式: x7()["l8"](bw, N5, EL, rU)` `if (` `types.isMemberExpression(node.callee) &&          // 必须是成员表达式` ``types.isCallExpression(node.callee.object) &&    // `Caller()` 必须是调用表达式`` ``types.isIdentifier(node.callee.object.callee) &&  // `Caller` 必须是标识符`` ``types.isStringLiteral(node.callee.property)       // `["method"]` 必须是字符串字面量`` `) {` `let path_str = path.toString();` `console.log(path_str);` `}` `// 检测第二种格式: Wb()["jW"].call(null, IN, ZJ, xn)` `if (` `types.isMemberExpression(node.callee) &&` ``types.isIdentifier(node.callee.property, {name: 'call'}) && // 必须是 `.call` `` `types.isMemberExpression(node.callee.object) &&` ``types.isCallExpression(node.callee.object.object) &&         // `Caller()` 必须是调用表达式`` ``types.isStringLiteral(node.callee.object.property) &&         // `["method"]` 必须是字符串字面量`` ``types.isNullLiteral(node.arguments[0])                        // 第一个参数必须是 `null` `` `) {` `let path_str = path.toString();` `console.log(path_str, "结果：", "")` `}` `}`
```

开始讲我目前的实际解密思路和方法：

还是用ast定位到这两种格式，然后把所有代码先替换修改，相当于hook了，然后把源代码以及结果都保存下来，再通过ast定位取结果修改，这样的话 就是网页运行后的分支才会被修改，最后结完的代码 没走的分支还是混淆的，在调试的时候也可以直接快速判断~

hook和保存代码：略~ 动动自己的小脑袋瓜想想 或者~

注意 要拿到结果为0的 才是走完全部分支(可以鼠标点一下 生成轨迹拿0 不校验鼠标事件的不用)

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8plrRoe8VsvF3nm7d5J2dlWXWaice6CX77nvlzdOIibRAwfI1mAHhJ7k2LQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

最后粘贴，再小改一下ast代码替换 就完成了

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8pl6Z2uGRgM3bxhibTiadYW1LPL7A8JW5xn7s0ibCdC8o186LNUQ4BPqTDog/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

最后可以看到 走过的分支 就被还原了，一些判断的分支 没经过的就没有还原，剩下的这些小东东 可以没比较纠结了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8plnDibGg7rt7POEvxtjro8kGWNYhnFiaibtadepHJJWAQfnicMzZUojfDv5Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

替换完成拿到了sensor_data数据，断点放开 测试发包 能拿到0 

![图片](https://mmbiz.qpic.cn/mmbiz_png/m9skQicAic14dZztJKXcjgeVoj1EtHV8pl7Lhmh5xvnVLPWLia2u6pJdhHkyNv5WcAXic4uWWKQbBjmtJN16kQJy7A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

好了，over~![图片](https://res.wx.qq.com/t/wx_fed/we-emoji/res/assets/Expression/Expression_1@2x.png?tp=webp&wxfrom=5&wx_lazy=1)