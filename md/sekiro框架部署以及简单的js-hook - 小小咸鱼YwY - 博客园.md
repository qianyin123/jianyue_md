> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/pythonywy/p/15637677.html)

一.环境支持
------

c++环境

java环境

maven安装

二.sekiro服务端部署
-------------

1.项目地址:[https://github.com/virjar/sekiro](https://github.com/virjar/sekiro)

[唯一官方文档](https://sekiro.virjar.com/sekiro-doc/index.html)

2.直接下载dome

[https://files.cnblogs.com/files/pythonywy/sekiro-release-demo-20210823.zip](https://files.cnblogs.com/files/pythonywy/sekiro-release-demo-20210823.zip)

三.nginx结合openssl实现https
-----------------------

### 1.nginx安装

```
1）前往用户根目录
>: cd ~

2）下载nginx1.13.7
>: wget http://nginx.org/download/nginx-1.13.7.tar.gz

3）解压安装包
>: tar -xf nginx-1.13.7.tar.gz

4）进入目标文件
>: cd nginx-1.13.7

5）配置安装路径且安装ssl插件：/usr/local/nginx
>: ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
  

6）编译并安装
>: make && sudo make install

7）建立软连接：终端命令 nginx
>: ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx

8）删除安装包与文件：
>: rm -rf nginx-1.13.7
>: rm -rf nginx-1.13.7.tar.xz

9）测试Nginx环境，服务器运行nginx，本地访问服务器ip
>: nginx
>: 服务器绑定的域名 或 ip:80 
```

### 2.openssl生成证书

#### 1.安装openssl

```
yum -y install openssl openssl-devel 
```

#### 2.生成ca秘钥

```
openssl genrsa -out local.key 2048 
```

#### 3.生成ca证书请求

```
openssl req -new -key local.key -out local.csr

#填写内容
Country Name (2 letter code) [XX]:CN  #国家
State or Province Name (full name) []:BJ   #省份
Locality Name (eg, city) [Default City]:BJ  #城市
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:test   #部门
Common Name (eg, your name or your server's hostname) []:test   #主机名
Email Address []:test@test.com  #邮箱
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:test  #密码
An optional company name []:test  #姓名 
```

#### 4.生成ca根证书

```
openssl x509 -req -in local.csr -extensions v3_ca -signkey local.key -out local.crt 
```

#### 5.根据CA证书创建server端证书

```
openssl genrsa -out my_server.key 2048 
openssl req -new -key my_server.key -out my_server.csr
openssl x509 -days 365 -req -in my_server.csr -extensions v3_req -CAkey local.key -CA local.crt -CAcreateserial -out my_server.crt 
```

#### 6.配置nginx

```
events {
    worker_connections  1024;
}

http{
upstream sekiro_business_netty {
  server 127.0.0.1:5620;
}

server {
        listen 80;
        listen       443 default  ssl;  #监听433端口
        keepalive_timeout 100;  #开启keepalive 激活keepalive长连接，减少客户端请求次数

        ssl_certificate      /root/test/local.crt;   #server端证书位置
        ssl_certificate_key  /root/test/local.key;   #server端私钥位置

        ssl_session_cache    shared:SSL:10m;         #缓存session会话
        ssl_session_timeout  10m;                    # session会话    10分钟过期

       ssl_ciphers  HIGH:!aNULL:!MD5;
       ssl_prefer_server_ciphers  on;

        server_name   test.com;
        charset utf-8;

        location /business-demo {    #后续获取加密参数时候用到的路由

          gzip on;
          gzip_min_length 1k;
          gzip_buffers 4 16k;
          gzip_http_version 1.0;
          gzip_comp_level 2;
          gzip_types application/json text/plain application/x-javascript text/css application/xml;
          gzip_vary on;

          proxy_read_timeout      500;
          proxy_connect_timeout   300;
          proxy_redirect          off;

          proxy_http_version 1.1;
          proxy_set_header    Upgrade $http_upgrade;
          proxy_set_header    Host                $http_host;
          proxy_set_header    X-Real-IP           $remote_addr;
          proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
          proxy_set_header    X-Forwarded-Proto   $scheme;

          proxy_pass http://sekiro_business_netty;
        }
  
        location /register {      #后续页面或者手机进行wss连接用到的路由
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "Upgrade";
          proxy_set_header X-Real-IP $remote_addr;
          proxy_pass http://sekiro_business_netty;
        }

        location / {                                            #这个可以不加
          client_max_body_size 0;

          proxy_read_timeout      300;
          proxy_connect_timeout   300;
          proxy_redirect          off;

          proxy_http_version 1.1;

          proxy_set_header    Host                $http_host;
          proxy_set_header    X-Real-IP           $remote_addr;
          proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
          proxy_set_header    X-Forwarded-Proto   $scheme;

          proxy_pass http://sekiro_business_netty;
        }
    }
} 
```

#### 7.开启防火墙并且启动nginx

```
firewall-cmd --add-port=5620/tcp --permanent  #端口5620防火墙打开
firewall-cmd --add-port=443/tcp --permanent   #端口443防火墙打开

firewall-cmd --reload   #防火墙重启

nginx -t  #查看nginx状态
nginx -s reload  #nginx重启 
```

四.页面js代码解读
----------

```
(function() {
    /*
  Copyright (C) 2020 virjar <virjar@virjar.com> for https://github.com/virjar/sekiro
  Redistribution and use in source and binary forms, with or without
  modification, are permitted provided that the following conditions are met:
    * Redistributions of source code must retain the above copyright
      notice, this list of conditions and the following disclaimer.
    * Redistributions in binary form must reproduce the above copyright
      notice, this list of conditions and the following disclaimer in the
      documentation and/or other materials provided with the distribution.
  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
  ARE DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
  DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
  (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
  LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
  ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
  (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
  THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
*/


function SekiroClient(wsURL) {
    this.wsURL = wsURL;
    this.handlers = {};
    this.socket = {};
    this.base64 = false;
    // check
    if (!wsURL) {
        throw new Error('wsURL can not be empty!!')
    }
    this.webSocketFactory = this.resolveWebSocketFactory();
    this.connect()
}

SekiroClient.prototype.resolveWebSocketFactory = function () {
    if (typeof window === 'object') {
        var theWebSocket = window.WebSocket ? window.WebSocket : window.MozWebSocket;
        return function (wsURL) {

            function WindowWebSocketWrapper(wsURL) {
                this.mSocket = new theWebSocket(wsURL);
            }

            WindowWebSocketWrapper.prototype.close = function () {
                this.mSocket.close();
            };

            WindowWebSocketWrapper.prototype.onmessage = function (onMessageFunction) {
                this.mSocket.onmessage = onMessageFunction;
            };

            WindowWebSocketWrapper.prototype.onopen = function (onOpenFunction) {
                this.mSocket.onopen = onOpenFunction;
            };
            WindowWebSocketWrapper.prototype.onclose = function (onCloseFunction) {
                this.mSocket.onclose = onCloseFunction;
            };

            WindowWebSocketWrapper.prototype.send = function (message) {
                this.mSocket.send(message);
            };

            return new WindowWebSocketWrapper(wsURL);
        }
    }
    if (typeof weex === 'object') {
        // this is weex env : https://weex.apache.org/zh/docs/modules/websockets.html
        try {
            console.log("test webSocket for weex");
            var ws = weex.requireModule('webSocket');
            console.log("find webSocket for weex:" + ws);
            return function (wsURL) {
                try {
                    ws.close();
                } catch (e) {
                }
                ws.WebSocket(wsURL, '');
                return ws;
            }
        } catch (e) {
            console.log(e);
            //ignore
        }
    }
    //TODO support ReactNative
    if (typeof WebSocket === 'object') {
        return function (wsURL) {
            return new theWebSocket(wsURL);
        }
    }
    // weex 鍜� PC鐜鐨剋ebsocket API涓嶅畬鍏ㄤ竴鑷达紝鎵€浠ュ仛浜嗘娊璞″吋瀹�
    throw new Error("the js environment do not support websocket");
};

SekiroClient.prototype.connect = function () {
    console.log('sekiro: begin of connect to wsURL: ' + this.wsURL);
    var _this = this;
    // 涓峜heck close锛岃
    // if (this.socket && this.socket.readyState === 1) {
    //     this.socket.close();
    // }
    try {
        this.socket = this.webSocketFactory(this.wsURL);
    } catch (e) {
        console.log("sekiro: create connection failed,reconnect after 2s");
        setTimeout(function () {
            _this.connect()
        }, 2000)
    }

    this.socket.onmessage(function (event) {
        _this.handleSekiroRequest(event.data)
    });

    this.socket.onopen(function (event) {
        console.log('sekiro: open a sekiro client connection')
    });

    this.socket.onclose(function (event) {
        console.log('sekiro: disconnected ,reconnection after 2s');
        setTimeout(function () {
            _this.connect()
        }, 2000)
    });
};

SekiroClient.prototype.handleSekiroRequest = function (requestJson) {
    console.log("receive sekiro request: " + requestJson);
    var request = JSON.parse(requestJson);
    var seq = request['__sekiro_seq__'];

    if (!request['action']) {
        this.sendFailed(seq, 'need request param {action}');
        return
    }
    var action = request['action'];
    if (!this.handlers[action]) {
        this.sendFailed(seq, 'no action handler: ' + action + ' defined');
        return
    }

    var theHandler = this.handlers[action];
    var _this = this;
    try {
        theHandler(request, function (response) {
            try {
                _this.sendSuccess(seq, response)
            } catch (e) {
                _this.sendFailed(seq, "e:" + e);
            }
        }, function (errorMessage) {
            _this.sendFailed(seq, errorMessage)
        })
    } catch (e) {
        console.log("error: " + e);
        _this.sendFailed(seq, ":" + e);
    }
};

SekiroClient.prototype.sendSuccess = function (seq, response) {
    var responseJson;
    if (typeof response == 'string' ) {
        try {
            responseJson = JSON.parse(response);
        } catch (e) {
            responseJson = {};
            responseJson['data'] = response;
        }
    } else if (typeof response == 'object') {
        responseJson = response;
    } else {
        responseJson = {};
        responseJson['data'] = response;
    }

    if (typeof response == 'string' ) {
         responseJson = {};
        responseJson['data'] = response;
    }

    if (Array.isArray(responseJson)) {
        responseJson = {
            data: responseJson,
            code: 0
        }
    }

    if (responseJson['code']) {
        responseJson['code'] = 0;
    } else if (responseJson['status']) {
        responseJson['status'] = 0;
    } else {
        responseJson['status'] = 0;
    }
    responseJson['__sekiro_seq__'] = seq;
    var responseText = JSON.stringify(responseJson);
    console.log("response :" + responseText);


    if (responseText.length < 1024 * 6) {
        this.socket.send(responseText);
        return;
    }

    if (this.base64) {
        responseText = this.base64Encode(responseText)
    }

    //澶ф姤鏂囪鍒嗘浼犺緭
    var segmentSize = 1024 * 5;
    var i = 0, totalFrameIndex = Math.floor(responseText.length / segmentSize) + 1;

    for (; i < totalFrameIndex; i++) {
        var frameData = JSON.stringify({
                __sekiro_frame_total: totalFrameIndex,
                __sekiro_index: i,
                __sekiro_seq__: seq,
                __sekiro_base64: this.base64,
                __sekiro_is_frame: true,
                __sekiro_content: responseText.substring(i * segmentSize, (i + 1) * segmentSize)
            }
        );
        console.log("frame: " + frameData);
        this.socket.send(frameData);
    }
};

SekiroClient.prototype.sendFailed = function (seq, errorMessage) {
    if (typeof errorMessage != 'string') {
        errorMessage = JSON.stringify(errorMessage);
    }
    var responseJson = {};
    responseJson['message'] = errorMessage;
    responseJson['status'] = -1;
    responseJson['__sekiro_seq__'] = seq;
    var responseText = JSON.stringify(responseJson);
    console.log("sekiro: response :" + responseText);
    this.socket.send(responseText)
};

SekiroClient.prototype.registerAction = function (action, handler) {
    if (typeof action !== 'string') {
        throw new Error("an action must be string");
    }
    if (typeof handler !== 'function') {
        throw new Error("a handler must be function");
    }
    console.log("sekiro: register action: " + action);
    this.handlers[action] = handler;
    return this;
};

SekiroClient.prototype.encodeWithBase64 = function () {
    this.base64 = arguments && arguments.length > 0 && arguments[0];
};

SekiroClient.prototype.base64Encode = function (s) {
    if (arguments.length !== 1) {
        throw "SyntaxError: exactly one argument required";
    }

    s = String(s);
    if (s.length === 0) {
        return s;
    }

    function _get_chars(ch, y) {
        if (ch < 0x80) y.push(ch);
        else if (ch < 0x800) {
            y.push(0xc0 + ((ch >> 6) & 0x1f));
            y.push(0x80 + (ch & 0x3f));
        } else {
            y.push(0xe0 + ((ch >> 12) & 0xf));
            y.push(0x80 + ((ch >> 6) & 0x3f));
            y.push(0x80 + (ch & 0x3f));
        }
    }

    var _PADCHAR = "=",
        _ALPHA = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/",
        _VERSION = "1.1";//Mr. Ruan fix to 1.1 to support asian char(utf8)

    //s = _encode_utf8(s);
    var i,
        b10,
        y = [],
        x = [],
        len = s.length;
    i = 0;
    while (i < len) {
        _get_chars(s.charCodeAt(i), y);
        while (y.length >= 3) {
            var ch1 = y.shift();
            var ch2 = y.shift();
            var ch3 = y.shift();
            b10 = (ch1 << 16) | (ch2 << 8) | ch3;
            x.push(_ALPHA.charAt(b10 >> 18));
            x.push(_ALPHA.charAt((b10 >> 12) & 0x3F));
            x.push(_ALPHA.charAt((b10 >> 6) & 0x3f));
            x.push(_ALPHA.charAt(b10 & 0x3f));
        }
        i++;
    }


    switch (y.length) {
        case 1:
            var ch = y.shift();
            b10 = ch << 16;
            x.push(_ALPHA.charAt(b10 >> 18) + _ALPHA.charAt((b10 >> 12) & 0x3F) + _PADCHAR + _PADCHAR);
            break;

        case 2:
            var ch1 = y.shift();
            var ch2 = y.shift();
            b10 = (ch1 << 16) | (ch2 << 8);
            x.push(_ALPHA.charAt(b10 >> 18) + _ALPHA.charAt((b10 >> 12) & 0x3F) + _ALPHA.charAt((b10 >> 6) & 0x3f) + _PADCHAR);
            break;
    }

    return x.join("");
};
function guid() {
    function S4() {
        return (((1 + Math.random()) * 0x10000) | 0).toString(16).substring(1);
    }

    return (S4() + S4() + "-" + S4() + "-" + S4() + "-" + S4() + "-" + S4() + S4() + S4());
}




 //上面固定写法
//####################################################分割线####################################################
//下面主要调用逻辑


function get_sign(token){
    var s = ['nonceStr',"aid=24&app_]
    var sign_1 = window.byted_acrawler.sign(s)
    return sign_1
}
  

//其中register对应你nignx配置的参数可以自由发挥
var client = new SekiroClient("wss://你主机的地址:443/register?group=JRTT&clientId=" + guid());
client.registerAction("test", function (request, resolve, reject) {
    console.log(request)
    console.log(request["token"])
    var token  = request["token"]
    var sign = get_sign(token)
    resolve(sign)
});
})(); 
```

五.调用加密参数
--------

```
直接get请求即刻,对应参数自己对着看

https://你主机的地址/business-demo/invoke?group=JRTT&action=test&url=MS4wLjABAAAAlzL5r88XAOMbJ57O-_0GuqznNRQJzgCDNuDNxZ_ACA4 
```