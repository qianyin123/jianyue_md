> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-ERHWKvhssJE4NEz_HTIPg)

从MuMu模拟器上下载v8.4.0的moji辞典没有防hook机制（有的话需要把这步加上），所以这里提供一个使用frIDA动态hook的思路

环境：MuMu模拟器12

第一步 会员搜索大法

把apk从模拟器中pull出来，然后拖到jadx中，搜索：会员![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0osibTx5mcb05r7gMUMIUG56ict5AWnjgpDibaqk4ojkhUxBhUDy3jfWFpBS7CxKU4bHibeAse0HWjAtg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

找到 “已开通永久会员”，这个字符串的变量名是 purchase_has_lifetime_vip

第二步 查看调用位置

全局搜索 purchase_has_lifetime_vip，定位类 MainMinePrivilegeCardViewHolder

![图片](https://mmbiz.qpic.cn/mmbiz_png/WJRHqUiaud0osibTx5mcb05r7gMUMIUG56VzmX8ZSgicibhbLxGia3YdwPiauiarPQ5YOZrRGoTcOuvz9JHCM5BiawSwQw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

简单分析逻辑后可以得知前面 if 语句中的条件决定了是否开通永久会员

第三步 写hook脚本

点进a6.a.c后复制为frida片段，写hook代码：

```
`Java.perform(function () {` `let a = Java.use("a6.a");` `a["c"].implementation = function (rVar) {` ``console.log(`a.c is called: rVar=${rVar}`);`` `let result = this["c"](rVar);` ``console.log(`a.c result=${result}`);`` ``console.log(`a.c return=true`);`` `return true;` `};``});`
```

接着加载hook脚本

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

实际效果

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

尝试下会员的手写功能

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

居然翻车了...

翻回去发现MainMinePrivilegeCardViewHolder类里还有个高级会员，死马当活马医了

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

写新的hook脚本

```
`Java.perform(function () {` `let g = Java.use("a6.g");` `g["f"].implementation = function () {` ``console.log(`g.f is called`);`` `let result = this["f"]();` ``console.log(`g.f result=${result}`);`` ``console.log(`g.f return=true`);`` `return true;` `};` `g["e"].implementation = function () {` ``console.log(`g.e is called`);`` `let result = this["e"]();` ``console.log(`g.e result=${result}`);`` ``console.log(`g.f return=true`);`` `return true;` `};``});`
```

再来一次

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

再试试手写

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

成了！收工

  

**·** **今 日 推 荐** **·**

  

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

  

本文内容来自网络，如有侵权请联系删除