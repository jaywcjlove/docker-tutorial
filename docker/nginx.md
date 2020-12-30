<p align="center">
  <a href="http://nginx.org/">
    <img width="210" src="https://raw.githubusercontent.com/jaywcjlove/nginx-tutorial/master/nginx.svg?sanitize=true" />
  </a>
</p>

Nginx 是一款面向性能设计的 HTTP 服务器，能反向代理 HTTP，HTTPS 和邮件相关(SMTP，POP3，IMAP)的协议链接。并且提供了负载均衡以及 HTTP 缓存。它的设计充分使用异步事件模型，削减上下文调度的开销，提高服务器并发能力。采用了模块化设计，提供了丰富模块的第三方模块。

所以关于 Nginx，有这些标签：「异步」「事件」「模块化」「高性能」「高并发」「反向代理」「负载均衡」

下面是 nginx 在 Docker 中的应用，这里还有 [nginx入门教程](https://github.com/jaywcjlove/nginx-tutorial) 供你参考

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
docker run --name my-nginx -d -p 8080:80 nginx /bin/bash
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

运行容器，记住先将目录中的 `/etc/nginx/conf.d/defautl.conf` 文件复制出来，不然会报错。

```bash
# 拷贝容器中的内容
docker container cp webserver:/etc/nginx/conf.d $HOME/docker/nginx
docker container cp webserver:/usr/share/nginx/html $HOME/docker/nginx

# 运行容器并挂载目录
docker run -d --name webserver  \
  --restart always \
  -p 443:443 -p 80:80 \
  -v $HOME/docker/nginx/html:/usr/share/nginx/html \
  -v $HOME/docker/nginx/conf.d:/etc/nginx/conf.d \
  -v /etc/letsencrypt:/etc/letsencrypt:rw \
  -v /etc/localtime:/etc/localtime:ro \
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
- **-v $HOME/docker/nginx/html:/usr/share/nginx/html**：将主机中当前目录下的 `html` 挂载到容器的 `/html` 
- **-v $HOME/docker/nginx/conf.d:/etc/nginx/conf.d**：将主机中当前目录下的 `nginx` 配置，挂载到容器的 `/etc/nginx/conf.d`
- **-v $HOME/docker/nginx/logs:/wwwlogs**：将主机中当前目录下的logs挂载到容器的/wwwlogs

默认挂载的路径权限为读写。如果指定为只读可以用：ro   
重载 nginx 配置

```bash
# 测试配置是否正确
docker exec -it webserver nginx -t
# 重新加载配置
docker exec -it webserver nginx -s reload
```
注意：⚠️ webserver 可以是 `容器名称` 或者 `容器ID`

## 启用 Gitlab Registry 功能

修改配置 `/etc/gitlab/gitlab.rb` 文件，将 `registry_external_url` 的值修改为 http://192.168.188.211:5008

```ruby
registry_external_url 'http://192.168.188.211:5008'
```

`registry_external_url` 这个地址是我们使用 `docker` 命令进行 `pull` 或者 `push` 镜像的仓库地址。

重启 `Gitlab` 后，可以在 `Gitlab` 左侧面板看到 `Container Registry` 的菜单。

按照 gitlab 给出的提示，我们先登录上 gitlab 的 registry：

```bash
docker login 192.168.188.211:5008
Username: ****
Password: **
```

注意：⚠️ 密码是需要通过 [Gitlab > User Settings > Access Tokens > Add a personal access token](http://gitlab.com/-/profile/personal_access_tokens) 生成一个 `personal_access_tokens` 而不是真正的密码


```
docker build -t 192.168.188.211:5008/docker/docker-static-service-template .
# 提交镜像
docker push 192.168.188.211:5008/docker/docker-static-service-template
```