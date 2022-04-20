企业聊天工具 Rocket.Chat 开源 IM 系统
===

[Rocket.Chat](https://github.com/rocketchat/) 是特性最丰富的 Slack 开源替代品之一。

主要功能：群组聊天，直接通信，私聊群，桌面通知，媒体嵌入，链接预览，文件上传，语音/视频 聊天，截图等等。

Rocket.Chat 原生支持 Windows，Mac OS X ，Linux，iOS 和 Android 平台。Rocket.Chat 通过 hubot 集成了非常流行的服务，比如 GitHub，GitLab，Confluence，JIRA 等等。

高级的特性包括：OTR 消息，XMPP 多用户聊天，Kerberos 认证，p2p 文件分享等等。

安装顺执行下面命令启动服务，[官方部署文档](https://rocket.chat/docs/installation/docker-containers/docker-compose/)

## docker-compose.yml

```yml
version: '2'

services:
  rocketchat:
    image: rocketchat/rocket.chat:latest
    restart: unless-stopped
    volumes:
      - ./uploads:/app/uploads
    environment:
      - PORT=3500
      - ROOT_URL=http://localhost:3500
      - MONGO_URL=mongodb://mongo:27017/rocketchat
      - MONGO_OPLOG_URL=mongodb://mongo:27017/local
      - MAIL_URL=smtp://smtp.email
#       - HTTP_PROXY=http://proxy.domain.com
#       - HTTPS_PROXY=http://proxy.domain.com
    depends_on:
      - mongo
    ports:
      - 3500:3500
    labels:
      - "traefik.backend=rocketchat"
      - "traefik.frontend.rule=Host: your.domain.tld"

  mongo:
    image: mongo:3.2
    restart: unless-stopped
    volumes:
     - ./data/db:/data/db
     #- ./data/dump:/dump
    command: mongod --smallfiles --oplogSize 128 --replSet rs0 --storageEngine=mmapv1
    labels:
      - "traefik.enable=false"

  # this container's job is just run the command to initialize the replica set.
  # it will run the command and remove himself (it will not stay running)
  mongo-init-replica:
    image: mongo:3.2
    command: 'mongo mongo/rocketchat --eval "rs.initiate({ _id: ''rs0'', members: [ { _id: 0, host: ''localhost:27017'' } ]})"'
    depends_on:
      - mongo

  # hubot, the popular chatbot (add the bot user first and change the password before starting this image)
  hubot:
    image: rocketchat/hubot-rocketchat:latest
    restart: unless-stopped
    environment:
      - ROCKETCHAT_URL=rocketchat:3500
      - ROCKETCHAT_ROOM=GENERAL
      - ROCKETCHAT_USER=bot
      - ROCKETCHAT_PASSWORD=botpassword
      - BOT_NAME=bot
  # you can add more scripts as you'd like here, they need to be installable by npm
      - EXTERNAL_SCRIPTS=hubot-help,hubot-seen,hubot-links,hubot-diagnostics
    depends_on:
      - rocketchat
    labels:
      - "traefik.enable=false"
    volumes:
      - ./scripts:/home/hubot/scripts
  # this is used to expose the hubot port for notifications on the host on port 3001, e.g. for hubot-jenkins-notifier
    ports:
      - 3001:8080

  #traefik:
  #  image: traefik:latest
  #  restart: unless-stopped
  #  command: traefik --docker --acme=true --acme.domains='your.domain.tld' --acme.email='your@email.tld' --acme.entrypoint=https --acme.storagefile=acme.json --defaultentrypoints=http --defaultentrypoints=https --entryPoints='Name:http Address::80 Redirect.EntryPoint:https' --entryPoints='Name:https Address::443 TLS.Certificates:'
  #  ports:
  #    - 80:80
  #    - 443:443
  #  volumes:
  #    - /var/run/docker.sock:/var/run/docker.sock
```

## 服务启动

```bash
docker-compose up -d mongo
# 第一次启动mongo时，您还需要初始化它才能使用Rocket.Chat。
# 确保mongo处于运行状态，然后：
docker-compose up -d mongo-init-replica
# mongo 启动完成启动主程序
docker-compose up -d rocketchat
# 如果你想要一个机器人，所以你不必自己说话，
# 在你创建了一个管理员用户和一个机器人用户之后，再次编辑文件 docker-compose.yml 来改变变量：
# - ROCKETCHAT_USER 和 ROCKETCHAT_PASSWORD 在 hubot 部分然后启动 hubot：
docker-compose up -d hubot
```