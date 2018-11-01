Docker 部署 Gitlab
---

在 Docker 中安装 Gitlab 教程，[官方文档](https://docs.gitlab.com/omnibus/docker/)，如果你想使用原生安装，教程在这里：[CentOS7安装维护Gitlab](https://github.com/jaywcjlove/handbook/blob/9adc40d9e684928ee68d3301afbd78eee7fe3816/CentOS/CentOS7%E5%AE%89%E8%A3%85%E7%BB%B4%E6%8A%A4Gitlab.md)

## 下载镜像

```bash
docker pull gitlab/gitlab-ce
```

## 运行容器

```bash
sudo docker run \
  --hostname gitlab.example.com \
  --publish 8443:443 --publish 8081:80 -p 2222:22 \
  --name gitlab \
  --restart always \
  --volume $HOME/_docker/gitlab/config:/etc/gitlab \
  --volume $HOME/_docker/gitlab/logs:/var/log/gitlab \
  --volume $HOME/_docker/gitlab/data:/var/opt/gitlab \
  -v /etc/localtime:/etc/localtime \
  -d \
  gitlab/gitlab-ce:latest
```

由于端口冲突，重新映射了一个端口 `2222`，如果不想麻烦，可以事先将 ssh 端口号更改成别的端口号，[修改ssh端口号的方法](https://github.com/jaywcjlove/handbook/blob/9adc40d9e684928ee68d3301afbd78eee7fe3816/CentOS/%E4%BF%AE%E6%94%B9ssh%E7%AB%AF%E5%8F%A3%E5%8F%B7%E7%9A%84%E6%96%B9%E6%B3%95.md)

```bash
# 要从之前的：
git clone git@gitlab.example.com:myuser/awesome-project.git
# 改为明确使用 `ssh://` 的 `URL` 方式。
git clone ssh://git@gitlab.example.com:2222/myuser/awesome-project.git
```

为了克隆不必麻烦，保留 `gitlab` 的 `22` 端口映射，将主机的 `sshd` 的 `22` 端口映射到容器中去。将主机的 sshd 端口更改为 `2222`

编辑文件 `/etc/ssh/sshd_config`，将其中的 `#Port 22` 注释去掉，将数字 `22` 更改为 `2222`，执行下面的命令重启 `sshd` 服务

```bash
systemctl restart sshd
```

防火墙的规则，添加开发 `2222` 端口

```bash
iptables -A INPUT -p tcp --dport 2222 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 2222 -j ACCEPT
# 再查看下是否添加上去, 看到添加了
iptables -L -n
```

如果此容器由于权限问题而无法启动，请尝试通过执行以下操作来修复它：

```bash
docker exec -it gitlab update-permissions
docker restart gitlab
```
## 容器手动备份


```bash
# 第一种进行入容器执行命令的方法进行手工备份
docker exec -it 容器名或容器id bash  # 进入容器
gitlab-rake gitlab:backup:create   # 执行gitlab备份命令

# 第二种直接使用外部命令执行，一次完成
docker exec 容器名或容器id gitlab-rake gitlab:backup:create
```


### 自动备份

通过在宿主机上使用 crontab 使用备份命令实现自动备份

添加备份脚本 `vi ~/_docker/gitlab/gitlab.backup.sh`，将下面内容添加到脚本中，保存之后添加可执行权限 `chmod +x gitlab.backup.shs`

```shell
#!/bin/bash
case "$1" in 
  start)
    docker exec gitlab-ce11.2.3 gitlab-rake gitlab:backup:create
    ;;
esac
```

创建定时执行计划

```bash
crontab -e # 进入编辑，添加下面内容

# 每天2点备份 gitlab 数据
0 2 * * * $HOME/_docker/gitlab/gitlab.backup.sh start
# *  *  *  *  *  command
# 分  时  日  月  周  命令

# 其中，
# 第1列表示分钟，1~59，每分钟用*表示
# 第2列表示小时，1~23，（0表示0点）
# 第3列表示日期，1~31
# 第4列表示月份，1~12
# 第5列表示星期，0~6（0表示星期天）
# 第六列表示要运行的命令。
```

上面两行保存之后，重新载入配置

```bash
service crond reload
# or
systemctl reload crond.service
```

### 备份保留七天

设置只保存最近7天的备份，编辑 `vi $HOME/_docker/gitlab/config/gitlab.rb` 配置文件，找到如下代码，删除注释 `#` 保存

```bash
# /etc/gitlab/gitlab.rb 配置文件 修改下面这一行
gitlab_rails['backup_keep_time'] = 604800  
```

重新加载gitlab配置文件

```bash
docker exec 容器名或容器id gitlab-ctl reconfigure  
```

### 容器管理

```bash
docker stop gitlab # 停止容器
docker rm gitlab # 删除容器
docker start gitlab # 启动容器
# 编辑 gitlab 容器配置
docker exec -it gitlab vim /etc/gitlab/gitlab.rb
# 重启 gitlab 容器
docker restart gitlab
```

### 通过 Docker Compose 按照

使用 Docker Compose，可以轻松配置，安装和升级基于 Docker 的 GitLab 安装，[官方教程在这里](https://docs.gitlab.com/omnibus/docker/README.html#install-gitlab-using-docker-compose)。

**第一步：**Docker [官方教程安装](https://docs.docker.com/compose/install/) Docker Compose。

**第二步：** 创建 `docker-compose.yml` 文件，将下面配置复制到文件中 (或者下载[官方示例](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/docker/docker-compose.yml)):


```yaml
web:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'gitlab.example.com'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'https://gitlab.example.com'
  ports:
    - '8081:80'
    - '8443:443'
    - '22:22'
  volumes:
    - ./gitlab-data/config:/etc/gitlab
    - ./gitlab-data/logs:/var/log/gitlab
    - ./gitlab-data/data:/var/opt/gitlab
    - /etc/localtime:/etc/localtime
```

**第三步：** 确保与 `docker-compose.yml` 文件同一目录下运行 `docker-compose up -d` 启动 Gitlab


### 使用 Docker Swarm

[官方教程](https://docs.gitlab.com/omnibus/docker/README.html#deploy-gitlab-in-a-docker-swarm) 创建 `docker-compose.yml` 文件


```yaml
version: "3.6"
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    ports:
      - "22:22"
      - "80:80"
      - "443:443"
    volumes:
      - /srv/gitlab/data:/var/opt/gitlab
      - /srv/gitlab/logs:/var/log/gitlab
      - /srv/gitlab/config:/etc/gitlab
    environment:
      GITLAB_OMNIBUS_CONFIG: "from_file('/omnibus_config.rb')"
    configs:
      - source: gitlab
        target: /omnibus_config.rb
    secrets:
      - gitlab_root_password
  gitlab-runner:
    image: gitlab/gitlab-runner:alpine
    deploy:
      mode: replicated
      replicas: 4
configs:
  gitlab:
    file: ./gitlab.rb
secrets:
  gitlab_root_password:
    file: ./root_password.txt
```

创建 `gitlab.rb` 文件

```rb
external_url 'https://my.domain.com/'
gitlab_rails['initial_root_password'] = File.read('/run/secrets/gitlab_root_password')
gitlab_rails['backup_keep_time'] = 604800  
```

创建 `root_password.txt` 文件

```
MySuperSecretAndSecurePass0rd!
```

确保您与 `docker-compose.yml` 在同一目录中并运行：

```bash
docker stack deploy --compose-file docker-compose.yml mystack
```