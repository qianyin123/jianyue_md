> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_35491275/article/details/117969108)

### 文章目录

*   [前言](#_1)
*   [第一式：鬼影迷踪](#_62)
*   [第二式：森罗万象](#_91)
*   *   *   [处理逻辑](#_110)
        *   [Step1-处理if...else if...](#Step1ifelse_if_141)
        *   [Step2-补全if...else...](#Step2ifelse_170)
        *   [Step3-平坦化](#Step3_266)
*   [小结](#_535)
*   [参考文章](#_543)

前言
==

最近一直在学习AST相关的知识，本篇文章就来小小的尝试下，利用AST平坦化控制流。

正常的执行逻辑： a -> b -> c

混淆后的执行逻辑则可能是这样：a -> d -> b -> d -> c -> d

![正常流程](https://img-blog.csdnimg.cn/img_convert/ee22fb01815877621859b38019d5b0d2.png)

![混淆后的流程](https://img-blog.csdnimg.cn/img_convert/43091bdee33b80f466cbe4ecf4734123.png)

下方是从瑞数中提出的一段代码。

**源代码**

```
`function func (_$eE) {
  var _$0n, _$2L, _$sC, _$o9, _$NU, _$qo, _$hK;
  var _$tg, _$DM, _$ZZ = 0, _$Ce = [7, 3, 4, 0, 9, 2, 8, 6, 1, 5];
  while (1) {
    _$DM = _$Ce[_$ZZ++];
    if (_$DM < 4) {
      if (_$DM < 1) {
        _$tg = !_$NU;
      } else if (_$DM < 2) {
        _$Uy(0);
      } else if (_$DM < 3) {
        _$NU = _$sC['$_ts'] = {};
      } else {
        _$sC = window, _$hK = String, _$o9 = Array;
      }
    } else if (_$DM < 8) {
      if (_$DM < 5) {
        _$NU = _$sC['$_ts'];
      } else if (_$DM < 6) {
        _$ZZ += -3;
      } else if (_$DM < 7) {
        return;
      } else {
        _$0n = [4, 16, 64, 256, 1024, 4096, 16384, 65536];
      }
    } else {
      if (_$DM < 9) {
        _$ZZ += 1;
      } else {
        if (!_$tg) _$ZZ += 1;
      }
    }
  }
}` 

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


```

先简单介绍上方代码的怎么执行的。

1.  有一个整数数组和下标起始值（在本例中，起始值是`_$ZZ=0`）
    
2.  进入while循环后，通过下标值去数组中取出一个整数，这个整数将经历多次判断，最终执行一条语句。
    

第一式：鬼影迷踪
========

纵使直接在while循环中调试比较复杂，但是执行过的语句总是会留下痕迹。

不知道小伙伴是否看过《三体》，其中的曲率引擎飞船达到光速后会在身后留下“死线”，这条“死线”也就是飞船的航行轨迹。

我们可以在while循环中，加入一些语句，记录下它的执行轨迹。

```
 `var _$tg, _$DM, _$ZZ = _$eE, _$Ce = _$2z[0];
  window.trace = [];
  while (1) {
    _$DM = _$Ce[_$ZZ++];
    window.trace.push(_$DM)` 

*   1
*   2
*   3
*   4
*   5


```

当执行完毕之后，查看`window.trace`就可以看到它的执行轨迹了。

最终通过执行轨迹就可以知道它具体执行了哪些语句。

> **Tips：**  
> 如果函数存在递归，直接加入`window.trace`就不可行了，此时需要加入判定条件，比如判断递归的层数。

这个方法是我自己的胡乱瞎搞的，只是针对非常简单的情况，所以这里就不写详细的过程了。其实，我也更推荐第二种方法，第二种方法是我看**@Nanda** 文章总结出来的，也可以关注下这位大佬的公众号《**爬虫术与道** 》

* * *

第二式：森罗万象
========

相比“鬼影迷踪”，我更推荐“森罗万象”。这将包含所有情况，并且一旦完成可以说是一劳永逸。

因为`_$DM`的值决定了执行流程，所以我们可以动态维护`_$DM`，将整个流程走完就可以拿到所有执行的语句。

需要使用到以下模块：

```
`const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const t = require("@babel/types")
const generator = require("@babel/generator").default;
const path = require("path");
const fs = require("fs");
let os = require("os");` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7


```

### 处理逻辑

先把大致逻辑写出来：

1.  将`if...else if...else` 转为`if...else`语句；
    
2.  补全`if...else...`的大括号；
    
3.  平坦化；
    

```
`fs.readFile(path.resolve(__dirname, './source.js'), {'encoding': 'utf-8'}, function (err, data) {
  const ast = parser.parse(data);

  // 将if...else if...else语句转为if...else语句
  step1(ast);

  // 给没有括号的if...else语句加上括号
  step2(ast);
  
  // 平坦化
  step3(ast);

  let code = generator(ast).code;
  fs.writeFile('./result.js', code, {'encoding': 'utf-8'}, (err) => {
    console.log(err);
  })
})` 

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


```

### Step1-处理if…else if…

先来看一下需要实现什么效果。

```
`if (1) {
  console.log(1);
} else if (2) {
  console.log(2);
} else {
  console.log(3)
}
// 转换为
if (1) {
  console.log(1);
} else {
  if (2) {
    console.log(2)
  } else {
    console.log(3);
  }
}` 

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


```

这一步是为了方便后续的遍历操作，对于一个`IfStatement`的语句

![](https://img-blog.csdnimg.cn/img_convert/6647c41f1859604439b550e48979a3bb.png)

### Step2-补全if…else…

为了确保`IfStatement`的后续节点都是`BlockStatement`所以就需要解决下方这样的情况：

```
`if (!_$tg) _$ZZ += 1;` 

*   1


```

解析上方代码，可以看见`consequent`对应的不是`BlockStatement`而是`ExpressionStatement`。

![](https://img-blog.csdnimg.cn/img_convert/73003461f7ca6f313e9080fcf34a2470.png)

要解决这种情况也是非常的简单，我们只需要用`BlockStatement`将其原本的语句给“包”起来即可。

在原来的基础上加上`{}` 再查看抽象树。

![](https://img-blog.csdnimg.cn/img_convert/9c0b95bcc3e2637d9f2c76ec6ca4e19c.png)

```
`function step2(ast) {
  traverse(ast, {
    ExpressionStatement: funcToStr
  })

  function funcToStr(path) {
    let node = path.node;
    let parentNode = path.parent;
    // 当前节点的父节点也是IfStatement的情况
    if (isIfStatement(parentNode)) {
      path.replaceWith(blockStatement([node]));
    }
  }
}` 

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


```

这一步处理完之后的效果如下。

```
`function func(_$eE) {
  var _$0n, _$2L, _$sC, _$o9, _$NU, _$qo, _$hK;

  var _$tg,
      _$DM,
      _$ZZ = 0,
      _$Ce = [7, 3, 4, 0, 9, 2, 8, 6, 1, 5];

  while (1) {
    _$DM = _$Ce[_$ZZ++];

    if (_$DM < 4) {
      if (_$DM < 1) {
        _$tg = !_$NU;
      } else {
        if (_$DM < 2) {
          _$Uy(0);
        } else {
          if (_$DM < 3) {
            _$NU = _$sC['$_ts'] = {};
          } else {
            _$sC = window, _$hK = String, _$o9 = Array;
          }
        }
      }
    } else {
      if (_$DM < 8) {
        if (_$DM < 5) {
          _$NU = _$sC['$_ts'];
        } else {
          if (_$DM < 6) {
            _$ZZ += -3;
          } else {
            if (_$DM < 7) {
              return;
            } else {
              _$0n = [4, 16, 64, 256, 1024, 4096, 16384, 65536];
            }
          }
        }
      } else {
        if (_$DM < 9) {
          _$ZZ += 1;
        } else {
          if (!_$tg) {
            _$ZZ += 1;
          }
        }
      }
    }
  }
}` 

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


```

### Step3-平坦化

执行流程既然是依靠**整数数组** 和**下标值** ，那么我们就来先来把这两个部分提出来。

在本例中，整数数组与下标变量的赋值均是在`while`循环上方完成，然后我们又是需要平坦化并替换掉`while`节点，所以我们递归的目标就是`WhileStatemtent`

```
`function step2(ast) {
  traverse(ast, {
    WhileStatement: funcToStr
  })
}` 

*   1
*   2
*   3
*   4
*   5


```

查看抽象树，可知`while`循环上方的赋值语句属于它的兄弟节点，如下图所示。

![](https://img-blog.csdnimg.cn/img_convert/0a6f4018078346d6cb00f488675e10b1.png)

获取上一个兄弟节点路径和兄弟节点

```
`var prevSiblingNodePath = path.getPrevSibling();
var prevSiblingNode = prevSiblingNodePath.node;` 

*   1
*   2


```

找到后节点后，获取数组和下标变量名。兄弟节点的类型是`VariableDeclaration`，当对一个节点的结构不熟悉时，就去可视化抽象树上看看。

可以看到`declaraions`中有四个变量声明符，与代码一一对应，我们所需要的就是第三个和第四个。

![](https://img-blog.csdnimg.cn/img_convert/922986c398e3452a9dfb27213bab8e2f.png)

```
`var node = path.node;
var prevSiblingNodePath = path.getPrevSibling();
var prevSiblingNode = prevSiblingNodePath.node;
var subscriptNameNode = prevSiblingNode.declarations[2];
var subscriptName = subscriptNameNode.id.name;
var arrNode = prevSiblingNode.declarations[3];
var arr = arrNode.init.elements;` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7


```

接下来就是“模拟”`while`循环，平坦化执行流。

之前处理成了层层嵌套的`if...else...`语句，就是为了方便此时的遍历。

此时的遍历，可以看成是在遍历一颗**二叉树** ，然后先写出一个大致的框架

```
`function test(node, inx) {
  if (arr[inx] < curNode.test.right.value) {
    // 满足
  } else {
    // 不满足
  }
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7


```

`node`是`BlockStatement` 节点，`inx`是数组的下标值，`curNode.test.right.value`就是`if` 表达式右边的值。

![](https://img-blog.csdnimg.cn/img_convert/e2fb19f1b5c15b5b1bce392b8e3ef067.png)

通过在第一步的操作，`if`后的子节点也都是`BlockStatement`

```
`function test(node, inx) {
  if (t.isExpressionStatement(curNode[0])) {
    // 如果是表达式，表示已经走到底了，将表达式作为结果返回
    return node.body;
  }
  if (arr[inx].value < curNode.test.right.value) {
    // 满足
    return test(node.consequent, inx)
  } else {
    // 不满足
    return test(node.alternate, inx)
  }
}` 

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


```

经过上方代码，已经可以拿到第一个执行语句了，为了拿到所有的执行语句，则需要在每拿到一个语句后就对**下标值+1，** 然后再进行下一次遍历，直到遇到`ReturnStatement`。

> Tips：下标值+1是每次循环的默认操作

```
`function test(node, inx) {
  var resultBody = [];
  while (1) {
    if (t.isExpressionStatement(node[0])) {
      // 如果是表达式，表示已经走到底了，将表达式作为结果返回
      return node.body;
    }

    if (t.isReturnStatement(resultBody[resultBody.length-1])) {
      break;
    }
    
    // curNode 为IfStatement
    var curNode = node.body[1] || node.body[0];

    if (arr[inx] < curNode.test.right.value) {
      // 满足
      resultBody = resultBody.concat(test(curNode.consequent, inx));
    } else {
      // 不满足
      resultBody = resultBody.concat(test(curNode.alternate, inx));
    }
    inx += 1;
  }
  return resultBody;
}` 

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


```

上方代码仍有缺陷，因为`while`循环是写在`test`里面的，所以每次递归进入函数都需要进入一次`while`循环，但是并不是每条语句后面都有`return`，这就会导致死循环。

`while`循环的本来意义是针对最顶层的遍历，所以可以再传入递归深度，**如果递归深度超过1则直接跳出while循环。**

```
`function test(node, inx, depth) {
  depth += 1;
  var resultBody = [];
  while (1) {
    if (t.isExpressionStatement(node[0])) {
      // 如果是表达式，表示已经走到底了，将表达式作为结果返回
      return node.body;
    }

    // curNode 为IfStatement
    var curNode = node.body[1] || node.body[0];

    if (arr[inx] < curNode.test.right.value) {
      // 满足
      resultBody = resultBody.concat(test(curNode.consequent, inx, depth));
    } else {
      // 不满足
      resultBody = resultBody.concat(test(curNode.alternate, inx, depth));
    }

    if (t.isReturnStatement(resultBody[resultBody.length-1]) || depth > 1) {
      // 如果深度超过1，则跳出while循环
      break
    }
    inx += 1;
  }
  return resultBody;
}` 

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


```

在源代码的`while`循环中，**可能因为判断结果而影响下标值** ，例如：

```
 `if (_$DM < 9) {
    _$ZZ += 1;
  } else {
    if (!_$tg) _$ZZ += 1;
  }` 

*   1
*   2
*   3
*   4
*   5


```

下标变量`_$ZZ`是否`+1` 取决于`_$tg`的值，而对于我们在外面的程序因为没有真正执行js代码所以完全不知道`_$tg`的值，也就无法得知这里具体是“走左还是走右”。

所以，最稳妥的方法就是考虑两种情况，分别遍历满足与不满足的两种情况，最终汇总为一个`IfStatement`。

**1、** 找到`if(!_$tg)`

首先它是一个`IfStatement`并且`test`为`UnaryExpression`

**2、** 计算下标值

```
``if (t.isUnaryExpression(curNode.test)) {
  var real_val;
  if (t.isUnaryExpression(curNode.right)) {
    // 注意： 右边的值可能是负数， 例如 a += -1
    var val = curNode.consequent.body[0].expression.right.argument.value;
    var op = curNode.consequent.body[0].expression.right.operator;
    real_val = eval(`${op}${val}`);
  } else {
    real_val = curNode.consequent.body[0].expression.right.value;
  }
  // 获取操作符, += , -=
  var operator = curNode.consequent.body["0"].expression.operator;
  inx = eval(`${inx}${operator.replace('=', '')}${real_val}`);
}`` 

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


```

**3、** 分别遍历两种情况

对于是否满足情况，都是继续调用递归函数，只是传入的实参不同罢了。

例如，条件为真的情况，在实际循环中，下一次遍历必将返回`while`的开头，所以这里传入**初始节点** ，下标值传入计算出来的新值，递归深度为0。

```
`test(initNode, inx+1, 0)` 

*   1


```

这样就可以拿到满足这个条件时的后续执行流程。

对于不满足条件的情况，按正常流程执行即可。

随后调整“亿”点点代码即可完成递归函数。

```
``function test(node, inx, depth) {
    depth += 1;
    var resultBody = [];
    while (1) {

      if (node.body.length === 1 && t.isExpressionStatement(node.body[0])) {
        return node.body
      }

      var curNode = node.body[1] || node.body[0];

      if (t.isReturnStatement(curNode)) {
        return node.body;
      }

      if (t.isIfStatement(curNode)) {
        if (t.isUnaryExpression(curNode.test)) {
          var real_val;
          if (t.isUnaryExpression(curNode.right)) {
            // 注意： 右边的值可能是负数， 例如 a += -1
            var val = curNode.consequent.body[0].expression.right.argument.value;
            var op = curNode.consequent.body[0].expression.right.operator;
            real_val = eval(`${op}${val}`);
          } else {
            real_val = curNode.consequent.body[0].expression.right.value;
          }
          // 获取操作符, += , -=
          var operator = curNode.consequent.body["0"].expression.operator;
          inx = eval(`${inx}${operator.replace('=', '')}${real_val}`);
          resultBody = resultBody.concat([t.ifStatement(curNode.test, t.blockStatement(test(initNode, inx + 1, 0)))])
        } else if (t.isBinaryExpression(curNode.test)) {
          if (arr[inx].value < curNode.test.right.value) {
            // 满足
            resultBody = resultBody.concat(test(curNode.consequent, inx, depth));
          } else {
            // 不满足
            resultBody = resultBody.concat(test(curNode.alternate, inx, depth));
          }
        }
      }

      if (t.isReturnStatement(resultBody[resultBody.length - 1])) {
        break;
      }

      if (depth > 1) {
        break;
      }
      inx += 1;
    }
    return resultBody;
    }`` 

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


```

![平坦化结果](https://img-blog.csdnimg.cn/img_convert/49806a5cec5dfa01506ee7626d965208.png)

小结
==

ast这玩意儿折腾了一个星期，最烦人的部分还是写递归（算法学的差痛苦了。。。），在刚开始学习ast时，无法记住每一个节点的结构，所以需要多使用[可视化工具](https://astexplorer.net/)，不清晰的结果就将代码丢进去瞧瞧。

还有一点，本文的平坦化方法不是最优解，多动手，多练习才是最有效的学习方式。

**共勉！peace！**

参考文章
====

[1] [Js Ast一部曲：高完整度还原某V5的加密](https://bbs.nightteam.cn/thread-417.htm)

[2] [Js Ast二部曲：某V5 “绝对不可逆加密” 一探究竟](https://bbs.nightteam.cn/thread-423.htm)

[3] [《JavaScript AST其实很简单》](https://www.52pojie.cn/home.php?mod=space&uid=557195&do=thread&type=thread&view=me&from=space)