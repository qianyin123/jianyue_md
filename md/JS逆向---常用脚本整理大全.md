> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/9D_HFqF80uSFoXe7Wei8Lg)

1：cv2识别滑块缺口距离
=============

```
`import cv2``def get_dis(bg, fg):` `"""` `计算前景图像 (fg) 和背景图像 (bg) 在匹配模板中计算的距离。` `参数:` `bg (str): 背景图像的文件路径。` `fg (str): 前景图像的文件路径。` `返回:` `float: 根据匹配结果计算的距离值。` `"""` `try:` `# 读取背景图像和前景图像` `img = cv2.imread(bg)` `temp = cv2.imread(fg)` `# 检查图像是否正确加载` `if img is None:` `raise ValueError(f"无法加载背景图像: {bg}")` `if temp is None:` `raise ValueError(f"无法加载前景图像: {fg}")` `# 使用模板匹配计算相似性` `res = cv2.matchTemplate(img, temp, cv2.TM_CCORR_NORMED)` `# 获取匹配结果中的最大值位置` `value = cv2.minMaxLoc(res)[2][0]` `# 根据公式计算距离` `dis = value * 242 / 360` `return dis` `except Exception as e:` `print(f"发生错误: {e}")` `return None`
```

2.常用的Hook脚本
-----------

### Hook Json.stringfy

```
`(function() {` `// 保存原始的 JSON.stringify 方法` `var originalStringify = JSON.stringify;` `// 重写 JSON.stringify 方法` `JSON.stringify = function(params) {` `// 打印日志，方便调试` `console.log("Hook JSON.stringify ->", params);` `// 触发调试器` `debugger;` `// 调用原始的 JSON.stringify 方法` `return originalStringify(params);` `};` `console.log("JSON.stringify 已被挂钩");``})();`
```

  

### Hook Json.parse

```
`(function() {` `// 保存原始的 JSON.parse 方法` `var originalParse = JSON.parse;` `// 重写 JSON.parse 方法` `JSON.parse = function(params) {` `// 打印日志，便于调试` `console.log("Hook JSON.parse ->", params);` `// 触发调试器` `debugger;` `// 调用原始的 JSON.parse 方法` `return originalParse(params);` `};` `console.log("JSON.parse 已被挂钩");``})();`
```

  

### Hook Header

```
`(function () {` `// 保存原始的 XMLHttpRequest.prototype.setRequestHeader 方法` `var originalSetRequestHeader = window.XMLHttpRequest.prototype.setRequestHeader;` `// 重写 setRequestHeader 方法` `window.XMLHttpRequest.prototype.setRequestHeader = function (key, value) {` `// 检查是否拦截到 Authorization 头` `if (key === 'Authorization') {` `console.log("拦截到 Authorization 请求头:", value);` `// 触发调试器` `debugger;` `}` `// 调用原始 setRequestHeader 方法` `return originalSetRequestHeader.apply(this, arguments);` `};` `console.log("XMLHttpRequest.setRequestHeader 已被挂钩");``})();`
```

  

### Hook Url

```
`(function () {` `// 保存原始的 XMLHttpRequest.prototype.open 方法` `var originalOpen = window.XMLHttpRequest.prototype.open;` `// 重写 open 方法` `window.XMLHttpRequest.prototype.open = function (method, url, async) {` `// 检查 URL 是否包含 "login"` `if (url.indexOf("login") !== -1) {` `console.log("拦截到包含 'login' 的请求 URL:", url);` `// 触发调试器` `debugger;` `}` `// 调用原始 open 方法` `return originalOpen.apply(this, arguments);` `};` `console.log("XMLHttpRequest.open 已被挂钩");``})();`
```

  

### Hook Eval

```
`(function () {` `// 保存原始的 eval 方法` `window.__cr_eval = window.eval;` `// 自定义 eval 方法` `var myEval = function (src) {` `// 打印被传递到 eval 的代码` `console.log("执行的 eval 代码:");` `console.log(src);` `console.log("=============== eval end ===============");` `// 触发调试器，便于调试` `debugger;` `// 调用原始的 eval 方法` `return window.__cr_eval(src);` `};` `// 绑定上下文并保留原始 toString 方法` `var _myEval = myEval.bind(null);` `_myEval.toString = window.__cr_eval.toString;` `// 使用 Object.defineProperty 重写 window.eval` `Object.defineProperty(window, 'eval', {` `value: _myEval,` `writable: false, // 禁止重新赋值` `configurable: false // 禁止重新配置` `});` `console.log("window.eval 已被挂钩");``})();`
```

  

### Hook Cookie

```
`(function () {` `'use strict';` `// 临时存储 cookie 的变量` `var cookieTemp = '';` `// 使用 Object.defineProperty 重写 document.cookie 的 setter 和 getter` `Object.defineProperty(document, 'cookie', {` `set: function (val) {` `// 如果设置的 cookie 包含特定关键字 '__dfp'，触发调试器` `if (val.indexOf('__dfp') !== -1) {` `console.log('拦截到包含 "__dfp" 的 cookie 设置:', val);` `debugger;` `}` `// 打印捕获到的 cookie 设置` `console.log('Hook 捕获到 cookie 设置 ->', val);` `// 更新临时存储变量` `cookieTemp = val;` `return val;` `},` `get: function () {` `// 返回临时存储的 cookie` `return cookieTemp;` `}` `});` `console.log('document.cookie 的 setter 和 getter 已被挂钩');``})();``(function () {` `'use strict';` `// 获取 document.cookie 原始的 setter 方法` `var originalSetter = document.cookie.__lookupSetter__('cookie');` `// 使用 __defineSetter__ 重写 document.cookie 的 setter` `document.__defineSetter__('cookie', function (cookie) {` `// 如果设置的 cookie 包含特定关键字 '__dfp'，触发调试器` `if (cookie.indexOf('__dfp') !== -1) {` `console.log('拦截到包含 "__dfp" 的 cookie 设置:', cookie);` `debugger;` `}` `// 调用原始的 setter 方法` `originalSetter = cookie;` `});` `// 使用 __defineGetter__ 重写 document.cookie 的 getter` `document.__defineGetter__('cookie', function () {` `// 返回通过原始 setter 方法存储的 cookie` `return originalSetter;` `});` `console.log('document.cookie 的 setter 和 getter 已被通过 __defineSetter__ 和 __defineGetter__ 挂钩');``})();`
```

### Hook Function

```
`(function () {` `'use strict';` `// 保存原始的 Function 构造函数` `window.__cr_fun = window.Function;` `// 自定义 Function 构造函数` `var myFunction = function () {` `// 获取参数列表（除最后一个参数外）` `var args = Array.prototype.slice.call(arguments, 0, -1).join(",");` `// 获取函数体（最后一个参数）` `var src = arguments[arguments.length - 1];` `// 输出拦截到的函数体内容` `console.log("捕获到 Function 构造函数体:");` `console.log(src);` `console.log("=============== Function end ===============");` `// 触发调试器，便于调试` `debugger;` `// 调用原始的 Function 构造函数` `return window.__cr_fun.apply(this, arguments);` `};` `// 重写 toString 方法以保持与原始 Function 一致的表现` `myFunction.toString = function () {` `return window.__cr_fun.toString();` `};` `// 使用 Object.defineProperty 替换 window.Function` `Object.defineProperty(window, 'Function', {` `value: myFunction,` `writable: false, // 禁止重新赋值` `configurable: false // 禁止重新配置` `});` `console.log('window.Function 已被挂钩');``})();`
```

Hook无限debugger

```
`// 重写 Function 构造函数的 constructor 方法``Function.prototype.constructor_ = Function.prototype.constructor;``Function.prototype.constructor = function (code) {` `// 检查传入的代码是否包含 "debugger"` `if (code === "debugger") {` `// 返回一个空函数，避免执行 "debugger"` `return function () {};` `}` `// 调用原始的 constructor 方法处理其他代码` `return Function.prototype.constructor_.call(this, code);``};``// 重写 setInterval 函数``var originalSetInterval = setInterval;``setInterval = function (callback, delay) {` `// 检查定时器的延迟是否为特定值 0x7d0 (2000 毫秒)` `if (delay === 0x7d0) {` `// 返回一个空函数，避免定时器执行` `return function () {};` `}` `// 调用原始的 setInterval 方法处理其他情况` `return originalSetInterval(callback, delay);``};``// 示例：通过 eval 执行 "debugger;" 的处理逻辑``// eval("debugger;");`
```

通过反调试

```
  

