> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/R6JnMtLSAWq0WUSRWyDZ9w)

### 六、设备指纹分析

#### 6.1、dfpid基本流程

java层调用fetchDfpId()获取dfpid

```
public boolean fetchDfpId(boolean arg13) {  
        Object[] v8 = new Object[]{((byte)(((byte)arg13)))};  
        ChangeQuickRedirect v10 = DFPManager.changeQuickRedirect;  
        if(PatchProxy.isSupport(v8, this, v10, false, "19489f6702cf81011acff69ff203e6a7", 0x4000000000000000L)) {  
            return ((Boolean)PatchProxy.accessDispatch(v8, this, v10, false, "19489f6702cf81011acff69ff203e6a7")).booleanValue();  
        }  
  
        try {  
            if(this.isOutDate()) {  
            label_49:  
                if(!this.postDFPID(this.encDfpDataForId())) {  // 服务器获取  
                    this.ensureLocalID();  // 读取本地ID  
                }  
            }  
            else {  
                if(arg13) {  
                    goto label_49;  
                }  
  
                String v4 = this.idStore.getLocalDFPId();  
                if(!TextUtils.isEmpty(v4)) {  
                    goto label_38;  
                }  
  
                if(!this.postDFPID(this.encDfpDataForId())) {  
                    this.ensureLocalID();  
                    return 1;  
                label_38:  
                    this.idCallback.onSuccess(v4, ((long)DFPManager.ONE_HOUR) * ((long)this.idStore.getInterval()) + ((long)this.idStore.getLastUpdateTime()), "get dfp from local store");  
                    long v7 = System.currentTimeMillis();  
                    this.doPersistence(v4, this.idStore.getInterval().longValue(), v7);  
                    return 1;  
                }  
            }  
        }  
        catch(Throwable unused_ex) {  
        }  
  
        return 1;  
    }
```

调用encDfpDataForId()方法走到Native层获取请求体

```
 public String encDfpDataForId() {  
        Object[] v8 = new Object[0];  
        ChangeQuickRedirect v9 = DFPManager.changeQuickRedirect;  
        if(PatchProxy.isSupport(v8, this, v9, false, "c899af894ac6d9a1a9df952ed8770d17", 0x4000000000000000L)) {  
            return (String)PatchProxy.accessDispatch(v8, this, v9, false, "c899af894ac6d9a1a9df952ed8770d17");  
        }  
  
        FamaCollector.hasCollected = true;  
        return NBridge.main1(41, new Object[0]);  // 请求体  
    }
```

调用postDFPID(String arg12)发送网络请求获取dfpid

