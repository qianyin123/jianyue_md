> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XJjO7F_K82lbqhxkSZ0kkg)

0x00 搞清楚Fri.da和Objection的区别

表格一 frida&objection

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0ozCU6EMk1AJjRJutibPfCrA6VWjxEACVGstoKlkSLFmyZH7ciawCJG7e660eUibT0h7mD8EVngkbPMg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

表格中跨平台支持解释：Frida的跨平台支持中提到的Windows平台的使用，这里的Windows平台使用是可以通过Frida直接对Windows系统的程序进行调试，使用macOS和Windows作为例子来说有两个连接方式：1.在Windows上运行frida-server使用macOS进行连接。

运行frida-server

确保 Windows 和 macOS 在同一个网络中。

获取 Windows 的 IP 地址（在 Windows 上运行 ipconfig）。

在 macOS 上使用 Frida 客户端连接到 Windows：

frida -H <Windows IP> -n <进程名>

连接成功后，在 macOS 上编写和加载 JavaScript 脚本，对 Windows 上的目标进程进行 Hook 和调试。

2.在windows上运行frida-server并调试本地进程

运行frida-server

frida -n <进程名>

连接成功后，直接在 Windows 上编写和加载 JavaScript 脚本，对本地进程进行 Hook 和调试。

表格二

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0ozCU6EMk1AJjRJutibPfCrAqy3konPYIVujvL4LM8qb3VgEfADrvXUWbx40TWoxuIqFTaRTLRiag4w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

0x01 frida的JavaScript脚本

表格一中提到的javascript脚本，Frida可以通过javascript脚本hook应用程序的函数，拦截函数的输入参数、返回值以及执行流程。

1.hook函数

修改函数行为（如绕过验证、修改返回值）。

监控函数的调用频率和参数值。

分析加密算法、网络请求等关键逻辑。

```
`Interceptor.attach(Module.findExportByName("libc.so", "strcmp"), {onEnter: function(args) {` `console.log("strcmp called with:", args[0].readCString(), args[1].readCString());},onLeave: function(retval) {` `console.log("strcmp returned:", retval);}});`
```

2. 动态修改内存

通过 JavaScript 脚本，Frida 可以读取和修改目标进程的内存数据。

修改内存中的变量值或字符串。

绕过某些内存中的校验逻辑。

动态篡改应用的行为。

