> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/X8eku-JvFL6CEPbPostWPA)

**本文仅供学习交流，因使用本文内容而产生的任何风险及后果，作者不承担任何责任，一起学习吧**

### 写在前面

我其实用这种方法很少，如果有错误，还请大佬们指正，谢谢！！！

都是个人理解，可能不准确，这个方法也是当时和别人聊天聊出来的。

建议先观看前一篇文章：https://www.52pojie.cn/thread-2022463-1-1.html

上一章插桩代码+补环境，我放到最后面，搞忘记了，抱歉！

### 啥是JSVMP?

`JSVMP（JavaScript Virtual Machine Protection）`是一种基于虚拟化技术的`JavaScript`代码保护方案，主要用于防止代码被逆向分析或篡改。其核心思想是将原始`JavaScript`代码转换为**自定义的字节码指令集**，并通过模拟的虚拟机环境来解释执行这些指令，从而增加逆向工程的难度。

是不是很抽象，那就对了，为了后面方便我们理解他的工作原理。首先了解一下JS的运作原理：

```
 _复制代码_ _隐藏代码_`[JS 源码]  
   ↓  
[Parser: 词法 + 语法分析]  --> 抽象语法树 (AST)  
   ↓  
[生成字节码 / 中间表示 (IR)]  
   ↓  
[解释执行]   (启动快，但比较慢)  
   → (检测到热点代码，调用优化编译器)  
      ↓  
   [JIT 编译器: 生成机器码, 优化运行]  
   → [执行机器码]  (运行速度更快)  
   → [可能在某些情况下反优化、回退到解释执行]  
  
并行地，还有一个垃圾回收线程/算法在做内存管理，确保不使用的对象被回收。`
```

举一个例子：  
`console.log("Hello World")`:当JS接收到这段代码时他会怎么做呢？

1.  **词法/语法分析**
    
    ：将 `console.log("Hello World")` 拆分为标识符 `console`、`点号 .`、`标识符 log`、`括号 ()`、`字符串 Hello World` 等 Token，最终组装成一个调用表达式（CallExpression）的 AST。
    
2.  **解释或编译**
    
    ：JS 引擎会将这个调用表达式变成可执行的字节码/机器码，准备执行。
    
3.  **执行**：
    

*   查找 console 对象（在浏览器或 Node.js 环境里由宿主环境提供）
    
*   找到其上的 log 方法
    
*   调用时把 "Hello World" 作为参数传入，交由宿主环境去处理输出（浏览器控制台或 Node.js 控制台输出）。
    
*   期间还会有各种内置检查，比如类型检查、作用域检查等
    

看懂了上面，JSVMP就是这里面动了手脚。如下：

```
 _复制代码_ _隐藏代码_`[原始 JS 源码]  
   ↓  
[Parser: 词法分析 + 语法分析]    
   → 抽象语法树（AST）  
   → 【JSVMP】分析 AST 并生成自定义指令集（VMP IR）  
   ↓  
[加载进 JS 层虚拟机（VMP 解释器）]  
   → 初始化运行时（栈、堆、上下文、指令表等）  
   ↓  
[在 JS 层解释执行指令集]（VMP 模拟执行流程）  
   → 每一条自定义指令被翻译为 JS 操作（变量、运算、跳转等）  
   → 多层嵌套虚拟机 + 花指令 + 加密解码逻辑  
   ↓  
[JS 引擎自身的 IR / 字节码阶段]（开始对“VMP脚本”做优化）  
   ↓  
[解释器执行字节码]  
   → 大量解释器循环、VM调度、字节码解密过程  
   → HotPath 检测  
   ↓  
[JIT 编译器优化 VMP 框架代码]  
   → 优化执行“VMP解释器本身”的性能  
   → 并不会还原出原始业务逻辑  
   ↓  
[机器码执行]（仍在跑 JS 层的 VMP 虚拟机）  
   → 若触发异常/特征变化 → 回退解释执行`
```

为了更进一步的了解他的混淆，我们再举一个例子：

原始js代码：

```
 _复制代码_ _隐藏代码_`functionadd(a, b) {  
  return a + b;  
}  
console.log(add(2, 3)); // 输出 5`
```

首先定义自己的“指令集”：

```
 _复制代码_ _隐藏代码_`// 指令格式： [OPCODE, 操作数1, 操作数2, ...]  
[  
  ["PUSH", 2],  
  ["PUSH", 3],  
  ["CALL", "add"],  
  ["CALL_BUILTIN", "console.log"]  
]`
```

<table style="overflow-wrap: break-word;empty-cells: show;border-collapse: collapse;table-layout: auto;width: 702.859px;margin: 0.25em auto;overflow: auto;border-spacing: 0px;color: rgb(68, 68, 68);font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Helvetica, Arial, sans-serif, &quot;Apple Color Emoji&quot;, &quot;Segoe UI Emoji&quot;;font-size: 14px;font-style: normal;font-variant-ligatures: normal;font-variant-caps: normal;font-weight: 400;letter-spacing: normal;orphans: 2;text-align: start;text-transform: none;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;white-space: normal;background-color: rgb(255, 255, 255);text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;"><thead><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);"><th style="overflow-wrap: break-word;text-align: left;font-weight: 600;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>思路如下：</section></th><th style="overflow-wrap: break-word;text-align: left;font-weight: 600;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>指令名</section></th><th style="overflow-wrap: break-word;text-align: left;font-weight: 600;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>含义</section></th></tr></thead><tbody><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>PUSH</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>压入字面量（数字、字符串、布尔）</section></td><td><section><br></section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);background-color: rgb(246, 248, 250);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>LOAD_VAR</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>加载变量</section></td><td><section><br></section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>STORE_VAR</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>存储变量</section></td><td><section><br></section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);background-color: rgb(246, 248, 250);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>ADD</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>执行加法运算（栈顶2元素）</section></td><td><section><br></section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>CALL</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>调用用户定义函数</section></td><td><section><br></section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);background-color: rgb(246, 248, 250);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>CALL_BUILTIN</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>调用内建函数（如 console.log）</section></td><td><section><br></section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>RET</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>返回函数执行结果</section></td><td><section><br></section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);background-color: rgb(246, 248, 250);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>JMP_IF_FALSE</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>条件跳转</section></td><td><section><br></section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>LABEL</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>跳转标记（给解释器用）</section></td><td><section><br></section></td></tr></tbody></table>

我将指令逐步抽出来：

```
 _复制代码_ _隐藏代码_`[  
  ["LABEL", "add"],  
  ["LOAD_VAR", "a"],  
  ["LOAD_VAR", "b"],  
  ["ADD"],  
  ["RET"],  
  ["PUSH", 2],  
  ["PUSH", 3],  
  ["CALL", "add"],  
  ["CALL_BUILTIN", "console.log"]  
]  
`
```

那么运行框架就出来了：

```
 _复制代码_ _隐藏代码_``functionrunVM(bytecode) {  
    const stack = [];  
    const labels = {};  
    let ip = 0;  
    let callStack = [];  
  
    // 第一次扫描：记录所有 LABEL 的位置  
    for (let i = 0; i < bytecode.length; i++) {  
      const [op, arg] = bytecode[i];  
      if (op === "LABEL") {  
        labels[arg] = i;  
      }  
    }  
  
    // 当前帧：函数的局部变量作用域  
    let currentFrame = {  
      vars: {},  
      returnValue: undefined,  
    };  
  
    while (ip < bytecode.length) {  
      const [op, ...args] = bytecode[ip];  
  
      switch (op) {  
        case"LABEL":  
          ip++;  
          break;  
  
        case"JUMP": {  
          const label = args[0];  
          if (!(label in labels)) {  
            thrownewError(`Label ${label} not found`);  
          }  
          ip = labels[label];  
          break;  
        }  
  
        case"PUSH":  
          stack.push(args[0]);  
          ip++;  
          break;  
  
        case"LOAD_VAR":  
          stack.push(currentFrame.vars[args[0]]);  
          ip++;  
          break;  
  
        case"STORE_VAR":  
          currentFrame.vars[args[0]] = stack.pop();  
          ip++;  
          break;  
  
        case"ADD": {  
          const b = stack.pop();  
          const a = stack.pop();  
          stack.push(a + b);  
          ip++;  
          break;  
        }  
  
        case"CALL": {  
          const funcName = args[0];  
          const funcLabel = labels[funcName];  
          if (funcLabel === undefined) {  
            thrownewError(`Function ${funcName} not found`);  
          }  
  
          const paramNames = ["a", "b"]; // 简化，假设都是两个参数  
          const newFrame = {  
            vars: {},  
            returnValue: undefined,  
          };  
          for (let i = paramNames.length - 1; i >= 0; i--) {  
            newFrame.vars[paramNames[i]] = stack.pop();  
          }  
  
          // 保存当前上下文  
          callStack.push({ ip: ip + 1, frame: currentFrame });  
          currentFrame = newFrame;  
          ip = funcLabel + 1;  
          break;  
        }  
  
        case"RET": {  
          const returnValue = stack.pop();  
          const prev = callStack.pop();  
          currentFrame = prev.frame;  
          ip = prev.ip;  
          stack.push(returnValue);  
          break;  
        }  
  
        case"CALL_BUILTIN": {  
          const fnName = args[0];  
          if (fnName === "console.log") {  
            const val = stack.pop();  
            console.log(val);  
          }  
          ip++;  
          break;  
        }  
  
        default:  
          thrownewError(`Unknown opcode: ${op}`);  
      }  
    }  
  }``