```
url: https://appsec-mobile.meituan.com/v5/sign  
  
  public boolean postDFPID(String arg12) {  
        Object[] v0 = new Object[]{null};  
        v0[0] = arg12;  
        ChangeQuickRedirect v9 = IDFPManager.changeQuickRedirect;  
        if(PatchProxy.isSupport(v0, this, v9, false, "f625ba5594fc8256be5f644877e817ae", 0x4000000000000000L)) {  
            return ((Boolean)PatchProxy.accessDispatch(v0, this, v9, false, "f625ba5594fc8256be5f644877e817ae")).booleanValue();  
        }  
  
        Interceptor v0_1 = this.getInterceptor();  
        String v1 = this.getMtgVersion();  
        DFPReporter v0_2 = new Builder().addInterceptor(v0_1).addResponseParser(new IResponseParser() {  
            public static ChangeQuickRedirect changeQuickRedirect;  
  
            @Override  // com.meituan.android.common.dfingerprint.network.IResponseParser  
            public boolean onError(Call arg11, IOException arg12) {  
                Object[] v0 = new Object[]{arg11, arg12};  
                ChangeQuickRedirect v12 = com.meituan.android.common.dfingerprint.interfaces.IDFPManager.1.changeQuickRedirect;  
                if(PatchProxy.isSupport(v0, this, v12, false, "0231e3e90aba1d77b6009d4f4e808ce1", 0x4000000000000000L)) {  
                    return ((Boolean)PatchProxy.accessDispatch(v0, this, v12, false, "0231e3e90aba1d77b6009d4f4e808ce1")).booleanValue();  
                }  
  
                Logger.logD("/v5/sign onError");  
                IDFPManager.this.ensureLocalID();  
                return 1;  
            }  
  
            @Override  // com.meituan.android.common.dfingerprint.network.IResponseParser  
            public boolean onResponse(Response arg23, long arg24, int arg26) {  
                String v2_1;  
                long interval;  
                String dfpid;  
                ResponseBody body;  
                Object[] v11 = new Object[]{arg23, new Long(arg24), ((int)arg26)};  
                ChangeQuickRedirect v14 = com.meituan.android.common.dfingerprint.interfaces.IDFPManager.1.changeQuickRedirect;  
                if(PatchProxy.isSupport(v11, this, v14, false, "cfa6814ee923204fe513080ced241785", 0x4000000000000000L)) {  
                    return ((Boolean)PatchProxy.accessDispatch(v11, this, v14, false, "cfa6814ee923204fe513080ced241785")).booleanValue();  
                }  
  
                Logger.logD("/v5/sign onResponse $response");  
                if(arg23 == null) {  
                    return 0;  
                }  
  
                if(arg23.code() != 200) {  
                    MTGlibInterface.raptorAPI("v5_/v5/sign", arg23.code(), arg26, 0, System.currentTimeMillis() - arg24);  
                    return 0;  
                }  
  
                try {  
                    body = arg23.body();  
                    if(body == null) {  
                        IDFPManager.this.idCallback.onFailed(-3, "request body is invalid");  
                        return 0;  
                    }  
  
                    String v3 = new String(body.bytes());  
                    MTGlibInterface.raptorAPI("v5_/v5/sign", arg23.code(), arg26, v3.length(), System.currentTimeMillis() - arg24);  
                    Logger.logD("RaptorMonitorService >> ${RaptorUtil.API_reportdfpidsync}");  
                    Logger.logD("RaptorMonitorService result>> ${result}");  
                    if(v3.isEmpty()) {  
                        IDFPManager.this.idCallback.onFailed(-3, "request body is invalid");  
                        MTGlibInterface.raptorAPI("v5_/v5/sign", 9401, arg26, 0, System.currentTimeMillis() - arg24);  
                        return 0;  
                    }  
  
                    Logger.logD("/v5/sign response.body() >> $result");  
                    DFPResponse body_json = (DFPResponse)new Gson().fromJson(v3, DFPResponse.class);  
                    if(body_json == null) {  
                        goto label_145;  
                    }  
  
                    if(body_json.code == 0xFFFFFF80) {  
                        goto label_145;  
                    }  
  
                    int v3_1 = body_json.code;  
                    if(v3_1 != 0) {  
                        IDFPManager.this.idCallback.onFailed(v3_1, body_json.message);  
                        MTGlibInterface.raptorAPI("v5_/v5/sign", 9402, arg26, 0, System.currentTimeMillis() - arg24);  
                        return 0;  
                    }  
  
                    dfpid = body_json.data.dataDfp;  
                    interval = body_json.data.dataInterval;  
                    v2_1 = body_json.message;  
                }  
                catch(Exception unused_ex) {  
                    IDFPManager.this.ensureLocalID();  
                    MTGlibInterface.raptorAPI("v5_/v5/sign", 9405, arg26, 0, System.currentTimeMillis() - arg24);  
                    return 0;  
                }  
  
                try {  
                    Logger.logD("/v5/sign response, dataDecrypt > $dataDecrypt");  
                }  
                catch(Exception unused_ex) {  
                }  
  
                try {  // 如果等空读取本地的  
                    if(dfpid == "") {  // 如果等空读取本地的  
                        IDFPManager.this.ensureLocalID();  
                    }  
                    else {  
                        Logger.logD("/v5/sign 返回新的 local_dfp_id，需要保存它");  
                        long currtime = System.currentTimeMillis();  
                        if(DFPConfigs.isDfpidFirst) {  
                            MTGlibInterface.raptorFakeAPI("v5_dfpid_duration", 200, System.currentTimeMillis() - MTGuard.sFirstLaunchTime);  
                            DFPConfigs.isDfpidFirst = false;  
                        }  
  
                        IDFPManager.this.doPersistence(dfpid, interval, currtime);  // 保存dfpid  
                        IDFPManager.this.idCallback.onSuccess(dfpid, interval * ((long)IDFPManager.ONE_HOUR) + currtime, v2_1);  
                    }  
  
                    body.close();  
                    return 1;  
                label_145:  
                    IDFPManager.this.idCallback.onFailed(-4, "body parse failed");  
                }  
                catch(Exception unused_ex) {  
                    IDFPManager.this.ensureLocalID();  
                    MTGlibInterface.raptorAPI("v5_/v5/sign", 9405, arg26, 0, System.currentTimeMillis() - arg24);  
                }  
  
                return 0;  
            }  
        }).build();  
        try {  
            ReqeustBody v2 = new ReqeustBody(arg12, v1, null);  
            Gson v12 = new GsonBuilder().disableHtmlEscaping().create();  
            if(v12 != null) {  
                String v12_1 = v12.toJson(v2);  
                Logger.logD("postDFPID body > $body");  
                return v0_2.reportDFPIDSync(v12_1, ContentType.application_json);  // 网络请求  
            }  
        }  
        catch(Exception unused_ex) {  
        }  
  
        return false;  
    }
```

#### 6.2、dfpid请求体分析

请求体调用Native层方法NBridge.main(41, new Object[0])在Native层计算得到  
循环获取设备信息

