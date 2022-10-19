> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.motivate.ltd:8087](http://www.motivate.ltd:8087/article/2022/10/19/425.html#Git__1)

【Git】全套操作教程（开发必知）
=================

[发表评论](/article/2022/10/19/425.html#comments) 2 views  

*   [djangoblog](/) 
*   [博客](/category/bo-ke.html) 
*   [转载](/category/zhuan-zai.html) 
*   【Git】全套操作教程（开发必知） 

### 文章目录

*   *   [Git 初始化](#Git__1)
    *   [Git 工作区](#Git__52)
    *   *   [git status](#git_status_53)
        *   [放弃修改已保存的文件](#_67)
        *   [单独存储已保存的文件](#_78)
    *   [Git 暂存区](#Git__107)
    *   *   [git add](#git_add_108)
        *   [添加文件到暂存区](#_122)
        *   [回退暂存区的文件](#_149)
    *   [Git 历史区](#Git__163)
    *   *   [git commit](#git_commit_171)
        *   [提交版本命令详解](#_187)
        *   [回退暂存区和工作区的文件](#_213)
        *   [将分支回退到某个指定版本](#_220)
        *   [将分支回退到倒数几个版本](#_255)
        *   [只回退单个文件到指定版本](#_277)
        *   [恢复被删除的文件或文件夹](#_294)
    *   [Git 添加仓库](#Git__307)
    *   [Git 克隆仓库](#Git__330)
    *   [Git pull 和 push](#Git_pull__push_354)
    *   [Git 分支命令](#Git__385)
    *   [Git 分支冲突](#Git__446)
    *   [Git config 设置](#Git_config__469)

Git 初始化
-------

*   git 安装教程：[https://blog.csdn.net/qq_45677671/article/details/114592426](https://blog.csdn.net/qq_45677671/article/details/114592426)
    
*   git 初始化意义：
    

```
1. 我们必须要把我们电脑中的某一个文件夹授权给 `git`
2. `git` 才能对这个文件夹里面的内容进行各种操作
3. 而 `git init` 就是在进行这个授权的操作
4. `git` 不光管理这一个文件夹，包括所有的子文件夹和子文件都会被管理
5. 只有当一个文件夹被 git 管理以后，我们才可以使用 git 的功能去做版本管理 
```

*   git 初始化流程：git 初始化可以将一个文件夹被 `git` 管理

```
1. 点进文件夹去，再点鼠标右键，点开 `Git Bash Here`
2. 输入 git 初始化的指令：`git init`
3. 然后文件夹内会多一个 `.git` 的文件夹（这个文件夹是一个隐藏文件夹） 
```

*   示图：  
    ![在这里插入图片描述](/media/editor/337a27b0677ed993030c3766a2c06535.png)  
    ![在这里插入图片描述](/media/editor/67587d636e931a62959d314eb486549d.png)  
    ![在这里插入图片描述](/media/editor/c4d936a84df4e4e5c57d98d7e175218c.png)
    
*   管理空文件夹
    

```
1. git 本来是不管理空文件夹的，但是也有一些办法让git能管理一个空文件夹
2. 在空文件夹中新建文件：`.gitkeep`
3. 这个文件没有实际意义，这是为了占位，让空文件夹能被管理的标识，以后要在文件夹中写文件的时候，这个文件可以被删除 
```

*   让某些文件或者文件夹被忽略管理
*   在和`.git`同级的位置，新建文件：`.gitignore`，在这个文件中书写要忽略的内容：

```
1. 直接写文件名，代表要忽略的是哪个文件
2. 写文件夹路径，表示要忽略的是哪个文件夹
3. *.后缀，表示要忽略的是所有后缀为指定后缀的文件 
```

> *   当一个文件夹被 `git` 管理了以后，`git` 就会对当前文件夹进行 **“分区”**
> *   会分为三个区域：工作区、暂存区、历史区
> *   查看当前分区git管理的文件的状态：`git status`

Git 工作区
-------

### git status

**工作区**：我们书写的源码就在工作区里面，文件路径为红色

```
➜ git status
On branch cs/isiliu
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   src/App.vue     # 在工作区的文件路径将为红色

no changes added to commit (use "git add" and/or "git commit -a") 
```

### 放弃修改已保存的文件

> 当我们写代码的时候，文件写出一堆bug，不想要了，没有使用git add添加到暂存区，就可以直接使用以下命令丢弃

*   放弃修改已保存的文件

```
# 丢弃工作区文件的修改：
git checkout -- 文件路径

# 丢弃工作区所有的修改：
git checkout -- . 
```

### 单独存储已保存的文件

> *   当我们代码写到一半的时候，需要切换分支，如果你不commit提交，切换的时候就会把有改动代码，转移到新分支上去，这样就会出问题。
> *   所以就需要不commit提交就能把代码保存下来的方式，那就是 git stash
> *   stash是本地的，不会通过git push命令上传到git server上

*   单独存储已保存的文件

```
# 保存未commit的修改
git stash

# 保存修改加提示信息
git stash save "说明性文字"

# 取出缓存的工作目录
git stash pop

# 拷出缓存的工作目录
git stash apply stash@{0}

# 查看现有的stash
git stash list

# 移除现有的stash
git stash drop

# 移除所有的stash
git stash clear 
```

Git 暂存区
-------

### git add

**暂存区**：把我们想要存储的内容放在暂存区，文件路径为绿色

```
➜ git add . #将工作区的文件添加到暂存区
➜ git status
On branch cs/isiliu
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   src/App.vue     # 在暂存区的文件路径将为绿色 
```

> git add ：用来跟踪新文件并添加到暂存区 或者 将已跟踪的文件添加到暂存区

### 添加文件到暂存区

*   添加文件到暂存区

```
1. 添加一个文件到暂存区：
git add 文件路径
# 例如：
# git add src/pages/data_access/DataAccessListPage.tsx

2. 添加整个文件夹到暂存区（空文件夹无效）：
git add 文件夹路径/
# 例如：
# git add src/pages/data_access/

3. 添加当前文件夹中所有文件和文件夹都到暂存区：
# 可以提交 新建的文件 和 修改的文件，但不处理删除的文件
git add .

# 可以提交 修改的文件 和 删除的文件，但不处理新建的文件
git add -u
# 全写：git add --update

# 提交未跟踪、修改和删除文件
git add -A 
# 全写：git add --all 
```

* * *

### 回退暂存区的文件

*   回退暂存区的文件：

```
1. 将某个文件从暂存区变为源文件：
git reset HEAD -- 文件路径

2. 将整个文件夹从暂存区变为源文件
git reset HEAD -- 文件夹路径

3. 将所有文件从暂存区变为源文件：
git reset HEAD -- . 
```

* * *

Git 历史区
-------

> *   暂存区：只是帮我们暂时存放内容，我们删除了还是会丢的
> *   要想帮我们保存下来，那么还需要把暂存区的内容提交到历史区
> *   `git` 的历史区，就是把我们暂存区里面的文件变成一个历史版本
> *   当一些文件形成一个版本的时候，就会被一直记录下来了
> *   向历史区里面添加内容的时候，必须保证 **暂存区** 有内容
> *   因为历史区就是把暂存区里面的内容收录进去

### git commit

**历史区**：把暂存区里面的内容拿出来形成一个历史版本

```
➜ git commit -m "feat: 完成某页面的开发" # 将暂存区的文件发版
➜ git status            # 再次查看，此时工作区和暂存区就干净了
On branch cs/isiliu
nothing to commit, working tree clean 
```

commit 关键字推荐命名规范:

```
- feat: 新功能
- fix: bug修复
- chore: 日常改动, 如一些优化什么的 
```

### 提交版本命令详解

*   提交文件，形成版本

```
1. 提交文件并添加说明
git commit -m "说明性文字"
# 一定要写一个简单的说明，方便后续查找版本
# 因为当我们的历史版本多了以后，我们自己也记不住哪个版本做了哪些修改

2. 提交跟踪过的文件（可省略`git add`），不添加文字说明
# 能帮你省一步 git add ，但也只是对修改和删除文件有效， 新文件还是要 git add，不然就是 untracked（未跟踪状态）
git commit -a
# 相当于：
# git add .
# git commit 

3. 提交跟踪过的文件（可省略`git add`），并添加文字说明
# 能帮你省一步 git add ，但也只是对修改和删除文件有效， 新文件还是要 git add，不然就是 untracked（未跟踪状态）
git commit -am "xxx"
# 相当于：
# git add .
# git commit -m "xxx"

4. 查看帮助，还有许多参数有其他效果
git commit --help 
```

### 回退暂存区和工作区的文件

```
# 相当于回退到当前的commit的版本，会清空工作区和暂存区的文件
git reset --hard HEAD 
```

### 将分支回退到某个指定版本

> 当我们提交的版本出bug后，如果不影响之前的功能，可能先回退到之前的版本

*   回退版本基础操作：

```
# log 查看所有版本消息，-2表示查看最近2次提交的版本，按 Q 键退出信息查看
git log -2
# commit：这一个版本的版本编号
# Author：作者
# Date：本次版本提交时的记录时间
commit 9f9d45f6a9c0fbbc11f2b1dab161edfa9f2ff68b
Author: xingliu
Date:   Fri Jan 7 14:10:13 2022 +0800

    feat: 此次为提交时填写的说明性文字

commit 4917f349f3b4bb802ebf4c91d96d6b4cbbd5db34
Author: xingliu
Date:   Fri Jan 7 14:08:31 2022 +0800

    feat: 此次为提交时填写的说明性文字
# 根据查看到的版本commit号，选择回退到相应的版本
1. 回退到第一次提交的版本
git reset --hard 4917f349f3b4bb802ebf4c91d96d6b4cbbd5db34

2. 回退到第二次提交的版本
git reset --hard 9f9d45f6a9c0fbbc11f2b1dab161edfa9f2ff68b

3. 其他命令

# 查看 git reset 帮助
git reset -h 
```

### 将分支回退到倒数几个版本

*   回退版本快捷操作：

```
# 一般使用hard，有其他需要可以使用mixed或soft
# hard：删除工作空间改动代码，撤销commit，撤销git add .
# 示例：将版本库回退3个版本
git reset --hard HEAD~3     # 数字大小代表的是上几个版本
git reset --hard HEAD^^^    # ^号数量代表的是上几个版本

# mixed：不删除工作空间改动代码，撤销commit，并且撤销git add . 操作
# 示例：将版本库软回退1个版本
git reset --mixed HEAD^     # ^号数量代表的是上几个版本
git reset HEAD^             # ^号数量代表的是上几个版本
git reset HEAD~1            # 数字大小代表的是上几个版本

# soft：不删除工作空间改动代码，撤销commit，不撤销git add . 
# 示例：将版本库软回退2个版本
git reset --soft HEAD~2 # 数字大小代表的是上几个版本
git reset --soft HEAD^^ # ^号数量代表的是上几个版本 
```

### 只回退单个文件到指定版本

*   还原某个特定的文件到之前的版本

```
第一步： git log 文件路径
在命令行中输入 git log src/main/main.c 得到该文件的commit 历史。 

第二步： 复制需要回退版本的hash
在此假设我们回退到 d98a0f565804ba639ba46d6e4295d4f787ff2949 ,则复制该序列即可

第三步：git checkout 文件对应的版本 文件路径。
如：在此即为命令行中输入 git checkout d98a0f565804ba639ba46d6e4295d4f787ff2949 src/main/main.c

第四步： 重新提交下版本 git commit -m 
如： git commit -m "revert to previous version" 
```

### 恢复被删除的文件或文件夹

```
# 查看被删除的文件路径
1. git status

# 找回被删除的文件
2. git reset HEAD 被删除的文件或文件夹

# 恢复被删除的文件
3. git checkout 被删除的文件或文件夹 
```

Git 添加仓库
--------

*   我们先要给 git 添加一个上传的地址
*   我们还是来到我们的项目文件夹
*   使用 `git remote add origin` 仓库地址 来添加

```
# 在项目文件夹下打开 git base
# 添加仓库地址
git remote add origin https://github.com/guoxiang910223/ceshi1913.git 
```

```
- remote：远程的意思
- add：添加的意思
- origin：是一个变量名（就是指代后面一长串的地址） 
```

*   `git remote -v`：查看是否关联上了远程仓库

```
git remote -v
orgin github.com/guoxiang910223/ceshi1913.git(fetch)
orgin github.com/guoxiang910223/ceshi1913.git(push) 
```

Git 克隆仓库
--------

> git 克隆是指把远程仓库里面的内容克隆一份到本地  
> 。可以克隆别人的 公开 的仓库，也可以克隆自己的仓库。克隆别人的仓库，我们只能拿下来用，修改后不能从新上传。克隆自己的仓库，我们修改后还可以再次上传更新

*   我们先找到一个别人的仓库，或者自己的仓库（这里以 jQuery 的仓库为例）  
    ![在这里插入图片描述](/media/editor/8c4f65bcd3e20d94a3c208c81f3e3062.png)
*   复制好地址以后，选择一个我们要存放内容的文件夹（我这里以桌面为例）
*   直接在想存放内容的位置打开 `git base`
*   输入克隆指令 `git clone 仓库地址`
*   例如：`git clone https://github.com/jquery/jquery.git`
*   你就会发现你的文件夹里面多了一个叫做 jquery 的文件夹
*   里面就是人家仓库的所有内容
*   长期储存密码：

```
git config credential.helper store 
```

*   查看配置：

```
git config --list 
```

Git pull 和 push
---------------

*   上传要保证 **历史区** 里面有内容，会把所有的内容上传到远端
    
*   上传指令：
    

```
# 指定远程仓库名和分支名（第一次）
git push -u origin master
# origin 指定推送的地址
# master 是上传到远程的 master 分支

# 不指定远程仓库名和分支名（第二次）
git push 
```

*   `-u` 是为了记录一下用户名和密码，下次上传的时候就不需要再写了
    
*   第一次上传要指定远程仓库名和分支名
    
*   第二次上传的时候，因为有刚才的记录，就不需要再 写 origin 和 master 了， 会默认传递到 origin 这个地址的 master 分支上
    
*   如果你要传递到别的分支上的时候，要再次指定远程仓库名和分支名
    
*   这个时候本地的文件夹就真的可以删除了， 因为远程有一份我们的内容，本地的删除了，可以直接把远程的拉回来就行
    
*   不管是你克隆下来的仓库还是别的方式弄得本地仓库，当人家的代码更新以后，你想获得最新的代码。我们不需要从新克隆。只要拉取一次代码就可以了
    
*   下拉指令：
    

```
# 将本地代码和远程代码同步：(在本地仓库使用命令)
git pull 
```

*   这样一来，你本地的仓库就可远程的仓库同步了

Git 分支命令
--------

> 一个大项目，会分很多人开发，每个人一个功能，这时候，每个功能作为一个分支，主分支只有目录结构。当所有人将自己负责的功能开发完成的时候，再将所有分支合并到主分支上，形成一个完整的项目。

*   常用分支命令

```
# 查看本地分支：
git branch

# 查看远程分支：
git branch -r

# 更新远程分支：
git fetch

# 创建分支：
git branch 分支名

### 切换分支：（常用）
git checkout 分支名

### 切换到上个分支：（常用）
git checkout -

### 创建并切换到这个分支：（常用）
git checkout -b 新分支名

### 合并分支：（常用）
git merge 被合并的分支 # 要先跳转到要合并其他分支的分支

# 终止合并分支：
git merge --abort

# 删除分支：(不能自己删自己)
git branch -D 要删除的分支

# 本地分支重命名：
git branch -m oldName  newName

# 删除远程的旧分支：
git push --delete origin oldName 
```

*   常见分支命名

1.  **master**：主分支，永远只存储一个可以稳定运行的版本，不能再这个分支上直接开发
2.  **develop**： 主要开发分支，主要用于所用功能开发的代码合并，记录一个个的完整版本
    *   包含测试版本和稳定版本
    *   不要再这个分支上进行开发
3.  **feature-xxx**：功能开发分支，从 `develop` 创建的分支
    *   主要用作某一个功能的开发
    *   以自己功能来命名就行，例如 `feature-login / feature-list`
    *   开发完毕后合并到 `develop` 分支上
4.  **feature-xxx-fix**: 某一分支出现 `bug` 以后，在当前分支下开启一个`fix`分支
    *   解决完 `bug` 以后，合并到当前功能分支上
    *   如果是功能分支已经合并之后发现 `bug` 可以直接在 `develop` 上开启分支
    *   修复完成之后合并到 `develop` 分支上
5.  **hotfix-xxx**： 用于紧急 `bug` 修复
    *   直接在`master`分支上开启
    *   修复完成之后合并回 `master`

Git 分支冲突
--------

> 两个人同时操作同一个分支，提交的时候会有先后顺序，先提交的人正常提交了，后一个人提交的时候会产生冲突。因为git规定，每次提交必须是在原来的版本基础上，但是第二个人在提交的时候，在远程已经有了第二个版本，所以第二个人相当于从第一个版本向第三个版本提交。如下图：

![在这里插入图片描述](/media/editor/9c6b534188c8644b888aa7e668ade219.png)  
![在这里插入图片描述](/media/editor/04847bb1af324720a007ef253af89e12.png)

*   冲突解决：[https://blog.csdn.net/qq_45677671/article/details/122574511](https://blog.csdn.net/qq_45677671/article/details/122574511)

记得养成一个良好git发布流程的习惯

```
# 分支合并发布流程：
git add .         # 将所有新增、修改或删除的文件添加到暂存区
git commit -m "版本发布" # 将暂存区的文件发版
git status          # 查看是否还有文件没有发布上去
git checkout test # 切换到要合并的分支
git pull            # 在test 分支上拉取最新代码，避免冲突
git merge dev       # 在test 分支上合并 dev 分支上的代码
git push            # 上传test分支代码 
```

*   关注我不迷路，我替你们把坑都踩平了

Git config 设置
-------------

> `git config`命令用于获取并设置存储库或全局选项，这些变量可以控制Git的外观和操作的各个方面

```
# 查看git配置信息
git config --list

# 查看git用户名
git config user.name

# 查看邮箱配置
git config user.email

# 全局配置用户名
git config --global user.name "nameVal"

# 全局配置邮箱
git config --global user.email "eamil@qq.com"

# 删除全局配置用户名和邮箱
git config --global unset user.name "用户名"
git config --global unset user.email "注册邮箱" 
```

> 对于git来说，配置文件的权重是`「仓库 > 全局 > 系统」`，即 `「local > global > system」`

*   添加配置项：`--add`
*   格式：`git config [-local | -global | -system] add section.key value`（默认是添加在`local`配置中）

```
git config --add sit.name Jack 
```

*   删除配置项：`--unset`
*   格式：`git config [-local | -global | -system] -unset section.key`

```
git config --local --unset user.name 
```

*   配置快捷键：`alias`

```
# 使用 git st 代替 git status
git config --global alias.st status

# 用 last 表示查看最后一次提交
git config --global alias.last 'log -1 HEAD' 
```

*   Git设置当前分支为默认push分支

```
git config --global push.default "current" 
```

转载地址：[【Git】全套操作教程（开发必知）](https://blog.csdn.net/qq_45677671/article/details/114594940)

版权声明：本文内容由互联网用户自发贡献，该文观点与技术仅代表作者本人。本站仅提供信息存储空间服务，不拥有所有权，不承担相关法律责任

本条目发布于 [2022-10-19](/article/2022/10/19/425.html "2022-10-19") 。属于[转载](/category/zhuan-zai.html) 分类， 作者是 [wensong](/author/wensong.html "查看所有由wensong发布的文章") 。