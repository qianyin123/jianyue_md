> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-GtiFOOu22DPfOoT_j6xkA)

某短视频去年年底新出了一个字段__NS_xfalcon,在反编译的代码中可以看到sig4的字眼,其实就是sig3的升级版.

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

为什么是浅谈,一来是本身有价值,如果大家都有算法了,价值也就下降了(虽然我不卖算法).二是说的太全某些博主就会借鉴(chaoxi),注明出处倒也没什么,本来就是分享,有的不注明还以为你是原创.三是避免风险,大家都懂.

代码定位

直接搜索即可

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

选中的比较像

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

一路跟就来到了native的地方,第一个参数2f3e68b0-9a52-4f53-9109-7ea8e9dbe89a应该是签名校验的,第二个明文.

判断初始化

粗略判断方法,4个native函数全部hook上,spawn启动

```
`Java.perform(function (){` `let GNative = Java.use("com.kuaishou.security.xgs.logic.base.GNative");` `GNative["a"].implementation = function (context, str) {` ``console.log(`GNative.a is called: context=${context}, str=${str}`);`` `call2()` `call()` `let result = this["a"](context, str);` ``console.log(`GNative.a result=${result}`);`` `return result;` `};` `GNative["b"].implementation = function (str, bArr, i4) {` ``console.log(`GNative.b is called: str=${str}, bArr=${bArr}, i4=${i4}`);`` `let result = this["b"](str, bArr, i4);` ``console.log(`GNative.b result=${result}`);`` `return result;` `};` `GNative["c"].implementation = function () {` ``console.log(`GNative.c is called`);`` `let result = this["c"]();` ``console.log(`GNative.c result=${result}`);`` `return result;` `};` `GNative["d"].implementation = function () {` ``console.log(`GNative.d is called`);`` `let result = this["d"]();` ``console.log(`GNative.d result=${result}`);`` `return result;` `};``})`
```

结果

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

明显第一个函数就是,正确要返回2f3e68b0-9a52-4f53-9109-7ea8e9dbe89a|1

模拟执行

模板