```
text:C8A8C684 09 98       LDR             R0, [SP,#0x24]  
.text:C8A8C686 61 F0 CD FF BL              getmContext_sub_CC9EF624 ; 获取上下文件  
.text:C8A8C68A 07 90       STR             R0, [SP,#0x1C]  
.text:C8A8C68C 00 F0 14 FE BL              init_DecString_sub_C84582B8  
.text:C8A8C690 06 90       STR             R0, [SP,#0x18]  
.text:C8A8C692 2B A8       ADD             R0, SP, #0xAC  
.text:C8A8C694 05 90       STR             R0, [SP,#0x14]  
.text:C8A8C696 09 99       LDR             R1, [SP,#0x24]  
.text:C8A8C698 07 9A       LDR             R2, [SP,#0x1C]  
.text:C8A8C69A 2B 00       MOVS            R3, R5  
.text:C8A8C69C 26 F0 7A FC BL              GetDeviceInfo_dispatch_loop_sub_C89F8F94 ; 获取设备信息1  
.text:C8A8C6A0 06 98       LDR             R0, [SP,#0x18]  
.text:C8A8C6A2 29 00       MOVS            R1, R5  
.text:C8A8C6A4 05 9A       LDR             R2, [SP,#0x14]  
.text:C8A8C6A6 23 F0 8D FF BL              jmp_checkField_sub_C89F65C4 ; 判断获取的内容是否为特殊字符mtg_unsupport,mtg_block,setup_empty  
.text:C8A8C6AA 2B 98       LDR             R0, [SP,#0xAC]  
.text:C8A8C6AC 04 99       LDR             R1, [SP,#0x10]  
.text:C8A8C6AE 40 18       ADDS            R0, R0, R1  
.text:C8A8C6B0 27 A9       ADD             R1, SP, #0x9C  
.text:C8A8C6B2 77 F0 49 F8 BL              free_sub_B2943748  
.text:C8A8C6B6 07 99       LDR             R1, [SP,#0x1C]  
.text:C8A8C6B8 00 29       CMP             R1, #0  
.text:C8A8C6BA 00 D1       BNE             loc_C8A8C6BE  
.text:C8A8C6BC 4C E7       B               loc_C8A8C558  

```

在下面地方下好断点跳到获取设备信息的地方，可以绕过代码流程混淆，快速定位到想要获取设备信息的方法：

```
.text:C8AB5134 80 00       LSLS            R0, R0, #2              ; 跳到真实执行方法  
.text:C8AB5136 02 A1       ADR             R1, loc_C8AB5140  
.text:C8AB5138 08 58       LDR             R0, [R1,R0]             ; 取方法表偏移  
.text:C8AB513A 08 18       ADDS            R0, R1, R0  
.text:C8AB513C 87 46       MOV             PC, R0  

```

获取完设备信息后组合成json，主要获取硬件类ID、imei、mac,系统属性、CPU、内存、陀螺仪传感器等  

