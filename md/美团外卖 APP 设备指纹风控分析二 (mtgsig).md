> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HLzuQfnqRlh3vCihludkoQ)

### 五、反爬虫 mtgsig 签名

#### 5.1、基本流程

APP 每一个业务网络请求的请求头中都有 mtgsig 参数，该参数是请求体与其它参数计算的签名值。  
java 层代码如下：

```
public static String makeHeader(byte[] arg13, MODE arg14) {
        Object[] v8 = new Object[]{arg13, arg14};
        ChangeQuickRedirect v11 = WTSign.changeQuickRedirect;
        if(PatchProxy.isSupport(v8, null, v11, true, "4371e5fcb0c4ae2bd761bbf35c1a43bf", 0x6000000000000000L)) {
            return (String)PatchProxy.accessDispatch(v8, null, v11, true, "4371e5fcb0c4ae2bd761bbf35c1a43bf");
        }

        if(arg13 == null) {
            return "-2003";
        }

        Object[] v13 = NBridge.main3(2, new Object[]{MTGuard.sAppKey, arg13, ((int)arg14.mode)});
        if(v13 == null) {
            return "-1001";
        }

        return (v13[0] instanceof Integer) ? v13[0] : ((String)v13[0]);
    }
```

最终会走到 Native 层进行签名计算

#### 5.2、加密系统环境信息

```
// 是否root、ADB调试状态、USB调试模式等
{
  "b1": "{\"1\":\"\",\"2\":\"1|2|3\",\"3\":\"\",\"4\":\"\",\"5\":\"\",\"6\":\"\",\"7\":\"2\",\"8\":\"\",\"9\":\"\",\"10\":\"\",\"11\":\"\",\"12\":\"32\",\"13\":\"\",\"14\":\"\",\"15\":\"\",\"33\":{\"0\":2,\"1\":\"\",\"2\":\"\",\"3\":\"\",\"4\":\"2|22\",\"5\":\"\",\"6\":\"\",\"7\":\"\",\"8\":\"\",\"9\":\"\",\"10\":\"\",\"11\":\"\",\"12\":\"\",\"13\":\"\",\"14\":\"2\",\"15\":\"\"}}",
  "b2": 35,
  "b3": 0,
  "b4": "com.sankuai.meituan",
  "b5": "11.12.204",
  "b6": "1100120204",
  "b7": 1631754559
}
```

压缩信息

```
E53E7D80  78 9C 9D 91 C1 0E C2 20  10 44 FF 65 CF 0D 61 76  x........D.e..av
E53E7D90  81 2A DF C2 A5 35 1E 1A  D3 7A D0 9E DA FE BB 40  .*...5....О ...@
E53E7DA0  4D C0 A4 C6 C4 D3 3E 66  17 66 36 2C D4 83 3C 2D  M......f.f6,ԃ <-
E53E7DB0  81 10 C8 07 0A D4 04 E2  8C 58 79 95 7C 96 D2 32  ..........y.|...
E53E7DC0  05 6D 41 57 B0 CD C8 99  4F 45 3E 17 84 AE B8 72  .mAW....OE>....r
E53E7DD0  C5 6E 2B FB 5D 54 A6 A8  5C 51 D9 4A 9A 89 D1 D3  ..+.]T..\Q......
E53E7DE0  83 DC 1C AC 70 9C 9E 57  E6 9F 1B FC BF C0 B7 F8  ....p..W........
E53E7DF0  FC 99 7F DB A8 A1 9E C9  8B 8D 55 C8 EB 58 4C FC  .......ɋ .U..XL.
E53E7E00  8B CB 7D 54 8F 6E BA CD  DD A0 C6 EB F0 9C BB 29  ...T.n..........
E53E7E10  4D DA D8 02 14 58 B1 36  49 70 59 D0 1A AC DF 4A  M....X.6IpY.....
```

组合密钥

```
1631754963 9b69f861-e054-4bc4-9daf-d36ae205ed3e  //当前时间加APPkey
```

RC4 加密压缩后数据

```
.text:C881966C 11 99       LDR             R1, [SP,#0x44]
.text:C881966E 0E 9E       LDR             R6, [SP,#0x38]
.text:C8819670 8A 59       LDR             R2, [R1,R6]
.text:C8819672 16 A8       ADD             R0, SP, #0x58 ; 'X'
.text:C8819674 1A F0 B0 FF BL              initkey_sub_C79725D8    ; R1:key 15525971099b69f861-e054-4bc4-9daf-d36ae205ed3e R2:长度0x2E
.text:C8819678 10 9B       LDR             R3, [SP,#0x40]
.text:C881967A 29 00       MOVS            R1, R5
.text:C881967C 2A 00       MOVS            R2, R5
.text:C881967E 1A F0 B2 FF BL              RC4_sub_C79725E6        ; R0:初始化的KEY，R1：压缩后数据,R3:压缩后大小
.text:C8819682 11 98       LDR             R0, [SP,#0x44]
```

加密后

```
E53E7D80  62 16 1F 8D D5 8D AF 42  8B D2 2C 32 77 29 5A 5F  b...Ս .B...2w)Z_
E53E7D90  1B ED 38 9E F7 85 82 50  E0 FA D1 0A CD C4 6F 29  ..............o)
E53E7DA0  69 0C FA 10 AE 63 57 E1  11 EA A4 DF 12 36 B2 4D  i....cW......6.M
E53E7DB0  A0 A2 78 3B 5A 60 E6 AB  E9 4C A1 13 CD DB EB F0  ..x;Z`...L......
E53E7DC0  1B FC 49 D2 6C CE 5A 96  C6 6B 71 45 80 8B 5D B7  ..I.......qE..].
E53E7DD0  97 54 D5 58 0F F8 5E 68  25 CE 31 58 2D 04 C0 F2  .T....^h%..X-...
E53E7DE0  70 E7 D4 2E E7 C9 2C DD  07 F2 7A F4 CA 06 F3 CB  p...............
E53E7DF0  C3 CC 14 76 0A 44 2C 48  A2 35 6B 7D 0D 8C 51 60  ...v.D,H.5k}..Q`
E53E7E00  3F A9 F8 C6 D1 02 04 2B  A3 BF 86 3F 54 83 D4 43  ?......+...?T...
E53E7E10  39 9C AB 66 0D DD 21 90  2B 73 B9 1F C3 C2 B8 86  9..f....+s......
```

Base64 加密

```
An2sai6nXEuFYeKbDUk/qEo/7am8Jtn3O2Has5efofCux7iubGRCS8TKpjUgotJ6MGdQrBsvwh/peZwuikT+5rCr4RzN8SXXCeDOtZQ6sbo/snVdESSJweNqd6i/WbIwDOgv5eaWRQoISjdgNfW3hk7tf0QhsZFbOELcPxz5pRzo6d6EsBLFu5Mq8DbVNgsYF+6aqZ7302/G+Rr7MlUT2M9y3EjgjH01L39q/eRJ
```

#### 5.3、获取 dfpid (设备指纹)

判断本地是否有存储，如果有优先读取本地，如果无反谢 java 层从服务器端获取，这部分详细分析见后面设备指纹部分。

