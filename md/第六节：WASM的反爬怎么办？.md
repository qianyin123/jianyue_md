> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-G2iN1JmBS_yJclyY9qDzQ)

  

WASM反爬
------

通过WASM进行反爬，其实就是将加密的核心逻辑通过C/C++进行实现，然后通过JavaScript调用api，生成对应的加密参数。那么，遇到此类生成的加密参数，我们该怎么破解？

pywasm
------

对于一般的WASM生成的加密参数，我们都可以用pywasm这个python包来进行尝试。我们借用part2的代码来进行演示：

```


  

```
<!doctype html>
```

```
<html lang="en">
```

```
<head>
```

```
    <meta charset="UTF-8">
```

```
    <meta 
```

```
          content="width=device-width, user-sca>
```

```
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
```

```
    <title>Document</title>
```

```
</head>
```

```
<body>
```

```
<script>
```

```
  

```

```
    window.Module = {}
```

```
    window.Module.onRuntimeInitialized = function () {
```

```
        console.log(Module._add(1.2, 3.4));
```

```
        // x();
```

```
    };
```

```
  

```

```
    // function x() {
```

```
    //     console.log(window.Module._add(1.5, 1.6));
```

```
    // }
```

```
    // setInterval(x, 1000);
```

```
</script>
```

```
  

```

```
<script src="add.js"></script>
```

```
</body>
```

```
</html>
```


```

比如， 我想调用WASM实现的add这个方法，我需要怎么来写呢？

```


  

```
import pywasm
```

```
  

```

```
runtime = pywasm.load("add.wasm")
```

```
r = runtime.exec("add", [1.2, 3.4])
```

```
print(r)
```


```

这样，你应该可以得到和页面一样的输出。

但是，如果我使用的是part3的代码呢？

```


  

```
<!DOCTYPE html>
```

```
<html lang="en">
```

```
<head>
```

```
    <meta charset="UTF-8">
```

```
    <title>Title</title>
```

```
</head>
```

```
<body>
```

```
  

```

```
<script>
```

```
    Module = {}
```

```
  

```

```
    Module.onRuntimeInitialized = function () {
```

```
        console.log(Module._compute(1.3, 1.4, 1));
```

```
        console.log(Module._compute(1.5, 1.1, 2));
```

```
    }
```

```
  

```

```
</script>
```

```
  

```

```
<script src="compute_func.js"></script>
```

```
</body>
```

```
</html>
```


```

这时候， 我们将python代码改成如下形式：

```


  

```
import pywasm
```

```
  

```

```
runtime = pywasm.load("add.wasm")
```

```
r = runtime.exec("add", [1.2, 3.4])
```

```
print(r)
```

```
  

```

```
runtime = pywasm.load("compute_func.wasm")
```

```
r = runtime.exec("compute", [1.2, 3.4, 1])
```

```
print(r)
```


```

你将会发现报了如下错误：

```


  

```
Traceback (most recent call last):
```

```
  File "C:/Users/wpj/Downloads/project/破解/js破解/wasm_test/test.py", line 7, in <module>
```

```
    runtime = pywasm.load("compute_func.wasm")
```

```
  File "C:\Users\wpj\Downloads\software\miniconda\envs\py372\lib\site-packages\pywasm\__init__.py", line 99, in load
```

```
    return Runtime(module, imps, opts)
```

```
  File "C:\Users\wpj\Downloads\software\miniconda\envs\py372\lib\site-packages\pywasm\__init__.py", line 28, in __init__
```

```
    raise Exception(f'pywasm: missing import {e.module}.{e.name}')
```

```
Exception: pywasm: missing import env.add
```

```
4.6
```


```

那么遇到这类问题，我们该怎么处理呢？

nodejs
------

其实，遇到此类问题，我们可以直接用node来处理了，我们只需要将其js的代码全盘复制进去然后进行执行即可：

```


  

```
compute_func.js的代码
```

```
  

```

```
Module.onRuntimeInitialized = function () {
```

```
    console.log(Module._compute(1.3, 1.4, 1));
```

```
    console.log(Module._compute(1.5, 1.1, 2));
```

```
}
```

```
  

```


```

之后，将可以看到：

```


  

```
PS C:\Users\wpj\Downloads\project\破解\js破解\wasm_test> node .\test.js
```

```
调用加法了哦
```

```
2.7
```

```
调用减法了哦
```

```
0.3999999999999999
```


```

输出一模一样哦