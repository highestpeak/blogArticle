---
title: Git wiki 详细版
date: 2020-10-17 11:04:00
status: PUBLISHED
comments: true
description: 这是一篇 wiki ,主要介绍一些常见的 Git 命令。这个 wiki 是详细版，还伴随着对一些命令的说明等，大量的命令请查看清单版。注意：你只应该把 wiki 来作为一个速查表。
tags: 
- Git
- 版本控制
- wiki
categories: 
- 软件开发环境与工具
---

注意：

- 本文所有带有 `<>` 的地方，均为参数值，需要根据具体场景填写，填写时不需要带上括号
- 简单命令、文件状态、撤销操作 属于 Git 基础，应当完全掌握
- 本文仅说明那些是难点，有要点的命令，对于例如简单的显示信息类的命令不在这里显示。那些简单易懂的命令请查看清单版 wiki

# 配置

**配置**存储在三个**位置**：

- 系统级配置文件：`C:\ProgramData\Git\config` 文件 或 `/etc/gitconfig` 文件
- 用户级配置文件：`C:\Users\$USER\.gitconfig` 文件 或 `~/.gitconfig` 文件 或 `~/.config/git/config` 文件
- 仓库目录下配置文件：`.git/config`

因为配置有不同的存储的地方，所以配置命令的作用域和生效位置也需要注意：

- `--global` 指定整个系统的配置
- 可以不使用`--global` ，从而修改的是项目目录下的用户名和邮箱

关于配置的常见命令：

- 列出所有配置→不同文件可能包含同一个配置→查看配置是由那个文件指定的
- 常见的配置命令（设置用户名、邮箱、文本编辑器等）
- （更多请看 wiki 版）

可以通过配置来设置一些常见命令或命令组合的**别名**：

- Git 只是简单地将别名替换为对应的命令。 然而，你可能想要执行外部命令，而不是一个 Git 子命令。 如果是那样的话，可以在命令前面加入 `!` 符号。

  如果你自己要写一些与 Git 仓库协作的工具的话，那会很有用。 

# 历史

按时间先后顺序列出**所有提交**：

- `git log` 不涉及每个提交的修改信息

log **选项**（在这里只列出几个常用的）：

# 标签

- 人们会使用标签功能来标记发布结点（ `v1.0` 、 `v2.0` 等等）
- 如果标签不在您的存储库中，您将无法检出标签，因此首先，您必须`fetch`将标签移至本地存储库。

**创建标签**：

- 轻量标签和附注标签：
  
- 轻量标签只需要提供标签的名字，附注标签可以提供额外信息，如：名称，电子邮件，日期，评论和签名
  

**删除**标签：

- 不会删除远程仓库的标签

**推送**标签：

- `git push` 命令**不会自动传送标签到远程仓库**服务器上，必须显式地推送标签到服务器上

- 推送标签并**不会区分轻量标签和附注标签**

- 使用`refs/tags`而不是仅指定`<tagname>`：

  使用`refs/tags`是因为有时标签可以与分支具有相同的名称，并且简单的`git push`会推送分支而不是标签


# 仓库

`fetch`、`pull` 和 `clone`：

- `git fetch` 命令只会将数据下载到你的本地仓库——它并**不会自动合并**或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作
- 若想下载远程仓库，本地无需**`git init`**,直接 `git clone url` 。这样就会在本地生成一个目录，该目录与远程仓库同名
- `git pull = git fetch + git merge`
- `git fetch`不会进行合并，执行后需要手动执行`git merge`合并，而`git pull`拉取远程分之后直接与本地分支进行合并

# 文件操作