```

运行指令集：

```
 _复制代码_ _隐藏代码_`const bytecode = [  
    ["JUMP", "main"],          // 跳过函数定义  
  
    ["LABEL", "add"],          // 函数体（不会立即执行）  
    ["LOAD_VAR", "a"],  
    ["LOAD_VAR", "b"],  
    ["ADD"],  
    ["RET"],  
  
    ["LABEL", "main"],         // 主程序入口  
    ["PUSH", 2],  
    ["PUSH", 3],  
    ["CALL", "add"],  
    ["CALL_BUILTIN", "console.log"]  
  ];  
  
runVM(bytecode); // 输出：5`
```

让我们来解构执行流程：

<table style="overflow-wrap: break-word;empty-cells: show;border-collapse: collapse;table-layout: auto;width: 702.859px;margin: 0.25em auto;overflow: auto;border-spacing: 0px;color: rgb(68, 68, 68);font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Helvetica, Arial, sans-serif, &quot;Apple Color Emoji&quot;, &quot;Segoe UI Emoji&quot;;font-size: 14px;font-style: normal;font-variant-ligatures: normal;font-variant-caps: normal;font-weight: 400;letter-spacing: normal;orphans: 2;text-align: start;text-transform: none;widows: 2;word-spacing: 0px;-webkit-text-stroke-width: 0px;white-space: normal;background-color: rgb(255, 255, 255);text-decoration-thickness: initial;text-decoration-style: initial;text-decoration-color: initial;"><thead><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);"><th style="overflow-wrap: break-word;text-align: left;font-weight: 600;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>名称</section></th><th style="overflow-wrap: break-word;text-align: left;font-weight: 600;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>说明</section></th></tr></thead><tbody><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>stack</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>栈，用于临时值存储、函数参数、返回值传递等</section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);background-color: rgb(246, 248, 250);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>ip</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>指令指针（instruction pointer），当前执行位置</section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>callStack</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>调用栈，存储函数调用现场（ip 和作用域）</section></td></tr><tr style="overflow-wrap: break-word;border-top: 1px solid rgb(198, 203, 209);background-color: rgb(246, 248, 250);"><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>currentFrame</section></td><td style="overflow-wrap: break-word;font-size: 14px;padding: 6px 13px;border: 1px solid rgb(198, 203, 209);"><section>当前函数的局部变量作用域 { vars: {}, returnValue: ... }</section></td></tr></tbody></table>

1.  初始化：
    

*   stack = []
    
*   labels = { add: 1, main: 6 }
    
*   ip = 0
    
*   currentFrame = { vars: {}, returnValue: undefined }
    
*   callStack = []
    

3.  执行 ["JUMP", "main"]
    

*   查找 labels["main"] = 6
    
*   设置 ip = 6，跳过 add 函数定义区域
    
*   函数定义不会被执行
    

5.  执行 ["PUSH", 2]
    

*   stack.push(2) → stack = [2]
    
*   ip++ → 7
    

7.  执行 ["PUSH", 3]
    

*   stack.push(3) → stack = [2, 3]
    
*   ip++ → 8
    

9.  执行 ["CALL", "add"]
    

*   funcName = "add"，查找 labels["add"] = 1
    
*   创建新作用域帧（参数逆序出栈）：newFrame.vars = { a: 2, b: 3 }
    
*   保存当前上下文：callStack.push({ ip: 9, frame: currentFrame })
    
*   更新 currentFrame = newFrame，ip = 2（跳过 label）
    

11.  执行 ["LOAD_VAR", "a"]
    

*   stack.push(currentFrame.vars["a"]) = 2
    
*   stack = [2]
    
*   ip++ → 3
    

13.  执行 ["LOAD_VAR", "b"]
    

*   stack.push(currentFrame.vars["b"]) = 3
    
*   stack = [2, 3]
    
*   ip++ → 4
    

15.  执行 ["ADD"]
    

*   b = stack.pop() = 3
    
*   a = stack.pop() = 2
    
*   stack.push(2 + 3 = 5) → stack = [5]
    
*   ip++ → 5
    

17.  执行 ["RET"]
    

*   returnValue = stack.pop() = 5
    
*   从调用栈中恢复上一个上下文：
    
    ```
     _复制代码_ _隐藏代码_`const prev = callStack.pop()  
    currentFrame = prev.frame  
    ip = prev.ip = 9`
    ```
    
*   把返回值压回栈：stack = [5]
    

19.  执行 ["CALL_BUILTIN", "console.log"]
    

*   val = stack.pop() = 5
    
*   输出：console.log(5)
    

这还只是一个简单的vmp,能看到里面很多明文字符串，而且指令很简单一点，如果加一点算法混淆（改 opcode 编码）那你还能清晰还原吗？

### 还原思路

*   拆解解释器逻辑 + 恢复字节码语义 + 重建原始源码
    

```
 _复制代码_ _隐藏代码_`[混淆代码 = 虚拟机解释器 + 字节码]  
        ↓  
[静态分析解释器结构]  
        ↓  
[动态跟踪执行流程]  
        ↓  
[逐条分析 opcode 语义]  
        ↓  
[还原伪源码 / 构建语义树]  
        ↓  
[生成近似原始 JS 源码]`
```

应对JSVMP需要将静态分析和动态分析结合起来

高层次流程如下：  
代码预处理与初步识别（静态）：使用代码美化或AST工具将混淆的JS代码格式化，寻找可疑结构。JSVMP代码通常包含大量字节数据和一个庞大的解释器函数。首先要锁定虚拟机入口和调度结构。

定位字节码加载与虚拟机框架（静态+动态）：找到用于生成字节码数组的代码段（例如对传入加密字符串解码）识别虚拟机主要变量（如字节码数组、程序计数器PC、寄存器/栈等）必要时通过调试器设置断点，获取字节码解码后的数据，以便后续分析。

分析虚拟机调度循环和指令集（静态）：深入阅读解释器代码，重点关注主循环（一般是while循环嵌套switch或大量case分支）。梳理每个指令的执行逻辑：每个case分支代表一条虚拟指令及其操作。初步猜测指令功能，并标记尚不清楚的部分。

动态插桩验证与补充（动态）：针对静态分析中不确定之处，修改代码或利用调试工具插入日志（即“插桩”）来观察运行时行为。例如，在每个指令执行时打印相关寄存器/栈变化，以验证指令语义。通过运行脚本，收集执行轨迹，加深对指令作用的理解。

还原原始代码逻辑（重构）：在掌握完整指令集后，将虚拟字节码翻译回高级逻辑。可以手工将每条指令序列转成等价JS代码，或编写辅助脚本按照指令集规则生成伪代码。处理流程控制指令（如跳转、条件）时，要重建出正常的JS结构（if/else/loop）。最终得到可读、可运行且逻辑等价的JavaScript源代码。

~说实话和插装没区别，你都要了解每个指令干了什么~

某q的vmp结构和我举例的几乎是一样，那我们开始简单分析（仅思路分析）我简单改写了一下代码：

```
 _复制代码_ _隐藏代码_`o = function(e, t, a, s, c) {  
    returnfunctionl() {  
        for (var f, p, d = [a, s, t, this, arguments, l, n, 0], h = void0, g = e, v = [],aaaa; ; )  
                for (; ; aaaa=n[++g],console.log(g),console.log('o--> ' + aaaa))  
                    switch (aaaa) {  
                    case0:  
                        d[n[++g]] = new d[n[++g]](d[n[++g]]);  
                        break;  
                    case1:  
                        return d[n[++g]];  
                    case2:  
                        for (f = [],  
                        p = n[++g]; p > 0; p--)  
                            f.push(d[n[++g]]);  
                        d[n[++g]] = u(g + n[++g], f, a, s, c);  
                            Object.defineProperty(d[n[g - 1]], "length", {  
                                value: n[++g],  
                                configurable: !0,  
                                writable: !1,  
                                enumerable: !1  
                            })  
                        break;  
                    case3:  
                        d[n[++g]] = d[n[++g]] < d[n[++g]];  
                        break;  
                    case4:  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]] = d[n[++g]][d[n[++g]]];  
                        break;  
                    case5:  
                        d[n[++g]] = d[n[++g]] >= n[++g];  
                        break;  
                    case6:  
                        d[n[++g]] = d[n[++g]] >> n[++g],  
                        d[n[++g]] = d[n[++g]][d[n[++g]]];  
                        break;  
                    case7:  
                        d[n[++g]] = d[n[++g]] < n[++g];  
                        break;  
                    case8:  
                        d[n[++g]] = d[n[++g]].call(h);  
                        break;  
                    case9:  
                        d[n[++g]] = "",  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]] = n[++g];  
                        break;  
                    case10:  
                        d[n[++g]] = d[n[++g]] | n[++g];  
                        break;  
                    case11:  
                        d[n[++g]] = d[n[++g]] & n[++g],  
                        d[n[++g]] = d[n[++g]][d[n[++g]]];  
                        break;  
                    case12:  
                        d[n[++g]] = {};  
                        break;  
                    case13:  
                        d[n[++g]] = d[n[++g]] | d[n[++g]],  
                        d[n[++g]][d[n[++g]]] = d[n[++g]],  
                        g += d[n[++g]] ? n[++g] : n[(++g,  
                        ++g)];  
                        break;  
                    case14:  
                        d[n[++g]] = h;  
                        break;  
                    case15:  
                        d[n[++g]] = n[++g],  
                        d[n[++g]] = d[n[++g]][n[++g]],  
                        d[n[++g]] = n[++g];  
                        break;  
                    case16:  
                        d[n[++g]] = !0;  
                        break;  
                    case17:  
                        d[n[++g]] = d[n[++g]] === d[n[++g]];  
                        break;  
                    case18:  
                        d[n[++g]] = d[n[++g]] / d[n[++g]];  
                        break;  
                    case19:  
                        d[n[++g]][d[n[++g]]] = d[n[++g]],  
                        d[n[++g]] = "",  
                        d[n[++g]] += String.fromCharCode(n[++g]);  
                        break;  
                    case20:  
                        d[n[++g]][n[++g]] = d[n[++g]],  
                        d[n[++g]][n[++g]] = d[n[++g]],  
                        d[n[++g]][n[++g]] = d[n[++g]];  
                        break;  
                    case21:  
                        d[n[++g]] = d[n[++g]] * d[n[++g]];  
                        break;  
                    case22:  
                        d[n[++g]] = ++d[n[++g]],  
                        d[n[++g]] = d[n[++g]];  
                        break;  
                    case23:  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]] = d[n[++g]][d[n[++g]]],  
                        d[n[++g]] = d[n[++g]];  
                        break;  
                    case24:  
                        d[n[++g]] = d[n[++g]] << n[++g];  
                        break;  
                    case25:  
                        d[n[++g]] = r(d[n[++g]]);  
                        break;  
                    case26:  
                        d[n[++g]] = d[n[++g]] | d[n[++g]];  
                        break;  
                    case27:  
                        d[n[++g]] = n[++g];  
                        break;  
                    case28:  
                        d[n[++g]] = d[n[++g]][n[++g]];  
                        break;  
                    case29:  
                        d[n[++g]] = n[++g],  
                        d[n[++g]][n[++g]] = d[n[++g]],  
                        d[n[++g]] = n[++g];  
                        break;  
                    case30:  
                        d[n[++g]] = d[n[++g]].call(h, d[n[++g]], d[n[++g]]);  
                        break;  
                    case31:  
                        d[n[++g]] = n[++g],  
                        d[n[++g]] = n[++g],  
                        d[n[++g]] = n[++g];  
                        break;  
                    case32:  
                        d[n[++g]] = n[++g],  
                        d[n[++g]][d[n[++g]]] = d[n[++g]];  
                        break;  
                    case33:  
                        d[n[++g]] = d[n[++g]] === n[++g];  
                        break;  
                    case34:  
                        d[n[++g]] = d[n[++g]] + n[++g];  
                        break;  
                    case35:  
                        d[n[++g]] += String.fromCharCode(n[++g]);  
                        break;  
                    case36:  
                        d[n[++g]] = "",  
                        d[n[++g]] += String.fromCharCode(n[++g]);  
                        break;  
                    case37:  
                        d[n[++g]] = d[n[++g]][n[++g]],  
                        d[n[++g]] = d[n[++g]][n[++g]],  
                        d[n[++g]] = d[n[++g]][n[++g]];  
                        break;  
                    case38:  
                        d[n[++g]] = "",  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]] += String.fromCharCode(n[++g]);  
                        break;  
                    case39:  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]] = d[n[++g]] === d[n[++g]],  
                        g += d[n[++g]] ? n[++g] : n[(++g,  
                        ++g)];  
                        break;  
                    case40:  
                        d[n[++g]] = d[n[++g]] > d[n[++g]];  
                        break;  
                    case41:  
                        d[n[++g]] = d[n[++g]] - d[n[++g]];  
                        break;  
                    case42:  
                        d[n[++g]] = d[n[++g]] << d[n[++g]];  
                        break;  
                    case43:  
                        d[n[++g]] = d[n[++g]] & d[n[++g]];  
                        break;  
                    case44:  
                        d[n[++g]] = d[n[++g]] & n[++g];  
                        break;  
                    case45:  
                        d[n[++g]] = -d[n[++g]];  
                        break;  
                    case46:  
                        for (f = [],  
                        p = n[++g]; p > 0; p--)  
                            f.push(d[n[++g]]);  
                        d[n[++g]] = o(g + n[++g], f, a, s, c);  
                            Object.defineProperty(d[n[g - 1]], "length", {  
                                value: n[++g],  
                                configurable: !0,  
                                writable: !1,  
                                enumerable: !1  
                            })  
                        break;  
                    case47:  
                        g += d[n[++g]] ? n[++g] : n[(++g,  
                        ++g)];  
                        break;  
                    case48:  
                        d[n[++g]][n[++g]] = d[n[++g]];  
                        break;  
                    case49:  
                        d[n[++g]] = ~d[n[++g]];  
                        break;  
                    case50:  
                        d[n[++g]] = d[n[++g]].call(d[n[++g]]);  
                        break;  
                    case51:  
                        d[n[++g]] = d[n[++g]] ^ d[n[++g]];  
                        break;  
                    case52:  
                        d[n[++g]] = ++d[n[++g]];  
                        break;  
                    case53:  
                        d[n[++g]] = !1;  
                        break;  
                    case54:  
                        d[n[++g]] = d[n[++g]] >>> n[++g];  
                        break;  
                    case55:  
                        d[n[++g]][n[++g]] = d[n[++g]],  
                        d[n[++g]] = n[++g],  
                        d[n[++g]][n[++g]] = d[n[++g]];  
                        break;  
                    case56:  
                        d[n[++g]] = Array(n[++g]);  
                        break;  
                    case57:  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]][n[++g]] = d[n[++g]];  
                        break;  
                    case58:  
                        d[n[++g]] = d[n[++g]] % d[n[++g]];  
                        break;  
                    case59:  
                        d[n[++g]] = d[n[++g]][d[n[++g]]],  
                        d[n[++g]] = d[n[++g]][n[++g]];  
                        break;  
                    case60:  
                        d[n[++g]] = d[n[++g]][n[++g]],  
                        d[n[++g]] = n[++g];  
                        break;  
                    case61:  
                        d[n[++g]] = d[n[++g]] - n[++g];  
                        break;  
                    case62:  
                        d[n[++g]] = d[n[++g]] + d[n[++g]];  
                        break;  
                    case63:  
                        d[n[++g]] = !d[n[++g]];  
                        break;  
                    case64:  
                        d[n[++g]][d[n[++g]]] = d[n[++g]];  
                        break;  
                    case65:  
                        for (d[n[++g]] += String.fromCharCode(n[++g]),  
                        f = [],  
                        p = n[++g]; p > 0; p--)  
                            f.push(d[n[++g]]);  
                        d[n[++g]] = o(g + n[++g], f, a, s, c);  
                            Object.defineProperty(d[n[g - 1]], "length", {  
                                value: n[++g],  
                                configurable: !0,  
                                writable: !1,  
                                enumerable: !1  
                            })  
                        d[n[++g]][d[n[++g]]] = d[n[++g]];  
                        break;  
                    case66:  
                        d[n[++g]] = d[n[++g]] - 0;  
                        break;  
                    case67:  
                        d[n[++g]] = d[n[++g]].call(d[n[++g]], d[n[++g]]);  
                        break;  
                    case68:  
                        d[n[++g]] = d[n[++g]][n[++g]],  
                        d[n[++g]] = d[n[++g]],  
                        d[n[++g]] = d[n[++g]] - 0;  
                        break;  
                    case69:  
                        d[n[++g]] = d[n[++g]][d[n[++g]]],  
                        d[n[++g]] = d[n[++g]] + d[n[++g]];  
                        break;  
                    case70:  
                        d[n[++g]] = n[++g] + d[n[++g]];  
                        break;  
                    case71:  
                        d[n[++g]] = d[n[++g]] << d[n[++g]],  
                        d[n[++g]] = d[n[++g]] | d[n[++g]],  
                        d[n[++g]][d[n[++g]]] = d[n[++g]];  
                        break;  
                    case72:  
                        d[n[++g]] = d[n[++g]].call(d[n[++g]], d[n[++g]], d[n[++g]]);  
                        break;  
                    case73:  
                        d[n[++g]] = d[n[++g]] >> n[++g];  
                        break;  
                    case74:  
                        d[n[++g]][d[n[++g]]] = d[n[++g]],  
                        d[n[++g]][d[n[++g]]] = d[n[++g]],  
                        d[n[++g]][d[n[++g]]] = d[n[++g]];  
                        break;  
                    case75:  
                        d[n[++g]] = n[++g],  
                        d[n[++g]][n[++g]] = d[n[++g]],  
                        g += d[n[++g]] ? n[++g] : n[(++g,  
                        ++g)];  
                        break;  
                    case76:  
                        d[n[++g]] = d[n[++g]].call(h, d[n[++g]]);  
                        break;  
                    case77:  
                        d[n[++g]] = d[n[++g]];  
                        break;  
                    case78:  
                        d[n[++g]] = d[n[++g]][d[n[++g]]];  
                        break;  
                    case79:  
                        d[n[++g]] = d[n[++g]][n[++g]],  
                        d[n[++g]] = d[n[++g]] >> n[++g],  
                        d[n[++g]] = d[n[++g]] & n[++g];  
                        break;  
                    case80:  
                        d[n[++g]] = "";  
                        break;  
                    case81:  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]] += String.fromCharCode(n[++g]);  
                        break;  
                    case82:  
                        d[n[++g]] += String.fromCharCode(n[++g]),  
                        d[n[++g]] = d[n[++g]][d[n[++g]]],  
                        g += d[n[++g]] ? n[++g] : n[(++g,  
                        ++g)]  
                    }  
    }  
}`
```

还原的关键在于弄清楚每一个case做了什么，以及它调度的顺序。

我们详细观看这个jsvmp,很清晰，数组n就相当于传入的程序字节码,通过控制指针g来实现具体操作，case作为每一个虚拟指令。

那么我们是不是有一种想法，如果我知道case执行的顺序，并且把每次case指令操作前后的指针打出来，那我不就知道了每一步case里面到底进行了什么操作吗(**如果每个case中指针总是仅进行++操作**)？

事实证明，这种方法是可行的。根据上面的修改代码，我们保存数组n和指针g。根据如下代码：

```
 _复制代码_ _隐藏代码_`# g 是数组n  
vmp_code = vmp.split('\n')  
withopen("output.txt", "w", encoding="utf-8") as file:  
    for i inrange(0, len(vmp_code), 2):  
            # 检查索引是否越界  
            if i + 2 < len(vmp_code):  
                begin = int(vmp_code[i])  # 开始索引  
                end = int(vmp_code[i + 2])  # 结束索引  
                code = vmp_code[i + 1]  # 指令集代码  
  
                substring = g[begin:end]  
                # print(f'{code} -- 索引入参{begin} -- 索引出参{end} -- 指令集  {substring}')  
                # # 写入文件  
                file.write(f'{code} -- 索引入参{begin} -- 索引出参{end} 执行差值{end-begin} -- 指令集  {substring}\n')`
