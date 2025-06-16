> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Vdl6reVhLLcLmubKs7VfMg)

准备工具

```
`x32dbg``Exeinfo PE`
```

样本下载：

拿到文件第一步；查壳：

       因为下载文件后电脑的杀毒软件就自动报毒了，可能是用病毒来保护他的程序吧。

       带着病毒进行查壳查出来的是：Borland Delphi ( 2.0 - 7.0 ) 1992 - www.borland.com

       当时忘记截图了所以只写了查出来的内容，病毒则是感染性病毒

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93kvbZiaziaHVXPcImictD0FsibQtfQSFibpSVjclRDVoVhCElxsPibLSeTzWPQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

       去掉病毒后的查壳如下图所示无壳：

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93kxMtafUzPxGuk7x2mZTjxxq6iaxELfe8L6b06FSDNCX7lpt2rmIEfpWA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

将文件拖入x32dbg中

运行并观察程序

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93kUF5sOzLw9iah6jGbXL7UaDQXIQN0JmiafRtP0p6nkf68ZfZ1Jpu6DITQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

如图可见一个登录窗口，随意输入两串数字点击登录观察。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93k69Sjd6picdr8b3mjy9RTgCf5SSvG2aZtooUVMUNibN7OvOYxgXPDraQg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

出现了登录失败信息框，那么就可以下MessageBoxA断点。

下断点，按住Ctrl+G，在输入框内输入MessageBoxA，点击确定后在相应的地址按F2下断点。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93kRCupKvTa7vnDbbQDIbNky6VicjMHRIkqSF1NpUmhezTZciaCAJkj1BYw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

继续运行程序查看堆栈

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93k8ndwgiaiaCuLKHt7Lw4jaKtnZzwic6jKOAZYiaD5xQ1kYVm4dZG80bTghg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

可以看到“返回到 安妮双城aexe.004629f0 自 ???”上面出现了账号不存在，而后面没有了，我们双击这条消息跳转到004629F0

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93koUO1vYEYoenicuJnTSsiciaq87dMiaqwOanoZrlfbEBeyEoXEqicQWd2lcw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

可以看到上方有一条call，这call我的理解为一个按钮事件，我们在这call下一个断点。

重新运行并观察寄存器

运行到刚刚下断点的call后，按F7进入这call，这时候我们可以在进来的地方下一个断点，这样如果重新运行可以少一个步骤

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93kibjb30YbvhWTKwhuJKwx72W63XSkrpg7lZrhhH8gDtbtP1Y4eLicO18g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

接下来我们不断的按f8并观察寄存器

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93kFfmxVP53XUNXUlILCibO5tpefyF0HWXtF9qoaclnWB9Y1Lt6Tgao8yw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

在0040EEC7这里有一个大的跳转，我们下一个断点。继续F8

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93kLjZalGG8VGo5wX47yqbNxibNlK6ibrz7HywoEZAtcVz1ZnKiaRynuZXLA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

F8跳转之后到达了0040F4C4,而上方有个jmp，看来是不应该跳下来的，继续F8

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93kgsBA9PwYj93rINRXbx6Lq15aJX0CH666ZIicVsJ9T7LjLVb9bmePRzQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

登入失败信息框又出来了，重新登录。

重新登录后回到上一个断点的地方，既然不能跳转那就不跳了，右键跳转语句，二进制-->>用nop填充。

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0qX28ibxxHgiaezeM6SQAZ93kSMiamVKywENc8k852NqRiceO1ibvaIeh2dlBsHBImvibr32Sw0O6A9LLpQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

nop掉之后继续按f8

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

发现到0040EF46这个call后程序崩溃了，下个断点并重新运行。

重新运行到0040EF46这个call，按f7进入，下一个断点

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

按F8继续运行并观察寄存器

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

此时运行到00420FCA这个call后，寄存器发生了变化，并给这个call下个断点

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

此时EAX变为了“-1”,那这个call很有可能就是计算时间的call，右键EAX，选在转储中跟随

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

选中00514AB6这个地址，右键-->>二进制-->>编辑

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

在ASCII文本框，将其中的-1修改为9999999，点击确定并F2运行

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

此时提示登录成功，并出现了到期时间，点击确定后出现了程序界面

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

破解成功，打上补丁修补文件。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

  

  

  

**·** **今 日 推 荐** **·**

  

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

本文内容来自网络，如有侵权请联系删除