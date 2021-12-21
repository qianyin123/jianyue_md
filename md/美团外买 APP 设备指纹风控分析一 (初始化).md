> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ef4GwJrS2-tlH5Yz2mbVmQ)

```
Lcom/meituan/android/common/mtguard/MTGuard;->loadSo(Ljava/lang/String;)V
System.loadLibrary("mtguard");
//注册Native方法
private static native Object[] main(int arg0, Object[] arg1)//arg0:传入不同的编号走不同的逻辑，arg1:参数
```

### 一、电商类 APP 业务风险类型

电商行业的各个业务场景面临不同的风险种类：客户端漏洞利用、协议逆向、注册小号、商品信息被抓取、推广渠道作弊、营销活动被薅羊毛、商品秒杀等。  
大多的防御方案是通过端上安全、链路安全、接口和数据传输安全保护，再借助设备安全核验技术、人机识别及时发现各种模拟行为和异操作风险、同时集合风控策略实现多节点防护。  

### 二、设备指纹在业务中的应用

设备指纹技术是使用更多的信息来完成对终端设备的唯一性识别，在业务中可以有效辨别设备是真实用户还是机器在注册、登录，及时检测出单设备登入多帐号、防止批量注册、登录等操作行为。

### 三、整体框架

因为框架流程过于复杂，我将框架分为两个部分，一是初始化，二是设备指纹，这样会更清楚些，如图 3-1 与 3-2 所示：

