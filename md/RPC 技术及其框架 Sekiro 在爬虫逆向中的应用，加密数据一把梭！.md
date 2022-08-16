> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIxNjQ2NzM3MA==&mid=2247484747&idx=1&sn=8fd2f3964a5e856d4769edc6b262df20&scene=21#wechat_redirect)

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/kicB09lvgibnnRjv0AAqQxyBODIttZXnQqcTPoF4Pt8tJmnia4CHaYUS3zqicFfKZTWibXTAew2ibFHDjy5Pf8nDnVEQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

点击上方「**蓝字**」关注我们

![图片](https://mmbiz.qpic.cn/mmbiz_png/kLQoJJzjYaicxneNzbOg7ynx3TfnIwmNTpJQ7orkaUNrJIV4u7PNdSJ25Mtn6XdRQTamLDDicHnYfdic2bsiaNQjCw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h8C9yp4amjgAtue9ocajsQbCb0RKR2lpNyVFrB32ib5myUmtsrdeP3NrQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

什么是 RPC  

----------

RPC，英文 RangPaCong，中文让爬虫，旨在为爬虫开路，秒杀一切，让爬虫畅通无阻！![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h8KQYxvO1sPqZaFR2F9XwoNLHxuIAWhIpC4o6sfZnKAPviaefVffZnumw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

开个玩笑，实际上 RPC 为远程过程调用，全称 Remote Procedure Call，是一种技术思想而非一种规范或协议。RPC 的诞生事实上离不开分布式的发展，RPC 主要解决了两个问题：

1.解决了分布式系统中，服务之间的互相调用问题；2.RPC 使得在远程调用时，像本地调用一样方便，让调用者感知不到远程调用的逻辑。

RPC 的存在让构建分布式系统更加容易，相比于 HTTP 协议，RPC 采用二进制字节码传输，因此也更加高效、安全。在一个典型 RPC 的使用场景中，包含了服务发现、负载、容错、网络传输、序列化等组件，完整 RPC 架构图如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/iabtD4jabia4JzgV5a872xpcP6D7FN77h8mwXZge8j4GeicL59nP49pepuIic9iaE73saNenkMC2mO8AsJc7SUNXSUQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

JSRPC  

--------

RPC 技术是非常复杂的，对于我们搞爬虫、逆向的来说，不需要完全了解，只需要知道这项技术如何在逆向中应用就行了。

RPC 在逆向中，简单来说就是将本地和浏览器，看做是服务端和客户端，二者之间通过 WebSocket 协议进行 RPC 通信，在浏览器中将加密函数暴露出来，在本地直接调用浏览器中对应的加密函数，从而得到加密结果，不必去在意函数具体的执行逻辑，也省去了扣代码、补环境等操作，可以省去大量的逆向调试时间。我们以某团网页端的登录为例来演示 RPC 在逆向中的具体使用方法。（假设你已经有一定逆向基础，了解 WebSocket 协议，纯小白可以先看看K哥以前的文章）

•主页（base64）：`aHR0cHM6Ly9wYXNzcG9ydC5tZWl0dWFuLmNvbS9hY2NvdW50L3VuaXRpdmVsb2dpbg==`•参数：h5Fingerprint

首先抓一下包，登录接口有一个超级长的参数 h5Fingerprint，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h8rpyBfTY6Zcpdas9aOMcfrVF2S5NqU6iaeOiabVkcsuzNg9R5Qd2x8gRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

直接搜一下就能找到加密函数：  

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h8ZNZZxoXdRgdXrUeszOLQTGwntTB9SYK9cOvgCw2FQz9ahgHEZtp5eg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其中 `utility.getH5fingerprint()` 传入的参数 `window.location.origin + url` 格式化后，参数如下：  

```
url = "https://passport.脱敏处理.com/account/unitivelogin"  
params = {  
    "risk_partner": "0",  
    "risk_platform": "1",  
    "risk_app": "-1",  
    "uuid": "96309b5f00ba4143b920.1644805104.1.0.0",  
    "token_id": "DNCmLoBpSbBD6leXFdqIxA",  
    "service": "www",  
    "continue": "https://www.脱敏处理.com/account/settoken?continue=https%3A%2F%2Fwww.脱敏处理.com%2F"  
}
```

uuid 和 token_id 都可以直接搜到，不是本次研究重点，这里不再细说，接下来我们使用 RPC 技术，直接调用浏览器里的 `utility.getH5fingerprint()` 方法，首先在本地编写服务端代码，使其能够一直输入待加密字符串，接收并打印加密后的字符串：

```
# ==================================  
# --*-- coding: utf-8 --*--  
# @Time    : 2022-02-14  
# @Author  : 微信公众号：K哥爬虫  
# @FileName: ws_server.py  
# @Software: PyCharm  
# ==================================  
  
  
import sys  
import asyncio  
import websockets  
  
  
async def receive_massage(websocket):  
    while True:  
        send_text = input("请输入要加密的字符串: ")  
        if send_text == "exit":  
            print("Exit, goodbye!")  
            await websocket.send(send_text)  
            await websocket.close()  
            sys.exit()  
        else:  
            await websocket.send(send_text)  
            response_text = await websocket.recv()  
            print("\n加密结果：", response_text)  
  
  
start_server = websockets.serve(receive_massage, '127.0.0.1', 5678)  # 自定义端口  
asyncio.get_event_loop().run_until_complete(start_server)  
asyncio.get_event_loop().run_forever()
```

编写浏览器客户端 JS 代码，收到消息就直接 `utility.getH5fingerprint()` 得到加密参数并发送给服务端：

```
/* ==================================  
# @Time    : 2022-02-14  
# @Author  : 微信公众号：K哥爬虫  
# @FileName: ws_client.js  
# @Software: PyCharm  
# ================================== */  
  
  
var ws = new WebSocket("ws://127.0.0.1:5678");  // 自定义端口  
  
ws.onmessage = function (evt) {  
    console.log("Received Message: " + evt.data);  
    if (evt.data == "exit") {  
        ws.close();  
    } else {  
        ws.send(utility.getH5fingerprint(evt.data))  
    }  
};
```

然后我们需要把客户端代码注入到网页中，这里方法有很多，比如抓包软件 Fiddler 替换响应、浏览器插件 ReRes 替换 JS、浏览器开发者工具 Overrides 重写功能等，也可以通过插件、油猴等注入 Hook 的方式插入，反正方法很多，对这些方法不太了解的朋友可以去看看K哥以前的文章，都有介绍。

这里我们使用浏览器开发者工具 Overrides 重写功能，将 WebSocket 客户端代码加到加密的这个 JS 文件里并 Ctrl+S 保存，这里将其写成了 IIFE 自执行方式，这样做的原因是防止污染全局变量，不用自执行方式当然也是可以的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h883Fnr02eKe1p4WwDJDtWZwhvEuJygkicTo31hAmmeDYW54dic4ODaPsA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后先运行本地服务端代码，网页上先登录一遍，网页上先登录一遍，网页上先登录一遍，重要的步骤说三遍！然后就可以在本地传入待加密字符串，获取 `utility.getH5fingerprint()` 加密后的结果了：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h84vEZy26iblh0fYlRjAHEMcw1SGhQy9AUnW2gDL0DDkgWMWYMicATcz8g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Sekiro  

---------

通过前面的示例，可以发现自己写服务端太麻烦了，不易扩展，那这方面有没有现成的轮子呢？答案是有的，这里介绍两个项目：

•JsRPC-hliang：https://github.com/jxhczhl/JsRpc•Sekiro：https://github.com/virjar/sekiro

JsRPC-hliang 是用 go 语言写的，是专门为 JS 逆向做的项目，而 Sekiro 功能更加强大，Sekiro 是由邓维佳大佬，俗称渣总，写的一个基于长链接和代码注入的 Android Private API 暴露框架，可以用在 APP 逆向、APP 数据抓取、Android 群控等场景，同样也可用于 Web JS 逆向方面，两者在 JS 逆向方面的使用方法其实都差不多，本文主要介绍一下 Sekiro 在 Web JS 逆向中的应用。

参考 Sekiro 文档，首先在本地编译项目：

•Linux & Mac：执行脚本 `build_demo_server.sh`，之后得到产出发布压缩包：`sekiro-service-demo/target/sekiro-release-demo.zip`•Windows：可以直接下载：https://oss.virjar.com/sekiro/sekiro-demo

然后在本地运行（需要有 Java 环境，自行配置）：

•Linux & Mac：`bin/sekiro.sh`•Windows：`bin/sekiro.bat`

以 Windows 为例，启动后如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h82fGAl8k3ocx8fsTbrNNKwvrsWF0OOrGqqs964iaA6LUk4cthe2xKQhw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接下来就需要在浏览器里注入代码了，需要将作者提供的 sekiro_web_client.js（下载地址：https://sekiro.virjar.com/sekiro-doc/assets/sekiro_web_client.js） 注入到浏览器环境，然后通过 SekiroClient 和 Sekiro 服务器通信，即可直接 RPC 调用浏览器内部方法，官方提供的 SekiroClient 代码样例如下：

```
function guid() {  
    function S4() {  
        return (((1+Math.random())*0x10000)|0).toString(16).substring(1);  
    }  
    return (S4()+S4()+"-"+S4()+"-"+S4()+"-"+S4()+"-"+S4()+S4()+S4());  
}  
  
var client = new SekiroClient("wss://sekiro.virjar.com/business/register?group=ws-group&clientId="+guid());  
  
client.registerAction("clientTime",function(request, resolve, reject){  
    resolve(""+new Date());  
})
```

wss 链接里，如果是免费版，要将 business 改成 business-demo，解释一下涉及到的名词：

•**group**：业务类型（接口组），每个业务一个 group，group 下面可以注册多个终端（SekiroClient），同时 group 可以挂载多个 Action；•**clientId**：指代设备，多个设备使用多个机器提供 API 服务，提供群控能力和负载均衡能力；•**SekiroClient**：服务提供者客户端，主要场景为手机/浏览器等。最终的 Sekiro 调用会转发到 SekiroClient。每个 client 需要有一个惟一的 clientId；•**registerAction**：接口，同一个 group 下面可以有多个接口，分别做不同的功能；•**resolve**：将内容传回给客户端的方法；•**request**：客户端传过来的请求，如果请求里有多个参数，可以以键值对的方式从里面提取参数然后再做处理。

说了这么多可能也不好理解，直接实战，还是以某团网页端登录为例，我们将 sekiro_web_client.js 与 SekiroClient 通信代码写在一起，然后根据需求，改写一下通信部分代码：

1.ws 链接改为：`ws://127.0.0.1:5620/business-demo/register?group=rpc-test&clientId=`，自定义 `group` 为 `rpc-test`；2.注册一个事件 `registerAction` 为 `getH5fingerprint`；3.`resolve` 返回的结果为 `utility.getH5fingerprint(request["url"])`，即加密并返回客户端传过来的 url 参数。

完整代码如下（留意末尾 SekiroClient 通信代码部分的写法）：

```
/* ==================================  
# @Time    : 2022-02-14  
# @Author  : 微信公众号：K哥爬虫  
# @FileName: sekiro.js  
# @Software: PyCharm  
# ================================== */  
  
(function () {  
    'use strict';  
    function SekiroClient(wsURL) {  
        this.wsURL = wsURL;  
        this.handlers = {};  
        this.socket = {};  
        // check  
        if (!wsURL) {  
            throw new Error('wsURL can not be empty!!')  
        }  
        this.webSocketFactory = this.resolveWebSocketFactory();  
        this.connect()  
    }  
  
    SekiroClient.prototype.resolveWebSocketFactory = function () {  
        if (typeof window === 'object') {  
            var theWebSocket = window.WebSocket ? window.WebSocket : window.MozWebSocket;  
            return function (wsURL) {  
  
                function WindowWebSocketWrapper(wsURL) {  
                    this.mSocket = new theWebSocket(wsURL);  
                }  
  
                WindowWebSocketWrapper.prototype.close = function () {  
                    this.mSocket.close();  
                };  
  
                WindowWebSocketWrapper.prototype.onmessage = function (onMessageFunction) {  
                    this.mSocket.onmessage = onMessageFunction;  
                };  
  
                WindowWebSocketWrapper.prototype.onopen = function (onOpenFunction) {  
                    this.mSocket.onopen = onOpenFunction;  
                };  
                WindowWebSocketWrapper.prototype.onclose = function (onCloseFunction) {  
                    this.mSocket.onclose = onCloseFunction;  
                };  
  
                WindowWebSocketWrapper.prototype.send = function (message) {  
                    this.mSocket.send(message);  
                };  
  
                return new WindowWebSocketWrapper(wsURL);  
            }  
        }  
        if (typeof weex === 'object') {  
            // this is weex env : https://weex.apache.org/zh/docs/modules/websockets.html  
            try {  
                console.log("test webSocket for weex");  
                var ws = weex.requireModule('webSocket');  
                console.log("find webSocket for weex:" + ws);  
                return function (wsURL) {  
                    try {  
                        ws.close();  
                    } catch (e) {  
                    }  
                    ws.WebSocket(wsURL, '');  
                    return ws;  
                }  
            } catch (e) {  
                console.log(e);  
                //ignore  
            }  
        }  
        //TODO support ReactNative  
        if (typeof WebSocket === 'object') {  
            return function (wsURL) {  
                return new theWebSocket(wsURL);  
            }  
        }  
        // weex 和 PC环境的websocket API不完全一致，所以做了抽象兼容  
        throw new Error("the js environment do not support websocket");  
    };  
  
    SekiroClient.prototype.connect = function () {  
        console.log('sekiro: begin of connect to wsURL: ' + this.wsURL);  
        var _this = this;  
        // 不check close，让  
        // if (this.socket && this.socket.readyState === 1) {  
        //     this.socket.close();  
        // }  
        try {  
            this.socket = this.webSocketFactory(this.wsURL);  
        } catch (e) {  
            console.log("sekiro: create connection failed,reconnect after 2s");  
            setTimeout(function () {  
                _this.connect()  
            }, 2000)  
        }  
  
        this.socket.onmessage(function (event) {  
            _this.handleSekiroRequest(event.data)  
        });  
  
        this.socket.onopen(function (event) {  
            console.log('sekiro: open a sekiro client connection')  
        });  
  
        this.socket.onclose(function (event) {  
            console.log('sekiro: disconnected ,reconnection after 2s');  
            setTimeout(function () {  
                _this.connect()  
            }, 2000)  
        });  
    };  
  
    SekiroClient.prototype.handleSekiroRequest = function (requestJson) {  
        console.log("receive sekiro request: " + requestJson);  
        var request = JSON.parse(requestJson);  
        var seq = request['__sekiro_seq__'];  
  
        if (!request['action']) {  
            this.sendFailed(seq, 'need request param {action}');  
            return  
        }  
        var action = request['action'];  
        if (!this.handlers[action]) {  
            this.sendFailed(seq, 'no action handler: ' + action + ' defined');  
            return  
        }  
  
        var theHandler = this.handlers[action];  
        var _this = this;  
        try {  
            theHandler(request, function (response) {  
                try {  
                    _this.sendSuccess(seq, response)  
                } catch (e) {  
                    _this.sendFailed(seq, "e:" + e);  
                }  
            }, function (errorMessage) {  
                _this.sendFailed(seq, errorMessage)  
            })  
        } catch (e) {  
            console.log("error: " + e);  
            _this.sendFailed(seq, ":" + e);  
        }  
    };  
  
    SekiroClient.prototype.sendSuccess = function (seq, response) {  
        var responseJson;  
        if (typeof response == 'string') {  
            try {  
                responseJson = JSON.parse(response);  
            } catch (e) {  
                responseJson = {};  
                responseJson['data'] = response;  
            }  
        } else if (typeof response == 'object') {  
            responseJson = response;  
        } else {  
            responseJson = {};  
            responseJson['data'] = response;  
        }  
  
  
        if (Array.isArray(responseJson)) {  
            responseJson = {  
                data: responseJson,  
                code: 0  
            }  
        }  
  
        if (responseJson['code']) {  
            responseJson['code'] = 0;  
        } else if (responseJson['status']) {  
            responseJson['status'] = 0;  
        } else {  
            responseJson['status'] = 0;  
        }  
        responseJson['__sekiro_seq__'] = seq;  
        var responseText = JSON.stringify(responseJson);  
        console.log("response :" + responseText);  
        this.socket.send(responseText);  
    };  
  
    SekiroClient.prototype.sendFailed = function (seq, errorMessage) {  
        if (typeof errorMessage != 'string') {  
            errorMessage = JSON.stringify(errorMessage);  
        }  
        var responseJson = {};  
        responseJson['message'] = errorMessage;  
        responseJson['status'] = -1;  
        responseJson['__sekiro_seq__'] = seq;  
        var responseText = JSON.stringify(responseJson);  
        console.log("sekiro: response :" + responseText);  
        this.socket.send(responseText)  
    };  
  
    SekiroClient.prototype.registerAction = function (action, handler) {  
        if (typeof action !== 'string') {  
            throw new Error("an action must be string");  
        }  
        if (typeof handler !== 'function') {  
            throw new Error("a handler must be function");  
        }  
        console.log("sekiro: register action: " + action);  
        this.handlers[action] = handler;  
        return this;  
    };  
  
    function guid() {  
        function S4() {  
            return (((1 + Math.random()) * 0x10000) | 0).toString(16).substring(1);  
        }  
  
        return (S4() + S4() + "-" + S4() + "-" + S4() + "-" + S4() + "-" + S4() + S4() + S4());  
    }  
  
    var client = new SekiroClient("ws://127.0.0.1:5620/business-demo/register?group=rpc-test&clientId=" + guid());  
  
    client.registerAction("getH5fingerprint", function (request, resolve, reject) {  
        resolve(utility.getH5fingerprint(request["url"]));  
    })  
  
})();
```

与前面的方法一样，使用浏览器开发者工具 Overrides 重写功能，将上面的代码注入到网页 JS 里：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h8RQv3VRgYYEXtDicjKXhrfL1LrbGNkGuibRhibmtLlu7pVpJ0ZaGxKichrg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后 Sekiro 为我们提供了一些 API：  

•查看分组列表：http://127.0.0.1:5620/business-demo/groupList•查看队列状态：http://127.0.0.1:5620/business-demo/clientQueue?group=test•调用转发：http://127.0.0.1:5620/business-demo/invoke?group=test&action=test&param=testparm

比如我们现在要调用 `utility.getH5fingerprint()` 加密方法该怎么办呢？很简单，代码注入到浏览器里后，首先还是要手动登录一遍，手动登录一遍，手动登录一遍，重要的事情说三遍！然后参考上面的调用转发 API 进行改写：

•我们自定义的分组 `group` 是 `rpc-test`；•事件 `action` 是 `getH5fingerprint`；•待加密参数名称为 `url`， 其值例如为：`https://www.baidu.com/`

那么我们的调用链接就应该是：http://127.0.0.1:5620/business-demo/invoke?group=rpc-test&action=getH5fingerprint&url=https://www.baidu.com/，直接浏览器打开，返回的字典，data 里面就是加密结果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h864rpqDJNn85fPqqInXJW4J95ia9oBnicSe1XWOtDwZkNgeoX75eDFqGg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

同样的，在本地用 Python 的话，直接 requests 请求调用链接就完事儿了：

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h8AtsTTflpia2sKpmzkzLAjXHrZJXbUxMSa14SCLQ1ZUtlriaPe77EKLLw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们前面是把 sekiro_web_client.js 复制下来和通信代码一起注入到浏览器的，这里我们还可以有更加优雅的方法，直接给 document 新创建一个 script，通过链接的形式插入 sekiro_web_client.js，这里需要注意一下几点问题：

1.第一个是时机的问题，需要等待 document 这些元素加载完成才能建立 SekiroClient 通信，不然调用 SekiroClient 是会报错的，这里可以用 setTimeout 方法，该方法用于在指定的毫秒数后调用函数或计算表达式，将 SekiroClient 通信代码单独封装成一个函数，比如 `function startSekiro()`，然后等待 1-2 秒后再执行 SekiroClient 通信代码；2.由于 SekiroClient 通信代码被封装成了函数，此时直接调用 `utility.getH5fingerprint` 是会提示未定义的，所以我们要先将其导为全局变量，比如 `window.getH5fingerprint = utility.getH5fingerprint`，后续直接调用 `window.getH5fingerprint` 即可。

完整代码如下所示：

```
/* ==================================  
# @Time    : 2022-02-14  
# @Author  : 微信公众号：K哥爬虫  
# @FileName: sekiro.js  
# @Software: PyCharm  
# ================================== */  
  
(function () {  
    var newElement = document.createElement("script");  
    newElement.setAttribute("type", "text/javascript");  
    newElement.setAttribute("src", "https://sekiro.virjar.com/sekiro-doc/assets/sekiro_web_client.js");  
    document.body.appendChild(newElement);  
  
    window.getH5fingerprint = utility.getH5fingerprint  
  
    function guid() {  
        function S4() {  
            return (((1 + Math.random()) * 0x10000) | 0).toString(16).substring(1);  
        }  
        return (S4() + S4() + "-" + S4() + "-" + S4() + "-" + S4() + "-" + S4() + S4() + S4());  
    }  
  
    function startSekiro() {  
        var client = new SekiroClient("ws://127.0.0.1:5620/business-demo/register?group=rpc-test&clientId=" + guid());  
  
        client.registerAction("getH5fingerprint", function (request, resolve, reject) {  
            resolve(window.getH5fingerprint(request["url"]));  
        })  
    }  
  
    setTimeout(startSekiro, 2000)  
})();
```

‍‍

![图片](https://mmbiz.qpic.cn/mmbiz_png/iabtD4jabia4JzgV5a872xpcP6D7FN77h8OyhYBnMk3o1rfywrndSv8nDZO7RAalxCvNWmqEDp0gznH9S0LTWv2g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

优缺点
---

目前如果不去逆向 JS 来实现加密参数的话，用得最多的就是自动化工具了，比如 Selenium、Puppeteer 等，很显然这些自动化工具配置繁琐、运行效率极低，而 RPC 技术不需要加载多余的资源，稳定性和效率明显都更高，RPC 不需要考虑浏览器指纹、各种环境，如果风控不严的话，高并发也是能够轻松实现的，相反，由于 RPC 是一直挂载在同一个浏览器上的，所以针对风控较严格的站点，比如检测 UA、IP 与加密参数绑定之类的，那么 PRC 调用太频繁就不太行了，当然也可以研究研究浏览器群控技术，操纵多个不同浏览器可以一定程度上缓解这个问题。

**总之 RPC 技术还是非常牛的，除了 JS 逆向，且没有针对 RPC 进行检测的话，可以说是目前比较万能、高效的方法了，一定程度上真正做到了加密参数一把梭！**

END
---

![图片](https://mmbiz.qpic.cn/mmbiz_png/oAhmb4Nz28Sdc7zWb4MN0rS20R0Zicwk0lGaYROjw9icicgib0MRxl7e3Ar4lvCIuVuFEicmRfWX29V56zk0FchbYtg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibbYNjL8QYGfMpvyjuGk3kMMaBia97r8ouDhv7ADtUzm2ZbfYibwhF2sgVnK6aWsxFcnTM0vGFQGe6icNNJsEQ60Aw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

点个

![图片](https://mmbiz.qpic.cn/mmbiz_png/RyMK9MJxOLicGkhhINvZJKBIM6hgMy9z8aPT7UIeVKTS0kMMnHOtomzmDY2EIICRcES69tEf04EC9YXDN414SqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在看

你最好看