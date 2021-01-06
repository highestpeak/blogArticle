---
title: Git wiki 清单版
date: 2020-10-17 11:04:00
status: PUBLISHED
comments: true
description: 这是一篇 wiki ,主要介绍一些常见的 Git 命令。这个 wiki 是清单版，一些复杂的命令的说明请查看清单版。注意：你只应该把 wiki 来作为一个速查表。
tags: 
- Git
- 版本控制
- wiki
categories: 
- 软件开发环境与工具
---

注意：

- 仅有命令和简要的一行说明，方便快速查阅
- 请自行尝试命令的组合情况

# 简单命令

**配置：**

``` bash
# 列出 Git 能找到的所有配置
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 查看所有的配置以及它们所在的文件
$ git config --list ==show-origin

# 检查 Git 的某项配置的值
$ git config <key>
$ git config user.name

# 检查最终是哪个配置文件设置了配置项的值
$ git config --show-origin <key>
$ git config --show-origin rerere.autoUpdate
# file:/home/johndoe/.gitconfig	false

# 设置用户名和邮箱
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com

# 设置默认文本编辑器
$ git config --globale core.editor emacs
# 在 Windows 系统上，如果你想要使用别的文本编辑器，那么必须指定可执行文件的完整路径
```

**别名：**

``` bash
# 简写 commit
$ git config --global alias.ci commit

# 取消暂存命令的别名
$ git config --global alias.unstage 'reset HEAD --'
# 下面两句等价
# git unstage fileA
# git reset HEAD == fileA

# 查看最后一次提交
$ git config --global alias.last 'log -l HEAD'
```

**帮助：**

``` bash
# 全面手册，弹出网页（本地网页）
$ git help <verb>
$ git <verb> --help
$ man git-<verb>

# 可用选项快速参考
$ git <verb> -h
```

# 显示信息

## 历史

> 尝试组合这些命令

- 简单命令

  ``` bash
  # 按时间先后顺序列出所有提交
  # 显示当前分支的版本历史
  $ git log
  
  # 显示当前分支的最近几次提交
  $ git reflog
  ```

- log 选项

  ``` bash
  # 每次提交的简略统计信息
  # 显示commit历史，以及每次commit发生变更的文件
  $ git log --stat
  
  # 按照数量限制
  $ git log -[n]
  # 例如 git log -2
  
  # 按照时间限制，显示最近两周提交
  $ git log --since=2.weeks
  
  # 搜索提交历史，根据关键词
  $ git log -S [keyword]
  
  # 显示某个文件的版本历史，包括文件改名
  $ git log --follow [file]
  $ git whatchanged [file]
  
  # 显示指定文件相关的每一次diff
  $ git log -p [file]
  $ git log -patch [file]
  
  # 形象地展示你的分支、合并历史
  $ git log --graph
  
  # ⭐ --pretty 选项==================
  # 将每个提交放在一行显示
  $ git log --pretty=oneline 
  
  # 定制记录的显示格式
  $ git log --pretty=format:"%h - %an, %ar : %s"
  ```

- 综合应用和其他

  ``` bash
  # 显示过去5次提交
  $ git log -5 --pretty --oneline
  
  # 显示所有提交过的用户，按提交次数排序
  $ git shortlog -sn
  
  # 显示某个commit之后的所有变动，每个commit占据一行
  $ git log [tag] HEAD --pretty=format:%s
  
  # 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
  $ git log [tag] HEAD --grep feature
  ```

## status 和 diff

``` bash
# 显示有变更的文件
# 检查所有文件的状态
$ git status

# 状态简览
$ git status -s
$ git status --short

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
# 已暂存文件与最后一次提交
# file 可选参数
$ git diff --cached [file]
$ git diff --staged [file]

# 如果暂存后又进行了编辑，则查看完整的变化需要
# 注意对比
# 看暂存前后的变化
$ git diff
# 查看已经暂存起来的变化,
$ git diff --cached

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 查看任意两个版本之间的改动
# 显示两次提交之间的差异
# file 可选
$ git diff [commit 1] [commit 2] [file]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"
```

## 其他

``` bash
# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]
```

# 仓库

``` bash
# 在当前目录新建一个Git代码库
$ git init

# 下载一个项目和它的整个代码历史
$ git clone [url]

# 克隆并重命名
$ git clone [url] [newName]

# Clone a specific tag name using git clone 
$ git clone <url> --branch=<tag_name>

# 添加远程仓库
$ git remote add <shortname> <url>

# 查看所有远程仓库: 每一个远程服务器的简写
$ git remote

# 查看所有远程仓库: 远程服务器的简写与其对应的 URL
$ git remote -v 

# 查看更多远程仓库信息
$ git remote show <remote>
# 输出：仓库URL、分支跟踪信息、哪些远程分支不在本地、哪些远程分支已经从服务器上移除

# 移除远程仓库跟踪
$ git remote remove <remote>
# 或者
$ git remote re <remote>

# 从远程仓库抓取
$ git fetch <remote>
```

