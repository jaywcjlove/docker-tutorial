Harbor
===

[Harbor](https://goharbor.io/) 是 `VMware` 公司开源了企业级 `Registry` 项目, 其的目标是帮助用户迅速搭建一个企业级的 `Docker registry` 服务。

由于 Harbor 是基于 Docker Registry V2 版本，所以 docker 版本必须 `>=1.10.0` [docker-compose](https://docs.docker.com/compose/install/#prerequisites) `>=1.6.0`

Github: [goharbor/harbor](https://github.com/goharbor/harbor)，官方[预览示例](https://demo.goharbor.io/)

对硬件需求

> CPU  => 最小 2CPU/4CPU(首选)  
> Mem  => 最小 4GB/8GB(首选)  
> Disk => 最小 40GB/160G(首选)  

## 下载安装包

CentOS中通过 [docker-compose](https://docs.docker.com/compose/install/#prerequisites) 安装部署。

```bash
# 下载最新版 `Docker Compose`
sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 对二进制文件应用可执行权限：
sudo chmod +x /usr/local/bin/docker-compose
# 测试是否安装成功
docker-compose --version
# docker-compose version 1.22.0, build f46880fe
```

可以从[发布页面](https://github.com/goharbor/harbor/releases)下载安装程序的二进制文件，选择在线或离线安装程序，使用tar命令解压缩包，天朝人民下面这种方式安装可能要翻墙，推荐这种方式，因为上面也不见得能下载下来。

```bash
# 下载离线安装包
wget https://storage.googleapis.com/harbor-releases/release-1.6.0/harbor-offline-installer-v1.6.0.tgz
# 解压缩包
tar xvf harbor-offline-installer-v1.6.0.tgz
```

## 修改配置

进去 `vim harbor/harbor.cfg` 修改文件相关配置。

```bash
# hostname 设置访问地址，支持IP，域名，主机名，禁止设置127.0.0.1
hostname = reg.mydomain.com
# 访问协议，可设置 http,https
ui_url_protocol = http

# 邮件通知, 配置邮件通知。
email_identity =
email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false
email_insecure = false

# harbor WEB UI登陆使用的密码
harbor_admin_password = Harbor12345

# 认证方式，这里支持多种认证方式，默认是 db_auth ，既mysql数据库存储认证。
# 这里还支持 ldap 以及 本地文件存储方式。
auth_mode = db_auth
# ldap 服务器访问地址。
ldap_url = ldaps://ldap.mydomain.com
ldap_basedn = uid=%s,ou=people,dc=mydomain,dc=com

# mysql root 账户的 密码
db_password = root123
self_registration = on
use_compressed_js = on
max_job_workers = 3 
verify_remote_cert = on
customize_crt = on
```

## 运行安装脚本

配置设置完成运行安装脚本，⚠️ 注意如果你事先部署了 nginx 需要停掉，避免端口冲突

```bash
sudo ./install.sh
```

要更改 Harbour 的配置，请先停止现有的 Harbor 实例并更新 `harbour.cfg`。 然后运行 `prepare` 脚本来填充配置，最后重新创建并启动Harbor的实例：

```bash
$ docker-compose down -v
# 注：其实上面是停止 docker-compose.yml 中定义的所有容器
$ vim harbor.cfg
$ prepare
$ docker-compose up -d
```

通过 http://192.168.188.222 就可以访问 Harbour 服务了

## 配置修改

因为 `harbor` 默认端口为 `80`，而大多数时候是不希望使用 `80` 端口的，修改端口方法如下

```bash
# vim docker-compose.yml

proxy:
  image: vmware/nginx-photon:v1.5.1
  container_name: nginx
  restart: always
  volumes:
    - ./common/config/nginx:/etc/nginx:z
  networks:
    - harbor
  ports:
    - 8070:80
    - 443:443
```

修改 `common/templates/registry/config.yml` 文件

```
# vim common/templates/registry/config.yml
auth:
  token:
    issuer: harbor-token-issuer
    realm: $public_url:8070/service/token
    rootcertbundle: /etc/registry/root.crt
service: harbor-registry
```

## 使用 harbor 

```bash
# 镜像推送
docker login 192.168.188.222:8070
# 查看 cat ~/.docker/config.json
# 镜像打包时候需要按一定规则 tag
docker pull nginx
docker tag nginx 192.168.188.222:8070/library/nginx:latest
docker push 192.168.188.222:8070/library/nginx
docker rmi -f 192.168.188.222:8070/library/nginx:latest
```

若推送镜像报以下错误:

> Error response from daemon: Get https://192.168.188.222:8070/v1/users/: http: server gave HTTP response to HTTPS client

原因为，`docker` 默认使用的是 `https` 协议，而搭建的 `Harbor` 是 `http` 提供服务的，所以要配置可信任。PS：如果 `Harbor` 是 `https` 的就不会报该错误。

方法1

```bash
# vim /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd --insecure-registry=192.168.188.222:8070
# 修改完毕后按安装步骤走一遍即可。
```

方法2

```bash
# vim /etc/docker/daemon.json

{
 "registry-mirrors": ["http://xxx.m.daocloud.io"],
 "insecure-registries":["192.168.100.127:8070"]
}
```

```json
{
  "insecure-registries" : [
    "docker.google.com"
  ],
  "debug" : true,
  "experimental" : true
}
```

客户机docker启动时候带上 `--insecure-registry=docker.xxx.com` 强制 `docker login` 走 `http` 的 `80` 端口，就可以正常 `push` 了
