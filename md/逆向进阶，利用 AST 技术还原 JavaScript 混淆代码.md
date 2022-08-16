> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg5NzY2MzA5MQ==&mid=2247488349&idx=1&sn=7a485dc2d9d902042a49e1d3183a9bca&scene=21#wechat_redirect)

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/kicB09lvgibnnRjv0AAqQxyBODIttZXnQqcTPoF4Pt8tJmnia4CHaYUS3zqicFfKZTWibXTAew2ibFHDjy5Pf8nDnVEQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

点击上方「**蓝字**」关注我们

![图片](https://mmbiz.qpic.cn/mmbiz_png/kLQoJJzjYaicxneNzbOg7ynx3TfnIwmNTpJQ7orkaUNrJIV4u7PNdSJ25Mtn6XdRQTamLDDicHnYfdic2bsiaNQjCw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4IgS329iaEoCWR2CHwXvibnFeO704d7E9sKTgia7HpM19aNnFn8mibdsLPkSgE32r16DpricyXI4fTrUvw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)  

目录
--

文章较长，可作为 AST Babel 入门手册，强烈建议收藏！

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJE2H0e1ibc8iawR56x0xCgSvgicznQ9IIia1zWeLFA1wPMAnw2y9ibeLq2DQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

什么是 AST
-------

AST（Abstract Syntax Tree），中文抽象语法树，简称语法树（Syntax Tree），是源代码的抽象语法结构的树状表现形式，树上的每个节点都表示源代码中的一种结构。语法树不是某一种编程语言独有的，JavaScript、Python、Java、Golang 等几乎所有编程语言都有语法树。

小时候我们得到一个玩具，总喜欢把玩具拆解成一个一个小零件，然后按照我们自己的想法，把零件重新组装起来，一个新玩具就诞生了。而 JavaScript 就像一台精妙运作的机器，通过 AST 解析，我们也可以像童年时拆解玩具一样，深入了解 JavaScript 这台机器的各个零部件，然后重新按照我们自己的意愿来组装。

AST 的用途很广，IDE 的语法高亮、代码检查、格式化、压缩、转译等，都需要先将代码转化成 AST 再进行后续的操作，ES5 和 ES6 语法差异，为了向后兼容，在实际应用中需要进行语法的转换，也会用到 AST。**AST 并不是为了逆向而生，但做逆向学会了 AST，在解混淆时可以如鱼得水。**

AST 有一个在线解析网站：https://astexplorer.net/，顶部可以选择语言、编译器、是否开启转化等，如下图所示，区域①是源代码，区域②是对应的 AST 语法树，区域③是转换代码，可以对语法树进行各种操作，区域④是转换后生成的新代码。图中原来的 Unicode 字符经过操作之后就变成了正常字符。

语法树没有单一的格式，选择不同的语言、不同的编译器，得到的结果也是不一样的，在 JavaScript 中，编译器有 Acorn、Espree、Esprima、Recast、Uglify-JS 等，使用最多的是 Babel，后续的学习也是以 Babel 为例。

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJ8exDGpfq0ACQsjY8b6cXL91TS9jCoRQaoAsmUnqymPMfFlpkCpibO9A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

AST 在编译中的位置
-----------

在编译原理中，编译器转换代码通常要经过三个步骤：词法分析（Lexical Analysis）、语法分析（Syntax Analysis）、代码生成（Code Generation），下图生动展示了这一过程：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJQs4SqP1XWS8JMByjBnzGOwibjl6U23lshRVCws2d4KicvXWFmL9CLVsw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 词法分析  

词法分析阶段是编译过程的第一个阶段，这个阶段的任务是从左到右一个字符一个字符地读入源程序，然后根据构词规则识别单词，生成 token 符号流，比如 `isPanda('🐼')`，会被拆分成 `isPanda`，`(`，`'🐼'`，`)` 四部分，每部分都有不同的含义，可以将词法分析过程想象为不同类型标记的列表或数组。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJIByIyrgjZxV0ZJ4BxVffNnkx8WfnMricfU9YWTd1HibR7f6vw4QVSwCA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

### 语法分析  

语法分析是编译过程的一个逻辑阶段，语法分析的任务是在词法分析的基础上将单词序列组合成各类语法短语，比如“程序”，“语句”，“表达式”等，前面的例子中，`isPanda('🐼')` 就会被分析为一条表达语句 `ExpressionStatement`，`isPanda()` 就会被分析成一个函数表达式 `CallExpression`，`🐼` 就会被分析成一个变量 `Literal` 等，众多语法之间的依赖、嵌套关系，就构成了一个树状结构，即 AST 语法树。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJ5ddH3pxPKtS80b0K3nEYoI4UYu32mHA9dcL90XAf0iaepnArtibveYvA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

### 代码生成

代码生成是最后一步，将 AST 语法树转换成可执行代码即可，在转换之前，我们可以直接操作语法树，进行增删改查等操作，例如，我们可以确定变量的声明位置、更改变量的值、删除某些节点等，我们将语句 `isPanda('🐼')` 修改为一个布尔类型的 `Literal`：`true`，语法树就有如下变化：

![图片](https://mmbiz.qpic.cn/mmbiz_gif/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJCiayvE0ibR82cI5iaiaGS7tPEuGcw5yWoOrtVONPyibaH0OMDd84VpeNNaQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

Babel 简介
--------

Babel 是一个 JavaScript 编译器，也可以说是一个解析库，Babel 中文网：https://www.babeljs.cn/，Babel 英文官网：https://babeljs.io/ ，Babel 内置了很多分析 JavaScript 代码的方法，我们可以利用 Babel 将 JavaScript 代码转换成 AST 语法树，然后增删改查等操作之后，再转换成 JavaScript 代码。

Babel 包含的各种功能包、API、各方法可选参数等，都非常多，本文不一一列举，在实际使用过程中，应当多查询官方文档，或者参考文末给出的一些学习资料。Babel 的安装和其他 Node 包一样，需要哪个安装哪个即可，比如 `npm install @babel/core @babel/parser @babel/traverse @babel/generator`

在做逆向解混淆中，主要用到了 Babel 的以下几个功能包，本文也仅介绍以下几个功能包：

1.  `@babel/core`：Babel 编译器本身，提供了 babel 的编译 API；
    
2.  `@babel/parser`：将 JavaScript 代码解析成 AST 语法树；
    
3.  `@babel/traverse`：遍历、修改 AST 语法树的各个节点；
    
4.  `@babel/generator`：将 AST 还原成 JavaScript 代码；
    
5.  `@babel/types`：判断、验证节点的类型、构建新 AST 节点等。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJYRRfV0F0jTrxzyMbC6Be0buxI7YwuhJOemgFkF6qRNq1ib9nia74VyHQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### @babel/core

Babel 编译器本身，被拆分成了三个模块：`@babel/parser`、`@babel/traverse`、`@babel/generator`，比如以下方法的导入效果都是一样的：

```
const parse = require("@babel/parser").parse;  
const parse = require("@babel/core").parse;  
  
const traverse = require("@babel/traverse").default  
const traverse = require("@babel/core").traverse  

```

### @babel/parser

`@babel/parser` 可以将 JavaScript 代码解析成 AST 语法树，其中主要提供了两个方法：

*   `parser.parse(code, [{options}])`：解析一段 JavaScript 代码；
    
*   `parser.parseExpression(code, [{options}])`：考虑到了性能问题，解析单个 JavaScript 表达式。
    

部分可选参数 `options`：

<table data-tool="markdown.com.cn编辑器"><thead><tr style="border-width: 1px 0px 0px;border-right-style: initial;border-bottom-style: initial;border-left-style: initial;border-right-color: initial;border-bottom-color: initial;border-left-color: initial;border-top-style: solid;border-top-color: rgb(204, 204, 204);background-color: white;"><th style="border-top-width: 1px;border-color: rgb(204, 204, 204);text-align: left;background-color: rgb(240, 240, 240);font-size: 14px;" width="137">参数</th><th style="border-top-width: 1px;border-color: rgb(204, 204, 204);text-align: left;background-color: rgb(240, 240, 240);font-size: 14px;" width="398">描述</th></tr></thead><tbody style="margin-top: 5px;margin-bottom: 5px;line-height: 26px;color: rgb(1, 1, 1);font-size: 16px;letter-spacing: 0px;word-break: break-word;font-family: PingFangSC-Light;padding-top: 30px;padding-bottom: 30px;text-align: left;"><tr style="margin-top: 5px;margin-bottom: 5px;line-height: 26px;color: rgb(1, 1, 1);font-size: 16px;letter-spacing: 0px;word-break: break-word;font-family: PingFangSC-Light;padding-top: 30px;padding-bottom: 30px;text-align: left;"><td style="margin-top: 5px;margin-bottom: 5px;line-height: 26px;color: rgb(1, 1, 1);font-size: 16px;letter-spacing: 0px;word-break: break-all;font-family: PingFangSC-Light;padding-top: 30px;padding-bottom: 30px;text-align: left;" width="55" height="49"><code style="font-size: 14px;padding: 2px 4px;border-radius: 4px;margin-right: 2px;margin-left: 2px;background-color: rgba(27, 31, 35, 0.05);font-family: &quot;Operator Mono&quot;, Consolas, Monaco, Menlo, monospace;word-break: break-all;color: rgb(255, 93, 108);">allowImportExportEverywhere</code>‍‍‍‍</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;word-break: break-all;" width="398" height="49">默认 import 和 export 声明语句只能出现在程序的最顶层，设置为 true 则在任何地方都可以声明</td></tr><tr style="letter-spacing: 0px;word-break: break-word;font-family: PingFangSC-Light;text-align: left;font-size: 17px;line-height: 26px;color: rgb(1, 1, 1);"><td style="letter-spacing: 0px;word-break: break-word;font-family: PingFangSC-Light;text-align: left;font-size: 17px;line-height: 26px;color: rgb(1, 1, 1);" width="55">allowReturnOutsideFunction</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="398">默认如果在顶层中使用 return 语句会引起错误，设置为 true 就不会报错</td></tr><tr style="letter-spacing: 0px;word-break: break-word;font-family: PingFangSC-Light;text-align: left;font-size: 17px;line-height: 26px;color: rgb(1, 1, 1);"><td style="letter-spacing: 0px;word-break: break-word;font-family: PingFangSC-Light;text-align: left;font-size: 17px;line-height: 26px;color: rgb(1, 1, 1);" width="55">sourceType</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="398">默认为 script，当代码中含有 import 、export 等关键字时会报错，需要指定为 module</td></tr><tr style="letter-spacing: 0px;word-break: break-word;font-family: PingFangSC-Light;text-align: left;font-size: 17px;line-height: 26px;color: rgb(1, 1, 1);"><td style="letter-spacing: 0px;word-break: break-word;font-family: PingFangSC-Light;text-align: left;font-size: 17px;line-height: 26px;color: rgb(1, 1, 1);" width="55">errorRecovery</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="398">默认如果 babel 发现一些不正常的代码就会抛出错误，设置为 true 则会在保存解析错误的同时继续解析代码，错误的记录将被保存在最终生成的 AST 的 errors 属性中，当然如果遇到严重的错误，依然会终止解析</td></tr></tbody></table>

举个例子看得比较清楚：

```
const parser = require("@babel/parser");  
  
const code = "const a = 1;";  
const ast = parser.parse(code, {sourceType: "module"})  
console.log(ast)  

```

`{sourceType: "module"}` 演示了如何添加可选参数，输出的就是 AST 语法树，这和在线网站 https://astexplorer.net/ 解析出来的语法树是一样的：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJmXBnewh9ibqlibqIUqVqSsrNNrKE57cdyFx6ZaQxA3Ie7MicCPJ8lI3EA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### @babel/generator

`@babel/generator` 可以将 AST 还原成 JavaScript 代码，提供了一个 `generate` 方法：`generate(ast, [{options}], code)`。

部分可选参数 `options`：

<table data-tool="markdown.com.cn编辑器"><thead><tr style="border-width: 1px 0px 0px;border-right-style: initial;border-bottom-style: initial;border-left-style: initial;border-right-color: initial;border-bottom-color: initial;border-left-color: initial;border-top-style: solid;border-top-color: rgb(204, 204, 204);background-color: white;"><th style="border-top-width: 1px;border-color: rgb(204, 204, 204);text-align: left;background-color: rgb(240, 240, 240);font-size: 14px;" width="199">参数</th><th style="border-top-width: 1px;border-color: rgb(204, 204, 204);text-align: left;background-color: rgb(240, 240, 240);font-size: 14px;" width="322">描述</th></tr></thead><tbody style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="45">auxiliaryCommentBefore</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="322">在输出文件内容的头部添加注释块文字</td></tr><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="45">auxiliaryCommentAfter</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="322">在输出文件内容的末尾添加注释块文字</td></tr><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="45">comments</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="322">输出内容是否包含注释</td></tr><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="45">compact</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="322">输出内容是否不添加空格，避免格式化</td></tr><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="45">concise</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="322">输出内容是否减少空格使其更紧凑一些</td></tr><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="45">minified</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="322">是否压缩输出代码</td></tr><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="45">retainLines</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="322">尝试在输出代码中使用与源代码中相同的行号</td></tr></tbody></table>

接着前面的例子，原代码是 `const a = 1;`，现在我们把 `a` 变量修改为 `b`，值 `1` 修改为 `2`，然后将 AST 还原生成新的 JS 代码：

```
const parser = require("@babel/parser");  
const generate = require("@babel/generator").default  
  
const code = "const a = 1;";  
const ast = parser.parse(code, {sourceType: "module"})  
ast.program.body[0].declarations[0].id.name = "b"  
ast.program.body[0].declarations[0].init.value = 2  
const result = generate(ast, {minified: true})  
  
console.log(result.code)  

```

最终输出的是 `const b=2;`，变量名和值都成功更改了，由于加了压缩处理，等号左右两边的空格也没了。

代码里 `{minified: true}` 演示了如何添加可选参数，这里表示压缩输出代码，`generate` 得到的 `result` 得到的是一个对象，其中的 `code` 属性才是最终的 JS 代码。

代码里 `ast.program.body[0].declarations[0].id.name` 是 a 在 AST 中的位置，`ast.program.body[0].declarations[0].init.value` 是 1 在 AST 中的位置，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJvpgiaUvFyZUiaWgeNkgjjf3iaGwCKyEQ0TPbMVwOUq5Oq7MIAxvT9aGvQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### @babel/traverse  

当代码多了，我们不可能像前面那样挨个定位并修改，对于相同类型的节点，我们可以直接遍历所有节点来进行修改，这里就用到了 `@babel/traverse`，它通常和 `visitor` 一起使用，`visitor` 是一个对象，这个名字是可以随意取的，`visitor` 里可以定义一些方法来过滤节点，这里还是用一个例子来演示：

```
const parser = require("@babel/parser");  
const generate = require("@babel/generator").default  
const traverse = require("@babel/traverse").default  
  
const code = `  
const a = 1500;  
const b = 60;  
const c = "hi";  
const d = 787;  
const e = "1244";  
`  
const ast = parser.parse(code)  
  
const visitor = {  
    NumericLiteral(path){  
        path.node.value = (path.node.value + 100) * 2  
    },  
    StringLiteral(path){  
        path.node.value = "I Love JavaScript!"  
    }  
}  
  
traverse(ast, visitor)  
const result = generate(ast)  
console.log(result.code)  

```

这里的原始代码定义了 abcde 五个变量，其值有数字也有字符串，我们在 AST 中可以看到对应的类型为 `NumericLiteral` 和 `StringLiteral`：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJFYopdyCKh8ibIqe2O2ecN1MMqZ9PesKumiaZxr9ButKUIKtMl6ftZ6Ow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后我们声明了一个 `visitor` 对象，然后定义对应类型的处理方法，`traverse` 接收两个参数，第一个是 AST 对象，第二个是 `visitor`，当 `traverse` 遍历所有节点，遇到节点类型为 `NumericLiteral` 和 `StringLiteral` 时，就会调用 `visitor` 中对应的处理方法，`visitor` 中的方法会接收一个当前节点的 `path` 对象，该对象的类型是 `NodePath`，该对象有非常多的属性，以下介绍几种最常用的：

<table data-tool="markdown.com.cn编辑器"><thead><tr style="border-width: 1px 0px 0px;border-right-style: initial;border-bottom-style: initial;border-left-style: initial;border-right-color: initial;border-bottom-color: initial;border-left-color: initial;border-top-style: solid;border-top-color: rgb(204, 204, 204);background-color: white;"><th style="border-top-width: 1px;border-color: rgb(204, 204, 204);text-align: left;background-color: rgb(240, 240, 240);font-size: 14px;" width="215">属性</th><th style="border-top-width: 1px;border-color: rgb(204, 204, 204);text-align: left;background-color: rgb(240, 240, 240);font-size: 14px;" width="225">描述</th></tr></thead><tbody style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="145">toString()</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="225">当前路径的源码</td></tr><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="145">node</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="225">当前路径的节点</td></tr><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="145">parent</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="225">当前路径的父级节点</td></tr><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="145">parentPath</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="225">当前路径的父级路径</td></tr><tr style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;"><td style="color: black;word-break: break-word;font-family: PingFangSC-Light;text-align: left;line-height: 1.75;letter-spacing: 0.05em;font-size: 17px;word-spacing: 0.1em;" width="145">type</td><td style="border-color: rgb(204, 204, 204);font-size: 14px;" width="225">当前路径的类型</td></tr></tbody></table>

PS：`path` 对象除了有很多属性以外，还有很多方法，比如替换节点、删除节点、插入节点、寻找父级节点、获取同级节点、添加注释、判断节点类型等，可在需要时查询相关文档或查看源码，后续介绍 `@babel/types` 部分将会举部分例子来演示，以后的实战文章中也会有相关实例，篇幅有限本文不再细说。

因此在上面的代码中，`path.node.value` 就拿到了变量的值，然后我们就可以进一步对其进行修改了。以上代码运行后，所有数字都会加上100后再乘以2，所有字符串都会被替换成 `I Love JavaScript!`，结果如下：

```
const a = 3200;  
const b = 320;  
const c = "I Love JavaScript!";  
const d = 1774;  
const e = "I Love JavaScript!";  

```

如果多个类型的节点，处理的方式都一样，那么还可以使用 `|` 将所有节点连接成字符串，将同一个方法应用到所有节点：

```
const visitor = {  
    "NumericLiteral|StringLiteral"(path) {  
        path.node.value = "I Love JavaScript!"  
    }  
}  

```

`visitor` 对象有多种写法，以下几种写法的效果都是一样的：

```
const visitor = {  
    NumericLiteral(path){  
        path.node.value = (path.node.value + 100) * 2  
    },  
    StringLiteral(path){  
        path.node.value = "I Love JavaScript!"  
    }  
}  

```

```
const visitor = {  
    NumericLiteral: function (path){  
        path.node.value = (path.node.value + 100) * 2  
    },  
    StringLiteral: function (path){  
        path.node.value = "I Love JavaScript!"  
    }  
}  

```

```
const visitor = {  
    NumericLiteral: {  
        enter(path) {  
            path.node.value = (path.node.value + 100) * 2  
        }  
    },  
    StringLiteral: {  
        enter(path) {  
            path.node.value = "I Love JavaScript!"  
        }  
    }  
}  

```

```
const visitor = {  
    enter(path) {  
        if (path.node.type === "NumericLiteral") {  
            path.node.value = (path.node.value + 100) * 2  
        }  
        if (path.node.type === "StringLiteral") {  
            path.node.value = "I Love JavaScript!"  
        }  
    }  
}  

```

以上几种写法中有用到了 `enter` 方法，在节点的遍历过程中，进入节点（enter）与退出（exit）节点都会访问一次节点，`traverse` 默认在进入节点时进行节点的处理，如果要在退出节点时处理，那么在 `visitor` 中就必须声明 `exit` 方法。

### @babel/types

`@babel/types` 主要用于构建新的 AST 节点，前面的示例代码为 `const a = 1;`，如果想要增加内容，比如变成 `const a = 1; const b = a * 5 + 1;`，就可以通过 `@babel/types` 来实现。

首先观察一下 AST 语法树，原语句只有一个 `VariableDeclaration` 节点，现在增加了一个：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJiaZoABoxgqeubmkRL7ZgwCOPrrLSOliaZfBBjbkDrxT0hBicZfwF6xSKA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

那么我们的思路就是在遍历节点时，遍历到 `VariableDeclaration` 节点，就在其后面增加一个 `VariableDeclaration` 节点，生成  `VariableDeclaration` 节点，可以使用 `types.variableDeclaration()` 方法，在 types 中各种方法名称和我们在 AST 中看到的是一样的，只不过首字母是小写的，所以我们不需要知道所有方法的情况下，也能大致推断其方法名，只知道这个方法还不行，还得知道传入的参数是什么，可以查文档，不过K哥这里推荐直接看源码，非常清晰明了，以 Pycharm 为例，按住 Ctrl 键，再点击方法名，就进到源码里了：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJYjBb41rUXAcWkbHGJ1BD962KQVTzwM98aH3blLGRvCfiaMvySRDpFPQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

```
function variableDeclaration(kind: "var" | "let" | "const", declarations: Array<BabelNodeVariableDeclarator>)  

```

可以看到需要 `kind` 和 `declarations` 两个参数，其中 `declarations` 是 `VariableDeclarator` 类型的节点组成的列表，所以我们可以先写出以下 `visitor` 部分的代码，其中 `path.insertAfter()` 是在该节点之后插入新节点的意思：

```
const visitor = {  
    VariableDeclaration(path) {  
        let declaration = types.variableDeclaration("const", [declarator])  
        path.insertAfter(declaration)  
    }  
}  

```

接下来我们还需要进一步定义 `declarator`，也就是 `VariableDeclarator` 类型的节点，查询其源码如下：

```
function variableDeclarator(id: BabelNodeLVal, init?: BabelNodeExpression)  

```

观察 AST，id 为 `Identifier` 对象，init 为 `BinaryExpression` 对象，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJ1EsoRsc6YjCl0ibbusiaPF5qAzeF8423axjYOjwXwO5Nrv6pN7CP4QqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

先来处理 id，可以使用 `types.identifier()` 方法来生成，其源码为 `function identifier(name: string)`，name 在这里就是 b 了，此时 `visitor` 代码就可以这么写：

```
const visitor = {  
    VariableDeclaration(path) {  
        let declarator = types.variableDeclarator(types.identifier("b"), init)  
        let declaration = types.variableDeclaration("const", [declarator])  
        path.insertAfter(declaration)  
    }  
}  

```

然后再来看 init 该如何定义，首先仍然是看 AST 结构：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJwWXKa34p8iacx3Pb4FmKFhCAWiaaZtSL6ZqTVNlic7AicxFsgib2J0mINrA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

init 为 `BinaryExpression` 对象，left 左边是 `BinaryExpression`，right 右边是 `NumericLiteral`，可以用 `types.binaryExpression()` 方法来生成 init，其源码如下：

```
function binaryExpression(  
    operator: "+" | "-" | "/" | "%" | "*" | "**" | "&" | "|" | ">>" | ">>>" | "<<" | "^" | "==" | "===" | "!=" | "!==" | "in" | "instanceof" | ">" | "<" | ">=" | "<=",  
    left: BabelNodeExpression | BabelNodePrivateName,   
    right: BabelNodeExpression  
)  

```

此时 `visitor` 代码就可以这么写：

```
const visitor = {  
    VariableDeclaration(path) {  
        let init = types.binaryExpression("+", left, right)  
        let declarator = types.variableDeclarator(types.identifier("b"), init)  
        let declaration = types.variableDeclaration("const", [declarator])  
        path.insertAfter(declaration)  
    }  
}  

```

然后继续构造 left 和 right，和前面的方法一样，观察 AST 语法树，查询对应方法应该传入的参数，层层嵌套，直到把所有的节点都构造完毕，最终的 `visitor` 代码应该是这样的：

```
const visitor = {  
    VariableDeclaration(path) {  
        let left = types.binaryExpression("*", types.identifier("a"), types.numericLiteral(5))  
        let right = types.numericLiteral(1)  
        let init = types.binaryExpression("+", left, right)  
        let declarator = types.variableDeclarator(types.identifier("b"), init)  
        let declaration = types.variableDeclaration("const", [declarator])  
        path.insertAfter(declaration)  
        path.stop()  
    }  
}  

```

注意：`path.insertAfter()` 插入节点语句后面加了一句 `path.stop()`，表示插入完成后立即停止遍历当前节点和后续的子节点，添加的新节点也是 `VariableDeclaration`，如果不加停止语句的话，就会无限循环插入下去。

插入新节点后，再转换成 JavaScript 代码，就可以看到多了一行新代码，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJsJHOSnecuN7rsAGsGUqvWZNht1x6WwN7iczkJsT2faPice0OOK45w7Sw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

常见混淆还原
------

了解了 AST 和 babel 后，就可以对 JavaScript 混淆代码进行还原了，以下是部分样例，带你进一步熟悉 babel 的各种操作。

### 字符串还原

文章开头的图中举了个例子，正常字符被换成了 Unicode 编码：

```
console['\u006c\u006f\u0067']('\u0048\u0065\u006c\u006c\u006f\u0020\u0077\u006f\u0072\u006c\u0064\u0021')  

```

观察 AST 结构：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJcr5YRE2TibdHSS7gYgQjek04Ld7HtPJA7ylEjHr2KoKMJ7tH4O1zWOw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们发现 Unicode 编码对应的是 `raw`，而 `rawValue` 和 `value` 都是正常的，所以我们可以将 `raw` 替换成 `rawValue` 或 `value` 即可，需要注意的是引号的问题，本来是 `console["log"]`，你还原后变成了 `console[log]`，自然会报错的，除了替换值以外，这里直接删除 extra 节点，或者删除 raw 值也是可以的，所以以下几种写法都可以还原代码：  

```
const parser = require("@babel/parser");  
const generate = require("@babel/generator").default  
const traverse = require("@babel/traverse").default  
  
const code = `console['\u006c\u006f\u0067']('\u0048\u0065\u006c\u006c\u006f\u0020\u0077\u006f\u0072\u006c\u0064\u0021')`  
const ast = parser.parse(code)  
  
const visitor = {  
    StringLiteral(path) {  
        // 以下方法均可  
        // path.node.extra.raw = path.node.rawValue  
        // path.node.extra.raw = '"' + path.node.value + '"'  
        // delete path.node.extra  
        delete path.node.extra.raw  
    }  
}  
  
traverse(ast, visitor)  
const result = generate(ast)  
console.log(result.code)  

```

还原结果：

```
console["log"]("Hello world!");  

```

### 表达式还原

之前K哥写过 [JSFuck 混淆的还原](https://mp.weixin.qq.com/s?__biz=Mzg5NzY2MzA5MQ==&mid=2247487102&idx=1&sn=3a4f360f3c70fb015cf4100f2606f4be&scene=21#wechat_redirect)，其中有介绍 `![]` 可表示 false，`!![]` 或者 `!+[]` 可表示 true，在一些混淆代码中，经常有这些操作，把简单的表达式复杂化，往往需要执行一下语句，才能得到真正的结果，示例代码如下：

```
const a = !![]+!![]+!![];  
const b = Math.floor(12.34 * 2.12)  
const c = 10 >> 3 << 1  
const d = String(21.3 + 14 * 1.32)  
const e = parseInt("1.893" + "45.9088")  
const f = parseFloat("23.2334" + "21.89112")  
const g = 20 < 18 ? '未成年' : '成年'  

```

想要执行语句，我们需要了解 `path.evaluate()` 方法，该方法会对 path 对象进行执行操作，自动计算出结果，返回一个对象，其中的 `confident` 属性表示置信度，`value` 表示计算结果，使用 `types.valueToNode()` 方法创建节点，使用 `path.replaceInline()` 方法将节点替换成计算结果生成的新节点，替换方法有一下几种：

*   `replaceWith`：用一个节点替换另一个节点；
    
*   `replaceWithMultiple`：用多个节点替换另一个节点；
    
*   `replaceWithSourceString`：将传入的源码字符串解析成对应 Node 后再替换，性能较差，不建议使用；
    
*   `replaceInline`：用一个或多个节点替换另一个节点，相当于同时有了前两个函数的功能。
    

对应的 AST 处理代码如下：

```
const parser = require("@babel/parser");  
const generate = require("@babel/generator").default  
const traverse = require("@babel/traverse").default  
const types = require("@babel/types")  
  
const code = `  
const a = !![]+!![]+!![];  
const b = Math.floor(12.34 * 2.12)  
const c = 10 >> 3 << 1  
const d = String(21.3 + 14 * 1.32)  
const e = parseInt("1.893" + "45.9088")  
const f = parseFloat("23.2334" + "21.89112")  
const g = 20 < 18 ? '未成年' : '成年'  
`  
const ast = parser.parse(code)  
  
const visitor = {  
    "BinaryExpression|CallExpression|ConditionalExpression"(path) {  
        const {confident, value} = path.evaluate()  
        if (confident){  
            path.replaceInline(types.valueToNode(value))  
        }  
    }  
}  
  
traverse(ast, visitor)  
const result = generate(ast)  
console.log(result.code)  

```

最终结果：

```
const a = 3;  
const b = 26;  
const c = 2;  
const d = "39.78";  
const e = parseInt("1.89345.9088");  
const f = parseFloat("23.233421.89112");  
const g = "\u6210\u5E74";  

```

### 删除未使用变量

有时候代码里会有一些并没有使用到的多余变量，删除这些多余变量有助于更加高效的分析代码，示例代码如下：

```
const a = 1;  
const b = a * 2;  
const c = 2;  
const d = b + 1;  
const e = 3;  
console.log(d)  

```

删除多余变量，首先要了解 `NodePath` 中的 `scope`，`scope` 的作用主要是查找标识符的作用域、获取并修改标识符的所有引用等，删除未使用变量主要用到了 `scope.getBinding()` 方法，传入的值是当前节点能够引用到的标识符名称，返回的关键属性有以下几个：

*   `identifier`：标识符的 Node 对象；
    
*   `path`：标识符的 NodePath 对象；
    
*   `constant`：标识符是否为常量；
    
*   `referenced`：标识符是否被引用；
    
*   `references`：标识符被引用的次数；
    
*   `constantViolations`：如果标识符被修改，则会存放所有修改该标识符节点的 Path 对象；
    
*   `referencePaths`：如果标识符被引用，则会存放所有引用该标识符节点的 Path 对象。
    

所以我们可以通过 `constantViolations`、`referenced`、`references`、`referencePaths` 多个参数来判断变量是否可以被删除，AST 处理代码如下：

```
const parser = require("@babel/parser");  
const generate = require("@babel/generator").default  
const traverse = require("@babel/traverse").default  
  
const code = `  
const a = 1;  
const b = a * 2;  
const c = 2;  
const d = b + 1;  
const e = 3;  
console.log(d)  
`  
const ast = parser.parse(code)  
  
const visitor = {  
    VariableDeclarator(path){  
        const binding = path.scope.getBinding(path.node.id.name);  
  
        // 如标识符被修改过，则不能进行删除动作。  
        if (!binding || binding.constantViolations.length > 0) {  
            return;  
        }  
  
        // 未被引用  
        if (!binding.referenced) {  
            path.remove();  
        }  
  
        // 被引用次数为0  
        // if (binding.references === 0) {  
        //     path.remove();  
        // }  
  
        // 长度为0，变量没有被引用过  
        // if (binding.referencePaths.length === 0) {  
        //     path.remove();  
        // }  
    }  
}  
  
traverse(ast, visitor)  
const result = generate(ast)  
console.log(result.code)  

```

处理后的代码（未使用的 b、c、e 变量已被删除）：

```
const a = 1;  
const b = a * 2;  
const d = b + 1;  
console.log(d);  

```

### 删除冗余逻辑代码

有时候为了增加逆向难度，会有很多嵌套的 if-else 语句，大量判断为假的冗余逻辑代码，同样可以利用 AST 将其删除掉，只留下判断为真的，示例代码如下：

```
const example = function () {  
    let a;  
    if (false) {  
        a = 1;  
    } else {  
        if (1) {  
            a = 2;  
        }  
        else {  
            a = 3;  
        }  
    }  
    return a;  
};  

```

观察 AST，判断条件对应的是 `test` 节点，if 对应的是 `consequent` 节点，else 对应的是 `alternate` 节点，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JRHSIwasFXfJga6s0lt3YJXgL4lA0picpU2hSzPe8B8ia6myEvvgmkxvsm081ye1gnqHjPQkCCTDAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

AST 处理思路以及代码：

1.  筛选出 `BooleanLiteral` 和 `NumericLiteral` 节点，取其对应的值，即 `path.node.test.value`；
    
2.  判断 `value` 值为真，则将节点替换成 `consequent` 节点下的内容，即 `path.node.consequent.body`；
    
3.  判断 `value` 值为假，则替换成 `alternate` 节点下的内容，即 `path.node.alternate.body`；
    
4.  有的 if 语句可能没有写 else，也就没有 `alternate`，所以这种情况下判断 `value` 值为假，则直接移除该节点，即 `path.remove()`
    

```
const parser = require("@babel/parser");  
const generate = require("@babel/generator").default  
const traverse = require("@babel/traverse").default  
const types = require('@babel/types');  
  
const code = `  
const example = function () {  
    let a;  
    if (false) {  
        a = 1;  
    } else {  
        if (1) {  
            a = 2;  
        }  
        else {  
            a = 3;  
        }  
    }  
    return a;  
};  
`  
const ast = parser.parse(code)  
  
const visitor = {  
    enter(path) {  
        if (types.isBooleanLiteral(path.node.test) || types.isNumericLiteral(path.node.test)) {  
            if (path.node.test.value) {  
                path.replaceInline(path.node.consequent.body);  
            } else {  
                if (path.node.alternate) {  
                    path.replaceInline(path.node.alternate.body);  
                } else {  
                    path.remove()  
                }  
            }  
        }  
    }  
}  
  
traverse(ast, visitor)  
const result = generate(ast)  
console.log(result.code)  

```

处理结果：

```
const example = function () {  
  let a;  
  a = 2;  
  return a;  
};  

```

### switch-case 反控制流平坦化

控制流平坦化是混淆当中最常见的，通过 `if-else` 或者 `while-switch-case` 语句分解步骤，示例代码：

```
const _0x34e16a = '3,4,0,5,1,2'['split'](',');  
let _0x2eff02 = 0x0;  
while (!![]) {  
    switch (_0x34e16a[_0x2eff02++]) {  
        case'0':  
            let _0x38cb15 = _0x4588f1 + _0x470e97;  
            continue;  
        case'1':  
            let _0x1e0e5e = _0x37b9f3[_0x50cee0(0x2e0, 0x2e8, 0x2e1, 0x2e4)];  
            continue;  
        case'2':  
            let _0x35d732 = [_0x388d4b(-0x134, -0x134, -0x139, -0x138)](_0x38cb15 >> _0x4588f1);  
            continue;  
        case'3':  
            let _0x4588f1 = 0x1;  
            continue;  
        case'4':  
            let _0x470e97 = 0x2;  
            continue;  
        case'5':  
            let _0x37b9f3 = 0x5 || _0x38cb15;  
            continue;  
    }  
    break;  
}  

```

AST 还原思路：

1.  获取控制流原始数组，将 `'3,4,0,5,1,2'['split'](',')` 之类的语句转化成 `['3','4','0','5','1','2']` 之类的数组，得到该数组之后，也可以选择把 split 语句对应的节点删除掉，因为最终代码里这条语句就没用了；
    
2.  遍历第一步得到的控制流数组，依次取出每个值所对应的 case 节点；
    
3.  定义一个数组，储存每个 case 节点 `consequent` 数组里面的内容，并删除 `continue` 语句对应的节点；
    
4.  遍历完成后，将第三步的数组替换掉整个 while 节点，也就是 `WhileStatement`。
    

不同思路，写法多样，对于如何获取控制流数组，可以有以下思路：

1.  获取到 `While` 语句节点，然后使用 `path.getAllPrevSiblings()` 方法获取其前面的所有兄弟节点，遍历每个兄弟节点，找到与 `switch()` 里面数组的变量名相同的节点，然后再取节点的值进行后续处理；
    
2.  直接取 `switch()` 里面数组的变量名，然后使用 `scope.getBinding()` 方法获取到它绑定的节点，然后再取这个节点的值进行后续处理。
    

所以 AST 处理代码就有两种写法，方法一：（code.js 即为前面的示例代码，为了方便操作，这里使用 fs 从文件中读取代码）

```
const parser = require("@babel/parser");  
const generate = require("@babel/generator").default  
const traverse = require("@babel/traverse").default  
const types = require("@babel/types")  
const fs = require("fs");  
  
const code = fs.readFileSync("code.js", {encoding: "utf-8"});  
const ast = parser.parse(code)  
  
const visitor = {  
    WhileStatement(path) {  
        // switch 节点  
        let switchNode = path.node.body.body[0];  
        // switch 语句内的控制流数组名，本例中是 _0x34e16a  
        let arrayName = switchNode.discriminant.object.name;  
        // 获得所有 while 前面的兄弟节点，本例中获取到的是声明两个变量的节点，即 const _0x34e16a 和 let _0x2eff02  
        let prevSiblings = path.getAllPrevSiblings();  
        // 定义缓存控制流数组  
        let array = []  
        // forEach 方法遍历所有节点  
        prevSiblings.forEach(pervNode => {  
            let {id, init} = pervNode.node.declarations[0];  
            // 如果节点 id.name 与 switch 语句内的控制流数组名相同  
            if (arrayName === id.name) {  
                // 获取节点整个表达式的参数、分割方法、分隔符  
                let object = init.callee.object.value;  
                let property = init.callee.property.value;  
                let argument = init.arguments[0].value;  
                // 模拟执行 '3,4,0,5,1,2'['split'](',') 语句  
                array = object[property](argument)  
                // 也可以直接取参数进行分割，方法不通用，比如分隔符换成 | 就不行了  
                // array = init.callee.object.value.split(',');  
            }  
            // 前面的兄弟节点就可以删除了  
            pervNode.remove();  
        });  
  
        // 储存正确顺序的控制流语句  
        let replace = [];  
        // 遍历控制流数组，按正确顺序取 case 内容  
        array.forEach(index => {  
                let consequent = switchNode.cases[index].consequent;  
                // 如果最后一个节点是 continue 语句，则删除 ContinueStatement 节点  
                if (types.isContinueStatement(consequent[consequent.length - 1])) {  
                    consequent.pop();  
                }  
                // concat 方法拼接多个数组，即正确顺序的 case 内容  
                replace = replace.concat(consequent);  
            }  
        );  
        // 替换整个 while 节点，两种方法都可以  
        path.replaceWithMultiple(replace);  
        // path.replaceInline(replace);  
    }  
}  
  
traverse(ast, visitor)  
const result = generate(ast)  
console.log(result.code)  

```

方法二：

```
const parser = require("@babel/parser");  
const generate = require("@babel/generator").default  
const traverse = require("@babel/traverse").default  
const types = require("@babel/types")  
const fs = require("fs");  
  
const code = fs.readFileSync("code.js", {encoding: "utf-8"});  
const ast = parser.parse(code)  
  
const visitor = {  
    WhileStatement(path) {  
        // switch 节点  
        let switchNode = path.node.body.body[0];  
        // switch 语句内的控制流数组名，本例中是 _0x34e16a  
        let arrayName = switchNode.discriminant.object.name;  
        // 获取控制流数组绑定的节点  
        let bindingArray = path.scope.getBinding(arrayName);  
        // 获取节点整个表达式的参数、分割方法、分隔符  
        let init = bindingArray.path.node.init;  
        let object = init.callee.object.value;  
        let property = init.callee.property.value;  
        let argument = init.arguments[0].value;  
        // 模拟执行 '3,4,0,5,1,2'['split'](',') 语句  
        let array = object[property](argument)  
        // 也可以直接取参数进行分割，方法不通用，比如分隔符换成 | 就不行了  
        // let array = init.callee.object.value.split(',');  
  
        // switch 语句内的控制流自增变量名，本例中是 _0x2eff02  
        let autoIncrementName = switchNode.discriminant.property.argument.name;  
        // 获取控制流自增变量名绑定的节点  
        let bindingAutoIncrement = path.scope.getBinding(autoIncrementName);  
        // 可选择的操作：删除控制流数组绑定的节点、自增变量名绑定的节点  
        bindingArray.path.remove();  
        bindingAutoIncrement.path.remove();  
  
        // 储存正确顺序的控制流语句  
        let replace = [];  
        // 遍历控制流数组，按正确顺序取 case 内容  
        array.forEach(index => {  
                let consequent = switchNode.cases[index].consequent;  
                // 如果最后一个节点是 continue 语句，则删除 ContinueStatement 节点  
                if (types.isContinueStatement(consequent[consequent.length - 1])) {  
                    consequent.pop();  
                }  
                // concat 方法拼接多个数组，即正确顺序的 case 内容  
                replace = replace.concat(consequent);  
            }  
        );  
        // 替换整个 while 节点，两种方法都可以  
        path.replaceWithMultiple(replace);  
        // path.replaceInline(replace);  
    }  
}  
  
traverse(ast, visitor)  
const result = generate(ast)  
console.log(result.code)  

```

以上代码运行后，原来的 `switch-case` 控制流就被还原了，变成了按顺序一行一行的代码，更加简洁明了：

```
let _0x4588f1 = 0x1;  
let _0x470e97 = 0x2;  
let _0x38cb15 = _0x4588f1 + _0x470e97;  
let _0x37b9f3 = 0x5 || _0x38cb15;  
let _0x1e0e5e = _0x37b9f3[_0x50cee0(0x2e0, 0x2e8, 0x2e1, 0x2e4)];  
let _0x35d732 = [_0x388d4b(-0x134, -0x134, -0x139, -0x138)](_0x38cb15 >> _0x4588f1);  

```

参考资料
----

本文有参考以下资料，也是比较推荐的在线学习资料：

*   Youtube 视频，Babel 入门：https://www.youtube.com/watch?v=UeVq_U5obnE **（作者 Nicolò Ribaudo，视频中的 PPT 资料可在 K 哥爬虫公众号后台回复 Babel 免费获取！）**
    
*   官方手册 Babel Handbook：https://github.com/jamiebuilds/babel-handbook
    
*   非官方 Babel API 中文文档：https://evilrecluse.top/Babel-traverse-api-doc/
    

END
---

Babel 编译器国内的资料其实不是很多，多看源码、同时在线对照可视化的 AST 语法树，耐心一点儿一层一层分析即可，本文中的案例也只是最基本操作，实际遇到一些混淆还得视情况进行修改，比如需要加一些类型判断来限制等，后续K哥会用实战来带领大家进一步熟悉解混淆当中的其他操作。

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4L2BEhohiciaycd6acftNGOyxLZkbZal8LW2qPk5MXg11hB0qia0s0mdecYM7sLzaezgIzkkiaiahIRY5g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_svg/XYrRG5UShDeGibNoQZgXicJOW4Ss1q8yN1xRqONKKlPnGh7dvAdcvuT8tYuGSeDJibicszI0CZPShu9UtRnEvA9shbglGps4fucP/640?wx_fmt=svg&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_svg/9UjCmequjUickLicqdUmtavXkUejKTHRF28k1CiayichS5TGzHLfAOF0UjWRmTaolibeFRpZQ5XOG0zEvfZZOGeJTVgRYZ3VvDDNZ/640?wx_fmt=svg&wxfrom=5&wx_lazy=1&wx_co=1)

点个在看你最好看

![图片](https://mmbiz.qpic.cn/mmbiz_svg/ylRhrSjQb8jeDpnF88X2eeSg1lzyKxW6EO1zSCZC3wCLAdPNomrSgTBWpHcGxxGNQTXbTC82mySYiaKThz99VBqX7t3uSBcrU/640?wx_fmt=svg&wxfrom=5&wx_lazy=1&wx_co=1)