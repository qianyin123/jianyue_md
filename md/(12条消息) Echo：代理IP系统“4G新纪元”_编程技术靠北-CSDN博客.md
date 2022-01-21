> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u013711462/article/details/117268046)

人在网上“爬”，哪有不挨“刀”。

反爬的首选第一件事就是封IP，爬虫选手第一件事就是上代理。

So…

一直代理IP资源都是紧俏资源，甚至花钱都不一定买得到好的。

于是有些需求就有了，IP代理系统是不是也可以搞一个？

当然，这样的需求早就有解决方案了。

一键启动XX代理，一键使用XX云申请100台主机启动代理…

这类方案差不多应有尽有了。

然而此类的方案问题在于，代理IP绑定在服务器上的，

流量出口总是很容易被查到是XX云厂商等等的。

那么，如果我们用手机客户端（Android） + 4G作为流量出口呢？

So…

Echo 4G代理系统应运而生。

项目地址： [https://github.com/virjar/echo](https://github.com/virjar/echo)

是我的老熟人 https://github.com/virjar(渣总) 开源，

PS: 最近我边用边维护，修修Bug

* * *

Echo
----

Echo是一个分布式的代理共享和管理系统，以长链接的方式连接多个运行在任意位置的终端，并将终端的网络资源整理为一套代理ip集群系统。

echo提供整体的鉴权、流量监控、quota控制的功能。

*   Echo天然支持复杂网络环境，所以可以将代理终端部署在手机(甚至树莓派等终端设备)
*   Echo支持代理ip统一的集群管理，所以可以作为ADSL拨号的服务资源的的统一管理出口。使用ADSl使用统一的，稳定的ip出口提供代理服务(而不需要沉重的redis负担)
*   Echo支持sdk，目前提供完善的android APK和gradle依赖(这个作用你懂的 )
*   Echo分布式设计，天然集群版，无资源瓶颈上限。各节点自动双通道HA热备，无单点风险。
*   Echo全程NIO设计，对资源消耗少，支持并发高(所以代码难度大，可以买个好价钱)，理论上代理最大吞吐占满节点带宽。
*   Echo系统扩展能力强，原则是echo的底层设计使得echo支持任意网络协议转发（udp、tcp、vpn等），且任意协议支持不需要终端升级
*   终端命令控制，你可以通过http接口将特定指令下发到对应终端.实现如shell执行、ip重播等需求。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210526003738329.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM3MTE0NjI=,size_16,color_FFFFFF,t_70)

PS：请java高级工程师以下(初级和中级)同学不要尝试Echo服务端的研究 ，请java初级(包括不会java语言的同学)不要尝试部署Echo服务端。（渣总原话

嗯？被劝退了？有宝哥在啊。

虽然系统部署比较复杂，不过我们有docker-compose神器啊。

* * *

部署服务端
-----

### 部署方法一：

git clone https://github.com/virjar/echo/;  
cd echo;  
docker-compose up -d;

### 部署方法二：

新建一个文件夹 echo-deploy，新建 docker-compose.yaml，填入下面docker-compose配置

```
`version: '3'
services:
  echo-mysql-local:
    image: mysql:5.7
    container_name: echo-mysql-local
    ports:
      - 4444:3306
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/echo_db_create.sql:/docker-entrypoint-initdb.d/echo_db_create.sql
    environment:
      MYSQL_ROOT_PASSWORD: "echo"
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
  echo-meta-server:
    image: registry.cn-beijing.aliyuncs.com/virjar/echo-meta-server:latest
    container_name: echo-meta-server
    ports:
      - 4826:8080
    environment:
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: echo
      SPRING_DATASOURCE_URL: jdbc:mysql://echo-mysql-local:3306/echo?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&autoConnect=true
    depends_on: 
      - echo-mysql-local
  echo-fe-ui:
    image: registry.cn-beijing.aliyuncs.com/virjar/echo-fe-ui:20210430
    container_name: echo-fe-ui
    ports:
      - 8999:80
    volumes:
      - ./echo-fe-nginx.conf:/etc/nginx/conf.d/default.conf
    environment:
      API_ENTRY: http://echo-meta-server:8080
    depends_on: 
      - echo-meta-server
  echo-nat-server:
    image: registry.cn-beijing.aliyuncs.com/virjar/echo-nat-server:latest
    container_name: echo-nat-server
    ports:
      - 12000-12010:12000-12010
      - 5699:5699
      - 5698:5698
    environment:
      API_ENTRY: http://echo-meta-server:8080
      SERVER_ID: echo-nat-server-001
      MAPPING_SPACE: 12000-12010
    depends_on: 
      - echo-meta-server
  echo-http-proxy-server:
    image: registry.cn-beijing.aliyuncs.com/virjar/echo-http-proxy-server:latest
    container_name: echo-http-proxy-server
    ports:
      - 13000-13020:13000-13020
      - 5710:5710
    environment:
      API_ENTRY: http://echo-meta-server:8080/
      MAPPING_SERVER_URL: http://echo-meta-server:8080/echoNatApi/connectionList
      AUTH_CONFIG_URL: http://echo-meta-server:8080/echoNatApi/syncAuthConfig
      SERVER_ID: echo-http-proxy-001
      MAPPING_SPACE: 13000-13020
    depends_on: 
      - echo-meta-server
  echo-client:
    image: registry.cn-beijing.aliyuncs.com/virjar/echo-client:latest
    container_name: echo-client
    environment:
      API_ENTRY: http://echo-meta-server:8080/
      ECHO_ACCOUNT: admin
      ECHO_PASSWORD: admin
    depends_on: 
      - echo-meta-server
      - echo-http-proxy-server
      - echo-nat-server` 

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
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48
*   49
*   50
*   51
*   52
*   53
*   54
*   55
*   56
*   57
*   58
*   59
*   60
*   61
*   62
*   63
*   64
*   65
*   66
*   67
*   68
*   69
*   70
*   71
*   72
*   73


```

