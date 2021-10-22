部署企业内部聊天工具Rocket.Chat开源IM系统
===

Rocket.Chat 是特性最丰富的 Slack 开源替代品之一。

主要功能：群组聊天，直接通信，私聊群，桌面通知，媒体嵌入，链接预览，文件上传，语音/视频 聊天，截图等等。

Rocket.Chat 原生支持 Windows，Mac OS X ，Linux，iOS 和 Android 平台。Rocket.Chat 通过 hubot 集成了非常流行的服务，比如 GitHub，GitLab，Confluence，JIRA 等等。

高级的特性包括：OTR 消息，XMPP 多用户聊天，Kerberos 认证，p2p 文件分享等等。

安装顺执行下面命令启动服务，[官方部署文档](https://rocket.chat/docs/installation/docker-containers/docker-compose/)

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