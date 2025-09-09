> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/QxsCltASy5j8iHVP_BTBPg)

我们先用IDA打开WeChatWin.dll 我的最新版本 [3.9.12.51]

多开请看之前文章：微信逆向之自己动手去除微信多开限制，小白也能看懂

https://www.52pojie.cn/thread-1951224-1-1.html

大佬文章：微信逆向：防撤回消息分析

https://www.52pojie.cn/thread-1947110-1-1.html

我们根据大佬的文章日志内容直接搜索字符串：SyncMgr::ProcessNewXMLMsg

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHO0cP4gmv9Zib22ibG5ic6amPfeHXovxficCtiamT17xXK8ib2j8ia9RwQqrIA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

找到撤回的处理逻辑地方 我们直接用deepseek分析每个函数的用处

使用ai分析 请安装插件 https://github.com/WPeace-HcH/WPeChatGPT 安装方法请看仓库说明

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmH1mE94LrjdtZQQPV3pAPLbPlh16EEwDdoYoRxGOpT3hVJOr0hRj5jIw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

sub_182313480方法是撤回逻辑方法

  

sub_182313D20 是处理安全消息（Secure Message）的同步逻辑，包括解析消息、更新数据库或内存中的消息列表、处理错误情况，并确保线程安全和资源管理

  

sub_182314430 **处理上传完成后的同步消息**，涉及对消息的解析、数据库操作、状态更新以及错误日志记录。核心逻辑包括：

  

// - 验证输入消息的有效性

// - 更新同步管理器的内部状态

// - 处理数据库中的消息记录

// - 错误处理（包括内存释放和日志记录）

sub_18232BB30 **同步管理器（SyncMgr）的消息处理分发器**，用于根据传入的XML消息类型（`a1`参数）调用不同的处理逻辑，完成对各类同步消息（如配置更新、登录同步、实时消息、联系人变更等）的解析、验证和业务处理。最终返回处理结果状态

```
`switch ( (_DWORD)v9 )` `{` `case 4:` `return sub_182313480(a2, a3, a4);` `case 0x21:` `sub_182313D20(a2, a3, a4);` `break;` `case 0x24:` `sub_182314430(a2, a3, a4);` `return 0;` `default:` `if ( sub_18232BB30(v9, a2, a3) )` `return 1;` `break;` `}` `return 0;` `}`
```

我们直接看这个sub_182313480方法 这个是撤回走的逻辑