```
`package com.atool;``import com.github.unidbg.AndroidEmulator;``import com.github.unidbg.Emulator;``import com.github.unidbg.Module;``import com.github.unidbg.arm.backend.Unicorn2Factory;``import com.github.unidbg.arm.context.RegisterContext;``import com.github.unidbg.debugger.BreakPointCallback;``import com.github.unidbg.debugger.Debugger;``import com.github.unidbg.file.FileResult;``import com.github.unidbg.file.IOResolver;``import com.github.unidbg.linux.android.AndroidEmulatorBuilder;``import com.github.unidbg.linux.android.AndroidResolver;``import com.github.unidbg.linux.android.dvm.*;``import com.github.unidbg.linux.android.dvm.array.ArrayObject;``import com.github.unidbg.memory.Memory;``import com.github.unidbg.pointer.UnidbgPointer;``import com.github.unidbg.utils.Inspector;``import com.github.unidbg.virtualmodule.android.AndroidModule;``import com.github.unidbg.virtualmodule.android.JniGraphics;``import com.github.unidbg.virtualmodule.android.MediaNdkModule;``import com.github.unidbg.virtualmodule.android.SystemProperties;``import keystone.Keystone;``import keystone.KeystoneArchitecture;``import keystone.KeystoneEncoded;``import keystone.KeystoneMode;``import java.io.File;``import java.io.FileNotFoundException;``import java.io.FileOutputStream;``import java.io.PrintStream;``import java.util.ArrayList;``import java.util.List;``public class demo extends AbstractJni implements IOResolver {` `private final AndroidEmulator emulator;` `private final VM vm;` `private final Module module;` `public FileResult resolve(Emulator emulator, String pathname, int oflags) {` `System.out.println("open file:" + pathname);` `return null;` `}` `demo(){` `// 创建模拟器实例` `emulator = AndroidEmulatorBuilder.for64Bit().setProcessName("").addBackendFactory(new Unicorn2Factory(false)).build();` `emulator.getSyscallHandler().addIOResolver(this);` `// 获取模拟器的内存操作接口` `final Memory memory = emulator.getMemory();` `// 设置系统类库解析` `memory.setLibraryResolver(new AndroidResolver(23));``//        memory.addModuleListener(new SearchData("9b332a80a8edcc723e9dbf64c13e24c2975451929dc2085c3a8249dd01820eaefe4d8c7b7861bf24b98819c5f46e0878", "lib.so", 1000));` `// 创建Android虚拟机,传入APK，Unidbg可以替我们做部分签名校验的工作` `vm = emulator.createDalvikVM(new File("unidbg-android/src/test/java/com//files/.apk"));` `// 4个虚拟模块` `new AndroidModule(emulator,vm).register(memory);` `new MediaNdkModule(emulator,vm).register(memory);` `new JniGraphics(emulator,vm).register(memory);` `new SystemProperties(emulator,null).register(memory);` `// 设置JNI` `vm.setJni(this);` `// 打印日志` `vm.setVerbose(true);` `// 加载目标SO` `DalvikModule dm = vm.loadLibrary("xxx", true);` `// DalvikModule dm = vm.loadLibrary(new File("unidbg-android/apks/xxx/lib.so"), true);` `//获取本SO模块的句柄,后续需要用它` `module = dm.getModule();` `// 调用JNI OnLoad` `dm.callJNI_OnLoad(emulator);` `};` `public void callByAddress(){` `// args list` `List<Object> list = new ArrayList<>(10);` `// jnienv` `list.add(vm.getJNIEnv());` `// jclazz` `list.add(0);` `// str1` `list.add(vm.addLocalObject(new StringObject(vm, "str1")));` `// strArr 假设字符串包含两个字符串` `// str6_1` `StringObject str6_1 = new StringObject(vm, "str6_1");` `vm.addLocalObject(str6_1);` `// str6_2` `StringObject str6_2 = new StringObject(vm, "str6_2");` `vm.addLocalObject(str6_2);` `ArrayObject arrayObject = new ArrayObject(str6_1,str6_2);` `list.add(vm.addLocalObject(arrayObject));` `// 最后的int` `list.add(1);` `Number number = module.callFunction(emulator, 0x2301, list.toArray());` `ArrayObject resultArr = vm.getObject(number.intValue());` `System.out.println("result:"+resultArr);` `};` `public void callByAPI(){` `DvmClass RequestCryptUtils = vm.resolveClass("com/meituan/android/payguard/RequestCryptUtils");` `StringObject str6_1 = new StringObject(vm, "str6_1");` `vm.addLocalObject(str6_1);` `StringObject str6_2 = new StringObject(vm, "str6_2");` `vm.addLocalObject(str6_2);` `ArrayObject arrayObject = new ArrayObject(str6_1,str6_2);` `ArrayObject result = RequestCryptUtils.callStaticJniMethodObject(emulator, "encryptRequestWithRandom()", "str1","str2", "str3","str4","str5",arrayObject,1);` `System.out.println(result);` `};` `public static void main(String[] args) {` `demo demo = new demo();` `//demo.HookByConsoleDebugger();` `//demo.callByAddress();` `}` `public void HookByConsoleDebugger() {` `Debugger debugger = emulator.attach();` `// debugger.addBreakPoint(module.base+0x0);``//        emulator.traceWrite(0x402e7000,0x402e7000+122);` `debugger.addBreakPoint(module.findSymbolByName("memcpy").getAddress(), new BreakPointCallback() {` `@Override` `public boolean onHit(Emulator<?> emulator, long address) {` `RegisterContext context = emulator.getContext();` `int len = context.getIntArg(2);` `UnidbgPointer pointer1 = context.getPointerArg(0);` `UnidbgPointer pointer2 = context.getPointerArg(1);` `UnidbgPointer pointer3 = context.getLRPointer();` `UnidbgPointer pointer4 = context.getPCPointer();` `Inspector.inspect(pointer2.getByteArray(0,len),"dest "+Long.toHexString(pointer1.peer)+" src "+Long.toHexString(pointer2.peer)+" lr "+Long.toHexString(pointer3.peer)+" pc "+Long.toHexString(pointer4.peer));` `return true;` `}` `});` `}` `public void trace(){` `String traceFile = "unidbg-android/src/test/java/com/demo/demo/trace/trace.txt";` `PrintStream traceStream = null;` `try{` `traceStream = new PrintStream(new FileOutputStream(traceFile), true);` `} catch (FileNotFoundException e) {` `e.printStackTrace();` `}` `//核心 trace 开启代码，也可以自己指定函数地址和偏移量` `emulator.traceCode(module.base,module.base+module.size).setRedirect(traceStream);` `}` `public void patch(){` `UnidbgPointer pointer = UnidbgPointer.pointer(emulator,module.base + 0x104CA);` `byte[] code = new byte[]{(byte) 0xa0, (byte) 0x0e,(byte) 0x00, (byte) 0x54};//直接用硬编码改原so的代码：  21230054` `pointer.write(code);` `}``}`
```

只需要改包名,apk和so路径即可,一般so用vm.loadLibrary("xxx", true);加载.xapk格式的话只能手动从外部加载,因为会找不到.

```
`public void init(){``//        emulator.traceCode(module.base, module.base+ module.size);` `// args list` `List<Object> list = new ArrayList<>(10);` `// jnienv` `list.add(vm.getJNIEnv());` `// jclazz` `list.add(0);` `DvmObject<?> context = vm.resolveClass("android/app/Application").newObject(null); // context` `list.add(vm.addLocalObject(context));` `// str1` `list.add(vm.addLocalObject(new StringObject(vm, "[{\"uuid\":\"2f3e68b0-9a52-4f53-9109-7ea8e9dbe89a\",\"version\":0,\"bits\":32,\"cdn_url\":\"\",\"file_md5\":\"\",\"vm_bc_path\":\"\"}]")));` `Number number = module.callFunction(emulator, 0x26cf8, list.toArray());` `ArrayObject resultArr = vm.getObject(number.intValue());` `System.out.println("init:"+resultArr);``}`
```

执行完返回空

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

说明环境有问题,so感知到了运行环境异常.

开启tracecode,逐指令分析.

```
emulator.traceCode(module.base, module.base+ module.size);
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

停在46dc,到so里去看看

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

plt是用于外部so跳转,这里的是libandroid中的AAssetManager_openDir,前面已经加载这个so的虚拟模块,不然会有提示缺少libandroid.so依赖的日志.

这个AndroidModule只实现了常用的6个函数

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

这里没有AAssetManager_openDir,所以跳转不过去直接就断下了.

解决办法:

1 实现这个函数,但是不是那么好实现

2 patch 这个的调用,但是会引起和它有关的其他调用.比如AAssetDir_getNextFileName和AAssetDir_close,这两个也可以实现,而且较容易,当然也可以patch.

```
`AndroidModule中添加下面两个``symbols.put("AAssetManager_openDir", svcMemory.registerSvc(is64Bit ? new Arm64Svc() {` `@Override` `public long handle(Emulator<?> emulator) {` `return openDir(emulator, vm);` `}``} : new ArmSvc() {` `@Override` `public long handle(Emulator<?> emulator) {` `return openDir(emulator, vm);` `}``}));``private static long openDir(Emulator<?> emulator, VM vm) {` `return 0;``}``然后在patch这个的调用``public void patch(){` `Debugger debugger = emulator.attach();` `debugger.addBreakPoint(0x40000000 + 0x2934c, new BreakPointCallback() {` `@Override` `public boolean onHit(Emulator<?> emulator, long address) {` `emulator.getBackend().reg_write(Arm64Const.UC_ARM64_REG_X0, 0x1); // AAssetManager_openDir` `return true;` `}` `});``}`
```

运行完还是null

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

但是在47dc了

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

这两个容易,直接实现即可

* * *

```
`symbols.put("AAssetDir_getNextFileName", svcMemory.registerSvc(is64Bit ? new Arm64Svc() {` `@Override` `public long handle(Emulator<?> emulator) {` `return getNextFileName(emulator, vm);` `}``} : new ArmSvc() {` `@Override` `public long handle(Emulator<?> emulator) {` `return getNextFileName(emulator, vm);` `}``}));``symbols.put("AAssetDir_close", svcMemory.registerSvc(is64Bit ? new Arm64Svc() {` `@Override` `public long handle(Emulator<?> emulator) {` `return AAssetDir_close(emulator, vm);` `}``} : new ArmSvc() {` `@Override` `public long handle(Emulator<?> emulator) {` `return AAssetDir_close(emulator, vm);` `}``}));``private static long getNextFileName(Emulator<?> emulator, VM vm) {` `if(emulator.getProcessName().equals("com.smile.gifmaker")){` `String newString = "2f3e68b0-9a52-4f53-9109-7ea8e9dbe89a"; // 十六进制字符串` `MemoryBlock fakeInputBlock = emulator.getMemory().malloc(newString.length(), true);` `byte[] byteArray = newString.getBytes();` `fakeInputBlock.getPointer().write(byteArray);` `return fakeInputBlock.getPointer().peer;` `}` `else {` `return 0;` `}``}``private static long AAssetDir_close(Emulator<?> emulator, VM vm) {` `return 0;``}`
```

再运行就正常了

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

调用函数

```
`public void callByAddress(){` `// args list` `List<Object> list = new ArrayList<>(10);` `// jnienv` `list.add(vm.getJNIEnv());` `// jclazz` `list.add(0);` `// str1` `list.add(vm.addLocalObject(new StringObject(vm, "2f3e68b0-9a52-4f53-9109-7ea8e9dbe89a")));` `byte[] bytes = "xxxxStr".getBytes();``//        byte[] bytes = "12345678".getBytes();` `ByteArray arr = new ByteArray(vm, bytes);` `list.add(vm.addLocalObject(arr));` `// 最后的int` `list.add(2096);` `Number number = module.callFunction(emulator, 0x27b10, list.toArray());` `ByteArray resultArr = vm.getObject(number.intValue());` `System.out.println("sig4:"+ new String(resultArr.getValue(), StandardCharsets.UTF_8));``};`
```

后面的问题补环境问题不大.就不说了

```
`@Override``public DvmObject<?> callObjectMethod(BaseVM vm, DvmObject<?> dvmObject, String signature, VarArg varArg) {` `switch (signature) {` `case "android/app/Application->getAssets()Landroid/content/res/AssetManager;":{` `return new AssetManager(vm, signature);` `}` `}` `return super.callObjectMethod(vm, dvmObject, signature, varArg);``}``@Override``public void callStaticVoidMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {` `switch (signature){` `case "com/kuaishou/security/xgs/logic/report/XLogProxy->nativeReport(IILjava/lang/String;)V":{` `return;` `}` `}` `super.callStaticVoidMethod(vm, dvmClass, signature, varArg);``}`
```

纯算法思路

用了很多常量来自初始化中的,初始化跑了1800w汇编,但是调用1次只用了50w汇编.两个思路,

1是分析初始化的时候常量怎么算的,然后推算法.

2常量别管怎么算的,直接拿来用,关注变化的量即可,会花费一些时间.

打个小广告

[安卓协议逆向开课啦!](https://mp.weixin.qq.com/s?__biz=Mzk0NzU3MjQzNA==&mid=2247485495&idx=1&sn=9040238e3969936c2218f6911b8fc393&scene=21#wechat_redirect)

[五一安卓逆向课程优惠](https://mp.weixin.qq.com/s?__biz=Mzk0NzU3MjQzNA==&mid=2247485566&idx=1&sn=bed3cb68b43bfa2683b3f38c7c2f1657&scene=21#wechat_redirect)

本人亲自录制的安卓逆向课程,感兴趣加我微信yruhua0