# 文件状态

``` bash
# 跟踪新文件
# 暂存更新
# git stage 作为 git add 的一个同义词
$ git add <files>
# 添加当前目录的所有文件到暂存区
$ git add .
# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 提交更新
$ git commit
# 带消息提交，不打开编辑器
$ git commit -m "message"
$ git commit [file1] [file2] ... -m [message]
# 跳过使用暂存区
# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a
# 提交时显示所有diff信息
$ git commit -v
# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]
# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...

# 从工作区删除
$ rm <file>
# 从版本库移除
$ git rm <file>
# 删除已修改未暂存，或者已暂存的文件
$ git rm -f <file> # f 是 force 首字母
# 让一个文件保留在磁盘上，但是不让 Git 继续跟踪
$ git rm --cached <file>

# 重命名文件
$ git mv <file_from> <file_to>
# 等效于
$ mv <file_from> <file_to>
$ git rm <file_from>
$ git add <file_to>
```

# 撤销

- `revert`、`reset`、`amend`、`checkout`

``` bash
# 抵消前两个提交，提交一个新的提交，前两次还在
$ git revert [倒数第一个提交] [倒数第二个提交]

# 重置当前分支的HEAD指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [last good SHA]

# 重置当前分支的HEAD指针为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [last good SHA]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]
# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...

# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit-id] [file]

# 撤销提交，但是提交做出的修改不撤销，修改会到暂存区
git reset --soft HEAD~1
```

<u>*todo： 关于分支相关的撤销*</u>

# 分支和标签

## 分支

 ```bash
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# ===========================
# 新建一个分支，但依然停留在当前分支，不会自动切换
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# ===========================
# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 创建新分支并立即切换过去
$ git checkout -b <newbranchname>

# ===========================
# 合并分支
# 把 `branch B` 合并到 `branch A`
# 首先切换到想合并入的分支，即 branch A
$ git checkout <branchA>
# 然后执行 merge 命令
$ git merge <branchB>

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]

# ===========================
# 删除分支
$ git branch -d [branch-name]

# 查看各个分支当前所指的对象
$ git log --oneline --decorate

# 查看分叉历史
$ git log --oneline --decorate --graph --all

# ===========================
# 查看每一个分支的最后一次提交
$ git branch -v

# 查看已经合并到当前分支的分支
$ git branch --merged

# 查看尚未合并到当前分支的分支
$ git branch --no-merged
 ```

## 远程分支操作

``` bash
# 获得远程引用的完整列表
$ git ls-remote <remote>

# 获取远程分支的更多信息
$ git remote show <remote>

# 列出所有远程分支
$ git branch -r

# 查看设置的所有跟踪分支
$ git branch -vv

# 推送本地分支
$ git push <remote> <branch>
# 将本地的 `serverfix` 分支推送到远程仓库上的 `awesomebranch` 分支
$ git push origin serverfix:awesomebranch

# 推送所有分支到远程仓库
$ git push [remote] --all

# 重命名远程分支
$ git remote rename <old name> <new name>

# 推送分支到远程仓库
$ git push <remote> <branch>

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 拉取远程分支
$ git checkout -b <branch> <remote>/<branch>
# 这条命令可以将本地分支与远程分支设置为不同的名字
$ git checkout -b sf origin/serverfix

# 更简单的创建跟踪分支
# [remote-branch] (a) 不存在且 (b) 刚好只有一个名字与之匹配的远程分支
$ git checkout [remote-branch]

# 删除远程分支
# 基本上这个命令做的只是从服务器上移除这个指针
$ git push <remote> --delete <branch>
$ git branch -dr [remote/branch]

# 拉取所有远程仓库
$ git fetch --all

# 拉取远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]
```

## 变基

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

## 标签

```bash
# 列出所有tag
$ git tag

# 创建轻量标签
$ git tag <tagname>

# 创建附注标签
$ git tag -a <tagname> -m "git-tag message"

# 对过去的提交打标签
# 需要在命令的末尾指定提交的校验和（或部分校验和）
$ git tag -a <tagname> <SHA-1 short or long>

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tagname]

# 删除远程tag
$ git push origin :refs/tags/[tagName]
# 更直观的删除远程标签的方式
$ git push origin --delete <tagname>

# 查看tag信息
# 标签信息和与之对应的提交信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
# 推送所有不在远程仓库的标签
# remote 可选
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]

# 拉取标签
$ git pull origin --tags
$ git fetch --tags
$ git pull origin :remotes/origin/v2.1

# 克隆特定的标签
$ git clone <url> --branch=<tag_name>

# 按照特定模式查找标签
# 如果你提供了一个匹配标签名的通配模式， `-l` 或 `--list` 就是强制使用的
$ git tag -l "xxxx"
# list all tags with given pattern ex: v-
$ git tag --list 'v-*'

```

