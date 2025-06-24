> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/okTRo60cjYTBjFbey8FzwQ)

**提示！本文章仅供学习交流，严禁用于任何商业和非法用途，如有侵权，可联系本文作者删除！**

* * *

### 前言

又是很久没写文章了，今天水一篇文章吧，鉴于之前有看到过别人的文章被警告的案例，所以这篇文章就仅写一下还原瑞数vmp的大概流程吧（仅记录下自己还原的思路）

* * *

### 网站链接：aHR0cDovL2VwdWIuY25pcGEuZ292LmNuL3BhdGVudC9DTjEwNTYzNzU1NUI=

* * *

#### 算法还原流程

瑞数大家应该都很熟悉了，已经经历过好几个版本迭代了，像前面的3代、4代、5代等，这里借用k哥文章[人均瑞数系列，瑞数 4 代 JS 逆向分析](https://mp.weixin.qq.com/s?__biz=Mzg5NzY2MzA5MQ==&mid=2247488564&idx=1&sn=0d0faad2a8a6354e10b5b112a37af3d1&scene=21#wechat_redirect)里描述的，看请求，会有典型的三次请求，首次请求响应码是 202（瑞数3、4代）或者 412（瑞数5代），接着单独请求一个 JS 文件，然后再重新请求页面，后续的其他 XHR 请求中，都带有一个后缀，这个后缀的值是由 JS 生成的，每次都会变化，后缀的值第一个数字为瑞数的版本，比如**MmEwMD=4xxxxx**就是4代瑞数， **bX3Xf9nD=5xxxxx**就是5代瑞数，更加具体的描述直接看k哥的文章就好，因为这篇是关于瑞数vmp篇的，所以下面就说瑞数vmp的流程，首先在访问网站的时候打上**Script**断点，之后就能看到如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic2RQ1ZJibV3Xib2C2qcjbgpPvwbJkibtkYDPfMXJCcH5FCG47DtsYaT38fiaS5u2m0AiblCmcvKESmZibiaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

上面圈出来的部分后面会用到，然后再看看**Network**面板中，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/mlPoGiabbRic2RQ1ZJibV3Xib2C2qcjbgpPvrEiaGRqEt0KpCDHicIXWL1wfkchqveSCv8l5YuJfsia4MnMibyqXsibndcQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

上图可以看到响应码202，同时还有一个Set-cookie，这个cookie是后面的请求需要用到的，3lyVKWvJXsxS.ce3512e.js这个js文件会将上面从202页面中获取到的内容先生成一大段js，这段js在前面几代中都是可以通过.call这个关键词来定位的，所以这里姑且就称之为call中js，之后通过eval的方式来执行这个js，由此来得到后面请求所需要的cookie，这一步的cookie长度是214，也就是下图圈出来的：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

整个流程和瑞数前几代的过程是一样的，就实现的代码发生了变化，在还原某数的时候，我是先将3lyVKWvJXsxS.ce3512e.js这个文件的过程进行了还原，欲善其功，必先利期器，既然要还原，那就直接还原全套的，在还原这个js的过程中能够了解到它生成整个call中js的过程以及过程中用到的参数及最后生成的参数，这部分还原出来大概300来行，如下：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

然后简单说一下3lyVKWvJXsxS.ce3512e.js里面做了些什么：

```
1.一开始获取一个时间戳  
2.将是上图中$_ts.nsd参数拿来生成一个参数名数组，因为每次请求时获取到的$_ts.nsd参数数值是不一样的，所以每次生成的参数名数组也是变化的，这也就解释了为什么每次请求时看到的call中js里面的代码都是变化的原因，其整个逻辑没有变，变的仅仅是参数而已。这里参数名数组生成完毕之后会将其放入_$lY数组中，步骤如下：  
  _$lY[1] = 变量名数组  
3.这一步是给aebi数组存值，这里面会有6组数组，会影响到call中js的执行逻辑，每个数组分别对应着call中js的每个while循环的逻辑，这个值是个固定值。  
4.将上面生成的参数名数组依次填入代码模板中，这一步就是完成整个call中js代码的生成  
5.将一些固定参数放入_$lY数组中，如下：  
  _$lY[0] = xxx1   //直接在js中搜索就可以看到了  
  _$lY[2] = xxx2   //直接在js中搜索就可以看到了  
  _$lY[6] = ""     //直接在js中搜索就可以看到了  
6.这里获取时间戳，然后与一开始时获取的时间戳做差，生成一个时间戳差值  
  _$lY[4] = 时间戳差值  
7.上面已经完成了call中js的生成，之后就是将这个call中js通过固定算法生成一个值，之后将这个值放入_$lY中  
  _$lY[3] = 生成的值  
8.通过eval来执行call中js，生成wIlwQR28aVgbT  
9.这里再次获取时间戳，然后与一开始时获取的时间戳做差，生成一个时间戳差值，之后将这个值放入_$lY中  
 _$lY[5] = 时间戳差值
```

整个流程大概就是这个样子了，**_$lY**这个数组在生成**wIlwQR28aVgbT**参数的时候会用到， **_$lY[5]**这个是在生成**wIlwQR28aVgbT**之后放入的，所以这一步可以不需要，到这里**3lyVKWvJXsxS.ce3512e.js**这个js所做的事情已经说完了，后面还有一个叫**7VAtOUmTEj4k.ce3512e.js**的文件，这个是后面获取**363**长度的**wIlwQR28aVgbT**时所用到的，整个流程也是如此。

上面前几个步骤完成之后生成的**$_ts**参数如下：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

之后再来说下今天的重点，也就是生成**wIlwQR28aVgbT**参数的大概步骤：

```
1.通过上图中的cp[0]和cp[2]来获取还原变量名的数组，总共有两个，一个数组是用来还原变量名的，一个是用来还原固定参数的（这个数组姑且称为固定参数数组），这一步在正式的还原算法逻辑中是可以不需要的，不过为了方便分析call中js代码，需要将代码还原的时候会用到  
2.通过上图中cd值来获取一个长度在1300多位的数组，这个数组的长度不固定，根据cd值变化，这个数组姑且称为初始数组，  
3.根据这个1300多位的数组通过一些计算获取一个包含4组数组的数组，过程如下:  
a.取数组第一个值+2，获取下一组数组取值的最大范围  
b.从生成的初始数组中根据范围获取数组  
c.根据cp[3]获取一个16位数组  
d.根据b步骤中获取的数组、c步骤中获取的16位数组及第一步中获取的固定参数数组生成一个新的数组  
e.根据d步骤中获取的数组及固定参数数组生成包含4组数组的数组  
4.通过这个包含4组数组的数组来通过特定计算获取一个8位数组（这个8位数组很关键）  
5.获取第3步d步骤中数组的长度并+2，之后通过slice从初始数组中获取一个1200多位长度的数组，因为初始数组长度变化的，所以该数组长度也是变化的  
6.根据第5步中获取的数组及获取的8位数组生成一个新的1200多位长度的数组  
7.根据这个新的1200多位长度的数组生成35位数组 （后面取35位索引的第几位，就直接用array35_index表示了）  
8.获取时间戳，姑且称为初始时间戳  
9.生成_$cn参数和$ne参数，参数名是变化的，根据自己分析的代码来即可，大致步骤如下：  
a.取35位数组中索引位为16的数组(array35_16)，根据这个数组生成一个21位数组  
b.根据35位数组获取一段字符  
c.根据a步骤中获取的21位数组生成新的21位数组  （这一步中涉及时间戳及随机值，分析的对比的时候固定一下）  
d.根据c步骤中获取的21位数组及b步骤获取的一段字符获取_$cn参数，这是一个8位数组  
e.从c步骤中获取的21位数组中通过slice(16,20)取4位数组  
f.通过e步骤中获取的4位数组和初始时间戳生成8位数组  
g.将该8位数组和""通过函数_$ne生成所需的_$ne数组  
----下面这部分我是分成了五个部分，每个部分会生成一个数组,且有些数组在这个期间会在其它步骤中涉及到存值等操作----  
9.user_agent生成第一步的数组，长度16位  
10.生成第二步的数组，长度19位 // 这一步会涉及到时间戳和随机数  
11.生成第三步的数组，长度12位 // 这一步需要用到生成的call中js，具体步骤就不说了，大家自行研究  
12.生成第四步的数组，长度4位  // 这一步用到了第一部分生成的数组（到这一步第一部分生成的数组是经过多次存值之后的）和第三部分数组  
13.生成第五部分数组，也就是后面要用到的84位数组 //这一步用到前面生成的_$cn数组、多次存值之后的第一部分数组、第四部分数组  
  
9-13这部分再做个说明，第一步的数组在后面几步中大致步骤就是将其它步生成的数组存入第一步生成的数组中  
-----------  
14.用生成的84位数组获取74/75/76位数组，长度不定  
15.用array35_2和14步中生成的数组通过固定函数及固定数值16生成最新的74/75/76位数组 //array_74_or_75_or_76  
16.前面生成的_$ne数组和array35_2生成44位数组 //array44  
17.array35_17和array_74_or_75_or_76通过固定算法生成80位数组  //array80，这一步涉及时间戳和随机数  
18.array44和array80通过固定算法生成125位数组  //array125  
19.array125通过固定算法生成129位数 //array129  
20.array129和array35_16通过固定算法生成160位数组 //array160，生成21位数组时涉及到时间戳和随机数  
21.array160通过固定函数生成最终的cookie
```

到这里整个大致的步骤就说完了，最后还原出来大概2000多行，这篇说的的也仅仅是长度为**214**位的**cookie**，后面还有长度为**363**和**406**的**cookie**，这里就不做过多说明了，大家在这个基础上研究就好了。  

最终还原出来到结果如下:

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)