```
{  
	"m20": "3",  
	"m22": "0",  
	"m24": "-1",  
	"m25": "0",  
	"m26": "0",  
	"m27": "DAD703FB1B789AFCFA0CAD4778CF9C8D0146219F941D41B66617A73F",  
	"m28": "0",  
	"m29": "28.5",  
	"m30": "1",  
	"m31": "0",  
	"m32": "0",  
	"m33": "1",  
	"m34": "",  
	"m35": "0",  
	"m36": "0",  
	"m37": "0",  
	"m38": "1",  
	"m39": "Google",  
	"m40": "",  
	"m41": "",  
	"m42": "1",  
	"m43": "23132356608",  
	"m44": "1230796800000",  
	"m45": "1",  
	"m46": "",  
	"m47": "android",  
	"m48": "0",  
	"m49": "nonetwork",  
	"m50": "user",  
	"m51": "1",  
	"m52": "google",  
	"m53": "unknown",  
	"m54": "adb",  
	"m55": "{\"data\":[36.107063],\"name\":\"TMD4903LightSensor\",\"vendor\":\"AMS\"}",  
	"m56": "google/sailfish/sailfish:9/PQ2A.190305.002/5240760:user/release-keys",  
	"m57": "[{\"ssid\":\"2F16C2\",\"bssid\":\"cc:ee:07:2f:16:c2\"},{\"ssid\":\"2F16C2\",\"bssid\":\"cc:ee:07:2f:16:c2\"},{\"ssid\":\"JinRong\",\"bssid\":\"cc:e9:e4:84:cc:76\"}]",  
	"m58": "unknown",  
	"m59": "0",  
	"m60": "2283765760",  
	"m61": "[1,BMI160accelerometer,1,Bosch,156,2500,0,0,4,BMI160gyroscope,1,Bosch,17,2500,0,0,2,AK09915magnetometer,1,AKM,1300,20000,0,0,6,BMP285pressure,1,Bosch,1100,100000,0,0,65536,BMP285temperature,1,Bosch,85,40000,0,0,8,TMD4903ProximitySensor,1,AMS,5,200000,0,1,5,TMD4903LightSensor,1,AMS,43000,200000,0,10,3,Orientation,1,Google,360,5000,0,1,18,BMI160Stepdetector,1,Bosch,1,0,0,1,17,Significantmotion,1,Google,1,-1,0,1,9,Gravity,1,Google,1000,5000,0,1,10,LinearAcceleration,1,Google,1000,5000,0,1,11,RotationVector,1,Google,1000,5000,0,1,20,GeomagneticRotationVector,1,Google,1000,5000,0,1,15,GameRotationVector,1,Google,1000,5000,0,1,25,PickupGesture,1,Google,1,-1,0,1,22,TiltDetector,1,Google,1,0,0,1,19,BMI160Stepcounter,1,Bosch,1,0,0,1,14,AK09915magnetometer(uncalibrated),1,AKM,1300,20000,0,0,16,BMI160gyroscope(uncalibrated),1,Bosch,17,2500,0,0,65537,SensorsSync,1,Google,1,0,0,1,65538,DoubleTwist,1,Google,1,0,0,1,65539,DoubleTap,1,Google,1,0,0,1,27,DeviceOrientation,1,Google,3,0,0,1,65540,DoubleTouch,1,Google,1,-1,0,1,35,BMI160accelerometer(uncalibrated),1,Bosch,156,2500,0,0,32,DynamicSensorManager,1,Google,1,1000,0,1]",  
	"m62": "1",  
	"m63": "0",  
	"m64": "1",  
	"m65": "1",  
	"m66": "Bosch",  
	"m67": "unknown",  
	"m68": "",  
	"m69": "352531086839980",  
	"m70": "sailfish",  
	"m71": "Gravity",  
	"m72": "1",  
	"m73": "com.qihoo.appstore-bin.mt.plus-cn.tongdun.sdkdemo-com.arcsoft.arcfacedemo-com.baidu.idl.face.demo-com.bjgas.shop-com.duapps.gif.emoji.gifmaker-com.mv.livebodyexample-com.sankuai.meituan-com.songheng.wubiime",  
	"m74": "FA7740302912",  
	"m75": "unknown",  
	"m76": "QualcommRIL1.0",  
	"m77": "BMI160accelerometer",  
	"m78": "com.android.chrome-com.android.settings-com.android.vending-com.google.android.GoogleCamera-com.google.android.apps.docs-com.google.android.apps.maps-com.google.android.apps.messaging-com.google.android.apps.photos-com.google.android.calendar-com.google.android.contacts",  
	"m79": "midi,adb",  
	"m80": "",  
	"m81": "8996-130181-1811270246",  
	"m82": "32",  
	"m83": "PQ2A.190305.002",  
	"m84": "midi,adb",  
	"m85": "",  
	"m86": "PQ2A.190305.002",  
	"m87": "lac:40980,cid:3909155,rssi:22|lac:-1,cid:-1,rssi:20",  
	"m88": "cn",  
	"m89": "release-keys",  
	"m90": "1",  
	"m91": "unknown",  
	"m92": "",  
	"m93": "{\"hashInfo\":[],\"number\":0}",  
	"m94": "sailfish",  
	"m95": "1",  
	"m96": "8996-012001-1812132253",  
	"m97": "",  
	"m98": "Google",  
	"m99": "",  
	"m100": "0",  
	"m101": "CN",  
	"m102": "1",  
	"m103": "26109874176",  
	"m104": "1593600",  
	"m105": "420",  
	"m106": "0",  
	"m107": "ac:37:43:df:02:7e",  
	"m108": "android-build",  
	"m111": "37cb81453cec878d",  
	"m112": "unknown",  
	"m116": "wlan0",  
	"m117": "1631706064788",  
	"m122": "AA==\n",  
	"m123": "",  
	"m125": "",  
	"m126": "Dalvik/2.1.0(Linux;U;Android9;PixelBuild/PQ2A.190305.002)",  
	"m127": "0.0000000000|0.0000000000",  
	"m128": "unknown",  
	"m129": "",  
	"m130": "",  
	"m131": "23132299264",  
	"m132": "26109874176",  
	"m133": "",  
	"m134": "bus",  
	"m135": "0.000000",  
	"m136": "",  
	"m137": "3.18.122-g665c9a1",  
	"m138": "[2,95]",  
	"m139": "4",  
	"m140": "arm64-v8a,armeabi-v7a,armeabi",  
	"m141": "0",  
	"m142": "0.16470589",  
	"m143": "0",  
	"m144": "11.12.204",  
	"m145": "mtguard",  
	"m146": "1",  
	"m147": "sailfish",  
	"m148": "1631706140783",  
	"m149": "1631706064",  
	"m150": "1631754658276",  
	"m151": "unknown",  
	"m152": "5.1.7",  
	"m153": "",  
	"m154": "com.sankuai.meituan",  
	"m155": "1629207179980",  
	"m156": "",  
	"m157": "3948302336",  
	"m158": "sailfish",  
	"m159": "abfarm722",  
	"m160": "9",  
	"m161": "0",  
	"m162": "unknown",  
	"m163": "0",  
	"m164": "zh",  
	"m165": "[GMT+08:00,Asia/Shanghai]",  
	"m166": "Pixel",  
	"m167": "1080*1794",  
	"m253": "",  
	"m254": "",  
	"m255": "0",  
	"m256": "",  
	"m293": "DAD7A2870A2DAAA2F67190B216A34A886A9E219F941D662641A653D9",  
	"m241": "",  
	"m242": "",  
	"m245": "{\"libc.so:\":\"E5592D3E966419A89DEF46F6BE029015\",\"libandroid.so:\":\"\",\"libandroid_runtime.so:\":\"2F4ADAB748DE1F97EACAC55B9584E504\",\"libandroid_servers.so:\":\"75AF8941A0B6C9A841C05442FE32CCFC\",\"framework.jar:\":\"\",\"services.jar:\":\"\",\"core.jar:\":\"\"}",  
	"m246": "ae:37:43:df:02:7e",  
	"m247": "35253108683998",  
	"m289": "352531086839980",  
	"m290": "352531086839980",  
	"m294": "{\"1\":\"\",\"2\":\"1|2|3\",\"3\":\"\",\"4\":\"\",\"5\":\"\",\"6\":\"\",\"7\":\"2\",\"8\":\"\",\"9\":\"\",\"10\":\"\",\"11\":\"\",\"12\":\"32\",\"13\":\"\",\"14\":\"\",\"15\":\"\",\"33\":{\"0\":2,\"1\":\"\",\"2\":\"\",\"3\":\"\",\"4\":\"2|22\",\"5\":\"\",\"6\":\"\",\"7\":\"\",\"8\":\"\",\"9\":\"\",\"10\":\"\",\"11\":\"\",\"12\":\"\",\"13\":\"\",\"14\":\"2\",\"15\":\"\"}}",  
	"m304": "-1",  
	"m305": "1080*1920",  
	"m217": "1631706064788185",  
	"m243": "-1",  
	"m244": "{\"DCIM\":\"00045F6737E705C00001C3B7F33CF3000005C810802CB9C0000083D1C295CA000005C810802CB9C0000083D1C295CA00\",\"Android\":\"00045F6737F648000002A40F89B18AC00005C5CB1C563D000000EE4ABA3499C00005C5CB1C563D000000EE4ABA3499C0\",\"misc\":\"0005BA6437AAF540000037EFB50958800005BDC662F5BF40000263501EE0BE000005BDC662F5BF40000263501EE0BE00\",\"settings\":\"00045F67360E000000005AF31103944000045F67360E000000005AF31103944000045F67360E000000005AF311039440\",\"saver\":\"0005BA6437AAF54000005F590941430000045F67377C360000014D7BCD19158000045F67377C360000014D7BCD191580\",\"mtp\":\"0005BA6437AAF5400000CC7CB757DE0000045F67360E00000001EB208F56774000045F67360E00000001EB208F567740\",\"calendar\":\"0005BA6437AAF54000008CD291CAAE4000045F6738AD630000019F5706658BC000045F6738AD630000019F5706658BC0\",\"media\":\"0005BA6437AAF5400000B7440023B8000000225E8BD0FF0000037B4E60CF5D000000225E8BD0FF0000037E567AF07C40\",\"misc_ce\":\"0005BA6437AAF540000062612371A48000045F6737D7C38000010AC9B10045C000057E646876D18000027F601A1B3540\",\"rollback\":\"\",\"system\":\"0005BA6437AAF54000005C50EF10E1800005CC17296EE1C0000373A4714044800005CC17296EE1C0000373A471404480\",\"data\":\"0005BA6437AAF5400000687157C325400005CC0732B344C000001BE686FAA0400005CC0732B344C000001BE686FAA040\",\"install_sessions\":\"0005BA6437AAF54000005F59094143000000225E8BD0FF00000329739E3E68C00000225E8BD0FF00000329739E3E68C0\",\"webview\":\"0005BA6437AAF54000008CD291CAAE4000045F6737C88140000104B976B8E40000045F6737C88140000107C190DA0340\"}",  
	"m248": "",  
	"m307": "{\"m34\":5,\"m40\":5,\"m41\":5,\"m46\":5,\"m58\":2,\"m67\":6,\"m68\":5,\"m75\":6,\"m80\":5,\"m85\":5,\"m91\":2,\"m92\":5,\"m97\":5,\"m99\":5,\"m112\":3,\"m123\":5,\"m125\":5,\"m128\":6,\"m129\":5,\"m130\":5,\"m133\":5,\"m136\":5,\"m151\":2,\"m153\":5,\"m156\":5,\"m162\":6,\"m253\":5,\"m254\":7,\"m256\":7,\"m241\":7,\"m242\":5,\"m248\":5}"  
}
```

