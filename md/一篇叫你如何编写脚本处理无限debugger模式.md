> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/pBT32MTOAdV15ow0YjZ5_Q)

😀 喜欢看的小伙记得关注我哦。

前几天收到一个小伙伴说有一个网站一直过不掉，试了一些方法都无法通过我这边今天得空看了一下。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/N1Fms3bbiaBAbQTSn6JVTGB3BsfNzLbG7ahlopB86d1t1lYSU4IlET9moia5UTHYlrKGwhVe10gx8CDpmGSgZh5g/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

📝 主旨内容
=======

目标网站
----

> 网站：aHR0cHM6Ly9qaXNodWxpbmsuY29tL3ZpZGVvL2MyNDYzMTY= 操作：点击网站的播放就可以看到本文章需要解决的问题

初步尝试
----

> 我们打开控制台之后就是如下的效果。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

首先我是用了最简单的模式，就是在这个地方去添加一个条件判断。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

然后再这里输入一个false。然后再回车。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

然后直接点击下一步执行。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

然后你就会发现神奇的一幕，他要跳转到一个新的debugger断点。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

然后再重新上面的操作，添加条件断点，然后填写false，在回车。你会发现，不会在弹出这个debugger断点了，但是呢浏览器卡炸了。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

而且视频也是加载了半天没有加载出来。什么都动不了。我发现行不通之后，我就开始重新分析。

Hook掉无线debugger模式

重新打开网站

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

我们可以在堆栈这里看到这个无线debugger的执行之前的一些逻辑。我们这里往上翻，找到他执行之前是怎么样样子的。

点到下一个个堆栈的时候，我就看到了。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

这段代码大体就是如下：

### 1. **`function() {}`**

*   定义了一个匿名函数，但未命名或执行。
    
*   本质上，这个匿名函数是一个普通的 `Function` 对象。
    

* * *

### 2. **`.constructor`**

*   `.constructor`
    
     是 JavaScript 中每个对象都具有的属性。
    
*   对于函数，`.constructor` 通常是指向 `Function` 构造函数。
    
*   即 `function() {}.constructor` 其实返回了 `Function` 构造函数本身。
    

* * *

### 3. **`("debugger")`**

*   调用了 `Function` 构造函数，传入了一个字符串 `"debugger"`。
    
*   `Function` 构造函数的用法是动态创建新的函数，语法为：
    
    ```
    new Function([arg1[, arg2[, ...argN]],] functionBody)  
    
    ```
    

*   它将参数解析为一个函数体。例如：
    
    ```
    const fn = new Function("a", "b", "return a + b");  
    console.log(fn(1, 2)); // 输出 3  
    
    ```
    

*   在这里，`"debugger"` 被作为函数体传入，因此会动态创建一个新函数，函数内容是：
    
    ```
    debugger;  
    
    ```
    
*   这个新函数在执行时会触发 JavaScript 调试器。
    

由此我想到了一个方案，我只要Hook这个函数，然后再判断他传入的参数是什么，如果包含这个debugger,那我加把他内容剔除掉，不让他执行这一部分的代码。

这里我们就需要用到油猴的这个插件。油猴的插件安装就直接去谷歌商店下载即可，或者去这里（https://www.tampermonkey.net/index.php?browser=chrome&locale=zh）安装也可以，这里我就不详细的编写了。

然后我们还需要了解一下油猴的配置。

### **1. `@name`**

*   **功能**
    
    ：定义脚本的名称。
    
*   **作用**
    
    ：在油猴管理面板中显示脚本的名称，方便识别。
    
*   **当前值**
    
    ：`New Userscript`，可以改为更具描述性的名字，比如 `My Custom Script`。
    

* * *

### **2. `@namespace`**

*   **功能**
    
    ：用于区分不同脚本的作用域。
    
*   **作用**
    
    ：避免命名冲突，比如不同开发者可能创建了名字相同的脚本，但通过 `@namespace` 可以区分开。
    
*   **当前值**
    
    ：`http://tampermonkey.net/`，这是默认值。如果你开发自己的脚本，建议改为自己的域名或唯一标识符。
    

