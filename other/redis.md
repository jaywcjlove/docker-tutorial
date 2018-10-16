
## 下载镜像

```bash
$ docker pull redis:4.0.11
```

## 运行容器

```bash
$ docker run -d --rm -p  6389:6379 --name redis2 redis:4.0.11 redis-server --appendonly yes
```

## 使用自己的配置

[Redis](https://hub.docker.com/_/redis/) 加载自己的配置文件，需要重新编译一个 `images`。

```bash
FROM redis:4.0.11
RUN mkdir -p /etc/redis
COPY ./redis.conf /etc/redis/redis.conf
CMD [ "redis-server", "/etc/redis/redis.conf" ]
EXPOSE 6379
```

创建 docker 镜像，镜像名字为 `redis`，标记 `4.0.11`

```bash
docker image build -t redis:4.0.11 .
```

如果你不需要更改配置，可以直接 `docker pull redis:4.0.11` 下载镜像。

```bash
# 先运行 redis
docker run -d --rm -p  6389:6379 --name redis2 redis:4.0.11 redis-server --appendonly yes
# docker 禁止用主机上不存在的文件挂载到 container 中已经存在的文件
docker container cp redis2:/etc/redis/redis.conf $HOME/_docker/redis/conf/redis.conf
# 完成拷贝文件，停止 redis 容器 --rm 参数表示停止删除 redis2 容器
docker stop redis2
# 这个时候，container 中已经存在的配置文件
docker run -d \
  -p 6389:6379 \
  --name redis2 \
  --restart always \
  -v $HOME/_docker/redis/data:/data \
  -v $HOME/_docker/redis/conf:/etc/redis \
  redis:4.0.11 redis-server --appendonly yes
# redis-server --appendonly yes 数据持久化
```

## 修改配置文件

修改配置文件 `$HOME/_docker/redis/conf/redis.conf` 将数据持久化目录指向 `/data` 目录，设置配置中的 `dir /data`。

```bash
vim ~/_docker/redis/redis.conf
```

## 重启容器让配置生效

```
docker restart redis2
```