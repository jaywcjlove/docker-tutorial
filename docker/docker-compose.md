Docker Compose
===


`docker-compose` 是用来做 `docker` 的多容器控制，这个工具是用于 docker 自动化的东西，将多个 docker 容器的操作命令，简化成一条命令，自动完成配置中的容器启动。

## 安装

[官方安装教程](https://docs.docker.com/compose/install/#install-compose)

```bash
# 在 Linux CentOS 7 系统中安装
# 如果 curl 不存在需要安装， `yum install curl`
sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 给 docker-compose 执行权限
sudo chmod +x /usr/local/bin/docker-compose
# 测试是否安装成功
docker-compose --version
# docker-compose version 1.22.0, build 1719ceb
```

## 服务运行

```bash

# 停止当前服务
docker-compose -p intelligent-community-dev -f docker-compose.dev.yml down
# 使用 docker-compose 后台启动服务
docker-compose -f docker-compose.dev.yml pull
docker-compose -p intelligent-community-dev -f docker-compose.dev.yml up -d

docker stack deploy --compose-file=docker-compose.yml my-name
docker stack services my-name # 部署成功之后查看详情
docker stack deploy	  # 部署新的堆栈或更新现有堆栈
docker stack ls	      # 列出现有堆栈
docker stack ps	      # 列出堆栈中的任务
docker stack rm	      # 删除一个或多个堆栈
docker stack services	# 列出堆栈中的服务
```

## 卸载

```bash
sudo rm /usr/local/bin/docker-compose
pip uninstall docker-compose
```