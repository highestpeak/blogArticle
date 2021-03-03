常见命令：

- sudo、su、ls、man、权限

vim：

- 模式切换

https://zhuanlan.zhihu.com/p/35147209

在这个wiki中，多采用分组

操作系统中 awk、sed 和 grep 这些 shell 文本处理命令的功能都十分强大

第一组

- `某个命令 --h`，对这个命令进行解释
- `某个命令 --help`，解释这个命令(更详细)
- `man某个命令`，文档式解释这个命令(更更详细)(执行该命令后,还可以按/+关键字进行查询结果的搜索)
- `TAB键`，自动补全命令（按一次自动补全，连续按两次，提示所有以输入开头字母的所有命令）

第三组

- `键盘上下键`，输入临近的历史命令
- `history`，查看所有的历史命令
- `Ctrl + r`，进入历史命令的搜索功能模式



- `clear`，清除屏幕里面的所有命令



第八组

- `shutdown`
  - `shutdown -hnow`，立即关机
  - `shutdown -h+10`，10 分钟后关机
  - `shutdown -h23:30`，23:30 关机
  - `shutdown -rnew`，立即重启
- `poweroff`，立即关机（常用）
- `reboot`，立即重启（常用）

第六组

- `date`，查看系统时间（常用）
  - `date -s20080103`，设置日期（常用）
  - `date -s18:24`，设置时间，如果要同时更改 BIOS 时间，再执行 `hwclock --systohc`（常用）
- `cal`，在终端中查看日历，肯定没有农历显示的
- `uptime`，查看系统已经运行了多久，当前有几个用户等信息（常用）
- `last`，显示最近登录的帐户及时间
- `lastlog`，显示系统所有用户各自在最近登录的记录，如果没有登录过的用户会显示 从未登陆过



基本上每个部分可以分为常见使用和原理解析

# 进程操作

主要分为几类：查询进程 `ps` 命令、进程监控、终止进程 `kill` 命令、进程分析

- `Ctrl + c`，结束命令
- `firefox&`，最后后面的 **&** 符号，表示使用后台方式打开 Firefox，然后显示该进程的 PID 值

## ps 命令

**常见使用：**



---

**原理解析**

- ps 命令的所有信息都是 linux kernel 生成，并通过 `/proc/` 目录输出给用户空间的。在 `/proc/` 目录下，每一个以数字开头的目录，就对应一个进程信息。

**命令选项格式：**

了解 ps 命令首先需要从 ps 命令的选项格式入手。像其他很多 linux shell 命令一样，ps 命令的选项也有长格式和短格式的区别。短选项中也可以带中横线、也可以不带中横线。

根据选项长短和是否有横线的情况，ps 命令的选项可以分为以下 3 类：

- BSD 风格语法，必须不能以中横线开头；`ps u`
- SYSV 风格语法，必须仅一个中横线开头；`ps -l`
- GNU 风格语法，必须以两个中横线开头；`ps --pid l`

linux ps 命令的长选项并不多，而且几乎每个长选项都有一个功能完全相同的短选项对应

ps 命令解析 SYSV 和 BSD 风格选项时，会分别将每组字符串都解析成单独的字母。以下三个实例，拆分前后的命令都是等价的。

``` bash
$ ps aux     等价于 $ ps a u x
$ ps -elL    等价于 $ ps -e -l -L
$ ps -el en  等价于 $ ps -e -l e n
```

- 注意第二个给每一个字母前都加上一个中横线
- 注意第三个，指明各风格的 ps 命令选项可以混合使用

**记录类选项：**

（这里的选项是会影响ps命令结果的行的显示数目的）

重点掌握 ps 命令中那些显示所有进程信息记录的选项，其他 ps 命令过滤选项都可以通过 shell 文本处理命令（awk、sed 和 grep）间接实现

- **all_processes 选项**

  显示所有进程信息的选项只有 2 个，即 SYSV 风格的 - e 和 - A，我们牢记 - e 选项