#### 6.3、加密设备数据

解密PIC数据获取AES KEY, 解密后数据

```
{"a1":0,"a10":400,"a2":"com.sankuai.meituan","a11":"c1ee9178c95d9ec75f0f076a374df94a032d54c8576298d4f75e653de3705449","a3":"0a16ecd60eb56a6a3349f66cdcf7f7bf5190e5a42d6280d8dc0ee3be228398ec","a4":1100030200,"k0":{"k1":"meituan1sankuai0","k2":"meituan0sankuai1","k3":"$MXMYBS@HelloPay","k4":"Maoyan010iauknaS","k5":"34281a9dw2i701d4","k6":"X%rj@KiuU+|xY}?f"},"a5":"11.3.200","a0":"pw/LhTdeoTTyaxPHcHMy+/ssGNS1ihNkrJ+uBI74FIfd90KlTil1m0i7FF/n0bhY","a6":"/HntC9XIfdUyII/UiVfx020EQPpHz2XZY3qzM2aiNmM0i0pB1yeSO689TY9SBB3s","a7":"QsHnU6kFjTYR8Z6tHEvkGMO2Hrt+NRnVQhmxg6EtVBzuzQcBpma3AdhTWNMpesFT","c0":{"c1":true,"c2":false},"a9":"SDEzWXi5LHL/cuMCZ1zYyv+0hIViqWWf+ShbUYILWf4=","a8":1603800117167}
```

