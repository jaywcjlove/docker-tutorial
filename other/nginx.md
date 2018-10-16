
## 查找镜像

在 docker hub 中查找 nginx 相关镜像。

```bash
$ docker search nginx

# INDEX       NAME                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
# docker.io   docker.io/nginx                 Official build of Nginx.                        7006      [OK]
# docker.io   docker.io/jwilder/nginx-proxy   Automated Nginx reverse proxy for docker c...   1137                 [OK]
```

## 下载镜像

拉取官方镜像，其中上面的非官方镜像是用户们根据自己的需要制作的镜像，方便大家的使用。

```bash
$ docker pull nginx
# Using default tag: latest
# Trying to pull repository docker.io/library/nginx ...
# latest: Pulling from docker.io/library/nginx
# bc95e04b23c0: Pull complete
# ...
# Digest: sha256:004ac1d5e791e705f12a1
```

## 启动容器

利用这个镜像启动一个新的容器

```bash
docker run --name my-nginx -d -p 8080:80 nginx
# faaed6a2d63af248961aab59713e515c76aea447
```

## 查看日志

查看容器运行日志

```
docker logs my-nginx
```

## 运行容器示例一

启动一个更复杂Nginx的例子：

```bash
# 上面的命令将本地文件中的 nginx.conf 配置文件挂载到容器，并且将要展示的静态页面也挂载到容器。
docker run --name my-nginx \ 
    -v /host/path/nginx.conf:/etc/nginx/nginx.conf:ro \
    -v /some/html:/usr/share/nginx/html:ro \
    -p 8080:80 \
    -d nginx
```

`-v` 参数语法为 `-v host dir:container dir[:ro|rw]`

- --name 为容器取一个名字
- -p 参数语法为 `-p host port:container port`; -p 8080:80 将主机上的8080端口绑定到容器上的80端口，因此在主机中访问8080端口时其实就是访问 nginx 容器的80端口
- -d 后台运行容器

## 运行容器示例二

新需求，nginx 容器需要加载 [Let's Encrypt](https://www.sslforfree.com/) 提供的免费 SSL 证书，需要挂载证书目录，需要挂载一个静态资源目录。

```bash
# 创建目录 nginx, 用于存放 nginx 的相关东西
mkdir -p ~/_docker/nginx
# 拉取官方的镜像
docker pull nginx
```

运行容器

```bash
# 拷贝容器中的内容
docker container cp webserver:/etc/nginx/conf.d $HOME/_docker/nginx
docker container cp webserver:/usr/share/nginx/html $HOME/_docker/nginx

# 运行容器并挂载目录
docker run -d --name webserver  \
  --restart always \
  -p 443:443 -p 80:80 \
  -v $HOME/_docker/nginx/html:/usr/share/nginx/html \
  -v $HOME/_docker/nginx/conf.d:/etc/nginx/conf.d \
  -v /etc/letsencrypt:/etc/letsencrypt:rw \
  -v /home/www/:/home/www:rw \
  -d \
  nginx

# 命令行进入容器
docker exec -it webserver /bin/bash
```

- **--rm**：容器停止运行后，自动删除容器文件
- **-d**：在后台运行
- **-p 802:80**：将容器的 `80` 端口映射到主机的 `802` 端口
- **--name webserver**：将容器命名为 `webserver`
- **-v $HOME/_docker/nginx/html:/usr/share/nginx/html**：将主机中当前目录下的 `html` 挂载到容器的 `/html` 
- **-v $HOME/_docker/nginx/conf.d:/etc/nginx/conf.d**：将主机中当前目录下的 `nginx` 配置，挂载到容器的 `/etc/nginx/conf.d`
- **-v $HOME/docker/nginx/logs:/wwwlogs**：将主机中当前目录下的logs挂载到容器的/wwwlogs

默认挂载的路径权限为读写。如果指定为只读可以用：ro   
重载 nginx 配置

```bash
docker exec -it webserver /etc/init.d/nginx reload
```