### docker-compose.yaml

*   echo-mysql-local 数据库
*   echo-meta-server 原信息服务 + 权限管理
*   echo-fe-ui admin Web管理
*   echo-nat-server nat映射服务，依赖echo-meta-server
*   echo-http-proxy-server http-proxy,依赖echo-meta-server
*   echo-client 代理出口，依赖echo-meta-server启动

1.  docker-compose up;  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210526004034710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM3MTE0NjI=,size_16,color_FFFFFF,t_70)

*   首次启动数据库初始化需要时间，echo-meta-server启动后可能连接不上数据库，重启一次就好
*   数据库初始化依赖于./mysql/echo_db_create.sql

3.  echo-deploy里面新建mysql文件夹，将 echo_db_create.sql 扔进去
    
4.  下载 echo-fe-nginx.conf 扔到 echo-deploy 文件夹
    
5.  docker-compose up -d;
    
6.  访问http://localhost:8999
    

### Admin配置

服务都正常启动之后，还需要做一下NATServer和http-proxy server配置。

#### 注册账号和设置admin账号

首先，http://localhost:8999 注册一下账号，本地测试一般直接使用 admin/admin就算了。

同时设置一下代理账号密码，都设置成 10086/10086 即可。

http://localhost:8999 (二维码自动识别)

注册完成之后，进入mysql容器（如果是自己的MySQL自行处理），

本地数据库密码默认是echo；

将刚刚注册的账号设置成管理员，然后重新登录。

```
`$:docker exec -it echo-mysql-local bash;
root@3c35bcc6c9e8:/# mysql -uroot echo -p
Enter password:
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 83
Server version: 5.7.34 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>  update user_info set admin=1 where id =1;` 

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


```

重新登录之后，就能看到Admin管理服务资源页面了。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210526004059388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM3MTE0NjI=,size_16,color_FFFFFF,t_70)

#### 添加nat-server和http-proxy-server

nat-server地址为：http://主机IP:5699

http-proxy-server地址为：http://主机IP:5710

如下图能显示服务的SID，则说明添加成功了。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210526004115966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM3MTE0NjI=,size_16,color_FFFFFF,t_70)

到这里系统已经搭建完成了，接着是最终一步，接入Android客户端。

### echo-client接入

*   支持本地主机和Android 客户端

本地主机接入推荐使用docker

```
`docker run -e API_ENTRY=http://192.168.31.135:4826/ \
-e CLIENT_ID=local_echo_client_2 \
-e ECHO_ACCOUNT=admin -e ECHO_PASSWORD=admin \
--restart=always --name=local__debug_echo_client_2 \
-d registry.cn-beijing.aliyuncs.com/virjar/echo-client` 

*   1
*   2
*   3
*   4
*   5


```

Android app 在Admin 页面可以下载到最新Apk，下载好后自行安装启动。

最后，在“代理”资源页面能看到代理IP信息，就说明成功了。

使用

```
`$ export https_proxy=http://10086:10086@192.168.31.135:13012;curl -vvv https://qq.com

* Uses proxy env variable https_proxy == 'http://10086:10086@192.168.31.135:13012'
*   Trying 192.168.31.135...
* TCP_NODELAY set
* Connected to 192.168.31.135 (192.168.31.135) port 13012 (#0)
* allocate connect buffer!
* Establish HTTP proxy tunnel to qq.com:443
* Proxy auth using Basic with user '10086'
> CONNECT qq.com:443 HTTP/1.1
> Host: qq.com:443
> Proxy-Authorization: Basic MTAwODY6MTAwODY=
> User-Agent: curl/7.64.1
> Proxy-Connection: Keep-Alive
>
< HTTP/1.1 200 Connection established
< Connection: keep-alive
< Via: 1.1 echo-proxy
<
* Proxy replied 200 to CONNECT request
* CONNECT phase completed!
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/cert.pem
  CApath: none
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* CONNECT phase completed!
* CONNECT phase completed!
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=CN; ST=Guangdong Province; L=Shenzhen; O=Shenzhen Tencent Computer Systems Company Limited; OU=R&D; CN=www.qq.com
*  start date: Jun 22 00:00:00 2020 GMT
*  expire date: Sep 22 12:00:00 2021 GMT
*  subjectAltName: host "qq.com" matched cert's "qq.com"
*  issuer: C=US; O=DigiCert Inc; OU=www.digicert.com; CN=Secure Site CA G2
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7f84b8808200)
> GET / HTTP/2
> Host: qq.com
> User-Agent: curl/7.64.1
> Accept: */*
>
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 302
< date: Thu, 20 May 2021 16:13:44 GMT
< content-type: text/html
< content-length: 161
< server: squid/3.5.24
< location: https://www.qq.com/
<
<html>
<head><title>302 Found</title></head>
<body bgcolor="white">
<center><h1>302 Found</h1></center>
<hr><center>squid/3.5.24</center>
</body>
</html>
* Connection #0 to host 192.168.31.135 left intact
* Closing connection 0` 

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
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48
*   49
*   50
*   51
*   52
*   53
*   54
*   55
*   56
*   57
*   58
*   59
*   60
*   61
*   62
*   63
*   64
*   65
*   66
*   67
*   68
*   69
*   70
*   71
*   72
*   73


```

完美！

撒花！！！

最后。

欢迎Start。

欢迎试用。

https://github.com/virjar/echo  
​

### 社区

加V：（virjar1）,备注echo入群

* * *

最后。

不要玩火哈。

毕竟。

爬虫写得好，牢饭吃得早。

手动狗头。