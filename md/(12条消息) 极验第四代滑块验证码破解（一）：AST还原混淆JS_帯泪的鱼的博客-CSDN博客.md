> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_42857999/article/details/122364575)

### 极验第四代滑块验证码破解（一）：AST还原混淆JS

*   *   [声明](#_3)
    *   [一、环境安装](#_19)
    *   [二、AST还原混淆JS](#ASTJS_22)
    *   *   [1. 需要还原的js代码链接](#1_js_26)
        *   [2. AST还原源码](#2_AST_31)
        *   [3. 极验不同js或不同版本还原方式](#3_js_278)
    *   [三、结语](#_282)
    *   *   [*本期文章结束啦，如果对您有帮助，记得收藏加关注哦，后期文章会持续更新 ~~~*](#__284)

声明
--

**原创文章，请勿转载！**

**本文内容仅限于安全研究，不公开具体源码。维护网络安全，人人有责。**

**本文关联文章超链接：**

1.  [极验第四代滑块验证码破解（一）：AST还原混淆JS](https://blog.csdn.net/qq_42857999/article/details/122364575)
2.  [极验第四代滑块验证码破解（二）：滑块缺口识别](https://blog.csdn.net/qq_42857999/article/details/122364690)
3.  [极验第四代滑块验证码破解（三）：滑块轨迹构造](https://blog.csdn.net/qq_42857999/article/details/122364712)
4.  [极验第四代滑块验证码破解（四）：请求分析及加密参数破解](https://blog.csdn.net/qq_42857999/article/details/122364731)

**首先，极验第四代验证码加密方式和极验第三代差不多。所以，有看过我之前极验三系列破解方式的小伙伴，破解起来还是比较轻松的。**  
**本系列文章为极验第四代滑块验证码破解，内容会对比极验三代的破解过程，会有些冗余，但很细致，小伙伴们仔细学习哦。**

一、环境安装
------

环境安装，请查看极验三破解链接：[极验滑块验证码破解与研究（一）：AST还原混淆JS](https://blog.csdn.net/qq_42857999/article/details/121586389#_17)

二、AST还原混淆JS
-----------

**注意：极验三代、四代js代码混淆方式一模一样，所以用极验三的AST还原函数改一下即可。**  
AST还原函数详解，请查看极验三破解链接：[极验滑块验证码破解与研究（一）：AST还原混淆JS](https://blog.csdn.net/qq_42857999/article/details/121586389#ASTJS_45)

### 1. 需要还原的js代码链接

```
`https://static.geetest.com/v4/static/v1.4.4/js/gcaptcha4.js` 

*   1


```

### 2. AST还原源码

```
`const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const t = require("@babel/types");
const generator = require("@babel/generator").default;
const fs = require("fs");

// #######################################
// 还原需要用到的js源码
// #######################################
khCTZ.$_AY = function() {
    // 从gcaptcha4.js源码中抠出来
}();
khCTZ.$_BS = function() {
    // 从gcaptcha4.js源码中抠出来
}();
khCTZ.$_Cl = function() {
    // 从gcaptcha4.js源码中抠出来
};
khCTZ.$_DX = function() {
    // 从gcaptcha4.js源码中抠出来
};
function khCTZ() {}

// #######################################


// #######################################
// AST解析函数
// #######################################
// 删除节点中的extra属性（二进制、Unicode等编码 -> utf-8）
function replace_unicode(path) {
    let node = path.node;
    if (node.extra === undefined)
        return;
    delete node.extra;
}

// 定义一个全局变量，存放待替换变量名
let name_array = [];

function get_name_array(path) {
    let {kind, declarations} = path.node
    if (kind !== 'var'
        || declarations.length !== 3
        || declarations[0].init === null
        || declarations[0].init.property === undefined)
        return;
    if (declarations[0].init.property.name !== "$_Cl")
        return;
    // 获取待替换节点变量名
    let name1 = declarations[0].id.name
    // 获取待输出变量名
    let name2 = declarations[2].id.name
    // 将变量名存入数组
    name_array.push(name1, name2)

    // 删除下一个节点
    path.getNextSibling().remove()
    // 删除下一个节点
    path.getNextSibling().remove()
    // 删除path节点
    path.remove()
}

function replace_name_array(path) {
    let {callee, arguments} = path.node
    if (callee === undefined || callee.name === undefined)
        return;
    // 不在name_array中的节点不做替换操作
    if (name_array.indexOf(callee.name) === -1)
        return;
    // 调用khCTZ.$_Cl函数获取结果
    let value = khCTZ.$_Cl(arguments[0].value);
    // 创建节点并替换结果
    let string_node = t.stringLiteral(value)
    path.replaceWith(string_node)
}

function replace_$_Cl(path) {
    let {arguments, callee} = path.node
    // 解析arguments参数
    if (arguments.length !== 1) return;
    if (arguments[0].type !== 'NumericLiteral') return;

    // 解析callee
    if (callee.type !== 'MemberExpression') return;
    let {object, property} = callee;
    if (object.type !== 'Identifier' || property.type !== 'Identifier') return;

    if (property.name === '$_Cl') {
        // 计算值
        let value = khCTZ.$_Cl(arguments[0].value);
        // 创建节点并替换
        let string_node = t.stringLiteral(value)
        path.replaceWith(string_node)
    }
}

// 控制流平坦化
function replace_ForStatement(path) {
    var node = path.node;

    // 获取上一个节点，也就是VariableDeclaration
    var PrevSibling = path.getPrevSibling();
    // 判断上个节点的各个属性，防止报错
    if (PrevSibling.type === undefined
        || PrevSibling.container === undefined
        || PrevSibling.container[0].declarations === undefined
        || PrevSibling.container[0].declarations[0].init === null
        || PrevSibling.container[0].declarations[0].init.object === undefined
        || PrevSibling.container[0].declarations[0].init.object.object === undefined)
        return;
    if (PrevSibling.container[0].declarations[0].init.object.object.callee.property.name !== '$_DX')
        return;

    // SwitchStatement节点
    var body = node.body.body;
    // 判断当前节点的body[0]属性和body[0].discriminant是否存在
    if (!t.isSwitchStatement(body[0]))
        return;
    if (!t.isIdentifier(body[0].discriminant))
        return;

    // 获取控制流的初始值
    var argNode = PrevSibling.container[0].declarations[0].init;
    var init_arg_f = argNode.object.property.value;
    var init_arg_s = argNode.property.value;
    var init_arg = khCTZ.$_DX()[init_arg_f][init_arg_s];

    // 提取for节点中的if判断参数的value作为判断参数
    var break_arg_f = node.test.right.object.property.value;
    var break_arg_s = node.test.right.property.value;
    var break_arg = khCTZ.$_DX()[break_arg_f][break_arg_s];

    // 提取switch下所有的case
    var case_list = body[0].cases;
    var resultBody = [];

    // 遍历全部的case
    for (var i = 0; i < case_list.length; i++) {
        for (; init_arg != break_arg;) {

            // 提取并计算case后的条件判断的值
            var case_arg_f = case_list[i].test.object.property.value;
            var case_arg_s = case_list[i].test.property.value;
            var case_init = khCTZ.$_DX()[case_arg_f][case_arg_s];

            if (init_arg == case_init) {
                //当前case下的所有节点
                var targetBody = case_list[i].consequent;

                // 删除break节点，和break节点的上一个节点的一些无用代码
                if (t.isBreakStatement(targetBody[targetBody.length - 1])
                    && t.isExpressionStatement(targetBody[targetBody.length - 2])
                    && targetBody[targetBody.length - 2].expression.right.object.object.callee.object.name == "khCTZ") {

                    // 提取break节点的上一个节点AJgjJ.EMf()后面的两个索引值
                    var change_arg_f = targetBody[targetBody.length - 2].expression.right.object.property.value;
                    var change_arg_s = targetBody[targetBody.length - 2].expression.right.property.value;

                    // 修改控制流的初始值
                    init_arg = khCTZ.$_DX()[change_arg_f][change_arg_s];

                    targetBody.pop(); // 删除break
                    targetBody.pop(); // 删除break节点的上一个节点
                }
                //删除break
                else if (t.isBreakStatement(targetBody[targetBody.length - 1])) {
                    targetBody.pop();
                }
                resultBody = resultBody.concat(targetBody);
                break;
            } else {
                break;
            }
        }
    }
    //替换for节点，多个节点替换一个节点用replaceWithMultiple
    path.replaceWithMultiple(resultBody);

    //删除上一个节点
    PrevSibling.remove();
}

// 删除无关函数
function delete_func(path) {
    let {expression} = path.node
    if (expression === undefined
        || expression.left === undefined
        || expression.left.property === undefined)
        return;
    if (expression.left.property.name === '$_AY'
        || expression.left.property.name === '$_Cl'
        || expression.left.property.name === '$_BS'
        || expression.left.property.name === '$_DX'
    ) {
        path.remove()
    }
}

// #######################################


// #######################################
// AST还原流程
// #######################################
// 需要解码的文件位置
let encode_file = "gcaptcha4.js"
// 解码后的文件位置
let decode_file = "gcaptcha4_decode.js"

// 读取需要解码的js文件, 注意文件编码为utf-8格式
let jscode = fs.readFileSync(encode_file, {encoding: "utf-8"});

// 将js代码修转成AST语法树
let ast = parser.parse(jscode);
// AST结构修改逻辑
const visitor = {
    StringLiteral: {
        enter: [replace_unicode]
    },
    VariableDeclaration: {
        enter: [get_name_array]
    },
    CallExpression: {
        enter: [replace_name_array, replace_$_Cl]
    },
    ForStatement: {
        enter: [replace_ForStatement]
    },
    ExpressionStatement: {
        enter: [delete_func]
    },
}

// 遍历语法树节点，调用修改函数
traverse(ast, visitor);

// 将ast转成js代码，{jsescOption: {"minimal": true}} unicode -> 中文
let {code} = generator(ast, opts = {jsescOption: {"minimal": true}});
// 将js代码保存到文件
fs.writeFile(decode_file, code, (err) => {
});` 

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
*   74
*   75
*   76
*   77
*   78
*   79
*   80
*   81
*   82
*   83
*   84
*   85
*   86
*   87
*   88
*   89
*   90
*   91
*   92
*   93
*   94
*   95
*   96
*   97
*   98
*   99
*   100
*   101
*   102
*   103
*   104
*   105
*   106
*   107
*   108
*   109
*   110
*   111
*   112
*   113
*   114
*   115
*   116
*   117
*   118
*   119
*   120
*   121
*   122
*   123
*   124
*   125
*   126
*   127
*   128
*   129
*   130
*   131
*   132
*   133
*   134
*   135
*   136
*   137
*   138
*   139
*   140
*   141
*   142
*   143
*   144
*   145
*   146
*   147
*   148
*   149
*   150
*   151
*   152
*   153
*   154
*   155
*   156
*   157
*   158
*   159
*   160
*   161
*   162
*   163
*   164
*   165
*   166
*   167
*   168
*   169
*   170
*   171
*   172
*   173
*   174
*   175
*   176
*   177
*   178
*   179
*   180
*   181
*   182
*   183
*   184
*   185
*   186
*   187
*   188
*   189
*   190
*   191
*   192
*   193
*   194
*   195
*   196
*   197
*   198
*   199
*   200
*   201
*   202
*   203
*   204
*   205
*   206
*   207
*   208
*   209
*   210
*   211
*   212
*   213
*   214
*   215
*   216
*   217
*   218
*   219
*   220
*   221
*   222
*   223
*   224
*   225
*   226
*   227
*   228
*   229
*   230
*   231
*   232
*   233
*   234
*   235
*   236
*   237
*   238
*   239
*   240
*   241
*   242
*   243


```

### 3. 极验不同js或不同版本还原方式

请查看极验三破解链接：[极验滑块验证码破解与研究（一）：AST还原混淆JS](https://blog.csdn.net/qq_42857999/article/details/121586389#42_js_314)

三、结语
----

**友情链接：**[**极验第四代滑块验证码破解（二）：滑块缺口识别**](https://blog.csdn.net/qq_42857999/article/details/122364690)

### _本期文章结束啦，如果对您有帮助，记得收藏加关注哦，后期文章会持续更新 ~~~_

![在这里插入图片描述](https://img-blog.csdnimg.cn/56e0933b867e421a9cfd8a02137635a3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5biv5rOq55qE6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)