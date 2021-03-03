---
title: Huginn 部署 
date: 2020-12-21 14:52:00 
status: PUBLISHED
comments: true 
description: 记录一下部署huginn的过程 
tags: 
- huginn  
- docker 
categories: 
- 软件开发环境与工具 
---

采用docker部署

https://www.jianshu.com/p/3e0bdde3cce4

# 安装 docker

step 1  安装必要的一些系统工具

```bash
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
```

step 2 安装GPG证书

```bash
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

step 3 写入软件源信息

```bash
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```

step 4  更新并安装Docker-CE

```bash
sudo apt-get -y update
sudo apt-get -y install docker-ce
```

step 5 验证docker 安装成功。

```bash
 docker --version
```

# 安装 huginn

前台启动

``` bash
docker run -it -p 3000:3000 huginn/huginn
```

后台启动

``` bash
docker run -d -it -p 3000:3000 huginn/huginn
```

- 即加一个 `-d` 选项

---

下载太慢可以启动阿里云镜像

https://blog.csdn.net/liu865033503/article/details/95936640

容器镜像服务》镜像中心》镜像加速器

# 配置 huginn

查看 docker 进程：

``` bash
docker ps
```

![image-20210117153216525](E:\_data\博文临时库\博文中的图片\记一次部署 huginn 的 docker 文件配置.png)

``` bash
docker exec -it 4382726ae373 bash
# 或者
docker exec -it charming_feistel bash
```

然后

``` bash
# 可能需要下载 vim
# apt-get update、apt-get install vim
# 如果不能下载可以以 root 用户进入（权益之计，不熟悉 docker ，勿喷）
# docker exec --user="root" -it 4382726ae373 bash
vim /app/.env
```

最后

``` bash
docker container restart 4382726ae373
```

## 启动邮件服务

如果是 gamil 则需要设置“安全性较低的应用的访问权限”

https://www.google.com/settings/security/lesssecureapps

# 其他

- 阿里云注意开启上面指定的 3000 端口

- nginx 配置

- https配置

  - 使用下列命令：

    ```bash
    certbot certificates
    ```

    就可以看到当前机器所有域名的证书情况，包括域名、 到期日、证书路径、私钥路径四条信息

- 本地使用docker安装的huginn和huginn.io含有的agent不一致

  - 本地含有的agent列表

  ``` bash
  "Select an Agent Type",
  "Change Detector Agent",
  "Evernote Agent",
  "Jabber Agent",
  "Weibo User Agent",
  "Twitter Action Agent",
  "Weather Agent",
  "User Location Agent",
  "Google Translation Agent",
  "Boxcar Agent",
  "Attribute Difference Agent",
  "Ftpsite Agent",
  "Witai Agent",
  "Phantom Js Cloud Agent",
  "Read File Agent",
  "Json Parse Agent",
  "Email Agent",
  "Aftership Agent",
  "Webhook Agent",
  "Dropbox Watch Agent",
  "Local File Agent",
  "Sentiment Agent",
  "Peak Detector Agent",
  "Jq Agent",
  "Website Agent",
  "Slack Agent",
  "Wunderlist Agent",
  "Twitter Publish Agent",
  "Twitter Search Agent",
  "Delay Agent",
  "Email Digest Agent",
  "Pushover Agent",
  "Human Task Agent",
  "Trigger Agent",
  "Scheduler Agent",
  "Growl Agent",
  "Tumblr Likes Agent",
  "Hipchat Agent",
  "Imap Folder Agent",
  "Commander Agent",
  "Dropbox File Url Agent",
  "Tumblr Publish Agent",
  "Twitter Favorites",
  "Pdf Info Agent",
  "Http Status Agent",
  "Jira Agent",
  "Twilio Agent",
  "Google Calendar Publish Agent",
  "Pushbullet Agent",
  "Digest Agent",
  "Basecamp Agent",
  "S3 Agent",
  "Telegram Agent",
  "Twitter Stream Agent",
  "Csv Agent",
  "Post Agent",
  "Weibo Publish Agent",
  "Java Script Agent",
  "Gap Detector Agent",
  "Public Transport Agent",
  "Adioso Agent",
  "Event Formatting Agent",
  "Stubhub Agent",
  "Rss Agent",
  "Data Output Agent",
  "De Duplication Agent",
  "Shell Command Agent",
  "Mqtt Agent",
  "Twilio Receive Text Agent",
  "Manual Event Agent",
  "Liquid Output Agent",
  "Twitter User Agent",
  ```

  - huginn.io 含有的

    ``` bash
    0: "Adioso Agent"
    1: "Aftership Agent"
    2: "Algorithmia Agent"
    3: "Attribute Difference Agent"
    4: "Basecamp Agent"
    5: "Boxcar Agent"
    6: "Brave Catalog Agent"
    7: "Change Detector Agent"
    8: "Commander Agent"
    9: "Csv Agent"
    10: "Data Output Agent"
    11: "De Duplication Agent"
    12: "Delay Agent"
    13: "Digest Agent"
    14: "Dropbox File Url Agent"
    15: "Dropbox Watch Agent"
    16: "Email Agent"
    17: "Email Digest Agent"
    18: "Epic Store New Free Game Agent"
    19: "Etherscan Agent"
    20: "Event Formatting Agent"
    21: "Evernote Agent"
    22: "Fitbit Agent"
    23: "Ftpsite Agent"
    24: "Gap Detector Agent"
    25: "Google Calendar Publish Agent"
    26: "Google Translation Agent"
    27: "Growl Agent"
    28: "Hipchat Agent"
    29: "Http Status Agent"
    30: "Human Task Agent"
    31: "Imap Folder Agent"
    32: "Instagram Agent"
    33: "Jabber Agent"
    34: "Java Script Agent"
    35: "Jira Agent"
    36: "Jq Agent"
    37: "Json Parse Agent"
    38: "Liquid Ext Agent"
    39: "Liquid Output Agent"
    40: "Local File Agent"
    41: "Manual Event Agent"
    42: "Moves Agent"
    43: "Moves Current Location Agent"
    44: "Mqtt Agent"
    45: "Naive Bayes Agent"
    46: "Northpass S3 Agent"
    47: "Pdf Info Agent"
    48: "Peak Detector Agent"
    49: "Phantom Js Cloud Agent"
    50: "Port Status Agent"
    51: "Post Agent"
    52: "Public Transport Agent"
    53: "Pushbullet Agent"
    54: "Pushover Agent"
    55: "R6 Server Status Agent"
    56: "R6 Top Suicide Agent"
    57: "Raw Webhook Agent"
    58: "Read File Agent"
    59: "Renault Ze Battery Agent"
    60: "Rss Agent"
    61: "S3 Agent"
    62: "Scheduler Agent"
    63: "Sentiment Agent"
    64: "Shell Command Agent"
    65: "Slack Agent"
    66: "Slackbot Agent"
    67: "Sms77 Agent"
    68: "Sms77 Voice Agent"
    69: "Stubhub Agent"
    70: "Telegram Agent"
    71: "The Division2 Server Status Agent"
    72: "Todoist Agent"
    73: "Todoist Close Item Agent"
    74: "Todoist Query Agent"
    75: "Transmission Agent"
    76: "Transport Opendata"
    77: "Trigger Agent"
    78: "Tumblr Likes Agent"
    79: "Tumblr Publish Agent"
    80: "Twilio Agent"
    81: "Twilio Receive Text Agent"
    82: "Twitter Action Agent"
    83: "Twitter Favorites"
    84: "Twitter Publish Agent"
    85: "Twitter Search Agent"
    86: "Twitter Stream Agent"
    87: "Twitter User Agent"
    88: "User Location Agent"
    89: "Weather Agent"
    90: "Webhook Agent"
    91: "Website Agent"
    92: "Website Metadata Agent"
    93: "Weibo Publish Agent"
    94: "Weibo User Agent"
    95: "Witai Agent"
    96: "Wunderlist Agent"
    97: "Xero Agent"
    ```

  - 不同的，区别的，本地少了的

    ``` bash
    0: "Algorithmia Agent"
    1: "Brave Catalog Agent"
    2: "Epic Store New Free Game Agent"
    3: "Etherscan Agent"
    4: "Fitbit Agent"
    5: "Instagram Agent"
    6: "Liquid Ext Agent"
    7: "Moves Agent"
    8: "Moves Current Location Agent"
    9: "Naive Bayes Agent"
    10: "Northpass S3 Agent"
    11: "Port Status Agent"
    12: "R6 Server Status Agent"
    13: "R6 Top Suicide Agent"
    14: "Raw Webhook Agent"
    15: "Renault Ze Battery Agent"
    16: "Slackbot Agent"
    17: "Sms77 Agent"
    18: "Sms77 Voice Agent"
    19: "The Division2 Server Status Agent"
    20: "Todoist Agent"
    21: "Todoist Close Item Agent"
    22: "Todoist Query Agent"
    23: "Transmission Agent"
    24: "Transport Opendata"
    25: "Website Metadata Agent"
    26: "Xero Agent"
    ```

    