```
`var ptr = Module.findBaseAddress("libnative.so").add(0x1234);Memory.writeUtf8String(ptr, "new_value");``//Module.findBaseAddress是Frida提供的一个函数 找到libnative.so库在内存中的基地址，在基地址的基础上加上偏移量0x1234，``//将字符串"new_value"写入到指定的内存地址ptr`
```

3. 调用函数

通过 JavaScript 脚本直接调用目标应用程序中的函数

触发特定功能以测试其行为。

调用未导出的函数或私有方法。

```
`var func = new NativeFunction(Module.findExportByName("libnative.so", "my_function"), 'int', ['int']);``//查找 libnative.so 库中名为 my_function 的函数的地址。Module.findExportByName根据模块名（libnative.so）和函数名（my_function）查找该函数在内存中的地址``//'int'->my_function返回一个整数, ['int']->表示 my_function 接受一个整数参数。``var result = func(123);``//调用 my_function 并传入参数 123``console.log("Function returned:", result);`
```

  

4. 监控和拦截系统调用

Frida 可以监控目标应用程序与操作系统之间的交互，如文件读写、网络请求等。

分析应用程序的文件操作行为。

拦截网络请求，查看或修改请求数据。

```
`var openPtr = Module.findExportByName(null, "open");``//open 是标准库函数（通常由 libc.so 提供），用于打开文件``Interceptor.attach(openPtr, {` `onEnter: function(args) {` `//onEnter：在函数调用时触发。` `console.log("Opening file:", args[0].readCString());` `},` `onLeave: function(retval) {` `//onLeave：在函数返回时触发` `console.log("File opened with fd:", retval);` `}``});`
```

open函数的原型int open(const char *pathname, int flags, mode_t mode);

args[0] 是第一个参数 pathname，表示要打开的文件路径。

args[1] 是第二个参数 flags，表示打开文件的标志。

args[2] 是第三个参数 mode，表示文件的权限模式（如果创建文件）

5.动态加载和卸载模块

Frida 可以在运行时加载或卸载目标应用程序中的模块（如 SO 文件）。

分析特定模块的功能。

绕过某些模块的加载校验。

```
`var module = Process.findModuleByName("libnative.so");[/font]``console.log("Module base address:", module.base);`
```

这个功能对我来说比较抽象，具体的场景是在目标应用程序在运行时加载的某些模块（插件，广告SDK，加密库），例如一些恶意软件可能会在运行时动态加载加密模块来隐藏行为。

举例：某个程序加密库在加载之后解密数据，此时就要hook掉解密函数来获取解密后的内容

  

```
`var module = Process.findModuleByName("libcrypto.so");``if (module) {` `var decryptFunc = Module.findExportByName("libcrypto.so", "decrypt");` `Interceptor.attach(decryptFunc, {` `onEnter: function(args) {` `console.log("Decrypting data:", args[0].readCString());` `},` `onLeave: function(retval) {` `console.log("Decrypted data:", retval.readCString());` `}` `});``}`
```

  

其他功能：应用程序支持插件或扩展功能，通过动态模块来实现，就要分析插件的导出函数和调用逻辑；检测恶意行为，检测模块的加载行为，用户不知情的情况下加载恶意模块来窃取数据；加载大量模块，导致内存占用过高或启动速度变慢的情况。

6.调试和日志记录

Frida 的 JavaScript 脚本可以用于动态调试和记录应用程序的运行状态。

打印函数调用栈。

记录关键变量的值。

跟踪应用程序的执行流程。

```
`console.log("Current thread ID:", Process.getCurrentThreadId());``console.log("Backtrace:", Thread.backtrace(this.context, Backtracer.ACCURATE));`
```

7.绕过反调试和检测

Frida 的 JavaScript 脚本可以用于绕过应用程序中的反调试机制。

修改反调试标志位。

Hook 反调试函数，使其失效。

```
`var ptracePtr = Module.findExportByName(null, "ptrace");``Interceptor.replace(ptracePtr, new NativeCallback(function() {` `console.log("ptrace called, bypassing...");` `return 0;``}, 'int', []));`
```

8. 自动化测试和分析

Frida 的 JavaScript 脚本可以用于自动化测试和分析应用程序的行为。

批量测试函数的输入输出。

自动化分析应用程序的逻辑。

```
`function testFunction(input) {` `var func = new NativeFunction(Module.findExportByName("libnative.so", "my_function"), 'int', ['int']);` `var result = func(input);` `console.log("Input:", input, "Output:", result);``}``for (var i = 0; i < 10; i++) {` `testFunction(i);``}`
```

0x02 Hook脚本

先搞清楚的是frida的JavaScript脚本是指使用Frida的JavaScript API编写的脚本，实现的功能：hook函数、读取和修改内存、调用native函数、动态调试。

hook脚本是frida的javascript脚本的一个子集，专门hook目标函数：拦截函数调用、修改函数行为、监控函数调用。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0ozCU6EMk1AJjRJutibPfCrA5VebzzwVA3sFP6wHqacamdqcn8t22pkibia8434MEbCYaYicibGnLfKgrw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

hook脚本经典结构

```
`Java.perform(function() {` `// 1. 获取目标类` `var TargetClass = Java.use("com.example.app.TargetClass");` `// 2. Hook 目标方法` `TargetClass.targetMethod.implementation = function(arg1, arg2) {` `// 3. 打印日志` `console.log("targetMethod called with:", arg1, arg2);` `// 4. 调用原始方法（可选）` `var result = this.targetMethod(arg1, arg2);` `// 5. 修改返回值（可选）` `return result + 1;` `};``});`
```

1.常见用途

动态调试：打印函数参数和返回值；跟踪程序的执行流程。

绕过验证：修改函数的返回值，绕过登录验证、许可证检查等。

分析加密算法：拦截加密函数的输入和输出，分析其逻辑。

修改应用行为：替换函数的实现，改变应用的行为。

数据监控：记录敏感数据（如密码、密钥等）。

2.应用

应用验证许可证是否有效

```
`Java.perform(function() {` `var LicenseManager = Java.use("com.example.app.LicenseManager");` `LicenseManager.checkLicense.implementation = function() {` `console.log("Bypassing license check...");` `return true; // 强制返回 true` `};``});`
```

  

3.hook脚本的难点

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0ozCU6EMk1AJjRJutibPfCrACzk9k2lKRDbafBSGd2kt7lhPNmxaue224FwlN2851nveeDY0aXaTiaw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

4. 部分hook脚本

4.1.枚举已加载的类

为什么要枚举已加载的类?   已加载的类通过枚举类名，结合方法名、字段名、调用关系来定位目标，动态分析应用的行为。

目标类名被混淆，就可以先枚举已经加载的类

```
`Java.perform(function() {` `Java.enumerateLoadedClasses({onMatch: function(className) {` `//enumerateLoadedClasses获取已加载的类的名称` `console.log(className);},onComplete: function() {}});});`
```

4.2.Hook目标方法

找到目标类com.example.app.MainActivity和方法login

```
`Java.perform(function() {` `var MainActivity = Java.use("com.example.app.MainActivity");` `MainActivity.login.implementation = function(username, password) {` `console.log("Login called with:", username, password);` `// 修改返回值` `return true;` `};``});`
```

4.3.监控网络请求

应用实现方式：

1.使用系统apiandroid应用使用系统api：HttpURLConnection、OkHttp、Retrofit 等ios应用使用系统api：NSURLSession、NSURLConnection 等。

2.发送请求：构建请求（url，请求头，请求体），发送请求服务器

3.接收响应，接收服务器返回的响应数据（状态码，响应头，响应体）。

Hook思路：

目标应用使用HttpURLConnection

```
`Java.perform(function() {` `var URL = Java.use("java.net.URL");` `var HttpURLConnection = Java.use("java.net.HttpURLConnection");` `// Hook URL.openConnection` `URL.openConnection.overload().implementation = function() {` `var connection = this.openConnection();` `console.log("URL:", this.toString());` `// Hook getInputStream` `if (connection.getClass().getName().indexOf("HttpURLConnection") !== -1) {` `var httpConnection = Java.cast(connection, HttpURLConnection);` `httpConnection.getInputStream.implementation = function() {` `console.log("Request Headers:", httpConnection.getRequestProperties());` `var inputStream = this.getInputStream();` `// 读取响应数据` `var response = readInputStream(inputStream);` `console.log("Response:", response);` `return inputStream;` `};` `}` `return connection;` `};` `// 读取 InputStream 的工具函数` `function readInputStream(inputStream) {` `var BufferedReader = Java.use("java.io.BufferedReader");` `var InputStreamReader = Java.use("java.io.InputStreamReader");` `var StringBuilder = Java.use("java.lang.StringBuilder");` `var reader = BufferedReader.$new(InputStreamReader.$new(inputStream));` `var builder = StringBuilder.$new();` `var line;` `while ((line = reader.readLine()) !== null) {` `builder.append(line);` `}` `reader.close();` `return builder.toString();` `}``});`
```

  

目标应用使用 OkHttp

```
`Java.perform(function() {` `var OkHttpClient = Java.use("okhttp3.OkHttpClient");` `var Request = Java.use("okhttp3.Request");` `var Response = Java.use("okhttp3.Response");` `// Hook OkHttpClient.newCall` `OkHttpClient.newCall.implementation = function(request) {` `console.log("Request URL:", request.url().toString());` `console.log("Request Headers:", request.headers().toString());` `var response = this.newCall(request).execute();` `console.log("Response Code:", response.code());` `console.log("Response Body:", response.body().string());` `return response;` `};``});`
```

4.4.绕过 SSL Pinning

SSL pinning （SSL 证书绑定）安全机制，防止中间人攻击，核心是将服务器的SSL\TLS证书或公钥硬编码到客户端应用中，客户端与特定的服务器通信，不是与拥有有效证书的服务器通信。

引入机制：

1.证书或公钥绑定，客户端在建立连接，检查服务器返回的证书或公钥是否与硬编码的值匹配

2.防止中间人攻击，攻击者的证书与硬编码值不匹配

实现方式：

1.证书绑定：将服务器的完整证书（通常是 DER 或 PEM 格式）硬编码到客户端应用中。

2.公钥绑定：将服务器的公钥（通常是公钥的哈希值）硬编码到客户端应用中3.证书链绑定：将服务器证书链中的某个中间证书或根证书硬编码到客户端应用中。

绕过方式：

1.修改客户端代码：反编译应用代码，删掉SSL Pinning的实现代码并修改删除

2.使用frida或者objection绕过：

Frida 使用脚本Hook SSL Pinning 相关的函数；

```
`Java.perform(function() {` `var X509TrustManager = Java.use("javax.net.ssl.X509TrustManager");` `var TrustManager = Java.registerClass({` `name: "com.example.TrustManager",` `implements: [X509TrustManager],` `methods: {` `checkClientTrusted: function(chain, authType) {},` `checkServerTrusted: function(chain, authType) {},` `getAcceptedIssuers: function() {` `return [];` `}` `}` `});` `var SSLContext = Java.use("javax.net.ssl.SSLContext");` `SSLContext.init.implementation = function(keyManager, trustManager, secureRandom) {` `this.init(keyManager, [TrustManager.$new()], secureRandom);` `};``});`
```

  

使用objection命令：

```
android sslpinning disable
```

4.5.动态修改内存

```
`var ptr = Module.findBaseAddress("libnative.so").add(0x1234);``Memory.writeUtf8String(ptr, "new_value");`
```

4.6. 处理反调试

目标应用检测Frida，可以hook反调试函数

```
`Java.perform(function() {` `var System = Java.use("java.lang.System");` `System.getProperty.implementation = function(key) {` `if (key === "java.vm.name") {` `return "Dalvik"; // 伪装成 Dalvik 虚拟机` `}` `return this.getProperty(key);` `};``});`
```

  

**·** **今 日 推 荐** **·**

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0ozCU6EMk1AJjRJutibPfCrAFiboyZhfibA9Zkric4hpfuKwORay41qSHuialkicQt6Pcicibb6To4VVbR4XA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

本文内容来自网络，如有侵权请联系删除