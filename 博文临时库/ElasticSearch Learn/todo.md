- typora notion vscode 

- git
- java
- ssr chrome 不必





- mac 快捷键
  - https://it.corp.kuaishou.com/webdoc/view/Pub4028c80c651e50df01652310a9370124.html
- mac 终端



- 备份之前的 settings.xml 文件，好习惯要养成
- 禁止在服务器等公共位置设置ssh协议访问，会导致任何登陆用户使用你的账号访问代码。
- 公钥 ~/.ssh/id_rsa.pub https://git.corp.kuaishou.com/-/profile/keys



- 软连接
- brew

- linux 命令！







- es + kibana +docker
- es demo run
- kibana help write complex es  code





- 查看所有的容器命令如下：

  ```
  $ docker ps -a
  ```

- 使用 

  ```
  $ docker start/stop/restart b750bbbcfd88 
  ```

- 希望 docker 的服务是在后台运行的，我们可以过 **-d** 指定容器的运行模式。

  ```
  $ docker run -itd --name ubuntu-test ubuntu /bin/bash
  ```

- ```
  docker attach 1e560fca3906   这个容器退出，会导致容器的停止
  ```

- ```
  docker exec -it 243c32535da7  从这个容器退出，容器不会停止
  ```

- ```
  docker rm -f 1e560fca3906
  ```







- es 节点 集群 
- 主节点 任何节点 
- 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。？
- 垂直扩容和水平扩容
- 副本分片
- *cluster* 、 *node* 、 *shard*
- 文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互
- 索引内任意一个文档都归属于一个主分片
  - ？所以主分片的数目决定着索引能够保存的最大数据量
- 一个 集群 是一组拥有相同 `cluster.name` 的节点？
- 

