> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XIAtuBI3RGfSUoUpL0s8dA)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/5aP6U4veSuxrk54SicTngC68j9xsVJ63u7nQHCYliaqKqdSqic8ibJFAtz4qCFCewboVxzlSHKia4IDY3GXiaWBdSECQ/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

  

简介
--

> ❝
> 
> Android中的Scheme是一种页面跳转协议， 和网站通过URL的形式访问一样，APP同样可以 通过这种方式进行跳转，它可以很方便的满足我们在一些场景中的需求

  

场景案例
----

```
自动化控制手机进入指定某音、某书博主主页，并使用  
mitmproxy拦截响应包采集数据  

```

**常规方式：搜索-输入id-点击**

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**schema协议跳转**

只需要一行adb命令，即可跳转（相当于url直达）

```
adb shell  am start -a android.intent.action.VIEW -d  
snssdk1128://user/profile/68618717249396  

```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

定位
--

```
样式：[scheme]://[host]/[path]?[query]  

```

不同apk，不同页面的schema协议不同，不像网站url点击之后可以在导航栏看到，一般协议url都需要自行寻找

一般apk会在AndroidManifest.xml中增加设置Scheme，比如某音

```
<activity android:launchMode="singleTask" android:@style/t">  
  <intent-filter>  
    <action android:/>  
    <category android:/>  
  </intent-filter>  
  <intent-filter>  
    <action android:/>  
    <category android:/>  
    <category android:/>  
    <data android:scheme="snssdk1128"/>  
  </intent-filter>  
</activity>  

```

剩下的路径和参数需要自己寻找了，下面列举下几个方式：

#### web端跳转

拿h5的某音商品详情来看，点击立即购买 会有个含有schema的包，这个就是详情详情的schema

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

#### jadx搜索

知道了协议头为snssdk1128 去jadx搜也可以拿到

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

分享
--

下面分享一些某书、某音的跳转了解

dy

```
//个人主页  
snssdk1128://user/profile/92514323808  
  
// 商品详情  
snssdk1128://ec_goods_detail?promotion_id=353925891431717476  
  
// 店铺  
snssdk1128://goods/store?sec_shop_id=RjgfJJPA  
  
// 视频详情  
snssdk1128://aweme/detail/6683443624597916941  

```

xhs

```
// 笔记  
xhsdiscover://item/5f34197f000000000101e575  
  
// 主页  
xhsdiscover://user/5f12d8cf0000000001004c15  
  
// 商品搜素  
xhsdiscover://instore_search/result?keyword=口罩  
  
// 普通搜索  
xhsdiscover://search/result?keyword=美女  
  
// 他的粉丝  
xhsdiscover://user/5fa79f0f0000000001002ed7/followers  
  
// 关注列表  
xhsdiscover://home/follow  
  
  
  
xhsdiscover://account/bind/ //账号与安全  
xhsdiscover://choose_share_user //分享给用户  
xhsdiscover://dark_mode_setting //深色设置  
xhsdiscover://video_feed/id //视频作品页  
xhsdiscover://general_setting/ //通用设置  
xhsdiscover://hey_home_feed/ //记录我的日常  
xhsdiscover://hey_post/ //发布语音  
xhsdiscover://home //主页  
xhsdiscover://home/explore //发现列表  
xhsdiscover://home/follow //关注列表  
xhsdiscover://home/localfeed //同城列表  
xhsdiscover://home/note //关注列表  
xhsdiscover://home/store //商城  
xhsdiscover://instore_search/result //商品搜索  
xhsdiscover://instore_search/result?keyword= //商品搜索关键词  
xhsdiscover://item/id  //文字作品页  
xhsdiscover://item/id?type=normal  //文字作品页  
xhsdiscover://item/id?type=video //视频作品页  
xhsdiscover://search/result?keyword= //搜索关键词  
xhsdiscover://me/profile //编辑资料  
xhsdiscover://message/collections //收到的赞和收藏  
xhsdiscover://message/comments //收到的评论和@  
xhsdiscover://message/followers //新增关注  
xhsdiscover://message/notifications //系统通知  
xhsdiscover://message/strangers ,//陌生人消息  
xhsdiscover://messages //消息  
xhsdiscover://notification_setting ,//通知设置  
xhsdiscover://post //发布作品-相册  
xhsdiscover://post_note //发布笔记  
xhsdiscover://post_video //发布视频  
xhsdiscover://post_video_album //发布视频-全部相册  
xhsdiscover://profile //我的个人页面  
xhsdiscover://instore_search/recommend //商品搜索  
xhsdiscover://recommend/contacts //通讯录好友  
xhsdiscover://recommend/user //推荐用户  
xhsdiscover://search/result //搜索  
xhsdiscover://store //商城  
xhsdiscover://system_settings/ //开发者模式,可以修改登陆账号  
xhsdiscover://topic/v2/keyword //话题  
xhsdiscover://user/user_id  //用户主页  
xhsdiscover://user/id/followers //TA的粉丝  
xhsdiscover://topic/v2/64140f23d044190001d68e77  // 话题  

```

END
---

> ❝
> 
> 有需要用于爬虫、逆向、抓包的测试机又不想花时间折腾Root、刷机、环境配置可以来看下下面联系方式，或者B站视频评论区见

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)