注意：关于git的文章在互联网上许许多多，因此对于很多具体的细节并不会在此陈述。相反，本文章会给出一些大体的方向、分类、重要要点等去辅助学习，读者或者作者本人也可以参考这个文章指南和引用资料来查找具体参数和教程加深理解

# Git基础

> 基础的git命令

``` shell
git init
git clone <url> [<repo name>]
git commit
git commit -m "message here to commit"

# 文件操作
git status
git add <filename>
git diff
git diff --staged
# 删除操作
git rm
git rm -f
git rm -cached
# 改名/移动
git mv <old name> <new name>

# 撤销操作
git commit --amend
git reset HEAD <file>
git checkout -- <file>
# 危险的/需要注意的操作
git reset
git checkout -- <file>

# 远程仓库简单命令
git remote
git remote -v
git remote show <remote>
git remote add <shortname> <url>
git fetch <remote>
git push <remote> <branch>
git remote rename <old name> <new name>
git remote remove <repo name>
git remote rm <repo name>

# 标签
git tag
git tag -l
git tag --list
git tag -l "特定模式"
git show <tag name>
# 创建附注标签
git tag -a <tag name> -m "msg"
# 创建轻量标签
git tag <tag name>
# 对过去的提交创建附注标签的示例
# (使用 git log --pretty=oneline 查看历史)
git tag -a <tag name> 9fceb02
# 推送标签
git push <remote> <tagname>
# 推送所有标签
git push origin --tags
# 删除标签
git tag -d <tagname>
# 删除/修改本地标签后 更新 远程仓库的标签
git push <remote> --delete <tagname>
git push <remote> :refs/tags/<tagname>

# .gitignore文件
```

要点：

- 文件的状态变化

  ![](E:\_data\博文临时库\博文中的图片\Git-文件状态变化.png)
  - 有一种情况：文件同时出现在暂存区和非暂存区。

    原因如下：

    - Git 只暂存了运行 `git add` 命令时的文件版本。
    - 如果运行 `git commit` 提交，文件的版本是最后一次运行 `git add` 命令时的那个版本，而不是运行 `git commit` 这条语句时工作目录中的当前版本。

    所以，运行了 `git add` 之后又作了修订的文件，需要重新运行 `git add` 把最新版本重新暂存起来

- `git add`: 添加文件到版本库 **或** 暂存更新 (**两个功能！**)

- 状态查看：

  - 繁琐详细：`git status`
  - 简单紧凑：`git status -s` 或 `git status --short` （标记指明了暂存区和工作区状态）

-  `.gitignore` 使用标准的 glob 模式匹配

  - 子目录下可以有额外的 `.gitignore` 文件，子目录中的 `.gitignore` 文件中的规则只作用于它所在的目录中

  - 其他参考：[GitHub 一个十分详细的针对数十种项目及语言的 `.gitignore` 文件列表](https://github.com/github/gitignore)

- `git diff`：比较的是工作目录中**当前文件和暂存区域**快照之间的差异。也就是**修改之后还没有暂存起来**的变化内容。

  - git diff 本身只显示尚未暂存的改动，而不是自上次提交以来所做的所有改动。所以有时候你一下子暂存了所有更新过的文件，运行 `git diff` 后却什么也没有，就是这个原因

- `git diff --staged`：比对**已暂存文件与最后一次提交**的文件差异

- 删除操作：

  `git rm` ：

  - 直接运行：从暂存区域移除，然后提交，并连带从工作目录中删除指定的文件
  - 手动删除文件后，再执行，也会记录一处文件操作
  - `git rm`命令的本质就是`rm` 和 `git add`
  - `git rm -f`：删除所有修改记录中的指定文件，这样的数据不会被恢复

  `git rm --cached`：

  - 让文件保留在磁盘，但是并不让 Git 继续跟踪

- 移动文件/重命名文件：`git mv <old name> <new name>`

  其实，运行 `git mv` 就相当于运行了下面三条命令：

  ```shell
  mv <old name> <new name>
  git rm <old name>
  git add <new name>
  ```
  - 有时候用其他工具**批处理**重命名的话，**要记得在提交前删除旧的文件名，再添加新的文件名**

- 撤销操作：

  - `git commit --amend` 这个命令会将暂存区中的文件提交，移交后会覆盖掉上一次提交

  - `git reset HEAD <file>`：取消暂存文件

  - `git checkout -- <file>`：还原成上次提交时的样子

    （这是个危险的命令，因为你对那个文件自上次提交后在本地的任何修改都会消失。如果你仍然想保留对那个文件做出的修改，但是现在仍然需要撤消，在 [Git 分支](https://git-scm.com/book/zh/v2/ch00/ch03-git-branching) 保存进度与分支，通常是更好的做法）

- 远程仓库：

  - 可以有好几个远程仓库
  - `git fetch <remote>`：
    - 会访问远程仓库，从中拉取所有你还没有的数据
    - 将会拥有远程仓库中所有分支的引用
    - 只会将数据下载到本地仓库——它不会自动合并或修改当前的工作，需要手动合并入工作
  - `git push <remote> <branch>`：
    - 只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效
    - 当别人先推送时，必须先抓取他们的工作并将其合并进你的工作后才能推送
  - `git remote show <remote>`：
    - 会列出远程仓库的 URL 与跟踪分支的信息
    - 在特定的分支上执行 `git push` 会自动地推送到哪一个远程分支
    - 哪些远程分支不在你的本地，哪些远程分支已经从服务器上移除了
    - 当执行 `git pull` 时哪些本地分支可以与它跟踪的远程分支自动合并

- 打标签：

  比较有代表性的是人们会使用这个功能来标记发布结点（ `v1.0` 、 `v2.0` 等等）

  - 轻量标签：

    很像一个不会改变的分支——它只是某个特定提交的引用

  - 附注标签：

    存储在 Git 数据库中的一个完整对象， 它们是可以被校验的，其中包含打标签者的名字、电子邮件地址、日期时间， 此外还有一个标签信息，并且可以使用 GNU Privacy Guard （GPG）签名并验证

  - 默认情况下，`git push` 命令并不会推送标签到远程仓库服务器上，需要显示推送

  - `git tag -d <tagname>`：

    并不会从任何远程仓库中移除这个标签，你必须用 `git push  :refs/tags/` 来更新你的远程仓库

# Git分支



# 分布式Git

## GitHub

如果你的当前分支设置了跟踪远程分支（阅读下一节和 [Git 分支](https://git-scm.com/book/zh/v2/ch00/ch03-git-branching) 了解更多信息）， 那么可以用 `git pull` 命令来自动抓取后合并该远程分支到当前分支。 这或许是个更加简单舒服的工作流程。默认情况下，`git clone` 命令会自动设置本地 master 分支跟踪克隆的远程仓库的 `master` 分支（或其它名字的默认分支）。 运行 `git pull` 通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支。

# 参考

> 1. [Pro Git book](https://git-scm.com/book/zh/v2)
> 2. [演练场: learnGitBranching](https://learngitbranching.js.org/?locale=zh_CN)
> 3. 

