> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43411585/article/details/121301297#comments_20075034)

### 目录

*   *   *   *   [一、python快的解析Protobuf方式](#pythonProtobuf_1)
            *   [二、什么是Protobuf](#Protobuf_34)
            *   [三、Protobuf环境配置](#Protobuf_50)
            *   [四、Protobuf实例序列化与反序列化](#Protobuf_69)
            *   [五、逆向解析 Protobuf案例](#_Protobuf_172)
            *   *   [1、python序列化](#1python_173)
                *   [2、python反序列化](#2python_200)

#### 一、python快的解析Protobuf方式

*   `注意：目录二、三、四、五可以作为了解，实际上目录一就可以解决了，比后面的解决方式更便捷，后面介绍的解决方式的前提是还原.proto文件。如果解决不了的话，再看二、三、四、五也可`
*   1、问题案例：请求参数和响应参数都是序列化的数据结构，不易读。两个名词`序列化`（将正常可读数据转为protobuf的数据结构，看似是乱码的形式），`非序列化`反之  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/1a6e5e3e4bf5497fbc496d8e2c0f22c7.png)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/3eb0efb017c3459095c906cf8b91526e.png)
*   2、解决方案：
    *   依赖安装：`pip install blackboxprotobuf`
    *   如果要将数据转为序列化数据，则需先确认`message_type`（Type information including field number, field name and field type），如何确认message_type，我们可以将非序列化的数据复制下来，如下赋值给a变量，然后可以得到消息类型message_type；如果下面代码无法运行，a变量的这个数据换成你自己的
        
        ```
        `import blackboxprotobuf
        a = "ï²®†Ë²®† ·Äô".encode()
        deserialize_data , message_type =blackboxprotobuf.protobuf_to_json(a)
        print(deserialize_data )  # 非序列化的原始数据
        print(message_type)  # 消息类型结构` 
        
        *   1
        *   2
        *   3
        *   4
        *   5
        
        
        ```
        
    *   将数据`序列化`样例如下，有了消息类型message_type，我们就可以将数据转为序列化数据了，如下
        
        ```
        `import blackboxprotobuf
        message_type = {'1': {'type': 'int', 'name': ''}, '2': {'type': 'int', 'name': ''}, '3': {'type': 'int', 'name': ''}, '4': {'type': 'int', 'name': ''}}
        before_data = {'1': 1, '2': 312312312442, '3': 8709708978, '4': 78971987}
        form_data = blackboxprotobuf.encode_message(before_data, message_type)
        print(form_data)` 
        
        *   1
        *   2
        *   3
        *   4
        *   5
        
        
        ```
        
    *   将数据`非序列化`样例如下，直接传入字节content，即可输出json
        
        ```
        `import blackboxprotobuf
        content = b'\x08\x9f\x19\x10\xbc)\x18\xa9) \x9f\x19(\x86#0\xc7(8\xa1"@\xfeBH\xd2:P\xcd\x03'
        print(blackboxprotobuf.protobuf_to_json(content))  
        # ('{\n  "1": "3231",\n  "2": "5308",\n  "3": "5289",\n  "4": "3231",\n  "5": "4486",\n  "6": "5191",\n  "7": "4385",\n  "8": "8574",\n  "9": "7506",\n  "10": "461"\n}', {'1': {'type': 'int', 'name': ''}, '2': {'type': 'int', 'name': ''}, '3': {'type': 'int', 'name': ''}, '4': {'type': 'int', 'name': ''}, '5': {'type': 'int', 'name': ''}, '6': {'type': 'int', 'name': ''}, '7': {'type': 'int', 'name': ''}, '8': {'type': 'int', 'name': ''}, '9': {'type': 'int', 'name': ''}, '10': {'type': 'int', 'name': ''}})` 
        
        *   1
        *   2
        *   3
        *   4
        
        
        ```
        
    *   最终解决如下：  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/02eeaf81f54844448526998c8fcf10c5.png)

#### 二、什么是Protobuf

*   1、protocol buffers介绍：是一种语言无关、平台无关、可扩展的序列化结构数据的方法。严格说不算是加密，`只能是叫序列化结构数据，让可读变为疑似的乱码，那反序列化即让疑似的乱码变为可读`
    
*   2、protobuf使用流程的`前提是有一个.proto文件`，对逆向而言就是还原`.proto文件`，选择编译成相应的编程语言包，然后调用包进行序列化和[反序列化](https://so.csdn.net/so/search?q=%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96&spm=1001.2101.3001.7020)
    
    *   比如编程语言选择python，则我们会将`.proto文件`先编译成python要用的`python模块包`(与requests等包类似)
    *   接着就是`用python调用已编译的模块包进行数据的序列化和反序列化`  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/b9b32490a38341a395a862b4744020c2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)
*   3、一个网站应用了Protobuf实例
    
    *   如下图不管是请求参数还是响应参数，都返回了序列化的数据，虽然看似是乱码，实际上只是用Protobuf将原始数据进行了序列化，变得不可读了，我们将`响应内容保存为test.bin文件`
    *   当然有的会直接在headers头指明媒体类型`content-type: application/grpc-web+proto`，也可通过此类方法判断  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/36855f9d69614ce5be57edf895d54075.png)  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/643f1356fac24beb95a95a1f6dcf9a83.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)
    *   执行cmd命令`protoc --decode_raw < test.bin`（执行该命令，需[下载解压protoc-3.19.1-win64.zip](https://github.com/protocolbuffers/protobuf/releases/)并添加bin目录为环境变量），如图结果已还原，和网站上的数据一致  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/65a0fe734c5547248eba27d433d64807.png)  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/51f2fb6b95f842238989e0f0545acb0a.png)

#### 三、Protobuf环境配置

*   1、[点击链接下载protobuf](https://github.com/protocolbuffers/protobuf/releases/)，如图下载`protobuf-python-3.19.1.zip` 和 `protoc-3.19.1-win64.zip`  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/4b8f8b0a15a349e48c8cc1504c3f48fc.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)
*   2、`protoc编译器安装`
    *   ① 解压**protoc-3.19.1-win64.zip**压缩包，并将该文件夹的bin目录**D:\Software\protoc-3.19.1-win64\bin**添加到环境变量  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/131d568c78ef49e5a8f664e2f543a0fc.png)  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/3ea1592e737c4feabf6db71a972320f6.png)
    *   ② 打开cmd输入`protoc --version`显示出版本号，则代表protoc编译器安装成功  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/84433318d1524210ac097bb17c49a147.png)
*   3、`python依赖protoc模块安装`
    *   ① 解压**protobuf-python-3.19.1.zip**压缩包，然后切换到D:\Software\protobuf-3.19.1\python目录下，打开cmd，执行如下两条命令：`python setup.py build` 和`python setup.py install`  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/896f3413386d480985af1cd2eea0f746.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/346d93c33a014feebd5d29b3efc57203.png)  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/4b71281792f04ec88cdfe847550eea41.png)
    *   ② 打开python解释器，导入`import google.protobuf` 可以检测protobuf模块是否安装成功，未报错即成功  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/e62ff1009c99448a9dc2cc300edeed0f.png)
*   4、`vscode安装vscode-proto3插件`，可以选择性安装，只是为了打开`.proto文件`好看点  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/17e21185049d4b53a6c719b40b62c720.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 四、Protobuf实例序列化与反序列化

*   [参考猿人学文档Protobuf协议逆向解析](https://www.yuanrenxue.com/app-crawl/parse-protobuf.html)
*   [可以先熟悉下Protobuf3语言指南](https://blog.csdn.net/u011518120/article/details/54604615)
*   如要传输的数据格式类似如下，然后我们用proto3语法写一个.proto文件
    
    ```
    `{
        "name": "shirmay",
        "id": 11,
        "mail": "21421312.@qq.com"
        {
            "telnumber": "133110120**",
            "type": 2  
        }
    }` 
    
    *   1
    *   2
    *   3
    *   4
    *   5
    *   6
    *   7
    *   8
    *   9
    
    
    ```
    
*   `Protobuf实例序列化与实例化的流程图介绍`，新建>编译>序列化（输出内容不可读）>反序列化（输出内容可读）

Created with Raphaël 2.3.0 1、新建yrx_example.proto文件 2.0、编译yrx_example.proto文件：执行下面cmd命令生成yrx_example_pb2.py文件 2.1、cmd命令编译文件：protoc -–python_out=. yrx_example.proto 3、python代码生成序列化文件：yrx_example_person.bin文件 4.0、python代码反序列化解析文件：将yrx_example_person.bin文件输出原始的可读结果 4.1、或者cmd命令输出.bin文件可读结果：protoc --decode_raw < yrx_example_person.bin 5、逆向还原.proto文件：已知bin文件执行如上cmd命令，根据反序列化结果过可逆向还原出yrx_example.proto文件

*   ① `新建yrx_example.proto文件`：按上数据格式用proto3语法新建yrx_example.proto文件，内容如下：如下数据有message消息类型，enum枚举类型， string字符串类型，int32整型；在消息定义中，每个字段都有唯一的一个数字标识符，也就是下面当中的1,2,3  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/4af7bca5b69841ff87561102efb2af84.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)
    
    ```
    `syntax = "proto3"; // 指定使用proto3的语法, 在一个.proto文件中可以定义多个message消息类型
    
    message Person {
    
        string name = 1;   // string类型 ：姓名, 标识符1
        int32 id = 2;      // 整型       ：id, 标识符2
        string email = 3;  // string类型 ：邮箱, 标识符3
    
        message PhoneNumber {
            string telnumber = 1;  // 电话号码
            // enum枚举类型，自定义一个PhoneType类型，每个枚举类型必须将其第一个类型映射为0（必须有有一个0值，我们可以用这个0值作为默认值）
            enum PhoneType {
                MOBILE = 0;  // 手机电话类
                HOME = 1;   // 家庭电话类
                WORK = 2;  // 工作电话类
            }
            PhoneType type = 2;
        }
    
        repeated PhoneNumber phones = 4;  // 将其他消息类型如PhoneNumber当作字段类型，如Person消息中包含PhoneNumber消息
    }
    
    message AddressBook{
        repeated Person person = 1;  // 将其他消息类型如Person当作字段类型，如希望AddressBook消息中包含Person消息
    }` 
    
    *   1
    *   2
    *   3
    *   4
    *   5
    *   6
    *   7
    *   8
    *   9
    *   10
    *   11
    *   12
    *   13
    *   14
    *   15
    *   16
    *   17
    *   18
    *   19
    *   20
    *   21
    *   22
    *   23
    *   24
    *   25
    
    
    ```
    
*   ② `编译yrx_example.proto文件`：在cmd里面输入`protoc -–python_out=. yrx_example.proto`，此时会在当前目录下生成`yrx_example_pb2.py`文件  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/fe73a3364a5a45e6bc531f166d0b2a74.png)
    
*   ③ `python代码序列化yrx_example.proto文件`，并将序列化内容存到`yrx_example_person.bin`文件中，此时数据完全不可读即看不懂
    
    ```
    `import yrx_example_pb2
    
    
    address_book = yrx_example_pb2.AddressBook()
    person = address_book.person.add()
    person.id = 11
    person.name = "shirmay"
    person.email = "110120119.@qq.com"
    
    phone = person.phones.add()
    phone.telnumber = "133110120**"
    phone.type = 2
    
    with open("yrx_example_person.bin", "wb") as f:
    	print(address_book.SerializeToString())  # b'\n/\n\x07shirmay\x10\x0b\x1a\x11110120119.@qq.com"\x0f\n\x0b133110120**\x10\x02'
        f.write(address_book.SerializeToString())` 
    
    *   1
    *   2
    *   3
    *   4
    *   5
    *   6
    *   7
    *   8
    *   9
    *   10
    *   11
    *   12
    *   13
    *   14
    *   15
    *   16
    
    
    ```
    
*   ③ `python代码反序列化yrx_example_person.bin文件`，还原为可读数据
    
    ```
    `import yrx_example_pb2
    
    
    def list_people(addr_book):
        for person in addr_book.person:
            print(f"id: {person.id}")
            print(f"name: {person.name}")
            print(f"email: {person.email}")
            for num in person.phones:
                print(f"phone_num: {num.telnumber}")
                print(f"phone type: {num.type}")
    
    
    address_book = yrx_example_pb2.AddressBook()
    with open("yrx_example_person.bin", "rb") as f:
        address_book.ParseFromString(f.read())
        list_people(address_book)` 
    
    *   1
    *   2
    *   3
    *   4
    *   5
    *   6
    *   7
    *   8
    *   9
    *   10
    *   11
    *   12
    *   13
    *   14
    *   15
    *   16
    *   17
    
    
    ```
    
*   ⑤ `逆向分析还原.proto文件`：通过上面的步骤，我们发现不管序列化和反序列化，我们首先得有`.proto文件`，只要有了`.proto`文件，我们就可以编译成python的proto模块，然后就可以正常序列化和反序列化了；`所以逆向解析Protobuf的过程就是还原.proto文件`
    
    *   已知我们有了`yrx_example_person.bin`文件，通过cmd执行命令`protoc --decode_raw < yrx_example_person.bin`即可反序列化看到左图第一版的结果，然后我们还原成中间第二版.proto文件的样子，再多次调整即可。有了`.proto文件`就可以正常的序列化和反序列化了  
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/c1a8e9c663ff4bd2999701251f61e8d5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 五、逆向解析 Protobuf案例

##### 1、python序列化

*   ① 如图，将序列化请求数据保存为`challenge_23_post.bin文件`，请求参数内容复制如下保存为.bin文件即可
    
    ```
    `õÿž†Ýÿž† Âý®` 
    
    *   1
    
    
    ```
    

![在这里插入图片描述](https://img-blog.csdnimg.cn/36855f9d69614ce5be57edf895d54075.png)

*   ② 然后输入cmd命令`protoc --decode_raw < challenge_23_post.bin`查看反序列结果  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/f17c64a583d145829d1c5fa8afdbd643.png)
*   ③ 根据反编译结果理解，直接编写`challenge_23_post.proto文件`如下  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/57f0f8b37da74dd0bfaa468c40d60c5f.png)
*   ④ 执行cmd命令`protoc -–python_out=. challenge_23_post.proto`将`challenge_23_post.proto`编译生成python包`challenge_23_post_pb2.py`  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/256652e304584a339188d06ff3b1df9b.png)
*   ⑤ 编写python代码生成序列化数据
    
    ```
    `import challenge_23_post_pb2
    
    
    post_serialize = challenge_23_post_pb2.Ms1()
    post_serialize.filed1 = 2
    post_serialize.filed2 = 219841801546160835
    post_serialize.filed3 = 219841801546157763
    post_serialize.filed4 = 36782765818179
    print(post_serialize.SerializeToString())
    with open(r"challenge_23_post.bin", "wb") as f:
        f.write(post_serialize.SerializeToString())` 
    
    *   1
    *   2
    *   3
    *   4
    *   5
    *   6
    *   7
    *   8
    *   9
    *   10
    *   11
    
    
    ```
    

##### 2、python反序列化

*   ① 如图，将序列化响应数据保存为`challenge_23_resp.bin文件`，响应内容样例复制如下保存为.bin文件即可
    
    ```
    `��0�G N(� 0�8�;@�LH�"P�"` 
    
    *   1
    
    
    ```
    
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/c2186521ab1f49699add8be0a3da0449.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   ② 然后输入cmd命令`protoc --decode_raw < challenge_23_resp.bin`查看反序列结果  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/8b9cb54b5d5d4a4fb754b3ac52930e19.png)
    
*   ③ 根据反编译结果理解，直接编写`challenge_23_resp.proto文件`如下  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/04e7326dbe07473ebe2dc866e6649d86.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAU2hyaW1heTE=,size_11,color_FFFFFF,t_70,g_se,x_16)
    
*   ④ 执行cmd命令`protoc -–python_out=. challenge_23_resp.proto`将`challenge_23_resp.proto`编译生成python包`challenge_23_resp_pb2.py`  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/9123d4daa9c34f059f554a1ab1a19f06.png)
    
*   ⑤ 编写python代码反序列化结果
    
    ```
    `import challenge_23_resp_pb2
    
    
    resp_deserialize = challenge_23_resp_pb2.Ms2()
    with open("challenge_23_resp.bin", "rb") as f:
        resp_deserialize.ParseFromString(f.read())
        count = resp_deserialize.filed1 + resp_deserialize.filed2 + resp_deserialize.filed3 + resp_deserialize.filed4 + resp_deserialize.filed5 + resp_deserialize.filed6 + resp_deserialize.filed7 + resp_deserialize.filed8 + resp_deserialize.filed9 + resp_deserialize.filed10
        print(count)  # 47662` 
    
    *   1
    *   2
    *   3
    *   4
    *   5
    *   6
    *   7
    *   8
    
    
    ```