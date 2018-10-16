
拉取官方的镜像，标签为`5.7`，[Docker官方资料](https://docs.docker.com/samples/library/mysql/#-via-docker-stack-deploy-or-docker-compose)、[MySQL 官方资料](https://dev.mysql.com/doc/refman/8.0/en/docker-mysql-more-topics.html)

```bash
docker pull mysql:5.7.23
# Trying to pull repository docker.io/library/mysql ...
# 5.7: Pulling from docker.io/library/mysql
# 85b1f47fba49: Already exists
# f34057997f40: Pull complete
# ....
# Digest: sha256:bfb22e93ee87c6aab6c1c9a4e70f28fa289f9ffae9fe8e173
```

## 运行容器示

- data 目录将映射为 `mysql` 容器配置的数据文件存放路径
- conf.d 目录里的配置文件将映射为 `mysql` 容器的配置文件

```bash
docker run --name mysql \
  -p 3306:3306 \
  -v $HOME/_docker/mysql/conf.d:/etc/mysql/conf.d \
  -v $HOME/_docker/mysql/data:/var/lib/mysq \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -d mysql:5.7.23
```

- `--name mysql`：容器名字为 `mysql`
- `-p 3306:3306`：将容器的 3306 端口映射到主机的 3306 端口
- `-v $HOME/_docker/mysql/conf.d`：将主机当前目录下的 `~/_docker/mysql/conf.d` 挂载到容器的 /etc/mysql/conf.d
- `-v $HOME/_docker/mysql/data`：将主机当前目录下的 data 目录挂载到容器的 `/var/lib/mysqs`
- `-e MYSQL_ROOT_PASSWORD=123456`：初始化root用户的密码

## 查看日志

docker exec 命令允许您在 Docker 容器内运行命令。 以下命令行将在 mysql 容器中为您提供一个 bash shell：

```bash
$ docker exec -it mysql bash
```

MySQL Server日志可通过 Docker 的容器日志获得：

```bash
$ docker logs mysql
```