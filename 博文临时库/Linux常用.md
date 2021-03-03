- Linux各大发行版本关联

- 各个主要版本的设置、系统安装、分区方案、桌面环境设置、网络设置、虚拟机设置、服务器设置、软件源设置、ssh等

- 可以通过type命令来查询命令所在的目录。

  ``` bash
  type cd grep
  ```

  



# ubuntu 18.04 开机启动

您可以创建一个systemd服务。

https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html

创建一个文件`/etc/systemd/system/ethtool.service`：

```bsh
[Unit]
Description=ethtool script

[Service]
ExecStart=/path/to/yourscript.sh

[Install]
WantedBy=multi-user.target
```

和脚本`/path/to/yourscript.sh`（别忘了`chmod +x`）

```bsh
#!/bin/bash
ethtool --offload <net> rx off
```

启用您的服务

```bsh
systemctl enable ethtool
```

它将以root身份在启动时运行。

---

最近在ubuntu18.04中设置开机启动一个GUI程序，试了很多网上的办法，都无法使用，后来找到了gnome设置自启动程序的办法，发现可以用。直接在终端中执行：gnome-session-properties，会弹出一个“启动应用程序首选项”的菜单，然后点击添加要开机自启动的程序或脚本的名称和位置就可以了，重启就自动启动这个程序了。