- **simple_select 选项**

  一共 5 个，具体包括 - a、-d、a、g 和 x （ 2 个 SYSV 风格和 3 个 BSD 风格）。相同风格内部可以组合使用，具体的组合可能性有 - ad、ga、ax、gx、agx 。对这六种组合情况，ps通过位图计算来实现筛选规则

  -  \- a 含义：显示所有 tty 值存在的且 session 值不等于当前进程 tgid(pid) 值的进程
  -  d 含义：显示所有 session 值不等于当前进程 tgid(pid) 值的进程；
  - \- ad 含义：过滤掉所有 tty 值为空的且 session 值等于当前进程 tgid(pid) 值的进程；
  - g 含义是：显示所有 tty 值存在的且 euid 值等于当前进程 euid 值的进程
  - ga（或选项 a）含义：显示所有 tty 值存在的进程；
  - gx（或选项 x）含义：显示所有 euid 值等于当前进程 euid 值的进程；

  ---

  ps 命令，会对 BSD 风格 simple_select 选项部分做 2 个特殊处理：

  - 在原来的 BSD 风格 simple_select 情况下，再额外增加一个 g 选项；
  - 如果已经有 a、g 和 x 三个选项都出现了，那么就直接替换为 - e 选项；

  按照这 2 个特殊处理规则，ps aux 选项组合等价于 ps auxg，等价于 ps agx u，等价于 ps -e u

- **selection_list 选项**

  根据进程的某个属性值对进程进行筛选。他们大多需要一个选项的参数。此类选项一共 13 个，主要分为如下几组：

  - 进程ID选项、进程会话ID选项、用户ID选项、用户组ID选项

    ``` bash
    $ ps p 2 或 $ ps -p 2 # 查询PID 值为一个 PID 值的进程信息
    $ ps --pid 2,3,12 # 查询PID 值为几个 PID 值范围的进程信息
    $ ps q 2 或 $ ps -q 2 # 快速模式，不应用额外筛选规则，不允许排序
    
    $ ps -s 1 或 $ ps --sid 1 # 进程会话ID选项
    
    $ ps -u root 或 $ ps U root 或 $ ps --User root # euid # 用户ID选项
    $ ps -U 0 #ruid # 用户ID选项
    
    $ ps -G 0 或 $ ps --Group 0 #rgid # 用户组ID选项
    $ ps -g root 或 $ ps -group root #rgid # 用户组ID选项
    ```

  - 进程名称选项

    ``` bash
    $ ps -C bash
    $ ps -C a2345678901234567 # 实际查询a23456789012345，只使用其前 15 位
    ```

  - 终端tty选项

    ``` bash
    $ ps T 或 $ ps t # 当前终端所有进程
    $ ps t 1 或 $ ps --tty 1
    $ ps -t tty1
    ```

  这些选项可以组合使用

  ``` bash
  $ ps -u root -p 1
  ```

- **特殊选择选项**

  ``` bash
  $ ps h # 不希望出现标题页头
  $ ps -e r # 只希望列出运行中的 R 状态和 D 状态的进程
  
  # 查询命令的补集
  $ ps -e h| wc -l; ps -u root h|wc -l; ps -u root -N h|wc-l
  # 结果为 472、456、16，可以发现472=456+16，即全集=本集+补集
  ```

