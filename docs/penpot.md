Penpot
===

开源设计和原型平台

## 启动 `Penpot`

第一步，您需要获取 `docker-compose.yaml` 文件。 您可以从 [`Penpot`](https://raw.githubusercontent.com/penpot/penpot/main/docker/images/docker-compose.yaml) 存储库下载它。

```bash
wget https://raw.githubusercontent.com/penpot/penpot/main/docker/images/docker-compose.yaml
```

or

```bash
curl -o docker-compose.yaml https://raw.githubusercontent.com/penpot/penpot/main/docker/images/docker-compose.yaml
```

然后只需启动 `Penpot`：


```bash
docker compose -p penpot -f docker-compose.yaml up -d
```

最后它将开始监听 http://localhost:9001


## 停止 `Penpot`

如果你想停止运行 Penpot，只需输入

```bash
docker compose -p penpot -f docker-compose.yaml down
```