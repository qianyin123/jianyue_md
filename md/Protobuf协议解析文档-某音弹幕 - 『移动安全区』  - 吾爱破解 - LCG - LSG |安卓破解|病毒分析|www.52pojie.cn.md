> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1271783-1-1.html)

### 0x00 : 前言

   近期因为工作原因着手分析某音协议,在做到直播间这步的时候通过抓包发现其直播间内弹幕数据为protobuf协议,之前用xposed做wx的时候虽然接触过,但在下从来都是只要结果的人,直接调用wx内部函数转成对象即可,谁理你什么格式,但现在做的既然是协议,那自然是无端可用,什么东西都要自己处理,这里分享一下记录的分析流程.

### 0x01 : 准备工作

   在开始分析之前需要准备好:  
      1.protobuf(安装方法百度即可);  
      2.直播间弹幕抓包;  
      3.通过protoc工具测试:将数据包里返回的弹幕内容复制到文本,通过指令  
      protoc --decode_raw < xxx.bin 查看解析后的效果:

![](https://attach.52pojie.cn/forum/202009/21/193101cf8oj35id5m2g2iy.jpg)

**1.jpg** _(223.93 KB, 下载次数: 2)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE1OXxmNzc2YjVhN3wxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

可以被工具正常解析,到这里准备工作就完成了, 接下来本文档以图中数据为例,进入分析流程;  
 

### 0x02 : 分析流程

   上面通过protoc解析出了个大概,那么里面的数据究竟是什么意思呢? 1 2 3, 1: 2:这些都是什么呢? 网上所搜一番得知:解析出来的数据中开头的1 2 3..或 1: 2: 3:的均为protobuf的tag, 是解析数据的关键值,带":"号的是字段,没":"号的是 repeated 集合.  
   那么知道了这些,要怎么还原数据呢,逆向中是没有原始.proto文件的,对于protobuf的解析就需要依靠分析源码,自己生成一份.proto文件,以上图为例,在源码中定位到解析代码:

```
 _复制代码_ _隐藏代码_`private ProtoApiResult a(b bVar) throws Exception {
        PatchProxyResult proxy = PatchProxy.proxy(new Object[]{bVar}, this, a, false, 22172);
        if (proxy.isSupported) {
            return (ProtoApiResult) proxy.result;
        }
        com.bytedance.android.tools.pbadapter.a.b protoDecoder = ((INetworkService) d.a(INetworkService.class)).getProtoDecoder(com.bytedance.android.livesdkapi.message.f.class);
        if (protoDecoder != null) {
            com.bytedance.android.livesdkapi.message.f fVar = (com.bytedance.android.livesdkapi.message.f) protoDecoder.decode(this.s.a(bVar));
            ProtoApiResult protoApiResult = new ProtoApiResult();
            protoApiResult.cursor = fVar.b;
            protoApiResult.fetchInterval = fVar.c;
            protoApiResult.now = fVar.d;
            protoApiResult.messages = new LinkedList();
            this.m = fVar.e;
            long currentTimeMillis = System.currentTimeMillis();
            this.n = currentTimeMillis - this.k;
            ag.b = (fVar.d + ((currentTimeMillis - this.k) / 2)) - currentTimeMillis;
            if (Lists.isEmpty(fVar.a)) {
                return protoApiResult;
            }
            //省略......` 
```

其中参数(b bVar)权当作http请求后的body即可,无关紧要,重点是com.bytedance.android.livesdkapi.message.f 这个类,进去看一下类里的结构:

```
 _复制代码_ _隐藏代码_`package com.bytedance.android.livesdkapi.message;
public class f {
    @SerializedName("messages")
    public List<a> a;
    @SerializedName("cursor")
    public String b;
    @SerializedName("fetch_interval")
    public long c;
    @SerializedName("now")
    public long d;
    @SerializedName("internal_ext")
    public String e;
    public static final class a {
        @SerializedName("method")   //这个实际上是类名,根据这个字段获取对应解析的类.
        public String a;
        @SerializedName("payload")  //解析的数据内容,同样是protobuf.
        public byte[] b;
    }
}`
```

对比一下解析出来的数据图,发现对应tag:2,3,4,5

![](https://attach.52pojie.cn/forum/202009/21/193103lpd8bdt7lcnb67zp.png)

**2.png** _(81.7 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE2MHxjMjA3YmQ1MnwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

tag1是个数组,类型是内部类a, 内部类中剩下的2个字段是用于动态定位接下来解析的类:  
method => 解析剩下数据包中的类名, payload => 数据内容,method 也就是这部分:

![](https://attach.52pojie.cn/forum/202009/21/193105l4s6y49j1aymd6fc.jpg)

**3.jpg** _(60.57 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE2MXxiMDQ5Zjk1ZnwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

分析到此数据结构有了大致了解,com.bytedance.android.livesdkapi.message.f是某音弹幕数据的最外层,用来保存直播间弹幕所有的消息类型,从格式上不难理解,会有多种method,这里先以"WebcastRoomMessage"为例查找对应关系:  
在代码里搜索一下很快找到了 com.bytedance.android.livesdkapi.depend.g.a:

![](https://attach.52pojie.cn/forum/202009/21/193107khl4obbxeexo4l1e.png)

**4.png** _(30.52 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE2MnxiZGU2MDcxMHwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

找到对这个枚举的引用,定位到 com.bytedance.android.livesdk.chatroom.bl.a:

![](https://attach.52pojie.cn/forum/202009/21/193109jsyk8q3y283qheh9.png)

**5.png** _(47.39 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE2M3w0MTcxNTFlZHwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

对应关系找到,说明剩下byte[]是在 cl.class这个类中解析,看一下里面的结构:

```
 _复制代码_ _隐藏代码_`public class cl extends d {             //注意继承关系
    public static ChangeQuickRedirect a;
    @SerializedName("content")
    public String b;
    @SerializedName("supprot_landscape")
    public boolean c;
    public cl() {
        this.type = com.bytedance.android.livesdkapi.depend.g.a.ROOM;   <-枚举类型
    }`
```

"WebcastRoomMessage"中只有2个字段,"content"和"supprot_landscape",可是根据工具解析出来的数据可以看到,还有一大坨呢,剩下的数据在哪,并且这2个的tag排列关系又是什么? 那只能继续查找cl这个类是怎么解析的,经过一番查找,找到了解析对应关系,com.bytedance.android.live.base.model.proto.d ,在这里可以看到负责解析cl.class的类是 gw.class:

![](https://attach.52pojie.cn/forum/202009/21/193111d9otwi6olyb95065.png)

**6.png** _(34.21 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE2NHw1ZmUyZTllZHwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

进入gw.class:

![](https://attach.52pojie.cn/forum/202009/21/193113mswt8z88wtwtw6nm.png)

**7.png** _(65.15 KB, 下载次数: 1)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE2NXwzODBlOTdiZnwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

在这里知道了"content"和"supprot_landscape"分别对应tag 2和3,还有第三个名为"baseMessage"的字段要解析,这个是cl.class的基类成员,通过多个数据包验证得知,所有method中,都继承同一个基类:

![](https://attach.52pojie.cn/forum/202009/21/193115bjpo7novsde7dvb2.png)

**8.png** _(27.28 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE2NnxkMDFkOTg5Y3wxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

之前没解析出来的数据也在其中,看下这个名为baseMessage成员的内部结构,并找一下"tag表":

![](https://attach.52pojie.cn/forum/202009/21/193117xee9bbqzj4wqqe9t.png)

**9.png** _(142.87 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE2N3w0YTI2ZTZiY3wxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

分析到此,弹幕结构渐渐清晰了, 如下图:

![](https://attach.52pojie.cn/forum/202009/21/193120nkpsp66bsrsbj6ah.png)

**10.png** _(124.12 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE2OHw5MmVkZmE4NHwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

现在可以根据分析结果开始写proto文件了.

### 0x03 : proto文件编写

   关于proto文件语法网上资料很多,这里先说几个关键的,proto语法有"proto2"和"proto3",我没看出来对于咱们逆向人员选择"proto3"有什么用处,并且3的语法是不需要字段修饰符的,这只会让我们在写proto文件的时候更加迷糊,所以我们选择"proto2"来写.  
   或许你已经发现,基类(baseMessage)有很多字段,怎么解析出来的就"method","msg_id","room_id","create_time"这么几个,其实这也是很正常的情况,并不是每一个字段都是必选的.proto有3个字段修饰符required(必选),optional(可选),repeated(重复),说白了就是咱们的proto文件不使用required,只用optional,因为咱们也不知道哪个字段什么时候才有值,如果是数组一律用repeated修饰.  
   注意proto的数据类型,以java为例.String => string ; int => int32; long => uint64;...更多类型可自行查找资料.  
开始吧,先写一份 ProtoApiResult.proto,对应上面的  
com.bytedance.android.livesdkapi.message.f ,用来保存弹幕整体数据:

![](https://attach.52pojie.cn/forum/202009/21/193122auviqxvvxuuiennd.png)

**11.png** _(162.41 KB, 下载次数: 1)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE2OXxiZTcxMmY5OXwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

外层有了,现在写"WebcastRoomMessage"对应的类:

![](https://attach.52pojie.cn/forum/202009/21/193124w56i6bjnaf9f1f6n.png)

**12.png** _(81.96 KB, 下载次数: 1)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE3MHw1ZGNmYTNmMnwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

接下来最后一个"CommonMessageData":

![](https://attach.52pojie.cn/forum/202009/21/193126gwpa2c3p2p2ccc4s.png)

**13.png** _(158.89 KB, 下载次数: 1)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE3MXw4YmY4ZGYwNXwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

注意这里的display_text字段,这还是一个对象,里面又是N多字段,这里我已经解析过了, 大家自己在分析过程中如果不想解析那么多,可以删除不需要的字段,或者将类型定义为bytes即可.只要tag不乱就行.  
   OK,将.proto打包成java文件:  protoc --java_out=out .*.proto  
之后在out目录就生成了可以使用的java文件, 导入到项目中即可,最后看下解码效果:

![](https://attach.52pojie.cn/forum/202009/21/193128xls3ugocrsn3hadu.png)

**14.png** _(100.76 KB, 下载次数: 1)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE3MnwxMTUzZTMyN3wxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

   嗯,费了一大堆事最后只解码一个系统提示...,这也没办法,某音一次请求动辄数万字节的返回数据,用那些做例子的话,我这图都没法截,迫不得已找了个最小的,根据上述流程其实其他类型的数据解析都是一样的,没什么区别,最后上个最终演示效果:

![](https://attach.52pojie.cn/forum/202009/21/193131a0l4la0ql0advvlq.png)

**15.png** _(189.75 KB, 下载次数: 2)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE3M3w0ZmNjZmRmZXwxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

![](https://attach.52pojie.cn/forum/202009/21/193133ry7ey76polwpnvwe.png)

**16.png** _(204.95 KB, 下载次数: 2)_

[下载附件](forum.php?mod=attachment&aid=MjA4MTE3NHwwMjllY2VmN3wxNjY5MTg0MzE0fDB8MTI3MTc4Mw%3D%3D&nothumb=yes)

2020-9-21 19:31 上传

首次发帖,多多关照.