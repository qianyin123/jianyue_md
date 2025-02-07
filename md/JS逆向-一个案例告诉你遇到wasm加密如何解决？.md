> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/jRxYRPDnU64arGO-Nxe3mg)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLswk8wwATj3XsUubR9yprAz5zUSibJJKVhblVb8k0iclaI0UZC6dyjp0A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**点击蓝字，关注我们**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**网址信息**

  

目标网站：aHR0cHM6Ly93d3cud2FpbWFveGlhLm5ldC9sb2dpbg== 分析登录的sign加密参数

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**定位分析**

我们先填写信息，点击登录，然后可以看到抓到一个数据包如下

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLHf3BvouBe6yXJvFXjnwbbVibQeXPquMZevW7qJtE024QNwHibcJYlKpw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里的sign就是我们今天的目标，我们主要是找到sign如何实现的，并且如何在我们本地进行运算得到正确的结果。

首先我们还是老样子找堆栈。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLxPTicnEu8IJCZrib5S9cckREEg7XbIbMGWAHbfZf9uEQ9LaH5Mzo24Uw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们其实可以看到，这里的堆栈不是有很多，根据我们经验和这些方法名分析，我们觉得最有可能的就是post和getUserLoginAction这两个方法实现的，我们先点击post打个断点尝试一下。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLY7yGnCDGHzanlVnwFiafw2jDQQjolJTibgvMqTUWiaWfcxQibUzysYPSRw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里我们其实是可以看到，sign已经计算出来了，所有我们向上查找，看看是否有可疑的地方。

我们找了几个发现这个地方比较可疑，方法都是用sign命名的。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLvZ68S8MRYKMRibrLp35uXpVia6HiaWndpRe1DKTGhDFiaTkFSFBzicpdhUg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们跟上去看到代码是如下这个样子的。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLcEfAPWH2m6lbcCKYwdjEC2YJRXd4ZRA7iaXnCsf0I1EBdicbYAqYCSow/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLhMkEn6geqlvplXicRUmX6DW9nibVCJbWGGibJxtyhboxLqZRH04kQLVxg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们在点击的时候就看到他文件是wasm_xxx这个样子，所以这里我们就知道他是wasm文件去调用的js文件，并且我们在js文件也看到很多异步操作，所以这里我们更加确定是wasm运行。

我们再看一下我们网络请求哪里有没有wasm文件返回。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKL2zS2U96kRdApThZibhtKP9U7EPLwy1GWibOgsxCcPVr77ibhF8ysjJG7w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们看到其实是有wasm的网络请求。到这里我们就已经基本了解了情况，我们接下来就是需要用nodejs来实现这整一个流程。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**wasm逆向流程**

首先我们先要了解要完整实现wasm的调用运行，我们需要什么东西。

这里如果你有学习过正向开发wasm的知识那你就会知道，如果没有，哪也没有关系，我这里简单的说一下。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLRDGZXwLjYsUTk5Y6JQFicBNrcfurmH6yYcOW9QTKyrR1kyZW8UmVGhw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

一般情况下，所有wasm文件调用都会使用到这些参数才能得到正确的结果，并且他可能会是异常区执行的。那么接下来，我们就继续回归主题继续操作。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**本地运行**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLkvXv1yibh9yJZYq2QwKXVhJzdM61tlsOAyWW025zQXmX7ibHHDzsFlQA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

把这些代码全部复制到本地，

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLxF1mNXwVuicHYuWAhnvm2p6KVdzmHLEr576dYXrZtaLhzibOVUE7z3uQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们重点要关注这个地方，这里其实读取wasm文件并且还要把所需要的env环境一起加载到内存中，然后通过run进行初始化。所以我们需要找到所有的env环境，他已经帮我们定义好了。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLib75OuIrTA0QtRyvm2eicXNcOwFMqKABCQvedB0EDstTVXP6dUdzb8Lg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

就是这里的importobject这个对象，这里就是他所需要的env环境。

这里我对他这个文件进行改造，使其可以正常的读取和初始化。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKL7HW6VibbblOF2fqUpgZgRPXqcSzpia0DNRZ6212T1jEicP1GXC6bpNvicg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

接下来我们点击对应的sign函数。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLsQkSsG1SAnvPu6QiafrQO8ibWnNxXGAPF5hqTfE97e0rkLPZT4pfGvSQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们可以看到这里就是最终获取结果的地方。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLznBn8sj4PZfyS1vUrnkAcmPXaou58dkf0cH0Ea6JOA80Ffgic4DnPMQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上参数其实就是计算所需要的参数。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLbkDPnATLHCsflcicWYb5M3QqiaZn5dTVGkia3n7F8SmZlyGMgfIibw01pw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过先调用函数，得到函数内的函数，然后再继续调用得到正确的结果。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**运行结果**

通过，把环境补充网站然后在执行我们的代码，得到sign加密结果。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLMgmAxHSlZpmTqYISUbPyJco3I5J8pXlF8BKIhx0ke0Og6W85VKickjg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们在根据网站进行对比一番验证结果是否正确。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLoiaicKzmX3LjpCXz06hRdBrAVibibsnRdH3IZbVYCpMsawsYotd3KCicticg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们可以看到结果是没有问题，也可以正确的得到我们的完整结果，说明我们整个流程运算的结果是没有错误的。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**🤗 总结归纳**

这篇文章主要是讲解如何解决有这种wasm文件的逆向，怎么从0到1的解决问题。

  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/N1Fms3bbiaBBRzEA3k4zyas2vjz32waEsmHPp1cFpcyMicpZgfOCKAibKnDUU6IFgJUQVaicKP5vp8hMwGeNDX6Z9A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**扫**

**码**

  

**加**

**我**

学习知识联系我

![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/N1Fms3bbiaBAWeP5w5MZo524ETLmxCxKLS0VT5Ab0k6dKYSH9iajib7eIXQicg3djHxW7VqQ65LPUyyibRo2SoRcfXQ/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)