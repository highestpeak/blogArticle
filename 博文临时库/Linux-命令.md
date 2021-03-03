文件操作：
1. 创建文件 : touch
2. 删除: rm
3. 重命名: mv
4. 移动文件: mv

文件夹：
1. 创建文件夹：mkdir mkdir a/b/c
2. 删除文件夹：rmdkir rm -rf xx 
3. 重命名：mv
4. 移动文件：mv

文件查看：
1. cat：比较小的文件
2. more/less
3. head / tail
4. vi

搜索：
1. 搜索命令：which / whereis
2. 搜索文件: locate(索引/updatedb) / find
3. grep
  搜索包含 hadoop这个词的所有文件，并打印出所在文件和行数
    （17892349234900）
    
系统命令：
1. top: 查看系统的总体运行状况
2. free -m: 查看内存使用率
3. df -h: 查看磁盘使用率
4. :查看某个文件夹占用的磁盘空间

权限：
1. chmod：改变权限位
2. chown：改变文件所属人
3. chgrp: 改变文件所属组

压缩/解压缩:
1. tar
  2.unzip 需要apt-get安装

进程/线程：
1. ps -aux | grep xxx: 打印进程信息。
2. kill: 杀死进程。
    -15： 
    -9： 强制杀掉

开关机命令：
1. reboot: 重启
2. shutdown: 关机

重定向：
1. > 重定向 
    2 黑洞：/dev/null

环境变量
  键值对
1. key -> value
2. $PATH这个环境变量下的所有路径存放可执行程序
3. 定义环境变量：
    export VAR=XXX
    全局（所有用户）：/etc/profile 不建议去改这里面的内容
    用户：.profile .bash_profile .bashrc 用户目录下面
    
4. 定义环境变量之后想让环境变量生效
  1. 重连ssh
  2. source .bash_profile
    
5. 更改path
    export PATH=$PATH:/xxx

管道：
1. | 将上一个命令的输出作为下一个命令的输入