![](https://mmbiz.qpic.cn/mmbiz_png/N0ibNguSP6ibU4rRxiclW8aJSibKJWX0D5Uo6dEPkdiacWHPpia0iaobQlXXoQsZ2j9OSMLAQ4lQhohaPlsa4KZzQyT1A/640?wx_fmt=png)

                                图 3-1

![](https://mmbiz.qpic.cn/mmbiz_png/N0ibNguSP6ibU4rRxiclW8aJSibKJWX0D5UogNib9vyTFicgLjOjUHrKYAKglDj5gWDPicf6BoyMpaNo6iaibiag0UTSDdww/640?wx_fmt=png)

                                图 3-2

### 四、初始化流程分析

#### 4.1、初始化准备

java 层调用 init() 初始化，获取 Context，包名，AppInfo，XML 配置信息等，然后加载 so libmtguard.so 在 so 中注册一个 Native 方法，该 Native 方法传入不同的数字代表不同的功能，代码如下所示：

```
.text:B1BF744E 01 26       MOVS            R6, #1
.text:B1BF7450 2E 70       STRB            R6, [R5]
.text:B1BF7452 20 48       LDR             R0, =(aSystemBinLs - 0xB1BF7458) ; "/system/bin/ls"
.text:B1BF7454 78 44       ADD             R0, PC                  ; "/system/bin/ls" ; file
.text:B1BF7456 00 25       MOVS            R5, #0
.text:B1BF7458 29 00       MOVS            R1, R5                  ; oflag
.text:B1BF745A CF F7 A0 EC BLX             open
.text:B1BF745E 04 00       MOVS            R4, R0
.text:B1BF7460 00 2C       CMP             R4, #0
.text:B1BF7462 25 DB       BLT             loc_B1BF74B0
.text:B1BF7464 01 AD       ADD             R5, SP, #0x28+buf
.text:B1BF7466 14 22       MOVS            R2, #0x14               ; nbytes
.text:B1BF7468 20 00       MOVS            R0, R4                  ; fd
.text:B1BF746A 29 00       MOVS            R1, R5                  ; buf
.text:B1BF746C CF F7 10 EE BLX             read
.text:B1BF7470 14 28       CMP             R0, #0x14
.text:B1BF7472 18 D1       BNE             loc_B1BF74A6
.text:B1BF7474 28 78       LDRB            R0, [R5]
.text:B1BF7476 7F 28       CMP             R0, #0x7F
.text:B1BF7478 15 D1       BNE             loc_B1BF74A6
.text:B1BF747A 01 A8       ADD             R0, SP, #0x28+buf
.text:B1BF747C 40 78       LDRB            R0, [R0,#1]
.text:B1BF747E 45 28       CMP             R0, #0x45 ; 'E'
.text:B1BF7480 11 D1       BNE             loc_B1BF74A6
.text:B1BF7482 01 A8       ADD             R0, SP, #0x28+buf
.text:B1BF7484 80 78       LDRB            R0, [R0,#2]
.text:B1BF7486 4C 28       CMP             R0, #0x4C ; 'L'
.text:B1BF7488 0D D1       BNE             loc_B1BF74A6
.text:B1BF748A 01 A8       ADD             R0, SP, #0x28+buf
.text:B1BF748C C0 78       LDRB            R0, [R0,#3]
.text:B1BF748E 46 28       CMP             R0, #0x46 ; 'F'
.text:B1BF7490 09 D1       BNE             loc_B1BF74A6
.text:B1BF7492 01 A8       ADD             R0, SP, #0x28+buf
.text:B1BF7494 80 7C       LDRB            R0, [R0,#0x12]
.text:B1BF7496 3E 28       CMP             R0, #0x3E ; '>'
.text:B1BF7498 01 D0       BEQ             loc_B1BF749E
.text:B1BF749A 03 28       CMP             R0, #3
.text:B1BF749C 03 D1       BNE             loc_B1BF74A6
.text:B1BF749E
.text:B1BF749E             loc_B1BF749E 
.text:B1BF749E 20 00       MOVS            R0, R4                  ; fd
.text:B1BF74A0 CF F7 88 EC BLX             close
```

#### 4.2、系统环境检测

调用 Native 层 Object[] v12_2 = NBridge.main3(1, new Object[1])，传入参数为 1, 表示检测环境， 检测系统目录中是否有 ls 文件且是否为 elf 格式:

```
//检测root 直接用svc指令，防止hook
.text:BB9F5444 ;faccessat
.text:BB9F5444 F0 B5       PUSH            {R4-R7,LR}
.text:BB9F5446 03 AF       ADD             R7, SP, #0xC
.text:BB9F5448 0B 00       MOVS            R3, R1
.text:BB9F544A 04 00       MOVS            R4, R0
.text:BB9F544C 63 20       MOVS            R0, #0x63 ; 'c'
.text:BB9F544E C5 43       MVNS            R5, R0
.text:BB9F5450 A7 20       MOVS            R0, #0xA7
.text:BB9F5452 46 00       LSLS            R6, R0, #1
.text:BB9F5454 28 46       MOV             R0, R5
.text:BB9F5456 21 46       MOV             R1, R4
.text:BB9F5458 1A 46       MOV             R2, R3
.text:BB9F545A 37 46       MOV             R7, R6
.text:BB9F545C 00 DF       SVC             0
.text:BB9F545E 03 46       MOV             R3, R0
.text:BB9F5460 18 00       MOVS            R0, R3
.text:BB9F5462 F0 BD       POP             {R4-R7,PC}

//检测文件特征
/system/bin/su 
/system/bin/360s
/system/xbin/krdem
/system/xbin/ku.sud
/system/xbin/ksud
/system/bin/.usr/.ku
/system/xbin/ku.sud.tmp
/system/bin/ddexe_real
/system/bin/.krsh
/system/bin/.suv
/system/bin/debuggerd-ku.bak
/system/xbin/kugote
/system/xbin/kugote
/system/usr/ikm/ikmsu
/system/etc/kds
/system/xbin/start_kusud.sh
/system/bin/ddexe-tcs.bak

//检测结果
1|2|3|32 //字符串的编号
```

检测 root:

```
//检测
de.robv.android.xposed.XposedBridge

//检测代理
http.proxyHost
https.proxyHost
getProperty
```

检测 XPOSED、代理、ROM 是否为自己编译的

```
//检测方法是否被hook
传入方法首地址进行对比 0x780x5F  0xF8DF
//如果检测到第一个方法被hook，结果如下
1|
如果检测到第2个方法被hook，结果如下
1|2|
如果检测到第3个方法被hook，结果如下
1|2|3|
```

检测关键方法是否被 hook

```
boolean v2 = MTGuard.isEmu();
boolean v3 = MTGuard.isRoot();
boolean v4 = MTGuard.hasMalware();
boolean v5 = MTGuard.isDarkSystem();
boolean v6 = MTGuard.isVirtualLocation();
boolean v7 = MTGuard.isRemoteCall();
boolean v8 = MTGuard.isSigCheckOK();
boolean v11 = MTGuard.inSandBox();
boolean v13 = MTGuard.isHook();
boolean v14 = MTGuard.isDebug();
boolean v15 = MTGuard.isProxy();
boolean v16 = MTGuard.isCameraHack();
最终都会走到Native方法中去
private static native Object[] main(int arg0, Object[] arg1);
```

其它检测，代码如下:

```
.text:B65E1FE4 F0 B5       PUSH            {R4-R7,LR}
.text:B65E1FE6 03 AF       ADD             R7, SP, #0xC
.text:B65E1FE8 8F B0       SUB             SP, SP, #0x3C
.text:B65E1FEA 0E 00       MOVS            R6, R1
.text:B65E1FEC 2D 49       LDR             R1, =(__stack_chk_guard_ptr - 0xB65E1FF2)
.text:B65E1FEE 79 44       ADD             R1, PC                  ; __stack_chk_guard_ptr
.text:B65E1FF0 0C 68       LDR             R4, [R1]                ; __stack_chk_guard
.text:B65E1FF2 21 68       LDR             R1, [R4]
.text:B65E1FF4 0E 91       STR             R1, [SP,#0x48+var_10]
.text:B65E1FF6 0A 21       MOVS            R1, #0xA
.text:B65E1FF8 CA 43       MVNS            R2, R1
.text:B65E1FFA 01 68       LDR             R1, [R0]                ; file
.text:B65E1FFC 48 1E       SUBS            R0, R1, #1
.text:B65E1FFE 82 58       LDR             R2, [R0,R2]
.text:B65E2000 00 25       MOVS            R5, #0
.text:B65E2002 E8 43       MVNS            R0, R5
.text:B65E2004 00 2A       CMP             R2, #0
.text:B65E2006 43 D0       BEQ             loc_B65E2090
.text:B65E2008 00 25       MOVS            R5, #0
.text:B65E200A 05 95       STR             R5, [SP,#0x48+var_34]
.text:B65E200C 04 90       STR             R0, [SP,#0x48+var_38]
.text:B65E200E 06 90       STR             R0, [SP,#0x48+var_30]
.text:B65E2010 07 95       STR             R5, [SP,#0x48+var_2C]
.text:B65E2012 08 90       STR             R0, [SP,#0x48+var_28]
.text:B65E2014 0C 90       STR             R0, [SP,#0x48+var_18]
.text:B65E2016 0D 95       STR             R5, [SP,#0x48+var_14]
.text:B65E2018 E8 43       MVNS            R0, R5
.text:B65E201A 0B 90       STR             R0, [SP,#0x48+var_1C]
.text:B65E201C 0A 90       STR             R0, [SP,#0x48+var_20]
.text:B65E201E 04 A8       ADD             R0, SP, #0x48+var_38    ; int
.text:B65E2020 0D F0 40 FF BL              openfile_sub_C6BADEA4   ; R3：要读取的文件
.text:B65E2024 01 30       ADDS            R0, #1
.text:B65E2026 30 D0       BEQ             loc_B65E208A
.text:B65E2028 02 96       STR             R6, [SP,#0x48+var_40]
.text:B65E202A 04 A8       ADD             R0, SP, #0x48+var_38
.text:B65E202C 1E 49       LDR             R1, =(sub_B65E20AC+1 - 0xB65E2032)
.text:B65E202E 79 44       ADD             R1, PC                  ; sub_B65E20AC
.text:B65E2030 0E F0 C8 F9 BL              size_sub_C6BAE3C4
.text:B65E2034 06 00       MOVS            R6, R0
.text:B65E2036 00 2E       CMP             R6, #0
.text:B65E2038 27 D0       BEQ             loc_B65E208A
.text:B65E203A 00 25       MOVS            R5, #0
.text:B65E203C 03 95       STR             R5, [SP,#0x48+byte_count]
.text:B65E203E 04 A8       ADD             R0, SP, #0x48+var_38
.text:B65E2040 03 AA       ADD             R2, SP, #0x48+byte_count
.text:B65E2042 31 00       MOVS            R1, R6
.text:B65E2044 0E F0 2E FB BL              read_sub_C6BAE6A4
.text:B65E2048 03 99       LDR             R1, [SP,#0x48+byte_count]
.text:B65E204A 00 29       CMP             R1, #0
.text:B65E204C 1D D0       BEQ             loc_B65E208A
.text:B65E204E 00 28       CMP             R0, #0
.text:B65E2050 1B D0       BEQ             loc_B65E208A
.text:B65E2052 08 00       MOVS            R0, R1                  ; byte_count
.text:B65E2054 01 91       STR             R1, [SP,#0x48+var_44]
.text:B65E2056 EF F7 48 EE BLX             malloc                  ; 分配存放空间
.text:B65E205A 02 00       MOVS            R2, R0
.text:B65E205C 00 28       CMP             R0, #0
.text:B65E205E 14 D0       BEQ             loc_B65E208A
.text:B65E2060 00 21       MOVS            R1, #0
.text:B65E2062 15 00       MOVS            R5, R2
.text:B65E2064 28 00       MOVS            R0, R5
.text:B65E2066 01 9A       LDR             R2, [SP,#0x48+var_44]
.text:B65E2068 16 F0 2A FA BL              memset_sub_C6BB64C0
.text:B65E206C 04 A8       ADD             R0, SP, #0x48+var_38
.text:B65E206E 31 00       MOVS            R1, R6
.text:B65E2070 2A 00       MOVS            R2, R5
.text:B65E2072 0E F0 6B FB BL              ReadFile_unzip_sub_C6BAE74C ; 读取并解压
.text:B65E2076 00 28       CMP             R0, #0
.text:B65E2078 03 D0       BEQ             loc_B65E2082
.text:B65E207A 03 98       LDR             R0, [SP,#0x48+byte_count]
.text:B65E207C 02 99       LDR             R1, [SP,#0x48+var_40]
.text:B65E207E 08 60       STR             R0, [R1]
```

#### 4.3、读取资源文件并解密

##### 4.3.1、读文件 META-INF/SANKUAI.RSA 计算 MD5

代码如下：

```
B3C7B000  30 82 03 F5 06 09 2A 86  48 86 F7 0D 01 07 02 A0  0.......H.......
B3C7B010  82 03 E6 30 82 03 E2 02  01 01 31 0F 30 0D 06 09  ..........1.0...
B3C7B020  60 86 48 01 65 03 04 02  01 05 00 30 0B 06 09 2A  `.H.e......0...*
B3C7B030  86 48 86 F7 0D 01 07 01  A0 82 02 83 30 82 02 7F  .H..........0...
B3C7B040  30 82 01 E8 A0 03 02 01  02 02 04 4D 69 1B B8 30  0..........Mi..0
B3C7B050  0D 06 09 2A 86 48 86 F7  0D 01 01 05 05 00 30 81  ...*.H........0.
B3C7B060  82 31 0B 30 09 06 03 55  04 06 13 02 43 4E 31 10  .1.0...U....CN1.
B3C7B070  30 0E 06 03 55 04 08 13  07 42 65 69 6A 69 6E 67  0...U....Beijing
B3C7B080  31 10 30 0E 06 03 55 04  07 13 07 42 65 69 6A 69  1.0...U....Beiji
B3C7B090  6E 67 31 24 30 22 06 03  55 04 0A 13 1B 53 61 6E  ng1$0"..U....San
B3C7B0A0  6B 75 61 69 20 54 65 63  68 6E 6F 6C 6F 67 79 20  kuai Technology 
B3C7B0B0  43 6F 2E 20 4C 74 64 2E  31 14 30 12 06 03 55 04  Co. Ltd.1.0...U.
B3C7B0C0  0B 13 0B 6D 65 69 74 75  61 6E 2E 63 6F 6D 31 13  ...meituan.com1.
```

读取的数据与 APk 包中的内容是一样的。

```
.text:B65EFCBC             isRSA_sub_CB471CBC 
.text:B65EFCBC             ; __unwind { // FDA46000
.text:B65EFCBC D0 B5       PUSH            {R4,R6,R7,LR}
.text:B65EFCBE 02 AF       ADD             R7, SP, #8
.text:B65EFCC0 01 00       MOVS            R1, R0
.text:B65EFCC2 00 20       MOVS            R0, #0
.text:B65EFCC4 00 29       CMP             R1, #0
.text:B65EFCC6 1B D0       BEQ             locret_B65EFD00
.text:B65EFCC8 0A 78       LDRB            R2, [R1]
.text:B65EFCCA 30 2A       CMP             R2, #0x30 ; '0'
.text:B65EFCCC 18 D1       BNE             locret_B65EFD00
.text:B65EFCCE 01 22       MOVS            R2, #1
.text:B65EFCD0 8C 56       LDRSB           R4, [R1,R2]
.text:B65EFCD2 7F 22       MOVS            R2, #0x7F
.text:B65EFCD4 22 40       ANDS            R2, R4
.text:B65EFCD6 FF 23       MOVS            R3, #0xFF
.text:B65EFCD8 23 40       ANDS            R3, R4
.text:B65EFCDA 00 2C       CMP             R4, #0
.text:B65EFCDC 00 DB       BLT             loc_B65EFCE0
.text:B65EFCDE 1A 00       MOVS            R2, R3
.text:B65EFCE0
.text:B65EFCE0             loc_B65EFCE0                     
.text:B65EFCE0 04 2A       CMP             R2, #4
.text:B65EFCE2 0D D8       BHI             locret_B65EFD00
.text:B65EFCE4 00 2A       CMP             R2, #0
.text:B65EFCE6 0B D0       BEQ             locret_B65EFD00
.text:B65EFCE8 02 31       ADDS            R1, #2
.text:B65EFCEA D3 00       LSLS            R3, R2, #3
.text:B65EFCEC 08 3B       SUBS            R3, #8
.text:B65EFCEE 00 20       MOVS            R0, #0
.text:B65EFCF0
.text:B65EFCF0             loc_B65EFCF0   
.text:B65EFCF0 0C 78       LDRB            R4, [R1]
.text:B65EFCF2 9C 40       LSLS            R4, R3
.text:B65EFCF4 20 43       ORRS            R0, R4
.text:B65EFCF6 01 3A       SUBS            R2, #1
.text:B65EFCF8 08 3B       SUBS            R3, #8
.text:B65EFCFA 01 31       ADDS            R1, #1
.text:B65EFCFC 00 2A       CMP             R2, #0
.text:B65EFCFE F7 D1       BNE             loc_B65EFCF0
.text:B65EFD00
.text:B65EFD00             locret_B65EFD00   
.text:B65EFD00 D0 BD       POP             {R4,R6,R7,PC}
```

检测是否为 RSA 文件：

```
int __fastcall md5_sub_C6FEF398(int a1, int a2, int a3)
{
  int i; // r0
  int v8[26]; // [sp+10h] [bp-68h] BYREF

  memset_sub_C6BB64C0(v8, 0, 88);
  v8[0] = 0;
  v8[1] = 0;
  v8[2] = 1732584193;
  v8[3] = -271733879;
  v8[4] = -1732584194;
  v8[5] = 271733878;
  sub_B65F11F4(v8, a1, a2);
  sub_B65F1294(v8, a3);
  for ( i = 0; i != 88; ++i )
    *((_BYTE *)v8 + i) = 0;
  return _stack_chk_guard - v8[22];
}
```

计算 MD5 值

```
638C81261479C2104EDE3F2518E91725
```

计算后的值为：

```
com.sankuai.meituanWU@TEN638C81261479C2104EDE3F2518E91725
```

该值会作为解密资源密钥的一部分。  

##### 4.3.2、解密 assets/ms_com.sankuai.meituan

###### 组合密钥：

包名 + 常量字符 (WU@TEN)+META-INF/SANKUAI.RSA(md5)

```
.text:C1D52AA4 F0 B5       PUSH            {R4-R7,LR}
.text:C1D52AA6 03 AF       ADD             R7, SP, #0xC
.text:C1D52AA8 8F B0       SUB             SP, SP, #0x3C
.text:C1D52AAA 15 00       MOVS            R5, R2
.text:C1D52AAC 0E 00       MOVS            R6, R1
.text:C1D52AAE 02 90       STR             R0, [SP,#0x48+var_40]
.text:C1D52AB0 1A 48       LDR             R0, =(__stack_chk_guard_ptr - 0xC1D52AB6)
.text:C1D52AB2 78 44       ADD             R0, PC                  ; __stack_chk_guard_ptr
.text:C1D52AB4 04 68       LDR             R4, [R0]                ; __stack_chk_guard
.text:C1D52AB6 20 68       LDR             R0, [R4]
.text:C1D52AB8 0E 90       STR             R0, [SP,#0x48+var_10]
.text:C1D52ABA 00 2E       CMP             R6, #0
.text:C1D52ABC 1F D0       BEQ             loc_C1D52AFE
.text:C1D52ABE 00 2D       CMP             R5, #0
.text:C1D52AC0 1D D0       BEQ             loc_C1D52AFE
.text:C1D52AC2 01 94       STR             R4, [SP,#0x48+var_44]
.text:C1D52AC4 05 AC       ADD             R4, SP, #0x48+var_34
.text:C1D52AC6 21 21       MOVS            R1, #0x21 ; '!'
.text:C1D52AC8 20 00       MOVS            R0, R4
.text:C1D52ACA F2 F7 26 E9 BLX             __aeabi_memclr4
.text:C1D52ACE 30 00       MOVS            R0, R6
.text:C1D52AD0 29 00       MOVS            R1, R5
.text:C1D52AD2 22 00       MOVS            R2, R4
.text:C1D52AD4 12 F0 8E FB BL              JMP_Sha1_256_sub_C6BB01F4 ; R0:原始值，R1：大小,R2:返回值
.text:C1D52AD8 03 A8       ADD             R0, SP, #0x48+var_3C
.text:C1D52ADA 20 22       MOVS            R2, #0x20 ; ' '
.text:C1D52ADC 21 00       MOVS            R1, R4
.text:C1D52ADE 01 9C       LDR             R4, [SP,#0x48+var_44]
.text:C1D52AE0 FF F7 72 FF BL              Hex2String_sub_C6B9D9C8 ; R1:返回值，R2：大小
.text:C1D52AE4 03 98       LDR             R0, [SP,#0x48+var_3C]
.text:C1D52AE6 02 99       LDR             R1, [SP,#0x48+var_40]
.text:C1D52AE8 08 60       STR             R0, [R1]
.text:C1D52AEA 0D 48       LDR             R0, =(off_C1DD8C48 - 0xC1D52AF0)
.text:C1D52AEC 78 44       ADD             R0, PC                  ; off_C1DD8C48
.text:C1D52AEE 00 68       LDR             R0, [R0]                ; dword_C1DEAADC
.text:C1D52AF0 01 00       MOVS            R1, R0
.text:C1D52AF2 0C 31       ADDS            R1, #0xC
.text:C1D52AF4 03 91       STR             R1, [SP,#0x48+var_3C]
.text:C1D52AF6 04 A9       ADD             R1, SP, #0x48+var_38
.text:C1D52AF8 6B F0 26 FE BL              free_sub_B2943748
```

将组合后的字符串加密 Sha1 值

```
//该值为解密assets/ms_com.sankuai.meituan的密钥
69fe5963f3b95d9718c8d3e4f924ad9379500e9b51d80686e65347890e1748fe
```

加密后的值为：

```
int __fastcall ReadFile_unzip_sub_C6BAE74C(int *a1, int a2, int a3)
{
  int v5; // r0
  int v6; // r5
  int v7; // r0
  int v8; // r6
  int v9; // r5
  int v10; // r0
  __int64 v13; // [sp+24h] [bp-28h] BYREF
  unsigned int v14; // [sp+30h] [bp-1Ch] BYREF
  int v15; // [sp+34h] [bp-18h] BYREF
  int v16; // [sp+38h] [bp-14h] BYREF

  v5 = a2 - 10000;
  v6 = 0;
  if ( a2 >= 10000 && v5 < a1[8] )
  {
    if ( *(_DWORD *)(a1[9] + 8 * v5) )
    {
      v6 = 0;
      if ( read_sub_C6BAE448(a1, a2, &v16, &v15, (int *)&v14, &v13, 0, 0) )
      {
        v7 = mapfile_sub_C6BAE6C0(a1, a2);
        v8 = v7;
        if ( v7 )
        {
          v9 = *(_DWORD *)(v7 + 24);
          if ( v14 > 0x8000 )
            sub_C1D62E3C(v7, 2);
          if ( v16 )
          {
            v10 = unzip_sub_C6BAE820(a3, v9, v15, v14);
            v6 = 0;
            if ( !v10 )
            {
LABEL_15:
              free_sub_CA79DF8C(v8);
              return v6;
            }
          }
          else
          {
            getvalu_sub_C87B550A(a3, v9, v15);
          }
          if ( v14 > 0x8000 )
            sub_C1D62E3C(v8, 0);
          v6 = 1;
          goto LABEL_15;
        }
      }
    }
  }
  return v6;
}
```

读取资源文件 assets/ms_com.sankuai.meituan

```
BCF4A000  89 50 4E 47 0D 0A 1A 0A  00 00 00 0D 49 48 44 52  .PNG........IHDR
BCF4A010  00 00 00 3C 00 00 00 3C  08 06 00 00 00 3A FC D9  ...<...<.....:..
BCF4A020  72 00 00 13 56 49 44 41  54 78 DA ED 5A 79 70 55  r...VIDATx..ZypU
BCF4A030  D7 79 BF 42 AC C6 80 09  5E 00 DB 98 5D E0 64 92  ...B.ƀ .^...]...
BCF4A040  4E 92 76 6C 37 EE 4C 26  4D 5D A7 6E DA 74 49 26  N.vl7...M].n..I&
BCF4A050  53 B7 C9 8C FF 68 DC 75  9C DA 1D D7 F5 C4 4E 13  S.Ɍ .h..........
BCF4A060  63 D7 98 C5 80 36 90 90  D0 CA 62 83 84 D9 F4 F4  cט ŀ 6....b.....
BCF4A070  16 BD 4D CB D3 BE 20 24  10 9B 10 12 9B 04 48 08  ..M... $......H.
BCF4A080  B4 DE 5F 7F E7 DC E5 9D  FB 24 40 24 B6 EB E9 F4  .........$@$....
```

读取后内容 (部分)

```
int __fastcall ParsePng_sub_CB4C8858(
        int *a1,
        unsigned int *a2,
        unsigned int *a3,
        _DWORD *a4,
        unsigned int data,
        unsigned int a6)
{
  int *v54; // r4
  void *v55; // r0
  int (__fastcall *v56)(void **, int *, unsigned __int8 *, unsigned int, _DWORD *); 
  v7 = 0;
  *a1 = 0;
  *a3 = 0;
  *a2 = 0;
  result = CRC32_sub_CB4C7D08((int *)a2, (int *)a3, a4, (unsigned __int8 *)data, a6);
  a4[92] = result;
  if ( result )
    return result;
  v110 = a4 + 92;
  v9 = a4[39];
  v10 = a4[38];
  v135 = a4 + 39;
  v104 = a4 + 38;
  v108 = a4;
  v131 = a4 + 35;
  v148 = (void *)*a3;
  v11 = *a2;
  v12 = 0;
  if ( v10 <= 6 )
    v12 = dword_C1DD1C40[v10];
  v13 = v12 * v9;
  v14 = v108[27];
  v15 = v108[28];
  if ( v14 <= 6 )
    v7 = dword_C1DD1C40[v14];
  if ( v13 <= v7 * v15 )
  {
    v18 = 0;
    v17 = *a3;
    if ( v14 <= 6 )
      v18 = dword_C1DD1C40[v14];
    v143 = v15 * v18;
  }
  else
  {
    v16 = 0;
    v17 = *a3;
    if ( v10 <= 6 )
      v16 = dword_C1DD1C40[v10];
    v143 = v9 * v16;
  }
  v19 = v11 * v17;
  v20 = 0;
  if ( v11 )
  {
    v21 = to1028_sub_CA812FD0(v11 * v17, v11);
    v20 = 0;
    v17 = (unsigned int)v148;
    if ( (void *)v21 != v148 )
      goto LABEL_125;
    if ( v19 )
    {
      v22 = to1028_sub_CA812FD0(8 * v19, v19);
      v20 = 0;
      v17 = (unsigned int)v148;
      if ( v22 != 8 )
        goto LABEL_125;
    }
    if ( v11 >> 3 )
    {
      v23 = to1028_sub_CA812FD0((v11 >> 3) * v143, v11 >> 3);
      v17 = (unsigned int)v148;
      v20 = (v11 >> 3) * v143;
      if ( v23 != v143 )
        goto LABEL_125;
    }
  }
  v24 = v20 + (((v11 & 7) * v143 + 7) >> 3);
  if ( v24 < v20 || v24 > 0xFFFFFFFA || to1028_sub_CA812FD0(v17 * (v24 + 5), v24 + 5) != v17 )
  {
LABEL_125:
    result = 92;
    *v110 = 92;
    return result;
  }
  v25 = v108;
  v111 = v108 + 1;
  v26 = (unsigned __int8 *)(data + 33);
  pdata = 0;
  v129 = 0;
  v119 = 0;
  v116 = 1;
  while ( 1 )
  {
    v27 = (unsigned int)&v26[-data + 12];
    if ( (unsigned int)v26 < data || v27 > a6 )
    {
      if ( v25[7] )
        goto LABEL_136;
      v44 = 30;
      goto LABEL_132;
    }
    v144 = v26;
    v28 = v26;
    v29 = _byteswap_ulong(*(_DWORD *)v26);
    v30 = v110;
    if ( v29 < 0 )
    {
      v25 = v108;
      if ( v108[7] )
        goto LABEL_136;
      v44 = 63;
      goto LABEL_133;
    }
    if ( v29 + v27 > a6 || (unsigned int)&v26[v29 + 12] < data )
      break;
    v31 = v26 + 8;
    if ( !byte_C1DEAAB4 )
    {
      DecString((const char *)byte_C1DE8374, aXpvn, 17, 4);
      v28 = v144;
      byte_C1DE8378 = 0;
    }
    byte_C1DEAAB4 = 1;
    if ( getIHDR_sub_CCBFBBC8(v28, byte_C1DE8374) )
    {
      v32 = v29 + v129;
      if ( v29 + v129 < v29 )
      {
        v43 = 95;
        goto LABEL_134;
      }
      v122 = v29 + v129;
      if ( v119 >= v32 )
      {
        v34 = (int)pdata;
      }
      else
      {
        v33 = v29 + v129;
        if ( v32 <= 2 * v119 )
          v33 = (3 * v32) >> 1;
        v119 = v33;
        v34 = realloc(pdata);
        if ( !v34 )
        {
          v43 = 83;
          goto LABEL_134;
        }
      }
      v35 = 0;
      v116 = 3;
      if ( v29 )
      {
        v37 = v129;
        do
        {
          *(_BYTE *)(v34 + v37) = *v31++;
          --v29;
          ++v37;
        }
        while ( v29 );
      }
      v129 = v122;
      pdata = (unsigned __int8 *)v34;
      goto LABEL_50;
    }
    if ( !byte_C1DEAAB5 )
    {
      DecString((const char *)byte_C1DE8379, aPv, 18, 4);
      byte_C1DE837D = 0;
    }
    v35 = 1;
    byte_C1DEAAB5 = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE8379) )
      goto LABEL_50;
    if ( !byte_C1DEAAA8 )
    {
      DecString((const char *)byte_C1DE8338, aSjI, 3, 4);
      unk_C1DE833C = 0;
    }
    byte_C1DEAAA8 = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE8338) )
    {
      v36 = dec_sub_CA801FF0(v104, v31, v29);
      *v110 = v36;
      v35 = 0;
      v116 = 2;
LABEL_67:
      v25 = v108;
      v38 = (unsigned int *)v144;
      if ( v36 )
        goto LABEL_136;
LABEL_51:
      if ( !v25[5] )
      {
        v39 = gencrc_sub_CA800C08(v38);
        v38 = (unsigned int *)v144;
        if ( v39 )
        {
          v44 = 57;
LABEL_132:
          v30 = v110;
LABEL_133:
          *v30 = v44;
          goto LABEL_136;
        }
      }
      if ( v35 )
        goto LABEL_136;
LABEL_55:
      v40 = *v110;
      goto LABEL_56;
    }
    if ( !byte_C1DEAAA9 )
    {
      DecString((const char *)byte_C1DE833D, aPud, 4, 4);
      byte_C1DE8341 = 0;
    }
    byte_C1DEAAA9 = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE833D) )
    {
      v36 = sub_C1DB9080(v104, v31, v29);
LABEL_66:
      *v110 = v36;
      v35 = 0;
      goto LABEL_67;
    }
    if ( !byte_C1DEAAAA )
    {
      DecString((const char *)byte_C1DE8342, aGclj, 5, 4);
      byte_C1DE8346 = 0;
    }
    byte_C1DEAAAA = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE8342) )
    {
      v36 = sub_C1DB9100(v131, v31, v29);
      goto LABEL_66;
    }
    if ( !byte_C1DEAAAB )
    {
      DecString((const char *)byte_C1DE8347, aRlt, 6, 4);
      byte_C1DE834B = 0;
    }
    byte_C1DEAAAB = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE8347) )
    {
      v35 = 0;
      if ( v108[9] )
      {
        v36 = sub_C1DB917C((int)v131, v31, v29);
LABEL_79:
        *v110 = v36;
        goto LABEL_67;
      }
      goto LABEL_50;
    }
    if ( !byte_C1DEAAAC )
    {
      DecString((const char *)byte_C1DE834C, aUd, 7, 4);
      byte_C1DE8350 = 0;
    }
    byte_C1DEAAAC = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE834C) )
    {
      v35 = 0;
      if ( v108[9] )
      {
        v36 = sub_C1DB923C(v131, v111, v31, v29);
        goto LABEL_79;
      }
LABEL_50:
      v25 = v108;
      v38 = (unsigned int *)v144;
      goto LABEL_51;
    }
    if ( !byte_C1DEAAAD )
    {
      DecString((const char *)byte_C1DE8351, aAVe, 8, 4);
      byte_C1DE8355 = 0;
    }
    byte_C1DEAAAD = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE8351) )
    {
      if ( v108[9] )
      {
        v41 = sub_C1DB936C(v131, v111, v31, v29);
        *v110 = v41;
        v25 = v108;
        v38 = (unsigned int *)v144;
        v35 = 0;
        if ( v41 )
          goto LABEL_136;
      }
      else
      {
        v25 = v108;
        v38 = (unsigned int *)v144;
        v35 = 0;
      }
      goto LABEL_51;
    }
    if ( !byte_C1DEAAAE )
    {
      DecString((const char *)byte_C1DE8356, aEbw, 9, 4);
      unk_C1DE835A = 0;
    }
    byte_C1DEAAAE = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE8356) )
    {
      if ( v29 != 7 )
      {
        v43 = 73;
LABEL_134:
        v30 = v110;
        goto LABEL_135;
      }
      v108[58] = 1;
      v38 = (unsigned int *)v144;
      v108[59] = (v144[8] << 8) | v144[9];
      v108[60] = v144[10];
      v108[61] = v144[11];
      v108[62] = v144[12];
      v108[63] = v144[13];
      v108[64] = v144[14];
LABEL_92:
      v35 = 0;
      *v110 = 0;
      v25 = v108;
      goto LABEL_51;
    }
    if ( !byte_C1DEAAAF )
    {
      DecString((const char *)byte_C1DE835B, aZei, 10, 4);
      unk_C1DE835F = 0;
    }
    byte_C1DEAAAF = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE835B) )
    {
      v36 = sub_C1DB95FC(v131, (int)v31, v29);
      goto LABEL_66;
    }
    if ( !byte_C1DEAAB0 )
    {
      DecString((const char *)byte_C1DE8360, aLoU, 11, 4);
      unk_C1DE8364 = 0;
    }
    byte_C1DEAAB0 = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE8360) )
    {
      if ( v29 != 4 )
      {
        v43 = 96;
        goto LABEL_134;
      }
      v108[69] = 1;
      v38 = (unsigned int *)v144;
      v108[70] = _byteswap_ulong(*((_DWORD *)v144 + 2));
      goto LABEL_92;
    }
    if ( !byte_C1DEAAB1 )
    {
      DecString((const char *)byte_C1DE8365, aOgX, 12, 4);
      unk_C1DE8369 = 0;
    }
    byte_C1DEAAB1 = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE8365) )
    {
      v36 = sub_C1DB9640(v131, (unsigned int *)v31, v29);
      goto LABEL_66;
    }
    if ( !byte_C1DEAAB2 )
    {
      DecString((const char *)byte_C1DE836A, aBtt, 13, 4);
      unk_C1DE836E = 0;
    }
    byte_C1DEAAB2 = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE836A) )
    {
      if ( v29 != 1 )
      {
        v43 = 98;
        goto LABEL_134;
      }
      v108[80] = 1;
      v108[81] = *v31;
      v35 = 0;
      *v110 = 0;
      goto LABEL_50;
    }
    if ( !byte_C1DEAAB3 )
    {
      DecString((const char *)byte_C1DE836F, aGrwg, 14, 4);
      unk_C1DE8373 = 0;
    }
    byte_C1DEAAB3 = 1;
    if ( getIHDR_sub_CCBFBBC8(v144, byte_C1DE836F) )
    {
      v36 = sub_C1DB9714(v131, v111, v31, v29);
      goto LABEL_66;
    }
    if ( !v108[6] && (v144[4] & 0x20) == 0 )
    {
      v43 = 69;
      goto LABEL_134;
    }
    if ( !v108[10] )
    {
      v25 = v108;
      v38 = (unsigned int *)v144;
      goto LABEL_55;
    }
    v25 = v108;
    v42 = sub_C1DB7CCE(&v108[v116 + 85], &v108[v116 + 88], (unsigned int *)v144);
    v38 = (unsigned int *)v144;
    *v110 = v42;
    v40 = 0;
    if ( v42 )
      goto LABEL_136;