```
.text:C8A93C78 F0 B5       PUSH            {R4-R7,LR}
.text:C8A93C7A 03 AF       ADD             R7, SP, #0xC
.text:C8A93C7C 91 B0       SUB             SP, SP, #0x44
.text:C8A93C7E 09 92       STR             R2, [SP,#0x50+var_2C]
.text:C8A93C80 0A 91       STR             R1, [SP,#0x50+var_28]
.text:C8A93C82 01 B4       PUSH            {R0}
.text:C8A93C84 10 BC       POP             {R4}
.text:C8A93C86 8E 48       LDR             R0, =(__stack_chk_guard_ptr - 0xC8A93C8C)
.text:C8A93C88 78 44       ADD             R0, PC                  ; __stack_chk_guard_ptr
.text:C8A93C8A 05 68       LDR             R5, [R0]                ; __stack_chk_guard
.text:C8A93C8C 28 68       LDR             R0, [R5]
.text:C8A93C8E 10 90       STR             R0, [SP,#0x50+var_10]
.text:C8A93C90 20 00       MOVS            R0, R4
.text:C8A93C92 00 F0 3F F9 BL              getClassLoader_sub_C6FD9F14
.text:C8A93C96 00 26       MOVS            R6, #0
.text:C8A93C98 00 28       CMP             R0, #0
.text:C8A93C9A 00 D1       BNE             loc_C8A93C9E
.text:C8A93C9C 06 E1       B               loc_C8A93EAC
.text:C8A93C9E             ; ---------------------------------------------------------------------------
.text:C8A93C9E
.text:C8A93C9E             loc_C8A93C9E                            ; CODE XREF: main2_sub_B6891C78+22↑j
.text:C8A93C9E 0B 90       STR             R0, [SP,#0x50+var_24]
.text:C8A93CA0 F6 F7 EE FC BL              malloc_sub_C8BC6680
.text:C8A93CA4 9C 21       MOVS            R1, #0x9C
.text:C8A93CA6 41 58       LDR             R1, [R0,R1]             ; char *
.text:C8A93CA8 0E AE       ADD             R6, SP, #0x50+var_18
.text:C8A93CAA 0D AA       ADD             R2, SP, #0x50+var_1C
.text:C8A93CAC 30 00       MOVS            R0, R6                  ; int
.text:C8A93CAE 6F F0 F7 FF BL              basic_string_sub_B2943CA0
.text:C8A93CB2 20 00       MOVS            R0, R4
.text:C8A93CB4 31 00       MOVS            R1, R6
.text:C8A93CB6 03 F0 35 FF BL              NewStringUTF_sub_C6B9DB24
.text:C8A93CBA 26 00       MOVS            R6, R4
.text:C8A93CBC 04 00       MOVS            R4, R0
.text:C8A93CBE 0E 98       LDR             R0, [SP,#0x50+var_18]
.text:C8A93CC0 0C 38       SUBS            R0, #0xC
.text:C8A93CC2 0F A9       ADD             R1, SP, #0x50+var_14
.text:C8A93CC4 6F F0 40 FD BL              free_sub_B2943748
.text:C8A93CC8 0C 96       STR             R6, [SP,#0x50+var_20]
.text:C8A93CCA 30 00       MOVS            R0, R6
.text:C8A93CCC 07 F0 FC FD BL              ExceptionCheck_sub_C6BA18C8
.text:C8A93CD0 00 26       MOVS            R6, #0
.text:C8A93CD2 00 28       CMP             R0, #0
.text:C8A93CD4 07 D0       BEQ             loc_C8A93CE6
.text:C8A93CD6 00 2C       CMP             R4, #0
.text:C8A93CD8 00 D1       BNE             loc_C8A93CDC
.text:C8A93CDA E2 E0       B               loc_C8A93EA2
.text:C8A93CDC             ; ---------------------------------------------------------------------------
.text:C8A93CDC
.text:C8A93CDC             loc_C8A93CDC                            ; CODE XREF: main2_sub_B6891C78+60↑j
.text:C8A93CDC 08 94       STR             R4, [SP,#0x50+var_30]
.text:C8A93CDE 07 95       STR             R5, [SP,#0x50+var_34]
.text:C8A93CE0 00 26       MOVS            R6, #0
.text:C8A93CE2 0C 9C       LDR             R4, [SP,#0x50+var_20]
.text:C8A93CE4 D7 E0       B               loc_C8A93E96
.text:C8A93CE6             ; ---------------------------------------------------------------------------
.text:C8A93CE6
.text:C8A93CE6             loc_C8A93CE6                            ; CODE XREF: main2_sub_B6891C78+5C↑j
.text:C8A93CE6 00 2C       CMP             R4, #0
.text:C8A93CE8 00 D1       BNE             loc_C8A93CEC
.text:C8A93CEA DA E0       B               loc_C8A93EA2
.text:C8A93CEC             ; ---------------------------------------------------------------------------
.text:C8A93CEC
.text:C8A93CEC             loc_C8A93CEC                            ; CODE XREF: main2_sub_B6891C78+70↑j
.text:C8A93CEC 08 94       STR             R4, [SP,#0x50+var_30]
.text:C8A93CEE 07 95       STR             R5, [SP,#0x50+var_34]
.text:C8A93CF0 74 4E       LDR             R6, =(byte_C8B2D4E0 - 0xC8A93CF6)
.text:C8A93CF2 7E 44       ADD             R6, PC                  ; byte_C8B2D4E0
.text:C8A93CF4 30 78       LDRB            R0, [R6]
.text:C8A93CF6 00 28       CMP             R0, #0
.text:C8A93CF8 0A D1       BNE             loc_C8A93D10
.text:C8A93CFA 73 4C       LDR             R4, =(aJavaLangClassl - 0xC8A93D00) ; "java/lang/ClassLoader"
.text:C8A93CFC 7C 44       ADD             R4, PC                  ; "java/lang/ClassLoader"
.text:C8A93CFE 73 49       LDR             R1, =(unk_C8B1E8A0 - 0xC8A93D04)
.text:C8A93D00 79 44       ADD             R1, PC                  ; unk_C8B1E8A0
.text:C8A93D02 00 25       MOVS            R5, #0
.text:C8A93D04 15 23       MOVS            R3, #0x15
.text:C8A93D06 20 00       MOVS            R0, R4
.text:C8A93D08 2A 00       MOVS            R2, R5
.text:C8A93D0A F7 F7 A9 F9 BL              DecString
.text:C8A93D0E 65 75       STRB            R5, [R4,#(aJavaLangClassl+0x15 - 0xC8B1E489)] ; ""
.text:C8A93D10
.text:C8A93D10             loc_C8A93D10                            ; CODE XREF: main2_sub_B6891C78+80↑j
.text:C8A93D10 01 20       MOVS            R0, #1
.text:C8A93D12 06 90       STR             R0, [SP,#0x50+var_38]
.text:C8A93D14 30 70       STRB            R0, [R6]
.text:C8A93D16 0C 9C       LDR             R4, [SP,#0x50+var_20]
.text:C8A93D18 20 68       LDR             R0, [R4]
.text:C8A93D1A 82 69       LDR             R2, [R0,#0x18]
.text:C8A93D1C 6C 49       LDR             R1, =(aJavaLangClassl - 0xC8A93D22) ; "java/lang/ClassLoader"
.text:C8A93D1E 79 44       ADD             R1, PC                  ; "java/lang/ClassLoader"
.text:C8A93D20 20 00       MOVS            R0, R4
.text:C8A93D22 90 47       BLX             R2
.text:C8A93D24 05 00       MOVS            R5, R0
.text:C8A93D26 20 00       MOVS            R0, R4
.text:C8A93D28 07 F0 CE FD BL              ExceptionCheck_sub_C6BA18C8
.text:C8A93D2C 00 26       MOVS            R6, #0
.text:C8A93D2E 00 28       CMP             R0, #0
.text:C8A93D30 05 D0       BEQ             loc_C8A93D3E
.text:C8A93D32 00 2D       CMP             R5, #0
.text:C8A93D34 20 B4       PUSH            {R5}
.text:C8A93D36 02 BC       POP             {R1}
.text:C8A93D38 00 D0       BEQ             loc_C8A93D3C
.text:C8A93D3A A8 E0       B               loc_C8A93E8E
.text:C8A93D3C             ; ---------------------------------------------------------------------------
.text:C8A93D3C
.text:C8A93D3C             loc_C8A93D3C                            ; CODE XREF: main2_sub_B6891C78+C0↑j
.text:C8A93D3C AB E0       B               loc_C8A93E96
.text:C8A93D3E             ; ---------------------------------------------------------------------------
.text:C8A93D3E
.text:C8A93D3E             loc_C8A93D3E                            ; CODE XREF: main2_sub_B6891C78+B8↑j
.text:C8A93D3E 00 2D       CMP             R5, #0
.text:C8A93D40 00 D1       BNE             loc_C8A93D44
.text:C8A93D42 A8 E0       B               loc_C8A93E96
.text:C8A93D44             ; ---------------------------------------------------------------------------
.text:C8A93D44
.text:C8A93D44             loc_C8A93D44                            ; CODE XREF: main2_sub_B6891C78+C8↑j
.text:C8A93D44 05 95       STR             R5, [SP,#0x50+var_3C]
.text:C8A93D46 63 4E       LDR             R6, =(byte_C8B2D4E1 - 0xC8A93D4C)
.text:C8A93D48 7E 44       ADD             R6, PC                  ; byte_C8B2D4E1
.text:C8A93D4A 30 78       LDRB            R0, [R6]
.text:C8A93D4C 00 28       CMP             R0, #0
.text:C8A93D4E 0A D1       BNE             loc_C8A93D66
.text:C8A93D50 61 4D       LDR             R5, =(aLoadclass - 0xC8A93D56) ; "loadClass"
.text:C8A93D52 7D 44       ADD             R5, PC                  ; "loadClass"
.text:C8A93D54 61 49       LDR             R1, =(aMkfnnRej - 0xC8A93D5A) ; "mkfnN|rej"
.text:C8A93D56 79 44       ADD             R1, PC                  ; "mkfnN|rej"
.text:C8A93D58 01 22       MOVS            R2, #1
.text:C8A93D5A 09 23       MOVS            R3, #9
.text:C8A93D5C 28 00       MOVS            R0, R5
.text:C8A93D5E F7 F7 7F F9 BL              DecString
.text:C8A93D62 00 20       MOVS            R0, #0
.text:C8A93D64 68 72       STRB            R0, [R5,#(aLoadclass+9 - 0xC8B1E49F)] ; ""
.text:C8A93D66
.text:C8A93D66             loc_C8A93D66                            ; CODE XREF: main2_sub_B6891C78+D6↑j
.text:C8A93D66 06 9D       LDR             R5, [SP,#0x50+var_38]
.text:C8A93D68 35 70       STRB            R5, [R6]
.text:C8A93D6A 5D 4E       LDR             R6, =(byte_C8B2D4E2 - 0xC8A93D70)
.text:C8A93D6C 7E 44       ADD             R6, PC                  ; byte_C8B2D4E2
.text:C8A93D6E 30 78       LDRB            R0, [R6]
.text:C8A93D70 00 28       CMP             R0, #0
.text:C8A93D72 0E D1       BNE             loc_C8A93D92
.text:C8A93D74 5B 48       LDR             R0, =(aLjavaLangStrin - 0xC8A93D7A) ; "(Ljava/lang/String;)Ljava/lang/Class;"
.text:C8A93D76 78 44       ADD             R0, PC                  ; "(Ljava/lang/String;)Ljava/lang/Class;"
.text:C8A93D78 03 90       STR             R0, [SP,#0x50+var_44]
.text:C8A93D7A 5B 49       LDR             R1, =(unk_C8B1E8C0 - 0xC8A93D80)
.text:C8A93D7C 79 44       ADD             R1, PC                  ; unk_C8B1E8C0
.text:C8A93D7E 02 22       MOVS            R2, #2
.text:C8A93D80 25 23       MOVS            R3, #0x25 ; '%'
.text:C8A93D82 04 93       STR             R3, [SP,#0x50+var_40]
.text:C8A93D84 04 9B       LDR             R3, [SP,#0x50+var_40]
.text:C8A93D86 F7 F7 6B F9 BL              DecString
.text:C8A93D8A 00 20       MOVS            R0, #0
.text:C8A93D8C 04 99       LDR             R1, [SP,#0x50+var_40]
.text:C8A93D8E 03 9A       LDR             R2, [SP,#0x50+var_44]
.text:C8A93D90 50 54       STRB            R0, [R2,R1]
.text:C8A93D92
.text:C8A93D92             loc_C8A93D92                            ; CODE XREF: main2_sub_B6891C78+FA↑j
.text:C8A93D92 35 70       STRB            R5, [R6]
.text:C8A93D94 20 68       LDR             R0, [R4]
.text:C8A93D96 84 21       MOVS            R1, #0x84
.text:C8A93D98 46 58       LDR             R6, [R0,R1]
.text:C8A93D9A 54 4A       LDR             R2, =(aLoadclass - 0xC8A93DA0) ; "loadClass"
.text:C8A93D9C 7A 44       ADD             R2, PC                  ; "loadClass"
.text:C8A93D9E 54 4B       LDR             R3, =(aLjavaLangStrin - 0xC8A93DA4) ; "(Ljava/lang/String;)Ljava/lang/Class;"
.text:C8A93DA0 7B 44       ADD             R3, PC                  ; "(Ljava/lang/String;)Ljava/lang/Class;"
.text:C8A93DA2 20 00       MOVS            R0, R4
.text:C8A93DA4 05 99       LDR             R1, [SP,#0x50+var_3C]
.text:C8A93DA6 B0 47       BLX             R6
.text:C8A93DA8 05 00       MOVS            R5, R0
.text:C8A93DAA 20 00       MOVS            R0, R4
.text:C8A93DAC 07 F0 8C FD BL              ExceptionCheck_sub_C6BA18C8
.text:C8A93DB0 00 26       MOVS            R6, #0
.text:C8A93DB2 00 28       CMP             R0, #0
.text:C8A93DB4 6A D1       BNE             loc_C8A93E8C
.text:C8A93DB6 00 2D       CMP             R5, #0
.text:C8A93DB8 68 D0       BEQ             loc_C8A93E8C
.text:C8A93DBA 20 00       MOVS            R0, R4
.text:C8A93DBC 0B 99       LDR             R1, [SP,#0x50+var_24]
.text:C8A93DBE 2A 00       MOVS            R2, R5
.text:C8A93DC0 08 9B       LDR             R3, [SP,#0x50+var_30]
.text:C8A93DC2 00 F0 13 F9 BL              CallObjectMethodV_sub_C6B99FEC
.text:C8A93DC6 05 00       MOVS            R5, R0
.text:C8A93DC8 20 00       MOVS            R0, R4
.text:C8A93DCA 07 F0 7D FD BL              ExceptionCheck_sub_C6BA18C8
.text:C8A93DCE 00 28       CMP             R0, #0
.text:C8A93DD0 04 D0       BEQ             loc_C8A93DDC
.text:C8A93DD2 00 26       MOVS            R6, #0
.text:C8A93DD4 00 2D       CMP             R5, #0
.text:C8A93DD6 05 99       LDR             R1, [SP,#0x50+var_3C]
.text:C8A93DD8 53 D1       BNE             loc_C8A93E82
.text:C8A93DDA 58 E0       B               loc_C8A93E8E
.text:C8A93DDC             ; ---------------------------------------------------------------------------
.text:C8A93DDC
.text:C8A93DDC             loc_C8A93DDC                            ; CODE XREF: main2_sub_B6891C78+158↑j
.text:C8A93DDC 00 2D       CMP             R5, #0
.text:C8A93DDE 05 99       LDR             R1, [SP,#0x50+var_3C]
.text:C8A93DE0 55 D0       BEQ             loc_C8A93E8E
.text:C8A93DE2 04 95       STR             R5, [SP,#0x50+var_40]
.text:C8A93DE4 43 4E       LDR             R6, =(byte_C8B2D4E3 - 0xC8A93DEA)
.text:C8A93DE6 7E 44       ADD             R6, PC                  ; byte_C8B2D4E3
.text:C8A93DE8 30 78       LDRB            R0, [R6]
.text:C8A93DEA 00 28       CMP             R0, #0
.text:C8A93DEC 0A D1       BNE             loc_C8A93E04
.text:C8A93DEE 42 4D       LDR             R5, =(aMain2 - 0xC8A93DF4) ; "main2"
.text:C8A93DF0 7D 44       ADD             R5, PC                  ; "main2"
.text:C8A93DF2 42 49       LDR             R1, =(aNgB - 0xC8A93DF8) ; "ng`b="
.text:C8A93DF4 79 44       ADD             R1, PC                  ; "ng`b="
.text:C8A93DF6 03 22       MOVS            R2, #3
.text:C8A93DF8 05 23       MOVS            R3, #5
.text:C8A93DFA 28 00       MOVS            R0, R5
.text:C8A93DFC F7 F7 30 F9 BL              DecString
.text:C8A93E00 00 20       MOVS            R0, #0
.text:C8A93E02 68 71       STRB            R0, [R5,#(aMain2+5 - 0xC8B1E4CF)] ; ""
.text:C8A93E04
.text:C8A93E04             loc_C8A93E04                            ; CODE XREF: main2_sub_B6891C78+174↑j
.text:C8A93E04 06 9D       LDR             R5, [SP,#0x50+var_38]
.text:C8A93E06 35 70       STRB            R5, [R6]
.text:C8A93E08 3D 4E       LDR             R6, =(byte_C8B2D4E4 - 0xC8A93E0E)
.text:C8A93E0A 7E 44       ADD             R6, PC                  ; byte_C8B2D4E4
.text:C8A93E0C 30 78       LDRB            R0, [R6]
.text:C8A93E0E 00 28       CMP             R0, #0
.text:C8A93E10 0E D1       BNE             loc_C8A93E30
.text:C8A93E12 3C 48       LDR             R0, =(aILjavaLangObje_0 - 0xC8A93E18) ; "(I[Ljava/lang/Object;)Ljava/lang/Object"...
.text:C8A93E14 78 44       ADD             R0, PC                  ; "(I[Ljava/lang/Object;)Ljava/lang/Object"...
.text:C8A93E16 02 90       STR             R0, [SP,#0x50+var_48]
.text:C8A93E18 3B 49       LDR             R1, =(unk_C8B1E8F0 - 0xC8A93E1E)
.text:C8A93E1A 79 44       ADD             R1, PC                  ; unk_C8B1E8F0
.text:C8A93E1C 04 22       MOVS            R2, #4
.text:C8A93E1E 28 23       MOVS            R3, #0x28 ; '('
.text:C8A93E20 03 93       STR             R3, [SP,#0x50+var_44]
.text:C8A93E22 03 9B       LDR             R3, [SP,#0x50+var_44]
.text:C8A93E24 F7 F7 1C F9 BL              DecString
.text:C8A93E28 00 20       MOVS            R0, #0
.text:C8A93E2A 03 99       LDR             R1, [SP,#0x50+var_44]
.text:C8A93E2C 02 9A       LDR             R2, [SP,#0x50+var_48]
.text:C8A93E2E 50 54       STRB            R0, [R2,R1]
.text:C8A93E30
.text:C8A93E30             loc_C8A93E30                            ; CODE XREF: main2_sub_B6891C78+198↑j
.text:C8A93E30 35 70       STRB            R5, [R6]
.text:C8A93E32 71 20 80 00 MOVS            R0, #0x1C4
.text:C8A93E36 21 68       LDR             R1, [R4]
.text:C8A93E38 0D 58       LDR             R5, [R1,R0]
.text:C8A93E3A 34 4A       LDR             R2, =(aMain2 - 0xC8A93E40) ; "main2"
.text:C8A93E3C 7A 44       ADD             R2, PC                  ; "main2"
.text:C8A93E3E 34 4B       LDR             R3, =(aILjavaLangObje_0 - 0xC8A93E44) ; "(I[Ljava/lang/Object;)Ljava/lang/Object"...
.text:C8A93E40 7B 44       ADD             R3, PC                  ; "(I[Ljava/lang/Object;)Ljava/lang/Object"...
.text:C8A93E42 20 00       MOVS            R0, R4
.text:C8A93E44 04 99       LDR             R1, [SP,#0x50+var_40]
.text:C8A93E46 A8 47       BLX             R5
.text:C8A93E48 05 00       MOVS            R5, R0
.text:C8A93E4A 20 00       MOVS            R0, R4
.text:C8A93E4C 07 F0 3C FD BL              ExceptionCheck_sub_C6BA18C8
.text:C8A93E50 00 26       MOVS            R6, #0
.text:C8A93E52 00 28       CMP             R0, #0
.text:C8A93E54 14 D1       BNE             loc_C8A93E80
.text:C8A93E56 2A 00       MOVS            R2, R5
.text:C8A93E58 00 2D       CMP             R5, #0
.text:C8A93E5A 04 9D       LDR             R5, [SP,#0x50+var_40]
.text:C8A93E5C 11 D0       BEQ             loc_C8A93E82
.text:C8A93E5E 68 46       MOV             R0, SP
.text:C8A93E60 09 99       LDR             R1, [SP,#0x50+var_2C]
.text:C8A93E62 01 60       STR             R1, [R0,#0x50+var_50]
.text:C8A93E64 20 00       MOVS            R0, R4
.text:C8A93E66 29 00       MOVS            R1, R5
.text:C8A93E68 0A 9B       LDR             R3, [SP,#0x50+var_28]
.text:C8A93E6A 00 F0 DD F8 BL              CallStaticObjectMethodV_sub_B6892028
.text:C8A93E6E 0A 90       STR             R0, [SP,#0x50+var_28]
.text:C8A93E70 20 00       MOVS            R0, R4
.text:C8A93E72 07 F0 29 FD BL              ExceptionCheck_sub_C6BA18C8
.text:C8A93E76 00 26       MOVS            R6, #0
.text:C8A93E78 00 28       CMP             R0, #0
.text:C8A93E7A 02 D1       BNE             loc_C8A93E82
.text:C8A93E7C 0A 9E       LDR             R6, [SP,#0x50+var_28]
.text:C8A93E7E 00 E0       B               loc_C8A93E82
.text:C8A93E80             ; ---------------------------------------------------------------------------
.text:C8A93E80
.text:C8A93E80             loc_C8A93E80                            ; CODE XREF: main2_sub_B6891C78+1DC↑j
.text:C8A93E80 04 9D       LDR             R5, [SP,#0x50+var_40]
.text:C8A93E82
.text:C8A93E82             loc_C8A93E82                            ; CODE XREF: main2_sub_B6891C78+160↑j
.text:C8A93E82                                                     ; main2_sub_B6891C78+1E4↑j ...
.text:C8A93E82 20 68       LDR             R0, [R4]
.text:C8A93E84 C2 6D       LDR             R2, [R0,#0x5C]
.text:C8A93E86 20 00       MOVS            R0, R4
.text:C8A93E88 29 00       MOVS            R1, R5
.text:C8A93E8A 90 47       BLX             R2
.text:C8A93E8C
.text:C8A93E8C             loc_C8A93E8C                            ; CODE XREF: main2_sub_B6891C78+13C↑j
.text:C8A93E8C                                                     ; main2_sub_B6891C78+140↑j
.text:C8A93E8C 05 99       LDR             R1, [SP,#0x50+var_3C]
```

获取到的 dfpid 如下:

```
DAD796C46B5A6525F4B89DF661A97C7A218A219FC24B93F689DEBD92
```

#### 5.4、获取 xid (设备指纹)

判断本地是否有存储，如果有优先读取本地，如果无反谢 java 层从服务器端获取，APP 第一次运行进就用 UUID 与时间加密生成一个，这部分详细分析见后面设备指纹部分。

```
.text:C8AEE3A4 F0 B5       PUSH            {R4-R7,LR}
.text:C8AEE3A6 03 AF       ADD             R7, SP, #0xC
.text:C8AEE3A8 81 B0       SUB             SP, SP, #4
.text:C8AEE3AA 0C 00       MOVS            R4, R1
.text:C8AEE3AC 06 00       MOVS            R6, R0
.text:C8AEE3AE 00 23       MOVS            R3, #0
.text:C8AEE3B0 20 00       MOVS            R0, R4
.text:C8AEE3B2 11 00       MOVS            R1, R2
.text:C8AEE3B4 1A 00       MOVS            R2, R3
.text:C8AEE3B6 A5 F7 5F FC BL              main2_sub_B6891C78 ; 反射调用java层
.text:C8AEE3BA 05 00       MOVS            R5, R0
.text:C8AEE3BC 20 00       MOVS            R0, R4
.text:C8AEE3BE AD F7 83 FA BL              ExceptionCheck_sub_C6BA18C8
.text:C8AEE3C2 00 28       CMP             R0, #0
.text:C8AEE3C4 09 D0       BEQ             loc_C8AEE3DA
.text:C8AEE3C6 9C F7 5B F9 BL              malloc_sub_C8BC6680
.text:C8AEE3CA 01 00       MOVS            R1, R0
.text:C8AEE3CC 7C 31       ADDS            R1, #0x7C ; '|'
.text:C8AEE3CE 30 00       MOVS            R0, R6
.text:C8AEE3D0 15 F0 30 FA BL              empty_sub_C6C09834
.text:C8AEE3D4 00 2D       CMP             R5, #0
.text:C8AEE3D6 07 D1       BNE             loc_C8AEE3E8
.text:C8AEE3D8 13 E0       B               loc_C8AEE402
.text:C8AEE3DA             ; ---------------------------------------------------------------------------
.text:C8AEE3DA
.text:C8AEE3DA             loc_C8AEE3DA                            ; CODE XREF: main2_sub_B292E3A4+20↑j
.text:C8AEE3DA 00 2D       CMP             R5, #0
.text:C8AEE3DC 0A D0       BEQ             loc_C8AEE3F4
.text:C8AEE3DE 30 00       MOVS            R0, R6
.text:C8AEE3E0 21 00       MOVS            R1, R4
.text:C8AEE3E2 2A 00       MOVS            R2, R5
.text:C8AEE3E4 A9 F7 CE FC BL              String_sub_B28D7D84     ; 出现字符串
```

获取到的 xid 如下:

```
Rs8NOy0BFS5JQxfdOoIxpMnKV3iqYWcblAjp0vpnWZyNzyF9rfsi3ekpm4ScaIZgeImizX/5AbS3e838Or4el4+PPPI2kD8XW+8vbvjDBSM=
```

组合 json

```
{
  "a0": "2.0",
  "a1": "9b69f861-e054-4bc4-9daf-d36ae205ed3e",
  "a3": 2,
  "a4": 1631754963,
  "a5": "An2sai6nXEuFYeKbDUk/qEo/7am8Jtn3O2Has5efofCux7iubGRCS8TKpjUgotJ6MGdQrBsvwh/peZwuikT+5rCr4RzN8SXXCeDOtZQ6sbo/snVdESSJweNqd6i/WbIwDOgv5eaWRQoISjdgNfW3hk7tf0QhsZFbOELcPxz5pRzo6d6EsBLFu5Mq8DbVNgsYF+6aqZ7302/G+Rr7MlUT2M9y3EjgjH01L39q/eRJ",
  "a6": 1025,
  "a7": "Rs8NOy0BFS5JQxfdOoIxpMnKV3iqYWcblAjp0vpnWZyNzyF9rfsi3ekpm4ScaIZgeImizX/5AbS3e838Or4el4+PPPI2kD8XW+8vbvjDBSM=",
  "a8": "DAD796C46B5A6525F4B89DF661A97C7A218A219FC24B93F689DEBD92",
  "a9": "f4ec12efNkQBqdtVlV78x1/Mln2Us/xw171NuJjdEXrGWsFDdMV5Te45wqjL0nPO8OFFKjvKthvva+lS9xqhMhSt1WZRjDYpsWc/eh3z4F2JTv3MOh8NEDmk7Frthx5/bczdDIKvRP0QneTfNKSm116fUjdLhOEYlNbym/xI+5jZAEJGfGjltFLEOmtOwgxasgHQh2woMl/vAyr7ePuoVC6wEcbv2+w6n/+Pl1U1KO2YcTw4peiZDqC7iHpTVQH4fWri9R5+Ev1zx/xObVqoxqe3TEW/t2EIRqQ4QoRZRix0xC6C280faz8U5vOqafUnm+qev7tjs7SOV4SNxBv+LEJTxr5IJU302FJEk/CqhKoz5eWRYtT5Z52kEanlfu4AGHcJLC343kpI3GxYw7uPeewA/Ye0qDgZUyfj0MPpaYMPj0UmtvnbXEU4+FaRaCb/LsQtWdOtEiEKveUQU9bTW4NfHch2+6gcHP2/E+UQSlREX67PPa9XN8tgL4H6qzghiE1NL3gYw0rrjzEXiO6jsjvIdzwDjeab9woJyr8W3xSACz3sezUS+AJAKohnJvlQkFM9cdG3lPYS7gByAK++K2/vI714kxJHqCZQQMbVNsoWj5w64YL+sE1A4byzOPgK71oPb9w6Cb+KwVlDtqH8F5vlVkO23Iq8E28BI2vQuq5TLRzMunjo45Ks2Py9ZZuHkGb+QIVpUz6ViB+JlUONKinIJF1p6g==",
  "a10": "{}",
  "x0": 1
}
```

每一个字段的解释：

```
a0:版本，a1:appkey, a3:版本,a4:时间,a5:加密的设备环境,a6:固定数字,a7:xid,a8:dfpid,a9:初始化时加密的设备环境信息(前8字符中CRC:f4ec12ef,因为我换手机了所以计算的值会不一样)
```

#### 5.5、计算请求体签名值

获取请求体数据

```
.text:C8A8FA4C 29 68       LDR             R1, [R5]
.text:C8A8FA4E 0A 58       LDR             R2, [R1,R0]
.text:C8A8FA50 28 00       MOVS            R0, R5
.text:C8A8FA52 31 00       MOVS            R1, R6
.text:C8A8FA54 90 47       BLX             R2                      ; 获取body长度
.text:C8A8FA56 0A 90       STR             R0, [SP,#0x28]
.text:C8A8FA58 28 00       MOVS            R0, R5
.text:C8A8FA5A 0B F0 35 FF BL              ExceptionCheck_sub_C6BA18C8
.text:C8A8FA5E 00 28       CMP             R0, #0

.text:C8A8FAF0 00 00       MOVS            R0, R0
.text:C8A8FAF2 00 00       MOVS            R0, R0
.text:C8A8FAF4 D8 74       STRB            R0, [R3,#0x13]
.text:C8A8FAF6 86 1B       SUBS            R6, R0, R6
.text:C8A8FAF8 10 99       LDR             R1, [SP,#0xC+arg_34]    ; 获取body
.text:C8A8FAFA 0B 20       MOVS            R0, #0xB
.text:C8A8FAFC C2 43       MVNS            R2, R0
.text:C8A8FAFE 03 91       STR             R1, [SP,#0xC+arg_0]
.text:C8A8FB00 08 68       LDR             R0, [R1]
.text:C8A8FB02 02 92       STR             R2, [SP,#0xC+var_4]
.text:C8A8FB04 80 58       LDR             R0, [R0,R2]
.text:C8A8FB06 0A 9C       LDR             R4, [SP,#0xC+arg_1C]
.text:C8A8FB08 00 19       ADDS            R0, R0, R4
.text:C8A8FB0A 04 90       STR             R0, [SP,#0xC+arg_4]
.text:C8A8FB0C 46 1C       ADDS            R6, R0, #1
.text:C8A8FB0E 30 00       MOVS            R0, R6
.text:C8A8FB10 FA F7 EA E8 BLX             malloc                  ; 分配body存储空间
.text:C8A8FB14 00 21       MOVS            R1, #0
.text:C8A8FB16 06 91       STR             R1, [SP,#0xC+arg_C]
.text:C8A8FB18 32 00       MOVS            R2, R6
.text:C8A8FB1A 06 00       MOVS            R6, R0
.text:C8A8FB1C 20 F0 D0 FC BL              memset_sub_C6BB64C0
.text:C8A8FB20 19 20 40 01 MOVS            R0, #0x320
.text:C8A8FB24 29 68       LDR             R1, [R5]
.text:C8A8FB26 08 58       LDR             R0, [R1,R0]
.text:C8A8FB28 05 90       STR             R0, [SP,#0xC+arg_8]
.text:C8A8FB2A 68 46       MOV             R0, SP
.text:C8A8FB2C 06 60       STR             R6, [R0,#0xC+var_C]
.text:C8A8FB2E 28 00       MOVS            R0, R5
.text:C8A8FB30 07 99       LDR             R1, [SP,#0xC+arg_10]
.text:C8A8FB32 06 9A       LDR             R2, [SP,#0xC+arg_C]
.text:C8A8FB34 23 00       MOVS            R3, R4
.text:C8A8FB36 05 9C       LDR             R4, [SP,#0xC+arg_8]
.text:C8A8FB38 A0 47       BLX             R4                      ; GetByteArrayRegion
.text:C8A8FB38                                                     ; 获取要计算签名的body值
.text:C8A8FB3A 28 00       MOVS            R0, R5
.text:C8A8FB3C 0B F0 C4 FE BL              ExceptionCheck_sub_C6BA18C8
.text:C8A8FB40 00 28       CMP             R0, #0
```

获取的数据 (部分)

```
CB092000  50 4F 53 54 20 2F 76 35  2F 73 69 67 6E 20 7B 22  POST /v5/sign {"
CB092010  64 61 74 61 22 3A 22 62  4A 62 50 67 72 4B 49 42  data":"bJbPgrKIB
CB092020  6A 6D 4E 4E 2B 6B 63 30  39 66 66 6F 54 70 67 48  jmNN+kc09ffoTpgH
CB092030  57 39 57 46 7A 43 6B 67  75 4D 4E 71 50 52 57 68  W9WFzCkguMNqPRWh
CB092040  6A 70 41 67 2B 4E 62 45  70 76 47 62 44 42 54 47  jpAg+NbEpvGbDBTG
CB092050  6E 79 31 38 6C 6C 39 38  75 43 37 44 68 67 2B 33  ny18ll98uC7Dhg+3
CB092060  56 44 39 31 62 38 50 67  67 47 2F 47 56 61 52 59  VD91b8PggG/GVaRY
CB092070  71 4D 4C 37 33 36 30 6E  63 71 41 59 57 37 68 4A  qML7360ncqAYW7hJ
CB092080  4A 52 69 34 44 59 73 50  31 66 73 59 35 38 4F 79  JRi4DYsP1fsY58Oy
```

解密 PIC 数据获取 key (a7), 解密流程与初始化时一样。解密后的值

```
{"a1":0,"a10":400,"a2":"com.sankuai.meituan","a11":"c1ee9178c95d9ec75f0f076a374df94a032d54c8576298d4f75e653de3705449","a3":"0a16ecd60eb56a6a3349f66cdcf7f7bf5190e5a42d6280d8dc0ee3be228398ec","a4":1100030200,"k0":{"k1":"meituan1sankuai0","k2":"meituan0sankuai1","k3":"$MXMYBS@HelloPay","k4":"Maoyan010iauknaS","k5":"34281a9dw2i701d4","k6":"X%rj@KiuU+|xY}?f"},"a5":"11.3.200","a0":"pw/LhTdeoTTyaxPHcHMy+/ssGNS1ihNkrJ+uBI74FIfd90KlTil1m0i7FF/n0bhY","a6":"/HntC9XIfdUyII/UiVfx020EQPpHz2XZY3qzM2aiNmM0i0pB1yeSO689TY9SBB3s","a7":"QsHnU6kFjTYR8Z6tHEvkGMO2Hrt+NRnVQhmxg6EtVBzuzQcBpma3AdhTWNMpesFT","c0":{"c1":true,"c2":false},"a9":"SDEzWXi5LHL/cuMCZ1zYyv+0hIViqWWf+ShbUYILWf4=","a8":1603800117167}
```

解析 json 获取 a7

```
QsHnU6kFjTYR8Z6tHEvkGMO2Hrt+NRnVQhmxg6EtVBzuzQcBpma3AdhTWNMpesFT
```

appkey 与 pic 中的 a7 异或

```
.text:C8A90424 20 00       MOVS            R0, R4
.text:C8A90426 02 99       LDR             R1, [SP,#8]
.text:C8A90428 7E F0 12 FE BL              sub_C8B0F050
.text:C8A9042C 03 98       LDR             R0, [SP,#0xC]
.text:C8A9042E 00 5D       LDRB            R0, [R0,R4]             ; 取a7 QsHnU6kFjTYR8Z6tHEvkGMO2Hrt+NRnVQhmxg6EtVBzuzQcBpma3AdhTWNMpesFT
.text:C8A90430 72 5C       LDRB            R2, [R6,R1]             ; appkey 9b69f861-e054-4bc4-9daf-d36ae205ed3e
.text:C8A90432 42 40       EORS            R2, R0
.text:C8A90434 6A 54       STRB            R2, [R5,R1]
.text:C8A90436 01 34       ADDS            R4, #1
.text:C8A90438 04 98       LDR             R0, [SP,#0x10]
.text:C8A9043A A0 42       CMP             R0, R4                  ; 判断是否结束
.text:C8A9043C F2 D1       BNE             loc_C8A90424
.text:C8A9043E 03 B5       PUSH            {R0,R1,LR}
.text:C8A90440 01 48       LDR             R0, =0
.text:C8A90442 FF F7 DB FF BL              loc_C8A903FC
```

异或后的值

```
BC79D120  5E 54 73 4D 30 7A 4C 44  57 34 53 77 44 40 55 51  ^TsM0zLDW4SwD@UQ
BC79D130  22 50 45 6D 33 2F 2B 5D  01 40 70 35 2B 60 5E 63  "PEm3/+].@p5+`^c
BC79D140  34 0C 5E 1D
```

再次异或

```
.text:C8A91C00             loc_C8A91C00                            ; CODE XREF: hmac_sha256_sub_BB754BAC+6C↓j
.text:C8A91C00 43 A9       ADD             R1, SP, #0x160+var_54
.text:C8A91C02 09 5C       LDRB            R1, [R1,R0]             ; 取APPkey与a7加密后数据
.text:C8A91C04 5C 22       MOVS            R2, #0x5C ; '\'
.text:C8A91C06 4A 40       EORS            R2, R1
.text:C8A91C08 32 AB       ADD             R3, SP, #0x160+var_98
.text:C8A91C0A 1A 54       STRB            R2, [R3,R0]             ; 存值
.text:C8A91C0C 36 22       MOVS            R2, #0x36 ; '6'
.text:C8A91C0E 4A 40       EORS            R2, R1
.text:C8A91C10 21 A9       ADD             R1, SP, #0x160+var_DC
.text:C8A91C12 0A 54       STRB            R2, [R1,R0]             ; 存值
.text:C8A91C14 01 30       ADDS            R0, #1
.text:C8A91C16 40 28       CMP             R0, #0x40 ; '@'         ; 判断是否结束
.text:C8A91C18 F2 D1       BNE             loc_C8A91C00
```

异或后的值

```
CB0D3000  68 62 45 7B 06 4C 7A 72  61 02 65 41 72 76 63 67  hbE{.Lzra.eArvcg
CB0D3010  14 66 73 5B 05 19 1D 6B  37 76 46 03 1D 56 68 55  .fs[...k7vF..VhU
CB0D3020  02 3A 68 2B 36 36 36 36  36 36 36 36 36 36 36 36  .:h+666666666666
CB0D3030  36 36 36 36 36 36 36 36  36 36 36 36 36 36 36 36  6666666666666666
```

将异或后的值与请求体组合

```
CB0D3000  68 62 45 7B 06 4C 7A 72  61 02 65 41 72 76 63 67  hbE{.Lzra.eArvcg
CB0D3010  14 66 73 5B 05 19 1D 6B  37 76 46 03 1D 56 68 55  .fs[...k7vF..VhU
CB0D3020  02 3A 68 2B 36 36 36 36  36 36 36 36 36 36 36 36  .:h+666666666666
CB0D3030  36 36 36 36 36 36 36 36  36 36 36 36 36 36 36 36  6666666666666666
CB0D3040  50 4F 53 54 20 2F 76 35  2F 73 69 67 6E 20 7B 22  POST /v5/sign {"
CB0D3050  64 61 74 61 22 3A 22 62  4A 62 50 67 72 4B 49 42  data":"bJbPgrKIB
CB0D3060  6A 6D 4E 4E 2B 6B 63 30  39 66 66 6F 54 70 67 48  jmNN+kc09ffoTpgH
CB0D3070  57 39 57 46 7A 43 6B 67  75 4D 4E 71 50 52 57 68  W9WFzCkguMNqPRWh
CB0D3080  6A 70 41 67 2B 4E 62 45  70 76 47 62 44 42 54 47  jpAg+NbEpvGbDBTG
CB0D3090  6E 79 31 38 6C 6C 39 38  75 43 37 44 68 67 2B 33  ny18ll98uC7Dhg+3
CB0D30A0  56 44 39 31 62 38 50 67  67 47 2F 47 56 61 52 59  VD91b8PggG/GVaRY
CB0D30B0  71 4D 4C 37 33 36 30 6E  63 71 41 59 57 37 68 4A  qML7360ncqAYW7hJ
```

计算组合值的 MD5

```
.text:C8A91C60 2A 00       MOVS            R2, R5
.text:C8A91C62 00 F0 79 F9 BL              md5_sub_BB7CFF58        ; R0:原始数后,R1:大小,R2:返回
.text:C8A91C66 05 AE       ADD             R6, SP, #0x160+var_14C
.text:C8A91C68 55 21       MOVS            R1, #0x55 ; 'U'
.text:C8A91C6A 30 00       MOVS            R0, R6
.text:C8A91C6C F8 F7 54 E8 BLX             __aeabi_memclr4
.text:C8A91C70 32 A9       ADD             R1, SP, #0x160+var_98
.text:C8A91C72 30 00       MOVS            R0, R6
.text:C8A91C74 02 9A       LDR             R2, [SP,#0x160+var_158]
.text:C8A91C76 1E F0 48 FC BL              getvalu_sub_C87B550A
.text:C8A91C7A 30 00       MOVS            R0, R6
.text:C8A91C7C 40 30       ADDS            R0, #0x40 ; '@'
.text:C8A91C7E 14 22       MOVS            R2, #0x14
.text:C8A91C80 29 00       MOVS            R1, R5
.text:C8A91C82 1E F0 42 FC BL              getvalu_sub_C87B550A
.text:C8A91C86 54 21       MOVS            R1, #0x54 ; 'T'
.text:C8A91C88 30 00       MOVS            R0, R6
.text:C8A91C8A 01 9A       LDR             R2, [SP,#0x160+var_15C]
.text:C8A91C8C 00 F0 64 F9 BL              md5_sub_BB7CFF58        ; R0:原始数后,R1:大小,R2:返回
.text:C8A91C90 20 00       MOVS            R0, R4                  ; p
.text:C8A91C92 F8 F7 30 E8 BLX             free
```

计算后得到的值

```
F0 EF 16 F8 BD C6 7B CC  8C B4 8F AC 4C EC 7A DB A8 A8 D1 05
```

  
解密 PIC 数据获取 KEY(a9)，解析 json 获取

```
a3:0a16ecd60eb56a6a3349f66cdcf7f7bf5190e5a42d6280d8dc0ee3be228398ec
```

a3 为 AES KEY 解密 a9 解密后数据，该值作为加密 body md5 值的 key  

```
3ey2scPxek170m6K
```

AES 加密 Body 计算得到的 MD5 值

```
IV 0102030405060708
key:3ey2scPxek170m6K

.text:C8AA94F0 E0 F7 FA EB BLX             malloc
.text:C8AA94F4 00 26       MOVS            R6, #0
.text:C8AA94F6 00 28       CMP             R0, #0
.text:C8AA94F8 1C D0       BEQ             loc_C8AA9534
.text:C8AA94FA 4A 99       LDR             R1, [SP,#0x120+arg_0]
.text:C8AA94FC 03 91       STR             R1, [SP,#0x120+var_114]
.text:C8AA94FE 07 AE       ADD             R6, SP, #0x120+var_104
.text:C8AA9500 F4 21       MOVS            R1, #0xF4
.text:C8AA9502 02 90       STR             R0, [SP,#0x120+var_118]
.text:C8AA9504 30 00       MOVS            R0, R6
.text:C8AA9506 E0 F7 08 EC BLX             __aeabi_memclr4
.text:C8AA950A 80 21       MOVS            R1, #0x80
.text:C8AA950C 20 00       MOVS            R0, R4
.text:C8AA950E 32 00       MOVS            R2, R6
.text:C8AA9510 56 F0 30 FF BL              AES_set_Encrypt_key_sub_CB601374 ; R0:key,R1:长度,R2:返回值
.text:C8AA9514 06 9A       LDR             R2, [SP,#0x120+byte_count]
.text:C8AA9516 01 20       MOVS            R0, #1
.text:C8AA9518 69 46       MOV             R1, SP
.text:C8AA951A 04 9B       LDR             R3, [SP,#0x120+var_110]
.text:C8AA951C 0B 60       STR             R3, [R1,#0x120+var_120]
.text:C8AA951E 48 60       STR             R0, [R1,#0x120+var_11C]
.text:C8AA9520 05 98       LDR             R0, [SP,#0x120+p]
.text:C8AA9522 02 9C       LDR             R4, [SP,#0x120+var_118]
.text:C8AA9524 21 00       MOVS            R1, R4
.text:C8AA9526 33 00       MOVS            R3, R6
.text:C8AA9528 57 F0 7A FD BL              AES_cbc_Encrypt_sub_CB602020 ; R0：原始数据,R1:返回,R2:大小,R3:key
.text:C8AA952C 06 98       LDR             R0, [SP,#0x120+byte_count]
.text:C8AA952E 03 99       LDR             R1, [SP,#0x120+var_114]
.text:C8AA9530 08 60       STR             R0, [R1]
.text:C8AA9532 26 00       MOVS            R6, R4
.text:C8AA9534
.text:C8AA9534             loc_C8AA9534                            ; CODE XREF: Aes_sub_CB5AA4B8+40↑j
.text:C8AA9534 05 98       LDR             R0, [SP,#0x120+p]       ; p
.text:C8AA9536 E0 F7 DE EB BLX             free
```

加密后数据

```
C7B9C460  06 34 4B B7 17 3B 29 9D  FE B9 85 8D 24 6C 52 AE
C7B9C470  BB AF 22 3F 8E 43 FB C5  66 2B 54 E2 C6 6A 54 EA
```

转换成字符串

```
06344bb7173b299dfeb9858d246c52aebbaf223f8e43fbc5662b54e2c66a54ea
```

组合 json, 生成签名

```
{
	"a0": "2.0",
	"a1": "9b69f861-e054-4bc4-9daf-d36ae205ed3e",
	"a3": 2,
	"a4": 1631754963,
	"a5": "An2sai6nXEuFYeKbDUk/qEo/7am8Jtn3O2Has5efofCux7iubGRCS8TKpjUgotJ6MGdQrBsvwh/peZwuikT+5rCr4RzN8SXXCeDOtZQ6sbo/snVdESSJweNqd6i/WbIwDOgv5eaWRQoISjdgNfW3hk7tf0QhsZFbOELcPxz5pRzo6d6EsBLFu5Mq8DbVNgsYF+6aqZ7302/G+Rr7MlUT2M9y3EjgjH01L39q/eRJ",
	"a6": 1025,
	"a7": "Rs8NOy0BFS5JQxfdOoIxpMnKV3iqYWcblAjp0vpnWZyNzyF9rfsi3ekpm4ScaIZgeImizX/5AbS3e838Or4el4+PPPI2kD8XW+8vbvjDBSM=",
	"a8": "DAD796C46B5A6525F4B89DF661A97C7A218A219FC24B93F689DEBD92",
	"a9": "f4ec12efNkQBqdtVlV78x1/Mln2Us/xw171NuJjdEXrGWsFDdMV5Te45wqjL0nPO8OFFKjvKthvva+lS9xqhMhSt1WZRjDYpsWc/eh3z4F2JTv3MOh8NEDmk7Frthx5/bczdDIKvRP0QneTfNKSm116fUjdLhOEYlNbym/xI+5jZAEJGfGjltFLEOmtOwgxasgHQh2woMl/vAyr7ePuoVC6wEcbv2+w6n/+Pl1U1KO2YcTw4peiZDqC7iHpTVQH4fWri9R5+Ev1zx/xObVqoxqe3TEW/t2EIRqQ4QoRZRix0xC6C280faz8U5vOqafUnm+qev7tjs7SOV4SNxBv+LEJTxr5IJU302FJEk/CqhKoz5eWRYtT5Z52kEanlfu4AGHcJLC343kpI3GxYw7uPeewA/Ye0qDgZUyfj0MPpaYMPj0UmtvnbXEU4+FaRaCb/LsQtWdOtEiEKveUQU9bTW4NfHch2+6gcHP2/E+UQSlREX67PPa9XN8tgL4H6qzghiE1NL3gYw0rrjzEXiO6jsjvIdzwDjeab9woJyr8W3xSACz3sezUS+AJAKohnJvlQkFM9cdG3lPYS7gByAK++K2/vI714kxJHqCZQQMbVNsoWj5w64YL+sE1A4byzOPgK71oPb9w6Cb+KwVlDtqH8F5vlVkO23Iq8E28BI2vQuq5TLRzMunjo45Ks2Py9ZZuHkGb+QIVpUz6ViB+JlUONKinIJF1p6g==",
	"a10": "{}",
	"x0": 1,
	"a2": "06344bb7173b299dfeb9858d246c52aebbaf223f8e43fbc5662b54e2c66a54ea"
}
```

a2: 就是请求体签名值，其它字段上面已经解释过，然后将签名值返回到 java 层，整个签名流程就完成了。

作者简介：  
我是小三，目前从事软件安全相关工作，虽己工作多年，但内心依然有着执着的追求，信奉终身成长，不定义自己，热爱技术但不拘泥于技术，爱好分享，喜欢读书和乐于结交朋友，欢迎加我微信与我交朋友 (公众号输入框回复“wx” 即可)