* * *

### **3. `@version`**

*   **功能**
    
    ：定义脚本的版本号。
    
*   **作用**
    
    ：标记脚本的版本，用于更新或管理。
    
*   **当前值**
    
    ：`2025-01-12`，通常使用语义化版本（如 `1.0.0`）或日期作为版本号。
    
*   **注意**
    
    ：当脚本版本更新时，油猴会根据版本号自动检查并提示更新。
    

* * *

### **4. `@description`**

*   **功能**
    
    ：描述脚本的功能或用途。
    
*   **作用**
    
    ：显示在油猴管理界面中，用于快速了解脚本的用途。
    
*   **当前值**
    
    ：`try to take over the world!`，建议改为更清晰的描述，比如 `Custom script for enhancing functionality on jishulink.com`.
    

* * *

### **5. `@author`**

*   **功能**
    
    ：定义脚本的作者。
    
*   **作用**
    
    ：标识脚本开发者，方便用户或其他开发者联系作者。
    
*   **当前值**
    
    ：`You`，建议改为真实名称或昵称。
    

* * *

### **6. `@match`**

*   **功能**
    
    ：定义脚本的运行匹配规则。
    
*   **作用**
    
    ：指定脚本在哪些网站或页面上运行。
    
*   **当前值**
    
    ：`https://jishulink.com/video/c246316`，表示脚本只在这个特定页面上生效。
    

**注意事项**：

*   支持通配符（``），例如：
    

*   `https://jishulink.com/*`
    
    ：适配 `jishulink.com` 下的所有页面。
    
*   `://*.jishulink.com/*`
    
    ：适配所有子域名和页面。
    

*   如果想让脚本在多个网站运行，可以添加多个 `@match` 条目。
    

* * *

### **7. `@icon`**

*   **功能**
    
    ：定义脚本的图标。
    
*   **作用**
    
    ：在油猴管理界面中显示为脚本的图标。
    
*   **当前值**
    
    ：从 `jishulink.com` 自动生成的 favicon。
    
*   **建议**
    
    ：你可以替换为自定义图标（例如一个图片的 URL）。
    

* * *

### **8. `@grant`**

*   **功能**
    
    ：定义脚本所需的特殊权限。
    
*   **作用**
    
    ：告知油猴脚本需要哪些 API（如 GM API）。
    
*   **当前值**
    
    ：`none`，表示不需要特殊权限。
    

**常见值**：

*   `@grant GM_xmlhttpRequest`
    
    ：允许使用跨域请求功能。
    
*   `@grant GM_setValue`
    
    ：允许保存数据到油猴的存储。
    
*   `@grant GM_getValue`
    
    ：允许读取油猴存储的数据。
    
*   如果不需要特殊权限，保持 `none` 是最佳实践。
    

然后我们根据这一部分的代码编写对应的Hook代码如下：

```
// ==UserScript==  
// @name         New Userscript  
// @namespace    http://tampermonkey.net/  
// @version      2025-01-12  
// @description  try to take over the world!  
// @author       You  
// @match        https://jishulink.com/video/c246316  
// @icon         https://www.google.com/s2/favicons?sz=64&domain=jishulink.com  
// @grant        none  
// ==/UserScript==  
  
(function() {  
    'use strict';  
  
    // Your code here...  
    const originalConstructor = Function.prototype.constructor;  
    Function.prototype.constructor = function(...args) {  
        console.log("Function constructor called with arguments:", args);  
        if (args.length && args[0].includes("debugger")) {  
            console.warn("Attempt to use 'debugger' in Function constructor intercepted!");  
            args[0] = args[0].replace(/debugger/g, "console.log('debugger intercepted');");  
            return;  
        };  
        return originalConstructor.apply(this, args);  
    };  
    console.log("Function.prototype.constructor is now hooked!");  
})();  

```

然后我们在回到之前的网站，重新打开。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

已经Hook到这个函数了。并且现在播放也正常了。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

结束收工。这里可以过掉这种用函数无限启动的debugger的场景。