解析json得到 aes key:meituan1sankuai0  
AES加密设备json数据

```
IV 0102030405060708  
.text:C8AA94EE 06 98       LDR             R0, [SP,#0x120+byte_count] ; byte_count  
.text:C8AA94F0 E0 F7 FA EB BLX             malloc                  ; 存放加密后数据  
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

加密后数据(部分)

```
C6A47000  6C 96 CF 82 B2 88 06 39  8D 37 E9 1C D3 D7 DF A1  l.ς ...9.7......  
C6A47010  A1 D7 C5 C7 A2 1F 21 47  F4 8D F8 E3 59 55 68 87  ...Ǣ .!G....YUh.  
C6A47020  13 D2 C7 A5 AD 47 51 04  64 B2 86 3A 86 AA 47 36  .....GQ.d..:..G6  
C6A47030  8A 26 F7 03 A0 42 3F 4E  B7 43 53 F5 E8 DE 9A 48  .&....?N.CS....H  
C6A47040  FF 7D CC D3 30 91 1C 59  34 6C 3C D5 A4 45 AD 8A  .}..0..Y4l<դ E..  
C6A47050  69 06 49 EA 1B B5 95 7F  40 3F AF BE DB 18 CB CD  i.I.....@?......  
C6A47060  85 A9 05 0E 02 43 1C 10  2C 50 53 9F A0 A1 6D 14  .....C..,PS...m.  
C6A47070  DF F9 23 99 64 3C E4 D8  7C 56 58 C0 35 E6 81 34  ..#.d<...VX.....  
C6A47080  9B 9C 6D A3 92 5B 36 50  6E 85 79 6E 2A 6C EB BF  ..m..[6Pn.yn*l..  
C6A47090  59 F5 77 F3 C3 F8 3D 47  04 D4 55 FB C1 D6 8D 3A  Y.....=G.......:  
C6A470A0  0F 4D EF 9E A2 C0 DF 93  58 B5 E1 79 22 EE 6A 65  .M  ...X.......  
C6A470B0  A4 A8 47 7F 68 BA C1 C7  19 96 8A 9E 29 90 48 07  ..G.h.......).H.  

```

base64加密(部分)

```
bJbPgrKIBjmNN+kc09ffoaHXxceiHyFH9I3441lVaIcT0selrUdRBGSyhjqGqkc2iib3A6BCP063Q1P16N6aSP99zNMwkRxZNGw81aRFrYppBknqG7WVf0A/r77bGMvNhakFDgJDHBAsUFOfoKFtFN/5I5lkPOTYfFZYwDXmgTSbnG2jkls2UG6FeW4qbOu/WfV388P4PUcE1FX7wdaNOg9N756iwN+TWLXheSLuamWkqEd/aLrBxxmWip4pkEgHlLK14AbB68tlUvIauN50i/KZVCun9lA13FmhY9t4JiZYMuQUiluXGtFLoVLUb3ybe9Uy156k8NFn8sZkCzx5I1nUFKEx2Mgb6Q+SDXY1r4TCh6jYCYd/Dfr+n1ccN+Z7M8oceexb3KD+yapCYKvhj2sm5/WptX+ncnTTalrLG8U53YZAJtwQ2NTZ/yuVJWNpnEjRE3EI  

```

返回java层组合请求体，计算请求头签名发送给服务器获取dfpid 请求体格式

```
{"data":"Native加密的设备数据","dfpVersion":"5.1.7","mtgVersion":"5.1.7","os":"Android","time":"2021-09-01 11:35:35"}  