- 这些纪录类选项有一个作用顺序，但是这里暂且不表，可以看[参考文](https://juejin.im/post/6844903938144075783)

- `ps`

  - `ps –ef|grep java`，查看当前系统中有关 java 的所有进程
  - `ps -ef|grep --color java`，高亮显示当前系统中有关 java 的所有进程

- `kill`
  - `kill 1234`，结束 pid 为 1234 的进程
  - `kill -9 1234`，强制结束 pid 为 1234 的进程（慎重）
  - `killall java`，结束同一进程组内的所有为 java 进程
  - `ps -ef|grep hadoop|grep -v grep|cut -c 9-15|xargs kill -9`，结束包含关键字 hadoop 的所有进程
- `jobs`，查看后台运行的程序列表

https://juejin.im/post/6844903938144075783

https://blog.csdn.net/juS3Ve/article/details/89007912

# 第四组 文件和目录操作

- `pwd`，显示当前目录路径（常用）

- `locate 搜索关键字`，快速搜索系统文件/文件夹（类似 Windows 上的 everything 索引式搜索）（常用）

  - `updatedb`，配合上面的 locate，给 locate 的索引更新（locate 默认是一天更新一次索引）（常用）

- `ls`，列出当前目录下的所有没有隐藏的文件 / 文件夹。

  - `ls -a`，列出包括以.号开头的隐藏文件 / 文件夹（也就是所有文件）

  - `ls -R`，显示出目录下以及其所有子目录的文件 / 文件夹（递归地方式，不显示隐藏的文件）

  - `ls -a -R`，显示出目录下以及其所有子目录的文件 / 文件夹（递归地方式，显示隐藏的文件）

  - `ls -al`，列出目录下所有文件（包含隐藏）的权限、所有者、文件大小、修改时间及名称（也就是显示详细信息）

  - `ls -ld 目录名`，显示该目录的基本信息

  - `ls -t`，依照文件最后修改时间的顺序列出文件名。

  - `ls -F`，列出当前目录下的文件名及其类型。以 **/** 结尾表示为目录名，以 ***** 结尾表示为可执行文件，以 **@** 结尾表示为符号连接

  - `ls -lg`，同上，并显示出文件的所有者工作组名。

  - `ls -lh`，查看文件夹类文件详细信息，文件大小，文件修改时间

  - `ls /opt | head -5`，显示 opt 目录下前 5 条记录

  - `ls -l | grep '.jar'`，查找当前目录下所有 jar 文件

  - `ls -l /opt |grep "^-"|wc -l`，统计 opt 目录下文件的个数，不会递归统计

  - `ls -lR /opt |grep "^-"|wc -l`，统计 opt 目录下文件的个数，会递归统计

  - `ls -l /opt |grep "^d"|wc -l`，统计 opt 目录下目录的个数，不会递归统计

  - `ls -lR /opt |grep "^d"|wc -l`，统计 opt 目录下目录的个数，会递归统计

  - `ls -lR /opt |grep "js"|wc -l`，统计 opt 目录下 js 文件的个数，会递归统计

  - `ls -l`，列出目录下所有文件的权限、所有者、文件大小、修改时间及名称（也就是显示详细信息，不显示隐藏文件）。显示出来的效果如下：

    ```
    -rwxr-xr-x. 1 root root 4096 3月 26 10:57，其中最前面的 - 表示这是一个普通文件
    lrwxrwxrwx. 1 root root 4096 3月 26 10:57，其中最前面的 l 表示这是一个链接文件，类似 Windows 的快捷方式
    drwxr-xr-x. 5 root root 4096 3月 26 10:57，其中最前面的 d 表示这是一个目录
    ```

- `cd`，目录切换

  - `cd ..`，改变目录位置至当前目录的父目录(上级目录)。
  - `cd ~`，改变目录位置至用户登录时的工作目录。
  - `cd 回车`，回到家目录
  - `cd -`，上一个工作目录
  - `cd dir1/`，改变目录位置至 dir1 目录下。
  - `cd ~user`，改变目录位置至用户的工作目录。
  - `cd ../user`，改变目录位置至相对路径user的目录下。
  - `cd /../..`，改变目录位置至绝对路径的目录位置下

- `cp 源文件 目标文件`，复制文件

  - `cp -r 源文件夹 目标文件夹`，复制文件夹
  - `cp -r -v 源文件夹 目标文件夹`，复制文件夹(显示详细信息，一般用于文件夹很大，需要查看复制进度的时候)
  - `cp /usr/share/easy-rsa/2.0/keys/{ca.crt,server.{crt,key},dh2048.pem,ta.key} /etc/openvpn/keys/`，复制同目录下花括号中的文件

- `mv 文件 目标文件夹`，移动文件到目标文件夹

  - `mv 文件`，不指定目录重命名后的名字，用来重命名文件

- `touch 文件名`，创建一个空白文件/更新已有文件的时间(后者少用)

- `mkdir 文件夹名`，创建文件夹

- `mkdir -p /opt/setups/nginx/conf/`，创建一个名为 conf 文件夹，如果它的上级目录 nginx 没有也会跟着一起生成，如果有则跳过

- `rmdir 文件夹名`，删除文件夹(只能删除文件夹里面是没有东西的文件夹)

- `rm 文件`，删除文件

  - `rm -r 文件夹`，删除文件夹
  - `rm -r -i 文件夹`，在删除文件夹里的文件会提示(要的话,在提示后面输入yes)
  - `rm -r -f 文件夹`，强制删除
  - `rm -r -f 文件夹1/ 文件夹2/ 文件夹3/`删除多个

- `find`，高级查找

  - `find . -name *lin*`，其中 . 代表在当前目录找，-name 表示匹配文件名 / 文件夹名，*lin* 用通配符搜索含有lin的文件或是文件夹
  - `find . -iname *lin*`，其中 . 代表在当前目录找，-iname 表示匹配文件名 / 文件夹名（忽略大小写差异），*lin* 用通配符搜索含有lin的文件或是文件夹
  - `find / -name *.conf`，其中 / 代表根目录查找，*.conf代表搜索后缀会.conf的文件
  - `find /opt -name .oh-my-zsh`，其中 /opt 代表目录名，.oh-my-zsh 代表搜索的是隐藏文件 / 文件夹名字为 oh-my-zsh 的
  - `find /opt -type f -iname .oh-my-zsh`，其中 /opt 代表目录名，-type f 代表只找文件，.oh-my-zsh 代表搜索的是隐藏文件名字为 oh-my-zsh 的
  - `find /opt -type d -iname .oh-my-zsh`，其中 /opt 代表目录名，-type d 代表只找目录，.oh-my-zsh 代表搜索的是隐藏文件夹名字为 oh-my-zsh 的
  - `find . -name "lin*" -exec ls -l {} \;`，当前目录搜索lin开头的文件，然后用其搜索后的结果集，再执行ls -l的命令（这个命令可变，其他命令也可以），其中 -exec 和 {} ; 都是固定格式
  - `find /opt -type f -size +800M -print0 | xargs -0 du -h | sort -nr`，找出 /opt 目录下大于 800 M 的文件
  - `find / -name "*tower*" -exec rm {} \;`，找到文件并删除
  - `find / -name "*tower*" -exec mv {} /opt \;`，找到文件并移到 opt 目录
  - `find . -name "*" |xargs grep "youmeek"`，递归查找当前文件夹下所有文件内容中包含 youmeek 的文件
  - `find . -size 0 | xargs rm -f &`，删除当前目录下文件大小为0的文件
  - `du -hm --max-depth=2 | sort -nr | head -12`，找出系统中占用容量最大的前 12 个目录

- `ln -s /opt/data /opt/logs/data`，表示给 /opt/logs 目录下创建一个名为 data 的软链接，该软链接指向到 /opt/data

- `grep`

  - `shell grep -H '安装' *.sh`，查找当前目录下所有 sh 类型文件中，文件内容包含 `安装` 的当前行内容
  - `grep 'test' java*`，显示当前目录下所有以 java 开头的文件中包含 test 的行
  - `grep 'test' spring.ini docker.sh`，显示当前目录下 spring.ini docker.sh 两个文件中匹配 test 的行

# 第五组 网络操作

- `ifconfig`，查看内网 IP 等信息（常用）
- `curl ifconfig.me`，查看外网 IP 信息
- `curl ip.cn`，查看外网 IP 信息
- `cat /etc/resolv.conf`，查看 DNS 设置
- `netstat -tlunp`，查看当前运行的服务，同时可以查看到：运行的程序已使用端口情况

# 第七组 文件内容操作

- `cat 文件路名`，显示文件内容（属于打印语句）
- `cat -n 文件名`，显示文件，并每一行内容都编号
- `more 文件名`，用分页的方式查看文件内容（按 space 翻下一页，按 *Ctrl + B* 返回上页）
- `less`文件名，用分页的方式查看文件内容（带上下翻页）
  - 按 **j** 向下移动，按 **k** 向上移动
  - 按 **/** 后，输入要查找的字符串内容，可以对文件进行向下查询，如果存在多个结果可以按 **n** 调到下一个结果出
  - 按 **？** 后，输入要查找的字符串内容，可以对文件进行向上查询，如果存在多个结果可以按 **n** 调到下一个结果出
- `head`
  - `head -n 10 spring.ini`，查看当前文件的前 10 行内容
- `tail`
  - `tail -n 10 spring.ini`，查看当前文件的后 10 行内容
  - `tail -200f 文件名`，查看文件被更新的新内容尾 200 行，如果文件还有在新增可以动态查看到（一般用于查看日记文件）

# 第九组 压缩文件命令

- `zip mytest.zip /opt/test/`，把 /opt 目录下的 test/ 目录进行压缩，压缩成一个名叫 mytest 的 zip 文件
  - `unzip mytest.zip`，对 mytest.zip 这个文件进行解压，解压到当前所在目录
  - `unzip mytest.zip -d /opt/setups/`，对 mytest.zip 这个文件进行解压，解压到 /opt/setups/ 目录下
- `tar -cvf mytest.tar mytest/`，对 mytest/ 目录进行归档处理（归档和压缩不一样）
- `tar -xvf mytest.tar`，释放 mytest.tar 这个归档文件，释放到当前目录
  - `tar -xvf mytest.tar -C /opt/setups/`，释放 mytest.tar 这个归档文件，释放到 /opt/setups/ 目录下
- `tar cpf - . | tar xpf - -C /opt`，复制当前所有文件到 /opt 目录下，一般如果文件夹文件多的情况下用这个更好，用 cp 比较容易出问题

# 第十组 系统变量操作

- `env`，查看所有系统变量
- `export`，查看所有系统变量
- `echo`
  - `echo $JAVA_HOME`，查看指定系统变量的值，这里查看的是自己配置的 JAVA_HOME。
  - `echo "字符串内容"`，输出 "字符串内容"
  - `echo > aa.txt`，清空 aa.txt 文件内容（类似的还有：`: > aa.txt`，其中 : 是一个占位符, 不产生任何输出）
- `unset $JAVA_HOME`，删除指定的环境变量

# 第十一组 用户和权限

- 使用 pem 证书登录：

  ```
  ssh -i /opt/mykey.pem root@192.168.0.70
  ```

  - 证书权限不能太大，不然无法使用：`chmod 600 mykey.pem`

- `hostname`，查看当前登陆用户全名

- `cat /etc/group`，查看所有组

- `cat /etc/passwd`，查看所有用户

- `groups youmeek`，查看 youmeek 用户属于哪个组

- `useradd youmeek -g judasn`，添加用户并绑定到 judasn 组下

- `userdel -r youmeek`，删除名字为 youmeek 的用户

  - 参数：`-r`，表示删除用户的时候连同用户的家目录一起删除

- 修改普通用户 youmeek 的权限跟 root 权限一样：

  - 常用方法（原理是把该用户加到可以直接使用 sudo 的一个权限状态而已）：

    - 编辑配置文件：`vim /etc/sudoers`
    - 找到 98 行（预估），有一个：`root ALL=(ALL) ALL`，在这一行下面再增加一行，效果如下：

    ```
     root    ALL=(ALL)   ALL
     youmeek    ALL=(ALL)   ALL
    ```

  - 另一种方法：

    - 编辑系统用户的配置文件：`vim /etc/passwd`，找到 **root** 和 **youmeek** 各自开头的那一行，比如 root 是：`root:x:0:0:root:/root:/bin/zsh`，这个代表的含义为：*用户名:密码:UserId:GroupId:描述:家目录:登录使用的 shell*
    - 通过这两行对比，我们可以直接修改 youmeek 所在行的 UserId 值 和 GroupId 值，都改为 0。

- `groupadd judasn`，添加一个名为 judasn 的用户组

- `groupdel judasn`，删除一个名为 judasn 的用户组（前提：先删除组下面的所有用户）

- `usermod 用户名 -g 组名`，把用户修改到其他组下

- `passwd youmeek`，修改 youmeek 用户的密码（前提：只有 root 用户才有修改其他用户的权限，其他用户只能修改自己的）

- `chmod 777 文件名/目录`，给指定文件增加最高权限，系统中的所有人都可以进行读写。

  - linux 的权限分为 rwx。r 代表：可读，w 代表：可写，x 代表：可执行
  - 这三个权限都可以转换成数值表示，r = 4，w = 2，x = 1，- = 0，所以总和是 7，也就是最大权限。第一个 7 是所属主（user）的权限，第二个 7 是所属组（group）的权限，最后一位 7 是非本群组用户（others）的权限。
  - `chmod -R 777 目录` 表示递归目录下的所有文件夹，都赋予 777 权限
  - `chown myUsername:myGroupName myFile` 表示修改文件所属用户、组
  - `chown -R myUsername:myGroupName myFolder` 表示递归修改指定目录下的所有文件权限

- `su`：切换到 root 用户，终端目录还是原来的地方（常用）

  - `su -`：切换到 root 用户，其中 **-** 号另起一个终端并切换账号
  - `su 用户名`，切换指定用户帐号登陆，终端目录还是原来地方。
  - `su - 用户名`，切换到指定用户帐号登陆，其中 **-** 号另起一个终端并切换账号

- `exit`，注销当前用户（常用）

- `sudo 某个命令`，使用管理员权限使用命令，使用 sudo 回车之后需要输入当前登录账号的密码。（常用）

- `passwd`，修改当前用户密码（常用）

- 添加临时账号，并指定用户根目录，并只有可读权限方法

  - 添加账号并指定根目录（用户名 tempuser）：`useradd -d /data/logs -m tempuser`
  - 设置密码：`passwd tempuser` 回车设置密码
  - 删除用户（该用户必须退出 SSH 才能删除成功），也会同时删除组：`userdel tempuser`

# 第十二组 磁盘管理

- `df -h`，自动以合适的磁盘容量单位查看磁盘大小和使用空间
  - `df -k`，以磁盘容量单位 K 为数值结果查看磁盘使用情况
  - `df -m`，以磁盘容量单位 M 为数值结果查看磁盘使用情况
- `du -sh /opt`，查看 opt 这个文件夹大小 （h 的意思 human-readable 用人类可读性较好方式显示，系统会自动调节单位，显示合适大小的单位）
- `du -sh ./*`，查看当前目录下所有文件夹大小 （h 的意思 human-readable 用人类可读性较好方式显示，系统会自动调节单位，显示合适大小的单位）
- `du -sh /opt/setups/`，显示 /opt/setups/ 目录所占硬盘空间大小（s 表示 –summarize 仅显示总计，即当前目录的大小。h 表示 –human-readable 以 KB，MB，GB 为单位，提高信息的可读性）
- `mount /dev/sdb5 /newDir/`，把分区 sdb5 挂载在根目录下的一个名为 newDir 的空目录下，需要注意的是：这个目录最好为空，不然已有的那些文件将看不到，除非卸载挂载。
  - 挂载好之后，通过：`df -h`，查看挂载情况。
- `umount /newDir/`，卸载挂载，用目录名
  - 如果这样卸载不了可以使用：`umount -l /newDir/`
- `umount /dev/sdb5`，卸载挂载，用分区名

# 第十三组 软件下载

- 常规下载：`wget http://www.gitnavi.com/index.html`
- 自动断点下载：`wget -c http://www.gitnavi.com/index.html`
- 后台下载：`wget -b http://www.gitnavi.com/index.html`
- 伪装代理名称下载：`wget --user-agent="Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.16 (KHTML, like Gecko) Chrome/10.0.648.204 Safari/534.16" http://www.gitnavi.com/index.html`
- 限速下载：`wget --limit-rate=300k http://www.gitnavi.com/index.html`
- 批量下载：`wget -i /opt/download.txt`，一个下载地址一行
- 后台批量下载：`wget -b -c -i /opt/download.txt`，一个下载地址一行



# VIM

- Vim 官网：http://www.vim.org/



# 参考

https://github.com/judasn/Linux-Tutorial

https://zhuanlan.zhihu.com/p/36801617