```
docker pull elasticsearch:6.8.2
docker pull kibana:6.8.2

docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -d elasticsearch:6.8.2

docker exec -it elasticsearch /bin/bash
cd /usr/share/elasticsearch/config/
vi elasticsearch.yml

# 添加 已解决跨域问题
http.cors.enabled: true
http.cors.allow-origin: "*"

docker restart elasticsearch
```

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