```

计算的请求头签名

```
{"a0":"2.0","a1":"9b69f861-e054-4bc4-9daf-d36ae205ed3e","a3":2,"a4":1631778284,"a5":"A18ywLdCS48BnwSLKup0wrjB+dsq4S6V+9BZPtgOWyf76KKpdYv8xejvBBC8HE6C4uRAl79/X7icd48pdktGG8B0+yAbMGtUl5Ri2kQc4/RgxjDiN64MjMvus7Aok+9LqzRZW1W9Qj2yJoMyK7h57/+amTSLDqyUzbat6zAgCAdeZ/vbKk+34+NVNwoz7fMLqvRlGbTVLZddTFIQvzbs1rNC3eV7mH9ORw==","a6":0,"a7":"lpArUzLcl+lgZ9YXlGR9nPVqJMM2UvrBYyFRM2UsDIidftxLwpEOzzcZ7NPmHxahYSpXrJr/eHv4y2FmjsLwnykKopieCU4Prh4FskqVsRg=","a8":"DAD796C46B5A6525F4B89DF661A97C7A218A219FC24B93F689DEBD92","a9":"97a62b63j1IexpGhrGBYdWqnstTXCpGUZncRa5MAcc5LtAO65uXQ/gPQuhqUcACAU5Cstbgs/wxZr8oFrMNqpSDo+Lt3QT7CenBY9BUnVn6bSHYNAaPhQ3WxYc1fpRi3+Lfpqz6/6CtwzNM113yKUoIgwMQZ2TTkRJ5S7JI0PztFc8u2I5ngz2thHx3rIXDjNXhmxoo7xH9rULUa65Y1zdPRx2xVNGDUJDFX78jvL39wIkvvnqBMK54GyQ8F8Vd/+lyPg1Wi3GoVUyFKrfw5RRDTElNyby2CvgQVKFRwHYbJu8OS3TjT6sW4uqmNxZQl4Zg4DBg/C6vlXAZISTCiCzC5yQ/xfqrQ5hNmKmsfHiVMduB7XYh9INAqceiTRlPV/ccY/JeGPRTbo52FoIHEponCLaToMU08uJVYC+hvT21OB/Wae2dr1l0vF4zDP+109YHgf7HbHg40TJW+UfIJdkAR0XHDINEqZIPmgE7jgon2B9KMXSyrWTg3OvOVA2USc7lBX9/8s1AXyI6MYgb7JwddM/cajZJle9ipMzJTmSZbJPw8o3bHGlt2pTtXzJb7mLgOxS1cwoc2kUTQYkRz5irPM1bhY4dT33d0tOr6C8IjROMgQMuIsVxKYCt88qo8ESk/nZEpVcWvBlsM+HWSwjHKtOJ8iPl3EqwSQiWRjVN6bOQyQIsPNVTfkZHSBqKnJ4dXcjXMTqcxGgxEzfD5sLfzXDz71A==","a10":"{}","x0":1,"a2":"0f4513356e64a6d70d962a764b1d68b8"}  

```

发送给服务器后成功返回dfpid

```
{"code":0,"data":{"serverTimestamp":0,"clientIp":"218.xx.xx.xx","interval":24,"dfp":"8ecde1544fbe09ba3ec851e72fdd28c0803df1f83c5c0862c8ad571c"},"message":"ok"}  

