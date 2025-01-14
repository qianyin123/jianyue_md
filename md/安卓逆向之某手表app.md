> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/VEzxZ_ZJMiO4raqbUf0wAA)

抓包
==

该案例中的入参和出参的结果都是加密的，且每次iv都不一样。

查看表款详情

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Oy4soMTIUbcz40kjM9Hg2ksr22NHIRzYMSFUbZXOV9Q2z2HHjtxfODA6iaYGYlKEPPAeTLrpjYu6xsia1e1VlJ3A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Hook IV
=======

```
adb connect 172.20.107.41  打开wifi ADB拿到手机的ip  
adb shell  && ./data/local/tmp/fs1280arm64  启动frida server  
frida-ps -Ua  客户端查看包名com.timez.TimeZ  
frida -UF com.timez.TimeZ -l tu.js -o time.txt  开始自吐
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

客户端调用了三个接口，搜索`2a652c4dd97cda2f`

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

根据以上信息可以尝试解密

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

同样的入参也通过解密

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

确认详情接口是"watch/byBrefV2"，搜索jadx

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

尝试使用objection内存漫游，确实触发了zg.c.z0方法

```
objection -g com.timez.TimeZ explore   
android hooking list classes  
android hooking list class_methods zg.c   
android hooking watch class_method zg.c.z0 --dump-args --dump-backtrace --dump-return  

```

不过依旧不知道发起请求时iv是如何生成的，data是如何生成的。。。

该结构是个map结构，尝试hook HashMap

```
function main() {  
    Java.perform(function () {  
        var HashMap = Java.use("java.util.HashMap");  
      
        HashMap.put.implementation = function(key,value){  
            if(key == "iv"){ // key  
                console.log(key,value);  
                console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));  
            }  
              
            var res = this.put(key,value);  
              
            return res;  
        }  
      
    });  
}  
  
setImmediate(main)  
  

```

hook发现请求的iv对应调用栈路径fj.f.b->k9.b.intercept，不过intercept却是一个接口中的方法

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

反编译的结果不全，使用gda打开，最终赋值给str5,`hashMap.put(str5, sg);`,而`sg = a.b();`

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

查看a.b()，即iv的生成方式

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

改成frida主动调用

```
function getiv() {  
    // hook 对比charles抓包验证  
    // Java.perform(function () {  
  
    //     Java.use("com.timez.core.data.repo.security.a").b.implementation = function () {  
    //         var result = this.b();  
    //         console.log("result",JSON.stringify(result));  
    //         return result;  
    //     };  
    // })  
    var res;  
    Java.perform(function () {  
        res = Java.use("com.timez.core.data.repo.security.a").b()  
    })  
    return res  
}  
rpc.exports = {  
    getiv: getiv  
}  

```

timez.py

```
# 主动调用  
import time  
import frida  
from flask import Flask, jsonify, request  
import json  
def my_message_handler(message , payload): #定义错误处理  
 print(message)  
 print(payload)  
#device = frida.get_device_manager().add_remote_device("172.20.107.41:5555")  
device = frida.get_usb_device()  
# 启动`timez`这个app  
pid = device.spawn(["com.timez.TimeZ"]) # pid = device.get_frontmost_application().pid  
device.resume(pid)  
time.sleep(1)  
session = device.attach(pid)  
# 加载脚本  
with open("timez.js") as f:  
    script = session.create_script(f.read())  
script.on("message" , my_message_handler) #调用错误处理  
script.load()  
  
print(script.exports.getiv()) 
```

不过通过生成的iv和aes加密后的数据直接请求时返回数据为空，带上请求头又能拿到数据，过一段时间后数据又返回空，那么请求头中一定有关于时间的参数，即user-agent

```
headers = {  
    'Host': 'api3.timez.com',  
    'user-agent': '1.1 android 1736835701 11499 e5e08dd4461b7a0e7aa0c23bf4209b73 zh-CN',  
    'app-channel': 'Official',  
    'content-type': 'application/x-www-form-urlencoded',  
}  

```

HOOK user-agent
===============

首先我想到的是hook System.currentTimeMillis和hook 字符串拼接

```
function main() {  
    // hook  
    Java.perform(function () {  
  
        Java.use("java.lang.System").currentTimeMillis.implementation = function () {  
  
        console.log(Java.use('android.util.Log').getStackTraceString(Java.use('java.lang.Throwable').$new()));  
  
        var result = this.currentTimeMillis();  
        console.log("result:",result)  
        return result;  
        };  
          
        var StringBuilder = Java.use('java.lang.StringBuilder');  
        StringBuilder.append.overload('java.lang.String').implementation = function (str) {  
  
            console.log("Appending: " + str);  
  
            console.log(Java.use('android.util.Log').getStackTraceString(Java.use('java.lang.Throwable').$new()));  
  
            var result = this.append(str);  
  
            console.log("Result: " + result.toString());  
            return result;  
        };  
   
    })  
}  
setImmediate(main)  