![](https://gitee.com/matrixPeak/BlogArticle/raw/image/Git-项目环节和文件状态.png)

## 查看状态

主要说两个命令： status 和 diff

- diff 相对于 status 可以说明每个文件具体修改了什么地方

> git status

检查所有文件的状态：

- 未跟踪状态的新文件会被列出来
- 更改会被列出
- 会显示当前分支

**状态简览**：

``` bash
$ git status -s
# 或者
$ git status --short
```

- `git status` 输出详细，而加上 `-s` 选项后输出简洁

- 输出说明：

  ``` bash
  $ git status -s
   M README # 在工作区已修改但尚未暂存
  MM Rakefile # 已修改，暂存后又作了修改。该文件的修改中既有已暂存的部分，又有未暂存的部分
  A  lib/git.rb # 新添加到暂存区中的文件
  M  lib/simplegit.rb # 已修改且已暂存，尚未提交
  ?? LICENSE.txt # 未跟踪文件
  ```

  输出中有两栏，左栏指明了暂存区的状态，右栏指明了工作区的状态。

> git diff

`git diff` 能通过文件补丁的格式更加具体地显示哪些行发生了改变

- 所谓的文件补丁就是说：文件中的哪些内容变成了哪些内容

工作目录中**当前文件和暂存区域快照**之间的差异，即**修改之后还没有暂存**起来的变化内容：

- git diff 本身只显示尚未暂存的改动，而不是自上次提交以来所做的所有改动

`git diff`、`git diff --staged` 和 `git diff HEAD` 的差别：

当一个文件做了 stage，然后又做了一些修改，则：

- `git diff` 显示当前工作区的文件和 stage 区文件的差异
- `git diff --staged` 显示 stage 区和 HEAD 的文件的差异
- `git diff HEAD` 显示工作区和上次递交文件的差异

## 状态转换

### add

**跟踪新文件**：

```bash
$ git add <files>
```

- 是从 `untracked` 状态转移到 `staged` 状态
- 命令使用文件或目录的**路径**作为参数
- 如果参数是目录的路径，该命令将**递归**地跟踪该**目录**下的所有文件

**暂存更新**：

``` bash
$ git add <files>
```

- 是从 `modified` 状态转移到 `staged` 状态

<u>*todo：`git add --patch` 来部分暂存文件？？？*</u>

> **关于 `git add` 命令**：
>
> - 可以用它开始**跟踪新文件**
> - 或者把已跟踪的文件**放到暂存区**
> - 还能用于合并时把有冲突的文件**标记为已解决状态**等
>
> 将这个命令理解为“精确地将内容添加到下一次提交中”而不是“将一个文件添加到项目中”要更加合适
>
> **注意**：
>
> 如果一个文件已经暂存，即处于了 staged 状态，等待提交；这时我们重新编辑这个文件，那么这个文件就会同时出现在两个状态中，即 staged 和 modified 状态（not staged）。如果现在提交，该文件的版本将会是最后一次运行 `git add` 时的那个版本，而不是在工作目录中的当前版本。
>
> 所以，运行了 `git add` 之后又作了修订的文件，需要重新运行 `git add` 把最新版本重新暂存起来
>
> 即：
>
> - 流程1：修改-->add-->commit
>
>   提交的就是修改
>
> - 流程2：修改1-->add-->修改2-->commit
>
>   提交的是修改2
>
> 实际上对流程2来说，如果使用的是 `git commit -a` 的话，那么就不用最终在 commit 之前执行一次 add 命令。
>

### commit

**提交**更新：

``` bash
$ git commit
```

- 一般来说**在 commit 之前，都会先用 `git status` 看下是不是需要的文件都暂存起来了**
- 执行这条命令会启动文本编辑器（配置指定），编辑器会显示最后一次运行 `git status` 的输出，退出编辑器后，Git 会丢弃注释行

直接连**带消息提交**，不打开编辑器：

``` bash
$ git commit -m "message"
```

- 提交之后会显示：在哪个分支提交、提交的 SHA-1 校验和、多少文件修订过、多少行添加删改过

跳过使用暂存区：

``` bash
$ git commit -a
```

-  `-a` 选项使本次提交包含了**所有修改过的文件**
- 请注意：有时这个 `-a` 选项会将**不需要的文件添加到提交中**。例如：我在实现功能A的时候发现一个功能B的BUG并且我把它修复了，这个时候我应该把B的BUG修复代码暂存然后提交，然后再把A的代码暂存再提交。如果我使用 `-a` 选项，那么就一并提交了，这样的话提交历史就不能反应真实情况

> **注意**：
>
> - ⭐提交的是暂存区的快照

### rm

从**工作区删除**：

``` bash
$ rm <file>
```

- 操作后进入 `modified` 状态
  - 如果想要恢复，用`git checkout -- <file>`
- 继续执行 `git add ` 或者 `git rm` 会把这次的删除操作记录到暂存区。然后通过 `commit`  会记录到版本库

从**版本库移除**：

``` bash
$ git rm <file>
```

- 操作后进入 `staged` 状态
  - 如果想要恢复，用`git reset HEAD <file>`，然后再`git checkout -- <file>` 
- ⭐注意和 `rm <file>` 的关系
  - `rm <file>` + `git add` = `git rm` 
  - `rm <file>` + `git rm` = `git rm`

---

**注意**：

`git rm <file>` 并且 `git commit` 是对工作目录下的删除操作进行了一个记录，会在仓库里生成一个新的版本号，在该版本下没有该文件。但是在之前的版本中还是有该文件的。

这个时候如果是误删除，仍然可以**“找回”文件**：

1. 用 `git reset --hard <commitid>` 进行**版本回退**，这样会改变当前的版本号
2. 用 `git checkout <commitid> <file>` 可以**取出某个版本下的文件放到工作区中**，这不会改变当前版本号

---

删除已修改未暂存，或者已暂存的文件：

``` bash
$ git rm -f <file> # f 是 force 首字母
```

- 操作后进入 `staged` 状态

让一个文件保留在磁盘上，但是**不想让 Git 继续跟踪**：

``` bash
$ git rm --cached <file>
```

- 相当于 `git rm <file>`，不同的是在工作区中还保留了文件

> **注意**：
>
> - 不管是何种操作，都可以还原文件。删除后不会删除过往版本的文件。

![](https://gitee.com/matrixPeak/BlogArticle/raw/image/Git-删除操作.png)

### mv

Git 并不显式跟踪文件移动操作。 如果在 Git 中**重命名**了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作，但是 Git 却能够推断出是改名

``` bash
$ git mv <file_from> <file_to>
```

上述命令等效于：

``` bash
$ mv <file_from> <file_to>
$ git rm <file_from>
$ git add <file_to>
```

- 即时使用这三条命令，Git 仍能识别出是重命名
- 有时候用其他工具批处理重命名的话，要记得在提交前删除旧的文件名，再添加新的文件名

# 撤销操作

撤销操作不一定是我们在删除后使用的，我们可能想要查看之前的修改，或者替换一个修改，或者替换一个提交等等。

注意：在 Git 中任何 **已提交** 的东西几乎总是可以恢复的。 甚至那些被删除的分支中的提交或使用 `--amend` 选项覆盖的提交也可以恢复 。 然而，**任何你未提交的东西丢失后很可能再也找不到了**

撤销有几种做法，可以对应处于不同状态的文件：

- **已经提交的文件**，对提交进行撤销：`revert` 和 `reset`
  - 替换上一次：`ammend`
- **未提交**：`checkout`
- 其他：`rm`系列

## revert

-  撤销某次操作

- 此次操作之前和之后的commit和history都会保留

- 把这次撤销，作为一次**最新的提交**

- **是提交一个新的版本，将需要revert的版本的内容再反向修改回去，版本会递增，不影响之前提交的内容**

- 不会改变过去的历史，所以是首选方式，没有任何丢失代码的风险

- `git revert` 命令只能抵消上一个提交，如果想抵消多个提交，必须在命令行依次指定这些提交

- 比如，抵消前两个提交

  ``` bash
  $ git revert [倒数第一个提交] [倒数第二个提交]
  ```

  `git revert`命令还有两个参数：

   - `--no-edit`：执行时不打开默认编辑器，直接使用 Git 自动生成的提交信息。
   - `--no-commit`：只抵消暂存区和工作区的文件变化，不产生新的提交。

## reset

``` bash
# 重置当前分支的HEAD指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [last good SHA]

# 重置当前分支的HEAD指针为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [last good SHA]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]
```

针对含有 `commit-id` 的命令（即上两条）：

- 丢弃掉某个提交之后的所有提交
- 🔴🔴🔴注意!**危险**的命令：以前的提交在历史中彻底消失
- 原理是，让最新提交的指针回到以前某个时点，该时点之后的提交都从历史中消失
- `git reset`不改变工作区的文件（但会改变暂存区），`--hard`参数可以让**工作区**里面的文件也回到以前的状态
  - ⭐⭐⭐执行`git reset`命令之后，如果想找回那些丢弃掉的提交，可以使用`git reflog`命令。不过，这种做法有时效性，**时间长了可能找不回来**
- 🔴🔴🔴注意!**危险**的命令：使用这样的命令，如果要对远端进行同步，则需要执行强制提交：`git push -f`

## --amend

``` bash
$ git commit --amend -m "Fixes bug #42"
```

- **替换掉上一次**提交产生的提交对象
- 最终只有一个提交对象
- 做这样的选择时需要考虑这么几点：
  1. 如果已经推送到远端，并且只要有其他人拉取下来的可能性，那么就不要使用这个命令
  2. 如果已经推送到远端，但是整个仓库只有自己使用，那么可以使用这个命令
  3. 如果在本地，那么就更可以使用这条命令了

## checkout

``` bash
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit-id] [file]
```

- （第一条命令）对还没有提交的文件进行撤销
- （第一条命令）如果工作区的某个文件被改乱了，但还没有提交，可以用`git checkout`命令找回本次修改之前的文件
- （第一条命令）先找暂存区，如果该文件有暂存的版本，则恢复该版本，否则恢复上一次提交的版本
- 🔴🔴🔴注意!**危险**的命令：工作区的文件变化一旦被撤销，就无法找回

## 找回文件

1. 用 `git reset --hard <commitid>` 进行**版本回退**，这样会改变当前的版本号
2. 用 `git checkout <commitid> <file>` 可以**取出某个版本下的文件放到工作区中**，这不会改变当前版本号

## 其他

- 从 `stage` 状态转移到 `modified` 状态 `git reset HEAD <file>`，从 `modified` 状态转移到 `unmodified` 状态 `git checkout -- <file>` ， 具体内容请查看 `rm` 命令小节

- 撤销当前分支的变化：

  即在当前分支上做了几次提交，突然发现放错了分支，这几个提交本应该放到另一个分支

  ```bash
  # 新建一个 feature 分支，指向当前最新的提交
  # 注意，这时依然停留在当前分支
  $ git branch feature
  
  # 切换到这几次提交之前的状态
  $ git reset --hard [当前分支此前的最后一次提交]
  
  # 切换到 feature 分支
  $ git checkout feature
  ```
  
- 当文件加入了 stage 区以后，如果要从 stage 删除，则使用 reset，此时工作区的文件不做任何修改，比如：`git reset hello.py`。这个命令就是 `git stage hello.py` 的反操作。

- 当文件加入了 stage 区以后，后来又做了一些修改，这时发现后面的修改有问题，想回退到stage 的状态，使用 checkout 命令：`git checkout hello.py`

---

本节参考和建议阅读：

- [阮一峰的网络日志：如何撤销 Git 操作？](http://www.ruanyifeng.com/blog/2019/12/git-undo.html)
- 详细的 revert 和 reset 区别，可以查看 [CSDN-游笑天涯-Git恢复之前版本的两种方法reset、revert（图文详解）](https://blog.csdn.net/yxlshk/article/details/79944535)，该文有详细的流程图，包括了涉及到远端以 GitHub 为例的图

# 分支操作

## 简单分支操作

分支切换：

``` bash
$ git checkout <branchName>
```

- 这条命令做了两件事。 一是使 HEAD 指回 `master` 分支，二是将**工作目录**恢复成 `master` 分支所指向的快照内容。 

合并分支：

把 `branch B` 合并到 `branch A`：

``` bash
# 首先切换到想合并入的分支，即 branch A
$ git checkout <branchA>
# 然后执行 merge 命令
$ git merge <branchB>
```

查看已经合并到当前分支的分支：

``` bash
$ git branch --merged
```

- 在这个列表中分支名字前没有 `*` 号的分支通常可以使用 `git branch -d` 删除掉；你已经将它们的工作整合到了另一个分支，所以并不会失去任何东西。

查看尚未合并到当前分支的分支：

``` bash
$ git branch --no-merged
```

- 如果分支包含了还未合并的工作，尝试使用 `git branch -d` 命令删除它时会失败

  当然也可以使用使用 `-D` 选项强制删除它

> 你总是可以提供一个附加的参数来查看其它分支的合并状态而不必检出它们。 例如，尚未合并到 `master` 分支的有哪些？
>
> ``` bash
> $ git branch --no-merged master
> ```
>
> - 这和上面两条查看已经合并和尚未合并到当前分支的分支，不同的是，这个命令不需要切换到 “当前分支”

## 远程分支操作

- 远程跟踪分支是远程分支状态的引用。它们是你无法移动的本地引用。
  - 远程分支一般无法修改，你只能在本地创建它的跟踪分支，然后将跟踪分支推送到远端，如果允许合并才会让你合并
- 远程跟踪分支以 `<remote>/<branch>` 的形式命名
  - “origin” 就和 “master” 一样并无特殊含义
- 本地的分支并**不会自动与远程仓库同步**——你必须显式地推送想要分享的分支
- 远程分支的数据基本来自于你从每个服务器上最后一次抓取的数据。它只会告诉你关于**本地缓存**的服务器数据。 如果想要统计最新的领先与落后数字，需要在运行此命令前抓取所有的远程仓库。

Git 自动将 `serverfix` 分支名字展开为 `refs/heads/serverfix:refs/heads/serverfix`， 那意味着，“推送本地的 `serverfix` 分支来更新远程仓库上的 `serverfix` 分支。” 

``` bash
$ git push origin serverfix:awesomebranch
```



**跟踪分支**：

- 从一个远程跟踪分支检出一个本地分支会自动创建所谓的“跟踪分支”
- 如果在一个跟踪分支上输入 `git pull`，Git 能自动地识别去哪个服务器上抓取、合并到哪个分支

查看设置的所有跟踪分支：

```bash
$ git branch -vv
  iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
  master    1ae2a45 [origin/master] deploying index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should ...
  testing   5ea463a trying something new
```

可以看到：

- `iss53` 分支正在跟踪 `origin/iss53` 并且 “ahead” 是 2，意味着本地有两个提交还没有推送到服务器上
-  `master` 分支正在跟踪 `origin/master` 分支并且是最新的
- `serverfix` 分支正在跟踪 `teamone` 服务器上的 `server-fix-good` 分支并且领先 3 落后 1， 意味着服务器上有一次提交还没有合并入同时本地有三次提交还没有推送
- `testing` 分支并没有跟踪任何远程分支

拉取所有远程仓库：

```bash
$ git fetch --all;
```

拉取远程仓库的变化，并与本地分支合并：

```bash
$ git pull [remote] [branch]
```

> **fetch 和 pull** ：
>
> - `fetch` 命令不会修改工作目录中的内容，它只会获取数据然后让你自己合并
> - `git pull` 在大多数情况下它的含义是一个 `git fetch` 紧接着一个 `git merge` 命令
> - `git pull` 会查找当前分支所跟踪的服务器与分支， 从服务器上抓取数据然后尝试合并入那个远程分支。
> - 由于 `git pull` 的魔法经常令人困惑所以通常单独显式地使用 `fetch` 与 `merge` 命令会更好一些。
> - 可以这样理解：fetch/clone 的内容会在版本库体现，但是不会在工作区体现出来；而pull 的内容会在工作区体现出来

删除远程分支：

``` bash
$ git push <remote> --delete <branch>
```

- 基本上这个命令做的只是从服务器上移除这个指针。

## 变基

在 Git 中整合来自不同分支的修改主要有两种方法：`merge` 以及 `rebase`。本节讲述变基操作

- 变基操作的实质是丢弃一些现有的提交，然后相应地新建一些内容一样但实际上不同的提交

请在合适的时候使用变基，变基使用情况示例：

- 如果别人可能在提交上进行开发，请不要执行变基（即变基可能“丢弃”的那些提交）
- 如果分支只处于本地，并且变基会导致我们提交一个干净整洁的历史，那么我们应当执行变基，这会使得远端仓库的工作更简单
- 总的原则是，只对尚未推送或分享给别人的本地修改执行变基操作清理历史， 从不对已推送至别处的提交执行变基操作

如果遇到别的“可恶”的同事或者说“愚蠢”的自己真的执行了某些麻烦的变基，请查看 Git 官方文档的[用变基解决变基](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA#_rebase_rebase)，本文不在此处说明

---

先切换到主题分支，再执行变基命令：

``` bash
$ git checkout <topicbranch>
$ git rebase <basebranch>
```

直接将主题分支变基到目标分支：

``` bash
$ git rebase <basebranch> <topicbranch>
```

在上面两种情况之后一般会再执行一次快进合并：

``` bash
$ git checkout <basebranch>
$ git merge <topicbranch>
```

---

**下面讲述两种不同的变基情况：**

> **对主题分支上的另一个主题分支执行变基（涉及两个分支）**

原提交历史：

![](https://git-scm.com/book/en/v2/images/basic-rebase-1.png)

检出 `experiment` 分支，然后将它变基到 `master` 分支上：

``` bash
$ git checkout experiment
$ git rebase master
```

![](https://git-scm.com/book/en/v2/images/basic-rebase-3.png)

然后回到 `master` 分支，进行一次快进合并：

``` bash
$ git checkout master
$ git merge experiment
```

![](https://git-scm.com/book/en/v2/images/basic-rebase-4.png)

> **截取主题分支上的另一个主题分支，然后变基到其他分支（涉及三个分支）**

原提交历史：

![](https://git-scm.com/book/en/v2/images/interesting-rebase-1.png)

```bash
$ git rebase --onto master server client
```

- 以上命令的意思是：“取出 `client` 分支，找出它从 `server` 分支分歧之后的补丁， 然后把这些补丁在 `master` 分支上重放一遍，让 `client` 看起来像直接基于 `master` 修改一样”

![](https://git-scm.com/book/en/v2/images/interesting-rebase-2.png)

快进合并 `master` 分支

``` bash
$ git checkout master
$ git merge client
```

![](https://git-scm.com/book/en/v2/images/interesting-rebase-3.png)

将 `server` 分支中的修改也整合进来

```bash
$ git rebase master server
```

- 使用 `git rebase  ` 命令可以直接将主题分支 （即本例中的 `server`）变基到目标分支（即 `master`）上

![](https://git-scm.com/book/en/v2/images/interesting-rebase-4.png)

然后就可以快进合并主分支 `master` 了：

```bash
$ git checkout master
$ git merge server
```

![](https://git-scm.com/book/en/v2/images/interesting-rebase-5.png)

# gitignore

创建名为 `.gitignore` 的文件。

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以 `#` 开头的行都会被 Git 忽略。
- 可以使用标准的 **glob 模式**匹配，它会**递归**地应用在整个工作区中。
- 匹配模式可以以（`/`）开头防止递归。
- 匹配模式可以以（`/`）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（`!`）取反。

常见的忽略情况：

- 自动生成的文件：日志文件、编译过程的临时文件
- <u>*todo：小组合作时避免小组成员提交不合适的文件？*</u>
- IDE等的配置文件：`.idea`、`.vscode`

GitHub 的一个常见  `.gitignore` 文件列表：

- https://github.com/github/gitignore

`.gitignore` 文件的意义、使用点和规范

> 一个仓库可能只根目录下有一个 `.gitignore` 文件，它递归地应用到整个仓库中。 然而，子目录下也可以有额外的 `.gitignore` 文件。子目录中的 `.gitignore` 文件中的规则只作用于它所在的目录中。
>
> <u>*todo：多个 `.gitignore` 文件*</u>

# 其他

## 令我迷惑的命令

- `git add`

- 在 git 的后续版本中，就做了两个修改：
  `git stage` 作为 `git add` 的一个同义词
  `git diff --staged` 作为 `git diff --cached` 的相同命令

- `git diff`、`git diff --staged` 和 `git diff HEAD` 的差别

- `rm`系列：这在 rm 小节结尾已经详细的解释了

- `checkout`命令：
  
  - 可以撤销
  
    可以回退到stage时的状态：`git checkout -- hello.py`
  
  - 可以切换分支
  
  - 创建本地的跟踪远端分支
  
- 那么如果一个文件从已跟踪变成未跟踪了呢？之前的记录还存在嘛？存在！

## 本地和远程的交互

`clone` 命令、仓库几乎全部、远程分支等都和远程相关，其中标签小节也涉及到一系列的远程操作

## 危险的命令

对危险命令的注意：

有些撤消操作是不可逆的。 这是在使用 Git 的过程中，会因为操作失误而导致之前的工作丢失的少有的几个地方之一。

一些危险的命令

- `git reset` 确实是个危险的命令，如果加上了 `--hard` 选项则更是如此。

- `git checkout -- ` 是一个危险的命令

<u>*todo：把所有危险的命令进行一遍整理*</u>

<u>*todo：如果你执行过 `prune` 和 `gc` 操作，可能就找不回。？？？*</u>

## 常见 GUI 工具

<u>*todo：对比 Git 的 GUI操作和命令操作，针对各个 GUI 操作的一个使用说明*</u>

- 包括单独针对文件系统的 `tortoisegit`
- vs code 的 git
- idea 的 git

---

<u>*todo：所有命令对工作区、版本库、暂存区的操作，做一个图*</u>

<u>*todo： merge-base ??*</u>

*<u>todo：如果一个分支b合并入另一个分支a，但这是分支b没有删除，那么每次提交的时候b的头指针会不会移动呢？</u>*

<u>*todo：“检出” 是什么意思，有几个意思*</u>

<u>*todo：倒数第一个提交？倒数第二个提交的快捷引用？ HEAD   HEAD^ ？？？*</u>

<u>*todo： git 的所有命令是否支持正则匹配或者说glob匹配*</u>