Docker Compose
---


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

## 卸载

```bash
sudo rm /usr/local/bin/docker-compose
pip uninstall docker-compose
```