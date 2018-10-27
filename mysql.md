
## 下载镜像

拉取官方的镜像，标签为`5.7`，[Docker官方资料](https://docs.docker.com/samples/library/mysql/#-via-docker-stack-deploy-or-docker-compose)、[MySQL 官方资料](https://dev.mysql.com/doc/refman/8.0/en/docker-mysql-more-topics.html)，[MySQL镜像](https://hub.docker.com/_/mysql/)

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

```bash
docker run --name mysql \
  -p 3306:3306 \
  -v $HOME/_docker/mysql/conf.d:/etc/mysql/conf.d \
  -v $HOME/_docker/mysql/data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -d mysql:5.7.23
```

- `--name mysql`：容器名字为 `mysql`
- `-p 3306:3306`：将容器的 3306 端口映射到主机的 3306 端口
- `-v $HOME/_docker/mysql/conf.d`：将主机当前目录下的 `~/_docker/mysql/conf.d` 挂载到容器的 `/etc/mysql/conf.d`，这个是挂载配置目录
- `-v $HOME/_docker/mysql/data`：将主机当前目录下的 data 目录挂载到容器的 `/var/lib/mysqs`，为数据文件存放路径
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

## 修改配置

```ini
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
# 忽略数据库表名的大小写区分
lower_case_table_names = 1
# 解决时区与中国时区不至问题
default-time_zone=+8:00
# 设置服务ID
server-id=1
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
# 开启 binlog，log_bin 等于 server-id
log_bin=1
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

#log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

通过[容器名字]或者[容器 ID]来重启 MySQL，让配置生效。

```bash
docker restart mysql
```

## 进入数据库

```bash
# 进入 mysql 容器
docker exec -it mysql /bin/bash
# 通过 mysql 命令登陆
mysql -uroot -p

# 查看是否开启了binlog
show binary logs;
show variables like '%server_id%';
```