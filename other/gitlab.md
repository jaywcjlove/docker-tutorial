在 Docker 中安装 Gitlab 教程，[官方文档](https://docs.gitlab.com/omnibus/docker/)

## 下载镜像

```bash
docker pull gitlab/gitlab-ce
```

## 运行容器

```bash
sudo docker run --detach \
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

由于端口冲突，重新映射了一个端口 `2222`

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

添加备份脚本 `vi ~/_docker/gitlab/gitlab.backup.sh`

```shell
#！ /bin/bash
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
```