```

全部定位到k9.b.intercept中

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

```
sg: 1.1  
str3: android  
vl: System.currentTimeMillis()/(long)1000;  
vi5: new Random().nextInt(0x00015f8f)+10000;  
c.s0(str1.append(vi5).toString()  
str1: e3ec6b7508ed0c83432a72651b103367  

```

hook zg.c.s0  生成的user-agent和抓包中的一致，验证成功

```
function main() {  
    // hook   
        Java.use("zg.c").s0.implementation = function (str) {  
            console.log("param: " + str);  
            var result = this.s0(str)  
            console.log("result: "+result);  
            return result  
        }  
    })  
}
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

看着像是md5，hook 入参并验证正确

完整代码
====

```
def aes_encrypt(data, iv):  
    key = '6b992acc0f965b82b84b1673abf8aaf1'  
    cipher = AES.new(key.encode('utf8'), AES.MODE_CBC, iv.encode('utf8'))  
    # 对明文进行填充，确保其长度是 AES 块大小（16 字节）的倍数  
    block_size = AES.block_size  
    padding_length = block_size - len(data.encode('utf8')) % block_size  
    padded_data = data.encode('utf8') + bytes([padding_length] * padding_length)  
    encrypted_data = cipher.encrypt(padded_data)  
    # 将加密结果进行 Base64 编码  
    encoded_data = base64.b64encode(encrypted_data)  
    return encoded_data.decode('utf8')  
  
  
def aes_decrypt(data, iv):  
    key = '6b992acc0f965b82b84b1673abf8aaf1'  
    cipher = AES.new(key.encode('utf8'), AES.MODE_CBC, iv.encode('utf8'))  
    # 先对 Base64 编码的数据进行解码  
    decoded_data = base64.b64decode(data)  
    # 使用 cipher 进行解密操作  
    decrypted_data = cipher.decrypt(decoded_data)  
    # 去除填充  
    padding_length = decrypted_data[-1]  
    original_data = decrypted_data[:-padding_length]  
    return original_data.decode('utf8')  
  
  
def md5_hashlib(input_string):  
    """  
    使用 hashlib 库进行 MD5 加密  
    :param input_string: 输入的字符串  
    :return: MD5 加密后的结果（十六进制字符串）  
    """  
    m = hashlib.md5()  
    m.update(input_string.encode('utf-8'))  
    return m.hexdigest()  
  
  
def generate_random_number():  
    # 创建一个随机数生成器对象  
    rand = random.Random()  
    # 生成一个在 0 到 0x00015f8f 之间的随机整数  
    random_number = rand.randint(0, 0x00015f8f)  
    # 加上 10000  
    result = random_number + 10000  
    return result  
  
  
def get_ua():  
    key = 'e3ec6b7508ed0c83432a72651b103367'  
    rd = str(generate_random_number())  
    yan = md5_hashlib(key + rd)  
    ts = int(time.time())  
    lst_yan = md5_hashlib('1.1' + 'android' + str(ts) + rd + yan + key)  
    return " ".join(i for i in ['1.1', 'android', str(ts), rd, lst_yan, "zh-CN"])  
  
  
def get_header():  
    return {  
        'Host': 'api3.timez.com',  
        'user-agent': get_ua(),  
        'app-channel': 'Official',  
        'content-type': 'application/x-www-form-urlencoded',  
    }  
  
def get_iv():  
    sstr = ""  
    for _ in range(16):  
        sstr += "abcdefghijklmnopqrstuvwxyz0123456789"[random.randint(0, 35)]  
    return sstr  
  
headers = {  
    'Host': 'api3.timez.com',  
    'user-agent': get_ua(),  
    'app-channel': 'Official',  
    'content-type': 'application/x-www-form-urlencoded',  
}  
iv = get_iv()  
d = {"area":"cny","_refer":"/brand/detail","refer":"{\"page\":\"brand\"}","_v":"3.1.0","bref":"a-lange-sohne-139032","bbq":9}  
data = {  
    'data': aes_encrypt(str(json.dumps(d)),iv),  
    'iv': iv,  
    'device': '5cce39599df9226a1e4feeb4b2e71b63',  
}  
  
response = requests.post('https://api3.timez.com/watch/byBrefV2', headers=headers, data=data)  
print(aes_decrypt(response.json()['data'],response.json()['iv']))  

```

后续
==

现在的逻辑是获取品牌->型号->详细信息，在获取型号的接口中`watch/byBrand`，2次分页后会产生脏数据，即同样的型号数据多次返回，抓包验证确实反复产生相同的数据。思路：从网页端拿到型号后，通过app端的接口进行抓取数据，实现获取网页端的参数(以图片形式)。