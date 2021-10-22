Rancher 单节点安装
===

### Rancher 2.5.x 及之后的版本

登录到 Linux 主机，然后运行下面这个非常简洁的安装命令。

与 2.4.x 或之前的版本相比，使用docker run命令安装 Rancher 2.5.x 时，需要添加 `--privileged` 标志变量，启用特权模式安装 Rancher。

```
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  rancher/rancher:latest
```

### 使用已有的可信证书

```bash
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    -v /etc/letsencrypt/live/xxxx.cn/cert.pem:/etc/rancher/ssl/cert.pem \
    -v /etc/letsencrypt/live/xxxx.cn/key.pem:/etc/rancher/ssl/key.pem \
    -v /etc/letsencrypt/live/xxxx.cn/cacerts.pem:/etc/rancher/ssl/cacerts.pem \
    -v $HOME/docker/rancher:/var/lib/rancher \
    --privileged \
    rancher/rancher:latest
```

### 使用 Let's Encrypt 证书

```bash
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    --privileged \
    rancher/rancher:latest \
    --acme-domain rancher.mydomain.com
```

### 持久化数据


```bash
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    -v $HOME/docker/rancher:/var/lib/rancher \
    --privileged \
    rancher/rancher:latest \
    --acme-domain rancher.mydomain.com
```