LABEL_56:
    v26 = isPng_sub_CA800C80((unsigned __int8 *)v38);
    if ( v40 )
      goto LABEL_136;
  }
  v43 = 64;
LABEL_135:
  *v30 = v43;
  v25 = v108;
LABEL_136:
  v154 = 0;
  v152 = 0;
  v153 = 0;
  v45 = v25[37];
  v123 = v25 + 37;
  v46 = *a2;
  if ( v45 )
  {
    v145 = *a3;
    v47 = (*a3 + 7) >> 3;
    v140 = sub_C1DBAC4C((v46 + 7) >> 3, v47, (unsigned int *)v104);
    v48 = v46 + 3;
    if ( v46 >= 5 )
    {
      v49 = sub_C1DBAC4C(v48 >> 3, v47, (unsigned int *)v104) + v140;
      v48 = v46 + 3;
      v140 = v49;
    }
    v50 = sub_C1DBAC4C(v48 >> 2, (v145 + 3) >> 3, (unsigned int *)v104) + v140;
    v51 = v46 + 1;
    if ( v46 >= 3 )
    {
      v50 += sub_C1DBAC4C(v51 >> 2, (v145 + 3) >> 2, (unsigned int *)v104);
      v51 = v46 + 1;
    }
    v52 = sub_C1DBAC4C(v51 >> 1, (v145 + 1) >> 2, (unsigned int *)v104) + v50;
    if ( v46 >= 2 )
      v52 += sub_C1DBAC4C(v46 >> 1, (v145 + 1) >> 1, (unsigned int *)v104);
    v53 = sub_C1DBAC4C(v46, v145 >> 1, (unsigned int *)v104) + v52;
  }
  else
  {
    v53 = sub_C1DBAC4C(*a2, *a3, (unsigned int *)v104);
  }
  v54 = v110;
  if ( !*v110 )
  {
    if ( !v53 )
      goto LABEL_149;
    v55 = (void *)realloc(0);
    if ( !v55 )
    {
      v57 = 83;
      goto LABEL_157;
    }
    v152 = v55;
    v154 = v53;
    if ( !*v110 )
    {
LABEL_149:
      v56 = (int (__fastcall *)(void **, int *, unsigned __int8 *, unsigned int, _DWORD *))v108[2];
      if ( v56 )
        v57 = v56(&v152, &v153, pdata, v129, v111);
      else
        v57 = sub_C1DB7A98(&v152, &v153, pdata, v129, v111);
      v58 = v57;
      v54 = v110;
      if ( v153 != v53 )
        v58 = 91;
      if ( !v57 )
        v57 = v58;
LABEL_157:
      *v54 = v57;
    }
  }
  free(pdata);
  if ( !*v54 )
  {
    v59 = *v135;
    v107 = *a2;
    v132 = *a3;
    v146 = *v104;
    v60 = sub_C1DB7DA4(*a2, *a3, *v104, *v135);
    v61 = malloc(v60);
    *a1 = v61;
    v62 = 83;
    if ( v61 )
    {
      if ( v60 )
      {
        for ( i = 0; i != v60; ++i )
        {
          *(_BYTE *)(v61 + i) = 0;
          v61 = *a1;
        }
        v64 = v108[38];
        v59 = *v135;
        v132 = *a3;
        v107 = *a2;
      }
      else
      {
        v64 = v146;
      }
      v147 = (char *)v152;
      v65 = 0;
      if ( v64 <= 6 )
        v65 = dword_C1DD1C40[v64];
      v66 = v59 * v65;
      v62 = 31;
      if ( v66 )
      {
        if ( *v123 )
        {
          v139 = v61;
          sub_C1DBAF5C(v159, v158, v157, v156, v155, v107, v132, v66);
          v67 = 0;
          do
          {
            v68 = *(_DWORD *)&v158[v67];
            v141 = v67;
            v136 = &v147[*(_DWORD *)&v156[v67]];
            v69 = *(_DWORD *)&v159[v67];
            v62 = sub_C1DBAC80(v136);
            if ( v62 )
            {
              v80 = 0;
              v54 = v110;
              goto LABEL_206;
            }
            if ( v66 <= 7 )
              sub_C1DBAECC(&v147[*(_DWORD *)&v155[v141]], v136, v69 * v66, (v69 * v66 + 7) & 0xFFFFFFF8, v68);
            v67 = v141 + 4;
          }
          while ( v141 != 24 );
          sub_C1DBAF5C(v164, v163, v162, v161, v160, v107, v132, v66);
          if ( v66 <= 7 )
          {
            v81 = 0;
            v118 = v66;
            do
            {
              v96 = v81;
              v82 = v81;
              v106 = v163[v82];
              if ( v106 )
              {
                v134 = v164[v82];
                v103 = &dword_C1DD1BAC[v82];
                v101 = (_DWORD *)((char *)&unk_C1DD1BC8 + v82 * 4);
                v99 = &dword_C1DD1BE4[v82];
                v98 = &dword_C1DD1C00[v82];
                v97 = &v160[v82 * 4];
                for ( j = 0; j != v106; ++j )
                {
                  if ( v134 )
                  {
                    pdatab = (void *)((*v99 * j + *v98) * v107 + *v101);
                    v125 = *v103;
                    v121 = 8 * *(_DWORD *)v97;
                    v83 = 0;
                    do
                    {
                      v84 = (v83 + v134 * j) * v66 + v121;
                      v138 = v83;
                      v85 = ((_DWORD)pdatab + v125 * v83) * v66;
                      v86 = v66;
                      do
                      {
                        v87 = ((unsigned __int8)v147[v84 >> 3] >> (~(_BYTE)v84 & 7)) & 1;
                        ++v84;
                        if ( v87 )
                          *(_BYTE *)(v139 + (v85 >> 3)) |= (_BYTE)v87 << (~(_BYTE)v85 & 7);
                        --v86;
                        ++v85;
                      }
                      while ( v86 );
                      v83 = v138 + 1;
                      v66 = v118;
                    }
                    while ( v138 + 1 != v134 );
                  }
                }
              }
              v81 = v96 + 1;
            }
            while ( v96 != 6 );
          }
          else
          {
            v70 = v66 >> 3;
            v71 = 0;
            v149 = v66 >> 3;
            do
            {
              v100 = v71;
              v72 = v71;
              pdataa = (void *)v163[v71];
              if ( pdataa )
              {
                v120 = &dword_C1DD1BAC[v72];
                v117 = (_DWORD *)((char *)&unk_C1DD1BC8 + v72 * 4);
                v112 = &dword_C1DD1BE4[v72];
                v105 = &dword_C1DD1C00[v72];
                v102 = &v160[v72 * 4];
                v142 = v164[v71];
                v124 = v142 * v70;
                v133 = 0;
                v130 = v147;
                do
                {
                  if ( v142 )
                  {
                    v73 = &v130[*(_DWORD *)v102];
                    v137 = *v120 * v70;
                    v74 = (_BYTE *)(v139 + (*v117 + (*v105 + *v112 * (_DWORD)v133) * v107) * v70);
                    for ( k = 0; k != v142; ++k )
                    {
                      if ( v70 )
                      {
                        v76 = v73;
                        v77 = (char *)v149;
                        v78 = v74;
                        do
                        {
                          *v78 = *v76++;
                          --v77;
                          ++v78;
                        }
                        while ( v77 );
                      }
                      v70 = v149;
                      v73 += v149;
                      v74 += v137;
                    }
                  }
                  v130 += v124;
                  v133 = (char *)v133 + 1;
                }
                while ( v133 != pdataa );
              }
              v71 = v100 + 1;
            }
            while ( v100 != 6 );
          }
          v62 = 0;
          v54 = v110;
          v80 = 1;
LABEL_206:
          if ( v80 )
LABEL_207:
            v62 = 0;
        }
        else if ( v66 > 7 || (v79 = v107 * v66, v107 * v66 == ((v107 * v66 + 7) & 0xFFFFFFF8)) )
        {
          v62 = sub_C1DBAC80(v61);
          if ( !v62 )
            goto LABEL_207;
        }
        else
        {
          v151 = (void *)((v107 * v66 + 7) & 0xFFFFFFF8);
          v95 = v61;
          v62 = sub_C1DBAC80(v152);
          if ( !v62 )
          {
            sub_C1DBAECC(v95, v147, v79, v151, v132);
            goto LABEL_207;
          }
        }
      }
    }
    *v54 = v62;
  }
  v88 = 0;
  v153 = 0;
  v154 = 0;
  free(v152);
  result = *v54;
  if ( !*v54 )
  {
    v89 = v108 + 27;
    if ( v108[8] )
    {
      if ( !sub_C1DB8460(v108 + 27, v104) )
      {
        if ( (*v89 | 4) != 6 )
        {
          result = 56;
          if ( v108[28] != 8 )
            return result;
        }
        v150 = (void *)*a1;
        v90 = *a3;
        v91 = *a2;
        v92 = sub_C1DB7DA4(*a2, *a3, *v89, v108[28]);
        v93 = malloc(v92);
        *a1 = v93;
        v94 = 83;
        if ( v93 )
          v94 = sub_C1DB8064(v93, v150, v89, v104, v91, v90);
        *v110 = v94;
        free(v150);
        return *v110;
      }
    }
    else
    {
      result = sub_C1DB7D34(v108 + 27, v104);
      *v110 = result;
      if ( result )
        return result;
    }
    return v88;
  }
  return result;
}
```

解析解密图片得到 PIC 数据

```
BDC74300  2E 50 49 43 90 01 00 00  10 02 00 00 B0 02 00 00  .PIC............
BDC74310  82 1B 33 E3 4F 0C 49 C7  00 00 00 00 2D AA F5 F4  ..3...I.....-...
BDC74320  52 22 D7 92 FB 3E EB AA  4A A2 74 C9 12 2D F9 92  R"ג .>....t..-..
BDC74330  F4 E7 CD 2C D8 33 1A B5  D4 8F F1 1E 11 6A 63 4D  ........ԏ ....cM
BDC74340  73 AC 1E 88 BB E5 A5 D2  19 F4 46 8A 7D 13 57 F8  s.............W.
BDC74350  0C 14 F1 4C C7 E2 89 01  1A BD C8 72 66 3F F0 A9  ............f?..
BDC74360  C2 BA 6B 75 6D E3 03 FC  8F 61 20 96 6F 2C 44 2D  º kum....a .o,D-
BDC74370  D6 19 DE F4 74 8E 6F 01  6B 9D 5F BB 6D C4 6C CD  ....t.o.k._.m...
BDC74380  49 00 8F 36 F9 18 8E 50  2D 35 F3 6C F0 6C 75 84  I..6...P-5....u.
BDC74390  2C 78 A0 D7 8E 64 A1 E9  CC 5C 1A 59 C3 78 E3 7A  ,x...d.....Y....
```

得到 PIC 数据 (部分)(这个数据是伴随 APP 整个生命周期, 主要用途是加解密密钥)

```
.text:C1AF5224 02 98       LDR             R0, [SP,#8]
.text:C1AF5226 06 59       LDR             R6, [R0,R4]
.text:C1AF5228 D6 F7 46 F8 BL              init_DecString_sub_C84582B8
.text:C1AF522C 06 90       STR             R0, [SP,#0x18]
.text:C1AF522E 0A AD       ADD             R5, SP, #0x28 ; '('
.text:C1AF5230 28 00       MOVS            R0, R5
.text:C1AF5232 04 99       LDR             R1, [SP,#0x10]
.text:C1AF5234 05 9A       LDR             R2, [SP,#0x14]
.text:C1AF5236 33 00       MOVS            R3, R6
.text:C1AF5238 FB F7 AC FE BL              GetDeviceInfo_dispatch_loop_sub_C89F8F94 ; 获取设备信息1
.text:C1AF523C 06 98       LDR             R0, [SP,#0x18]
.text:C1AF523E 31 00       MOVS            R1, R6
.text:C1AF5240 2A 00       MOVS            R2, R5
.text:C1AF5242 03 9D       LDR             R5, [SP,#0xC]
.text:C1AF5244 F9 F7 BE F9 BL              jmp_checkField_sub_C89F65C4 ; 判断采集的设备信息是否等于如下特殊字符[]，{}，mtg_block，mtg_unsupport
.text:C1AF5248 0A 98       LDR             R0, [SP,#0x28]
.text:C1AF524A 40 19       ADDS            R0, R0, R5
.text:C1AF524C 09 A9       ADD             R1, SP, #0x24 ; '$'
.text:C1AF524E 4C F0 7B FA BL              free_sub_B2943748
.text:C1AF5252 04 34       ADDS            R4, #4
.text:C1AF5254 34 2C       CMP             R4, #0x34 ; '4'         ; 判断是否结束
.text:C1AF5256 E5 D1       BNE             loc_C1AF5224
.text:C1AF5258 03 B5       PUSH            {R0,R1,LR}              ; 获取结束
```

#### 4.4、采集系统环境信息加密

##### 4.4.1、循环获取信息 0x34 次

```
.text:C207A134 80 00       LSLS            R0, R0, #2              ; 跳到真实执行方法
.text:C207A136 02 A1       ADR             R1, loc_C207A140
.text:C207A138 08 58       LDR             R0, [R1,R0]             ; 取方法表偏移
.text:C207A13A 08 18       ADDS            R0, R1, R0
.text:C207A13C 87 46       MOV             PC, R0
```

在下面地方下是跳转到每一个采集信息方法入口的地方，下好断点:

```
{
  "0": 2,
  "1": ["-", "-", "-", "-", "-", "-", "-", "-", "google", "user", "-", "PQ2A.190305.002", "-", "-", "-", "sailfish", "-", "-", "Pixel", "-", "-", "-", "-", "adb", "-", "-", "-", "-", "-", "{\"1\":\"-\",\"2\":\"-\",\"3\":\"-\",\"4\":\"\",\"5\":\"\",\"6\":\"-\",\"7\":\"-\",\"8\":\"\",\"9\":\"\",\"10\":\"-\",\"11\":\"-\",\"12\":\"32\",\"13\":\"\",\"14\":\"-\",\"15\":\"\",\"33\":{\"0\":0,\"1\":\"-\",\"2\":\"-\",\"3\":\"-\",\"4\":\"-\",\"5\":\"-\",\"6\":\"-\",\"7\":\"-\",\"8\":\"-\",\"9\":\"-\",\"10\":\"-\",\"11\":\"-\",\"12\":\"-\",\"13\":\"-\",\"14\":\"-\",\"15\":\"-\"}}", "midi,adb", "midi,adb", "release-keys", "-", "1", "1", "-", "9"],
  "2": ["-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"],
  "3": "{}"
}
```

获取的系统环境信息

```
result = deflateInit_(&strm, -1, a128, 56);
  if ( !result )
  {
    v6 = deflate(&strm, 4);
    if ( v6 == 1 )
    {
      *a4 = strm.total_out;
      return deflateEnd(&strm);
    }
    deflateEnd(&strm);
    if ( v6 != 2 )
    {
      if ( v6 != -5 )
        return v6;
      result = -3;
      if ( strm.avail_in )
        return v6;
      return result;
    }
    return -3;
  }
  return result;
```

##### 4.4.2、加密设备信息

压缩信息

```
C3904500  78 9C 9D 92 DB 0A C2 30  0C 86 DF 25 D7 CD E8 61  x...............
C3904510  53 B7 3B DF 60 5E 5B 2F  26 AB 5A 9C 0C 56 04 65  S.;..^[/&.Z..V.e
C3904520  F4 DD ED 0E B0 20 4E A6  17 81 2F E1 4F D2 9F B4  ..... N.../.....
C3904530  05 0E 99 64 20 20 DB 03  02 FB 1A E7 BA 3E 57 26  ...d  ........W&
C3904540  C0 DD 99 66 AC E5 3B B9  8D 44 CA 15 4F 22 CE E5  ...f.....D..O"..
C3904550  5B 87 2B 6C 75 B2 EE 42  4A B9 7D 98 EA C3 F0 A2  [.+lu.....}.....
C3904560  3C CE AC 6D 35 08 0D 99  06 D4 C0 34 48 C2 8A 70  <ά m5......4H..p
C3904570  DC 73 8F C9 84 2B A2 58  13 DE 4C 92 74 42 C1 89  ...Ʉ +.X....tB..
C3904580  44 D0 AD 62 58 AB E4 90  29 D2 13 53 19 59 AD 3A  DЭ bX......S.Y.:
C3904590  51 78 7C 37 93 B3 5F 4C  20 71 81 0B 6C 20 F1 81  Qx|7.._L q..l ..
```

压缩后信息

```
69fe5963f3b95d9718c8d3e4f924ad9379500e9b51d80686e65347890e1748fe
69 FE 59 63 F3 B9 5D 97 18 C8 D3 E4 F9 24 AD 93 79 50 0E 9B 51 D8 06 86
E6 53 47 89 0E 17 48 FE
```

解密 PIC 数据获取加密 key 上面计算得到的 sha256 值，(包名 + 固定字符 + SANKUAI.RSA 的 MD5 值)-> 转换成 hex

```
69 FE 59 63 F3 B9 5D 97 18 C8 D3 E4 F9 24 AD 93
```

取前 0x10 字节

```
.text:C1AE9DA2 0A A8       ADD             R0, SP, #0x28 ; '('     ; 生成AES key
.text:C1AE9DA4 08 90       STR             R0, [SP,#0x20]
.text:C1AE9DA6 84 5D       LDRB            R4, [R0,R6]
.text:C1AE9DA8 82 B0       SUB             SP, SP, #8
.text:C1AE9DAA 03 B5       PUSH            {R0,R1,LR}
.text:C1AE9DAC DF F7 72 E9 BLX             jmp_sub_B288C094
.text:C1AE9DB0 54 01       LSLS            R4, R2, #5
.text:C1AE9DB2 00 00       MOVS            R0, R0
.text:C1AE9DB4 01 BC       POP             {R0}
.text:C1AE9DB6 20 18       ADDS            R0, R4, R0
.text:C1AE9DB8 29 00       MOVS            R1, R5
.text:C1AE9DBA 63 F0 49 F9 BL              sub_C1B4D050
.text:C1AE9DBE 07 91       STR             R1, [SP,#0x1C]
.text:C1AE9DC0 82 B0       SUB             SP, SP, #8
.text:C1AE9DC2 03 B5       PUSH            {R0,R1,LR}
.text:C1AE9DC4 DF F7 66 E9 BLX             jmp_sub_B288C094
.text:C1AE9DC8 40 01       LSLS            R0, R0, #5
.text:C1AE9DCA 00 00       MOVS            R0, R0
.text:C1AE9DCC 01 BC       POP             {R0}
.text:C1AE9DCE 20 18       ADDS            R0, R4, R0
.text:C1AE9DD0 04 99       LDR             R1, [SP,#0x10]
.text:C1AE9DD2 63 F0 3D F9 BL              sub_C1B4D050
.text:C1AE9DD6 69 43       MULS            R1, R5
.text:C1AE9DD8 07 98       LDR             R0, [SP,#0x1C]
.text:C1AE9DDA 08 18       ADDS            R0, R1, R0              ; 初始化密钥
.text:C1AE9DDC 03 21       MOVS            R1, #3
.text:C1AE9DDE 41 43       MULS            R1, R0
.text:C1AE9DE0 06 98       LDR             R0, [SP,#0x18]
.text:C1AE9DE2 40 18       ADDS            R0, R0, R1
.text:C1AE9DE4 80 78       LDRB            R0, [R0,#2]
.text:C1AE9DE6 20 40       ANDS            R0, R4
.text:C1AE9DE8 FE 21       MOVS            R1, #0xFE
.text:C1AE9DEA 01 40       ANDS            R1, R0
.text:C1AE9DEC 08 98       LDR             R0, [SP,#0x20]
.text:C1AE9DEE 81 55       STRB            R1, [R0,R6]             ; 存储密钥
.text:C1AE9DF0 01 36       ADDS            R6, #1
.text:C1AE9DF2 10 2E       CMP             R6, #0x10               ; 判断是否结束
.text:C1AE9DF4 D5 D1       BNE             loc_C1AE9DA2            ; 生成AES key
```

生成 AES KEY

```
68 98 08 02 F2 80 1C 94 08 C8 90 E4 A0 04 AC 82
```

生成最终的 AES KEY

```
//IV 0102030405060708
.text:C1AE7598 57 F0 7A F8 BL              AES_set_decrypt_key_sub_CB601690 ; R0:key,R1:大小,R2:初始AES_KEY返回结构
.text:C1AE759C 00 20       MOVS            R0, #0
.text:C1AE759E 69 46       MOV             R1, SP
.text:C1AE75A0 06 9A       LDR             R2, [SP,#0x128+var_110]
.text:C1AE75A2 0A 60       STR             R2, [R1,#0x128+var_128]
.text:C1AE75A4 48 60       STR             R0, [R1,#0x128+var_124]
.text:C1AE75A6 08 98       LDR             R0, [SP,#0x128+var_108]
.text:C1AE75A8 04 9E       LDR             R6, [SP,#0x128+var_118]
.text:C1AE75AA 31 00       MOVS            R1, R6
.text:C1AE75AC 07 9C       LDR             R4, [SP,#0x128+var_10C]
.text:C1AE75AE 22 00       MOVS            R2, R4
.text:C1AE75B0 03 9B       LDR             R3, [SP,#0x128+var_11C]
.text:C1AE75B2 57 F0 35 FD BL              AES_cbc_Dncrypt_sub_CB602020
.text:C1AE75B6 30 00       MOVS            R0, R6
```

AES 解密 pic

```
{"a1":0,"a10":400,"a2":"com.sankuai.meituan","a11":"c1ee9178c95d9ec75f0f076a374df94a032d54c8576298d4f75e653de3705449","a3":"0a16ecd60eb56a6a3349f66cdcf7f7bf5190e5a42d6280d8dc0ee3be228398ec","a4":1100030200,"k0":{"k1":"meituan1sankuai0","k2":"meituan0sankuai1","k3":"$MXMYBS@HelloPay","k4":"Maoyan010iauknaS","k5":"34281a9dw2i701d4","k6":"X%rj@KiuU+|xY}?f"},"a5":"11.3.200","a0":"pw/LhTdeoTTyaxPHcHMy+/ssGNS1ihNkrJ+uBI74FIfd90KlTil1m0i7FF/n0bhY","a6":"/HntC9XIfdUyII/UiVfx020EQPpHz2XZY3qzM2aiNmM0i0pB1yeSO689TY9SBB3s","a7":"QsHnU6kFjTYR8Z6tHEvkGMO2Hrt+NRnVQhmxg6EtVBzuzQcBpma3AdhTWNMpesFT","c0":{"c1":true,"c2":false},"a9":"SDEzWXi5LHL/cuMCZ1zYyv+0hIViqWWf+ShbUYILWf4=","a8":1603800117167}
```

解密解压后 PIC 数据

```
X%rj@KiuU+|xY}?f
```

解析 json 获取 key k6

```
.text:C1B2C832             crc32_loc_CCBA9832  
.text:C1B2C848↓j
.text:C1B2C832 FF 25       MOVS            R5, #0xFF
.text:C1B2C834 05 40       ANDS            R5, R0
.text:C1B2C836 16 78       LDRB            R6, [R2]
.text:C1B2C838 6E 40       EORS            R6, R5
.text:C1B2C83A B5 00       LSLS            R5, R6, #2
.text:C1B2C83C 65 59       LDR             R5, [R4,R5]
.text:C1B2C83E 00 0A       LSRS            R0, R0, #8
.text:C1B2C840 68 40       EORS            R0, R5
.text:C1B2C842 01 39       SUBS            R1, #1
.text:C1B2C844 01 32       ADDS            R2, #1
.text:C1B2C846 00 29       CMP             R1, #0
.text:C1B2C848 F3 D1       BNE             crc32_loc_CCBA9832
```

计算压缩后信息的 CRC 值

```
744d7275X%rj@Kiu //crc32+k6前8字节
```

计算后的 CRC 值为 744d7275 组合加密压缩后设备环境信息的 key

```
.text:C1AE750E 32 00       MOVS            R2, R6
.text:C1AE7510 56 F0 30 FF BL              AES_set_Encrypt_key_sub_CB601374 ; R0:key,R1:长度,R2:返回值
.text:C1AE7514 06 9A       LDR             R2, [SP,#0x120+byte_count]
.text:C1AE7516 01 20       MOVS            R0, #1
.text:C1AE7518 69 46       MOV             R1, SP
.text:C1AE751A 04 9B       LDR             R3, [SP,#0x120+var_110]
.text:C1AE751C 0B 60       STR             R3, [R1,#0x120+var_120]
.text:C1AE751E 48 60       STR             R0, [R1,#0x120+var_11C]
.text:C1AE7520 05 98       LDR             R0, [SP,#0x120+p]
.text:C1AE7522 02 9C       LDR             R4, [SP,#0x120+var_118]
.text:C1AE7524 21 00       MOVS            R1, R4
.text:C1AE7526 33 00       MOVS            R3, R6
.text:C1AE7528 57 F0 7A FD BL              AES_cbc_Encrypt_sub_CB602020 ; R0：原始数据,R1:返回,R2:大小,R3:key
.text:C1AE752C 06 98       LDR             R0, [SP,#0x120+byte_count]
.text:C1AE752E 03 99       LDR             R1, [SP,#0x120+var_114]
```

AES 加密压缩后数据

```
C37514E0  2E 5D 33 AF C8 C3 6B 6A  7F C2 9F F6 39 16 52 57  .]3...kj.......W
C37514F0  1D CB 01 FF A2 FA 48 0B  2D 2B 7E 39 73 EB 65 E7  ......H.-+~9s...
C3751500  14 93 59 D9 15 F2 D7 CB  C7 40 DC B5 9D 0D 34 16  ..Y......@....4.
C3751510  9C BF 69 C6 54 97 81 C8  69 7A 26 03 BF FA F3 D9  ..i......z&.....
C3751520  A3 D2 BE 38 B0 99 48 4A  5E 56 F0 C6 88 54 79 BD  .Ҿ 8..HJ^V....y.
C3751530  56 86 96 A7 40 D4 61 32  6D A4 07 55 F1 46 46 EF  V...@..2m..U....
C3751540  4E A1 CA 86 F8 1E 69 09  0B CF 6D 05 3B D3 0F B4  N.ʆ ..i.....;...
C3751550  63 78 65 6D AD F7 B2 C4  75 86 C5 35 B5 6F 42 BA  cxem.........oB.
C3751560  FA AF E7 4C 56 41 CF 36  46 86 3E E0 11 A3 35 9E  .....A..F.>...5.
```

加密后数据 (部分)

```
.text:C1ACFE62 27 DB       BLT             loc_C1ACFEB4
.text:C1ACFE64 00 25       MOVS            R5, #0
.text:C1ACFE66 3A 4A       LDR             R2, =(aAbcdefghijklmn - 0xC1ACFE6C) ; "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklm"...
.text:C1ACFE68 7A 44       ADD             R2, PC                  ; "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklm"...
.text:C1ACFE6A 01 9E       LDR             R6, [SP,#0x28+var_24]
.text:C1ACFE6C
.text:C1ACFE6C             loc_C1ACFE6C                            ; CODE XREF: base64_sub_C795AE10+A0↓j
.text:C1ACFE6C 43 5D       LDRB            R3, [R0,R5]             ; 加密后数据
.text:C1ACFE6E 9B 08       LSRS            R3, R3, #2
.text:C1ACFE70 D3 5C       LDRB            R3, [R2,R3]             ; base64 key
.text:C1ACFE72 33 70       STRB            R3, [R6]
.text:C1ACFE74 43 5D       LDRB            R3, [R0,R5]
.text:C1ACFE76 1B 01       LSLS            R3, R3, #4
.text:C1ACFE78 30 24       MOVS            R4, #0x30 ; '0'
.text:C1ACFE7A 1C 40       ANDS            R4, R3
.text:C1ACFE7C 43 19       ADDS            R3, R0, R5
.text:C1ACFE7E 08 00       MOVS            R0, R1
.text:C1ACFE80 59 78       LDRB            R1, [R3,#1]
.text:C1ACFE82 09 09       LSRS            R1, R1, #4
.text:C1ACFE84 21 43       ORRS            R1, R4
.text:C1ACFE86 51 5C       LDRB            R1, [R2,R1]
.text:C1ACFE88 71 70       STRB            R1, [R6,#1]
.text:C1ACFE8A 59 78       LDRB            R1, [R3,#1]
.text:C1ACFE8C 89 00       LSLS            R1, R1, #2
.text:C1ACFE8E 3C 24       MOVS            R4, #0x3C ; '<'
.text:C1ACFE90 0C 40       ANDS            R4, R1
.text:C1ACFE92 99 78       LDRB            R1, [R3,#2]
.text:C1ACFE94 89 09       LSRS            R1, R1, #6
.text:C1ACFE96 21 43       ORRS            R1, R4
.text:C1ACFE98 51 5C       LDRB            R1, [R2,R1]
.text:C1ACFE9A B1 70       STRB            R1, [R6,#2]
.text:C1ACFE9C 99 78       LDRB            R1, [R3,#2]
.text:C1ACFE9E 3F 23       MOVS            R3, #0x3F ; '?'
.text:C1ACFEA0 0B 40       ANDS            R3, R1
.text:C1ACFEA2 D1 5C       LDRB            R1, [R2,R3]
.text:C1ACFEA4 F1 70       STRB            R1, [R6,#3]
.text:C1ACFEA6 01 00       MOVS            R1, R0
.text:C1ACFEA8 04 98       LDR             R0, [SP,#0x28+var_18]
.text:C1ACFEAA 04 36       ADDS            R6, #4
.text:C1ACFEAC 03 35       ADDS            R5, #3
.text:C1ACFEAE 8D 42       CMP             R5, R1
.text:C1ACFEB0 DC DB       BLT             loc_C1ACFE6C            ; 加密后数据
.text:C1ACFEB2 02 9A       LDR             R2, [SP,#0x28+var_20]
.text:C1ACFEB4
.text:C1ACFEB4             loc_C1ACFEB4                            ; CODE XREF: base64_sub_C795AE10+52↑j
.text:C1ACFEB4 95 42       CMP             R5, R2
.text:C1ACFEB6 1F DA       BGE             loc_C1ACFEF8
.text:C1ACFEB8 41 5D       LDRB            R1, [R0,R5]
.text:C1ACFEBA 14 00       MOVS            R4, R2
.text:C1ACFEBC 8A 08       LSRS            R2, R1, #2
.text:C1ACFEBE 25 49       LDR             R1, =(aAbcdefghijklmn - 0xC1ACFEC4) ; "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklm"...
.text:C1ACFEC0 79 44       ADD             R1, PC                  ; "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklm"...
.text:C1ACFEC2 8A 5C       LDRB            R2, [R1,R2]
.text:C1ACFEC4 32 70       STRB            R2, [R6]
.text:C1ACFEC6 42 5D       LDRB            R2, [R0,R5]
.text:C1ACFEC8 13 01       LSLS            R3, R2, #4
.text:C1ACFECA 30 22       MOVS            R2, #0x30 ; '0'
.text:C1ACFECC 1A 40       ANDS            R2, R3
.text:C1ACFECE 63 1E       SUBS            R3, R4, #1
.text:C1ACFED0 9D 42       CMP             R5, R3
.text:C1ACFED2 14 D1       BNE             loc_C1ACFEFE
.text:C1ACFED4 89 5C       LDRB            R1, [R1,R2]
.text:C1ACFED6 71 70       STRB            R1, [R6,#1]
.text:C1ACFED8 3D 21       MOVS            R1, #0x3D ; '='
.text:C1ACFEDA 1B E0       B               loc_C1ACFF14
```

base64 加密

```
Ll0zr8jDa2p/wp/2ORZSVx3LAf+i+kgLLSt+OXPrZecUk1nZFfLXy8dA3LWdDTQWnL9pxlSXgchpeiYDv/rz2aPSvjiwmUhKXlbwxohUeb1WhpanQNRhMm2kB1XxRkbvTqHKhvgeaQkLz20FO9MPtGN4ZW2t97LEdYbFNbVvQrr6r+dMVkHPNkaGPuARozWeaQGSSMkoMMJ+ve9rA1L+aQTgGSootSpmPnT3TPLrCPN0Z6HzPKtpxopqfsjfcIV7YcP7M8Wc847wXfhMA2hYTRO24or97heGubGuxo8hQyw=
```

Base64 加密后

```
744d7275Ll0zr8jDa2p/wp/2ORZSVx3LAf+i+kgLLSt+OXPrZecUk1nZFfLXy8dA3LWdDTQWnL9pxlSXgchpeiYDv/rz2aPSvjiwmUhKXlbwxohUeb1WhpanQNRhMm2kB1XxRkbvTqHKhvgeaQkLz20FO9MPtGN4ZW2t97LEdYbFNbVvQrr6r+dMVkHPNkaGPuARozWeaQGSSMkoMMJ+ve9rA1L+aQTgGSootSpmPnT3TPLrCPN0Z6HzPKtpxopqfsjfcIV7YcP7M8Wc847wXfhMA2hYTRO24or97heGubGuxo8hQyw=
```

将 CRC 值与 Base64 加密后值组合,(签名时使用到)

```
{"voltage":"1","type":"user","brand":"google","psuc":"adb","temp":"1","suc":"midi,adb","id":"PQ2A.190305.002","sus":"midi,adb","tags":"release-keys","timestamp":"1631688160837","hardware":"sailfish","version":"9","model":"Pixel"}
```

#### 4.5、第二次加密设备信息

设备信息数据

```
C42FEF80  78 9C 55 8E 41 12 82 30  0C 45 EF D2 35 32 2D 8C  x.U.A..0.E...2-.
C42FEF90  08 EE BC 81 1E 21 D8 08  19 5B CA 34 80 32 8E 77  .  .!...[...2.w
C42FEFA0  37 15 37 AE 92 FF F2 93  FC 97 5A 82 9B A0 43 75  7.7.......Z...Cu
C42FEFB0  54 46 65 6A 5A C7 D4 CE  8C 51 54 1B 61 B0 22 BB  TFejZ..Ό QT.a.".
C42FEFC0  10 3A 87 02 46 9E AF A2  C1 B6 C9 8B 7E FC AD 6D  .:..F.....ɋ ~..m
C42FEFD0  D8 93 A5 6C 9B 51 5A 3B  5F 8A 53 6E 1A 5D EA 7D  ...l.QZ;_.Sn.]..
C42FEFE0  AE 75 F1 F5 F1 BF 4F 5E  27 12 D1 21 30 EE EE B8  .u....O^'...0...
C42FEFF0  72 A2 E4 91 27 D8 CE 57  A5 A9 EA DA 54 BA 2E 0F  r......W........
```

压缩数据

```
解密后得到key meituan1sankuai0
```

解密 PIC 数据获取 key(k1), 流程和上面一样

```
.text:C1AE7510 56 F0 30 FF BL              AES_set_Encrypt_key_sub_CB601374 ; R0:key,R1:长度,R2:返回值
.text:C1AE7514 06 9A       LDR             R2, [SP,#0x120+byte_count]
.text:C1AE7516 01 20       MOVS            R0, #1
.text:C1AE7518 69 46       MOV             R1, SP
.text:C1AE751A 04 9B       LDR             R3, [SP,#0x120+var_110]
.text:C1AE751C 0B 60       STR             R3, [R1,#0x120+var_120]
.text:C1AE751E 48 60       STR             R0, [R1,#0x120+var_11C]
.text:C1AE7520 05 98       LDR             R0, [SP,#0x120+p]
.text:C1AE7522 02 9C       LDR             R4, [SP,#0x120+var_118]
.text:C1AE7524 21 00       MOVS            R1, R4
.text:C1AE7526 33 00       MOVS            R3, R6
.text:C1AE7528 57 F0 7A FD BL              AES_cbc_Encrypt_sub_CB602020 ; R0：原始数据,R1:返回,R2:大小,R3:key
.text:C1AE752C 06 98       LDR             R0, [SP,#0x120+byte_count]
```

AES 加密压缩后数据

```
DEED6F00  2E 0F E3 72 62 1C 2B FB  35 B0 A9 CA E5 29 9F 79  ......+.5....).y
DEED6F10  C6 37 F0 02 CB 17 12 AE  6E 3D 5E AC EB A6 B5 DD  ........n=^.릵  .
DEED6F20  33 B9 24 A9 F4 E0 6D 6C  64 D6 76 E4 65 88 6C 6F  3.$.....d.....lo
DEED6F30  F3 53 20 FA B3 F8 02 6C  6D BC 75 C9 87 66 92 CE  .......lm.uɇ f..
DEED6F40  51 7A BC 09 BF 69 5D C0  C3 0A 71 77 46 5B E0 81  Qz...i]...qwF[..
DEED6F50  E2 15 67 05 5B 87 48 2D  0D D4 FA 99 EF 1C AC BD  ....[.H-........
DEED6F60  53 26 56 13 16 91 68 A8  EC 5B 1F D8 5F F4 61 BB  S&V...h.........
DEED6F70  FA 2A 2F 03 B5 3F 6E 02  6E 57 ED 46 8B A1 1A 0A  .*/..?n.nW......
DEED6F80  F5 C5 E7 38 9F E5 8E 70  B1 8C D6 CF E8 D5 16 6F  ...............o
DEED6F90  F6 36 3B A5 D8 2B 3D D9  83 BD 17 C5 3F AA 3D 96  ......=ك .....=.
```

加密后数据 (部分)

```
Lg/jcmIcK/s1sKnK5SmfecY38ALLFxKubj1erOumtd0zuSSp9OBtbGTWduRliGxv81Mg+rP4AmxtvHXJh2aSzlF6vAm/aV3Awwpxd0Zb4IHiFWcFW4dILQ3U+pnvHKy9UyZWExaRaKjsWx/YX/Rhu/oqLwO1P24CblftRouhGgr1xec4n+WOcLGM1s/o1RZv9jY7pdgrPdmDvRfFP6o9liGe0rCXBoG85J1mm/6GmqQ=
```

Base64 加密后

```
模板1
.text:B845707C 01 10 CE E3 BIC             R1, LR, #1
.text:B8457080 00 11 91 E7 LDR             R1, [R1,R0,LSL#2]
.text:B8457084 0E 10 81 E0 ADD             R1, R1, LR
.text:B8457088 08 E0 9D E5 LDR             LR, [SP,#arg_8]
.text:B845708C 08 10 8D E5 STR             R1, [SP,#arg_8]
.text:B8457090 03 80 BD E8 POP             {R0,R1,PC}

模板2
.text:C18BDAC0 7F B5       PUSH            {R0-R6,LR}
.text:C18BDAC2 7F B5       PUSH            {R0-R6,LR}
.text:C18BDAC4 0D 26       MOVS            R6, #0xD
.text:C18BDAC6 63 21       MOVS            R1, #0x63 ; 'c'
.text:C18BDAC8 68 46       MOV             R0, SP
.text:C18BDACA 10 30       ADDS            R0, #0x10
.text:C18BDACC 02 26       MOVS            R6, #2
.text:C18BDACE 08 30       ADDS            R0, #8
.text:C18BDAD0 01 1D       ADDS            R1, R0, #4
.text:C18BDAD2 8D 46       MOV             SP, R1
.text:C18BDAD4 04 A6 05 36 ADRL            R6, byte_C18BDAED
.text:C18BDAD8 02 21       MOVS            R1, #2
.text:C18BDADA 0A 36       ADDS            R6, #0xA
.text:C18BDADC 76 18       ADDS            R6, R6, R1
.text:C18BDADE 46 62       STR             R6, [R0,#0x24]
.text:C18BDAE0 40 BC       POP             {R6}
.text:C18BDAE2 01 E0       B               loc_C18BDAE8
.text:C18BDAE4             ;
.text:C18BDAE4 84 44       ADD             R12, R0
.text:C18BDAE6 12 34       ADDS            R4, #0x12
.text:C18BDAE8
.text:C18BDAE8             loc_C18BDAE8   
.text:C18BDAE8 00 46       MOV             R0, R0
.text:C18BDAEA 7F BD       POP             {R0-R6,PC}
```

#### 4.6、代码混淆

代码逻辑跳转动态计算：

```
.text:C1D46060             DecString
.text:C1D46060             ; __unwind { // 91B9000
.text:C1D46060 10 B5       PUSH            {R4,LR}
.text:C1D46062 01 2B       CMP             R3, #1
.text:C1D46064 08 DB       BLT             locret_C1D46078
.text:C1D46066
.text:C1D46066             loc_C1D46066
.text:C1D46066 0C 78       LDRB            R4, [R1]
.text:C1D46068 54 40       EORS            R4, R2
.text:C1D4606A 04 70       STRB            R4, [R0]
.text:C1D4606C 01 3B       SUBS            R3, #1
.text:C1D4606E 01 30       ADDS            R0, #1
.text:C1D46070 01 31       ADDS            R1, #1
.text:C1D46072 03 32       ADDS            R2, #3
.text:C1D46074 00 2B       CMP             R3, #0
.text:C1D46076 F6 D1       BNE             loc_C1D46066
.text:C1D46078
.text:C1D46078             locret_C1D46078
.text:C1D46078 10 BD       POP             {R4,PC}
```

字符串解密:

```
.text:C1AF41F0 00 2C       CMP             R4, #0
.text:C1AF41F2 01 DA       BGE             loc_C1AF41F8
.text:C1AF41F4 FC F7 C8 FF BL              loc_C1AF1188
.text:C1AF41F8
.text:C1AF41F8             loc_C1AF41F8                            ; CODE XREF: .text:C1AF41F2↑j
.text:C1AF41F8 03 B5       PUSH            {R0,R1,LR}
.text:C1AF41FA 01 48       LDR             R0, =0x4A
.text:C1AF41FC FC F7 F0 FE BL              ret_sub_CA67BFE0

.text:C1AF0FE0             ret_sub_CA67BFE0
.text:C1AF0FE0 
.text:C1AF0FE0 D8 F7 4C E8 BLX             jmp_sub_B288C07C
.text:C1AF0FE4 F0 1D       ADDS            R0, R6, #7
```

流程混淆:

```
.text:C1AF41F0 00 2C       CMP             R4, #0
.text:C1AF41F2 01 DA       BGE             loc_C1AF41F8
.text:C1AF41F4 FC F7 C8 FF BL              loc_C1AF1188
.text:C1AF41F8
.text:C1AF41F8             loc_C1AF41F8                            ; CODE XREF: .text:C1AF41F2↑j
.text:C1AF41F8 03 B5       PUSH            {R0,R1,LR}
.text:C1AF41FA 01 48       LDR             R0, =0x4A
.text:C1AF41FC FC F7 F0 FE BL              ret_sub_CA67BFE0
.text:C1AF0FE0             ret_sub_CA67BFE0
.text:C1AF0FE0 
.text:C1AF0FE0 D8 F7 4C E8 BLX             jmp_sub_B288C07C
.text:C1AF0FE4 F0 1D       ADDS            R0, R6, #7
```

作者简介：  
我是小三，目前从事软件安全相关工作，虽己工作多年，但内心依然有着执着的追求，信奉终身成长，不定义自己，热爱技术但不拘泥于技术，爱好分享，喜欢读书和乐于结交朋友，欢迎加我微信与我交朋友 (公众号输入框回复“wx” 即可)