```

以上就是完整的dfpid设备指纹生成过程，xid生成也是类似的，也是在Native层获取设备信息加密返回到java层组合请求体，计算请求头签名，发送给服务器计算ID返回ID的过程，我就不再重复分析了。

### 七、设备指纹攻击

#### 7.1、设备指纹原理

设备指纹是用来标识手机设备的唯一ID，能够通过这个ID关联到手机相关的全部数据，因此设备指纹是风控中最核心的数据之一，所以它需要具备以下几个条件时ID不变：设备重置、设备更新、设备刷机。要同时具备这样的条件，必须从多个维度采集不同的信息来生成ID，这些信息可以大致分为：软件ID、软件静态特征、硬件静态特征和硬件动态特征。采集完信息后如何基于这些信息计算出一个稳定的设备指纹ID，还是有比较大挑战，再加上目前国家对公民隐私保护严格，用户敏感信息不能采集，很多唯一性比较好的数据就失效，要做一稳定性高的ID就难上加难了。  

#### 7.2、设备指纹变与不变

既然设备指纹ID是根据手机设备的数据生成的，哪么我们的攻击思路就是修改设备信息，同一台手机不断变化设备信息让生成设备指纹的服务返回新的ID就算攻击成功了，因为它无法识别手机的唯一性。  
实战修改设备字段生成新的dfpid  
测试过程就是在内存中将采集的某设备字段改掉就可以生成新的设备ID(该字段是软件相关的ID，比较敏感就不说了，总共没有多少字段，想玩的朋友可以自己调试测试)  
修改前后对比图7-1与7-2所示，返回新的ID:

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

                                图7-1  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

                                图7-2  

### 八、黑产工具特征检测

#### 8.1、解密黑产工具特征

获取密钥，解密PIC数据获取key(a9)

```
.text:C8571594 20 00       MOVS            R0, R4  
.text:C8571596 32 00       MOVS            R2, R6  
.text:C8571598 57 F0 7A F8 BL              AES_set_decrypt_key_sub_CB601690 ; R0:key,R1:大小,R2:初始AES_KEY返回结构  
.text:C857159C 00 20       MOVS            R0, #0  
.text:C857159E 69 46       MOV             R1, SP  
.text:C85715A0 06 9A       LDR             R2, [SP,#0x128+var_110]  
.text:C85715A2 0A 60       STR             R2, [R1,#0x128+var_128]  
.text:C85715A4 48 60       STR             R0, [R1,#0x128+var_124]  
.text:C85715A6 08 98       LDR             R0, [SP,#0x128+var_108]  
.text:C85715A8 04 9E       LDR             R6, [SP,#0x128+var_118]  
.text:C85715AA 31 00       MOVS            R1, R6  
.text:C85715AC 07 9C       LDR             R4, [SP,#0x128+var_10C]  
.text:C85715AE 22 00       MOVS            R2, R4  
.text:C85715B0 03 9B       LDR             R3, [SP,#0x128+var_11C]  
.text:C85715B2 57 F0 35 FD BL              AES_cbc_Encrypt_sub_CB602020  
.text:C85715B6 30 00       MOVS            R0, R6
```

解密后特征

```
{  
	"0": 2,  
	"1": "",  
	"2": ["/sys/devices/virtual/misc/qemu_pipe", "/sys/module/vboxsf", "/system/droid4x", "/system/bin/droid4x-vbox-sf", "/system/lib/egl/libEGL_tiDetectanVM.so", "/system/bin/ttVM-vbox-sf", "/system/lib/libnox.so", "/system/bin/nox-vbox-sf", "/system/bin/androVM-vbox-sf", "/ueventd.vbox86.rc", "/sys/class/misc/qemu_pipe", "/sys/qemu_trace", "/dev/com.bluestacks.superuser.daemon", "/system/framework/libqemu_wl.txt", "/system/lib/libc_malloc_debug_qemu.so-arm", "/data/downloads/qemu_list.txt", "/system/bin/yiwan-prop", "/system/bin/yiwan-sf", "/system/bin/qemu_props", "/system/bin/androVM-prop", "/system/bin/microvirt-prop", "/system/lib/libdroid4x.so", "/system/bin/windroyed", "/system/bin/microvirtd", "/system/bin/nox-prop", "/system/bin/ttVM-prop", "/dev/qemu_pipe", "/vendor/bin/qemu-props", "/system/bin/droid4x-prop", "/data/.bluestacks.prop", "/system/app/com.mumu.launcher", "/system/lib/vboxguest.ko", "/system/lib/vboxsf.ko", "/sys/class/misc/vboxguest", "/sys/class/misc/vboxuser"],  
	"3": ["de.robv.android.xposed.installer", "com.saurik.substrate", "com.soft.apk008v", "com.soft.apk008Tool", "com.mockgps.outside.ui", "com.tim.apps.mockgps", "com.huichongzi.locationmocke", "com.lxzs"],  
	"4": ["XposedBridge.jar", "frida", "substrate", "libriru_edxp.so", "libsandhook.edxp.so", "libwhale.edxp.so", "libxposed_art.so", "edxp.jar", "fasthook", "libfakeloc_initzygote.so", "qssq666", "com.lxzs", "sandhook", "libsubstrate.so", "libiohook.so", "libepic.so", "libepic64.so", "libexp824.so", "libexp82464.so", "libexposed.so", "cydia", "/data/local/tmp"],  
	"5": "",  
	"6": ["de.robv.android.xposed.XposedBridge", "de.robv.android.xposed.XposedHelpers"],  
	"7": ["/system/lib/lic", "/system/lib/ccc", "/system/lib/.aa"],  
	"8": ["de.robv.android.xposed.XposedBridge", "xposed", "com.lxzs"],  
	"9": ["dexguard_DGResourcesSuperClass.dex"],  
	"10": ["VirtualBox"],  
	"11": ["com.gsxz.location", "net.anylocation", "com.huichongzi.locationmocker", "top.a1024bytes.mockloc.ca.pro", "com.deniu.daniu"],  
	"12": ["fridaserver", "magisk", "xposed"],  
	"13": "",  
	"14": ["charles", "fiddler"],  
	"15": ["com.netease.nemu_vinput.nemu"]  
}
```

#### 8.2、解析json检测特征

```
//直接调用SVC指令,检测特征是否存在系统中。  
.text:C8571444             faccessat_sub_C8BE5444  
.text:C8571444   
.text:C8571444 F0 B5       PUSH            {R4-R7,LR}  
.text:C8571446 03 AF       ADD             R7, SP, #0xC  
.text:C8571448 0B 00       MOVS            R3, R1  
.text:C857144A 04 00       MOVS            R4, R0  
.text:C857144C 63 20       MOVS            R0, #0x63 ; 'c'  
.text:C857144E C5 43       MVNS            R5, R0  
.text:C8571450 A7 20       MOVS            R0, #0xA7  
.text:C8571452 46 00       LSLS            R6, R0, #1  
.text:C8571454 28 46       MOV             R0, R5  
.text:C8571456 21 46       MOV             R1, R4  
.text:C8571458 1A 46       MOV             R2, R3  
.text:C857145A 37 46       MOV             R7, R6  
.text:C857145C 00 DF       SVC             0  
.text:C857145E 03 46       MOV             R3, R0  
.text:C8571460 18 00       MOVS            R0, R3  
.text:C8571462 F0 BD       POP             {R4-R7,PC}
```

### 九、总结

#### 9.1、安全点

代码混淆，代码块间跳转动态计算获取，单步调试容易跑飞、流程混淆无法F5，字符串加密。采集设备信息都在Native层反射来获取，  

#### 9.1、不足点

指纹稳定性比较差，在华为的机型上基本不行，防hook只是简单做了前几个字节判断，检测黑产工具的特征写死的。

样本获取,关注公众号，公众号输入框回复“MT”即可

  

作者简介：  
我是小三，目前从事软件安全相关工作，虽己工作多年，但内心依然有着执着的追求，信奉终身成长，不定义自己，热爱技术但不拘泥于技术，爱好分享，喜欢读书和乐于结交朋友，欢迎加我微信与我交朋友(公众号输入框回复“wx”即可)