```
`// ### 预期目的``// 该函数用于处理消息撤回的同步逻辑，核心目标是响应来自其他设备的撤回消息请求，验证消息合法性，更新本地消息状态，并处理可能出现的错误场景（如重复同步、类型不匹配等）。``//``// ---``//``// ### 参数分析```// 1. **`a1`**（`__int64`）:`` ``//    指向 `SyncMgr` 类实例的指针，可能包含同步管理器的上下文信息（如当前会话ID、设备状态等）。通过 `a1 + 72` 访问的字段可能是管理器内部的状态标识。```//```// 2. **`a2`**（`__int64`）:`` ``//    消息元数据指针，可能包含消息头信息（如消息类型、发送者等）。通过 `a2 + 136` 访问的字段可能是消息类型标识符，用于校验是否为合法的撤回操作。```//```// 3. **`a3`**（`_QWORD *`）:`` ``//    指向消息数据结构的指针，包含待处理的撤回消息列表及其状态（如消息ID、时间戳）。通过 `a3[2]` 和 `a3[3]` 的操作推测其为动态数组或链表结构，用于存储批量撤回消息。```//``// ---``//``// ### 详细功能``// 1. **类型校验**：` ``//    检查 `a2` 中的消息类型是否为合法的撤回类型（`sub_181C22F30`），若非法则记录错误日志（"type not match!!!"）并终止流程。```//``// 2. **消息查找**：` ``//    通过 `sub_18232D780` 在消息列表（`*a3`）中定位目标消息的索引（`v8`），并遍历内部数据结构（可能是红黑树或链表）确认消息是否存在。```//``// 3. **重复同步处理**：` ``//    若发现消息已由当前设备同步（`repeat sync msg send from this device`），则跳过更新，仅记录日志。```//``// 4. **状态更新**：` ``//    - 调用 `sub_181C20430` 或 `sub_181C20F40` 更新消息为“已撤回”状态。````//    - 处理特殊场景（如消息类型为 `10000` 或状态码 `4`），触发额外清理逻辑（`sub_182502CF0`）。```//``// 5. **错误处理与日志**：` ``//    使用 `sub_182627590` 记录不同级别的日志（如错误、调试信息），包含代码文件路径（`SyncMgr.cpp`、`ChatRevokeMgr.cpp`）和行号，便于问题追踪。```//``// 6. **资源管理**：` ``//    函数末尾释放临时缓冲区（`v39`、`v41`）和重置指针，避免内存泄漏。``
```

我们双击sub_182313480方法 进去方法内部找日志打印的信息 找字符串 add or update rv msg : %d exist %d

这个字符串从大佬文章中可以看到日志

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHFVURib1gPcibIiaSoTzjWt2H9XIibrhpiaLO7VZlKMyZSwZMJ6omY07wK6A/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

这个日志下面的方法sub_181C20430 我用ai分析了一下就是去把消息转换成系统的撤回消息

所以我们直接nop掉这个方法就可以实现消息防撤回

开干 先右击鼠标把我们伪代码的行信息和汇编窗口的行同步 看图 选一下

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHrnLu7uAyOYvVdqgppVtU0s0C1UUTjaOsrI4VEv4qExbbY5ak9t7S3A/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

先把鼠标光标放到这个方法上 然后我们点击ida view窗口 nop这个方法调用

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHKsfQFDP9vZ7SF61iaZEo7p6XPnm3LZFicRibYR4veTlOkBbM1QmwRTq4A/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

然后我们修补dll文件 看图操作

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHMc4OcC9sr5g596n1Zf6YDr83x2rPSNgWmw83MCoskhlGTOpN33FialQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHOnh8waJDU21rR2zVYPLE9Zq4Cp4X2ibv3lIiapjkkknpgf12F0HUWcCw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=6)

最后登陆测试 撤回 消息不会消失就成功了

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHElmyOK6KFFvGI6c4v2tZh3ItGmSnNPzoyE6FwFYNiazmKpSc2Ld9pwQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

然后我想实现带提示撤回消息的 详细分析了sub_182313480这个方法 用ai搞出来了python伪代码 里面的每个方法都用ai分析了一下

发现这个实现就是把要撤回的原消息 转换成系统撤回消息进行更新 实在是不知道怎么改 才能实现 原消息不消失 并且出现系统撤回消息提示

有咩有大佬指点迷津

对您有帮助请给个免费的评分谢谢

伪代码：

```
`import time``from typing import Optional, Any``class ChatRevokeMgr:` `def __init__(self):` `self.global_state = self.get_global_state()` `def get_global_state(self) -> Any:` `# 模拟获取全局状态的函数` `return {"allow_operation": True}` `def ProcessAndSaveRevokedMessage(self, a1: Any, a2: Any) -> Optional[int]:` `# 模拟获取全局状态的函数` `result = self.sub_181B5CA40()` `if not result.get("allow_operation"):` `return result` `v5 = a1.get("type")` `v84 = False` `# 10000 系统消息处理` `if v5 and a2.get("message_type") == 10000:` `# 输出日志` `self.sub_182627590(2, "ChatRevokeMgr::SaveRevokeSysMsg", "original msg is sys msg")` `return None` `if not a2.get("timestamp"):` `a2["timestamp"] = a1.get("timestamp")` `a2["status"] &= 0x80` `if v5:` `# 处理群组消息撤销` `self.sub_181C21660(a2)` `else:` `a2["sender"] = a1.get("sender")` `a2["status"] &= 8` `v6 = a1.get("content")` `if not v6:` `v6 = "default_content"` `# 动态管理一个宽字符串缓冲区，根据输入的字符串内容动态调整内存分配，并在输入为空时释放所有关联资源。其核心功能是安全地替换或清空目标结构体中存储的宽字符串数据` `self.sub_18262DED0(a2, v6, a1.get("content_length"))` `a2["status"] &= 0x100` `if a2.get("message_id"): # 有的是本地消息？` `v7 = a2.get("message_type")` `if v7 == 1:` `v84 = True` `v8 = 0` `elif v7 == 49: # 应该是测回本地消息` `# 在多线程环境下**安全地初始化并获取一个全局变量**，确保线程安全的单次初始化（类似双重检查锁定模式），同时注册退出时的清理函数` `v9 = self.sub_181B601E0()` `# 输入结构体中的不同状态标志和缓存机制，返回一个特定的计算结果或缓存值。它通过条件分支处理不同的计算逻辑，并在首次计算后缓存结果以提升性能` `v10 = self.sub_1825040B0(a2)` `# 取消指定文件的上传任务**。通过哈希表查找目标消息记录后，若存在有效记录则触发取消逻辑，否则返回错误信息。` `self.sub_1820E2B90(v9, v10)` `# 模拟获取另一个存储句柄` `v11 = self.sub_181C24E20()` `# 输入结构体中的不同状态标志和缓存机制，返回一个特定的计算结果或缓存值。它通过条件分支处理不同的计算逻辑，并在首次计算后缓存结果以提升性能` `v12 = self.sub_1825040B0(a2)` `# 释放与特定本地ID关联的文件资源**，包括关闭文件句柄、清理内存和移除管理结构中的对应节点，属于资源回收操作` `self.sub_1822B6260(v11, v12)` `v8 = 0` `else:` `if not a2.get("is_processed"):` ``# 该函数用于根据第二个参数（`a2`）提供的配置信息，动态更新第一个参数（`a1`）指向的结构体中的多个字段。具体逻辑是通过一系列条件检查（标志位或配置项），将不同数据源的值复制到结构体的特定偏移位置`` `self.sub_1825026B0(a2, a2)` `a2["is_processed"] = True` `# 将输入的宽字符串（wchar_t类型）**深拷贝**到目标结构体的动态分配内存中，同时维护目标结构体的内存指针、长度等元数据，并在输入为空或无效时正确初始化目标结构体的状态` `self.sub_18262D800(a1, a2)` `v8 = 2` `v13 = True` `if v13 and a2.get("message_type") == 49 and a2.get("sub_type") == 2:` `# 在多线程环境下**安全地初始化并获取一个全局变量**，确保线程安全的单次初始化（类似双重检查锁定模式），同时注册退出时的清理函数` `v14 = self.sub_181B601E0()` `#根据当前线程状态执行不同的资源管理操作``，可能涉及**异步任务调度**和**线程安全的状态同步**。通过检查当前线程ID决定直接执行操作（主线程）或将任务封装后加入队列（其他线程），最终确保资源释放和状态更新` `self.sub_1820F0A90(v14, a2)` `# **处理聊天消息的删除操作 删除文件**，特别是针对不同类型消息（如文本、媒体、附件等）的删除逻辑。它涉及资源释放、日志记录、条件分支处理，并可能调用底层存储接口完成实际删除` `self.sub_181D8C330(a2)` `# 在多线程环境下**安全地初始化并获取一个全局变量**，确保线程安全的单次初始化（类似双重检查锁定模式），同时注册退出时的清理函数` `v15 = self.sub_181B601E0()` `# 输入结构体中的不同状态标志和缓存机制，返回一个特定的计算结果或缓存值。它通过条件分支处理不同的计算逻辑，并在首次计算后缓存结果以提升性能` `v16 = self.sub_1825040B0(a2)` `v86 = v16` `v17 = v15.get("storage_handle")` `v18 = v17.get("next")` `v19 = v17` `while not v18.get("is_processed"):` `if v18.get("message_id") >= v16:` `v19 = v18` `v18 = v18.get("next")` `else:` `v18 = v18.get("prev")` `if v19 != v17:` ``# **安全删除指定类型的文件**。其核心逻辑是通过校验文件路径合法性、匹配预设的文件类型/目录规则（如临时文件、备份文件等），并最终调用系统API `DeleteFileW` 删除文件。`` `self.sub_1825F9FB0(v19, 0, v17)` ``# 在某种树状数据结构（可能是**平衡树**或**红黑树**）中**删除满足特定条件的节点**，并返回被删除的节点数量。其核心操作是遍历树结构，根据参数 `a2` 提供的值对节点进行范围筛选或条件匹配，最终释放符合条件的节点内存`` `self.sub_181DBA390(v17, v86)` `v21 = a1.get("sender_type")` `# 将输入的宽字符串（wchar_t类型）**深拷贝**到目标结构体的动态分配内存中，同时维护目标结构体的内存指针、长度等元数据，并在输入为空或无效时正确初始化目标结构体的状态` `v22 = self.sub_18262D800(a1, a2)` `# 该函数用于验证某个结构体/对象内部的状态字段（通过偏移量访问）是否满足一系列特定的条件组合，可能用于判断对象是否处于有效状态或满足某种业务逻辑的合法性检查` `v23 = self.sub_182502BD0(a2)` `v24 = bool(a2.get("message_id"))` `# **构建微信消息撤回后的重新编辑内容**，生成符合特定协议格式的XML结构，并处理内存分配与数据拼接，最终返回格式化后的消息体。` `self.sub_181C22A90(a2, v24, v23, v22, v21)` `v25 = v8 | 4` `if a2.get("message_id"):` `# 根据输入的参数配置某个对象（可能涉及文件路径或资源处理），执行初始化操作后更新相关状态，最后返回处理结果。核心逻辑包含首次运行时的初始化保护、路径参数的合法性校验、参数存储以及状态更新` `self.sub_181BF8B80(a2, a2)` `v26 = a2.get("content")` `if not v26:` `v26 = "default_content"` `# 动态管理一个宽字符串缓冲区，根据输入的字符串内容动态调整内存分配，并在输入为空时释放所有关联资源。其核心功能是安全地替换或清空目标结构体中存储的宽字符串数据` `self.sub_18262DED0(a2, v26, len(v26))` `a2["message_type"] = 10000` `a2["status"] = 0` `v27 = a2.get("sub_type")` `if not v27:` `# 对输入数据进行**特定字符集验证**，并在满足条件时更新目标结构体中的字段值。核心逻辑是检查输入字符是否符合预设的位掩码范围（如数字、空格或特定符号），若验证通过则进行数据格式转换和存储` `self.sub_182502CF0(a2)` `v27 = a2.get("sub_type")` `if v27 != 57:` `if not v27:` `# 对输入数据进行**特定字符集验证**，并在满足条件时更新目标结构体中的字段值。核心逻辑是检查输入字符是否符合预设的位掩码范围（如数字、空格或特定符号），若验证通过则进行数据格式转换和存储` `self.sub_182502CF0(a2)` `v27 = a2.get("sub_type")` `if v27 != 1:` `a2["sub_type"] = 0` `if not a2.get("timestamp"):` `# 模拟获取当前时间` `self.sub_181B8E930()` `# 模拟获取当前时间戳` `v28 = self.sub_182651830()` `if self.global_state.get("time_offset"):` `v28 -= self.global_state.get("time_offset")` `a2["timestamp"] = v28` `a2["status"] &= 128` `if a2.get("sequence_id") <= 0:` `# 模拟获取当前时间` `self.sub_181B8E930()` `if a1.get("timestamp"):` `v29 = a1.get("timestamp")` `else:` `# 模拟获取当前时间戳` `v29 = self.sub_182651830()` `if self.global_state.get("time_offset"):` `v29 -= self.global_state.get("time_offset")` `a2["sequence_id"] = 1000 * v29` `# 输出日志` `self.sub_182627590(2, "ChatRevokeMgr::SaveRevokeSysMsg", "Regen rv msg sortseq id")` `a2["sub_type"] = 2` `a2["status"] &= 64` `# 模拟获取当前时间` `self.sub_181B8E930()` `if v5:` `#群消息处理` `# 据输入参数中的特定字符串特征（如前缀/后缀格式）判断账号类型（可能是微信公众号、企业号、应用号等），并返回对应的配置或处理句柄` `v32 = self.sub_18213CC90(a2)` `if v32:` `v32.process(a2, 1)` `v86 = "new_content"` ``# 初始化一个`MsgChangeEvent`对象，为其设置虚函数表、分配内部数据结构，并完成成员变量的基础初始化，最终构造出一个完整可用的消息事件对象`` `v33 = self.sub_181C20DA0(v86)` `v34 = v33.get("next")` `if v34 == v33.get("prev"):` ``# 该函数用于在动态数组（类似`std::vector`的结构）的指定位置插入元素，并在必要时扩容`` `self.sub_181BEC420(v33, v34, a2)` `else:` ``# **对象拷贝构造函数**（或深拷贝函数），用于将一个 `ChatMsg` 类实例（`a2`）的数据完整复制到另一个实例（`a1`）中，同时初始化虚函数表（vftable）和可能存在的内部资源（如字符串、嵌套对象等），确保目标对象与源对象完全独立`` `self.sub_181B7CD20(v34, a2)` `v33["next"] += 1104` `else:` `# **在特定条件验证通过后，通过间接调用某个对象的方法来执行安全操作**` `self.sub_182147800(a2, a2)` `v86 = "new_content"` ``# 该函数用于初始化一个`MsgChangeEvent`对象，为其设置虚函数表、分配内部数据结构，并完成成员变量的基础初始化，最终构造出一个完整可用的消息事件对象`` `v33 = self.sub_181C20DA0(v86)` `v34 = v33.get("next")` `if v34 == v33.get("prev"):` ``# 该函数用于在动态数组（类似`std::vector`的结构）的指定位置插入元素，并在必要时扩容`` `self.sub_181BEC420(v33, v34, a2)` `else:` ``# **对象拷贝构造函数**（或深拷贝函数），用于将一个 `ChatMsg` 类实例（`a2`）的数据完整复制到另一个实例（`a1`）中，同时初始化虚函数表（vftable）和可能存在的内部资源（如字符串、嵌套对象等），确保目标对象与源对象完全独立`` `self.sub_181B7CD20(v34, a2)` `v33["next"] += 1104` `# 模拟获取日志句柄` `v35 = self.sub_1823865C0()` `# 模拟日志记录操作` `self.sub_1823878F0(v35, {"log_id": 965, "message": a2})` `if v84: # 消息类型 = 1 执行` `# 线程安全的单例初始化函数，用于在多线程环境下确保全局/静态数据仅被初始化一次，并在程序退出时注册清理操作` `v36 = self.sub_181C1CB20()` ``# 从源对象 `a1` 中提取资源信息（数据和引用计数管理的指针），将其安全复制到目标结构体 `a2` 中，并通过原子操作管理引用计数，确保资源在传递过程中保持有效，避免提前释放`` `v37 = self.sub_181C280C0(v36, "default_content")` `v86 = "new_content"` ``# **对象拷贝构造函数**（或深拷贝函数），用于将一个 `ChatMsg` 类实例（`a2`）的数据完整复制到另一个实例（`a1`）中，同时初始化虚函数表（vftable）和可能存在的内部资源（如字符串、嵌套对象等），确保目标对象与源对象完全独立`` `self.sub_181B7CD20(v86, a2)` `v82 = "new_content"` `v38 = "new_content"` `v49 = v38` ``# **对象拷贝构造函数**（或深拷贝函数），用于将一个 `ChatMsg` 类实例（`a2`）的数据完整复制到另一个实例（`a1`）中，同时初始化虚函数表（vftable）和可能存在的内部资源（如字符串、嵌套对象等），确保目标对象与源对象完全独立`` `self.sub_181B7CD20(v38, a2)` `v82 = v38` `# 对象资源清理函数，用于释放动态分配的内存、重置指针和计数器，并执行对象的析构操作，确保内存安全` `self.sub_181B5CAC0(v86)` ``# **处理事件参数的传递和事件触发逻辑**，根据不同的条件（`a3`参数）创建并初始化事件参数对象，通过虚函数调用分发到对应的事件处理模块，最终执行资源清理操作`` `self.sub_18265F3D0(v86, v37, 0)` `v39 = a2.get("sub_type")` `if not v39:` `# 对输入数据进行**特定字符集验证**，并在满足条件时更新目标结构体中的字段值。核心逻辑是检查输入字符是否符合预设的位掩码范围（如数字、空格或特定符号），若验证通过则进行数据格式转换和存储` `self.sub_182502CF0(a2)` `v39 = a2.get("sub_type")` `# 输出日志` `self.sub_182627590(2, "ChatRevokeMgr::SaveRevokeSysMsg", "has msg")` `# 模拟获取全局状态的函数` `v41 = self.sub_181B5CA40()` ``# **多字节字符串（UTF-8编码）**转换为**宽字符字符串（UTF-16编码）**，并将结果存储到目标结构体（`a1`）中。如果输入无效或转换失败，函数会清空目标结构体的内容并释放内存`` `self.sub_18262DB20(a2, v41)` `if not a2.get("is_processed"):` ``# 该函数用于根据第二个参数（`a2`）提供的配置信息，动态更新第一个参数（`a1`）指向的结构体中的多个字段。具体逻辑是通过一系列条件检查（标志位或配置项），将不同数据源的值复制到结构体的特定偏移位置`` `self.sub_1825026B0(a2, a2)` `a2["is_processed"] = True` `# 将输入的宽字符串（wchar_t类型）**深拷贝**到目标结构体的动态分配内存中，同时维护目标结构体的内存指针、长度等元数据，并在输入为空或无效时正确初始化目标结构体的状态` `self.sub_18262D800(a1, a2)` `v42 = v25 | 8` `v43 = a2.get("content")` `if not v43:` `v43 = "default_content"` `v44 = a2.get("content")` `if v44:` `if v43:` `if len(v44) > 0:` `v45 = v44.find(v43)` `if v45 >= 0:` `v85 = "new_content"` `# 将输入的宽字符串（wchar_t类型）**深拷贝**到目标结构体的动态分配内存中，同时维护目标结构体的内存指针、长度等元数据，并在输入为空或无效时正确初始化目标结构体的状态` `self.sub_18262D800(a1, a2)` `v46 = v42 | 0x11` `v86 = v46` `# 一个C++类构造函数/初始化函数，主要用于：` ``# // 1. 初始化一个`EventParamMMString`类对象（继承自`InstanceCounter<EventParam,100>`模板类）`` `v47 = self.sub_182389C70(v85, a2)` `if v46 & 1:` `if a2.get("content"):` `a2["content"] = None` `a2["content_length"] = 0` `# **初始化一个全局的线程安全数据结构**（可能用于资源管理或任务调度），通过单例模式确保全局唯一性。若该结构已存在则直接返回，否则执行复杂的初始化流程，包含内存分配、临界区（Critical Section）创建以及多个内部链表节点的初始化` `v48 = self.sub_1823865C0()` `# 模拟日志记录操作` `self.sub_1823878F0(v48, {"log_id": 858, "message": a2})` `if v44:` `a2["content"] = None` `if a2.get("content"):` `a2["content"] = None` `if a2.get("content"):` `a2["content"] = None` `return None``# 示例用法``a1 = {"type": 1, "timestamp": 1633024800, "sender": "user1", "content": "Hello, World!", "content_length": 13}``a2 = {"message_type": 1, "timestamp": 0, "status": 0, "message_id": 12345, "sub_type": 0, "sequence_id": 0, "is_processed": False}``mgr = ChatRevokeMgr()``mgr.ProcessAndSaveRevokedMessage(a1, a2)`
```

  

  

**·** **今 日 推 荐** **·**

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/WJRHqUiaud0p59iaiaFMzWbpqCtFkNHqwmHOaTlVncsyCic5iaBibcrQcZR7B7UOQMQxjgCibxvEjFQoQU43ibiaWkpp9og/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8)

本文内容来自网络，如有侵权请联系删除