```

```
`(function () {` `// 保存原始的 Function.prototype.constructor` `var originalConstructor = unsafeWindow.Function.prototype.constructor;` `// 重写 Function.prototype.constructor` `unsafeWindow.Function.prototype.constructor = function () {` `var functionContent = arguments[0]; // 获取传入的函数内容` `// 检查函数内容是否存在` `if (functionContent) {` `// 如果函数内容包含 "debugger"，则可能是反调试机制` `if (functionContent.includes('debugger')) {` `// 获取调用该构造函数的函数` `var caller = Function.prototype.constructor.caller;` `// 检查调用者的代码内容是否包含 "debugger"` `if (caller) {` `var callerContent = caller.toString();` `if (callerContent.match(/\bdebugger\b/gi)) {` `// 移除调用者中的所有 "debugger" 语句` `callerContent = callerContent.replace(/\bdebugger\b/gi, '');` `// 使用 eval 替换调用者的代码` `eval('caller = ' + callerContent);` `}` `}` `// 返回一个空函数，防止原函数内容执行` `return function () {};` `}` `}` `// 如果没有特殊情况，执行原始的 Function 构造函数` `return originalConstructor.apply(this, arguments);` `};` `console.log('Function.prototype.constructor 已成功挂钩');``})();`
```

感谢一路以来的陪伴和支持，你们的每一次点赞、评论、分享，都是我前行的动力。在这个快速变化的世界里，能有你们的鼓励与支持，让我更加坚定地追逐梦想，努力创造更多有价值的内容。

  

新的一年已经悄然启程，我衷心祝愿每一位家人都能心想事成，万事如意！无论是工作、生活，还是追逐梦想的路上，都能充满阳光与希望。愿你们健康常伴，幸福相随！

  

未来的路，我们一起走下去。你们的支持是我不断前进的力量源泉，让我们共同迎接更加美好的明天！