```

可以得到一个**错误**的指令集:

```
 _复制代码_ _隐藏代码_`指令集第一位均为case  
o--> 56 -- 索引入参4157 -- 索引出参4160 执行差值3 -- 指令集  [56, 24, 0]  
o--> 27 -- 索引入参4160 -- 索引出参4163 执行差值3 -- 指令集  [27, 17, 25]  
o--> 56 -- 索引入参4163 -- 索引出参4166 执行差值3 -- 指令集  [56, 43, 0]  
o--> 56 -- 索引入参4166 -- 索引出参4169 执行差值3 -- 指令集  [56, 33, 0]  
o--> 56 -- 索引入参4169 -- 索引出参4172 执行差值3 -- 指令集  [56, 34, 0]  
o--> 56 -- 索引入参4172 -- 索引出参4175 执行差值3 -- 指令集  [56, 25, 0]  
o--> 56 -- 索引入参4175 -- 索引出参4178 执行差值3 -- 指令集  [56, 20, 0]  
o--> 56 -- 索引入参4178 -- 索引出参4181 执行差值3 -- 指令集  [56, 27, 0]  
o--> 56 -- 索引入参4181 -- 索引出参4184 执行差值3 -- 指令集  [56, 26, 0]  
o--> 46 -- 索引入参4184 -- 索引出参4190 执行差值6 -- 指令集  [46, 1, 26, 30, 2589, 1]  
o--> 48 -- 索引入参4190 -- 索引出参4194 执行差值4 -- 指令集  [48, 24, 0, 30]  
o--> 46 -- 索引入参4194 -- 索引出参4200 执行差值6 -- 指令集  [46, 1, 24, 30, 1572, 1]  
o--> 48 -- 索引入参4200 -- 索引出参4204 执行差值4 -- 指令集  [48, 43, 0, 30]  
o--> 48 -- 索引入参4204 -- 索引出参4208 执行差值4 -- 指令集  [48, 6, 4240, 17]  
o--> 46 -- 索引入参4208 -- 索引出参4213 执行差值5 -- 指令集  [46, 0, 17, -802, 0]`
```

为什么说是错误的呢？因为分支里面有坏人，`13，39，47，75，82`分支对指针g进行了大额加算，这导致我们从指令集取指令出现异常。这部分可以通过前后差值计算得出正确的指令。而且也不是还原的关键。而且对应++位置的指令仍然是正确的。例如：

```
 _复制代码_ _隐藏代码_`o--> 1 -- 索引入参261 -- 索引出参4818 执行差值4557 -- 指令集  [1, 18....]  
return d[n[++g]]; --> g 移动一位，指令集18 仍然有效。这个理解很重要！！！  
因为每一个case指针位移是确定的！所以指令集后面的数组是可以直接使用的`
```

#### 关键还原

py输出的指令一共有六千多行，但是我们关注的重点其实很少，很清晰。重点关注，指令集很长的case操作，尤其是`case 1 --> return`。通过这个指令你可以直接在对应位置打上断点输出结果。通过对`o--> 1 -- 索引入参`的观察在`g=3319`的时候,入参`12345`的sha1加密就完成了。如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0ogcYOYg86YN0lqPfnibkkahEAWVK6JJT5C0ibZA1xNrKf7KnZ13rXuFwg7tnONCLjFamqdBYUMZHxw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

  

向上看，你就可以发现一大把循环指令`case69,case79循环`,还是一样对异常长度的指令集进行断点调试（这个就把对应的数字填进去很简单的）确定出入口，把这一个大的output文件分块，基本上就能还原这个代码了。~工作量很大说实话~你还原下了，插装也基本解出来了。对于包含`String.fromCharCode`这种代码的，你通过指令集都可以直接写出结果。

后面你继续断查看你会发现 `A873DD4 DC4C4D03` 等串的生成，方法都是一样的,还原就是将指令替换`n[++g]`即可，剩下的都是重复性代码。

#### 谈谈心得体会

我主要是不会写ast(这个改完之后很清晰，直接可以写出伪代码)，不然这个方法确实是一个好方法。个人更加偏向于使用插桩。通过这种方法可以辅助插桩。因为插桩最重要的是不知道插在哪里？但是这种方法，你很明确的就知道你要分析哪里。各有各的好处。文章省略的地方确实有很多，很抱歉，实在是不知道怎么表达这一部分。下面是我当时还原的一部分代码：

```
 _复制代码_ _隐藏代码_`d = []  
  
d[24] = Array(0);  // 24, 0 --> 创建了一个空列表  
  
d[17] = 25; // 17, 25 -->   
  
d[n[++g]] = Array(n[++g]);  // 43,0  
  
d[n[++g]] = Array(n[++g]);  // 33,0  
  
d[n[++g]] = Array(n[++g]);  // 34,0  
  
d[n[++g]] = Array(n[++g]);  // 25,0  
  
d[n[++g]] = Array(n[++g]);  // 20,0  
  
d[n[++g]] = Array(n[++g]);  // 27,0  
  
d[n[++g]] = Array(n[++g]);  // 26,0  
  
// 1, 26, 30, 2589, 1  
// for (f = [],p = n[++g]; p > 0; p--)  
//     f.push(d[n[++g]]);  
//             d[n[++g]] = o(g + n[++g], f, a, s, c);  
//             Object.defineProperty(d[n[g - 1]], "length", {  
//                 value: n[++g],  
//                 configurable: !0,  
//                 writable: !1,  
//                 enumerable: !1  
//             })  改写为下面这个  
  
f = []  
f.push(d[26])  
a = globalThis  
s = [undefined,1732584193,4023233417,2562383102,3285377520,false,true,2147483648,4294967295,4294967296,1518500249,1859775393,1894007588,]  
    //[undefined,1732584193,4023233417,2562383102,3285377520,false,true,2147483648,4294967295,4294967296,1518500249,1859775393,1894007588,]  
c = undefined  
d[30] = o(4187 + 2589, f, a, s, c);  // 把原本函数赋值给了d[30]  
  
d[24][0] = d[30];  // 24, 0, 30   
  
// for (f = [],  
//             p = n[++g]; p > 0; p--)  
//                 f.push(d[n[++g]]);  
//             d[n[++g]] = o(g + n[++g], f, a, s, c);  
//                 Object.defineProperty(d[n[g - 1]], "length", {  
//                     value: n[++g],  
//                     configurable: !0,  
//                     writable: !1,  
//                     enumerable: !1  
//                 })  // 1, 24, 30, 1572, 1  
  
f = []  
f.push(d[24])  
d[30] = o(4197 + 1572, f, a, s, c)  // 同上  
  
d[43][0] = d[30];  // 43, 0, 30  
  
d[6][4240] = d[17];  // 6, 4240, 17  
  
// for (f = [],  
//             p = n[++g]; p > 0; p--)  
//                 f.push(d[n[++g]]);  
//             d[n[++g]] = o(g + n[++g], f, a, s, c);  
//                 Object.defineProperty(d[n[g - 1]], "length", {  
//                     value: n[++g],  
//                     configurable: !0,  
//                     writable: !1,  
//                     enumerable: !1  
//                 })  // 0, 17, -802, 0  
f = []  
d[17] = o(4210 + -802, f, a, s, c)  // 同上  
  
d[30] = d[17].call(undefined);  // 30, 17  
// 这里分析开始跳转了   
// d[17] --> 调用自己 回去了，你可以跟一下，你会发现g = 4210 + -802 = 3408  这里指令顺序就变了  
  
d[10] = "",  
d[10] += String.fromCharCode(103),  
d[10] += String.fromCharCode(108);  // 10, 10, 103, 10, 108  
  
d[10] += String.fromCharCode(111),  
d[10] += String.fromCharCode(98),  
d[10] += String.fromCharCode(97);  // 10, 111, 10, 98, 10, 97  
  
d[10] += String.fromCharCode(108);  // 10, 108  上面几步都是为了d[10] = 'global'  
  
d[10] = d[0][d[10]];  // 10, 0, 10  
  
d[8] = undefined//r(d[10]);  // 8, 10  --> 这里校验了环境 查看了window 里面是否有global，这个也是补环境很常见的监测点  
  
d[10] = 63;  // 10, 63  
`
```

耐心一点，这个方法是完全可行的。

插桩代码

```
`window = globalThis``window.global = undefined``window.navigator = {}``window.location = {` `constructor : '',` `host:'y.qq.com',``}``var webpack_output;``function getEnvs(proxyObjs) {` `for (let i = 0; i < proxyObjs.length; i++) {` ``const handler = `{`` `get: function(target, property, receiver) {` `console.log("方法:", "get  ", "对象:", "${proxyObjs[i]}", "  属性:", property, "  属性类型：", typeof property, ", 属性值：", target[property], ", 属性值类型：", typeof target[property]);` `return target[property];` `},` `set: function(target, property, value, receiver) {` `console.log("方法:", "set  ", "对象:", "${proxyObjs[i]}", "  属性:", property, "  属性类型：", typeof property, ", 属性值：", value, ", 属性值类型：", typeof target[property]);` `return Reflect.set(...arguments);` `}` ``}`;`` ``eval(`try {`` `${proxyObjs[i]};` `${proxyObjs[i]} = new Proxy(${proxyObjs[i]}, ${handler});` `} catch (e) {` `${proxyObjs[i]} = {};` `${proxyObjs[i]} = new Proxy(${proxyObjs[i]}, ${handler});` ``}`);`` `}``}``proxyObjs = ['window', 'document', 'location', 'navigator', 'history', 'screen','host']``// getEnvs(proxyObjs);``!function(e) {` `function t(t) {` `for (var a, n, d = t[0], c = t[1], i = t[2], l = 0, b = []; l < d.length; l++)` `n = d[l],` `Object.prototype.hasOwnProperty.call(o, n) && o[n] && b.push(o[n][0]),` `o[n] = 0;` `for (a in c)` `Object.prototype.hasOwnProperty.call(c, a) && (e[a] = c[a]);` `for (u && u(t); b.length; )` `b.shift()();` `return f.push.apply(f, i || []),` `r()` `}` `function r() {` `for (var e, t = 0; t < f.length; t++) {` `for (var r = f[t], a = !0, n = 1; n < r.length; n++) {` `var c = r[n];` `0 !== o[c] && (a = !1)` `}` `a && (f.splice(t--, 1),` `e = d(d.s = r[0]))` `}` `return e` `}` `var a = {}` `, n = {` `21: 0` `}` `, o = {` `21: 0` `}` `, f = [];` `function d(t) {` `console.log('调用模块---> ',t)  // 如果你理解了上面的代码，那么我们就可以实现webpack自吐模块了` `var a = {};     // 初始化模块缓存对象` `// 如果模块 a[t] 已经被加载过了（缓存存在），直接返回它的 exports（模块导出内容）` `if (a[t])` `return a[t].exports;` `// 否则开始加载模块：创建模块对象 r` `var r = a[t] = {` `i: t,         // 模块 id` `l: !1,        // 模块是否已经加载，false 表示还没加载` `exports: {}   // 用来存放模块导出的内容` `};` `// 执行模块函数 e[t]，绑定 this 为 r.exports --> 很明显模块是缺失的需要我们自己找` `// 传入参数：模块对象 r、模块的 exports 和 require 函数 d（自己）` `e[t].call(r.exports, r, r.exports, d);` `// 标记模块为已加载` `r.l = !0;` `// 返回模块的 exports` `return r.exports;` `}` `webpack_output = d //全局暴露` `d.e = function(e) {` `var t = [];` `n[e] ? t.push(n[e]) : 0 !== n[e] && {` `你那么聪明肯定写得出来` `}[e] && t.push(n[e] = new Promise((function(t, r) {` `for (var a = "css/" + ({` `你那么聪明肯定写得出来` `}[e] || e) + "." + {` `你那么聪明肯定写得出来` `}[e] + ".chunk.css?max_age=2592000", o = d.p + a, f = document.getElementsByTagName("link"), c = 0; c < f.length; c++) {` `var i = (u = f[c]).getAttribute("data-href") || u.getAttribute("href");` `if ("stylesheet" === u.rel && (i === a || i === o))` `return t()` `}` `var l = document.getElementsByTagName("style");` `for (c = 0; c < l.length; c++) {` `var u;` `if ((i = (u = l[c]).getAttribute("data-href")) === a || i === o)` `return t()` `}` `var b = document.createElement("link");` `b.rel = "stylesheet",` `b.type = "text/css",` `b.onload = t,` `b.onerror = function(t) {` `var a = t && t.target && t.target.src || o` `, f = new Error("Loading CSS chunk " + e + " failed.\n(" + a + ")");` `f.code = "CSS_CHUNK_LOAD_FAILED",` `f.request = a,` `delete n[e],` `b.parentNode.removeChild(b),` `r(f)` `}` `,` `b.href = o,` `0 !== b.href.indexOf(window.location.origin + "/") && (b.crossOrigin = "anonymous"),` `document.getElementsByTagName("head")[0].appendChild(b)` `}` `)).then((function() {` `n[e] = 0` `}` `)));` `var r = o[e];` `if (0 !== r)` `if (r)` `t.push(r[2]);` `else {` `var a = new Promise((function(t, a) {` `r = o[e] = [t, a]` `}` `));` `t.push(r[2] = a);` `var f, c = document.createElement("script");` `c.charset = "utf-8",` `c.timeout = 120,` `d.nc && c.setAttribute("nonce", d.nc),` `c.src = function(e) {` `return d.p + "js/" + ({` `你那么聪明肯定写得出来` `}[e] || e) + ".chunk." + {` `你那么聪明肯定写得出来` `}[e] + ".js?max_age=2592000"` `}(e),` `0 !== c.src.indexOf(window.location.origin + "/") && (c.crossOrigin = "anonymous");` `var i = new Error;` `f = function(t) {` `c.onerror = c.onload = null,` `clearTimeout(l);` `var r = o[e];` `if (0 !== r) {` `if (r) {` `var a = t && ("load" === t.type ? "missing" : t.type)` `, n = t && t.target && t.target.src;` `i.message = "Loading chunk " + e + " failed.\n(" + a + ": " + n + ")",` `i.name = "ChunkLoadError",` `i.type = a,` `i.request = n,` `r[1](i)` `}` `o[e] = void 0` `}` `}` `;` `var l = setTimeout((function() {` `f({` `type: "timeout",` `target: c` `})` `}` `), 12e4);` `c.onerror = c.onload = f,` `document.head.appendChild(c)` `}` `return Promise.all(t)` `}` `,` `d.m = e,` `d.c = a,` `d.d = function(e, t, r) {` `d.o(e, t) || Object.defineProperty(e, t, {` `enumerable: !0,` `get: r` `})` `}` `,` `d.r = function(e) {` `"undefined" !== typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {` `value: "Module"` `}),` `Object.defineProperty(e, "__esModule", {` `value: !0` `})` `}` `,` `d.t = function(e, t) {` `if (1 & t && (e = d(e)),` `8 & t)` `return e;` `if (4 & t && "object" === typeof e && e && e.__esModule)` `return e;` `var r = Object.create(null);` `if (d.r(r),` `Object.defineProperty(r, "default", {` `enumerable: !0,` `value: e` `}),` `2 & t && "string" != typeof e)` `for (var a in e)` `d.d(r, a, function(t) {` `return e[t]` `}` `.bind(null, a));` `return r` `}` `,` `d.n = function(e) {` `var t = e && e.__esModule ? function() {` `return e.default` `}` `: function() {` `return e` `}` `;` `return d.d(t, "a", t),` `t` `}` `,` `d.o = function(e, t) {` `return Object.prototype.hasOwnProperty.call(e, t)` `}` `,` `d.p = "你那么聪明肯定写得出来",` `d.oe = function(e) {` `throw e` `}` `;` `var c = window.webpackJsonp = window.webpackJsonp || []` `, i = c.push.bind(c);` `c.push = t,` `c = c.slice();` `for (var l = 0; l < c.length; l++)` `t(c[l]);` `var u = i;` `r()``}({` `'350':function(e, t, n) {` `"use strict";` `n.r(t),` `function(e) {` `var n = "undefined" !== typeof e ? e : "undefined" !== typeof window ? window : "undefined" !== typeof self ? self : void 0` `, r = function(e) {` `return e && "undefined" != typeof Symbol && e.constructor === Symbol ? "symbol" : typeof e` `};` `(function() {` `var e = function(e, t, n) {` `for (var r = [], i = 0; i++ < t; )` `r.push(e += n);` `return r` `}` `, t = function(e) {` `for (var t, n, r = String(e).replace(/[=]+$/, ""), i = r.length, a = 0, u = 0, s = []; u < i; u++)` `~(n = o[r.charCodeAt(u)]) && (t = a % 4 ? 64 * t + n : n,` `a++ % 4) && s.push(255 & t >> (-2 * a & 6));` `return s` `}` `, n = function(e) {` `return e >> 1 ^ -(1 & e)` `}` `, i = []` `, o = e(0, 43, 0).concat([62, 0, 62, 0, 63]).concat(e(51, 10, 1)).concat(e(0, 8, 0)).concat(e(0, 25, 1)).concat([0, 0, 0, 0, 63, 0]).concat(e(25, 26, 1))` `, a = function(e) {` `for (var r = [], i = new Int8Array(t(e)), o = i.length, a = 0; o > a; ) {` `var u = i[a++]` `, s = 127 & u;` `u >= 0 ? r.push(n(s)) : (s |= (127 & (u = i[a++])) << 7,` `u >= 0 || (s |= (127 & (u = i[a++])) << 14,` `u >= 0 || (s |= (127 & (u = i[a++])) << 21,` `u >= 0 || (s |= (u = i[a++]) << 28))),` `r.push(n(s)))` `}` `return r` `};` `return function(e, t) {` `var n = a(e)` `, o = function(e, t, a, s, c) {` `return function l() {` `for (var f, p, d = [a, s, t, this, arguments, l, n, 0], h = void 0, g = e, v = [],aaaa; ; )` `//try {` `for (; ; console.log(g),aaaa=n[++g],console.log('执行到--》 ' + aaaa))` `switch (aaaa) {` `case 0:` `d[n[++g]] = new d[n[++g]](d[n[++g]]);` `break;` `case 1:` `return d[n[++g]];` `case 2:` `for (f = [],` `p = n[++g]; p > 0; p--)` `f.push(d[n[++g]]);` `d[n[++g]] = u(g + n[++g], f, a, s, c);` `//try {` `Object.defineProperty(d[n[g - 1]], "length", {` `value: n[++g],` `configurable: !0,` `writable: !1,` `enumerable: !1` `})` `//} catch (m) {}` `break;` `case 3:` `d[n[++g]] = d[n[++g]] < d[n[++g]];` `break;` `case 4:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] = d[n[++g]][d[n[++g]]];` `break;` `case 5:` `d[n[++g]] = d[n[++g]] >= n[++g];` `break;` `case 6:` `d[n[++g]] = d[n[++g]] >> n[++g],` `d[n[++g]] = d[n[++g]][d[n[++g]]];` `break;` `case 7:` `d[n[++g]] = d[n[++g]] < n[++g];` `break;` `case 8:` `{` `// 通常写法: d[n[++g]] = d[n[++g]].call(h)` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `// 函数` `let theFunc = d[idxFunc];` `console.log(` `` `[case 8] 准备调用: d[${idxFunc}] (func) .call(h)` `` `);` `console.log("    => 函数 theFunc = ", theFunc);` `console.log("    => this = h =", h);` ``console.log(`    => 无其他参数 (call 不带额外参数)`);`` `d[idxResult] = theFunc.call(h);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] =`,`` `d[idxResult]` `);` `}` `break;` `case 9:` `d[n[++g]] = "",` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] = n[++g];` `break;` `case 10:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = n[++g];` `const result = left | right;` ``console.log(` 按位或运算: ${left} | ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 11:` `d[n[++g]] = d[n[++g]] & n[++g],` `d[n[++g]] = d[n[++g]][d[n[++g]]];` `break;` `case 12:` `d[n[++g]] = {};` `break;` `case 13:` `d[n[++g]] = d[n[++g]] | d[n[++g]],` `d[n[++g]][d[n[++g]]] = d[n[++g]],` `g += d[n[++g]] ? n[++g] : n[(++g,` `++g)];` `break;` `case 14:` `d[n[++g]] = h;` `break;` `case 15:` `d[n[++g]] = n[++g],` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = n[++g];` `break;` `case 16:` `d[n[++g]] = !0;` `break;` `case 17:` `d[n[++g]] = d[n[++g]] === d[n[++g]];` `break;` `case 18:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = d[n[++g]];` `const result = left / right;` ``console.log(` 除法运算: ${left} / ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 19:` `d[n[++g]][d[n[++g]]] = d[n[++g]],` `d[n[++g]] = "",` `d[n[++g]] += String.fromCharCode(n[++g]);` `break;` `case 20:` `d[n[++g]][n[++g]] = d[n[++g]],` `d[n[++g]][n[++g]] = d[n[++g]],` `d[n[++g]][n[++g]] = d[n[++g]];` `break;` `case 21:` `{` `console.log('case21')` `//console.log(d)` `//console.log('case21')` `const out = n[++g];` `const left = d[n[++g]];` `const right = d[n[++g]];` `const result = left * right;` ``console.log(` 乘法运算: ${left} * ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 22:` `d[n[++g]] = ++d[n[++g]],` `d[n[++g]] = d[n[++g]];` `break;` `case 23:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] = d[n[++g]][d[n[++g]]],` `d[n[++g]] = d[n[++g]];` `break;` `case 24:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = n[++g];` `const result = left << right;` ``console.log(` 左移运算: ${left} << ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 25:` `d[n[++g]] = r(d[n[++g]]);` `break;` `case 26:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = d[n[++g]];` `const result = left | right;` ``console.log(` 按位或运算: ${left} | ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 27:` `d[n[++g]] = n[++g];` `break;` `case 28:` `{` `console.log('[Hook] case 28 start, g=', g);` `// 第一次 ++g` `const key1 = n[++g];` ``console.log(`[Hook] key1 = n[${g}] = ${key1}`);`` `// 第二次 ++g` `const key2 = n[++g];` ``console.log(`[Hook] key2 = n[${g}] = ${key2}`);`` `// 第三次 ++g` `const key3 = n[++g];` ``console.log(`[Hook] key3 = n[${g}] = ${key3}`);`` `// 取 d[key2][key3], 并赋给 d[key1]` ``console.log(`[Hook] d[${key1}] = d[${key2}][${key3}] =`, d[key2]?.[key3]);`` `d[key1] = d[key2][key3];` ``console.log(`[Hook] case 28 end, d[${key1}] =>`, d[key1]);`` `}` `break;` `case 29:` `d[n[++g]] = n[++g],` `d[n[++g]][n[++g]] = d[n[++g]],` `d[n[++g]] = n[++g];` `break;` `case 30:` `{` `// 常见写法: d[n[++g]] = d[n[++g]].call(h, d[n[++g]], d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxArg1   = n[++g];` `let idxArg2   = n[++g];` `let theFunc = d[idxFunc];` `let arg1 = d[idxArg1];` `let arg2 = d[idxArg2];` `console.log(` `` `[case 30] 准备调用: d[${idxFunc}].call(h, d[${idxArg1}], d[${idxArg2}])` `` `);` `console.log("    => 函数 theFunc  = ", theFunc);` `console.log("    => this = h = ", h);` ``console.log(`    => 参数1 = d[${idxArg1}] = `, arg1);`` ``console.log(`    => 参数2 = d[${idxArg2}] = `, arg2);`` `d[idxResult] = theFunc.call(h, arg1, arg2);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] =`,`` `d[idxResult]` `);` `}` `break;` `case 31:` `d[n[++g]] = n[++g],` `d[n[++g]] = n[++g],` `d[n[++g]] = n[++g];` `break;` `case 32:` `d[n[++g]] = n[++g],` `d[n[++g]][d[n[++g]]] = d[n[++g]];` `break;` `case 33:` `d[n[++g]] = d[n[++g]] === n[++g];` `break;` `case 34:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = n[++g];` `const result = left + right;` ``console.log(` 加法运算: ${left} + ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 35:` `d[n[++g]] += String.fromCharCode(n[++g]);` `break;` `case 36:` `d[n[++g]] = "",` `d[n[++g]] += String.fromCharCode(n[++g]);` `break;` `case 37:` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = d[n[++g]][n[++g]];` `break;` `case 38:` `d[n[++g]] = "",` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] += String.fromCharCode(n[++g]);` `break;` `case 39:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] = d[n[++g]] === d[n[++g]],` `g += d[n[++g]] ? n[++g] : n[(++g,` `++g)];` `break;` `case 40:` `d[n[++g]] = d[n[++g]] > d[n[++g]];` `break;` `case 41:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = d[n[++g]];` `const result = left - right;` ``console.log(` 减法运算: ${left} - ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 42:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = d[n[++g]];` `const result = left << right;` ``console.log(` 左移运算: ${left} << ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 43:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = d[n[++g]];` `const result = left & right;` ``console.log(` 按位与运算: ${left} & ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 44:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = n[++g];` `const result = left & right;` ``console.log(` 按位与运算（右侧为常量）: ${left} & ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 45:` `d[n[++g]] = -d[n[++g]];` `break;` `case 46:` `for (f = [],` `p = n[++g]; p > 0; p--)` `f.push(d[n[++g]]);` `d[n[++g]] = o(g + n[++g], f, a, s, c);` `//try {` `Object.defineProperty(d[n[g - 1]], "length", {` `value: n[++g],` `configurable: !0,` `writable: !1,` `enumerable: !1` `})` `//} catch (y) {}` `break;` `case 47:` `g += d[n[++g]] ? n[++g] : n[(++g,` `++g)];` `break;` `case 48:` `d[n[++g]][n[++g]] = d[n[++g]];` `break;` `case 49:` `d[n[++g]] = ~d[n[++g]];` `break;` `case 50:` `{` `// 常见写法: d[n[++g]] = d[n[++g]].call(d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxThis   = n[++g];` `let theFunc   = d[idxFunc];` `let theThis   = d[idxThis];` `console.log(` `` `[case 50] 准备调用: d[${idxFunc}].call(d[${idxThis}])` `` `);` `console.log("    => 函数 theFunc = ", theFunc);` ``console.log(`    => this = d[${idxThis}] = `, theThis);`` `console.log("    => 无其他参数");` `d[idxResult] = theFunc.call(theThis);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}` `break;` `case 51:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = d[n[++g]];` `const result = left ^ right;` ``console.log(` 异或运算: ${left} ^ ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 52:` `d[n[++g]] = ++d[n[++g]];` `break;` `case 53:` `d[n[++g]] = !1;` `break;` `case 54:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = n[++g];` `const result = left >>> right;` ``console.log(` 无符号右移运算: ${left} >>> ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 55:` `d[n[++g]][n[++g]] = d[n[++g]],` `d[n[++g]] = n[++g],` `d[n[++g]][n[++g]] = d[n[++g]];` `break;` `case 56:` `d[n[++g]] = Array(n[++g]);` `break;` `case 57:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]][n[++g]] = d[n[++g]];` `break;` `case 58:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = d[n[++g]];` `const result = left % right;` ``console.log(` 取模运算: ${left} % ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 59:` `d[n[++g]] = d[n[++g]][d[n[++g]]],` `d[n[++g]] = d[n[++g]][n[++g]];` `break;` `case 60:` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = n[++g];` `break;` `case 61:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = n[++g];` `const result = left - right;` ``console.log(` 减法运算（右侧为常量）: ${left} - ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 62:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = d[n[++g]];` `const result = left + right;` ``console.log(` 加法运算: ${left} + ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 63:` `d[n[++g]] = !d[n[++g]];` `break;` `case 64:` `// console.log(g)` `// let aa = n[++g]` `// let bb = n[++g]` `// let cc = n[++g]` `// d[aa][d[bb]] = d[cc];` `d[n[++g]][d[n[++g]]] = d[n[++g]];` `break;` `case 65:` `for (d[n[++g]] += String.fromCharCode(n[++g]),` `f = [],` `p = n[++g]; p > 0; p--)` `f.push(d[n[++g]]);` `d[n[++g]] = o(g + n[++g], f, a, s, c);` `//try {` `Object.defineProperty(d[n[g - 1]], "length", {` `value: n[++g],` `configurable: !0,` `writable: !1,` `enumerable: !1` `})` `//} catch (A) {}` `d[n[++g]][d[n[++g]]] = d[n[++g]];` `break;` `case 66:` `d[n[++g]] = d[n[++g]] - 0;` `break;` `case 67:` `{` `console.log('case67')` `//console.log(d)` `//console.log('case67')` `// 常见写法: d[n[++g]] = d[n[++g]].call(d[n[++g]], d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxThis   = n[++g];` `let idxArg    = n[++g];` `let theFunc = d[idxFunc];` `let theThis = d[idxThis];` `let theArg  = d[idxArg];` `console.log(` `` `[case 67] 准备调用: d[${idxFunc}].call(d[${idxThis}], d[${idxArg}])` `` `);` `console.log("    => 函数 theFunc = ", theFunc);` ``console.log(`    => this = d[${idxThis}] = `, theThis);`` ``console.log(`    => 参数1 = d[${idxArg}] = `, theArg);`` `d[idxResult] = theFunc.call(theThis, theArg);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}` `break;` `case 68:` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = d[n[++g]],` `d[n[++g]] = d[n[++g]] - 0;` `break;` `case 69:` `{` `console.log('[Hook] case 69 start, g =', g);` `// console.log(d)` `// ==========================` `// 第一个操作：` `// d[n[++g]] = d[n[++g]][ d[n[++g]] ]` `// ==========================` `const s1_leftKey = n[++g];` ``console.log(`[Hook] s1_leftKey = n[${g}] = ${s1_leftKey}`);`` `const s1_baseKey = n[++g];` ``console.log(`[Hook] s1_baseKey = n[${g}] = ${s1_baseKey}`);`` `const s1_subKeyIndex = n[++g];` ``console.log(`[Hook] s1_subKeyIndex = n[${g}] = ${s1_subKeyIndex}`);`` `// 右侧：d[s1_baseKey][ d[s1_subKeyIndex] ]` `const s1_subKey = d[s1_subKeyIndex];` ``console.log(`[Hook] s1_subKey = d[${s1_subKeyIndex}] =>`, s1_subKey);`` `// 读取右侧实际值` `const s1_rightVal = d[s1_baseKey]?.[s1_subKey];` ``console.log(`[Hook] s1_rightVal = d[${s1_baseKey}][${s1_subKey}] =>`, s1_rightVal);`` ``console.log(`[Hook] s1_rightVal = d[${s1_baseKey}] = ${JSON.stringify(d[s1_baseKey])}`)`` `// 赋值给左侧` `d[s1_leftKey] = s1_rightVal;` ``console.log(`[Hook] d[${s1_leftKey}] =`, d[s1_leftKey]);`` `// ==========================` `// 第二个操作：` `// d[n[++g]] = d[n[++g]] + d[n[++g]]` `// ==========================` `const s2_leftKey = n[++g];` ``console.log(`[Hook] s2_leftKey = n[${g}] = ${s2_leftKey}`);`` `const s2_rightKey1 = n[++g];` ``console.log(`[Hook] s2_rightKey1 = n[${g}] = ${s2_rightKey1}`);`` `const s2_rightKey2 = n[++g];` ``console.log(`[Hook] s2_rightKey2 = n[${g}] = ${s2_rightKey2}`);`` `const s2_val1 = d[s2_rightKey1];` `const s2_val2 = d[s2_rightKey2];` ``console.log(`[Hook] s2_val1 = d[${s2_rightKey1}] =>`, s2_val1);`` ``console.log(`[Hook] s2_val2 = d[${s2_rightKey2}] =>`, s2_val2);`` `const s2_result = s2_val1 + s2_val2;` ``console.log(`[Hook] s2_result = ${s2_val1} + ${s2_val2} =>`, s2_result);`` `d[s2_leftKey] = s2_result;` ``console.log(`[Hook] d[${s2_leftKey}] =`, d[s2_leftKey]);`` `console.log('[Hook] case 69 end');` `}` `break;` `case 70:` `{` `const out = n[++g];` `const prefix = n[++g];` `const right = d[n[++g]];` `const result = prefix + right;` ``console.log(` 加法运算（左值为常量）: ${prefix} + ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 71:` `d[n[++g]] = d[n[++g]] << d[n[++g]],` `d[n[++g]] = d[n[++g]] | d[n[++g]],` `d[n[++g]][d[n[++g]]] = d[n[++g]];` `break;` `case 72:` `{` `// 常见写法: d[n[++g]] = d[n[++g]].call(d[n[++g]], d[n[++g]], d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxThis   = n[++g];` `let idxArg1   = n[++g];` `let idxArg2   = n[++g];` `let theFunc = d[idxFunc];` `let theThis = d[idxThis];` `let arg1    = d[idxArg1];` `let arg2    = d[idxArg2];` `console.log(` `` `[case 72] 准备调用: d[${idxFunc}].call(d[${idxThis}], d[${idxArg1}], d[${idxArg2}])` `` `);` `console.log("    => 函数 theFunc = ", theFunc);` ``console.log(`    => this = d[${idxThis}] = `, theThis);`` ``console.log(`    => 参数1 = d[${idxArg1}] = `, arg1);`` ``console.log(`    => 参数2 = d[${idxArg2}] = `, arg2);`` `d[idxResult] = theFunc.call(theThis, arg1, arg2);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}` `break;` `case 73:` `{` `const out = n[++g];` `const left = d[n[++g]];` `const right = n[++g];` `const result = left >> right;` ``console.log(` 算术右移运算: ${left} >> ${right} = ${result}，写入 d[${out}]`);`` `d[out] = result;` `break;` `}` `case 74:` `d[n[++g]][d[n[++g]]] = d[n[++g]],` `d[n[++g]][d[n[++g]]] = d[n[++g]],` `d[n[++g]][d[n[++g]]] = d[n[++g]];` `break;` `case 75:` `d[n[++g]] = n[++g],` `d[n[++g]][n[++g]] = d[n[++g]],` `g += d[n[++g]] ? n[++g] : n[(++g,` `++g)];` `break;` `case 76:` `{` `// 常见写法: d[n[++g]] = d[n[++g]].call(h, d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxArg    = n[++g];` `let theFunc = d[idxFunc];` `let argVal  = d[idxArg];` `console.log(` `` `[case 76] 准备调用: d[${idxFunc}].call(h, d[${idxArg}])` `` `);` `console.log("    => 函数 theFunc =", theFunc);` `console.log("    => this = h =", h);` ``console.log(`    => 参数1 = d[${idxArg}] = `, argVal);`` `d[idxResult] = theFunc.call(h, argVal);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}` `break;` `case 77:` `{` `console.log('[Hook]77 assignment start, g =', g);` `// 第一次 ++g：获取赋值左侧的 key` `const key1 = n[++g];` ``console.log(`[Hook] key1 = n[${g}] = ${key1}`);`` `// 第二次 ++g：获取赋值右侧的 key` `const key2 = n[++g];` ``console.log(`[Hook] key2 = n[${g}] = ${key2}`);`` `// 将 d[key2] 赋给 d[key1]` `d[key1] = d[key2];` ``console.log(`[Hook] d[${key1}] = d[${key2}] =>`, d[key1]);`` `console.log('[Hook]77 assignment end');` `}` `break;` `case 78:` `{` `console.log('[Hook] case 78 start, g =', g);` `// 第一次 ++g : 用来做左侧的 key` `const key1 = n[++g];` ``console.log(`[Hook] key1 = n[${g}] = ${key1}`);`` `// 第二次 ++g : 用来做右侧第一层的 key` `const key2 = n[++g];` ``console.log(`[Hook] key2 = n[${g}] = ${key2}`);`` `// 第三次 ++g : 用来做右侧第二层 (嵌套) 的 key；实际要拿到 d[key3]，再用它作为 d[key2] 的下标` `const key3 = n[++g];` ``console.log(`[Hook] key3 = n[${g}] = ${key3}`);`` `// 计算右侧表达式` `// d[key2][ d[key3] ]` `const subKey = d[key3];` ``console.log(`[Hook] subKey = d[${key3}] =>`, subKey);`` `const rightVal = d[key2]?.[subKey];` ``console.log(`[Hook] rightVal = d[${key2}][${subKey}] =>`, rightVal);`` `try{` ``console.log(`[Hook] d[${key2}] = ${JSON.stringify(d[key2])}`)`` `}catch{` `}` `// 最终赋值给 d[key1]` `d[key1] = rightVal;` ``console.log(`[Hook] d[${key1}] now =>`, d[key1]);`` `console.log('[Hook] case 78 end');` `}` `break;` `case 79:` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = d[n[++g]] >> n[++g],` `d[n[++g]] = d[n[++g]] & n[++g];` `break;` `case 80:` `d[n[++g]] = "";` `break;` `case 81:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] += String.fromCharCode(n[++g]);` `break;` `case 82:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] = d[n[++g]][d[n[++g]]],` `g += d[n[++g]] ? n[++g] : n[(++g,` `++g)]` `}` `// } catch (b) {` `//     if (v.length > 0 && (i = []),` `//     i.push(g),` `//     0 === v.length)` `//         throw c ? c(b, d, i) : b;` `//     g = v.pop(),` `//     i.pop()` `// }` `}` `}` `, u = function(e, t, a, s, c) {` `return function l() {` `for (var f, p, d = [a, s, t, this, arguments, l, n, 0], h = void 0, g = e, v = []; ; )` `//try {` `for (; ; )` `switch (n[++g]) {` `case 0:` `d[n[++g]] = new d[n[++g]](d[n[++g]]);` `break;` `case 1:` `return d[n[++g]];` `case 2:` `for (f = [],` `p = n[++g]; p > 0; p--)` `f.push(d[n[++g]]);` `d[n[++g]] = u(g + n[++g], f, a, s, c);` `//try {` `Object.defineProperty(d[n[g - 1]], "length", {` `value: n[++g],` `configurable: !0,` `writable: !1,` `enumerable: !1` `})` `//} catch (m) {}` `break;` `case 3:` `d[n[++g]] = d[n[++g]] < d[n[++g]];` `break;` `case 4:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] = d[n[++g]][d[n[++g]]];` `break;` `case 5:` `d[n[++g]] = d[n[++g]] >= n[++g];` `break;` `case 6:` `d[n[++g]] = d[n[++g]] >> n[++g],` `d[n[++g]] = d[n[++g]][d[n[++g]]];` `break;` `case 7:` `d[n[++g]] = d[n[++g]] < n[++g];` `break;` `case 8:` `{` `// 常见写法: d[n[++g]] = d[n[++g]].call(h)` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let theFunc = d[idxFunc];` `console.log(` `` `[case 8] (u) 准备调用: d[${idxFunc}] (func).call(h)` `` `);` `console.log("    => 函数 theFunc =", theFunc);` `console.log("    => this = h =", h);` `console.log("    => 无其它实参");` `d[idxResult] = theFunc.call(h);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}` `break;` `case 9:` `d[n[++g]] = "",` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] = n[++g];` `break;` `case 10:` `d[n[++g]] = d[n[++g]] | n[++g];` `break;` `case 11:` `d[n[++g]] = d[n[++g]] & n[++g],` `d[n[++g]] = d[n[++g]][d[n[++g]]];` `break;` `case 12:` `d[n[++g]] = {};` `break;` `case 13:` `d[n[++g]] = d[n[++g]] | d[n[++g]],` `d[n[++g]][d[n[++g]]] = d[n[++g]],` `g += d[n[++g]] ? n[++g] : n[(++g,` `++g)];` `break;` `case 14:` `d[n[++g]] = h;` `break;` `case 15:` `d[n[++g]] = n[++g],` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = n[++g];` `break;` `case 16:` `d[n[++g]] = !0;` `break;` `case 17:` `d[n[++g]] = d[n[++g]] === d[n[++g]];` `break;` `case 18:` `d[n[++g]] = d[n[++g]] / d[n[++g]];` `break;` `case 19:` `d[n[++g]][d[n[++g]]] = d[n[++g]],` `d[n[++g]] = "",` `d[n[++g]] += String.fromCharCode(n[++g]);` `break;` `case 20:` `d[n[++g]][n[++g]] = d[n[++g]],` `d[n[++g]][n[++g]] = d[n[++g]],` `d[n[++g]][n[++g]] = d[n[++g]];` `break;` `case 21:` `d[n[++g]] = d[n[++g]] * d[n[++g]];` `break;` `case 22:` `d[n[++g]] = ++d[n[++g]],` `d[n[++g]] = d[n[++g]];` `break;` `case 23:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] = d[n[++g]][d[n[++g]]],` `d[n[++g]] = d[n[++g]];` `break;` `case 24:` `d[n[++g]] = d[n[++g]] << n[++g];` `break;` `case 25:` `d[n[++g]] = r(d[n[++g]]);` `break;` `case 26:` `d[n[++g]] = d[n[++g]] | d[n[++g]];` `break;` `case 27:` `d[n[++g]] = n[++g];` `break;` `case 28:` `d[n[++g]] = d[n[++g]][n[++g]];` `break;` `case 29:` `d[n[++g]] = n[++g],` `d[n[++g]][n[++g]] = d[n[++g]],` `d[n[++g]] = n[++g];` `break;` `case 30:` `{` `// 常见写法: d[n[++g]] = d[n[++g]].call(h, d[n[++g]], d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxArg1   = n[++g];` `let idxArg2   = n[++g];` `let theFunc = d[idxFunc];` `let arg1 = d[idxArg1];` `let arg2 = d[idxArg2];` `console.log(` `` `[case 30] (u) 准备调用: d[${idxFunc}].call(h, d[${idxArg1}], d[${idxArg2}])` `` `);` `console.log("    => 函数 theFunc =", theFunc);` `console.log("    => this = h =", h);` ``console.log(`    => 参数1 = d[${idxArg1}] =`, arg1);`` ``console.log(`    => 参数2 = d[${idxArg2}] =`, arg2);`` `d[idxResult] = theFunc.call(h, arg1, arg2);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}{` `// 常见写法: d[n[++g]] = d[n[++g]].call(h, d[n[++g]], d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxArg1   = n[++g];` `let idxArg2   = n[++g];` `let theFunc = d[idxFunc];` `let arg1 = d[idxArg1];` `let arg2 = d[idxArg2];` `console.log(` `` `[case 30] (u) 准备调用: d[${idxFunc}].call(h, d[${idxArg1}], d[${idxArg2}])` `` `);` `console.log("    => 函数 theFunc =", theFunc);` `console.log("    => this = h =", h);` ``console.log(`    => 参数1 = d[${idxArg1}] =`, arg1);`` ``console.log(`    => 参数2 = d[${idxArg2}] =`, arg2);`` `d[idxResult] = theFunc.call(h, arg1, arg2);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}` `break;` `case 31:` `d[n[++g]] = n[++g],` `d[n[++g]] = n[++g],` `d[n[++g]] = n[++g];` `break;` `case 32:` `d[n[++g]] = n[++g],` `d[n[++g]][d[n[++g]]] = d[n[++g]];` `break;` `case 33:` `d[n[++g]] = d[n[++g]] === n[++g];` `break;` `case 34:` `d[n[++g]] = d[n[++g]] + n[++g];` `break;` `case 35:` `d[n[++g]] += String.fromCharCode(n[++g]);` `break;` `case 36:` `d[n[++g]] = "",` `d[n[++g]] += String.fromCharCode(n[++g]);` `break;` `case 37:` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = d[n[++g]][n[++g]];` `break;` `case 38:` `d[n[++g]] = "",` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] += String.fromCharCode(n[++g]);` `break;` `case 39:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] = d[n[++g]] === d[n[++g]],` `g += d[n[++g]] ? n[++g] : n[(++g,` `++g)];` `break;` `case 40:` `d[n[++g]] = d[n[++g]] > d[n[++g]];` `break;` `case 41:` `d[n[++g]] = d[n[++g]] - d[n[++g]];` `break;` `case 42:` `d[n[++g]] = d[n[++g]] << d[n[++g]];` `break;` `case 43:` `d[n[++g]] = d[n[++g]] & d[n[++g]];` `break;` `case 44:` `d[n[++g]] = d[n[++g]] & n[++g];` `break;` `case 45:` `d[n[++g]] = -d[n[++g]];` `break;` `case 46:` `for (f = [],` `p = n[++g]; p > 0; p--)` `f.push(d[n[++g]]);` `d[n[++g]] = o(g + n[++g], f, a, s, c);` `//try {` `Object.defineProperty(d[n[g - 1]], "length", {` `value: n[++g],` `configurable: !0,` `writable: !1,` `enumerable: !1` `})` `//} catch (y) {}` `break;` `case 47:` `g += d[n[++g]] ? n[++g] : n[(++g,` `++g)];` `break;` `case 48:` `d[n[++g]][n[++g]] = d[n[++g]];` `break;` `case 49:` `d[n[++g]] = ~d[n[++g]];` `break;` `case 50:` `{` `// 常见写法: d[n[++g]] = d[n[++g]].call(d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxThis   = n[++g];` `let theFunc = d[idxFunc];` `let theThis = d[idxThis];` `console.log(` `` `[case 50] (u) 准备调用: d[${idxFunc}].call(d[${idxThis}])` `` `);` `console.log("    => 函数 theFunc =", theFunc);` ``console.log(`    => this = d[${idxThis}] =`, theThis);`` `console.log("    => 无其它参数");` `d[idxResult] = theFunc.call(theThis);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}` `break;` `case 51:` `{` `console.log('[Hook] case 51 start, g =', g);` `// 第一次 ++g：左侧的 key` `const key1 = n[++g];` ``console.log(`[Hook] key1 = n[${g}] = ${key1}`);`` `// 第二次 ++g：^ 运算中的第一个值所在的 key` `const key2 = n[++g];` ``console.log(`[Hook] key2 = n[${g}] = ${key2}`);`` `// 第三次 ++g：^ 运算中的第二个值所在的 key` `const key3 = n[++g];` ``console.log(`[Hook] key3 = n[${g}] = ${key3}`);`` `// 取出要做异或(^)的两个值` `const val2 = d[key2];` `const val3 = d[key3];` ``console.log(`[Hook] val2 = d[${key2}] =>`, val2);`` ``console.log(`[Hook] val3 = d[${key3}] =>`, val3);`` `// 计算异或结果` `const xorResult = val2 ^ val3;` ``console.log(`[Hook] xorResult = ${val2} ^ ${val3} =>`, xorResult);`` `// 赋给 d[key1]` `d[key1] = xorResult;` ``console.log(`[Hook] d[${key1}] =`, d[key1]);`` `console.log('[Hook] case 51 end');` `}` `break;` `case 52:` `d[n[++g]] = ++d[n[++g]];` `break;` `case 53:` `d[n[++g]] = !1;` `break;` `case 54:` `d[n[++g]] = d[n[++g]] >>> n[++g];` `break;` `case 55:` `d[n[++g]][n[++g]] = d[n[++g]],` `d[n[++g]] = n[++g],` `d[n[++g]][n[++g]] = d[n[++g]];` `break;` `case 56:` `d[n[++g]] = Array(n[++g]);` `break;` `case 57:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]][n[++g]] = d[n[++g]];` `break;` `case 58:` `d[n[++g]] = d[n[++g]] % d[n[++g]];` `break;` `case 59:` `d[n[++g]] = d[n[++g]][d[n[++g]]],` `d[n[++g]] = d[n[++g]][n[++g]];` `break;` `case 60:` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = n[++g];` `break;` `case 61:` `d[n[++g]] = d[n[++g]] - n[++g];` `break;` `case 62:` `d[n[++g]] = d[n[++g]] + d[n[++g]];` `break;` `case 63:` `d[n[++g]] = !d[n[++g]];` `break;` `case 64:` `d[n[++g]][d[n[++g]]] = d[n[++g]];` `break;` `case 65:` `for (d[n[++g]] += String.fromCharCode(n[++g]),` `f = [],` `p = n[++g]; p > 0; p--)` `f.push(d[n[++g]]);` `d[n[++g]] = o(g + n[++g], f, a, s, c);` `//try {` `Object.defineProperty(d[n[g - 1]], "length", {` `value: n[++g],` `configurable: !0,` `writable: !1,` `enumerable: !1` `})` `//} catch (A) {}` `d[n[++g]][d[n[++g]]] = d[n[++g]];` `break;` `case 66:` `d[n[++g]] = d[n[++g]] - 0;` `break;` `case 67:` `{` `// 常见写法: d[n[++g]] = d[n[++g]].call(d[n[++g]], d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxThis   = n[++g];` `let idxArg    = n[++g];` `let theFunc = d[idxFunc];` `let theThis = d[idxThis];` `let theArg  = d[idxArg];` `console.log(` `` `[case 67] (u) 准备调用: d[${idxFunc}].call(d[${idxThis}], d[${idxArg}])` `` `);` `console.log("    => 函数 theFunc =", theFunc);` ``console.log(`    => this = d[${idxThis}] =`, theThis);`` ``console.log(`    => 参数1 = d[${idxArg}] =`, theArg);`` `d[idxResult] = theFunc.call(theThis, theArg);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}` `break;` `case 68:` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = d[n[++g]],` `d[n[++g]] = d[n[++g]] - 0;` `break;` `case 69:` `d[n[++g]] = d[n[++g]][d[n[++g]]],` `d[n[++g]] = d[n[++g]] + d[n[++g]];` `break;` `case 70:` `d[n[++g]] = n[++g] + d[n[++g]];` `break;` `case 71:` `d[n[++g]] = d[n[++g]] << d[n[++g]],` `d[n[++g]] = d[n[++g]] | d[n[++g]],` `d[n[++g]][d[n[++g]]] = d[n[++g]];` `break;` `case 72:` `{` `// d[n[++g]] = d[n[++g]].call(d[n[++g]], d[n[++g]], d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxThis   = n[++g];` `let idxArg1   = n[++g];` `let idxArg2   = n[++g];` `let theFunc = d[idxFunc];` `let theThis = d[idxThis];` `let arg1 = d[idxArg1];` `let arg2 = d[idxArg2];` `console.log(` `` `[case 72] (u) 准备调用: d[${idxFunc}].call(d[${idxThis}], d[${idxArg1}], d[${idxArg2}])` `` `);` `console.log("    => 函数 theFunc =", theFunc);` ``console.log(`    => this = d[${idxThis}] =`, theThis);`` ``console.log(`    => 参数1 = d[${idxArg1}] =`, arg1);`` ``console.log(`    => 参数2 = d[${idxArg2}] =`, arg2);`` `d[idxResult] = theFunc.call(theThis, arg1, arg2);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}` `break;` `case 73:` `d[n[++g]] = d[n[++g]] >> n[++g];` `break;` `case 74:` `d[n[++g]][d[n[++g]]] = d[n[++g]],` `d[n[++g]][d[n[++g]]] = d[n[++g]],` `d[n[++g]][d[n[++g]]] = d[n[++g]];` `break;` `case 75:` `d[n[++g]] = n[++g],` `d[n[++g]][n[++g]] = d[n[++g]],` `g += d[n[++g]] ? n[++g] : n[(++g,` `++g)];` `break;` `case 76:` `{` `// d[n[++g]] = d[n[++g]].call(h, d[n[++g]]);` `let idxResult = n[++g];` `let idxFunc   = n[++g];` `let idxArg    = n[++g];` `let theFunc = d[idxFunc];` `let argVal  = d[idxArg];` `console.log(` `` `[case 76] (u) 准备调用: d[${idxFunc}].call(h, d[${idxArg}])` `` `);` `console.log("    => 函数 theFunc =", theFunc);` `console.log("    => this = h =", h);` ``console.log(`    => 参数1 = d[${idxArg}] =`, argVal);`` `d[idxResult] = theFunc.call(h, argVal);` `console.log(` `` `    => 调用结束，赋值 d[${idxResult}] = `,`` `d[idxResult]` `);` `}` `break;` `case 77:` `{` `console.log('[Hook] case 77 start, g =', g);` `// 第一次 ++g：用来作为左侧的 key` `const key1 = n[++g];` ``console.log(`[Hook] key1 = n[${g}] = ${key1}`);`` `// 第二次 ++g：用来作为右侧的 key` `const key2 = n[++g];` ``console.log(`[Hook] key2 = n[${g}] = ${key2}`);`` `// 取 d[key2] 并赋给 d[key1]` `const rightVal = d[key2];` ``console.log(`[Hook] rightVal = d[${key2}] =>`, rightVal);`` `d[key1] = rightVal;` ``console.log(`[Hook] d[${key1}] =`, d[key1]);`` `console.log('[Hook] case 77 end');` `}` `break;` `case 78:` `d[n[++g]] = d[n[++g]][d[n[++g]]];` `break;` `case 79:` `d[n[++g]] = d[n[++g]][n[++g]],` `d[n[++g]] = d[n[++g]] >> n[++g],` `d[n[++g]] = d[n[++g]] & n[++g];` `break;` `case 80:` `d[n[++g]] = "";` `break;` `case 81:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] += String.fromCharCode(n[++g]);` `break;` `case 82:` `d[n[++g]] += String.fromCharCode(n[++g]),` `d[n[++g]] = d[n[++g]][d[n[++g]]],` `g += d[n[++g]] ? n[++g] : n[(++g,` `++g)]` `}` `// } catch (b) {` `//     if (v.length > 0 && (i = []),` `//     i.push(g),` `//     0 === v.length)` `//         throw c ? c(b, d, i) : b;` `//     g = v.pop(),` `//     i.pop()` `// }` `}` `};` `return t ? o : u` `}` `}` `)()("你那么聪明肯定做得出来的！", !1)(3944, [], n, [void 0, 1732584193, 4023233417, 2562383102, 3285377520, !1, !0, 2147483648, 4294967295, 4294967296, 1518500249, 1859775393, 1894007588], void 0)();` `var i = n._getSecuritySign;` `delete n._getSecuritySign,` `t.default = i` `}` `.call(this, window)` `//.call(this, n(80))` `},` `// '80':function(){` `//     return Window` `// },``});``o = webpack_output(350).default``input = '12345'``a = o(input)``if(a=='zzca873dd41zwq69wr8hrqun6rvk1b5srwqncdc4c4d03'){` `console.log('答案校验通过')``console.log(a)``}else{` `console.log('环境检测不通过')``}``console.log()`
```

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0ogcYOYg86YN0lqPfnibkkahDTAaAL03dk9r8E9CqYShqMZNUJYB9pzibWrIwh28B3Fkmb49Cc0Catw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

  

**·** **今 日 推 荐** **·**

  

本文内容来自网络，如有侵权请联系删除

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0ogcYOYg86YN0lqPfnibkkah7OibqMm6d8goQA1AHrMERTLyJNTbXlqbzsKcsBXBs22icmdCiahrsRP4g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)