> 同时推送标签和分支

- 从 Git 2.4.x 开始：

  ``` bash
  $ git push --atomic origin <branch name> <tag>
  ```

- 在 push 时自动推送：

  ``` bash
  $ git config --global push.followTags true
  ```

  - 如果设置为true，则默认情况下启用--follow-tags选项
  - 您可以在推送时通过指定--no-follow-tags覆盖此配置
  - `--follow-tags`的推送条件：
    -  **仅推送带注释的标签**
    -  来自当前分支的历史记录的可到达标签

- 或者如果不进行全局设置，可以在需要的时候进行附加选项：

  ``` bash
  $ git push --follow-tags
  ```

# 其他命令和注意

- stash 命令

  ``` bash
  # 暂时将未提交的变化移除，稍后再移入
  $ git stash
  $ git stash pop
  ```

  https://www.cnblogs.com/tocy/p/git-stash-reference.html

- GitHub 的一个常见  `.gitignore` 文件列表：

  - https://github.com/github/gitignore

# 标准和最佳实践

## 标准

每一个项目都有一点儿不同。 影响因素包括活跃贡献者的数量、选择的工作流程、提交权限与可能包含的外部贡献方法

- **活跃贡献者的数量**

  随着开发者增多，确保代码能干净地应用或轻松地合并时会遇到更多问题。如何保证代码始终是最新的，并且提交始终是有效的

- **工作流程**

  - 是中心化的吗？
- 每一个开发者都对主线代码有相同的写入权限？ 
  
  - 项目是否有一个检查所有补丁的维护者或整合者？
- 是否所有的补丁是同行评审后批准的？ 
  
  - 是否参与了那个过程？ 
- 是否存在副官系统，你必须先将你的工作提交到上面？
  
- **提交权限**

  - 是否有项目的写权限会使向项目贡献所需的流程有极大的不同。 如果没有写权限，项目会选择何种方式接受贡献的工作？ 

  - 是否甚至有一个如何贡献的规范？ 你一次贡献多少工作？ 你多久贡献一次？

  （<u>*todo: 这不是工作流程的内容嘛？*</u>）

- **提交准则**

  - 不应该包含任何空白错误
  - 每一个提交成为一个逻辑上的独立变更集，一个提交只完成一个问题
  - 使用工具来帮助生成一个干净又易懂的历史（重写历史、 `git add --patch`）
  - 优质的提交信息

## 最佳实践

- 做了阶段性修改，但是还不能做一次递交，这时先 `git stage`/`git add` 一下

  如果有问题，可以随时 checkout 回退。

- 递交之前，使用 `git status`、`git diff HEAD` 仔细查看是否需要的提交，保证提交了所有内容

### 提交信息模板

有一个创建优质提交信息的习惯会使 Git 的使用与协作容易的多。 一般情况下，信息应当以少于 50 个字符（25个汉字）的单行开始且简要地描述变更，接着是一个空白行，再接着是一个更详细的解释。 Git 项目要求一个更详细的解释，包括做改动的动机和它的实现与之前行为的对比——这是一个值得遵循的好规则。 使用指令式的语气来编写提交信息，比如使用“Fix bug”而非“Fixed bug”或“Fixes bug”。这里是一份 [最初由 Tim Pope 写的模板](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)：

```text
首字母大写的摘要（不多于 50 个字符）

如果必要的话，加入更详细的解释文字。在大概 72 个字符的时候换行。
在某些情形下，第一行被当作一封电子邮件的标题，剩下的文本作为正文。
分隔摘要与正文的空行是必须的（除非你完全省略正文），
如果你将两者混在一起，那么类似变基等工具无法正常工作。

使用指令式的语气来编写提交信息：使用“Fix bug”而非“Fixed bug”或“Fixes bug”。
此约定与 git merge 和 git revert 命令生成提交说明相同。

空行接着更进一步的段落。

- 标号也是可以的。

- 项目符号可以使用典型的连字符或星号，后跟一个空格，行之间用空行隔开，
  但是可以依据不同的惯例有所不同。

- 使用悬挂式缩进
```

# 参考

> 1. https://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html



