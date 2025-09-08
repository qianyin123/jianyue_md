> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/1Te54ouKaCb-kjZIH9tX3A)

平时都用x红书下载头像、壁纸，会有水印，尝试hook一下加水印的操作看看能不能获得无水印的图。

frida脚本
-------

### 定位hook点

调研了一些常用的加水印方式，直接跑一下frida，看看哪些api用到了。

（ps:在此之前是x红书的frida检测，这里就不细说了，是常用的libmsaoaidsec.so库。

*   b站、豆瓣、爱奇艺都是用的这个，过的手段都一样hook dlopen修改pthread_creat的返回值）
    

#### 一些常用的api：

```
 _复制代码_ _隐藏代码_`1、Bitmap.compress  
  
2、FileOutputStream.write  
  
3、MediaStore.insertImage  
  
4、Canvas.drawText  
  
5、Canvas.drawBitmap`
```

跑一遍可以发现定位到了几个点

![image-20250630222358476](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHs523wDLrZN4vZNJan28Jw1yuwUkhUCtJb1iay3ocwNmuJTnTgagsC2A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

#### `Bitmap.compress`

✅ **作用：将 Bitmap 压缩并输出到文件或流中**

通常用于 **保存图片到磁盘**（例如 PNG/JPEG 格式），会被调用来将内存中的 Bitmap 转成可写入文件的数据。

#### `Canvas.drawText`

✅ **作用：在 Canvas（画布）上绘制文字**

通常用于：

*   添加**文字水印**
    
*   显示图像上的文字说明
    
*   合成图像时叠加文字
    

**那这个大概就是添加水印文字（x红书号）的api**

#### `Canvas.drawBitmap`

**✅ 作用：在 Canvas 上绘制 Bitmap 图像**

常用于：

*   把一个 Bitmap 绘制到另一个 Bitmap（比如背景 + 水印图）
    
*   图像合成处理（Logo、滤镜）
    
*   自定义控件、相机拍照回显等场景
    

**那这个大概就是添加水印图像（x红书log）的api**

### 替换操作

#### 去除文字水印

前面猜测是Canvas.drawText  api，根据调用规则进行hook

```
 _复制代码_ _隐藏代码_`drawText(String text, float x, float y, Paint paint)  
//text 是文字内容，x 和 y 是文字的坐标。`
```

通过打印text来验证是否为添加文字水印的地方，然后在返回值处将text替换为空

```
 _复制代码_ _隐藏代码_`try {  
    Canvas.drawText.overload(  
        "java.lang.String", "float", "float", "android.graphics.Paint"  
    ).implementation = function (text, x, y, paint) {  
        console.log(" Canvas.drawText() called with text: " + text);  
        console.log("`

*   `Canvas.drawText() called with x, y: " + x + "   " + y);  
            if (text.toLowerCase().includes("小红书") || text.toLowerCase().includes("xhs")) {  
                console.warn(">>>>> Suspected watermark text found!");  
            }  
            //将原有的 text（小红书号:xxxxx)替换为空  
            return this.drawText('', x, y, paint);  
        };  
    } catch (e) {  
        console.error("[!] Error hooking Canvas.drawText:", e);  
    }`


```

![image-20250625212323446](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHP4jw5iabNHwP6FlADBUoutKRApn6RKOXZiaiahd3smibw4NwGWa1VLgMzA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

成功去除了文字水印，这个api定位是对的

![image-20250630224207298](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHtoyf3pHsCEjb5RQvgia6Wcfwhv0QbjW9hWFVSpD05CLrcHONM74OicQQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

#### 去除图片logo水印

```
 _复制代码_ _隐藏代码_`drawBitmap(bitmap, float left, float top, Paint paint)  
//参数：bitmap：图片对象，left:偏移左边的位置，top： 偏移顶部的位置`
```

这部分一共有两个思路

##### 构造透明bitmap伪造水印

根据这块bitmap的大小，太大了不可能是水印

![image-20250625214547303](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHtlk0uiax3S5NGic7VibcnxvX85Eu9QOsXqvPUOVicWeoaibwYv4ZyUOk6icg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

一开始猜测的drawBitmap是将水印附加在原图上，实际操作发现，hook到的drawBitmap其实是第一步，将原图叠加到一个纯黑的背景上，之后才调用另一个api附加到这个上面（尝试过发现下载了个黑图）

继续批量测试一些bitmap的重载，**最后定位到有Rect参数的api上**

```
 _复制代码_ _隐藏代码_`// 重载1: drawBitmap(Bitmap, float, float, Paint)  
// 重载2: drawBitmap(Bitmap, Rect, Rect, Paint)  
等`
```

这里的大小就合适了，测试发现确实是它

![image-20250625215627556](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHa1AoK1uAU2sMjwiac44x8K1BqoRfpukxytQIbTibhIHMEfFpXkcKJ6Cg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

直接将bitmap转换为png下载下来可以发现，这两个rect尺寸的是水印图

![image-20250630224841145](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHE8pibvAJj6W6RVW1I2SibnpTpepMFwuxumx6K03t6jYuyavmE292mxTA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

```
 _复制代码_ _隐藏代码_``constFileOutputStream = Java.use("java.io.FileOutputStream");  
constFile = Java.use("java.io.File");  
constCompressFormat = Java.use("android.graphics.Bitmap$CompressFormat");  
constBitmap = Java.use("android.graphics.Bitmap");  
functiondumpBitmap(bitmap, name) {  
    try {  
        const file = File.$new("/sdcard/" + name + ".png");  
        const out = FileOutputStream.$new(file);  
        bitmap.compress(CompressFormat.PNG.value, 100, out);  
        out.close();  
        console.log(` ``

*   ``Bitmapdumped: ${file.getAbsolutePath()}`);  
        } catch (e) {  
            console.error("Bitmap dump failed:", e);  
        }  
    }``


```

最后就获得了没有任何水印的图片

![image-20250630225018753](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHBzwmAEHUhh03Stl94YJykOhoTwC0mHIvu35icrKtxIsTjZoqHjDyicpQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

##### hook原图并保存在指定位置

另一种方法就是，将图片

```
 _复制代码_ _隐藏代码_`Canvas.drawBitmap.overload(  
"android.graphics.Bitmap", "float", "float", "android.graphics.Paint"  
).implementation = function (bitmap, left, top, paint) {  
    const width = bitmap.getWidth();  
    const height = bitmap.getHeight();  
    dumpBitmap(bitmap, "hook1")  
    console.log("drawBitmap called: w=" + width + ", h=" + height + ", top=" + top);  
`
```

![image-20250630225208099](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHqY9OMtGB9U7QbOl0zXXQsxFTlibotj9V6aibmlia8qHOFJRkYojDausaQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

获得无水印原图

![image-20250630225220961](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHbfbnasDHUSa8w4jTRib8ejmqrVnLHlMnCIhHGX60PjuF6jN79k0m68Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

（脚本详见github）  
https://github.com/SimpsonGet/XhsWatermarkRemove

Xposed模块实现
----------

在frida实现之后就想着形成xposed模块安装在手机上，固化驻留

一开始是直接改写了frida脚本，使用`XposedHelpers.findAndHookMethod`方法来hook `drawText、drawBitmap`这两个方法，但是时机运行过程中无法hit到这两个方法，查阅资料后

```
 _复制代码_ _隐藏代码_`首先 xposed模块只能hook 目标app自身类加载器加载的类  
使用  
public void handleLoadPackage(final LoadPackageParam lpparam) throws Throwable {  
    // Hook 代码写这里  
}  
去hook方法时，能访问的是 App 的默认类加载器PthClassLoader  
而XposedHelpers.findClass("android.graphics.Canvas", lpparam.classLoader);   访问的系统类是由BootClassLoader加载的`
```

因此改为采用更加底层的`XposedBridge.hookAllMethods()`进行hook，遍历所有方法并判断目标重载方法，实现hook

```
 _复制代码_ _隐藏代码_`[#文字水印](javascript:;)  
privatevoidhookCanvasDrawText() {  
    for (Method method : Canvas.class.getDeclaredMethods()) {  
        if (method.getName().equals("drawText")  
            && method.getParameterTypes().length == 4  
            && method.getParameterTypes()[0] == String.class  
            && method.getParameterTypes()[1] == float.class  
            && method.getParameterTypes()[2] == float.class  
            && method.getParameterTypes()[3] == Paint.class) {  
  
            XposedBridge.hookMethod(method, newXC_MethodHook() {  
                @Override  
                protectedvoidbeforeHookedMethod(MethodHookParam param)throws Throwable {  
                    Stringtext= (String) param.args[0];  
                    floatx= (float) param.args[1];  
                    floaty= (float) param.args[2];  
                    XposedBridge.log(" drawText called with text: " + text + " at (" + x + ", " + y + ")");  
                    if (text != null && (text.toLowerCase().contains("小红书") || text.toLowerCase().contains("xhs"))) {  
                        param.args[0] = "";  
                        XposedBridge.log(">>> Watermark text cleared.");  
                    }  
                }  
            });  
        }  
    }  
}  
[#logo水印](javascript:;)  
privatevoidhookCanvasDrawBitmap() {  
    for (Method method : Canvas.class.getDeclaredMethods()) {  
        if (method.getName().equals("drawBitmap")  
            && method.getParameterTypes().length == 4  
            && method.getParameterTypes()[0].getName().equals("android.graphics.Bitmap")  
            && method.getParameterTypes()[1].getName().equals("android.graphics.Rect")  
            && method.getParameterTypes()[2].getName().equals("android.graphics.Rect")  
            && method.getParameterTypes()[3].getName().equals("android.graphics.Paint")) {  
  
            XposedBridge.hookMethod(method, newXC_MethodHook() {  
                @Override  
                protectedvoidbeforeHookedMethod(MethodHookParam param)throws Throwable {  
                    Bitmapbmp= (Bitmap) param.args[0];  
                    intw= bmp.getWidth();  
                    inth= bmp.getHeight();  
                    XposedBridge.log("`

*    `drawBitmap size: " + w + "x" + h);  
      
                        if (w < 400 && h < 150) {  
                            String key = w + "x" + h;  
                            Bitmap transparent = transparentCache.get(key);  
                            if (transparent == null) {  
                                transparent = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);  
                                transparentCache.put(key, transparent);  
                                XposedBridge.log("[+] Created transparent bitmap for" + key);  
                            }  
                            param.args[0] = transparent;  
                            XposedBridge.log(">>> Replaced suspected watermark bitmap");  
                        }  
                    }  
                });  
            }  
        }`


```

*   目前这两种方式判断bitmap是否为水印的方式是根据size来的，**（width<400,height<150）**，可以达到要求
    
*   在已经按照LSPosed的手机上，安装地址中的xposed模块即可
    
    https://github.com/SimpsonGet/XhsWatermarkRemove/releases/download/release-v1.0.0/xhsXposed.apk
    

![image-module](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHaFbiaZ5ib9LKv9HKdpUNRm31rFHo2bH4pMK22gAmjW7U8DiaLMeg2tXEg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=9)